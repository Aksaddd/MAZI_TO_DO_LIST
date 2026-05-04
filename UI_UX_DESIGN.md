# Mazi — UI/UX design principles (built for flow state)

A sales rep doing deep work on their pipeline is no different from a writer
in flow or a programmer in the zone. The interface either gets out of the way
and **becomes invisible** — or it breaks the spell. This doc codifies how we
build for the former.

---

## 1. The 8 flow conditions → 8 product rules

Csikszentmihalyi's flow research identifies eight conditions. Each one becomes
a non-negotiable product rule.

| Flow condition | Our rule | What it means in the UI |
|---|---|---|
| **Clear goals** | One primary action per screen | Each screen has exactly one obvious "what to do now" — visually dominant, top-left or top-center. Secondary actions are quieter. |
| **Immediate feedback** | Optimistic UI, zero spinners under 300ms | Click "Done" → checkmark in 16ms. Network reconciles silently. If it fails, an undo toast — never a modal. |
| **Challenge/skill balance** | Progressive disclosure | New user sees 5 cards on Today. Power user sees the same 5 cards plus shortcuts hints fade in. Never paywalled features dangling in the nav. |
| **Concentration / no distraction** | Quiet visual field | One accent color. No badges-on-badges. Notifications batched, never modal. Sync indicator is a 6px dot, not a banner. |
| **Sense of control** | Keyboard-first, undoable | Every action has a shortcut. Every destructive action has 10s undo. No "are you sure?" modals. |
| **Loss of self-consciousness** | No anxiety triggers | No streaks shaming you. No "your manager will see this" copy. No empty-state guilt. |
| **Time distortion** | Don't interrupt | No toast notifications during typing. No popovers triggered by mouse-near. Sync, sounds, and pings respect a "do not disturb" focus mode. |
| **Intrinsic reward** | Visible momentum | Pipeline value bar fills as you move deals. Today's tasks shrink as you complete them. Subtle haptic-style pulse on `Done`. |

---

## 2. Concrete design system

### Typography

- **UI:** Inter, 14px base, 1.5 line-height. Weights: 400 / 500 / 600 only.
- **Numbers/data:** Inter with `font-variant-numeric: tabular-nums`. Aligned
  decimals make scanning a pipeline value column effortless.
- **Headings:** -0.01em letter-spacing, never bold uppercase.
- **Hierarchy via size + weight, never color.** Color is for state.

### Color

A reduced palette. The interface should feel like good paper, not a casino.

```
Neutrals (light mode)
  bg-base       #fafaf9   page
  bg-raised     #ffffff   cards, drawers
  bg-sunken     #f4f4f3   table stripes, inputs
  border        #e7e5e4   1px hairlines only
  text-primary  #0c0a09   headings, key data
  text-body     #44403c   prose
  text-muted    #78716c   meta, labels
  text-subtle   #a8a29e   placeholders

Accent (one, that's it)
  accent        #16a34a   the only color that says "go"
  accent-soft   #dcfce7   accent backgrounds

State (used sparingly, never decoratively)
  warn          #d97706   stale, overdue
  danger        #dc2626   errors only — never used for "delete" buttons

Stage colors (muted, not crayon — used in 8px dots, not fills)
  new           #94a3b8
  discovery     #60a5fa
  qualified     #a78bfa
  proposal      #f59e0b
  won           #16a34a
  lost          #78716c
```

Dark mode mirrors this with the same hue logic, just inverted lightness.

### Spacing

- 4px base grid. Use `4 / 8 / 12 / 16 / 24 / 32 / 48 / 64`. Nothing else.
- Touch targets ≥ 36px even on desktop. Click anxiety kills flow.
- 16–24px between unrelated groups; 8–12px within a group. **Whitespace is
  the design.**

### Motion

- 150ms standard, 80ms for state toggles, 220ms for drawer slide.
- Easing: `cubic-bezier(0.2, 0.8, 0.2, 1)` (slight spring, no bounce).
- **Reduced-motion respected** — instantly, no fades.
- No animation on page load. No animated icons. No hover-popover-pop reveals.

### Iconography

- Lucide icon set, 16px or 20px only, 1.5px stroke. Never filled.
- Icons paired with text on first use; icon-only allowed once a user has done
  the action 3+ times (we measure this).

---

## 3. Interaction patterns

### Keyboard system (the spine of the app)

```
Navigation
  ⌘K       Command palette (search anything, run any action)
  ⌘1–6     Jump to nav: Today / Pipeline / Leads / Calendar / Tasks / Stats
  G then T Today    G then P Pipeline    (Vim-style chords)

In any list
  J / K    Move selection down/up
  Enter    Open detail drawer
  Esc      Close drawer / clear selection
  /        Focus filter
  N        New (lead/task — context-aware)

In the drawer
  E        Edit field
  C        Log call    M  Log meeting    .  Log note
  S        Move stage  → / ←  Stage forward / back
  ⌘Enter   Save and close
```

Every shortcut is **discoverable** — hold `⌘` to dim the UI and overlay every
available shortcut on the corresponding control. Users learn by doing.

### Optimistic everything

- State updates happen in the local cache the instant you act.
- Network confirms in the background.
- On failure: toast with `Undo` (10s) and `Retry`. Never a blocking modal.
- A small heartbeat dot in the top bar shows sync health — present but quiet.

### One-handed flow

The most common loop — **see next action → do it → log it → next** — is
designed to be doable from the keyboard alone, in under 5 seconds:

```
J            (next lead in the queue)
Enter        (open drawer)
C            (log call)
"left vm"    (auto-typed, parsed)
⌘Enter       (save + close + advance to next)
```

The mouse exists for kanban dragging and chart drilling. Everything else is
hands-on-keyboard.

### Empty states

Every empty state has **one CTA, never two**, and never a sad illustration.
Examples:

- Today, all done: a single line — *"You're clear. Pipeline opens with `⌘2`."*
- No leads yet: *"Connect your sheet → see your pipeline in 30 seconds."*
- Stale leads = 0: *"Nothing's rotting. Nice."*

### Notifications & focus mode

- Default: toast in bottom-right, 4s, never overlapping content you're
  reading.
- Focus mode (`⌘.`): suppresses all notifications, dims the nav, hides the
  sync indicator. The only thing visible is the lead you're working on.
- Calendar reminders fire 5 minutes before — a single banner, dismissable
  with `Esc`.

---

## 4. Information density

Two density modes. The user picks once, in setup, based on monitor size.

- **Comfortable** (default, ~24px row height) — for laptops, fewer rows but
  easier on the eye in long sessions.
- **Compact** (~18px row) — for big monitors and power users.

Both modes obey the same hierarchy and spacing rules, just scaled.

---

## 5. The home screen as a flow gateway

The Today screen is the most important UI in the app because it sets the
*emotional tone* of the next 30 minutes of work. It must:

1. **Open in under 200ms** (server-rendered, hydrated late).
2. **Show one clear next thing** above the fold — the next meeting or top
   follow-up — at hero scale, not a list item.
3. **Promise a finite session** — "4 follow-ups due today," not "47 leads
   need attention." Bounded scope is psychologically critical.
4. **Reward completion** — as you check things off, the list visibly shrinks
   and the hero card advances. Momentum is the dopamine.

---

## 6. What we will NOT do

A list of common SaaS patterns we explicitly ban because they break flow:

- Onboarding checklists that follow you around for weeks.
- "How are we doing?" feedback popups.
- Confetti animations on completion.
- Dark patterns: hidden cancel, dim "no thanks," default-on toggles.
- Notifications about other users' activity (single-user app, but worth
  saying — even Calendar invites are surfaced quietly).
- Tooltips that block the thing you were about to click.
- Loading skeletons longer than 200ms (if it's slower, we redesign the
  request, not the spinner).

---

## 7. Measuring whether we got it right

A flow-state app should be measurable. Track (locally, never sent home):

- **Time to first action** after opening the app (target: <3s)
- **Keyboard:mouse action ratio** (target: 70:30 for power users)
- **Session length** (target: clusters of 15–45 min — too short = friction,
  too long = doom-scrolling pipeline)
- **Action latency p95** (target: <100ms perceived for any local action)

If a screen scores poorly, redesign that screen — don't just tweak it.

---

See `prototype.html` for a working visual demo of these principles applied
to the Today screen, Pipeline, and Lead drawer.
