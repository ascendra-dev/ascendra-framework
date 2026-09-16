# FW-048 — Walking Skeleton Milestone

| Field | Value |
|-------|-------|
| **ID** | FW-048 |
| **Date** | 2026-09-17 |
| **Status** | Decided — implemented |
| **Area** | `brd.template.md`, `gen-brd.md`, `run-brd-discovery.md`, `epic.template.md`, `gen-epics.md`, `review-epics.md`, `story.template.md`, `gen-stories.md`, `review-stories.md`, `gen-sprint-plan.md`, `assess-change.md`, `judgment-check.md` (BRD/Epics/Sprint Planning coverage), `practitioner-guide/04-product-structuring-epics.md`, `practitioner-guide/07-sprint-planning.md`, `CLAUDE.md`, `PROJECT-LIFECYCLE.md` |

---

## Decision

Every project's BRD identifies exactly one **Walking Skeleton** — the thinnest possible real, end-to-end user flow that exercises the full intended architecture — as a first-class, structurally tagged element, not free text. A dedicated `Walking Skeleton` column (`Yes` / `—`) is added to the BRD requirements table (Section 5), carried forward unmodified through the epic and story templates, and enforced as a hard sequencing rule at sprint-planning time: Walking-Skeleton-tagged stories are scheduled first, ahead of ordinary wave/priority ordering, and must reach Done/Merged with a passing verification report before any epic is built out to full breadth. `/review-epics`, `/review-stories`, `/assess-change`, and `/judgment-check` each gain a check enforcing this. This formalizes a discipline already proven correct in practice — see below — rather than leaving it as a per-project improvisation with no framework backing.

By default, a project has **exactly one** Walking Skeleton. A second is a signal, not a green light — see "The Convention" below.

---

## Why This Was Needed

A sibling project (Ascendra Pay, a real product built by this framework's own author) surfaced this the hard way. Its first attempt — documented in that project's own `lessons-learned` record — failed for a specific, named reason: *"The core of the system was not established first. Rather than delivering the product core first and adding capability in phases, breadth was built out across many epics before a single thin end-to-end path was proven."* Downstream effects followed directly: stories weren't independent (heavy cross-story coupling meant nothing could be built or verified in isolation), and both authorization and notifications were over-built before there was any proven need for that depth.

The rebuild corrected this by naming one specific, real, thin end-to-end flow explicitly as the Walking Skeleton in the BRD — a Success Criteria bullet, a User Flow section, and inline Scope/REQ annotations — and holding the discipline that nothing else gets built to full breadth until that path is demonstrably working. It worked. But checking this framework's own repository confirms the concept exists nowhere in it — not in `brd.template.md`, not in `gen-epics.md`, not in `gen-sprint-plan.md`, not in the practitioner guide. What corrected the failure was PO discipline layered on top of the framework, expressed as improvised prose (a heading suffix, sentences appended to a Priority column) — not something the framework itself knows to ask for, check, or enforce on the next project that doesn't have a PO who already learned this lesson the hard way.

---

## The Convention

**BRD (Section 5 — Requirements):** every requirements table gains a `Walking Skeleton` column alongside `Priority | Source | Layer`:

```
| ID | Description | Priority | Source | Layer | Walking Skeleton |
```

Value is `Yes` or `—` (dash, not `No` — matches the framework's existing terseness convention elsewhere, e.g. "None identified"). **Rule:** every REQ tagged `Yes`, taken together, must compose exactly one complete, thin, real, end-to-end path — not a fragment of one persona's steps with gaps, and not a checklist of "important" requirements that don't actually chain into a single flow. A BRD naming a *second* Walking Skeleton is not forbidden, but `/gen-brd` must stop and ask the PO to justify it explicitly before proceeding — a second one is far more often a sign that the first wasn't actually kept thin, or that two unrelated concerns got bundled, than a genuine case for two.

**Section 4 (User Journeys):** the specific flow the tagged REQs form is written up as its own labeled journey entry, its heading explicitly suffixed "(Walking Skeleton)" — formalizing the pattern Ascendra Pay already used informally.

**Section 1.3 (Business Goals):** a required bullet naming the Walking Skeleton journey (cross-referencing its Section 4 entry by name) and stating it must be demonstrated working end-to-end before any epic is built out to full breadth — this fits the section's existing "how will we know this project succeeded" framing directly; no new top-level section is needed. (Corrected 2026-09-17 from an earlier draft of this decision that referred to a "Section 1.3 / 7 (Success Criteria)" — Section 7/Success Criteria belongs to `brief.template.md`, a different document; the BRD has no Success Criteria section.)

**Section 13.1 (Content Integrity):** a new check — every `Walking Skeleton: Yes` REQ set composes one complete path; no gaps; at most one Walking Skeleton without an explicit PO-justified exception recorded.

**Epics (`epic.template.md` Section 2 — BRD Requirements Covered):** the column carries forward unchanged:

```
| REQ ID | Description (exact from BRD) | Priority | Walking Skeleton |
```

**`epics/index.md`:** gains a **Walking Skeleton Coverage** table — same tier as the existing REQ Coverage Map and Dependency Graph — listing which epics carry a WS-tagged REQ and confirming the stated build order actually delivers them first.

**`/review-epics`:** new check — build-order reasoning explicitly accounts for Walking Skeleton epics coming before breadth epics, not just the ordinary dependency-graph logic.

**Stories (`story.template.md` header metadata block):** gains a field next to the existing `Layer:` / `Target:` / `Status:` fields:

```
**Walking Skeleton:** Yes / —
```

Populated by `/gen-stories` from the parent REQ's tag. A single REQ can produce multiple stories (per the existing "FR References" rule) — every story descending from a `Yes`-tagged REQ inherits `Yes`.

**`/review-stories`:** new check, the real enforcement point — Walking-Skeleton-tagged stories, taken together, must be buildable and demoable end-to-end **without depending on any non-tagged story**. This is what actually catches Ascendra Pay's original failure mode (breadth stories silently propping up the "thin path") rather than just labeling REQs and hoping the decomposition held the line.

**`stories/index.md`:**
- Story registry table (Section 1) gains a `Walking Skeleton` column: `Story ID, Epic, Title, Layer, Target, Size, Status, Sprint, Walking Skeleton`.
- Sprint plan section (Section 8) gains a `Walking Skeleton Complete: Yes / No` field per relevant sprint, written by `/update-status` and flipping to `Yes` only once every WS-tagged story project-wide reaches `Done` — QA-verified, not merely `Merged`. This is a load-bearing field beyond documentation — it is the exact mechanical signal the in-progress `/anchor-project` design (`ANCHOR-PROJECT-DESIGN.md`) needs to know when its Problem-Domain-through-first-slice involvement is complete and execution can hand off to normal developer-led sprints.

**`/assess-change`:** a change touching a `Walking Skeleton: Yes` REQ or story is flagged for higher scrutiny than an ordinary scope change — it affects the foundational proof-of-architecture milestone, not routine scope. Not a new formality tier; a named flag inside the existing patterns.

**`/judgment-check`:** the BRD, Epics, and Sprint Planning chapters gain a specific test operationalizing the actual root cause named above — is breadth being built before a thin path is proven. This is what makes the lesson checkable on every future project, not something a PO has to already know to ask for.

---

## Practitioner Guide

`practitioner-guide/04-product-structuring-epics.md` and `07-sprint-planning.md` each gain a worked passage teaching this discipline. This ADR's rationale above cites Ascendra Pay directly, as evidence — but the guide's worked examples should stay inside the guide's existing continuity (every `.example.md` across the framework already threads one fictional case, Harborview Consulting Ltd's Invoice Management System, per the practitioner-guide audit's own key finding). Rather than import a second, real, named client project into that continuity, the guide passage should construct a Harborview-consistent illustration of the same discipline (e.g. a thin Invoice-creation-through-payment path as Harborview's own Walking Skeleton) — keeping the ADR as the place real evidence lives, and the guide as the place the teaching example lives, consistent with how every other chapter in it already works.

---

## Relationship to Other Decisions

- **FW-024** (Parking Lot) — structural precedent for adding a new table/column cleanly to `brd.template.md` without disturbing existing sections.
- **FW-026** (Screen Design Phase) — structural precedent for a decision that threads one concept through multiple templates and commands rather than a single-file patch; this decision follows the same shape.
- **`ANCHOR-PROJECT-DESIGN.md`** (in-progress, not yet a decision) — §9 (Solution Domain Boundary — The Walking Skeleton Handoff) now references the real `Walking Skeleton Complete: Yes/No` field in `stories/index.md` Section 8 as the actual, mechanical handoff trigger, replacing the placeholder language an earlier draft had.

---

## Full Implementation Scope

Not yet applied — tracked here for the implementation pass:

- [x] `projects/TEMPLATE/brds/brd.template.md` — add `Walking Skeleton` column to Section 5 requirements tables; add Section 4 User Journey labeling guidance; add Section 1.3 Business Goals bullet requirement; add Section 13.1 check
- [x] `projects/TEMPLATE/brds/brd.example.md` — matching worked example added: Journey 4.4 (Finance Manager / Payer — Create, Send, and Collect Payment), 7 REQs tagged Yes (REQ-001, 006-009, 014-015), Section 1.3 goal bullet, Section 13.1 check entry
- [x] `.claude/commands/gen-brd.md` — generation guidance added for the column, the Section 4 journey label, the Section 1.3 goal bullet, and the "stop and ask if a second Walking Skeleton appears" rule
- [x] `.claude/commands/run-brd-discovery.md` — discovery-session guidance added for identifying the Walking Skeleton with the PO after journey walkthroughs
- [x] *(added, not in original scope list)* `projects/TEMPLATE/brds/brd-discovery-state.template.md` — added a Walking Skeleton field under Section 6 (Journey Walkthroughs) so there's a real place to record the PO's answer between discovery and `/gen-brd`
- [x] `projects/TEMPLATE/epics/epic.template.md` — `Walking Skeleton` column added to Section 2
- [x] `.claude/commands/gen-epics.md` — tag carried forward in Section 2; Walking Skeleton Coverage table and build-order rule added to Step 6
- [x] `.claude/commands/review-epics.md` — check 14 (Walking Skeleton Build Order) added
- [x] `projects/TEMPLATE/stories/story.template.md` — `Walking Skeleton:` header field and a Part 2 verification item added
- [x] `.claude/commands/gen-stories.md` — field populated from parent epic's Section 2; column added to Step 6 stories-index registry table spec; checklist count corrected 15→16
- [x] `.claude/commands/review-stories.md` — check 11 (per-story tag accuracy) and check 7 (epic-level Walking Skeleton Independence — the real enforcement point) added
- [x] *(corrected during implementation)* The WS-first sequencing rule does not actually live in `gen-sprint-plan.md` — that command only extracts a sprint assignment that must already exist. The real decision point is `.claude/commands/review-stories.md` Part F (Sprint Assignment), where a note was added confirming ordering is inherited from `epics/index.md`'s build order and flagging a PO request to skip epic order as something to confirm, not silently follow.
- [x] `.claude/commands/gen-sprint-plan.md` — Story Board data-fill instructions updated to carry the Walking Skeleton column and status line
- [x] `projects/TEMPLATE/sprints/sprint-plan.template.md` — Walking Skeleton column added to Section 4 (Story Board) with a status line convention; Part 2 verification item added *(added, not in original scope list — the actual per-sprint document a PO reads needed this, not just `stories/index.md`)*
- [x] *(added, not in original scope list)* `.claude/commands/update-status.md` — the `Walking Skeleton Complete: Yes/No` field itself is written here, on the "Any status → Done" transition: once every `Walking Skeleton: Yes` story project-wide reaches `Done` (QA-verified, not merely `Merged`), the field flips from `No` to `Yes` in `stories/index.md` Section 8. This is the actual mechanical trigger anchor's design depends on.
- [x] `.claude/commands/assess-change.md` — Walking Skeleton flag added to Step 4's blast-radius output; explicitly does not introduce a new formality tier
- [x] *(corrected during implementation)* `.claude/commands/judgment-check.md` needs no direct edit — it sources its tests entirely from the matching `practitioner-guide/` chapter per its own design, so the two chapter edits below are what "adding the test" actually means. Also corrected: stories are covered by Chapter 7 (Sprint Planning), not Chapter 4, per `judgment-check.md`'s own artifact-type table.
- [x] `practitioner-guide/04-product-structuring-epics.md` — new section "The Walking Skeleton: a harder constraint than ordinary dependency ordering," a Common Mistakes bullet, and a Harborview-in-practice addition citing `epic.example.md`'s real REQ-001 tag and the EPIC-001/EPIC-003 split
- [x] `practitioner-guide/07-sprint-planning.md` — new section "Walking Skeleton Independence: the check that actually enforces FW-048," a Common Mistakes bullet, and a Harborview-in-practice addition citing the three real worked stories in `story.example.md` (all three happen to be Walking Skeleton stories)
- [x] *(added, not in original scope list)* `projects/TEMPLATE/epics/epic.example.md` and `projects/TEMPLATE/stories/story.example.md` — Walking Skeleton column/field added so the practitioner-guide citations above trace to real, updated files, not stale ones
- [x] *(caught on a follow-up pass — initially missed)* `projects/TEMPLATE/brds/brd-discovery-state.example.md` and `projects/TEMPLATE/sprints/sprint-plan.example.md` — every template that gained a Walking Skeleton field/column now has its matching `.example.md` updated too; none were left showing the pre-FW-048 shape
- [x] *(reviewed, no change needed)* `CLAUDE.md`, `PROJECT-LIFECYCLE.md` — both describe the pipeline at a level that doesn't enumerate individual template columns or check counts (confirmed no other FW-XXX is cited inline in either file either, so this isn't a gap specific to this decision) — nothing in either file became inaccurate
- [x] `ANCHOR-PROJECT-DESIGN.md` — §9/§11 updated to reference the real `Walking Skeleton Complete` field
