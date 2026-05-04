# Mazi Sales & Leads Analytics — System Design

**Author:** CTO draft, v0.1
**Status:** Proposal — pending sign-off before implementation
**Branch:** `claude/sales-analytics-system-design-afynv`

---

## 1. Goal

Turn the user's Google Sheets of leads and sales data into a working operating
system for sales: a single UI to (a) keep leads organized, (b) move them through
a pipeline, and (c) see analytics that drive the next action — not vanity charts.

**Non-goals (for v1):** replacing Google Sheets entirely, marketing automation,
email sequencing, billing/invoicing, multi-tenant SaaS.

---

## 2. Market scan — what currently works

Quick scan of what successful tools in this category get right, so we copy the
patterns that work and skip the ones that don't.

| Tool | What works | What we'll borrow |
|------|------------|-------------------|
| **Pipedrive** | Visual kanban pipeline; activity-based selling (every deal has a "next action"); customizable dashboard cards; goal tracking | Kanban as the primary view; "next action" as a first-class field on every lead |
| **HubSpot** | Unified contact/deal/company model; templated dashboards; lifecycle stages; AI-assisted reporting | Lifecycle stages, templated starter dashboards |
| **Close** | Inbox-style lead view; keyboard-first; "what should I do next?" focus | Inbox view + keyboard shortcuts |
| **Coefficient / Coupler.io** | Two-way Google Sheets sync as a first-class integration | Sheets as source-of-truth at MVP, scheduled + manual sync |
| **Notion / Airtable** | Spreadsheet-feeling table views with rich filters/saved views | Saved views, inline edit on the table |

**Anti-patterns to avoid:**

- Burying the "next action" under three clicks (Salesforce problem).
- Forcing a rigid schema before you know your fields (kills adoption).
- Big bang dashboards full of metrics nobody acts on. Pick few, make them
  decision-driving.

**Metrics that actually matter** (per consensus across Amplemarket, Outreach,
MNTN, Lucky Orange):

- **Stage-by-stage conversion %** (lead → MQL → SQL → won)
- **Sales velocity** = (# opps × avg deal size × win rate) ÷ cycle length
- **Avg time-in-stage** (find where deals rot)
- **Lead source ROI** (CPL vs. revenue won, per source)
- **Pipeline coverage** (open pipeline ÷ remaining quota)
- **Win/loss reasons** (qualitative, surfaced as a tag cloud)

These are the v1 dashboard. Everything else is v2.

---

## 3. High-level architecture

```
 ┌──────────────────┐         ┌────────────────────────────────┐
 │  Google Sheets   │◄───────►│   Sync worker (Node, BullMQ)   │
 │  (source today)  │  OAuth  │   - pull on cron + on-demand   │
 └──────────────────┘         │   - push on user edit (v1.1)   │
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                              ┌────────────────────────────────┐
                              │   Postgres (canonical store)   │
                              │   + Redis (queues, cache)      │
                              └──────────────┬─────────────────┘
                                             │
                              ┌──────────────┴─────────────────┐
                              │   Next.js app (UI + API)       │
                              │   - tRPC/REST                  │
                              │   - NextAuth (Google OAuth)    │
                              └──────────────┬─────────────────┘
                                             │
                                             ▼
                                ┌────────────────────────┐
                                │   Browser (React)      │
                                │   shadcn/ui + Tremor   │
                                └────────────────────────┘
```

**Why this shape:**

- **Postgres as the canonical store, Sheets as the import surface.** Sheets API
  has read quotas (~300 reads/min/project) and is too slow to query on every
  page load. We pull into Postgres and serve all UI/analytics from there.
- **Sync is one-way at MVP** (Sheets → app). Two-way sync is a foot-gun (merge
  conflicts, accidental overwrites). We ship two-way in v1.1 once the schema
  has settled and we have an audit log.
- **Single Next.js app** for UI + API for v1. Split out a worker service when
  job volume justifies it.

---

## 4. Tech stack

| Layer | Choice | Why |
|-------|--------|-----|
| Frontend | **Next.js 15 (App Router) + React + TypeScript** | One framework for UI + API; you ship faster |
| UI kit | **shadcn/ui + Tailwind** | Owned components, no library lock-in |
| Charts | **Tremor** (built on Recharts) | Built for dashboards, looks good out of the box |
| Tables | **TanStack Table** | Filtering, sorting, virtualization for big lead lists |
| Data fetching | **TanStack Query** + **tRPC** | End-to-end typesafe API |
| Auth | **NextAuth (Auth.js) w/ Google OAuth** | You already use Google; reuses Sheets scopes |
| DB | **Postgres 16** (Neon or Supabase to start) | Boring, reliable, rich SQL for analytics |
| ORM | **Prisma** or **Drizzle** | Drizzle if we want raw-SQL escape hatch for analytics |
| Queue | **BullMQ** + Redis (Upstash) | Cron sync + retry on Sheets API failures |
| Hosting | **Vercel** (app) + **Neon** (db) + **Upstash** (redis) | Zero-ops, generous free tiers |
| Sheets I/O | **googleapis** Node SDK | Official, OAuth out of the box |

**Estimated infra cost at MVP:** $0–$25/mo. Scales to ~$80/mo before we'd need
to think about it.

---

## 5. Data model (canonical, in Postgres)

Minimal to start; easy to extend.

```
User              id, email, googleRefreshToken, role
Workspace         id, name, ownerUserId
SheetSource       id, workspaceId, spreadsheetId, sheetName,
                  columnMap (JSON: sheetCol → field), lastSyncedAt
Lead              id, workspaceId, externalRowId, name, company, email, phone,
                  source, stageId, ownerUserId, score, value, currency,
                  createdAt, updatedAt, customFields (JSONB)
Stage             id, workspaceId, name, order, type (open|won|lost)
Activity          id, leadId, type (call|email|meeting|note), body,
                  dueAt, completedAt, createdByUserId
LeadHistory       id, leadId, fieldChanged, from, to, changedAt, source (user|sync)
SyncRun           id, sheetSourceId, startedAt, finishedAt, status, rowsAdded,
                  rowsUpdated, errors (JSON)
ScoringRule       id, workspaceId, field, operator, value, points, enabled
SavedView         id, workspaceId, userId, name, filters (JSON), columns (JSON)
```

**Key decisions:**

- `customFields` as JSONB so users can map arbitrary Sheet columns without
  schema migrations. Indexed via `GIN` for filtering.
- `LeadHistory` from day one. Without it you cannot compute time-in-stage,
  velocity, or "what changed" — and you can't add it retroactively.
- `externalRowId` is a stable hash of (spreadsheetId, sheetName, rowKey) so
  re-sync is idempotent.

---

## 6. Google Sheets integration

**MVP flow:**

1. User connects Google account via OAuth (scopes: `spreadsheets.readonly`,
   `drive.metadata.readonly`).
2. User picks a spreadsheet + tab.
3. **Column mapping wizard:** we read row 1, suggest mappings (`name`,
   `email`, `company`, `stage`, `value`, `source`, `owner`), user confirms,
   anything unmapped goes to `customFields`.
4. Initial backfill: full sheet → `Lead` rows, deduped by email or row hash.
5. **Scheduled sync:** every 15 min via BullMQ cron + a "Sync now" button.
6. **Diff strategy:** hash each row; only upsert changed rows. Log to
   `SyncRun` and `LeadHistory`.

**v1.1 — two-way sync:** when user edits a lead in the app, write back to the
Sheet via `values.update`. Conflict policy: last-write-wins with a banner if
the cell changed in Sheets since the last sync (user picks).

**Why not Google Apps Script / a Sheet sidebar?** Apps Script is fine for
quick wins but doesn't give us a real DB, real auth, or real analytics. Worth
revisiting if the user prefers staying inside the Sheet UI for editing.

---

## 7. UI design

Three primary surfaces. Keyboard-first wherever possible.

### 7.1 Pipeline (kanban) — default view

- Columns = stages, drag to move a lead forward.
- Each card: name, company, value, **next action + due date**, owner avatar,
  score badge.
- Quick-add lead with `N`. Move with arrow keys.
- Filter bar: owner, source, value range, stage age.

### 7.2 Leads inbox (table)

- Spreadsheet-feel: dense rows, inline edit, multi-select bulk actions.
- Saved views (e.g., "My hot leads", "No activity in 7 days", "New this week").
- Row click opens a **detail drawer** — never a full page nav. Keeps context.

### 7.3 Dashboard

Five cards, in this order, because this is the order of decisions a sales
person makes in the morning:

1. **What needs me today** — overdue activities + leads with no next action.
2. **Pipeline by stage** — funnel chart with conversion % between stages.
3. **Sales velocity (last 30d vs. prior 30d)** — single big number + sparkline.
4. **Source ROI table** — leads, won, win rate, revenue, per source.
5. **Stuck deals** — deals in stage > P75 time-in-stage, sorted by value.

Every chart is clickable → drills into the filtered lead list. No dead-end
charts.

### 7.4 Lead detail drawer

- Header: name, company, score, stage selector, owner.
- Tabs: **Activity timeline** (default), **Fields**, **History**.
- Right rail: next action input (always visible), value, source, links to
  the original Sheet row.

---

## 8. Lead scoring (v1)

Rules engine, not ML. ML on a few hundred leads is theater.

- User defines weighted rules in settings: `if source = "Referral" → +20`,
  `if email domain matches enterprise list → +15`, etc.
- Score recomputed on lead create/update and on rule change.
- Score shown as a 0–100 badge with color bands (cold/warm/hot).
- v2: learn weights from won/lost outcomes (logistic regression on tabular
  features).

---

## 9. Security & permissions

- Google OAuth refresh tokens encrypted at rest (`pgcrypto` or app-level KMS).
- Row-level workspace scoping on **every** query (enforce in a query helper,
  not by convention).
- Roles: `owner`, `admin`, `member`. Members see only their leads unless
  granted "see all".
- Audit log = `LeadHistory` + `SyncRun`. Exportable.
- Secrets in Vercel env, never in repo. `.env.example` only.

---

## 10. Roadmap

**MVP (Weeks 1–3) — "It replaces my Sheet-staring time"**

- Google OAuth + Sheets connect + column mapping
- One-way sync (cron + manual)
- Lead table + detail drawer + inline edit
- Pipeline kanban with drag-to-stage
- Dashboard cards 1, 2, 3 (today, funnel, velocity)
- Activity logging (manual)

**v1 (Weeks 4–6) — "It tells me what to do"**

- Saved views + filters
- Lead scoring rules engine
- Source ROI + stuck deals dashboard cards
- Bulk actions, CSV export
- Keyboard shortcuts pass

**v1.1 (Weeks 7–8) — "It writes back"**

- Two-way Sheets sync with conflict UI
- Email reminders for overdue next actions
- Slack notification on stage change (optional)

**v2 (later) — bets, not commitments**

- Calendar/Gmail integration to auto-log activities
- Learned lead scoring
- Forecasting view
- Mobile-friendly read view

---

## 11. Open questions for you

Before I start building, I need answers on:

1. **Single user or team?** Changes auth complexity and the data model.
2. **One spreadsheet or many?** And do leads + deals live in separate tabs?
3. **What stages do you use today?** Send me a screenshot of the Sheet and
   I'll pre-build the column mapping.
4. **Any existing definition of "qualified"?** Drives the scoring defaults.
5. **Hosting preference** — Vercel + Neon is the fastest path; happy to use
   your existing infra if you have one.

---

## 12. Sources

- [Pipedrive vs HubSpot 2026 (Zapier)](https://zapier.com/blog/pipedrive-vs-hubspot/)
- [Best CRM for Lead Management 2026 (Success Knocks)](https://successknocks.com/best-crm-for-lead-management-2026-picks/)
- [Pipedrive vs HubSpot (Salesflare)](https://blog.salesflare.com/compare-salesforce-zoho-hubspot-pipedrive)
- [Google Sheets Sales Dashboard (Coupler.io)](https://blog.coupler.io/google-sheets-sales-dashboard/)
- [Sales Pipeline Tracker in Google Sheets (Coefficient)](https://coefficient.io/sales-operations/build-sales-pipeline-tracking-google-sheets)
- [15+ Sales Funnel KPIs (Amplemarket)](https://www.amplemarket.com/blog/15-of-the-most-useful-sales-funnel-metrics-to-optimize-your-sales-processes)
- [7 most useful sales funnel metrics (Outreach)](https://www.outreach.ai/resources/blog/sales-funnel-metrics)
- [B2B Sales Funnel Metrics (Lead Forensics)](https://www.leadforensics.com/blog/b2b-sales-pipeline-metrics-the-essential-list/)
