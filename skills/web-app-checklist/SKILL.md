---
name: web-app-checklist
description: Audit a codebase for production / go-live readiness against a 250-item checklist covering secrets, auth & RBAC, input handling, business logic & payments, database access (RLS vs backend), AI/LLM risks, dependencies, hosting, HTTPS & headers, email, observability, release safety, legal and SEO. Every item is marked PASS / FAIL / N/A / CANNOT VERIFY with file:line evidence. Use this whenever the user asks if their app is ready to launch, ship, deploy or go live, wants a pre-launch / pre-production / security audit or readiness review, mentions a vibe-coded or AI-built app they're about to release, or asks "what am I missing before launch", even if they don't say "checklist" or "audit".
---

# Go-Live Audit

Audit the current codebase against the go-live checklist in `references/checklist.md`, and produce a report the user can act on.

AI-built apps often look finished while shipping secret keys in the browser bundle, skipping ownership checks, or leaving the database open. The user is asking you because they want those problems found **before** real users and real data arrive. A report that marks things PASS without proof is worse than none, because it gives false confidence. So the core rule is:

**No evidence, no PASS.** Every PASS cites `file:line` (or a command and its output). If you can't find evidence, the item is FAIL (the thing is clearly missing) or CANNOT VERIFY (it lives outside the repo, like DNS, provider dashboards, or backups).

## Workflow

### 1. Recon the stack (before reading the checklist)

Figure out what you're auditing, because it decides which items apply and where to look:

- **Language and framework:** `package.json`, `requirements.txt` / `pyproject.toml`, `composer.json`, `Gemfile`, `go.mod`.
- **How the database is reached.** This is the most important fork:
  - The **browser talks to the DB directly** (Supabase/Firebase/PostgREST/Hasura/PocketBase client in frontend code). Then RLS / Security Rules are the access control, and they're BLOCKERs.
  - **Only the backend talks to the DB** (ORM or driver on the server). Then authorization must live in the API layer.
- **Auth:** an auth provider (Auth.js/NextAuth, Clerk, Supabase Auth, Auth0, Django auth, Devise) or hand-rolled.
- **Payments:** Stripe/Paddle/Lemon Squeezy, and whether there are webhook handlers.
- **LLM usage:** OpenAI/Anthropic/etc. SDKs.
- **File uploads, websockets, GraphQL, background jobs, a mobile app.**
- **Hosting/deploy config:** `Dockerfile`, `docker-compose.yml`, `vercel.json`, `fly.toml`, `render.yaml`, `nginx.conf`, CI workflows, IaC.

Write the recon down in 5–10 lines. It goes at the top of the report.

### 2. Load the checklist

Read `references/checklist.md`. It's long (~700 lines), so read it phase by phase as you audit rather than holding all of it at once. Its **Verification Cheat Sheet** section has ready-made commands for items you can't verify from code.

If the user asked for a subset ("just check auth", "only security"), audit only those phases, and say in the report that the rest weren't checked.

### 3. Audit each item

For each item in the applicable phases:

1. Search for evidence (grep/glob, read the relevant files). Use the high-signal searches below to get started quickly, then read the actual code. A grep hit is a lead, not a verdict.
2. Mark one status:
   - **PASS:** evidence found that it's done correctly. Cite `file:line`.
   - **FAIL:** missing or done wrong. Cite `file:line` of the problem, or say what's absent and where you looked.
   - **PARTIAL:** done in some places but not others (e.g. 3 of 5 routes check ownership). List the gaps.
   - **N/A:** doesn't apply to this stack (e.g. RLS when only the backend touches the DB, GraphQL when there is none). Give the reason in one line.
   - **CANNOT VERIFY:** outside the repo (DNS, provider settings, backups, billing alerts, production headers). Give the exact command or dashboard check the user should run.
3. For each FAIL, write the **concrete risk** in one sentence (what an attacker or a bad day does with it) and the **minimal fix**.

Keep the checklist's priority tags: 🔴 BLOCKER, 🟡 SHOULD, 🟢 NICE.

Group items that share evidence instead of repeating yourself, and keep the N/A list terse. Spend the effort on the BLOCKERs.

### 4. High-signal searches

These catch the most common, most damaging problems. Adapt the patterns to the stack:

| Look for | Why |
|---|---|
| `NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, `EXPO_PUBLIC_`, `PUBLIC_` env vars whose names contain `SECRET`, `PRIVATE`, `SERVICE_ROLE`, `sk_`, or an API key for a paid service | Secrets shipped to every browser |
| `git ls-files \| grep -E '(^\|/)\.env'` (anything except `.env.example`), plus regexes for `sk_live_`, `sk-`, `AKIA`, `-----BEGIN`, `ghp_`, `xox[bp]-` | Committed secrets. Recommend `gitleaks detect --log-opts="--all"` for full history |
| Route handlers that load a record by an id from the URL or body, without the current user/org in the query | IDOR: user A reads user B's data |
| Role checks like `role === 'admin'` scattered in handlers, roles read from the request body/JWT claims the client controls, routes with no auth middleware | Broken RBAC, privilege escalation |
| `req.body` / `request.json()` passed straight into `create(` / `update(` | Mass assignment (`role: "admin"`, `price: 0`) |
| String-built SQL: template literals or `+` inside `query(`, `$queryRawUnsafe`, `.raw(`, `cursor.execute(f"` | SQL injection |
| `dangerouslySetInnerHTML`, `v-html`, `innerHTML =`, `\|safe`, `{!! !!}` | XSS |
| CORS config: `origin: '*'` or a reflected origin together with `credentials: true` | Cross-origin data theft |
| Password hashing: `md5`, `sha1`, `sha256`, `createHash(` near "password"; plaintext compare | Weak password storage |
| Login/signup/reset/OTP routes with no rate limiter | Brute force, signup spam |
| Paid access granted on a success/return page or from a client call, instead of a verified webhook; webhook handlers without `constructEvent` / signature check; no idempotency | Free premium, double grants |
| Prices, totals, or plans read from the client | Price tampering |
| Supabase/Firebase: tables in migrations without `enable row level security` and policies; `service_role` key used client-side; `firestore.rules` with `allow read, write: if true` | Whole DB readable and writable with the public key |
| `?next=` / `returnTo` / `redirect` params passed to a redirect without validation | Open redirect, used for phishing |
| `Cache-Control` on authenticated responses; CDN/ISR caching of personalized pages | User A's page served to user B |
| LLM calls: key location, per-user quota, `max_tokens`, output passed to `eval`/SQL/shell/HTML | Denial of wallet, prompt-injection fallout |
| Health endpoint (`/health`, `/healthz`, `/readyz`): does liveness hit the DB? | Restart storms during a DB blip |
| `DEBUG = True`, verbose error handlers returning stack traces, `X-Powered-By` | Information leakage |
| Docker: `ports: "5432:5432"` / `"6379:6379"` on services, `USER` missing (runs as root), `--privileged` | Exposed DB, container escape surface |
| File upload handlers: size limit, type check by magic bytes, random storage name, private bucket | Malware hosting, overwrite, XSS via SVG |
| Tests: is there any test for authorization (user A vs user B)? | Silent regressions |

Run `npm audit --audit-level=high` / `pip-audit` / equivalent if the tooling is available and it doesn't need network access the environment lacks. If you can't run it, mark the item CANNOT VERIFY with the command.

### 5. Ground rules

- **Read-only.** Don't modify code, install packages, or change config during the audit. The user decides what to fix, and an audit that rewrites code mid-way can't be trusted as a baseline.
- **Never touch production.** Don't run commands against production URLs, databases, or provider APIs unless the user explicitly asks. Put such checks under CANNOT VERIFY with the command for the user to run.
- **Mask secrets.** If you find a leaked secret, show only its location and a masked prefix (`sk_live_51Ab…`). Never paste the full value into the report or chat. Tell the user to **rotate** it, since removing it from the code doesn't help once it's been committed or deployed.
- **Be calibrated.** Don't inflate severity to look thorough, and don't soften a real BLOCKER. If a finding depends on how the app is deployed, say what it depends on.

## Report

Write the report to `GO_LIVE_AUDIT.md` in the project root (unless the user asked for chat only), then give a short summary in chat: the verdict, the BLOCKER count, and the top 3 fixes.

Use this structure:

```markdown
# Go-Live Audit: <project name>

**Date:** <YYYY-MM-DD> · **Commit:** <short sha, if git> · **Scope:** <all phases | which ones>

## Verdict
**NOT READY** | **READY WITH CAVEATS** | **READY**: one or two sentences why.
(Any 🔴 FAIL or PARTIAL means NOT READY.)

| | PASS | FAIL | PARTIAL | N/A | CANNOT VERIFY |
|---|---|---|---|---|---|
| 🔴 Blocker | | | | | |
| 🟡 Should | | | | | |
| 🟢 Nice | | | | | |

## Stack recon
<5–10 lines: framework, DB access pattern, auth, payments, LLM, hosting>

## 🔴 Blockers to fix before launch
| # | Item (phase) | Status | Evidence | Risk | Minimal fix |
|---|---|---|---|---|---|

## 🟡 Fix in week one
<same table>

## 🟢 Nice to have
<short list>

## Needs manual verification (outside the repo)
| Item | How to check |
|---|---|
<commands from the Verification Cheat Sheet, adapted to this project's domains and stack>

## Passed
<grouped by phase, one line each with file:line>

## Not applicable
<one line each with the reason>

## Suggested fix order
1. <the BLOCKER with the biggest blast radius first>
2. ...
```

## After the report

Offer to fix the failures **one at a time**, BLOCKERs first, each with a regression test that would have caught it (for example, a test where user A requests user B's invoice and gets 403/404). Fixing everything in one pass tends to introduce new bugs, and the tests stop old ones from coming back.

Suggest adding the project's key rules to its agent rules file (`CLAUDE.md` / `AGENTS.md`), for example "scope every query by `owner_id`" or "never put secrets in `NEXT_PUBLIC_*`", so future sessions don't reintroduce them.
