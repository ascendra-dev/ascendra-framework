# Update Status

You are updating the lifecycle status of a project artifact on the Ascendra platform. This command handles stories, epics, sprints, BRDs, architecture documents, and project briefs. For most artifact types, it updates the Status field in the artifact file AND the corresponding row in any index file — both together, never one without the other. The one exception: `brief.md`'s **Content Status** field (document readiness) never touches `projects/index.md` — only its separate **Status** field (project lifecycle) does. See the Brief / Project update section in Step 4.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain: a file path or identifier, and a new status value. Parse them as follows:

**Story update:**
`/update-status projects/ASCENDRA-PAY/stories/EPIC-001/US-01-003-core-schema.md "In Progress"`

**Epic update:**
`/update-status projects/ASCENDRA-PAY/epics/EPIC-001-platform-foundation.md "In Progress"`

**Story plan update:**
`/update-status projects/ASCENDRA-PAY/story-plans/US-01-001-plan.md Confirmed`

**Sprint status update:**
`/update-status projects/ASCENDRA-PAY sprint 03 Active`

**BRD update:**
`/update-status projects/ASCENDRA-PAY/brd-v1.md Approved`

**Architecture update:**
`/update-status projects/ASCENDRA-PAY/architecture/arch-v1.md Locked`

**Brief Content Status update (document readiness):**
`/update-status projects/ASCENDRA-PAY/brief.md Approved`

**Brief/Project Status update (overall lifecycle — same path shape, different value):**
`/update-status projects/ASCENDRA-PAY/brief.md "On Hold"`

Detect artifact type from the file path:
- Path contains `/story-plans/` → story plan (check before the `/stories/` rule below — both could match a naive substring check, but `/story-plans/` is a distinct folder)
- Path contains `/stories/` → story
- Path contains `/epics/` → epic
- Path contains `sprint` keyword → sprint
- Path contains `/brd-` or filename starts with `brd-` → BRD
- Path contains `/architecture/` → architecture
- Path contains `brief.md` → brief

If arguments are missing or ambiguous, ask:
> "What do you want to update? Provide: the artifact path + new status. Examples: a story/epic file path, `projects/{CODE}/brd-v1.md`, `projects/{CODE}/architecture/arch-v1.md`, `projects/{CODE}/brief.md`, or project path + 'sprint' + number + status."

---

## Step 2 — Validate the status value

Valid statuses differ by artifact type. Reject any value not in the list below and tell the user the valid options.

**`brief.md` has two distinct status fields — never confuse them:**

**Content Status** (this document's own readiness — governs the `**Content Status:**` field only; in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Placeholders only, awaiting `/run-intake` |
| `Under Review` | Being revised or reviewed |
| `Brief complete` | `/run-intake` finished; awaiting PO approval |
| `Approved` | Gates `/gen-domain-knowledge` |
| `Locked` | No further changes without a formal change request |
| `Superseded` | Replaced by a later brief version |

**Status** (the project's overall lifecycle — governs the `**Status:**` field, kept in sync with `projects/index.md`'s Status column; in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Active` | Project in delivery |
| `On Hold` | Paused — reason must be recorded in brief Section 8 |
| `Complete` | Project delivered and support period ended |
| `Cancelled` | Project cancelled — all work preserved for reference |

These two value sets never overlap — which field applies is determined by the value given, not by any extra argument. A path pointing at `brief.md` with value `Approved` updates **Content Status**; the same path with value `Active` updates **Status** (and `projects/index.md` alongside it). See the Brief / Project update section in Step 4 for the exact routing.

**BRD statuses** (in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Being generated or revised |
| `Under Review` | Submitted for PO review |
| `Approved` | PO approved — requirements locked; `/gen-epics` unblocked |
| `Locked` | Requirements frozen — formal change request required for any change |

**Architecture statuses** (in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Being generated or revised |
| `Under Review` | Submitted for PO review |
| `Locked` | PO approved — is now law; `/implement-story` and `/gen-sprint-plan` unblocked |

**Story statuses** (in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Generated — not yet reviewed |
| `Reviewed` | Passed story review process |
| `In Progress` | `/implement-story` is running for this story |
| `PR Created` | PR submitted — awaiting PO review |
| `Changes Requested` | PO has requested changes |
| `Merged` | PR merged to main branch |
| `Done` | Acceptance criteria verified by QA — final state |
| `Deprecated` | Requirement removed — story no longer valid |

**Story plan statuses** (`FW-031`, in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Generated by `/gen-story-plan` — awaiting PO review |
| `Confirmed` | PO reviewed — `/implement-story` is unblocked for this story |

**Epic statuses** (in lifecycle order):

| Status | Meaning |
|--------|---------|
| `Draft` | Generated — not yet reviewed |
| `Approved` | Passed epic review — ready for story generation |
| `In Progress` | At least one story is In Progress or beyond |
| `Done` | All stories Merged and QA-verified |
| `Deprecated` | Epic removed — requirement descoped |

**Sprint statuses:**

| Status | Meaning |
|--------|---------|
| `Planning` | Sprint plan being assembled |
| `Active` | Sprint is in progress |
| `Complete` | All sprint stories are Done |

---

## Step 3 — Read the relevant files

**For a story update:** read the story file and `projects/{PROJECT_CODE}/stories/index.md`
**For a story plan update:** read the plan file only — there is no story-plans index
**For an epic update:** read the epic file and `projects/{PROJECT_CODE}/epics/index.md`
**For a sprint update:** read `projects/{PROJECT_CODE}/stories/index.md` only
**For a BRD update:** read the BRD file only
**For an architecture update:** read the architecture file only
**For a brief update:** read the brief file always. If the new value is a Status (project lifecycle) value, also read `projects/index.md` (if it exists) — Content Status values never touch that file

---

## Step 4 — Apply the update

### Story update

**In the story file:**
- Find the `**Status:**` line in the header (it appears after the Layer line)
- Replace the current status value with the new one
- Do not change any other field

**In the stories index (`stories/index.md`):**
- Find the row in the story registry table matching this Story ID
- Update the Status column value

**Additional actions by transition:**

| Transition | Additional action |
|-----------|-----------------|
| Any status → `In Progress` | Check the story's Section 6 (Dependencies). If any dependency story is not yet `Merged` or `Done`, warn: "This story has unresolved dependencies: [list]. Are you sure you want to set it to In Progress?" Wait for confirmation. |
| Any status → `Done` | Check the epic's Section 7 Stories table. If all other stories in this epic are also `Done`, ask: "All stories in EPIC-{NNN} are now Done. Do you want to also update the epic status to Done?" (FW-048) If this story's Walking Skeleton field is `Yes`, check every other `Walking Skeleton: Yes` story across the whole project in `stories/index.md`. If every one is now `Done` (not merely `Merged` — this must be QA-verified, not just code-landed), update that sprint's `Walking Skeleton Complete` field in Section 8 from `No` to `Yes` and report it: "All Walking Skeleton stories are now Done — Walking Skeleton Complete set to Yes for Sprint {NN}." |
| Any status → `Deprecated` | Ask: "Confirm deprecation. This story will be marked Deprecated in the file and index. This does not delete the file. Confirm?" |

---

### Story plan update (`FW-031`)

**In the plan file:**
- Find the `**Plan Status:**` line in the header
- Replace the current value with the new one
- Do not change any other field

There is no index file for story plans to update alongside.

**Additional actions by transition:**

| Transition | Additional action |
|-----------|-----------------|
| `Draft` → `Confirmed` | No rationale prompt required — the plan is a scoped extraction of already-Locked architecture, not a new gate decision on the scale of a BRD or architecture lock. Simply confirm: "Plan for `{Story ID}` set to Confirmed. `/implement-story` is now unblocked for this story." |
| `Confirmed` → `Draft` (re-opening after `/gen-story-plan` regenerated the plan) | This should already have happened automatically when `/gen-story-plan` regenerated the file — if invoked manually instead, just apply it, no additional prompt |

---

### Epic update

**In the epic file:**
- Find the `**Status:**` line in the header (below the Priority line)
- Replace the current value with the new one

**In the epics index (`epics/index.md`):**
- Find the row matching this Epic ID
- Update the Status column value

**Additional actions by transition:**

| Transition | Additional action |
|-----------|-----------------|
| Draft → `Approved` | Confirm and add DC entry — see below |
| Any status → `In Progress` | Check the epic's Section 5 (Dependencies). If any dependency epic is not yet `Done`, warn: "This epic has unresolved epic dependencies: [list]. Are you sure?" Wait for confirmation. |
| Any status → `Done` | Verify all stories in the epic's Section 7 Stories table are `Done` or `Deprecated`. If any are not: "Cannot set epic to Done — the following stories are not yet Done: [list]. Resolve these first." Block the update. |

**Epic approval — Document Control entry:**

On confirmation, add a row to the epic's Document Control table before updating the status field:
```
| — | {today's date} | Product Owner | Approved: epic review complete, story generation unblocked |
```
No rationale prompt required — the review session (`/review-epics`) is the record of what was verified. This entry records the date the gate was formally closed.

---

### Sprint update

**In the stories index (`stories/index.md`):**
- Find the row in Section 8 Sprint Planning table matching Sprint {N}
- Update the **Status column** for that row to the new status value
- If the new status is `Complete`: cross-reference Section 1 Story Registry — expand the Stories column for Sprint {N} (handle ranges like US-01-001 → US-01-009 and comma-separated lists) to get all story IDs in this sprint. If any story in that set is not `Done` or `Deprecated`: "Sprint {N} has stories that are not yet Done: [list]. Mark the sprint Complete anyway?" Wait for confirmation.

**If a sprint plan document exists** at `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md`:
- Also update the `**Status:**` field in that document's header block to match the new status.

---

---

### BRD update

**In the BRD file:**
- Find the `| Status |` row in the Document Control table near the top of the file
- Replace the current status value with the new one
- Do not change any other field

**Additional actions by transition:**

| Transition | Additional action |
|-----------|-----------------|
| Any status → `Approved` | Ask for approval rationale — see guidance below |
| Any status → `Locked` | Confirm: "BRD is being set to Locked. This is a permanent freeze. Confirm?" |

**BRD approval rationale — required before setting status to `Approved`:**

Ask the PO:

> "BRD is being set to Approved. This locks all requirements — no changes without a formal change request via `/assess-change`.
>
> Before I update the status, record your approval rationale in one or two sentences. This is added to the Document Control table as the permanent record of this gate decision.
>
> **What a good rationale states:**
> - Which parts of the BRD were verified (requirement count, sections reviewed, open items resolved)
> - Whether scope matches the client brief and discovery session output
> - Whether any gaps or open items were explicitly resolved or deferred
>
> **Examples:**
> - `All 23 FRs reviewed and accepted. Scope confirmed matches client brief and discovery session output. Section 9 integration list verified against client confirmation. No open items.`
> - `Phase 1 requirements approved. Phase 2 features explicitly deferred in Section 5 Out of Scope. RBAC model confirmed against real user roles — no invented personas. No open items.`
> - `BRD v1.1 approved following change request. FR-017 revised wording confirmed correct. All downstream impact reviewed via /assess-change. No further open items.`
>
> **Not acceptable:** `'Approved'`, `'Looks good'`, `'OK to proceed'` — these record a decision without recording what was verified. Write what you checked, not just the outcome."

After receiving the rationale, add a row to the BRD Document Control table:
```
| — | {today's date} | Product Owner | Approved: {rationale} |
```
Then update the Status field to `Approved`.

---

### Architecture update

**In the architecture file:**
- Find the `| Status |` row in the Document Control table at the top of the file
- Replace the current status value with the new one
- If the new status is `Locked`, also fill in the `| Locked on |` row with today's date (if it is blank)

**Additional actions by transition:**

| Transition | Additional action |
|-----------|-----------------|
| Any status → `Locked` | Ask for lock rationale — see guidance below |

**Architecture lock rationale — required before setting status to `Locked`:**

> Note: if `/review-architecture` was used and the PO typed 'approved', the lock and rationale entry are handled automatically by that command. This manual path applies only when `update-status` is run directly (e.g. after 'approved — do not lock yet').

Ask the PO:

> "Architecture document is being set to Locked. This makes it law — no implementation may deviate from it without a formal change request via `/assess-change`. `/implement-story` and `/gen-sprint-plan` are now unblocked.
>
> Before I lock it, record your rationale in one or two sentences. This is added to the Change History table as the permanent record of this gate decision.
>
> **What a good rationale states:**
> - Which design decisions were specifically verified (modules, state machines, security model, data model)
> - Whether all BRD requirements have covering modules
> - Whether any Section 8 Open Decisions were resolved
>
> **Examples:**
> - `All 16 modules confirmed against BRD scope. Invoice and payment state machines match domain knowledge. Three-JWT security model accepted. Section 8 open decisions all resolved before lock.`
> - `Architecture locked. All 5 BRD integrations have dedicated modules. RBAC table covers all 5 user roles from BRD Section 3. Money columns use multi-currency _amount suffix as agreed. No deviations from standard stack.`
> - `Revised architecture locked after /assess-change for FR-019 (refund flow). New RefundModule reviewed and accepted. No other changes. All downstream stories confirmed consistent.`
>
> **Not acceptable:** `'Locked'`, `'Approved'`, `'Architecture reviewed'` — state what was verified, not just the outcome."

After receiving the rationale, add a row to the architecture Change History table:
```
| — | {today's date} | Product Owner | Locked: {rationale} |
```
Fill in the `Locked on` field with today's date. Then update the Status to `Locked`.

---

### Brief / Project update

`brief.md` has two distinct status fields. Determine which one applies from the value given (the two sets never overlap, per Step 2) — **never write to the wrong field**:

**If the value is a Content Status** (`Draft` / `Under Review` / `Brief complete` / `Approved` / `Locked` / `Superseded`):
- Update **only** the `**Content Status:**` field in `brief.md`
- Do not touch the `**Status:**` field or `projects/index.md`

**If the value is a Status (project lifecycle)** (`Active` / `On Hold` / `Complete` / `Cancelled`):
- Update the `**Status:**` field in `brief.md` **and** the matching row's Status column in `projects/index.md` (if it exists) — both together, same as before
- Do not touch the `**Content Status:**` field

**Additional actions by transition (Status/project-lifecycle values only):**

| Transition | Additional action |
|-----------|-----------------|
| Any status → `On Hold` | Ask: "Why is the project going On Hold? Record the reason in brief Section 8 (Related Artifacts / Notes) for future reference." Record the reason before updating status. |
| Any status → `Cancelled` | Confirm: "Project is being set to Cancelled. All files are preserved. No further work should be done on this project. Confirm?" |

---

## Step 5 — Report the update

After applying all changes:

```
STATUS UPDATED
──────────────────────────────────────────
Artifact:   [Story ID / Epic ID / Sprint N]
Previous:   [old status]
New:        [new status]
Updated in: [file path]
            [index path]

[Any warnings or additional actions taken]
```

If any additional action was confirmed (e.g. epic also set to Done), include it in the report.
