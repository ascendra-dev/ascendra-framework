# Sprint Plan Template

> **[AI Guide — Document Level]**
> A sprint plan is a single-sprint kickoff, tracking, and closure record. It is produced by `/gen-sprint-plan` once all of that sprint's epics are Approved, all of its stories are Reviewed, and the architecture is Locked. It is regenerated once per sprint — never for more than one sprint at a time.
>
> **What this document is:**
> - A pre-sprint readiness record — the gate that must show all-clear before `update-status ... sprint {NN} Active` starts the sprint
> - A live tracking aid during the sprint — the Story Board reflects current status as stories move through the delivery pipeline
> - A closure record — `/close-sprint` fills the Retrospective section once UAT is signed off
>
> **What this document is not:**
> - A story repository — stories are individual files in `stories/EPIC-{NNN}/`. This document references them by ID only
> - A multi-sprint plan — this document covers exactly one sprint. Cross-sprint scope and ordering live in the Sprint Roadmap section (Section 2), sourced from the stories index, not duplicated here
> - A formal PO sign-off artefact — there is no Draft → Approved → Locked promotion. The Pre-Sprint Readiness Checklist (Section 3) is the objective gate; `update-status` records the PO's decision to start the sprint
>
> **One sprint at a time:**
> Stories for the sprint's epic(s) must already be Reviewed before this document is generated — sprint assignment itself happens during `/review-stories`, not here. This command only extracts what was already assigned in `stories/index.md` Section 8 and turns it into a kickoff/tracking document.
>
> **File location:**
> `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md` — e.g. `projects/ASCENDRA-PAY-001/sprints/sprint-01.md`. Sprint number is 2-digit zero-padded.

---

## Document Control

> **[AI Guide]** Add a row each time this document is revised (e.g. a re-run after a story is added or descoped before the sprint goes Active). Version 1.0 is the initial generation.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |

---

## Part 1 — Template

---

**Project:** [Project name]
**Sprint:** Sprint [NN]
**Status:** Planning / Active / Complete
**Epic(s):** [EPIC-NNN — Title, EPIC-NNN — Title, ...]
**Stories:** [N] total ([size breakdown, e.g. 2L + 3M + 4S])
**Dates:** [YYYY-MM-DD] → [YYYY-MM-DD], or `TBD`
**Generated:** [Date]

---

### 1. Sprint Goal

> **[AI Guide]** One sentence. What coherent, demonstrable value does this sprint deliver from the Product Owner's perspective? The goal describes an outcome, not a list of features.
>
> Test: could you demo this sprint to a stakeholder in one sentence and have them understand what changed? If not, rewrite.
>
> **Correct:** "Finance Officers can register Payers, configure deduplication settings, and import Payer records from CSV — the Payer management foundation is in place."
>
> **Incorrect:** "Implement US-04-001 through US-04-006" (a task list, not a goal) or "Build the Payer module" (too vague to be demonstrable).
>
> If the user typed `auto`, draft the goal from the epic(s) scope and present it for confirmation before writing the file (see command Step 4).

[Sprint goal statement]

---

### 2. Sprint Roadmap

> **[AI Guide]** Sourced from `stories/index.md` Section 8 (Sprint Planning table) — do not maintain a separate roadmap file. Every epic already has its sprint-assignment status recorded there (assigned to a sprint number, or not yet assigned) by the time this command runs, since assignment happens during `/review-stories`, wave by wave.
>
> **Sprint 1's plan:** embed the full current-state table below — every epic, its assigned sprint (or "Not yet assigned" if its wave hasn't been reached), and status.
> **Sprint 2+ plans:** embed the same table, refreshed to current state at generation time — this reflects what's actually known now, not a fixed original plan. Note beneath the table: "Refreshed from `stories/index.md` Section 8 at generation time — see that file for the live source of truth."

| Epic | Title | Assigned Sprint | Story Count | Status |
|------|-------|-----------------|--------------|--------|
| EPIC-NNN | [Title] | [NN / Not yet assigned] | [N] | Planning / Active / Complete / Not yet assigned |

---

### 3. Pre-Sprint Readiness Checklist

> **[AI Guide]** Results from the six gate checks the command runs before generating this document. Record Pass / Fail / N/A for each. If any gate is Fail, add a resolution note immediately below the table naming exactly what must happen before the sprint can go Active.

| Gate | Result |
|------|--------|
| Architecture Locked | Pass / Fail |
| Epics Approved | Pass / Fail |
| Stories Reviewed | Pass / Fail |
| Dependencies cleared | Pass / Fail / Warn |
| L-story split decision recorded | Pass / Fail / N/A |
| Capacity decision recorded | Pass / Fail / N/A |

[If any Fail: resolution note — what must happen before this sprint can go Active.]

---

### 4. Story Board

> **[AI Guide]** Every story in this sprint, in delivery sequence order (not registry order) — this is the order Section 5 lays out, not the order stories appear in `stories/index.md`. Status is live — this table is the one section expected to change during the sprint as stories move through the pipeline.
>
> If this sprint's epics carry a documented story-level interleaving note in `epics/index.md`'s Dependency Graph (e.g. two epics whose true build order only resolves at the story level), reflect that interleaved order here rather than grouping strictly by epic.

| Order | Story ID | Title | Epic | Size | Status |
|-------|----------|-------|------|------|--------|
| 1 | US-NN-NNN | [Title] | EPIC-NNN | [Size] | [Status] |

---

### 5. Delivery Sequence

> **[AI Guide]** Reformatted from the Notes column in `stories/index.md` Section 8 into a readable step sequence. If an interleaving note applies (see Section 4), state it explicitly here — e.g. "EPIC-001's capability-assignment mechanism (US-01-002) before EPIC-002's signup story (US-02-001), then EPIC-001's remaining staff-management stories."

Step 1 → [Story ID: what it establishes]
Step 2 → [Story ID: what it builds on Step 1]
...

---

### 6. Capacity Analysis

> **[AI Guide]** Size breakdown and Must Have / Should Have split for this sprint only.

| Metric | Value |
|--------|-------|
| Total stories | [N] |
| XS / S / M / L / XL | [N] / [N] / [N] / [N] / [N] |
| Must Have | [N] |
| Should Have | [N] |

---

### 7. Should Have Decisions

> **[AI Guide]** List each Should Have story in this sprint with its inclusion decision. If none, write "None in this sprint."

| Story ID | Title | Decision | Reason |
|----------|-------|----------|--------|
| US-NN-NNN | [Title] | Include / Defer | [one line] |

---

### 8. Definition of Done

> **[AI Guide]** Use these five items exactly — do not add project-specific items here; those belong in `standards/test-strategy.md` if needed.

1. All acceptance criteria pass verification (`/verify-story`)
2. PR reviewed and merged to main branch
3. No open Critical or High severity defects
4. Story status updated to `Done` in `stories/index.md`
5. No regression in previously merged stories for this project

---

### 9. Sprint Notes

> **[AI Guide]** Left blank at generation. Used during the sprint for ad-hoc notes (e.g. QA confirmation record from Sprint Closure). Not populated by `/gen-sprint-plan`.

[Blank at generation]

---

### 10. Sprint Retrospective

> **[AI Guide]** Left blank at generation. Filled by `/close-sprint` after UAT sign-off — do not populate this section here.

[Blank at generation — filled by `/close-sprint`]

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before writing this document.

- [ ] Sprint Goal is a single sentence describing a demonstrable outcome, not a story list
- [ ] Sprint Roadmap reflects the current state of `stories/index.md` Section 8, not a stale or invented plan
- [ ] Pre-Sprint Readiness Checklist has a result for all six gates; any Fail has a resolution note
- [ ] Story Board is in delivery sequence order, not registry order
- [ ] Any documented story-level interleaving note from `epics/index.md` is reflected in the Story Board and Delivery Sequence
- [ ] Capacity Analysis total matches the Story Board row count
- [ ] Every Should Have story in this sprint has a recorded decision
- [ ] Definition of Done uses the five standard items, unmodified
- [ ] Sprint Notes and Sprint Retrospective are left blank, not pre-filled
- [ ] Every story ID follows the `US-{epic}-{seq}` format
