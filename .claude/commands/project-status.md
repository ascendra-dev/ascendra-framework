# Project Status Dashboard

You are generating a text-based status dashboard for an Ascendra project. The dashboard reads the epics index and stories index and outputs a clean, scannable view of project progress — epic health, story completion, sprint tracking, and an overall progress indicator.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Read the index files

If `$ARGUMENTS` contains a project path (e.g. `projects/ASCENDRA-PAY`), read:
1. `projects/{PROJECT_CODE}/epics/index.md` — epic statuses, REQ coverage, dependencies
2. `projects/{PROJECT_CODE}/stories/index.md` — story registry, sprint plan, should-have register, flags register

Read only these two files. Do not read individual epic or story files — the indexes contain all the status information needed.

If `$ARGUMENTS` is empty, ask:
> "Which project? Provide the project path (e.g. `projects/ASCENDRA-PAY`)."

---

## Step 2 — Compute the metrics

From the two index files, compute every number below before rendering the dashboard. Do the arithmetic explicitly — do not estimate.

**Epic metrics:**
- Total epics
- Count by status: Done / In Progress / Approved / Draft / Deprecated
- % complete = Done ÷ (Total − Deprecated) × 100, rounded to nearest whole number

**Story metrics:**
- Total stories (exclude Deprecated from denominators)
- Count by status: Done / Merged / PR Created / Changes Requested / In Progress / Reviewed / Draft / Deprecated
- "Closed" = Done + Merged (both mean work is complete or merged)
- "Open" = everything except Closed and Deprecated
- % complete = Closed ÷ (Total − Deprecated) × 100

**Story metrics by size:**
- Count of XS / S / M / L remaining (not yet Merged or Done)
- This shows the remaining workload weight

**Sprint metrics — for each sprint in the sprint plan:**
- Sprint status — read the **Status column** from the Section 8 sprint table in stories/index.md. Do not infer status from story statuses. Values are Planning / Active / Complete.
- Total stories in sprint — read the Count column from the same row
- Done + Merged count — cross-reference Section 1 Story Registry: find stories that belong to this sprint (expand any US-XX-001 → US-XX-009 ranges into individual story IDs) and count those with Done or Merged status
- In Progress + PR Created count — same cross-reference
- Not started count (Draft + Reviewed) — same cross-reference
- % complete for the sprint = (Done + Merged) ÷ Count × 100

**Should Have stories:**
- Total Should Have stories
- How many are Done or Merged

**Flags:**
- L-sized stories not yet resolved (from Flags Register in stories index)
- Any deprecated stories or epics

---

## Step 3 — Render the dashboard

Output the dashboard in this exact format. Use plain ASCII only — no emoji, no Unicode box characters that may not render correctly in a terminal.

```
================================================================================
  PROJECT: [Full project name and phase]
  Generated: [today's date]
================================================================================

OVERALL PROGRESS
  [====================....................] [N]% ([closed] / [total] stories closed)

--------------------------------------------------------------------------------
EPICS                                                         [N] / [total] Done
--------------------------------------------------------------------------------

  [EPIC-001] [Title]                                              [Status]
  [EPIC-002] [Title]                                              [Status]
  [EPIC-003] [Title]                                              [Status]
  ...

  Summary:  Done [N]  |  In Progress [N]  |  Approved [N]  |  Draft [N]

--------------------------------------------------------------------------------
STORIES BY STATUS
--------------------------------------------------------------------------------

  Done            [N]   [=============================................] [N]%
  Merged          [N]
  PR Created      [N]
  Changes Req.    [N]
  In Progress     [N]
  Reviewed        [N]
  Draft           [N]
  Deprecated      [N]   (excluded from progress)

  Total active: [N]   Closed: [N]   Remaining: [N]

--------------------------------------------------------------------------------
REMAINING WORKLOAD (open stories by size)
--------------------------------------------------------------------------------

  L   [N] stories   (complex — flag for PO split review if not yet split)
  M   [N] stories
  S   [N] stories
  XS  [N] stories

  Should Have stories remaining: [N] / [total Should Have]

--------------------------------------------------------------------------------
SPRINT TRACKER
--------------------------------------------------------------------------------

  Sprint [NN]  [status]    [bar]  [done]/[total]  [N]%
  Sprint [NN]  [status]    [bar]  [done]/[total]  [N]%
  Sprint [NN]  [status]    [bar]  [done]/[total]  [N]%
  ...

  (Use [=] for completed slots, [>] for in-progress slots, [.] for not started)
  Example: [=========>>..........]  7 done, 2 in progress, 10 not started

--------------------------------------------------------------------------------
ATTENTION
--------------------------------------------------------------------------------

  [List any items that need action. If none, write "None — project is on track."]

  Examples of items to flag here:
  - L-sized stories in active sprint not yet split (from Flags Register)
  - Stories marked "Changes Requested" (PO has open review feedback)
  - Active sprint is > 50% of its duration with < 30% stories closed (potential slip)
  - Epics with In Progress status but no stories in In Progress state (stale)
  - Dependencies: any story/epic marked In Progress whose dependency is not yet Done

================================================================================
```

**Progress bar rendering:**
- Bar width: 30 characters
- `=` for complete slots
- `>` for in-progress slots (round to nearest bar slot)
- `.` for not started slots

Example: 27 of 101 stories closed → `[========>......................] 27%`

---

## Step 4 — Output the dashboard

Print the dashboard directly. Do not add commentary or explanation around it — let the dashboard speak for itself.

If there are items in the ATTENTION section, add one sentence after the dashboard:
> "Run `/update-status` to update any stale statuses, or `/assess-change` to handle any scope changes."

If there are no attention items:
> "All clear. No blockers detected."
