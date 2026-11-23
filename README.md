<div align="center">

# Web App Go-Live Checklist

**The non-negotiable pre-launch checklist for web apps built with AI agents.**

From the first prompt to launch day: security, infrastructure, reliability, legal and discoverability.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude_Code-plugin-D97757)](#option-1-claude-code-skill-recommended)
[![Checks](https://img.shields.io/badge/checks-252-blue)](skills/web-app-checklist/references/checklist.md)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)
[![GitHub stars](https://img.shields.io/github/stars/agusmakmun/web-app-checklist?style=social)](https://github.com/agusmakmun/web-app-checklist)

[**Read the checklist**](skills/web-app-checklist/references/checklist.md) · [**Quick start**](#quick-start) · [**What's inside**](#whats-inside) · [**Contributing**](#contributing)

</div>

---

## Why

AI coding agents produce apps that *look* finished. The demo works, and that hides the problems that actually hurt you after launch:

- 🔑 **Secret keys in the browser bundle**, because the agent added `NEXT_PUBLIC_` to make an error go away.
- 🗄️ **An open database**: Row Level Security never enabled, or Postgres published on a public port.
- 🚪 **Missing ownership checks**, so user A can read user B's invoice by changing an ID.
- 💸 **Paid access granted from the success page** instead of a verified payment webhook.
- 📈 **A surprise bill** from a leaked key, a bot, or an unmetered LLM endpoint.

This checklist exists so those problems are found **before** real users and real data arrive.

## Highlights

- **Ordered the way you work.** Plan → build → deploy → pre-launch → launch day → first 30 days. Domain setup comes after the code works, not before.
- **Prioritized.** 81 🔴 blockers to fix before launch, plus 🟡 week-one items and 🟢 nice-to-haves.
- **Verifiable.** Items come with a way to check them, and there's a [command cheat sheet](skills/web-app-checklist/references/checklist.md#verification-cheat-sheet) for things that live outside the code.
- **Architecture-aware.** Different rules for browser-to-database setups (Supabase, Firebase) and backend-only access (self-hosted or managed Postgres).
- **AI-native.** It covers LLM-specific risks, safe ways of working with coding agents, and ships as a Claude Code skill that audits your codebase with `file:line` evidence.
- **Sourced.** [References](skills/web-app-checklist/references/checklist.md#references) to OWASP, NIST, and vendor documentation for each area.

---

## Quick Start

### Option 1: Claude Code skill (recommended)

This repo is a Claude Code plugin. Once it's installed, Claude audits your codebase in four steps:

1. Recons your stack.
2. Checks every applicable item against your code.
3. Gives each item a status (PASS / FAIL / PARTIAL / N/A / CANNOT VERIFY), backed by `file:line` evidence.
4. Writes a `GO_LIVE_AUDIT.md` report with a verdict and a fix order.

```text
/plugin marketplace add agusmakmun/web-app-checklist
/plugin install web-app-checklist@web-app-checklist
```

Then ask from your project:

```text
> is this app ready to go live?
> run a go-live audit, auth and payments only
```

> [!NOTE]
> The audit is **read-only**. It never modifies your code, never touches production, and masks any secrets it finds.

<details>
<summary><b>Example report</b> (excerpt from an audit of a sample Next.js app)</summary>

```markdown
## Verdict
**NOT READY.** A live Stripe secret key is committed and set up to ship to every browser.
An unauthenticated endpoint returns the database connection string, password included.

|            | PASS | FAIL | PARTIAL | N/A | CANNOT VERIFY |
|------------|------|------|---------|-----|---------------|
| 🔴 Blocker |    2 |   36 |       5 |   8 |            30 |
| 🟡 Should  |    2 |   37 |       5 |  28 |            39 |

## 🔴 Blockers to fix before launch
| #   | Item                | Status | Evidence                                                   | Minimal fix                              |
|-----|---------------------|--------|------------------------------------------------------------|------------------------------------------|
| 3.1 | No secrets in frontend | FAIL | `.env.local:2` `NEXT_PUBLIC_STRIPE_SECRET_KEY=sk_live_51Ab…` | Rotate the key; drop the `NEXT_PUBLIC_` prefix |
| 4.2 | Ownership check (IDOR) | FAIL | `app/api/invoices/[id]/route.ts:7` no owner filter       | `WHERE id = $1 AND owner_id = $2`        |
| 5.3 | SQL injection       | FAIL   | `app/api/search/route.ts:4` builds `ILIKE '%${q}%'`        | Parameterized query                      |
| 6.2 | Paid access via webhook | FAIL | `app/checkout/success/page.tsx:5` upgrades the plan on page load | Grant Pro only from a verified Stripe webhook |
```

</details>

### Option 2: Any coding agent

Download the checklist into your project:

```bash
curl -o LAUNCH_CHECKLIST.md https://raw.githubusercontent.com/agusmakmun/web-app-checklist/main/skills/web-app-checklist/references/checklist.md
```

Then paste this prompt into Claude Code, Cursor, Copilot, or any other coding agent:

```text
You are a senior security engineer doing a pre-launch audit of this codebase.
Go through LAUNCH_CHECKLIST.md phase by phase. For EVERY item:
  1. Search the code/config for evidence (cite file:line).
  2. Mark it PASS / FAIL / NOT APPLICABLE / CANNOT VERIFY (needs infra access).
  3. For FAIL, explain the concrete risk and propose the minimal fix.
Do not mark anything PASS without evidence. Do not modify code yet.
Output a table sorted by priority: BLOCKER failures first.
```

> [!TIP]
> Fix the failures **one at a time**, each with a regression test. A single "fix everything" prompt tends to introduce new bugs.

### Option 3: Manually

Work through the [checklist](skills/web-app-checklist/references/checklist.md) phase by phase. Tick an item only once you've **verified** it with a command, a test, or a screenshot. *"The AI said it's done"* doesn't count.

---

## What's Inside

| Part | # | Phase | Items | 🔴 Blockers |
|------|---|-------|------:|------------:|
| **0 · Plan** | 1 | [Plan & Ownership](skills/web-app-checklist/references/checklist.md#phase-1-plan--ownership) | 7 | 3 |
|  | 2 | [Working Safely With AI Coding Agents](skills/web-app-checklist/references/checklist.md#phase-2-working-safely-with-ai-coding-agents) | 7 | 3 |
| **A · Build** | 3 | [Secrets & Configuration](skills/web-app-checklist/references/checklist.md#phase-3-secrets--configuration) | 11 | 6 |
|  | 4 | [Authentication & Authorization](skills/web-app-checklist/references/checklist.md#phase-4-authentication--authorization) | 16 | 8 |
|  | 5 | [Input, Output & File Uploads](skills/web-app-checklist/references/checklist.md#phase-5-input-output--file-uploads) | 13 | 7 |
|  | 6 | [Business Logic & Payments](skills/web-app-checklist/references/checklist.md#phase-6-business-logic--payments) | 10 | 3 |
|  | 7 | [Database](skills/web-app-checklist/references/checklist.md#phase-7-database) | 14 | 7 |
|  | 8 | [AI / LLM-Specific Risks](skills/web-app-checklist/references/checklist.md#phase-8-ai--llm-specific-risks) | 8 | 4 |
|  | 9 | [Dependencies & Supply Chain](skills/web-app-checklist/references/checklist.md#phase-9-dependencies--supply-chain) | 7 | 2 |
|  | 10 | [Code Quality & Testing](skills/web-app-checklist/references/checklist.md#phase-10-code-quality--testing) | 11 | 3 |
|  | 11 | [Product Essentials](skills/web-app-checklist/references/checklist.md#phase-11-product-essentials) | 13 | 1 |
|  | 12 | [UX Polish](skills/web-app-checklist/references/checklist.md#phase-12-ux-polish) | 7 | 1 |
| **B · Deploy** | 13 | [Domain & DNS](skills/web-app-checklist/references/checklist.md#phase-13-domain--dns) | 6 | 2 |
|  | 14 | [Cloud Accounts & Server Hardening](skills/web-app-checklist/references/checklist.md#phase-14-cloud-accounts--server-hardening) | 8 | 4 |
|  | 15 | [Hosting & Serving](skills/web-app-checklist/references/checklist.md#phase-15-hosting--serving) | 12 | 5 |
|  | 16 | [Transport, Headers & API Protection](skills/web-app-checklist/references/checklist.md#phase-16-transport-headers--api-protection) | 10 | 4 |
|  | 17 | [Email Deliverability](skills/web-app-checklist/references/checklist.md#phase-17-email-deliverability) | 8 | 5 |
|  | 18 | [Performance & Scale](skills/web-app-checklist/references/checklist.md#phase-18-performance--scale) | 10 | – |
|  | 19 | [Observability & Monitoring](skills/web-app-checklist/references/checklist.md#phase-19-observability--monitoring) | 9 | 3 |
| **C · Pre-launch** | 20 | [Release Safety](skills/web-app-checklist/references/checklist.md#phase-20-release-safety) | 6 | 2 |
|  | 21 | [Support, Operations & Abuse](skills/web-app-checklist/references/checklist.md#phase-21-support-operations--abuse) | 8 | 1 |
|  | 22 | [Legal & Compliance](skills/web-app-checklist/references/checklist.md#phase-22-legal--compliance) | 18 | 7 |
|  | 23 | [Discoverability (SEO / AEO / GEO)](skills/web-app-checklist/references/checklist.md#phase-23-discoverability-seo--aeo--geo) | 8 | – |
| **Launch** | | [Launch Day Runbook](skills/web-app-checklist/references/checklist.md#launch-day-runbook) · [First 72 Hours & 30 Days](skills/web-app-checklist/references/checklist.md#after-launch-first-72-hours--first-30-days) | 25 | |
| | | **Total** | **252** | **81** |

Also included: a [Verification Cheat Sheet](skills/web-app-checklist/references/checklist.md#verification-cheat-sheet) of ready-to-run `curl`, `dig`, `gitleaks` and `nc` checks, and a curated list of [References](skills/web-app-checklist/references/checklist.md#references).

### Priority levels

| Tag | Meaning |
|-----|---------|
| 🔴 **BLOCKER** | Do not launch without this. It can leak data, lose data, or get you sued. |
| 🟡 **SHOULD** | Launch is possible without it, but fix it in week one. |
| 🟢 **NICE** | Improves quality, growth, or developer experience. |

---

## Scope

**Covered:** web apps, SaaS products, APIs and mobile-app backends, whether they're on a managed platform (Vercel, Render, Fly), serverless, containers, or a self-managed VPS.

**Not covered:** regulated and enterprise requirements such as HIPAA, PCI DSS beyond using a payment processor, SOC 2 / ISO 27001, SAML SSO, SCIM, and data residency. If those apply to you, use this as a baseline, not the finish line.

> [!IMPORTANT]
> This checklist is not legal advice, and it doesn't replace a professional security review. Get a pentest if you handle sensitive data or payments.

---

## Repository Layout

```text
.
├── .claude-plugin/
│   ├── plugin.json               # Claude Code plugin manifest
│   └── marketplace.json          # marketplace entry for /plugin install
└── skills/web-app-checklist/
    ├── SKILL.md                  # the audit skill
    └── references/
        └── checklist.md          # the checklist (single source of truth)
```

## Contributing

Contributions are welcome, especially items that come from real incidents.

1. Edit [`checklist.md`](skills/web-app-checklist/references/checklist.md) in the right phase.
2. Give each new item a **priority tag**, a **one-line why**, and, where possible, a way to **verify** it.
3. If the item has a high-signal code pattern, add it to the search table in [`SKILL.md`](skills/web-app-checklist/SKILL.md) too.
4. Open a PR describing the failure it prevents.

## License

[MIT](LICENSE) © Agus Makmun
