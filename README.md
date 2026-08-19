# GrowEdge Capital

**Official public documentation** of the GrowEdge Capital investment platform.

| | |
|---|---|
| Product | [growedgecapital.com](https://growedgecapital.com) |
| Investor app | [app.growedgecapital.com](https://app.growedgecapital.com) |
| Document type | Product, architecture, engineering, and security overview |
| Audience | Investors, partners, engineers, and the public |
| Source code | Private. This repository contains documentation only. |
| Last updated | August 2026 |

This file is written to be dropped into a public GitHub repository as `README.md`.

---

## How to read this document

GrowEdge Capital runs a **private production platform**. The source, infrastructure credentials, operational runbooks, and exact engine implementation stay in a private repository.

This public document exists so anyone can understand:

- what the product does
- how the system is structured
- how money, identity, and documents are treated
- how deep the engineering is
- how seriously security and safety are taken

It is **not** a rebuild guide, not an API specification, and not a penetration-test brief.

If a detail would help someone clone the product, attack production, impersonate staff, or reconstruct proprietary financial logic, it is omitted on purpose. See [What this document does not contain](#what-this-document-does-not-contain).

---

## 1. What GrowEdge Capital is

GrowEdge Capital is a **private wealth platform** for Indian retail investors. Investors place capital with the firm under contracted monthly-return terms. The software is the operating system for that business: onboarding, KYC, deposits, monthly returns, withdrawals, referrals, certificates, receipts, staff operations, and finance control.

It is not a brokerage, not a public market trading terminal, and not a self-custodial wallet. Capital movement is **staff-supervised**. The platform records, calculates, documents, and controls. Bank and UPI transfers happen in the real world; the system makes those movements accountable.

### The problem it solves

A fixed-return investment desk without software looks like spreadsheets, WhatsApp, paper receipts, and memory. That breaks as soon as you have:

- more than a handful of investors
- two plan types (payout vs growth)
- monthly return days that differ per client
- top-ups that must not be double-counted
- lock-in rules
- referrals
- KYC documents
- cash-desk receipts that need an office copy and a client copy
- an audit trail of who approved what

GrowEdge Capital replaces that with a single production system used by investors, operations, and finance.

### Who uses it

| Persona | What they do |
|---|---|
| **Investor** | Registers, completes KYC, views portfolio and returns, requests deposits and withdrawals, downloads certificates and receipts, refers others |
| **Operations** | Reviews leads and KYC, verifies payment proofs, manages investor records and documents, handles the human side of onboarding |
| **Finance** | Activates investments, confirms payouts, approves withdrawals, controls rates and settings, issues certificates, owns the money trail |
| **Developer / platform admin** | System health, staff accounts, operational tooling — no substitute for finance authority over money |

Investors never share staff consoles. Staff never log in as investors. Money-moving actions are reserved for finance, after operations has done verification work.

---

## 2. Public surfaces

The company runs two public internet surfaces plus a private API.

```
                         Internet
                            │
          ┌─────────────────┴─────────────────┐
          ▼                                   ▼
  Marketing site                      Investor & staff app
  growedgecapital.com                 app.growedgecapital.com
  (Next.js)                           (React SPA)
          │                                   │
          │  lead / contact                   │  authenticated API
          └─────────────────┬─────────────────┘
                            ▼
                    Application API
                    (FastAPI, private network)
                            │
        ┌─────────┬─────────┼─────────┬─────────┐
        ▼         ▼         ▼         ▼         ▼
     Database   Cache    Object    Email     Background
     (Postgres) (Redis)  storage   delivery  jobs
```

| Surface | Role |
|---|---|
| **Marketing site** | Brand, product story, FAQ, blog, investment framework, refer-and-earn, contact, lead capture |
| **Application** | Investor portal, operations console, finance console |
| **Public verification** | Anyone can check that a certificate or document serial is genuine, without logging in |
| **System status** | A separate “is the product up” page, not an internals dump |
| **API** | Not a public developer platform. It serves the app. There is no public API key program. |
| **Native shell (optional)** | The same web app can be wrapped for store distribution. It talks to the same API. There is no second backend. |

The application is a **role-gated single-page app**. Same product, three working worlds: portal, operations, finance.

---

## 3. Feature catalog

### 3.1 Investor portal

**Identity and access**

- Email-based sign-in with one-time verification
- Optional password for established accounts
- Optional Google / phone verification when that path is enabled for registration
- Password recovery that still requires proving email control
- Session that belongs to the person, not to a shared browser forever
- Profile, nominee, and bank / UPI details held on the investor record

**Onboarding and KYC**

- Multi-step KYC wizard (identity documents, selfie, signature, bank details, nominee)
- Registration fee collection with payment proof (UPI / bank transfer / desk)
- Progress that can be resumed; incomplete sign-ups do not become active investors
- Status the investor can see: what is submitted, what operations still needs, when the account is live

**Portfolio**

- Dashboard: capital, plan type, next return context, recent activity
- Simple (monthly payout) and compound (growth / reinvestment) plans
- Multiple lots over time (first deposit plus later top-ups), presented as one relationship with the firm
- Transaction history: deposits, returns, withdrawals, referral credits
- Monthly returns view

**Money requests**

- Deposit / additional investment request with fee preview
- Withdrawal request after lock-in rules are applied
- Payment proof upload
- Status of every request until finance completes it

**Documents**

- Investment certificate
- Deposit, add-on, payout, withdrawal, and referral receipts
- Monthly statements
- Uploaded KYC copies the investor is allowed to see
- Authenticated download — documents are not dumped on a public CDN

**Referrals**

- Personal referral identity
- Visibility into referred investors and reward status
- Rewards are created by the system and **released by finance**, not self-claimed

**Help**

- In-product help, terms, and support path
- In-app notifications for KYC, deposits, returns, and withdrawals

### 3.2 Operations console

Operations is the **verification layer**. It does not mark capital as paid.

- Lead inbox from the marketing site and other sources
- Registration and KYC review queue
- Document completeness checks (including legacy investors who were funded before full digital KYC)
- Payment-proof review
- Investor directory, notes, callbacks
- Document vault for staff (scoped, authenticated)
- Broadcast / operational messaging (role-limited)
- Operations analytics and reporting
- Read of payout / request queues so ops can prepare the desk, not override finance

### 3.3 Finance console

Finance is the **authority layer**. Nothing that creates or destroys capital completes without finance.

- Firm-wide dashboard: AUM, active investors, pending money events
- New-client activation after KYC is clean
- Deposit confirmation and investment creation
- Additional-deposit (top-up) handling
- Monthly return queue: system proposes, finance confirms
- Withdrawal approval and completion
- Referral reward release
- Certificates: generate, regenerate when a lawful correction is needed
- Dual-copy receipts for cash-desk events
- Transaction manager for recorded movements (with audit)
- Reports and analytics
- Finance calendar of upcoming payouts
- Platform settings (commercial terms live here, not hardcoded as the only source of truth)
- Audit log
- Payout simulation for staff (what the engine *would* do — not a client-facing tool)

### 3.4 Trust features anyone can use

- **Certificate / document verification** on a public page: a serial or token can be checked without an account
- Branded PDFs (certificate, receipts) that match what finance issued
- Clear separation of **office copy** and **client copy** on cash receipts so the desk and the investor do not share one ambiguous paper

### 3.5 Platform administration

A restricted developer surface exists for:

- staff account hygiene
- system status
- operational logs appropriate to platform health
- settings that are not financial policy

This surface cannot replace finance confirmation of payouts or withdrawals.

---

## 4. How the business actually works

Commercial numbers (return rate, fees, lock-in, minimums, referral terms) are published to investors through the **website, client agreement, and in-app settings**. They are configurable by finance. This document does not republish a formula sheet. The **control model** is what matters.

### 4.1 Dual control

Every rupee that enters or leaves the book follows the same idea:

```
Investor or desk initiates
        │
        ▼
Operations verifies people and paper
  (KYC, proof, completeness)
        │
        ▼
Finance authorizes the money event
  (activate, confirm payout, approve withdrawal)
        │
        ▼
System writes the ledger, issues documents, notifies people
```

Operations cannot complete a paid investment. Finance should not be the first pair of eyes on a blurry Aadhaar. The split is deliberate.

### 4.2 Investor lifecycle

```
Visitor
  → lead (optional)
  → register and prove email
  → KYC + fee proof
  → operations verifies
  → finance activates capital
  → ACTIVE investor
       │
       ├─ monthly returns (system prepares, finance confirms)
       ├─ top-ups (same dual control)
       ├─ referrals (system tracks, finance releases)
       └─ withdrawal (lock-in + dual control → ledger + receipt)
```

An investor is not “active” because they filled a form. They are active when finance has accepted capital onto the book.

### 4.3 Two plan types

| Plan | Investor experience | Capital on the book |
|---|---|---|
| **Payout (simple)** | Monthly return is paid out | Principal stays the base |
| **Growth (compound)** | Monthly return stays invested | Book value grows when returns are confirmed |

A client may hold these as distinct plans. Top-ups attach to a plan as **lots** so later deposits do not silently rewrite the original deal.

### 4.4 Lots, not a single blob of cash

Real investors add money later. A naive system would dump everything into one number and mis-pay the first month. GrowEdge treats later deposits as lots with their own start-of-return timing, still rolled up in the investor’s portfolio.

Finance can see the lots. The investor sees a coherent account, not an engineering diagram.

### 4.5 Returns are proposed, then confirmed

The platform does **not** silently wire money on a cron clock.

1. Automation builds the upcoming return queue (who is due, on which day, for which plan).
2. Finance reviews and confirms.
3. Only then is the return a ledger fact.
4. Compound plans update book value from confirmed returns, not from a guess.
5. The investor is notified and can download the matching receipt where the desk requires it.

If finance has not confirmed it, it is not paid. That is the safety property.

### 4.6 Withdrawals

Withdrawals are requested by the investor (or raised at the desk), checked against lock-in and remaining-capital rules, reviewed by staff, and completed only when finance marks the outflow. Partial withdrawals cannot leave a nonsense residual; full withdrawal closes the plan.

Early-exit exceptions are **policy decisions**, not a silent engine bypass.

### 4.7 Referrals

Referrals are first-class: an investor can invite, the system tracks the relationship, and rewards accrue only against an active referred investment. Top-ups do not automatically inflate referral base. Finance marks rewards paid. Receipts exist for that event too.

### 4.8 Cash desk vs bank / UPI

Not every movement is a neat UTR. The desk still takes cash. For those events the platform issues:

- **Client copy** — what the investor keeps
- **Office copy** — what the firm keeps, including wet-ink signature practice at the desk

Bank and UPI events carry proof (screenshot, UTR) through the same approval chain. PDFs are generated from the ledger, not from a Word template on someone’s laptop.

---

## 5. Domain model (conceptual)

These are the ideas the system is built around. Column lists, indexes, and table names are not published.

```
User identity ──1:1── Investor profile
                         │
                         ├── KYC documents
                         ├── Bank / UPI / nominee
                         ├── Investments (plans)
                         │      ├── Lots / top-ups
                         │      ├── Transactions (ledger)
                         │      ├── Payout schedule
                         │      └── Certificates
                         ├── Fund requests (deposits / outflows in flight)
                         ├── Withdrawals
                         ├── Receipts (client + office where required)
                         ├── Notifications
                         └── Referral edge (who invited whom)

Staff identity ── role ── operations | finance | platform

Leads (pre-account CRM)
Audit log (append-only actions)
Platform settings (commercial and operational knobs)
Monthly run markers (so automation cannot double-apply a period)
```

**Ledger first.** Documents, emails, and dashboards are projections of the ledger. If a PDF and the ledger disagree, the ledger wins and the PDF is regenerated by finance — not edited in isolation.

**Identity split.** Login identity and investor profile are related but not the same object. Staff users are not fake investors.

**Requests vs facts.** A fund request is an intention. A confirmed transaction is a fact. Automation may create *pending* return rows. Finance turns them into facts.

---

## 6. System architecture

### 6.1 Shape

The production system is a **modular monolith** with a clear API boundary, not a mesh of undocumented microservices. That is a choice: financial consistency beats fashion.

```
┌──────────────────────────────────────────────────────────┐
│  Clients: browsers (and later, native apps against the     │
│  same API). No business logic of record in the UI.         │
└────────────────────────────┬─────────────────────────────┘
                             │ HTTPS
┌────────────────────────────▼─────────────────────────────┐
│  Edge: CDN + app hosting (Vercel)                          │
│  Static UI, security headers, same-origin API proxy        │
└────────────────────────────┬─────────────────────────────┘
                             │
┌────────────────────────────▼─────────────────────────────┐
│  Application services (Railway)                            │
│                                                            │
│  FastAPI  ·  auth & RBAC  ·  domain routes                 │
│  Financial engine (pure calculation)                       │
│  Document factory (certificates, receipts, statements)     │
│  Notification dispatch                                     │
│                                                            │
│  Scheduler (leader-elected)  ·  Task workers (queue)       │
└───────┬──────────────┬──────────────┬──────────┬─────────┘
        │              │              │          │
        ▼              ▼              ▼          ▼
   PostgreSQL        Redis      Object store    Email
   (Supabase)     cache, OTP,    (R2)         + optional
                   queue, locks               staff calendar
```

### 6.2 Stack (what we are willing to name)

| Layer | Choice | Why |
|---|---|---|
| Investor & staff UI | React, TypeScript, Vite, Tailwind | One app, strict types, fast iteration |
| Marketing | Next.js | SEO and content |
| API | Python, FastAPI, Pydantic | Explicit contracts, async I/O |
| Persistence | PostgreSQL, SQLAlchemy 2 async, Alembic | Relational integrity for money |
| Cache / coordination | Redis | Sessions, abuse controls, job locks, queues |
| Files | Cloudflare R2 (S3 API) | Private documents off the API disks |
| Jobs | In-process scheduler + Celery workers | Time-based finance work + heavy PDF/email |
| Email | Transactional providers with SMTP fallback | OTP and lifecycle mail must not be a single-vendor bet |
| Optional identity | Third-party Google / phone verification | Extra registration paths without replacing email proof as the default |
| Hosting | Vercel + Railway + Supabase + R2 | Each vendor does the job it is good at |

Money math uses **decimal** semantics, not floating-point UI leftovers. Time for business rules is **Asia/Kolkata**, not “whatever the server clock is.”

### 6.3 Trust boundaries

| Zone | Trust |
|---|---|
| Browser | Untrusted. May be scripted, shared, or stolen. |
| Marketing site | Public. Can submit leads. Cannot move money. |
| Authenticated app | Trusted only after role checks on the **server**. |
| API | The enforcement point. UI hiding a button is not security. |
| Database | Server-side only. |
| Object storage | Server-side only. The browser receives short-lived access through the API, not a permanent bucket URL. |
| Job workers | Same trust as the API. They are not a backdoor around RBAC; they run privileged work the API has already authorized or that policy allows as system work. |

The important rule: **the UI never decides that a payout is paid.**

### 6.4 Why a monolith

Payout confirmation, certificate generation, referral check, and audit write need a **single transaction story**. Splitting “returns service” from “ledger service” too early creates dual writes and ghost money. Modules inside one API (auth, finance, operations, documents, engine) give isolation of *code* without isolation of *truth*.

Workers exist for slow or retryable work (PDF, email, CRM sync), not for a second ledger.

---

## 7. Engineering depth

This is the part that is easy to undersell and dangerous to over-specify. Below are **properties the system is built to have**, not a source listing.

### 7.1 Financial engine as a library

Calculation is separated from HTTP.

- Routes authenticate, authorize, load state, and persist.
- The engine answers questions: what is the book value, what return is due, what is withdrawable, what would a top-up do to timing.
- Staff simulation uses the same ideas as production confirmation, so finance can preview without writing the ledger.

That split is how you avoid “the dashboard shows 4.2 lakh but the payout job used 3.9 lakh.”

### 7.2 Idempotency

Monthly returns and referral periods are exactly the kind of work that gets you double-paid if a job retries.

The platform treats a return period as a **unique fact per investment**. If the daily job, the weekly job, the startup job, and a finance button all try to create “August for this plan,” one of them wins and the others no-op. Locks exist so those jobs do not interleave into two pending rows.

Idempotency is a financial control, not a performance trick.

### 7.3 Leader election for clocks

The API may run more than one process. Only one process is allowed to be the clock. A distributed lock decides the leader so two machines cannot both create the month’s returns.

### 7.4 State machines, not booleans

Onboarding, KYC, fund requests, investments, and withdrawals are **lifecycles**. Invalid jumps are rejected. You cannot mark paid something that was never approved. You cannot activate an investor who never existed as a user. Side effects (certificate, receipt, email, referral edge) hang off transitions, not off random extra buttons.

### 7.5 Serials and documents

Certificates and receipts carry **serials**. Serial generation is coordinated so two finance clerks cannot mint the same number. Public verification checks the serial against issued documents, not against a guessable pattern published here.

PDFs are generated server-side from templates and uploaded to private storage. Regeneration is a finance action and is auditable.

### 7.6 Event-shaped side effects

After a domain fact is written (KYC submitted, deposit confirmed, return confirmed, withdrawal completed), the system fans out:

- investor email
- staff email
- in-app notification
- optional WhatsApp where configured
- CRM / spreadsheet sync for leads
- PDF generation

Fan-out is **after** the database write. If email is down, the investment still exists. Jobs retry. That is an intentional order: ledger first, theater second.

### 7.7 Read models and hot dashboards

Finance dashboards aggregate a lot of investors. The API is built to aggregate on the server (not by downloading every row to the browser), with caching on expensive read models and invalidation when money events happen. Portal bootstrap is a single “what I need to render home” payload so mobile networks do not waterfall ten calls.

### 7.8 Human-in-the-loop by default

Anything irreversible with money has a person in the loop. Automation **prepares**. People **confirm**. That is slower than a fully automatic yield protocol. It is also how a desk stays honest.

### 7.9 Internationalization and device reality

The investor UI is built as a responsive web app (phones are the real device). Copy can be localized. Native distribution, when used, is a shell around **that same app and API**, not a second ledger.

### 7.10 Observability

Production API logs are structured. Failures in scheduled finance jobs notify operators. There is a public-facing status page path for “is the product up,” separate from internal traces. Error tracking is wired for server faults, not for harvesting investor payloads into a third-party playground.

---

## 8. Machines (automation)

“Machines” here means **scheduled and queued workers**, not trading bots. They do not send capital to banks by themselves.

| Machine | What it is for | Safety |
|---|---|---|
| **Upcoming payout sync** | Keep the finance queue populated for the next window of payout days | Idempotent; cannot double-create a period |
| **Wider periodic sync** | Catch anything a daily pass missed | Same guards |
| **Month-open full sync** | Make sure the new month’s book is queued | Same guards |
| **Startup bootstrap** | After a deploy, do not wait until tomorrow to be correct | Same guards; not a second ledger |
| **Integrity audit** | Periodically recompute and *log* disagreements; does not silently rewrite history | Alert, don’t invent money |
| **Finance calendar sync** | Push upcoming payout / referral dates to the staff calendar | Read of ledger, write to calendar — not the reverse |
| **Daily finance digest** | Email a picture of what is due | Notification only |
| **Return reminder** | Nudge staff if a cycle has not been confirmed | Notification only |
| **Stale signup cleanup** | Remove abandoned registrations that never became KYC | Never touches funded investors |
| **Async workers** | PDFs, email, lead CRM sync | Retryable; cannot approve payouts |

**Design rules for machines**

- One leader.
- Coalesce overlapping runs.
- Unique period keys for returns.
- Failure emails to operators.
- Finance confirmation still required for the return to be real.
- Exact clocks, lock key names, and window sizes are not published. Publishing them would help someone time load or reconstruct the job graph. They exist; they are internal.

---

## 9. Document factory

Documents are how a desk proves itself to an investor and to itself.

| Document | Issued when | Copies |
|---|---|---|
| Investment certificate | Finance activates (and on lawful regenerate) | Investor + firm record |
| New investment / deposit receipt | Capital is accepted | Client copy; office copy for cash |
| Additional deposit receipt | Top-up accepted | Same split where cash |
| Monthly payout receipt | Return confirmed (especially cash payout) | Client + office where cash |
| Withdrawal receipt | Outflow completed | Client + office where cash |
| Referral receipt | Reward released | Client + office where needed |
| Monthly statement | Period close / investor request | Investor copy |

**Properties**

- Generated from current ledger + investor identity (legal name, client code, signatures on file).
- Stored privately.
- Streamed through authenticated API routes.
- Cash office copies are designed for **wet-ink** at the desk; the PDF is the printed form, not a replacement for the signed paper.
- Public verification lets a third party check genuineness without opening the whole archive.

The factory is slow and CPU-heavy, so large batches go to workers. A failed PDF does not roll back a confirmed payout; finance can regenerate.

---

## 10. Communication

| Channel | Used for |
|---|---|
| Email | OTP, onboarding, KYC decisions, deposit / payout / withdrawal facts, staff alerts, broadcasts |
| In-app notifications | Same facts, inside the portal |
| WhatsApp | Optional operational pings where the investor has given a number and policy allows it |
| Staff calendar | Upcoming payout workload |
| CRM spreadsheet | Lead capture follow-up for operations (not the ledger) |

Templates are branded HTML. Sending is asynchronous. Staff inboxes for finance and operations are configured in the environment, not hardcoded in public docs.

The platform will not describe every template name or trigger table here. That list is an enumeration of attack and phishing opportunities.

---

## 11. Security

Security is described as **posture**. Thresholds, algorithms-in-use beyond the obvious, header configs, and “gaps we still have” are not listed. An honest public security section can still be deep: it says what you **optimize for**, what you **refuse**, and how you **think**.

### 11.1 Threat model (the one that matters)

| Attacker | What they want | What we assume |
|---|---|---|
| Random internet | OTP spam, account stuffing, scraping | Always on |
| Other investors | Someone else’s portfolio, documents, or referral rewards | IDOR is the default bug class in this product; tests treat it as such |
| Stolen laptop / shared cybercafe | Session reuse | Sessions must die on password change and on competing login policy |
| Malicious or phished staff | Unauthorized payout or data export | RBAC + dual control + audit; staff  credentials are not investor OTP |
| Insider with finance access | The job itself is powerful | Audit log, dual control with ops, no silent engine apply |
| Competitor | Clone the desk | Source stays private; this doc stays capability-level |
| Document thief | KYC images, PAN, Aadhaar | Storage is private; access is scoped and time-bounded |

We do **not** pretend a public README makes you un-hackable. We do refuse to publish a map of the locks.

### 11.2 Principles

1. **Server is the security boundary.** Hidden routes in React are UX, not ACL.
2. **Least privilege.** Investors see themselves. Operations verifies. Finance authorizes. Platform admin is not a second finance chief.
3. **Dual control on money.** One role must not be able to complete the full path from “stranger uploaded a screenshot” to “capital is live” without the other.
4. **Confirm, don’t infer, for payouts.** Jobs prepare; people confirm.
5. **Private documents.** KYC and certificates are not world-readable objects with lucky URLs.
6. **Secrets stay in secret stores.** Nothing in this repository, nothing in the SPA bundle, nothing in PDFs.
7. **Abuse resistance.** Authentication and OTP are rate-limited and monitored. Exact limits are not public.
8. **Auditability.** Privileged actions write an append-only trail (who, what, when, from where).
9. **Fail closed on auth, fail open on email.** If identity is unclear, deny. If mail delivery is down, the ledger still stands.
10. **Do not help attackers.** This file will not include bypasses, payload examples, header tricks, or “we currently lack X.”

### 11.3 Controls (capability list)

**Identity**

- Separate investor and staff login paths
- Staff: password credentials with a password quality policy
- Investors: proof-of-email as the primary factor; password optional for established accounts
- Session invalidation when credentials change
- Refresh rotation so stolen access is not immortal
- Device-aware login signals and staff geo alerts (non-blocking warnings, not a claimed firewall)

**Authorization**

- Role checks on every privileged API
- Object-level scoping (your id, not “any uuid in the URL”)
- Automated tests whose only job is “client A cannot read client B”

**Transport and browser**

- HTTPS everywhere
- Strict CORS allowlist
- Security headers (framing, sniffing, CSP) on the app host
- Production frontend without source maps

**Input and files**

- Typed request bodies
- Amount bounds on money fields
- File type and size checks before storage
- Unpredictable stored filenames
- Request size caps

**Data**

- TLS to the database
- Passwords hashed with a slow hash
- PII in Postgres and private object storage, not in git
- Presigned or proxied file access, scoped to the caller

**Operations**

- Structured logs with request correlation
- Security-oriented alerting on abusive auth patterns
- Dependency scanning in CI
- Branch protection expected on the private production repo
- Production config crash-fast if the signing secret is still a placeholder

**What we will not claim in public**

- That we are certified against a named standard unless the firm publishes that certificate separately
- That staff 2FA, WAF, or malware scanning details are a certain way
- That a number of “ pentest findings closed” exists for you to scrape

If you are a security researcher: see [Responsible disclosure](#responsible-disclosure).

### 11.4 Safety of money, distinct from cybersecurity

A lot of “security” in this product is **book safety**:

- pending vs confirmed transactions
- lock-in checks
- remaining-capital rules on partial withdrawal
- unique return periods
- finance confirmation
- office vs client receipts
- regeneration of documents instead of Photoshop
- integrity jobs that *report* drift

Those controls protect investors from the software being sloppy. They are as important as TLS.

---

## 12. Reliability and scale

The platform is built for a **real desk**: dozens to hundreds of active investors first, with headroom, not for a fictional million-QPS launch.

What that means in engineering terms:

- Async I/O for the API so document uploads and mail do not stall unrelated requests
- Connection pooling to Postgres, with a pooler in production
- Redis shared across processes so rate limits and OTP are not per-box fiction
- Object storage so the API containers stay stateless
- CDN for the UI
- Workers for CPU-heavy PDFs
- Graceful degradation: if the queue is down, lead capture still writes the database; CRM sync catches up

Capacity numbers from internal load exercises are **not** published. They go stale and they become a dare.

---

## 13. Quality

Production launch (June 2026) shipped with a substantial automated backend suite (financial calculations, business rules, auth, onboarding, KYC, RBAC) plus production smoke checks. Frontend has lint, production build, and optional Playwright journeys.

Tests that matter for a desk:

- money math
- “this role cannot hit that route”
- “this investor cannot open that investor”
- onboarding cannot skip proof
- payouts do not double-create

Internal QA checklists exist. They are not in this repository.

---

## 14. What we built against (engineering history, short)

The product went from first production commits to a live investor app in a compressed 2026 cycle, then kept accumulating the unglamorous layer that actually makes a desk safe: dual receipts, lot-aware top-ups, finance calendar, referral receipts, integrity jobs, credential handoff for legacy clients, KYC backfill for people who were funded on paper first.

The interesting problems were not “can we put a dashboard in React.” They were:

- two interest modes that must not share a naive `balance` field
- top-ups that must not earn in the wrong month
- cash desk reality vs UPI reality
- staff who will click twice
- jobs that will overlap after a restart
- documents that must match the book years later
- investors on mobile networks in India
- KYC images that must never leak through a static URL

That is the engineering. The private repo holds the code that does it.

---

## 15. Roadmap (public)

Items we are willing to say out loud:

- Native investor distribution against the existing API (same backend, not a rewrite)
- Continued hardening (the unglamorous kind)
- Richer statements and tax-time exports as finance requires
- Stronger staff authentication options as policy requires

Items we will not put on a public roadmap: vendor migrations, unpublished rate changes, unreleased control gaps.

---

## 16. What this document does not contain

On purpose, you will **not** find:

- Source code, infrastructure-as-code, or Terraform
- Environment variable names or values
- API route catalogs or example requests
- JWT contents, header names, cookie recipes
- Rate-limit numbers, OTP shape, lock TTLs, or Redis key layouts
- Database schemas, index names, or SQL
- Exact commercial formulas beyond what the public website already tells investors
- Cron clocks, worker counts, pool sizes, or deploy commands
- Staff emails, phone numbers, or UPI IDs
- Runbooks, incident notes, or “known gaps”
- Exploit write-ups, payload samples, or bypass narratives
- Test passwords or seed users
- Object-storage path layouts
- Anything that would let you *operate* or *attack* the system without being us

If you found a gist or screenshot that contradicts this policy, treat it as leaked internal material, not as public spec.

### Why this is still “deep”

Depth in public is **models and constraints**: dual control, ledger-first documents, idempotent periods, leader-elected clocks, role split, private files, human confirmation of money. Depth that would be irresponsible is **reproduction**: the exact function, the exact limit, the exact header.

A competitor can learn that we are serious. They cannot ship our desk from this file. An attacker can learn that we thought about IDOR and double-payouts. They cannot read a tutorial for bypassing either.

---

## 17. Responsible disclosure

If you believe you have found a security issue in GrowEdge Capital production systems:

1. **Do not** open a public GitHub issue with proof-of-concept details.
2. Email the firm through the contact path on [growedgecapital.com](https://growedgecapital.com) with a description of the impact.
3. Give a reasonable window before any public discussion.
4. Do not access other people’s data, do not run destructive tests, and do not use this as permission to attack the app.

We will not publish a bug bounty table here. That is a policy document, not a README dare.

---

## 18. Legal

© GrowEdge Capital. All rights reserved.

This documentation is provided for **information**. It is not an offer, not investment advice, not a client agreement, and not a specification you may implement.

The GrowEdge Capital name, logo, certificate design, receipt design, and software are protected. You may not copy the product, scrape the app, or represent yourself as the firm.

Investor terms (returns, fees, lock-in, withdrawal, referral) are governed by the **client agreement and in-app disclosures**, not by this README.

The production source remains private. This public repository contains documentation only.

---

nyone may reuse this as a product spec” — All Rights Reserved is the correct default.

If you later want a second public page (security.txt, brand guidelines), keep the same rule: capabilities and principles, never knobs and never people.
