# Web App Go-Live Checklist

> **Don't trust AI blindly.** AI agents write code fast. They also ship API keys in the frontend, skip auth checks, and open CORS to the world, and the result still looks finished.
> This is the checklist to go through **before** real users and real data touch your app.

It follows the order you actually work in: **plan** before the first prompt, **build** the app safely in the codebase, **deploy** it onto real infrastructure, get it **ready to launch**, then launch and watch it.

---

## Table of Contents

- [How to Use This Checklist](#how-to-use-this-checklist)
- **[Part 0: Before You Code](#part-0-before-you-code)**
  - [Phase 1: Plan & Ownership](#phase-1-plan--ownership)
  - [Phase 2: Working Safely With AI Coding Agents](#phase-2-working-safely-with-ai-coding-agents)
- **[Part A: Build (in the codebase)](#part-a-build-in-the-codebase)**
  - [Phase 3: Secrets & Configuration](#phase-3-secrets--configuration)
  - [Phase 4: Authentication & Authorization](#phase-4-authentication--authorization)
  - [Phase 5: Input, Output & File Uploads](#phase-5-input-output--file-uploads)
  - [Phase 6: Business Logic & Payments](#phase-6-business-logic--payments)
  - [Phase 7: Database](#phase-7-database)
  - [Phase 8: AI / LLM-Specific Risks](#phase-8-ai--llm-specific-risks)
  - [Phase 9: Dependencies & Supply Chain](#phase-9-dependencies--supply-chain)
  - [Phase 10: Code Quality & Testing](#phase-10-code-quality--testing)
  - [Phase 11: Product Essentials](#phase-11-product-essentials)
  - [Phase 12: UX Polish](#phase-12-ux-polish)
- **[Part B: Deploy & Infrastructure](#part-b-deploy--infrastructure)**
  - [Phase 13: Domain & DNS](#phase-13-domain--dns)
  - [Phase 14: Cloud Accounts & Server Hardening](#phase-14-cloud-accounts--server-hardening)
  - [Phase 15: Hosting & Serving](#phase-15-hosting--serving)
  - [Phase 16: Transport, Headers & API Protection](#phase-16-transport-headers--api-protection)
  - [Phase 17: Email Deliverability](#phase-17-email-deliverability)
  - [Phase 18: Performance & Scale](#phase-18-performance--scale)
  - [Phase 19: Observability & Monitoring](#phase-19-observability--monitoring)
- **[Part C: Pre-Launch](#part-c-pre-launch)**
  - [Phase 20: Release Safety](#phase-20-release-safety)
  - [Phase 21: Support, Operations & Abuse](#phase-21-support-operations--abuse)
  - [Phase 22: Legal & Compliance](#phase-22-legal--compliance)
  - [Phase 23: Discoverability (SEO / AEO / GEO)](#phase-23-discoverability-seo--aeo--geo)
- [Launch Day Runbook](#launch-day-runbook)
- [After Launch: First 72 Hours & First 30 Days](#after-launch-first-72-hours--first-30-days)
- [Verification Cheat Sheet](#verification-cheat-sheet)
- [References](#references)

---

## How to Use This Checklist

Work through the phases in order. Every item has a priority tag:

| Tag | Meaning |
|-----|---------|
| 🔴 **BLOCKER** | Do not launch without this. It can leak data, lose data, or get you sued. |
| 🟡 **SHOULD** | Launch is possible without it, but fix it in week one. |
| 🟢 **NICE** | Improves quality, growth, or developer sanity. |

**Rule of thumb:** an item is only checked when you have **verified** it with a command, a test, or a screenshot. "The AI said it's done" doesn't count.

Copy this file into your project as `LAUNCH_CHECKLIST.md` and tick boxes in PRs so the history shows who verified what. To have an agent audit your code against it, see the [README](../../../README.md#quick-start).

---

# Part 0: Before You Code

Takes about an hour, and decides how much rework you'll face later.

---

## Phase 1: Plan & Ownership

- [ ] 🔴 **List the data you'll store** (emails, names, payment info, health, location, children's data) and how sensitive each is. **Data you never collect can't leak.** Drop anything you don't need.
- [ ] 🔴 **Write a permission matrix before coding**, as a table of roles (e.g. owner, admin, member, viewer, guest) × actions (read, create, update, delete, invite, billing, export). Decide whether roles are global or per team/org. This becomes the spec for RBAC in Authentication & Authorization.
- [ ] 🟡 **Do a lightweight threat model.** Spend 30 minutes on who would attack you, what they'd want, and your top 5 abuse cases (account takeover, scraping, spam signups, free-tier abuse, running up your bill).
- [ ] 🔴 **The business owns every account:** domain, cloud, DB, email provider, payments, GitHub org, app stores, analytics. Register them with a company email, not a freelancer's or ex-cofounder's personal one. Keep them in a shared password manager, with 2FA recovery codes stored safely.
- [ ] 🟡 **Use proven building blocks** for auth, database, and payments (managed auth, managed DB, Stripe/Paddle). Don't let an AI hand-roll auth, crypto, or payment logic.
- [ ] 🟡 **Read your plan limits and terms** before you build on them. For example, Vercel Hobby is for non-commercial use only, Supabase free projects pause after a week of inactivity, built-in auth email services are heavily rate-limited, and serverless functions have execution timeouts.
- [ ] 🟢 **Estimate cost per user** (hosting, DB, LLM tokens, email, SMS) so growth doesn't bankrupt you.

---

## Phase 2: Working Safely With AI Coding Agents

- [ ] 🔴 **The agent never has production credentials** or access to the production database. Development uses dev keys and a dev DB. Agents have deleted production databases before.
- [ ] 🔴 **Use git from day one, and commit before every agent session** so you can always undo.
- [ ] 🔴 **Review every diff before merging**, especially anything touching auth, payments, migrations, or deletes. Keep changes small and on branches.
- [ ] 🟡 **Put your security rules in the agent's rules file** (`CLAUDE.md`, `AGENTS.md`, `.cursorrules`), e.g. "always scope queries by `owner_id`", "never put secrets in `NEXT_PUBLIC_*`", "validate every input with Zod". The agent then applies them in every session.
- [ ] 🟡 **Ask for tests together with the feature**, not afterwards.
- [ ] 🟡 **Vet the MCP servers, plugins, and extensions you install**, because they run with your permissions. Don't auto-approve destructive commands.
- [ ] 🟢 Don't paste customer data or real secrets into prompts.

---

# Part A: Build (in the codebase)

Start here, while you're still coding. These are cheap to fix now and expensive after launch.

---

## Phase 3: Secrets & Configuration

- [ ] 🔴 **No secret API keys in the frontend.** Anything in browser JS is public. Watch for framework prefixes that **ship to the client**: `NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*`, `EXPO_PUBLIC_*`, `PUBLIC_*`. Only publishable keys belong there (e.g. Stripe `pk_`, Supabase anon key *with RLS enabled*). Secret keys belong on the server only.
- [ ] 🔴 **`.env` is never committed or publicly served.** `.env*` must be in `.gitignore` (keep a committed `.env.example` with dummy values). Also request `https://app.yourdomain.com/.env` and confirm it returns 404.
- [ ] 🔴 **No hardcoded secrets** in code, configs, Dockerfiles, CI files, or mobile app bundles.
- [ ] 🔴 **Scan the full git history for secrets**, not just the current files:
  ```bash
  gitleaks detect --source . --log-opts="--all"
  # or
  trufflehog git file://. --only-verified
  ```
  **If a secret was ever committed, rotate it.** Deleting the commit isn't enough because forks, clones, and caches keep a copy.
- [ ] 🔴 **Debug mode is OFF in production** (`DEBUG=False`, `NODE_ENV=production`, `APP_DEBUG=false`). Source maps are not publicly served, or are uploaded privately to your error tracker.
- [ ] 🔴 **Errors don't leak internals.** Users see a generic message plus a request ID. Stack traces, SQL, file paths, and framework versions go to logs only.
- [ ] 🟡 **Separate environments** (dev / staging / prod) with **separate keys and databases**. Never test against the production DB.
- [ ] 🟡 **Validate config at startup.** The app refuses to boot if a required env var is missing or malformed (check it with a schema), instead of failing later on the first request that needs it.
- [ ] 🟡 Secrets live in a secret manager or the platform's env settings (Vercel, Fly, AWS Secrets Manager, Doppler, 1Password), with **least-privilege, scoped keys**.
- [ ] 🟢 Write down a key-rotation procedure and test it once.
- [ ] 🟢 Remove the `X-Powered-By` and server version headers.

---

## Phase 4: Authentication & Authorization

- [ ] 🔴 **All auth is enforced server-side.** Hiding a button in the UI is not access control. Every API route checks the session itself.
- [ ] 🔴 **Check ownership on every request (prevent IDOR).** Log in as user A, then request user B's resource (`/api/users/2`, `/api/invoices/123`). It must return 403 or 404. Scope every query by the authenticated user:
  ```sql
  SELECT * FROM invoices WHERE id = $1 AND owner_id = $current_user_id;
  ```
- [ ] 🔴 **Role-based access control (RBAC) is implemented, not implied:**
  - **Roles and permissions are defined in code from the permission matrix** (see Plan & Ownership). Check permissions, not role names: `can(user, "invoice:delete", invoice)` instead of `if (user.role === "admin")` scattered across files.
  - **Deny by default.** A route or action with no explicit permission rule is forbidden. New endpoints don't become public by accident.
  - **Enforce in one place:** a single middleware or policy layer (CASL, Casbin, Pundit, Django permissions, Supabase RLS policies) that every route goes through. Don't copy checks into handlers, where they drift apart.
  - **Check on the server for every request**, including API routes, server actions, GraphQL resolvers, background jobs, file downloads, and websockets. Hiding a button in the UI doesn't count.
  - **Roles are scoped to a tenant.** Being an admin of org A grants nothing in org B.
  - **Prevent privilege escalation.** Users can't change their own role, and a role can only grant roles at or below its own level. The last owner can't be removed or demoted.
  - **Role changes take effect immediately.** Re-check permissions from the DB, or keep token lifetimes short. A demoted user's old JWT must not still say `admin` for a week.
  - **Write a test for each cell of the matrix** (every role × every sensitive action, expecting allow or deny), so a refactor can't quietly open access.
- [ ] 🔴 **Admin accounts are protected.** The role is read from the server and DB, never from a request body, query param, JWT field the client can edit, or localStorage. Admin routes have their own middleware. Admins have **MFA** enabled, and there are as few of them as possible (least privilege).
- [ ] 🔴 **Passwords are hashed with Argon2id** (preferred) **or bcrypt** (cost ≥ 10). Never MD5, SHA-1, SHA-256, or plaintext. Better still, use a proven auth provider (Auth.js, Clerk, Supabase Auth, Auth0, Lucia-style patterns) instead of rolling your own.
- [ ] 🔴 **Sessions are secure.** Cookies use `HttpOnly; Secure; SameSite=Lax` (or `Strict`). Generate a new session ID after login and on privilege change. Logout invalidates the session **server-side**. Sessions have idle and absolute timeouts.
- [ ] 🔴 **Password reset is secure:**
  - The token is cryptographically random, **single-use**, **short-lived** (≤ 1 hour), and stored **hashed**.
  - The response is identical whether or not the email exists (no user enumeration).
  - After a reset, all existing sessions are invalidated.
  - The reset link uses your configured base URL, **never the `Host` header** (prevents host-header poisoning).
- [ ] 🔴 **Brute-force protection** on login, signup, reset, and OTP: rate limits per IP **and** per account, plus a CAPTCHA/Turnstile after repeated failures.
- [ ] 🟡 **Prevent account enumeration and duplicate accounts.** Signup, login, and reset give the same response whether or not the email exists. Store emails normalized (trimmed, lowercase) behind a case-insensitive unique constraint, so `Bob@x.com` and `bob@x.com` can't become two accounts.
- [ ] 🟡 Enforce a minimum password length of 8+ (ideally 12+), and check passwords against breached lists (HaveIBeenPwned k-anonymity API). Don't impose annoying composition rules.
- [ ] 🟡 If you use JWTs: verify signature and `alg` (reject `none`), use short expiry, a refresh token rotation strategy, and **don't put them in localStorage** if you can avoid it.
- [ ] 🟡 OAuth: validate `state`, use PKCE, and allowlist exact redirect URIs.
- [ ] 🟡 Re-authenticate before sensitive actions (changing email or password, deleting the account, viewing API keys).
- [ ] 🟢 Offer MFA (TOTP or passkeys) to all users.
- [ ] 🟡 **Verify the email address** before enabling sensitive features. An email change must be confirmed from the **new** address, with a notification sent to the **old** one.
- [ ] 🟢 Users can see their active sessions and log out of all devices.

---

## Phase 5: Input, Output & File Uploads

- [ ] 🔴 **Validate every input on the server** with a schema (Zod, Valibot, Pydantic, Joi, class-validator). Check type, length, format, and range. Prefer **allowlists**. Client-side validation is for UX only.
- [ ] 🔴 **Prevent mass assignment.** Never pass `req.body` straight into `update()`. Pick the allowed fields explicitly so users can't set `role: "admin"` or `price: 0`.
- [ ] 🔴 **Prevent SQL injection.** Use parameterized queries or an ORM everywhere. Grep for string-concatenated SQL and raw query helpers (`$queryRawUnsafe`, `.raw(`, f-strings in SQL).
- [ ] 🔴 **Prevent XSS.** Rely on framework auto-escaping. Audit every `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, `{!! !!}`, and `|safe`. Sanitize any user HTML/Markdown with **DOMPurify**. Add a **Content-Security-Policy**.
- [ ] 🔴 **Sanitize output by context.** HTML, attributes, URLs (block `javascript:` links), JSON, shell, and logs each need different escaping.
- [ ] 🟡 **CSRF protection** on state-changing requests when using cookie auth (`SameSite` cookies + CSRF token or Origin check).
- [ ] 🟡 **Prevent SSRF.** If the server fetches user-supplied URLs (webhooks, link previews, image imports), block private IP ranges, `localhost`, and cloud metadata (`169.254.169.254`).
- [ ] 🟡 **Prevent path traversal.** Never build file paths from user input without normalizing and allowlisting.
- [ ] 🟢 **Prevent CSV/formula injection** in exports. Prefix cells starting with `=`, `+`, `-`, or `@` with `'`, so user data doesn't run as a formula when someone opens the file in Excel.
- [ ] 🔴 **Restrict file uploads:**
  - Enforce a max file size at the proxy **and** the app.
  - Allowlist extensions **and** check magic bytes. Don't trust `Content-Type`.
  - Rename files to random IDs. Never use the user's filename as the storage path.
  - Store files **outside the web root** / in object storage, served from a separate domain or with `Content-Disposition: attachment`.
  - Private buckets by default, with short-lived signed URLs.
  - Strip EXIF metadata from images (it can contain GPS location).
- [ ] 🟡 **Scan uploaded files** for malware (ClamAV, VirusTotal API, or your cloud provider's scanner) before other users can download them.
- [ ] 🟡 **Block SVG uploads** or sanitize them, since SVGs can contain scripts.
- [ ] 🔴 **Prevent open redirects.** `?next=`, `?returnTo=`, and `?redirect=` may only point to relative paths or your own domains. Otherwise attackers use your login page for phishing.

---

## Phase 6: Business Logic & Payments

Scanners don't find these bugs, but attackers and bargain hunters do.

- [ ] 🔴 **Never trust prices, totals, discounts, quantities, or plan names from the client.** Compute them on the server from your own data.
- [ ] 🔴 **Grant paid access from verified payment webhooks**, not from the "success" redirect page, because anyone can open `/checkout/success`.
- [ ] 🔴 **Make payments and grants idempotent.** A double click, a retried request, or a webhook delivered twice must not charge twice or grant credits twice. Use idempotency keys and unique constraints.
- [ ] 🟡 **Prevent race conditions.** For example, a coupon redeemed twice in parallel, a balance spent twice, or seats over the limit. Use DB transactions, row locks, or atomic `UPDATE ... WHERE balance >= x`. Test by firing 20 parallel requests.
- [ ] 🟡 **Reject negative, zero, and absurd values** (quantity `-1`, amount `0.0001`, limit `999999`).
- [ ] 🟡 **Isolate tenants (teams/orgs).** Every query is scoped by org. Invites expire, are single-use, and are bound to the invited email. Removing a member revokes access immediately, including active sessions and API keys.
- [ ] 🟡 **Handle the full subscription lifecycle:** trial end, failed payment (dunning), card expiry, upgrade/downgrade proration, cancel, refund. Each one changes access correctly.
- [ ] 🟡 **Prevent signup and free-tier abuse:** bot protection (Cloudflare Turnstile, hCaptcha), trial limits per card or device, and a disposable-email policy.
- [ ] 🟡 **Make background jobs and cron reliable:** authenticated triggers, idempotent, retried with backoff, with a dead-letter queue and an alert on failure.
- [ ] 🟢 Store timestamps in **UTC** and convert to the user's timezone only for display.

---

## Phase 7: Database

- [ ] 🔴 **The database is not publicly reachable.** It sits in a private network or behind an IP allowlist. Test it: `nc -vz your-db-host 5432` from a random machine must fail.
- [ ] 🔴 **Use least-privilege DB credentials.** The app user can't `DROP`, can't access other databases, and isn't a superuser. Use a separate user for migrations.
- [ ] 🔴 **Know who talks to your database, and lock that path down.** Which item applies depends on your architecture:
  - **The browser talks to the DB directly.** This is the case with Supabase, Firebase/Firestore, PostgREST, Hasura, Appwrite, PocketBase, and similar backend-as-a-service tools. **Row Level Security / Security Rules must be enabled on every table or collection** and tested as an anonymous user and as another user. With the public key and no RLS, **anyone can read and write your whole database.** This is the #1 leak in vibe-coded apps.
  - **Only your backend talks to the DB.** This is the case with self-hosted or managed PostgreSQL/MySQL/MongoDB (RDS, Cloud SQL, Neon, PlanetScale, a Docker container) behind your own API. The browser must never get DB credentials or a direct connection, and **authorization lives in your API layer** (see RBAC in Authentication & Authorization). Postgres RLS is optional extra protection for multi-tenant data.
- [ ] 🔴 **If you self-host the database** (on a VPS, or in Docker or Kubernetes):
  - It listens on `localhost` / a private network only (`listen_addresses`, `bind-address`). Never publish the DB port in Docker (`ports: "5432:5432"`) on a public server.
  - Lock down client authentication (`pg_hba.conf`): no `trust` entries, `scram-sha-256` passwords, and only the app's host allowed.
  - Use strong, unique passwords, and disable or rename the default superuser for remote access.
  - Apply minor-version security updates regularly, and plan major-version upgrades.
  - Run automated, off-server backups (`pg_dump` / `pgBackRest` / WAL-G to object storage in another account or region), and test restoring them.
  - Monitor disk space, connections, and replication lag. A full disk is a common way for a self-hosted DB to go down.
- [ ] 🔴 **Automated backups are on**, with point-in-time recovery when available. Store them in a different account or region from production.
- [ ] 🔴 **Test the restore.** A backup you have never restored is only a hope. Restore into a scratch DB, run the app against it, and record how long it took (your real RTO).
- [ ] 🟡 **Write a disaster recovery plan.** Define RPO (how much data you can lose) and RTO (how long you can be down), and write the steps for losing a database, a region, your hosting provider, or a key person's access. Check that backups, DNS, and the runbook are reachable when your main provider is down.
- [ ] 🟡 **Plan the schema** before you have users: primary keys, foreign keys, `NOT NULL`, unique constraints, `created_at`/`updated_at`. Let the DB enforce invariants.
- [ ] 🟡 **Use migrations** (Prisma, Drizzle, Alembic, Rails, Flyway) and keep them in git. Never apply hand-edited schema changes to prod.
- [ ] 🟡 **Fix N+1 queries and add indexes** on foreign keys and on every column used in `WHERE`, `ORDER BY`, or `JOIN`. Run `EXPLAIN ANALYZE` on your slowest queries.
- [ ] 🟡 **Encrypt at rest** (default on most managed DBs) and use **TLS in transit**.
- [ ] 🟡 **Encrypt especially sensitive fields at the application level** (OAuth refresh tokens, third-party API keys users give you, national IDs), so a leaked DB dump alone doesn't expose them.
- [ ] 🔴 **Back up uploaded files too.** Enable object-storage versioning or replication, because a DB backup doesn't include the files it points to.
- [ ] 🟢 Add a connection pooler (PgBouncer, Supavisor, RDS Proxy) if you run on serverless.

---

## Phase 8: AI / LLM-Specific Risks

If your app calls an LLM, these apply as well.

- [ ] 🔴 **LLM API keys are server-side only**, with **spending caps / budget alerts** set in the provider dashboard.
- [ ] 🔴 **Per-user rate limits and quotas on AI endpoints.** One abusive user can run up a four-figure bill overnight.
- [ ] 🔴 **Treat LLM output as untrusted input.** Never `eval` it, run it as SQL or shell, or render it as raw HTML without sanitizing.
- [ ] 🔴 **Defend against prompt injection.** Content from users, web pages, emails, or documents can contain instructions. Give tools/agents least privilege, and require human confirmation for destructive or outbound actions (sending email, payments, deleting data).
- [ ] 🟡 **Don't put secrets or other users' data in prompts.** Assume the system prompt can be extracted.
- [ ] 🟡 **Respect PII/data agreements.** Know whether your provider trains on or retains data, and disclose AI processing in your privacy policy.
- [ ] 🟡 Set `max_tokens`, request timeouts, and graceful fallbacks for when the provider is down.
- [ ] 🟢 Log prompts and responses (with PII redacted) so you can debug and evaluate quality.

---

## Phase 9: Dependencies & Supply Chain

- [ ] 🔴 **No known critical/high vulnerabilities:** `npm audit` / `pnpm audit` / `pip-audit` / `bundle audit` / `govulncheck`.
- [ ] 🔴 **The lockfile is committed**, and CI installs with `npm ci` (not `npm install`).
- [ ] 🟡 **Enable Dependabot or Renovate** plus GitHub secret scanning and push protection.
- [ ] 🟡 **Check for hallucinated packages.** AI agents sometimes invent package names, and attackers register them ("slopsquatting"). Confirm that every new dependency exists, is maintained, and is the one you meant.
- [ ] 🟡 Remove unused dependencies (`npx depcheck`, `knip`).
- [ ] 🟡 Pin GitHub Actions to a commit SHA, and give CI tokens minimal permissions.
- [ ] 🟢 Scan container images (`trivy image`, `docker scout`) and use slim base images.

---

## Phase 10: Code Quality & Testing

- [ ] 🔴 **You can explain and debug your own code.** If you can't trace a request from the UI to the DB and back, you can't fix it at 3 AM. Read every file the AI wrote that touches auth, payments, or data.
- [ ] 🔴 **Tests exist for the critical paths:** signup, login, password reset, payment, the core feature, and **authorization** (user A can't access user B's data).
- [ ] 🔴 **Handle errors.** Every external call (DB, API, email, payment) has failure handling. No unhandled promise rejections, and no silent `catch {}`.
- [ ] 🟡 **Make outbound calls resilient.** Every call to a third-party API has a **timeout**. Only idempotent calls are retried, with exponential backoff and jitter, and there's a fallback when the provider is down (queue and retry later, degrade the feature, or show a clear message). Then a slow provider can't take down your whole app.
- [ ] 🟡 **Add a regression test with every bug fix.** "Prompt fix = new bugs" happens when nothing stops old bugs from coming back.
- [ ] 🟡 **CI runs on every PR:** lint, type-check, tests, audit, secret scan. Main is protected.
- [ ] 🟡 **Add static security analysis (SAST) in CI** (Semgrep, CodeQL, or SonarQube) to catch injection, unsafe deserialization, and hardcoded secrets in every PR.
- [ ] 🟡 **Run a dynamic scan (DAST) against staging before launch** (e.g. OWASP ZAP baseline scan), which probes the running app the way an attacker would.
- [ ] 🟡 **Give the code structure.** Clear modules, no copy-pasted logic across files, consistent patterns. Duplicated security checks drift apart.
- [ ] 🟡 **Developer documentation:** a `README` with setup steps and an `.env.example`, a short architecture overview (services, data flow, where secrets live), and API docs (e.g. OpenAPI) if anyone else integrates with you. A new developer should be able to run the app locally in under an hour.
- [ ] 🟢 Add one end-to-end smoke test (Playwright/Cypress) that runs against staging before every deploy.

---

## Phase 11: Product Essentials

These are the features users expect from any platform, so their absence reads as unfinished. How much of each you need depends on the product (blog, drive, scheduler, marketplace, etc.).

- [ ] 🔴 **Account settings:** view and edit profile, change email (verified), change password, manage connected social logins, and delete account, all self-serve.
- [ ] 🟡 **Profile and personalization** (if your product has public or shared profiles). Profile fields are validated, avatars go through the upload rules, and users control what is public.
- [ ] 🟡 **Search, filter, and sort** on every list that can grow. Search results respect permissions (they never return other tenants' data), use pagination, are rate-limited, and have a helpful empty state. Use DB full-text search before reaching for a search service.
- [ ] 🟡 **Notifications for important events** by email (and SMS/push where it matters): signup, password/email change, new login, payment receipt or failure, invites, mentions.
- [ ] 🟡 **Notification preferences.** Users choose which non-critical notifications they get and on which channel. Security and billing notices can't be switched off.
- [ ] 🟢 **Real-time messaging or notifications** (if needed). Websockets/SSE authenticate the connection, authorize every channel subscription, and are rate-limited.
- [ ] 🟡 **In-app announcement banner** for planned maintenance and ongoing incidents. Admins can switch it on without a deploy (from a DB flag or feature flag, pushed over websocket or SSE where available).
- [ ] 🟡 **Feedback and "report a problem"** inside the app. Attach the URL, browser, and request ID automatically.
- [ ] 🟢 **Ratings and reviews** (if relevant): one per user per item, moderated, protected against fake-review bursts.
- [ ] 🟢 **Integrations** (if relevant): a public API with docs and versioning, per-user API keys that are scoped, revocable, and stored hashed, and outgoing webhooks signed with a secret.
- [ ] 🟢 **Progressive Web App** (installable, offline fallback page) or a mobile app, if your users need it.
- [ ] 🟡 **If you ship native mobile apps:**
  - **No secrets in the app binary.** Apps can be decompiled, so they only hold publishable keys.
  - **In-app account deletion** is required by both Apple and Google if users can create accounts in the app.
  - Fill in the **App Store privacy labels and the Google Play Data safety form** accurately, matching your privacy policy.
  - Build a **forced-update mechanism** (a minimum supported version check), because old app versions stay installed for years and keep calling your API.
  - Keep your **API backward-compatible**, or version it, for the same reason.
  - Set up **deep links** (Universal Links / App Links), push notification keys and certificates, and their expiry reminders.
  - **Plan for store review time.** Submit days before launch, not on launch day.
- [ ] 🟢 **Internationalization** (if you serve more than one market): locale-aware dates, numbers, and currency, translatable strings rather than hardcoded text, and right-to-left (RTL) layout support if needed.

---

## Phase 12: UX Polish

- [ ] 🟡 **Custom error pages and handlers** for 404, 500, and 503 (maintenance), with your branding, a way home, a support link, and no stack traces. The app also handles 400, 401, 403, and 429 responses gracefully: an expired session redirects to login and **returns the user to where they were**, 403 explains that they lack access, and 429 says to try again later. Moved or renamed URLs get 301 redirects instead of breaking.
- [ ] 🔴 **Check it on mobile**: a real iPhone and a real Android, not just devtools. Test signup, login, and the core flow, and look for keyboards covering inputs and tap targets ≥ 44 px.
- [ ] 🟡 **Cross-browser check:** Chrome, Safari (desktop and iOS), Firefox, Edge.
- [ ] 🟡 **Accessibility (target WCAG 2.2 AA):** keyboard navigation works, focus is visible, images have `alt`, form fields have labels, color contrast ≥ 4.5:1, and errors are announced. Run axe DevTools or Lighthouse, **and test the core flow with a real screen reader** (VoiceOver on macOS/iOS, NVDA on Windows, TalkBack on Android).
- [ ] 🟡 Design loading, empty, and error states for every screen. Use **skeleton loaders** that match the final layout (not spinners) for content that takes more than ~300 ms, to make waits feel shorter and avoid layout shift.
- [ ] 🟢 Add a favicon, app icons, Open Graph / Twitter card images, and `<title>` and meta descriptions for each page.
- [ ] 🟡 **Site footer / standard pages:** About Us, Contact, FAQ/Help, Terms of Service, Privacy Policy, Cookie settings, and your status page, linked from every page. Add social and community links (X, LinkedIn, YouTube, Discord or forum) if you have them.

---

# Part B: Deploy & Infrastructure

Once the app works on staging, set up where it lives and how it's watched.

---

## Phase 13: Domain & DNS

- [ ] 🔴 **Host the app on a subdomain.** `yourdomain.com` is the marketing site/homepage, `app.yourdomain.com` is the product. You can then change, redeploy, or move either one without touching the other, and set separate cookie scopes and CSPs.
- [ ] 🔴 **Enable 2FA on your registrar and DNS provider.** Anyone who takes over your DNS controls your email, TLS certificates, and login pages.
- [ ] 🟡 **Turn on registrar lock** (transfer lock) and **auto-renew**. Expired domains get sniped.
- [ ] 🟡 **Add CAA records** so that only your chosen CA can issue certificates:
  `yourdomain.com. CAA 0 issue "letsencrypt.org"`
- [ ] 🟢 **Enable DNSSEC** if your registrar and DNS host support it.
- [ ] 🟡 **Remove dangling DNS records** that point to deleted servers, buckets, or Heroku/Vercel apps. They enable subdomain takeover.

---

## Phase 14: Cloud Accounts & Server Hardening

- [ ] 🔴 **MFA on every provider account**: cloud, hosting, DB, GitHub, email, payments, DNS. Don't use the cloud root account day to day; lock it away with hardware-key MFA.
- [ ] 🔴 **Set billing alerts and hard spend limits on every provider.** A leaked key, a bot attack, or an infinite loop can produce a five-figure bill overnight ("denial of wallet").
- [ ] 🔴 **Storage buckets are private** unless deliberately public, and public ones don't allow listing. Open `https://<bucket-url>/` in a private window to check.
- [ ] 🔴 **Admin panels, DB dashboards, and staging are not publicly open.** Put them behind auth plus a VPN, IP allowlist, or Cloudflare Access.
- [ ] 🟡 **Preview deployments are protected** (password or SSO), don't use production secrets, and are `noindex`.
- [ ] 🟡 **Least-privilege IAM.** CI uses short-lived OIDC credentials instead of long-lived cloud keys, and secrets are masked in CI logs.
- [ ] 🟡 **VPS hardening (if you run your own server):** only ports 80/443 open (SSH only from your IP, or via a tunnel), SSH keys only, root login disabled, automatic security updates, fail2ban.
- [ ] 🟡 **Containers run as non-root** with a read-only filesystem where possible, and never with `--privileged`.

---

## Phase 15: Hosting & Serving

How the app actually runs in production. The Cloud Accounts & Server Hardening phase covers locking the server down.

- [ ] 🔴 **Choose a hosting model on purpose:**

  | Model | Examples | Good for | You own |
  |-------|----------|----------|---------|
  | Managed platform (PaaS) | Vercel, Netlify, Render, Fly.io, Railway | Most new apps, small teams | App code and config |
  | Serverless functions | AWS Lambda, Cloudflare Workers, Vercel Functions | Spiky or low traffic, event-driven work | Timeouts, cold starts, connection pooling |
  | Containers (managed) | Cloud Run, ECS/Fargate, Fly Machines | Long-running services, background workers | Image, scaling rules |
  | VPS | Hetzner, DigitalOcean, Linode, EC2 | Cheap fixed cost, full control | **Everything:** OS patches, TLS, proxy, backups, uptime |
  | Kubernetes | EKS, GKE, DOKS | Many services, a dedicated platform team | A lot. Usually overkill at launch |

  If you have no ops experience, start with a managed platform. A VPS is cheaper per month, but you're paying for it with your own time and pager.
- [ ] 🟡 **Pick the region on purpose.** Host close to most of your users, and in the **same region as your database**. A function in the US calling a DB in Singapore adds latency to every query.
- [ ] 🔴 **Make builds reproducible:** a `Dockerfile` or the platform's build config in git, pinned runtime versions (`.nvmrc`, `engines`, `.python-version`, base image tag or digest), and dependencies installed from the lockfile. Build **one artifact** and promote that same artifact from staging to production rather than rebuilding it.
- [ ] 🔴 **Deploy automatically from CI.** A merge to `main` runs tests and then deploys. No manual SSH, FTP, or `git pull` on the server, and no "works on my machine" builds pushed by hand.
- [ ] 🔴 **On a VPS, put a reverse proxy in front of the app.** Use Caddy (automatic HTTPS) or Nginx + Certbot, and **check that certificate auto-renewal works** (`certbot renew --dry-run`). The app listens on `127.0.0.1` / a private network only, never directly on a public port.
- [ ] 🔴 **The app restarts on its own** after a crash or reboot: systemd unit with `Restart=always`, a PM2 startup script, or a Docker `restart: unless-stopped` policy. Test it with `sudo reboot` on staging.
- [ ] 🟡 **Deploy with zero downtime:** rolling or blue-green deploys, or at least two instances. The app shuts down gracefully on `SIGTERM`, meaning it stops accepting traffic, finishes in-flight requests and jobs, then closes DB connections.
- [ ] 🟡 **Wire the platform health checks to your health endpoints** (see Observability & Monitoring). Use the liveness path for restart decisions and the readiness path for routing traffic, so the orchestrator replaces dead instances and the load balancer skips ones that aren't ready.
- [ ] 🟡 **Size the server and set limits.** Set CPU and memory limits per container or process. Set Node `--max-old-space-size` / worker counts to match the available RAM, and add swap on small VPSes so a memory spike slows the server down instead of crashing it.
- [ ] 🟡 **Server housekeeping:** log rotation (`logrotate`, Docker `max-size`), a disk-usage alert at 80%, server clock in **UTC** and synced with NTP, and cleanup of old Docker images and releases.
- [ ] 🟡 **Run background workers and cron as separate processes** from the web server, each with its own restart policy and monitoring.
- [ ] 🟢 **Define infrastructure as code** (Terraform, Pulumi, `fly.toml`, `render.yaml`, Docker Compose) so you can rebuild the environment from git when a server dies.

---

## Phase 16: Transport, Headers & API Protection

- [ ] 🔴 **Force HTTPS everywhere.** Redirect HTTP to HTTPS (301), send HSTS: `Strict-Transport-Security: max-age=63072000; includeSubDomains`. Only add `preload` once you're sure, because it's hard to undo. Aim for an **A on [SSL Labs](https://www.ssllabs.com/ssltest/)**.
- [ ] 🔴 **CORS is not `*`** on authenticated APIs. Allowlist exact origins. Never reflect an arbitrary `Origin` together with `Access-Control-Allow-Credentials: true`.
- [ ] 🔴 **Rate-limit your APIs.** Use stricter limits on auth, signup, email-sending, search, and any **paid third-party call** (LLMs, SMS). Return `429` with `Retry-After`. Without this, one script can run up your bill or take you down.
- [ ] 🟡 **Security headers.** Score A on [securityheaders.com](https://securityheaders.com):
  ```
  Content-Security-Policy: default-src 'self'; ...
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  X-Frame-Options: DENY            (or CSP frame-ancestors 'none')
  ```
- [ ] 🟡 **Request limits:** body size limits, timeouts, and pagination caps (`?limit=1000000` must not work).
- [ ] 🔴 **The CDN or proxy never caches private responses.** Authenticated or personalized responses send `Cache-Control: private, no-store`, and cache keys include anything that changes the response. Otherwise user A's dashboard can be served to user B. Test it: load a logged-in page, then request it with `curl` and no cookies.
- [ ] 🟡 **If you use GraphQL:** disable introspection in production, and enforce query depth, complexity, and batch limits. Field-level authorization covers every resolver, not just the top-level query.
- [ ] 🟡 **Webhook endpoints verify signatures** (Stripe, GitHub, etc.) and are idempotent.
- [ ] 🟢 **DDoS / bot protection** via the CDN/WAF (Cloudflare, AWS WAF, Vercel Firewall).
- [ ] 🟡 **Minimize third-party scripts** (analytics, chat widgets, tag managers). Each one runs with full access to your page. Load them only after consent, list them in your CSP, use Subresource Integrity (`integrity=`) for scripts served from a CDN, and keep them off sensitive pages like login and payment.

---

## Phase 17: Email Deliverability

- [ ] 🔴 **Use separate sending subdomains for separate purposes.**

  | Subdomain | Used for | Reputation risk |
  |-----------|----------|-----------------|
  | `mail.yourdomain.com` | Sign-ups, password resets, receipts, billing (transactional) | Must stay clean |
  | `news.yourdomain.com` | Newsletters, marketing, announcements | Gets spam complaints |

  Subdomains don't prevent spam complaints, but they **contain** them: if marketing mail gets flagged, password resets still arrive.
- [ ] 🔴 **SPF, DKIM, and DMARC configured and passing on every sending subdomain.** Most providers (Resend, Postmark, SES, SendGrid) give you the exact records to add.
  - SPF: exactly **one** `v=spf1` record per name, at most 10 DNS lookups.
  - DKIM: a 2048-bit key, provided by your email service.
  - DMARC: start with `v=DMARC1; p=none; rua=mailto:dmarc@yourdomain.com`, then move to `p=quarantine` and later `p=reject` once the reports are clean.
- [ ] 🔴 **Test password reset emails end to end.** Check that they arrive in Gmail, Outlook, and iCloud, that they land in the inbox and not spam, that the link works, and that the link expires.
- [ ] 🔴 **Test every transactional email:** welcome, verify email, reset, receipt, invite, account deletion.
- [ ] 🟡 Score **10/10 on [mail-tester.com](https://www.mail-tester.com)**.
- [ ] 🔴 **Use a real SMTP provider for auth emails.** The built-in email senders of Supabase, Firebase, and similar services are heavily rate-limited and meant for testing. On launch day, signups would stop receiving verification emails.
- [ ] 🟡 **Handle bounces and complaints.** Stop sending to hard-bounced addresses, and add a one-click unsubscribe (`List-Unsubscribe` header) to marketing mail. Gmail and Yahoo require this for bulk senders.
- [ ] 🟢 Set up a monitored `support@` / `security@` inbox and publish `/.well-known/security.txt`.

---

## Phase 18: Performance & Scale

- [ ] 🟡 **Compress your images.** Serve WebP/AVIF, use responsive `srcset`, lazy-load below the fold, and don't ship 4 MB hero images.
- [ ] 🟡 **Cache static assets** with fingerprinted filenames and `Cache-Control: public, max-age=31536000, immutable`. HTML stays `no-cache`.
- [ ] 🟡 **Put the app behind a CDN** (Cloudflare, CloudFront, Fastly, or your platform's edge).
- [ ] 🟡 **Load-test the signup flow** and the core flow at roughly 10× expected launch traffic. Watch DB connections, email-provider rate limits, and third-party quotas.
  ```bash
  k6 run --vus 100 --duration 2m signup-test.js
  ```
- [ ] 🟡 **Design for horizontal scale.** App servers are stateless (sessions, uploads, and caches live in the DB, Redis, or object storage, not on local disk or in memory), so you can run 2+ instances behind a load balancer.
- [ ] 🟡 **Add a caching layer** (Redis, or framework/HTTP caching) for expensive, frequently read data. Plan how the cache gets invalidated.
- [ ] 🟡 **Core Web Vitals in the green:** LCP < 2.5 s, INP < 200 ms, CLS < 0.1 (Lighthouse / PageSpeed Insights).
- [ ] 🟢 **Scan every page, not just the homepage**, with [Unlighthouse](https://unlighthouse.dev), and add a Lighthouse CI budget so performance regressions fail the build.
- [ ] 🟢 Enable gzip/brotli compression and keep the JS bundle small (code-split heavy routes).
- [ ] 🟢 Move slow work (emails, image processing, AI calls) to background jobs or queues.

---

## Phase 19: Observability & Monitoring

- [ ] 🔴 **Add health check endpoints.** The name is up to you: `/healthz` (the Kubernetes/Google convention, which won't clash with a real page) or `/health` (the default in many frameworks and uptime tools). **Pick one and use it everywhere:** the load balancer, the platform health check, uptime monitoring, and the runbook.
  - **Liveness** (`/healthz` or `/livez`) answers "is the process up?" It returns `200` without touching the database or other dependencies. If it checks the DB, a brief DB blip makes the orchestrator restart **every** instance at once.
  - **Readiness** (`/readyz`) answers "can this instance serve traffic?" It checks the DB and critical dependencies with short timeouts, and returns `503` when they're down so the load balancer routes around it.
  - Both are fast and unauthenticated, reveal no versions, hostnames, or dependency details, and are cheap enough that they can't be abused to load the database.
- [ ] 🔴 **Set up error tracking** (Sentry, Bugsnag, Rollbar) on both frontend and backend, with source maps uploaded privately and alerts sent to a channel you actually read.
- [ ] 🔴 **Turn on uptime monitoring** (UptimeRobot, Better Stack, Checkly) for the homepage, the app, your health endpoint, and the API, checked from several regions, with alerts going to your phone.
- [ ] 🟡 **Use structured logs** (JSON) with request IDs. **Never log** passwords, tokens, full card numbers, or session cookies. Keep logs for long enough to investigate an incident.
- [ ] 🟡 **Write an audit log** for sensitive actions: logins, role changes, data exports, deletions, admin actions.
- [ ] 🟡 **Set up a status page** (Better Stack, Instatus, statuspage.io) on a **different host** from your app, e.g. `status.yourdomain.com`.
- [ ] 🟡 **Add analytics events** for the funnel: visit → signup → activation → payment. Use a privacy-friendly or consent-gated tool (Plausible, PostHog, GA4 with consent).
- [ ] 🟢 Alert on resource limits: DB CPU/storage, queue depth, error rate spikes, LLM/API spend.
- [ ] 🟡 **Monitor TLS certificate and domain expiry.** Certificate auto-renewal fails silently more often than you'd think.

---

# Part C: Pre-Launch

Before real users arrive.

---

## Phase 20: Release Safety

- [ ] 🔴 **Write a rollback plan.** One command or one click returns you to the previous version. **Practice it once.** Make DB migrations backward-compatible (expand → migrate → contract) so a code rollback doesn't break against the new schema.
- [ ] 🟡 **Run a staging environment** that mirrors production, and deploy there first.
- [ ] 🟡 **Use feature flags** for risky features, so you can turn them off without a deploy (LaunchDarkly, PostHog, Unleash, or a simple DB/env flag).
- [ ] 🟡 **Write a runbook** covering how to deploy, roll back, restore the DB, rotate a leaked key, and whom to call.
- [ ] 🟡 **Write an incident / data breach procedure.** Decide who decides, who communicates, and the notification deadlines (GDPR: 72 hours to the regulator).
- [ ] 🔴 **Don't launch on a Friday** (or before a holiday, or when you're about to sleep or travel). Launch when you can watch it for the next 24–48 hours.

---

## Phase 21: Support, Operations & Abuse

- [ ] 🔴 **Users can reach you.** Provide an in-app contact link or support email that someone actually monitors.
- [ ] 🟡 **Build internal admin tools** to look up a user, resend verification, refund, extend a trial, or ban, **without running SQL on production**. Log every admin action, and make impersonation explicit and audited.
- [ ] 🟡 **Publish your OAuth consent screen** ("Sign in with Google") out of *Testing* mode before launch. Testing mode is limited to 100 test users and refresh tokens expire after 7 days. Submit for verification early, because review can take days to weeks. Finish the Sign in with Apple and app store setup early too.
- [ ] 🟡 **If users can post content:** add a report button, a moderation queue, and a takedown process (DMCA agent / notice-and-action), and know your legal reporting obligations for illegal content.
- [ ] 🟡 **Write user documentation:** a help center/FAQ, getting-started tutorials and guides for the core flows, and an onboarding flow for first-time users.
- [ ] 🟢 **Live chat or a ticketing system** (Crisp, Intercom, Help Scout, Zendesk) once email support gets busy, so nothing gets lost.
- [ ] 🟢 **Soft-launch first.** Start with a waitlist or a private beta with a small group, then open publicly once the error rate is quiet.
- [ ] 🟢 Add a changelog or announcement channel so users know what changed.

---

## Phase 22: Legal & Compliance

*This is not legal advice. Talk to a lawyer for your jurisdiction, especially if you handle health, finance, or children's data.*

- [ ] 🔴 **Privacy Policy** saying what you collect, why, who you share it with, and how long you keep it.
- [ ] 🔴 **Terms & Conditions** covering acceptable use, liability limits, termination, and governing law.
- [ ] 🔴 **GDPR compliance** (if you have EU/UK users), plus CCPA/CPRA, LGPD, PDPA, or UU PDP (Indonesia) as applicable.
- [ ] 🔴 **Cookie consent.** Non-essential cookies and trackers are blocked **until** the user consents, and rejecting is as easy as accepting.
- [ ] 🟡 **A legal basis** documented for each type of processing (consent, contract, legitimate interest).
- [ ] 🟡 **Marketing consent** is a separate opt-in checkbox, unchecked by default, never bundled with the terms.
- [ ] 🔴 **Data deletion.** Users can delete their account and data, and the deletion also covers backups and third parties within your stated retention period.
- [ ] 🟡 **Data export.** Users can download their data in a machine-readable format (GDPR right to data portability).
- [ ] 🟡 **A data retention** policy, implemented in code: old logs and data are actually deleted.
- [ ] 🟡 **Data Processing Agreements** signed with every processor (hosting, email, analytics, LLM provider, payments).
- [ ] 🟡 **Third-party disclosure.** Your list of subprocessors is published and accurate.
- [ ] 🔴 **Data security.** The technical measures you claim (encryption, access control) actually exist, which Part A and the Cloud Accounts & HTTPS phases cover.
- [ ] 🟡 **A data breach procedure** (see Release Safety).
- [ ] 🟡 **Intellectual property.** You own or have licensed your name, logo, content, fonts, and images. Check that your name doesn't infringe an existing trademark.
- [ ] 🟡 **Open-source licenses.** Your dependencies' licenses are compatible with your use (watch for GPL/AGPL in SaaS), and attributions are included where required (`npx license-checker`).
- [ ] 🔴 **Payments & subscriptions.** Use a PCI-compliant processor (Stripe, Paddle, Lemon Squeezy) and never touch raw card numbers. Show clear pricing, renewal terms, a cancel flow as easy as signup, a refund policy, and the sales tax/VAT handling required in your markets.
- [ ] 🟢 Age restrictions / COPPA if your app could attract children.
- [ ] 🟡 **Accessibility law.** The European Accessibility Act (in force since June 2025) and the ADA (US) can apply to consumer-facing digital services. The UX Polish phase covers the basics.

---

## Phase 23: Discoverability (SEO / AEO / GEO)

- **SEO** gets your site **listed** in search results.
- **AEO** (Answer Engine Optimization) gets your site used as **the answer** (Google AI Overviews, featured snippets).
- **GEO** (Generative Engine Optimization) gets your site **mentioned** by AI assistants like Claude and ChatGPT.

- [ ] 🟡 **Tell Google to index your site.** Verify the domain in **Google Search Console** and submit `https://yourdomain.com/sitemap.xml`. Do the same in **Bing Webmaster Tools** (Bing also feeds several AI assistants).
- [ ] 🟡 **`robots.txt` and `sitemap.xml`** exist and are correct. Make sure staging is `noindex` and that **production is not accidentally `noindex`**. This often happens when a staging config gets copied over.
- [ ] 🟡 **Unique `<title>` and meta description** for each page, canonical URLs, one `<h1>`.
- [ ] 🟡 **Server-rendered or pre-rendered marketing pages.** Crawlers and AI bots often don't run your JS.
- [ ] 🟢 **Structured data (JSON-LD)** for `Organization`, `Product`/`SoftwareApplication`, `FAQPage`, `Article`. Validate it with Google's Rich Results Test.
- [ ] 🟢 **Answer-first content.** Write clear FAQ and comparison pages that directly answer the questions your customers ask.
- [ ] 🟢 **Get mentioned off-site** (reviews, directories, communities, Product Hunt) and keep your name, description, and URL consistent everywhere. AI assistants learn from these sources.
- [ ] 🟢 Consider an `llms.txt` file summarizing your product for AI crawlers, and decide your policy on AI crawlers in `robots.txt`.

---

## Launch Day Runbook

**T-minus 1 week**
- [ ] Every 🔴 BLOCKER is checked and verified.
- [ ] Final backup restore test passed.
- [ ] Load test passed at 10× expected traffic.
- [ ] Rollback rehearsed on staging.

**T-minus 1 day**
- [ ] Code freeze: only launch-critical fixes go in.
- [ ] Delete test accounts, seed/demo users, and default admin passwords, and remove leftover debug routes and `console.log` calls with data.
- [ ] Run the AI-agent audit prompt again on the final commit.
- [ ] Production env vars double-checked (no test keys, no `DEBUG`, correct base URLs).
- [ ] Stripe/payment provider in **live mode** and a real $1 transaction tested and refunded.
- [ ] Alerts tested: trigger a test error and a test downtime alert, and confirm your phone gets them.

**Launch (Tue–Thu morning, your timezone)**
- [ ] Deploy, then run the smoke test against production.
- [ ] Sign up as a brand-new user on mobile and desktop and receive every email.
- [ ] Watch error tracking, uptime, DB metrics, and spend dashboards.
- [ ] Have someone available to deploy a fix or roll back for the next 24–48 hours.

---

## After Launch: First 72 Hours & First 30 Days

**First 72 hours**
- [ ] Triage every new error in the tracker daily.
- [ ] Check email deliverability and DMARC reports.
- [ ] Watch the signup → activation funnel for drop-offs.
- [ ] Watch costs: hosting, DB, email, LLM, SMS.

**First 30 days**
- [ ] Fix every 🟡 SHOULD item still open.
- [ ] Move DMARC from `p=none` to `p=quarantine`.
- [ ] Review access: remove ex-collaborators, rotate any shared credentials.
- [ ] Get an external security review or pentest if you handle sensitive data or payments.
- [ ] Write a short, blameless postmortem for every incident, and turn each one into a checklist item or a test.
- [ ] Review user feedback, support tickets, and analytics every week, and feed them into the roadmap.
- [ ] Schedule recurring work: dependency updates weekly, restore test quarterly, and this checklist re-run before every major release.

---

## Verification Cheat Sheet

```bash
# --- HTTPS & headers ---
curl -sI http://app.yourdomain.com | head -3          # expect 301 → https
curl -sI https://app.yourdomain.com | grep -iE 'strict-transport|content-security|x-content-type|referrer-policy|x-frame'

# --- Exposed files (all should be 404) ---
for p in .env .git/config .DS_Store backup.sql config.json; do
  echo "$p -> $(curl -s -o /dev/null -w '%{http_code}' https://app.yourdomain.com/$p)"; done

# --- CORS (must NOT echo evil.com) ---
curl -sI -H "Origin: https://evil.com" https://api.yourdomain.com/me | grep -i access-control

# --- Email DNS ---
dig +short TXT mail.yourdomain.com            # SPF
dig +short TXT _dmarc.yourdomain.com          # DMARC
dig +short CAA yourdomain.com                 # CAA

# --- Secrets in git history ---
gitleaks detect --source . --log-opts="--all"

# --- Dependencies ---
npm audit --audit-level=high                  # or pip-audit / govulncheck

# --- Rate limiting (expect 429s) ---
for i in $(seq 1 50); do curl -s -o /dev/null -w '%{http_code} ' \
  -X POST https://api.yourdomain.com/auth/login -d 'email=x@x.com&password=wrong'; done

# --- DB not public (should time out / refuse) ---
nc -vz -w 5 your-db-host 5432

# --- Health check ---
curl -s -o /dev/null -w '%{http_code}\n' https://app.yourdomain.com/healthz   # or /health, whichever you chose
```

Online checks: [SSL Labs](https://www.ssllabs.com/ssltest/) · [securityheaders.com](https://securityheaders.com) · [mail-tester.com](https://www.mail-tester.com) · [MXToolbox](https://mxtoolbox.com) · [PageSpeed Insights](https://pagespeed.web.dev) · [Rich Results Test](https://search.google.com/test/rich-results)

---

## References

### Security standards
- [OWASP Top 10](https://owasp.org/www-project-top-ten/): the most critical web app risks
- [OWASP API Security Top 10](https://owasp.org/API-Security/)
- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/): a full verification standard for when you outgrow this checklist
- [OWASP MASVS](https://mas.owasp.org/): the mobile app security standard
- [CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [NIST SP 800-63B](https://pages.nist.gov/800-63-4/sp800-63b.html): password, MFA, and session guidelines
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security): free hands-on labs for every vulnerability class here

### OWASP Cheat Sheets (by phase)
- **Plan:** [Threat Modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
- **Secrets:** [Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- **Auth:** [Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) · [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) · [Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) · [Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) · [Forgot Password](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html) · [Multifactor Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- **Input:** [Input Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) · [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) · [SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) · [CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) · [SSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) · [Mass Assignment](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html) · [Unvalidated Redirects](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html) · [File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) · [CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- **API & headers:** [HTTP Security Headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html) · [Content Security Policy](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html) · [GraphQL](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- **Infra & ops:** [Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html) · [Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

### Databases & backends
- [PostgreSQL: Row Security Policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [PostgreSQL: Client Authentication (`pg_hba.conf`)](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
- [Supabase: Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Firebase Security Rules](https://firebase.google.com/docs/rules)

### Payments
- [Stripe: Webhooks](https://docs.stripe.com/webhooks), covering signature verification and handling duplicate events
- [Stripe: Idempotent requests](https://docs.stripe.com/api/idempotent_requests)

### Tools
- **Secrets:** [Gitleaks](https://github.com/gitleaks/gitleaks) · [TruffleHog](https://github.com/trufflesecurity/trufflehog) · [GitHub secret scanning](https://docs.github.com/en/code-security/secret-scanning)
- **Dependencies:** [Dependabot](https://docs.github.com/en/code-security/dependabot) · [Renovate](https://docs.renovatebot.com/) · [OSV-Scanner](https://github.com/google/osv-scanner) · [Trivy](https://github.com/aquasecurity/trivy)
- **Code & app scanning:** [Semgrep](https://semgrep.dev/) · [CodeQL](https://codeql.github.com/) · [OWASP ZAP](https://www.zaproxy.org/)
- **Load testing:** [k6](https://k6.io/)
- **Performance:** [PageSpeed Insights](https://pagespeed.web.dev) · [Unlighthouse](https://unlighthouse.dev) · [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci)
- **Accessibility:** [axe DevTools](https://www.deque.com/axe/) · [WAVE](https://wave.webaim.org/)

### HTTPS, headers & domains
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [Mozilla HTTP Observatory](https://developer.mozilla.org/en-US/observatory)
- [SSL Labs Server Test](https://www.ssllabs.com/ssltest/) · [securityheaders.com](https://securityheaders.com)
- [HSTS Preload List](https://hstspreload.org/): read the removal section before submitting
- [Let's Encrypt docs](https://letsencrypt.org/docs/) · [Caddy automatic HTTPS](https://caddyserver.com/docs/automatic-https)
- [security.txt](https://securitytxt.org/)

### Email deliverability
- [Gmail email sender guidelines](https://support.google.com/a/answer/81126)
- [Yahoo sender best practices](https://senders.yahooinc.com/best-practices/)
- [Google Postmaster Tools](https://postmaster.google.com/)
- [DMARC.org](https://dmarc.org/)
- [RFC 8058: One-Click Unsubscribe](https://datatracker.ietf.org/doc/html/rfc8058)
- [mail-tester.com](https://www.mail-tester.com) · [MXToolbox](https://mxtoolbox.com)

### Reliability & operations
- [The Twelve-Factor App](https://12factor.net/): config, stateless processes, disposability
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/): monitoring, incident response, postmortems
- [Kubernetes: Liveness, Readiness & Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [GitHub Actions: Security hardening with OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

### Performance, UX & accessibility
- [web.dev: Core Web Vitals](https://web.dev/articles/vitals)
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) · [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)

### Legal & privacy (not legal advice)
- [GDPR full text](https://gdpr-info.eu/)
- [UK ICO: Guidance for organisations](https://ico.org.uk/for-organisations/)
- [California CCPA](https://oag.ca.gov/privacy/ccpa)
- [European Accessibility Act](https://ec.europa.eu/social/main.jsp?catId=1202)
- [SPDX License List](https://spdx.org/licenses/), for checking dependency licenses

### Mobile apps
- [Apple: Offering account deletion in your app](https://developer.apple.com/support/offering-account-deletion-in-your-app/)
- [Google Play: Data safety section](https://support.google.com/googleplay/android-developer/answer/10787469)

### Discoverability (SEO / AEO / GEO)
- [Google Search Central](https://developers.google.com/search/docs)
- [Bing Webmaster Tools](https://www.bing.com/webmasters)
- [Schema.org](https://schema.org/) · [Rich Results Test](https://search.google.com/test/rich-results)
- [llms.txt proposal](https://llmstxt.org/)
