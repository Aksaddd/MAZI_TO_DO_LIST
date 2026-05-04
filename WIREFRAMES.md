# Mazi — Wireframes

Low-fidelity ASCII wireframes for the screens in `SYSTEM_DESIGN.md`. The
intent is to lock layout and information hierarchy before any pixels.

**Global shell:** left nav (collapsible), top bar with global search + sync
status, main content area. Lead detail always opens as a right-side drawer
over the current screen — never a full-page navigation.

---

## 0. Global shell

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ ≡  mazi          🔍 Search leads, companies, notes…       ⟳ synced 2m  👤    │
├────────┬─────────────────────────────────────────────────────────────────────┤
│        │                                                                     │
│ ◉ Today│                                                                     │
│ ▦ Pipe │                                                                     │
│ ☰ Leads│                  ── main content area ──                            │
│ 📅 Cal │                                                                     │
│ ✓ Tasks│                                                                     │
│ 📊 Stats│                                                                    │
│        │                                                                     │
│ ─────  │                                                                     │
│ ⚙ Setup│                                                                     │
└────────┴─────────────────────────────────────────────────────────────────────┘
```

- Sync indicator turns amber if last sync >30m, red on error (click → sync log).
- `⌘K` opens command palette from anywhere.

---

## 1. Today (home / default screen)

The morning view. Answers: *"what do I do right now?"*

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Today  ·  Tue, May 5                                       [+ New lead  N]   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ▌ Next up — 9:30 AM                                                         │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  📅  Discovery — Acme Corp · Jane Doe (VP Sales)             in 24m    │  │
│  │      Stage: Discovery  ·  Value: $24,000  ·  Source: Referral          │  │
│  │      Notes from last touch: "Interested in Q3 rollout, send pricing"   │  │
│  │      [ Open lead ]   [ Join Meet ↗ ]   [ Reschedule ]                  │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ▌ Follow-ups due today (4)                       [ Snooze all ]  [ View ]   │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  ☐  Call back Mark @ Initech              · Qualified  · $8k     ✓ ⋯  │  │
│  │  ☐  Send proposal to Globex               · Proposal   · $42k    ✓ ⋯  │  │
│  │  ☐  Check in: Hooli (no reply 5d)         · Discovery  · $15k    ✓ ⋯  │  │
│  │  ☐  LinkedIn DM: Sara @ Umbrella          · New        · —       ✓ ⋯  │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ▌ Stale leads — no activity in 7+ days (3)                       [ View ]   │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  ⏳  Wayne Enterprises    · Discovery  · 11d quiet  · $30k             │  │
│  │  ⏳  Stark Industries      · Proposal   · 9d quiet   · $55k             │  │
│  │  ⏳  Pied Piper            · Qualified  · 8d quiet   · $12k             │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ▌ Today at a glance                                                         │
│  ┌──────────────┬──────────────┬──────────────┬──────────────────────────┐  │
│  │ Meetings     │ Tasks done   │ Pipeline open│ This week vs last        │  │
│  │   3          │   2 / 6      │  $312k       │  +8% activities  ▲       │  │
│  └──────────────┴──────────────┴──────────────┴──────────────────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**Why this layout:** the next meeting is the single most time-sensitive
decision — it gets the top slot. Then due follow-ups (Tasks). Then stale
leads (proactive nudge). KPIs go *last*, not first — they're context, not
action.

---

## 2. Pipeline (Kanban)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Pipeline    [ All sources ▾ ]  [ This quarter ▾ ]  [ Owner: me ]  [ + New ] │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  NEW (12)        DISCOVERY (8)     QUALIFIED (5)    PROPOSAL (4)   WON (2)   │
│  $48k            $124k             $98k             $186k          $67k      │
│  ┌──────────┐    ┌──────────┐      ┌──────────┐    ┌──────────┐   ┌───────┐  │
│  │ Acme     │    │ Hooli    │      │ Initech  │    │ Globex   │   │ Soylnt│  │
│  │ Jane D.  │    │ T. Brad. │      │ Mark R.  │    │ Sara K.  │   │ ✓ won │  │
│  │ $24k  🟢 │    │ $15k  🟡 │      │ $8k   🟢 │    │ $42k  🟢 │   │ $30k  │  │
│  │ ▶ Call   │    │ ⏳ 11d   │      │ ▶ Call   │    │ ▶ Send   │   │       │  │
│  └──────────┘    └──────────┘      └──────────┘    └──────────┘   └───────┘  │
│  ┌──────────┐    ┌──────────┐      ┌──────────┐    ┌──────────┐   ┌───────┐  │
│  │ Umbrella │    │ Wayne    │      │ Stark    │    │ Pied Pip │   │ Cyber.│  │
│  │ Sara T.  │    │ Bruce W. │      │ Tony S.  │    │ R. Hend. │   │ ✓ won │  │
│  │ — 🟡     │    │ $30k 🟡  │      │ $55k 🟡  │    │ $12k 🟢  │   │ $37k  │  │
│  └──────────┘    └──────────┘      └──────────┘    └──────────┘   └───────┘  │
│                                                                              │
│                         (drag cards horizontally to advance stage)           │
│                                                                              │
│  LOST (collapsed ▸)                                                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Card legend: 🟢 hot (score 70+) · 🟡 warm (40–69) · ⚪ cold (<40)
- ⏳ icon means "stale" — no activity in 7+ days.
- ▶ shows the next action (verb + 1 word). Click runs it.
- Click anywhere on a card → opens detail drawer (see §4).
- Keyboard: `←/→` move selected card between stages.

---

## 3. Leads — table / inbox view

Spreadsheet-feel. The transition from Google Sheets has to feel familiar.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Leads   [ View: Hot leads ▾ ]  [ + Filter ]  [ Columns ]  [ ⬇ Export CSV ]  │
├──────────────────────────────────────────────────────────────────────────────┤
│ ☐ │ Name           │ Company       │ Stage      │ Value │ Source   │ Next ▾ │
├───┼────────────────┼───────────────┼────────────┼───────┼──────────┼────────┤
│ ☐ │ Jane Doe       │ Acme Corp     │ Discovery  │ $24k  │ Referral │ Today  │
│ ☐ │ Mark Roberts   │ Initech       │ Qualified  │ $8k   │ Inbound  │ Today  │
│ ☐ │ Sara Kim       │ Globex        │ Proposal   │ $42k  │ Outbound │ Today  │
│ ☐ │ Bruce Wayne    │ Wayne Ent.    │ Discovery  │ $30k  │ Event    │ Fri    │
│ ☐ │ Tony Stark     │ Stark Ind.    │ Proposal   │ $55k  │ Referral │ Fri    │
│ ☐ │ Richard Hend.  │ Pied Piper    │ Qualified  │ $12k  │ Inbound  │ Mon    │
│ ☐ │ T. Bradshaw    │ Hooli         │ Discovery  │ $15k  │ Outbound │ —      │
│ ☐ │ Sara Tate      │ Umbrella      │ New        │ —     │ LinkedIn │ —      │
│   │                │               │            │       │          │        │
├──────────────────────────────────────────────────────────────────────────────┤
│  Showing 8 of 142   ·   Selected: 0     [ Bulk: stage ▾ owner ▾ delete ]    │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Inline edit: click a cell → edit. Tab/enter to save.
- Saved views in the dropdown: *Hot leads · No next action · Quiet 7d+ ·
  Closing this month · By source · All*.
- `J/K` move selection up/down. `Enter` opens detail drawer.

---

## 4. Lead detail drawer (overlay)

Slides in from the right over whatever screen you were on. `Esc` closes.

```
                                ┌─────────────────────────────────────────────┐
                                │ ← Acme Corp · Jane Doe          🟢 78  ⋯ ✕ │
                                │ VP Sales · jane@acme.com · (555) 010-2244   │
                                ├─────────────────────────────────────────────┤
                                │ Stage:  [ Discovery       ▾ ]   $24,000     │
                                │ Owner:  me        Source: Referral          │
                                │                                             │
                                │ ▌ Next action                               │
                                │  ┌────────────────────────────────────────┐ │
                                │  │ Call to confirm Q3 timeline             │ │
                                │  │ 📅 Today, 2:00 PM       [ Done ✓ ]     │ │
                                │  └────────────────────────────────────────┘ │
                                │                                             │
                                │ [ Activity ]  Fields   History   Files      │
                                │ ──────────                                  │
                                │  Today                                      │
                                │   📅  Discovery call · 30 min · 9:30 AM     │
                                │       (linked from Google Calendar)         │
                                │                                             │
                                │  Yesterday                                  │
                                │   ✉  Sent intro email                       │
                                │   📝  Note: "Asked about SSO support"       │
                                │                                             │
                                │  May 1                                      │
                                │   ✓  Task done: LinkedIn connect            │
                                │                                             │
                                │  ── Log activity ─────────────────────────  │
                                │  [ Call ] [ Email ] [ Meeting ] [ Note ]    │
                                │                                             │
                                │ Source row: Sheets · "Leads Q2" · Row 47 ↗ │
                                └─────────────────────────────────────────────┘
```

- Score badge top-right; click → shows which scoring rules contributed.
- Activity timeline merges Calendar events, Tasks, manual notes.
- "Source row" deep-links back to the original Google Sheet row.

---

## 5. Calendar view

Weekly view with leads attached to each event.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Calendar   [ Week ▾ ]   ‹  May 5 – May 11, 2026  ›        [ + New event ]   │
├────────┬────────┬────────┬────────┬────────┬────────┬────────┬───────────────┤
│        │  Mon 5 │  Tue 6 │  Wed 7 │  Thu 8 │  Fri 9 │ Sat 10 │   Sun 11     │
├────────┼────────┼────────┼────────┼────────┼────────┼────────┼───────────────┤
│  9 AM  │        │ ▓▓▓▓▓▓ │        │ ▓▓▓▓▓  │        │        │              │
│        │        │ Acme   │        │ Globex │        │        │              │
│ 10 AM  │ ▓▓▓▓▓  │ Jane D │        │ Sara K │        │        │              │
│        │ Hooli  │        │        │        │        │        │              │
│ 11 AM  │        │        │ ▓▓▓▓▓▓ │        │ ▓▓▓▓▓▓ │        │              │
│        │        │        │ Initech│        │ Wayne  │        │              │
│ 12 PM  │  ─lunch│        │ Mark R │        │ Bruce  │        │              │
│  1 PM  │        │        │        │        │        │        │              │
│  2 PM  │ ▓▓▓▓▓  │        │        │ ▓▓▓▓▓  │        │        │              │
│        │ Stark  │        │        │ Pied   │        │        │              │
│  3 PM  │        │        │        │        │        │        │              │
└────────┴────────┴────────┴────────┴────────┴────────┴────────┴───────────────┘
   Click event → opens lead drawer. Drag to reschedule (writes back to Google).
```

- Events without a linked lead show in gray; one-click "Link to lead…".
- Color-coded by stage (Discovery/Proposal/etc.) so you can see deal mix at
  a glance.

---

## 6. Tasks / follow-ups

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Follow-ups        [ Today ] [ This week ] [ Overdue (2) ] [ Done ]  [ + ]   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Overdue                                                                     │
│   ☐ Send proposal to Globex          · Sara K  · was Mon         ⋯          │
│   ☐ Follow up Wayne Enterprises      · Bruce W · was Fri         ⋯          │
│                                                                              │
│  Today                                                                       │
│   ☐ Call Mark @ Initech              · Mark R  · 2:00 PM         ⋯          │
│   ☐ LinkedIn DM Sara @ Umbrella      · Sara T  · —               ⋯          │
│   ☐ Check in Hooli                   · T.Brad. · —               ⋯          │
│                                                                              │
│  Tomorrow                                                                    │
│   ☐ Discovery prep — Stark Ind.      · Tony S  · 8:00 AM         ⋯          │
│                                                                              │
│  This week                                                                   │
│   ☐ Quarterly review — Pied Piper    · R.Hend. · Thu              ⋯          │
│   ☐ Demo follow-up — Acme            · Jane D  · Fri              ⋯          │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Each task is linked to a lead (avatar/name on the right). Click → drawer.
- ✓ marks task done; syncs back to Google Tasks.
- "+ " quick-adds: type "Call Acme tomorrow 2pm" → parsed into title + due.

---

## 7. Analytics dashboard

Five cards, in the order from §7.3 of the design doc.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Analytics       [ Last 30 days ▾ ]                          [ Compare on ]   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ▌ What needs me                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │   8 overdue follow-ups   ·   5 leads with no next action               │  │
│  │   3 stale 7d+            ·   2 meetings without prep notes             │  │
│  │                                                       [ Fix it now → ] │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ▌ Pipeline funnel                                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  New ████████████████████████████████   142                            │  │
│  │  Discovery  ████████████████   78   (55% →)                            │  │
│  │  Qualified  █████████   42         (54% →)                             │  │
│  │  Proposal   █████   22             (52% →)                             │  │
│  │  Won        ██   9                 (41% →)                             │  │
│  │                                                                        │  │
│  │  Overall conversion: 6.3%       Worst drop-off: Proposal → Won         │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌────────────────────────────┬───────────────────────────────────────────┐  │
│  │ ▌ Sales velocity           │ ▌ Lead source ROI                         │  │
│  │                            │ ┌──────────┬──────┬─────┬──────┬───────┐ │  │
│  │     $11.4k / day           │ │ Source   │ Lead │ Won │ Win% │  Rev  │ │  │
│  │     ▲ +18% vs prior 30d    │ │ Referral │  34  │  6  │ 17%  │ $98k  │ │  │
│  │                            │ │ Inbound  │  62  │  2  │  3%  │ $24k  │ │  │
│  │  ▁▂▃▂▃▅▆▅▆▇▇▆▇▇▇          │ │ Outbound │  41  │  1  │  2%  │ $12k  │ │  │
│  │                            │ │ Event    │   5  │  0  │  0%  │  —    │ │  │
│  └────────────────────────────┴──┴──────────┴──────┴─────┴──────┴───────┘  │
│                                                                              │
│  ▌ Stuck deals (in stage > 14 days)                                          │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │  Stark Industries  · Proposal · 21d  · $55k       [ Nudge ] [ Open ]   │  │
│  │  Wayne Enterprises · Discovery · 18d · $30k       [ Nudge ] [ Open ]   │  │
│  │  Pied Piper        · Qualified · 15d · $12k       [ Nudge ] [ Open ]   │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Every chart is clickable → drills to the filtered lead list. No dead-ends.
- "Compare on" overlays prior period as faded line.

---

## 8. Setup — connect Google + map columns (onboarding)

One-time wizard, three steps.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Setup          Step 2 of 3   ●●○                                             │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Map your spreadsheet                                                       │
│   We read row 1 of "Leads Q2" — confirm where each field lives.              │
│                                                                              │
│   ┌─────────────────────────┬───────────────────────────────────────────┐   │
│   │  Sheet column           │  Maps to                                  │   │
│   ├─────────────────────────┼───────────────────────────────────────────┤   │
│   │  Full Name              │  [ Name           ▾ ]    ✓ auto-detected  │   │
│   │  Company                │  [ Company        ▾ ]    ✓ auto-detected  │   │
│   │  Email                  │  [ Email          ▾ ]    ✓ auto-detected  │   │
│   │  Phone                  │  [ Phone          ▾ ]    ✓ auto-detected  │   │
│   │  Pipeline Stage         │  [ Stage          ▾ ]    ✓ auto-detected  │   │
│   │  Deal Value             │  [ Value          ▾ ]    ✓ auto-detected  │   │
│   │  Where from             │  [ Source         ▾ ]                     │   │
│   │  Industry               │  [ Custom: text   ▾ ]                     │   │
│   │  Notes                  │  [ Notes          ▾ ]                     │   │
│   │  Created                │  [ Created at     ▾ ]                     │   │
│   └─────────────────────────┴───────────────────────────────────────────┘   │
│                                                                              │
│   Stages found: New · Discovery · Qualified · Proposal · Won · Lost          │
│   Looks right?  [ Edit stages ]                                              │
│                                                                              │
│                                       [ ← Back ]      [ Continue → ]         │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

- Step 1: Sign in with Google + grant Sheets/Calendar/Tasks scopes.
- Step 2: Pick spreadsheet + tab + map columns (above).
- Step 3: Pick Calendar(s) + Tasks list(s) to sync, then "Run first sync".

---

## 9. Command palette (`⌘K`)

```
                    ┌──────────────────────────────────────────────┐
                    │ 🔍  jane                                     │
                    ├──────────────────────────────────────────────┤
                    │  Leads                                       │
                    │   ▸ Jane Doe — Acme Corp · Discovery         │
                    │   ▸ Jane Smith — Hooli · New                 │
                    │  Actions                                     │
                    │   ▸ + New lead "jane"                        │
                    │   ▸ + New task for Jane Doe                  │
                    │   ▸ Log call with Jane Doe                   │
                    │  Navigate                                    │
                    │   ▸ Today      ⌘1                            │
                    │   ▸ Pipeline   ⌘2                            │
                    └──────────────────────────────────────────────┘
```

The fastest path between any two things in the app.

---

## 10. Visual style notes

Not a design system yet — guardrails so it stays clean:

- **Density**: comfortable, not cramped. Compromise between Linear (dense)
  and Notion (airy).
- **Type**: Inter for UI, JetBrains Mono for any tabular numbers.
- **Color**: neutral grays + one accent (probably a confident green for
  "won/positive"). Stage colors are muted, not crayon.
- **Motion**: drawer slides in 150ms, kanban drag uses spring. Everything
  else is instant — no fade transitions.
- **Empty states**: each empty screen has one CTA, never a sad illustration
  with nothing to do.

---

## 11. What's intentionally not here

- A separate "Companies" entity. v1 treats company as a string on the lead;
  add an entity once you have ≥3 leads at the same company often enough that
  it hurts.
- Email composition. You'll use Gmail. We just log that it happened.
- Reports builder. The five fixed dashboard cards are enough until they
  aren't.
- Mobile screens. Read-only mobile in v2 if it matters.
