# Close Sprint

You are preparing a sprint for closure. This command fills in the retrospective section of the sprint plan document, identifies carry-forward items, and calculates velocity. It does NOT mark the sprint Complete — that is a PO decision made via `update-status` after UAT sign-off.

Run this command after UAT is complete and the PO has confirmed all scenarios passed (or deferred failing ones with documented decisions).

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a project path and a sprint number. Examples:

```
/close-sprint projects/ASCENDRA-PAY 01
/close-sprint projects/ASCENDRA-PAY 07
```

Extract:
- **Project path** — e.g. `projects/ASCENDRA-PAY`
- **PROJECT_CODE** — from project path
- **SPRINT_NUMBER** — strip whitespace, then zero-pad to two digits (e.g. `1` → `01`, `7` → `07`, `01` stays `01`). State the normalised value: "Sprint number: {NN}"

If missing:
> "Provide project path and sprint number (e.g. `/close-sprint projects/ASCENDRA-PAY 01`)."

---

## Step 2 — Confirm UAT is complete

Before reading any files, ask:

> "Confirm UAT is complete before filling the retrospective:
> - Has the Product Owner signed off the UAT checklist? (yes/no)
> - Are there any outstanding defects that are blocking closure? (yes/no)"

Wait for confirmation. If UAT is not complete:
> "Sprint closure should happen after UAT sign-off. Complete UAT first using the checklist at `projects/{PROJECT_CODE}/sprints/sprint-{NN}-uat-checklist.md`. Come back once the PO has signed off."

---

## Step 3 — Read required files

Read both files in full:

1. `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md` — the sprint plan document; you are updating the Retrospective section at the bottom
2. `projects/{PROJECT_CODE}/stories/index.md` — Section 1 Story Registry; to check which stories are Done vs Merged vs still in progress

---

## Step 4 — Assess story completion

From Section 8 (Sprint Planning) in `stories/index.md`, expand all story IDs for Sprint {NN}.

For each story, check its current status in Section 1:

| Story ID | Title | Size | Status |
|----------|-------|------|--------|

Classify:
- **Done** — AC verified, ready for closure
- **Merged** — PR merged but not yet marked Done (awaiting final QA confirmation)
- **Carry-forward** — any story not in Done or Merged status

---

## Step 5 — Calculate velocity

**Velocity = count of Must Have stories in Done or Merged status at sprint end**

Record:
- Total stories in sprint
- Must Have completed
- Should Have completed (if included)
- Carry-forward count (and their total size)
- Sprint dates (from the sprint plan header) — calculate elapsed days if both dates are present

---

## Step 6 — Gather retrospective inputs

Ask all at once:

> "Retrospective for Sprint {NN} — answer these to fill the sprint record:
>
> 1. **What went well?** (1–3 things worth repeating)
> 2. **What to improve?** (1–3 things to change next sprint)
> 3. **For any carry-forward stories:** What is the target sprint for each? (e.g. US-01-008 → Sprint 02)"

Wait for the answers before writing anything.

---

## Step 7 — Update the sprint plan document

Update `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md`. Fill in only the Retrospective section — do not change any other section.

Replace the empty retrospective placeholders with:

```markdown
## Sprint Retrospective

**What went well:**
[User's answers — 1 item per bullet]

**What to improve:**
[User's answers — 1 item per bullet]

**Carry-forward items:**
| Story ID | Title | Reason | Target Sprint |
|----------|-------|--------|---------------|
[One row per carry-forward story, with target sprint from user's answer]

[If no carry-forward: "None — all sprint stories completed."]

**Velocity:** {N} stories completed ({N} Must Have + {N} Should Have) in {N} days
**Carry-forward:** {N} stories ({total size}) moving to Sprint {target sprint numbers}
```

---

## Step 8 — Check for forward-looking learnings

This is the checkpoint where sprint learnings are meant to reach not-yet-built epics — `/close-sprint`'s retrospective (Step 6) only logs process notes; it does not by itself correct any requirement, epic, or story. That correction path is `/assess-change`, and this is where it gets triggered, before the next wave's `/gen-stories` runs against possibly-stale scope.

Ask:

> "Before starting the next wave: did anything from this sprint's delivery, verification, or UAT reveal a gap, defect, or scope issue relevant to an epic or requirement that hasn't had stories written yet? (yes/no — if yes, describe it, or list them one at a time)"

Wait for PO response.

- **If no:** proceed to Step 9.
- **If yes:** for each item raised, tell the PO:
  > "Run `/assess-change projects/{PROJECT_CODE} \"[the item]\"` now, before generating stories for the next epic — this classifies the change and propagates it through BRD/epics/stories as needed. I'll wait here until that's done, or you can tell me to proceed without it."
  Do not run `/assess-change` automatically — it is a separate command with its own confirmation gates. Wait for the PO to either run it or explicitly say to proceed without it.

---

## Step 9 — Report to the user

After updating the file:

```
SPRINT {NN} RETROSPECTIVE FILLED
─────────────────────────────────────────
Sprint file:        projects/{PROJECT_CODE}/sprints/sprint-{NN}.md
Stories Done:       {N} of {total} ({N} Must Have, {N} Should Have)
Carry-forward:      {N} stories → targeting Sprint {NN+1}
Velocity:           {N} stories in {N} days

Retrospective summary:
  What went well:   [first item]
  To improve:       [first item]
─────────────────────────────────────────
```

Then tell the user:

> **Next steps:**
> 1. Mark the sprint Complete once UAT sign-off is on record: `update-status projects/{PROJECT_CODE} sprint {NN} Complete`
> 2. Generate release notes: `/gen-release-notes projects/{PROJECT_CODE} {NN} <version>`
> 3. If Step 8 raised any items, run `/assess-change` for them before continuing — do not start the next wave against stale scope.
> 4. Start the next wave: identify the next epic in dependency order from `epics/index.md` Section 3 (checking for story-level interleaving notes), then `/gen-stories` for it, followed by `/review-stories` — which assigns the sprint number as part of its own Sprint Assignment step. Only run `/gen-sprint-plan projects/{PROJECT_CODE} {NN+1}` once that assignment exists; it does not decide sprint scope itself.
>
> Note: Carry-forward stories from this sprint are still `Merged` status and already carry a sprint assignment — no naming or planning gap. If the next wave alone would leave Sprint {NN+1} under capacity, the Sprint Assignment step will surface combining it with carry-forward stories or the next-next epic.
