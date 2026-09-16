# Review Stories

You are running the story review for an Ascendra project. This review happens after all stories for one or more epics are generated and before sprint lock. No story enters a sprint for development without passing this review.

The review is a sequential walkthrough — one story at a time, within one epic at a time. For each story: structural checks run, the story is framed as what it will actually build so the PO can confirm it matches intent, then FLAGS are surfaced and fixed. Changes are applied immediately on agreement.

Progress is recorded in `projects/{PROJECT_CODE}/stories/review-record.md` — the single source of truth for review state and the permanent change record. It enables the review to be stopped and resumed across sessions without losing progress or requiring the PO to recall where things stand.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Locate project and run gate checks

Resolve `{PROJECT_CODE}` from `$ARGUMENTS`. If not provided, ask:
> "Which project are these stories being reviewed for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

**Gate check — BRD must be Approved:**

Read the most recent `brd-core-v*.md` in `projects/{PROJECT_CODE}/brds/`. Check the `**Status:**` field.
- If `Approved` or `Locked` → proceed
- If not → stop: > "The BRD is not yet Approved. Complete `/review-brd` before reviewing stories."

**Gate check — all epics must be Approved:**

Read `projects/{PROJECT_CODE}/epics/index.md`. Check the Status column for every epic.
- If all epics are `Approved` or beyond → proceed
- If any are `Draft` → stop: > "The following epics are not yet Approved: [list]. Run `/review-epics projects/{PROJECT_CODE}` first."

**Gate check — architecture must be Locked:**

Check `projects/{PROJECT_CODE}/architecture/arch-v1.md`.
- If not found → stop: > "No architecture document found. Run `/gen-architecture {PROJECT_CODE}` before reviewing stories."
- If found but Status is not `Locked` → stop: > "Architecture is not yet Locked. Run `/review-architecture projects/{PROJECT_CODE}` first."

**Gate check — stories must exist:**

Check `projects/{PROJECT_CODE}/stories/index.md` and the Section 7 Stories tables in the epic files.
- If no stories exist for any epic → stop: > "No stories found. Run `/gen-stories` for at least one epic first."

---

## Step 2 — Read shared context

Read these files in full before starting the walkthrough. Loaded once and held for the entire review session.

1. The BRD (already read in Step 1 — load any sections not yet in context)
2. `projects/{PROJECT_CODE}/epics/index.md` — REQ coverage map, dependency order, epic list
3. Domain knowledge file(s) referenced in the BRD or found in `projects/{PROJECT_CODE}/domain/`
4. `projects/{PROJECT_CODE}/standards/estimation.md` — the project sizing standard needed for story Check 8 (Size Validity)
5. `projects/{PROJECT_CODE}/stories/index.md` — story registry, sprint plan, Should Have register, L-flag register (read if it exists)

Do **not** load individual epic or story files here — epics are loaded one at a time at the start of each epic's walkthrough; stories are loaded one at a time during the per-story walkthrough.

---

## Step 3 — Gather inputs

Check `$ARGUMENTS` and the files loaded in Step 2. If any of the following are missing, ask for them **all at once** before doing any work:

| Input | What to ask if missing |
|-------|----------------------|
| **Layer** | "Which layer is being reviewed? (Core / Ext:XX / other)" |
| **Key decisions** | "Were any story-generation decisions made that I must preserve? Examples: In Scope bullets collapsed into another story's ACs, domain rules intentionally excluded because a different story owns them, persona choices that differ from the obvious choice, sprint capacity splits. List them or type 'none'." |

The review sequence is derived from the dependency order in `epics/index.md` — producer epics before consumer epics. If you need a different order, state it now. Otherwise the review begins immediately after this step.

---

## Step 4 — Review Record: initialise or resume

Check for `projects/{PROJECT_CODE}/stories/review-record.md`.

---

**Fresh session (review-record.md does not exist):**

Create `projects/{PROJECT_CODE}/stories/review-record.md` with the structure below. Pre-populate the Epic Log from the index in dependency order, and the Story Log with every story listed in each epic's Section 7 Stories table, all set to `Pending`.

```markdown
# Story Review Record — {PROJECT_CODE}

**Review started:** {today's date}
**Review status:** In Progress
**Layer:** {layer}
**Key decisions:** {key decisions from Step 3, or "None"}

## Epic Log

| Epic | Title | Total Stories | Reviewed | Flags | Status |
|------|-------|--------------|---------|-------|--------|
| EPIC-001 | [Title] | [N] | 0 | 0 | Pending |

## Story Log

| Story | Epic | Title | Status | Changes Made | Flags Raised |
|-------|------|-------|--------|-------------|-------------|
| US-01-001 | EPIC-001 | [Title] | Pending | — | 0 |

## Flags

| # | Story | Epic | Issue | Status |
|---|-------|------|-------|--------|

## Review Summary

*(Filled on completion)*
```

Proceed to Step 5 starting from the first epic.

---

**Resumed session (review-record.md exists, status `In Progress`):**

Read the Epic Log to find the first epic with status `Pending` or `In Progress`. Within that epic, read the Story Log to find the first story with status `Pending`. That is the resume point.

> "Resuming story review for {PROJECT_CODE}.
> Progress so far: [N] epics complete, [N] stories reviewed, [N] pending.
> Picking up at [EPIC-ID]: [Epic Title] → [US-XX-XXX]: [Story Title]."

Jump to Step 5 at the resume point.

---

**Review already complete (review-record.md exists, status `Complete`):**

> "All stories have already been reviewed per `stories/review-record.md`. Run `/gen-sprint-plan {PROJECT_CODE}` if sprint planning has not yet begun."

---

## Step 5 — Per-epic, per-story walkthrough

For each epic in sequence (starting from the resume point if resuming):

---

### A — Load epic context

Read this epic's file only. Do not load other epic files. The shared context from Step 2 (BRD, index, domain knowledge) is already loaded.

---

### B — Per-story walkthrough

For each story in this epic's Section 7 Stories table, in order (starting from the resume point within the epic if resuming):

**Load the story:**
Read this story's file only. Do not load other story files.

**Run structural checks:**

Run all 10 checks against this story using the shared context and the loaded epic file. Apply key decisions from Step 3 — do not flag items that were explicitly decided during story generation.

1. **Bullet-to-Story Mapping** — This story maps to exactly one In Scope bullet from epic Section 3.1. Section 8 Notes overrides take precedence if stated explicitly. Flag if no bullet maps to this story, or if two bullets both point to this story (collapse may be undocumented).
2. **Story Statement Quality** — Exactly one persona (from BRD Section 3, or valid `developer`/`platform` exception for technical stories), specific action, specific outcome. No compound story statement ("and" joining two distinct actions is a compound — flag it).
3. **FR Traceability** — All REQ-IDs listed in story Section 2 exist in the approved BRD and belong to this epic per the index coverage map. No REQ from a different epic. No invented REQ-IDs.
4. **AC Completeness** — ACs cover: (a) happy path end-to-end, (b) boundary or rejection case, (c) enforcement or idempotency where the domain knowledge or BRD specifies a once-only, permanent, or cannot-be-undone operation, (d) audit or observability entry for financial state changes or privileged actions.
5. **AC Quality** — Each AC is independently testable. No implementation detail (no field names, API routes, table names, library names, SQL statements). No compound ACs — one condition per AC. Each AC starts with "Given…" or "When…" or a clear precondition.
6. **Domain Rule Coverage** — All UBRs, VALs, BRs, and SEC requirements from the domain knowledge document and the epic's Section 8 Notes that govern this story's capability are covered in the ACs. A domain rule that constrains this capability and has no corresponding AC is a gap.
7. **Out of Scope Integrity** — Every OOS item names a specific story ID (US-XX-XXX), a phase, or an architectural exclusion. "To be handled later" without a named home is not acceptable. Epic-level references (EPIC-XXX) in OOS are acceptable only if story IDs do not yet exist for those items.
8. **Size Validity** — Size declared in the story header is consistent with the AC count per the project estimation standard (`projects/{PROJECT_CODE}/standards/estimation.md`). XL stories must be split before sprint lock. L stories require a PO split decision recorded in the review record before approval.
9. **Dependency Correctness** — All dependencies in story Section 6 reference valid existing story IDs (US-XX-XXX format). No epic-level references remaining. No circular dependencies. No missing dependency where this story reads data or invokes a capability that another story produces.
10. **Should Have / Layer / Target / Verification** — Should Have tag correctly applied per `stories/index.md` register; Layer field matches the parent epic's layer; Target field is populated, names only repo roles declared in architecture Section 3.1 (Project Structure), and is consistent with what the ACs describe (a story whose ACs describe a screen but whose Target says only `API` is a flag, and vice versa); Verification checklist is present and all items are ticked.
11. **Walking Skeleton Tag Accuracy** (FW-048) — the story header's Walking Skeleton field matches the tag on the REQ it references (Section 2) in the parent epic's own Section 2 table, exactly. Never re-derived at story level.

**Real-world framing:**

Before presenting FLAGS, classify what this story will produce and present it in plain PO language. Wait for PO confirmation before surfacing FLAGS.

> "**[US-XX-XXX]: [Story Title]**
> When this story is implemented: [classify and describe in 2–3 sentences what will exist — what the user sees, what the system does, what constraint is enforced]
>
> Does this match what you intended for this story?"

Classification guide for the framing:

| Output type | How to frame it |
|-------------|----------------|
| UI screen or form | "The [persona] will see: [describe the form or screen, the key fields, and what happens on submit or action]." |
| REST endpoint | "This creates: [METHOD /path] — accepts [inputs]. Returns [output]. Called when [trigger]." |
| Background job | "This runs automatically: triggered by [event]. It [action] and [outcome]. No user interaction required." |
| Business rule enforcement | "This adds a constraint: [rule]. The system [enforces it as]. The user sees [message or block] when the rule is violated." |
| Database schema | "This creates: [entity name] storage with [key attributes]. Used by [dependent capabilities]." |
| Combined | Describe each output type separately in the order a developer would build them. |

Wait for PO confirmation. If the PO's description of intent differs from the framing, surface it as a FLAG — the scope or ACs need correction before proceeding.

**Present FLAGS and apply fixes:**

```
FLAGS — [US-XX-XXX]: [Story Title]
[Numbered list. Each flag: check number, section in the story file, precise issue, fix tier.]
[If none: "No flags — all checks passed."]
```

**Fix tiers:**

**Tier 1 — Apply immediately, no pause:**
- Typo in story title, section heading, or AC text
- Part 2 checklist item unticked that clearly passes
- Cross-reference error (wrong REQ-ID, wrong epic ID, wrong story ID in OOS or dependencies)
- Should Have tag missing when the story is clearly in the register

**Tier 2 — Present and confirm before applying:**
- AC rewrite (missing happy path, missing enforcement AC, missing audit AC, implementation detail removal)
- OOS item rewrite (too vague, or missing named home)
- Story statement reword (persona correction, action or outcome clarification)
- Domain rule addition to Section 8 / Notes
- Missing or incorrect dependency addition

**Tier 3 — PO decides before executing:**
- Story covers two In Scope bullets → PO states which ACs stay in each; create two stories and remove the original
- Two stories both claim the same bullet → PO decides which absorbs the other; merge and remove the original
- Story is XL → PO states the split boundary; create two stories and remove the original
- In Scope bullet with no story → PO confirms a new story is needed; draft it and confirm before writing

**Tier 4 — Halt:**
- Multiple ACs missing across a story (story is not reviewable — rewrite and re-run `/gen-stories` for this epic)
- Story covers a capability not in the epic's In Scope at all (scope was added outside the review — address at epic level before continuing)

**Clarification window:**

After FLAGS are addressed, ask:
> "Any questions about what this story covers or excludes? I'll answer from the ACs — if the answer isn't there, that's a gap we fix now."

Wait for PO response. Apply any AC additions or OOS additions raised. If the PO raises a question that has no answer in the current ACs, that is a new FLAG — surface it and apply the appropriate tier fix before closing the story.

**Update story in review record:**

Update `stories/review-record.md` Story Log:
- Status → `Reviewed`
- Changes Made → brief note per change applied (or `—` if none)
- Flags Raised → count (or `0`)

> "[US-XX-XXX] reviewed. Moving to next story."

---

### C — Epic-level checks

After all stories in this epic are walked through, run the 6 epic-level checks:

1. **Complete In Scope Coverage** — Every In Scope bullet in epic Section 3.1 has at least one story covering it. Account for Section 8 Notes collapses (multiple bullets mapped to one story is valid if documented). Flag any bullet with no story and no documented collapse.
2. **No Scope Duplication** — No two stories both claim the same In Scope bullet as their primary scope without a documented collapse justification.
3. **No AC Duplication** — No acceptance criterion appears verbatim or near-verbatim in two different stories within this epic. Shared preconditions (Given clauses) are acceptable if the consequent is different.
4. **Dependency Chain Coherence** — The intra-epic dependency chain declared in story Section 6 is consistent with the delivery sequence in `stories/index.md`. A story cannot depend on a story that runs in a later sprint.
5. **Sprint Capacity** — flag only; the actual sprint-boundary decision happens in the new Part F below, once this epic's real story count is known. This check just confirms Part F has not been skipped for a previously reviewed epic still showing "TBD" in `stories/index.md`.
6. **Cross-epic data dependency** — For each story that consumes data produced by another epic (e.g. `payer_id`, `invoice_amount`, a computed status), verify that an upstream story in the referenced epic has an AC that explicitly produces that data. Flag if this epic depends on data that no upstream story's ACs produce.
7. **Walking Skeleton Independence** (FW-048) — for every `Walking Skeleton: Yes` story in this epic, every dependency listed in its Section 6 (intra-epic or cross-epic) is itself a `Yes`-tagged story. This is the real enforcement point for the whole convention: a Walking Skeleton story silently depending on a non-Walking-Skeleton story means the "thin, real, end-to-end path" isn't actually thin — some breadth story is propping it up. Flag any violation by name; do not wave it through as "practically fine."

If epic-level FLAGS are raised, work through them with the PO. Apply agreed changes to the relevant story files and update `stories/index.md` if story counts or IDs change.

---

### D — Targeted PO questions

After the epic-level checks, ask **1–2 questions** that structural analysis cannot answer. Tailor them to this epic's specific risk profile:

> "[Ask one or two of the following, selecting the most relevant to this epic:]
>
> - Is there any acceptance criterion that would let a developer declare 'done' but still leave something the client actually needs undelivered?
> - Are there edge cases from the discovery session that aren't covered by any story's ACs in this epic?
> - For any L-sized story flagged above: do you know exactly where the split boundary sits?
> - The dependency chain for this epic — does it match the order you would actually demo these capabilities to the client?"

Wait for PO response. Apply any changes raised.

---

### E — Update epic in review record

After the PO confirms the epic:

Update `stories/review-record.md` Epic Log:
- Status → `Reviewed`
- Stories Reviewed → total count
- Flags → total flags raised across all stories and epic-level checks

---

### F — Sprint Assignment

No prior step in this pipeline decides sprint boundaries — `/gen-stories` deliberately leaves the Sprint column `TBD`, and `/gen-sprint-plan` only extracts an assignment that must already exist. This is the step that makes the decision, using this epic's now-known real story count.

**Walking Skeleton ordering (FW-048):** this is enforced upstream by `epics/index.md`'s build order (Walking Skeleton epics sequenced first, per `/gen-epics` Step 6 and `/review-epics` check 14) — as long as epics are worked through `/gen-stories`/`/review-stories` in that build order, Walking Skeleton stories land in the earliest sprint automatically. Treat a PO request to review a later epic out of that build order as a flag to raise, not silently follow: confirm whether skipping ahead is intentional (e.g. an unrelated fast-track) before it displaces Walking Skeleton stories from the sprint that's actually being filled first.

**If this epic carries a documented story-level interleaving note** (checked in Step 2 against `epics/index.md` Section 3 — e.g. two epics whose true build order only resolves at the story level): do not run Sprint Assignment until **both** epics in the pair have completed Parts A–E. Combine their story counts before proceeding.

**Compute the running total** for this wave: this epic's (or interleaved pair's) story count, plus any story count already accumulated earlier in this same review session without yet being assigned to a sprint.

Compare against the sprint capacity target in `projects/{PROJECT_CODE}/standards/estimation.md` (8–10 stories for T-Shirt Sizing).

- **At or near target:** Assign the next sprint number to every unassigned story accumulated so far this session. Write the assignment into `stories/index.md` Section 8, replacing `TBD`/`Not yet planned` with the sprint number for these stories. State it:
  > "EPIC-{NNN} brings this wave to [N] stories — within sprint capacity. Assigning to Sprint {NN}. Confirm before I write this to the stories index."
  Wait for PO response.

- **Below target:** Check `epics/index.md` Section 3 for the next epic in dependency order with no unmet story-level dependency (nothing it needs is still unwritten). If one exists, ask:
  > "This wave is [N] stories — below the [8–10] capacity target. EPIC-{NNN} ({next epic title}) is next in dependency order and appears unblocked. Generate its stories now and fold it into this same sprint before assigning, or accept a lighter Sprint {NN} at [N] stories?"
  Wait for PO response. If continuing: stop this walkthrough here, run `/gen-stories` for that epic, then resume Step 5 at Part A for it — Sprint Assignment runs again once it completes. If accepting a lighter sprint: also check for any earlier-epic leftover stories already `Reviewed` but still unassigned (e.g. remainder from a prior oversized-epic split) that could top up capacity — surface them explicitly rather than silently combining:
  > "EPIC-{NNN} also has [N] Reviewed, unassigned stories left over from an earlier split. Include them in Sprint {NN} too?"
  Wait for PO response, then assign as agreed.

- **Above target (this epic alone exceeds capacity):** Ask the PO for a split boundary along this epic's own intra-epic delivery order (Section 6 dependencies already establish this order):
  > "EPIC-{NNN} has [N] stories — above the [8–10] capacity target on its own. Where should the split fall? Stories before the boundary go to Sprint {NN}; the remainder carries forward as the start of Sprint {NN+1}."
  Wait for PO response. Assign the first slice to Sprint {NN} now in `stories/index.md`; leave the remainder `TBD` — it will be picked up by Sprint Assignment again once the next wave begins (it does not need its own `/gen-stories` run; the stories already exist).

**Write the decision:** Update `stories/index.md` Section 8 (Sprint plan section) with the sprint number, epic(s), story IDs, and count for every story assigned in this step. Update the Sprint column in Section 1 (Story Registry) for each assigned story from `TBD` to the sprint number.

> "EPIC-{NNN} complete. [N] stories reviewed, [N] flags raised. [Sprint assignment outcome — assigned to Sprint {NN} / continuing into next epic / split recorded]. Moving to [next EPIC-ID]: [Title]."

---

## Step 6 — After all epics in this session are reviewed

**Resolve open flags:**

Check the Flags table in `stories/review-record.md`. If any flags are `Open`:

> "The following items were not resolved during the walkthrough:
> [List each open flag: story, epic, issue]
> These must be resolved before any story can be approved for sprint lock."

Work through each with the PO. Apply agreed changes to the relevant story files. Update the Flags table — set Status to `Resolved` as each is closed.

**Update story statuses and cross-references:**

Once all flags are resolved:

1. Update `**Status:**` in each reviewed story file from `Draft` to `Reviewed`
2. Update the Status column in `projects/{PROJECT_CODE}/stories/index.md` for all reviewed stories
3. For any Tier 3 structural changes (splits, merges, new stories): update the affected epic's Section 7 Stories table and `stories/index.md` to reflect the new story IDs, revised sprint plan, and updated story count

**Complete the review record:**

Update `stories/review-record.md`:
- `**Review status:**` → `Complete`
- Add `**Review completed:** {today's date}`
- Fill the Review Summary:

```markdown
## Review Summary

**Completed:** {today's date}
**Epics reviewed:** [N]
**Stories reviewed:** [N]
**Approved without changes:** [N]
**Approved with changes:** [N] — [list which stories and one-line summary of what changed]
**Structural changes (Tier 3):** [N] — [list splits, merges, new stories — or "None"]
**Flags raised:** [N total] — all resolved before approval
```

**Report to the user:**

```
STORY REVIEW COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Layer:              {layer}
Epics reviewed:     [N]
Stories reviewed:   [N]
Approved clean:     [N]
Changes applied:    [N]
Structural changes: [N] (splits / merges / new stories)
Sprint(s) assigned: [Sprint {NN}: N stories / Sprint {NN}: N stories / — none, all below capacity and held for next wave]
Record:             projects/{PROJECT_CODE}/stories/review-record.md
─────────────────────────────────────────
All stories reviewed in this session are Reviewed and sprint-assigned per Part F above.
Next step: Run /gen-sprint-plan {PROJECT_CODE} {NN} for each sprint number assigned above.
If any epic remains for a future wave, run /gen-stories on it only when that wave is reached —
not now.
```
