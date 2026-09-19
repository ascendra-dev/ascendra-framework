# FW-049 — Anchor Project and Priming Session

| Field | Value |
|-------|-------|
| **ID** | FW-049 |
| **Date** | 2026-09-18 |
| **Status** | Decided — implemented, pending validation against a real project |
| **Area** | `.claude/commands/anchor-project.md`, `.claude/commands/run-priming-session.md`, `.claude/commands/init-project.md`, `projects/TEMPLATE/priming/priming-package.template.md` + `.example.md`, `conventions/command-conventions.md` (two exceptions: `C-011`, `C-013`), `conventions/priming-command-conventions.md`, `observations/index.md` + `TEMPLATE.md`, `CLAUDE.md`, `PROJECT-LIFECYCLE.md`, `practitioner-guide/00-orientation.md` |
| **Depends on** | `FW-048` (Walking Skeleton Milestone) — `/anchor-project`'s Solution Domain handoff trigger is `FW-048`'s `Walking Skeleton Complete` field; this decision would not be checkable without it |
| **Amended by** | `FW-050` (Project Journal and Framework Learning) — replaces this decision's original self-improvement logging mechanism (Step 11, the summary in "Key Mechanisms" below) with a two-layer journal + extraction model. The four "Amended 2026-09-19" sections below (priming-package sections and flags) are unaffected and remain current. |

---

## Decision

Two new commands give the framework a second, optional way to run its own lifecycle, without changing how any existing command behaves for a PO who never touches either one.

**`/anchor-project {PROJECT_CODE}`** resolves or bootstraps a project, reconciles its own lightweight state against the real artifacts on every resume (never trusting stale state blindly), determines the current stage the same way `PROJECT-LIFECYCLE.md`'s own "How to Resume a Mid-Project Session" already documents doing by hand, then invokes whichever existing command owns that stage and executes its instructions exactly as written. It adds three things no existing command provides: state continuity across sessions, a compression layer on PO-facing interaction (dense questions and review output made terser without lowering rigor), and cross-artifact drift checking. It drives a project through Problem Domain and Architecture, then through implementation of exactly its Walking Skeleton (`FW-048`), and narrows afterward to release preparation and formal scope changes — day-to-day story implementation across multiple developers is explicitly outside its remit, because its state model is single-session, single-actor by design.

**`/run-priming-session {PROJECT_CODE}`** is a standalone companion command, not a mode of anchor: a free-form, optional, repeatable session — with or without raw material the PO drops into a new `source-material/` project folder — that materializes facts into `priming/priming-package.md`, scoped to Problem Domain only (Brief, Domain, BRD). It gives `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery` a head start; none of them are modified to know it exists. Its own output template is composed from the sections those commands' existing discovery-state templates already define, not a new parallel schema — so it can never drift out of sync with what they actually require.

Neither command is a hard gate. Every command in the framework, including both of these, remains fully usable standalone.

---

## Amended 2026-09-19 — Source-Specific Sections

`priming-package.template.md`'s Sections 4-6 mirror `brief.template.md` / `domain-discovery-state.template.md` / `brd-discovery-state.template.md` so the package can never drift out of sync with what those commands expect (§5.1's original reasoning, unchanged). That held for the one command that existed at the time — prose only (an SRS, raw notes, live conversation), per §5.11's "one command per mechanic" principle. The gap: a future mechanic-specific command (starting with the still-deferred legacy-code-priming command §5.11 already names) will surface facts that don't compress into Brief/Domain/BRD vocabulary — an existing API surface, schema-as-implemented, business rules embedded in validation logic, a tech-debt inventory. The only place for those before this amendment was Section 9 (Open/Uncategorized), meant for genuine one-off leftovers, not a whole expected output category from a class of source material.

**Resolution:** a new `conventions/priming-command-conventions.md` (rule prefix `PC-0NN`) governs every priming-producing command uniformly — today's `/run-priming-session` and any future one — covering shared-spine mechanics, mapping priority (Brief/Domain/BRD first, always), and a new, formally sanctioned **Source-Specific Section** mechanism: `priming-package.template.md`'s new Section 10 (Source-Specific Material), where a mechanic-specific command may contribute its own attributed subsection for facts that are real but genuinely don't map onto the existing taxonomy. `/run-priming-session` itself owns no subsection there today — everything it extracts has a home in Sections 4-6.

**Naming note:** deliberately not called "extension" anything — that word already governs a different concept in this framework (Core/Extension methodology's architecture-level technical seams, `arch.template.md` §3.2). "Source-Specific Section" was chosen instead, reusing vocabulary already load-bearing in this exact template (`Source Material Index`, the `source-material/` folder).

Sections 10-11 of the original template renumbered to 11-12 as a result (Resume Instructions, Pre-Handoff Verification).

---

## Amended 2026-09-19 — Conflicting Source Flags

Tracing how a conflict *within* source material (not just between the priming package and the PO) gets raised and resolved surfaced three gaps: Section 4's Flag guidance mentioned "contradiction" but Sections 5-6 and Pre-Handoff Verification's typing rule (scoped to Sections 5-7 only) didn't, leaving Section 4 silently exempt; no recognized Flag type actually fit "two sources disagree" (`AI Knowledge Correction`/`Legacy Design Question` both trigger on divergence from generic domain understanding, not on two provided facts disagreeing); and the package's Status and Flag fields are independent, so a topic could read `Covered` while carrying an unresolved conflict — `/anchor-project`'s pre-fill logic checks for a known answer in a way that in practice reads Status, risking a flagged topic being silently substituted without the PO ever seeing the disagreement.

**Resolution**, kept origin-agnostic — the same mechanism handles a conflict between two places in one source file, two different files, or the package and something the PO says live: a new recognized Flag type `Conflicting Source`, defined once at the document level rather than per-section; a designed comparison step in Triage/Extract that checks new content against what a topic already holds for disagreement, not only redundancy; both (or all) conflicting statements always recorded in full with origin attribution, never silently dropped or preferred; a live PO present at detection time is asked directly rather than deferred to a flag; and resolution reuses the existing decomposition routing — a flagged Sections 4-6 topic surfaces as a live question when its owning phase command reaches it, exactly like a `Pending` topic already does. The closing guarantee: any unresolved Flag, of any type, disqualifies a topic from `/anchor-project`'s pre-fill regardless of its Status field.

Full rule set: `conventions/priming-command-conventions.md` Section 6 (`PC-050`-`PC-054`); design rationale: `ANCHOR-PROJECT-DESIGN.md` §5.16.

---

## Amended 2026-09-19 — Section 7/8/9 Consumption Gap, and Two Smaller Fixes

A follow-up review of `anchor-project.md`'s own decomposition logic (rather than a new priming-package feature) found that it reads exactly two things out of the priming package — Sections 4-6 (pre-filled into a phase command's fields) and Section 10 (handed along as background material, per PC-041). Sections 7 (Terms As Used), 8 (Source Material Index), and 9 (Open/Uncategorized) are all written by `/run-priming-session` but were never read by anything downstream, including the package's own Resume Instructions — a fact captured in any of the three could be recorded once and never surface again.

The first candidate fix — auto-writing leftover Section 9 rows into the BRD's Parking Lot (`FW-024` Section 15) at `/gen-brd` time — was rejected: `FW-024` itself states Parking Lot entries are "not [added] during discovery-driven authoring," the exact moment that fix would fire, and a Parking Lot entry is supposed to be a specific, deliberate idea a human raised, not raw un-triaged material no one has vetted yet.

**Resolution, kept in two parts:**
- Sections 7 and 8 are reference material like Section 10 — never pre-filled, handed along as background material instead. Section 8 goes to whichever phase is running; Section 7 goes specifically to `/run-domain-discovery`, the one phase where a flagged possible-synonym is actually actionable.
- Section 9 is unresolved content, not reference material — each row surfaces as a live question at the phase named in its own "where it probably belongs" column, exactly like a genuinely `Pending` topic. It is never pre-filled, never handed along silently, and never auto-written into any artifact (Parking Lot included) by anchor itself — only through that live session's own ordinary judgment, same as any other Parking Lot entry.
- Section 11 (Resume Instructions) gained a third list (unresolved Section 9 rows) alongside its existing Pending-topics and Flags lists.

Two smaller, unrelated fixes landed in the same pass: Step 3's Reference Material Index cited "Section 3" as the source for a per-topic Pending count that section doesn't actually hold (corrected to scan Sections 4-6's own Status lines); and Step 7's pre-fill logic described only a binary Covered/Pending split, silently omitting the `Partial` status PC-011 already makes canonical (corrected so a `Partial` topic surfaces as an informed live question — stating what's known, asking what's missing — never silently treated as complete).

Full rule set: `conventions/priming-command-conventions.md` PC-042/PC-043; design rationale: `ANCHOR-PROJECT-DESIGN.md` §5.17.

---

## Amended 2026-09-19 — "Out of Scope" Was Silently Swallowing New Facts

Walking through a real mid-project scenario — Brief and Domain already Approved/Locked, BRD still Draft, new source material dropped in — surfaced a fourth gap. `/run-priming-session`'s Step 3 says an already-Approved/Locked topic is "out of scope for this session — do not re-ask about it," meant to stop it wasting the PO's time re-confirming settled things. Read literally, it doesn't distinguish that from "ignore a new fact that shows up on its own" — so a genuinely new fact touching a locked topic (a new integration the Domain doc never captured, say) could be silently dropped before it ever reached the constraint-propagation drift check that exists specifically to catch a fact recorded after an artifact was approved. The check only fires on facts that got written down somewhere first.

**Resolution:** "out of scope" now explicitly governs asking, not noticing. A fact touching an already-Approved/Locked topic still gets recorded — into Section 9 (Open/Uncategorized), but with "Where it probably belongs" naming the affected artifact and its status (e.g. "Domain — already Locked, needs `/assess-change`") rather than a phase to raise a live question in. `/anchor-project`'s Section 9 handling recognizes this phrasing and routes straight to `/assess-change`, since there's no open discovery session left to raise it in.

Full rule set: `conventions/priming-command-conventions.md` PC-015; design rationale: `ANCHOR-PROJECT-DESIGN.md` §5.18.

---

## Why This Was Needed

Running a project through this framework end to end means the PO carries the entire sequence in their head — which of ~30 commands to run next, what state each artifact is in, how to answer each command's own dense discovery/review questions, repeated at every one of twelve stages. Nothing in the framework addressed the sequencing burden itself, only the quality of what happens at each individual stage once the PO gets there.

The design that resulted — captured in full in `ANCHOR-PROJECT-DESIGN.md` at the repo root, which remains the authoritative reference for every mechanism this decision only summarizes — went through several corrections worth recording here because they shaped the final shape:

1. **The interaction layer had to be additive, not a rewrite.** An early direction proposed editing roughly 20 existing commands to adopt a shared "lighter interaction" convention. Rejected: it would have violated commands-stay-frozen and created a second place PO-interaction logic could drift from itself. The resolved mechanism — anchor invokes a command and executes its own instructions unmodified, intercepting only the single moment a PO-facing question gets composed — needed no command file touched at all.
2. **Document ingestion was originally embedded inside `/anchor-project` itself**, framed as a mode. Applying the framework's own composability principle to the finished design surfaced a real contradiction: ingestion is source-agnostic by its own stated design, but it was welded into the one command that isn't source-agnostic — anchor is a sequencer. Every future input mechanic (legacy code, named explicitly as the next one, deferred) would have meant editing `anchor-project.md` again instead of adding a command. `/run-priming-session` is the correction — one command per extraction mechanic, not per input format, composable with anchor or run standalone.
3. **The Solution Domain boundary needed a real, checkable trigger, not a judgment call anchor would have to eyeball each time.** `FW-048`'s Walking Skeleton milestone — itself informed by a documented prior failure on a sibling project (breadth built before a single thin path was proven working) — supplied it: `stories/index.md`'s `Walking Skeleton Complete` field is the actual mechanical handoff signal.

---

## Key Mechanisms (summarized — see `ANCHOR-PROJECT-DESIGN.md` for full detail)

- **State is derived, never authoritative.** `.anchor-state.md` is rewritten and reconciled against real artifacts every resume; a missing or stale state file is a normal, fully-supported bootstrap case, not an error — this is what makes adopting an already-in-progress, non-anchor-driven project a first-class case.
- **PO interaction protocol:** compression without lowering the PO's assumed capability, a bounded (~3 question) draft-and-correct loop, fidelity confirmation scaled to whether an answer is binding, and a standing escape hatch back to full detail on request.
- **Priming session density and scope discipline:** every captured entry matches discovery-state-file density (a short "PO stated" distillation), never final-artifact density; 100% coverage is never the bar — gaps are marked `Pending` and the real discovery sessions still run in full afterward; no Ubiquitous Language enforcement at this stage, since no domain glossary exists yet to enforce against — terms are captured as used, apparent synonyms flagged, never normalized.
- **Reproducibility** across repeat priming sessions is handled the same way `/run-domain-discovery` already handles it for itself — structural completeness (a Covered/Pending flag and a Pre-Handoff Verification checklist) rather than identical wording, since an adaptive session can't be made deterministic.
- **Anti-drift is anchor's own distinct contribution** — cross-artifact coherence checks (version-lineage drift, constraint-propagation drift, orphaned open items, run cheaply on every resume; traceability and Ubiquitous Language consistency at gate-close) that nothing else in the pipeline runs proactively today.
- **Self-improvement logging** (`observations/`, new root-level folder, separate from `decisions/`) captures both anchor's own friction and command-level friction found while anchor wears another command's hat — with an explicit, still-placeholder prompt encouraging a PO to report a finding back to the framework author, since `observations/` lives in each PO's own clone and doesn't reach the author automatically.

---

## Conventions Exceptions (proposed centrally, per `command-conventions.md`'s own governing principle)

- **`C-011`** (gate-check placement) — a cross-cutting orchestration command that invokes other commands rather than performing one artifact's own generation/review/session work follows session-command placement (`## Step 2 — Gate check`).
- **`C-013`** (file reading) — such a command's file needs are inherently sequence-dependent and may be distributed across the steps that actually need them, rather than consolidated into one dedicated reading step.

Both added to `command-conventions.md` itself, not improvised inside `anchor-project.md`.

---

## Relationship to Other Decisions

- **`FW-048`** (Walking Skeleton Milestone) — the mechanical trigger this decision's Solution Domain boundary depends on; implemented first, deliberately, so this decision would have something real to check rather than a placeholder.
- **`FW-020`** (Command Conventions) — both new commands were authored against and self-checked against its verification checklist; the two exceptions above were added under its own stated process for doing so.
- **`FW-024`** (Parking Lot) — `priming-package.template.md`'s Section 9 (Open/Uncategorized) reuses the same discipline: nothing discovered gets dropped just because it doesn't fit a defined field. Section 9 stays scoped to genuine one-off leftovers; the 2026-09-19 amendment's Section 10 (Source-Specific Material) is a distinct mechanism for an *expected* category of fact from a given source mechanic — the two are not interchangeable, see `conventions/priming-command-conventions.md` PC-021.

---

## Implementation Status

Built and self-checked, 2026-09-17/18:

- [x] `.claude/commands/anchor-project.md` — self-checked against `command-conventions.md`; one real structural bug (a step referencing another "before" it in sequence) caught and fixed during that check
- [x] `.claude/commands/run-priming-session.md`
- [x] `projects/TEMPLATE/priming/priming-package.template.md` + `.example.md` (Harborview-consistent, deliberately partial to demonstrate the coverage calibration)
- [x] `.claude/commands/init-project.md` — `source-material/` and `priming/` folders added to its skeleton
- [x] `conventions/command-conventions.md` — `C-011`/`C-013` exceptions
- [x] `observations/index.md` + `TEMPLATE.md` — scaffolding `run-priming-session`'s logging depends on
- [x] `CLAUDE.md`, `PROJECT-LIFECYCLE.md`, `practitioner-guide/00-orientation.md` — reflected, including several stale pre-existing references caught and fixed in passing (a `judgment-check.md` rollout claim, a `judgment-check.md` omission from `PROJECT-LIFECYCLE.md`'s own Completeness Check table)

Amended 2026-09-19 — built and self-checked:

- [x] `conventions/priming-command-conventions.md` — new file, `PC-0NN` rules
- [x] `projects/TEMPLATE/priming/priming-package.template.md` — Section 10 (Source-Specific Material) inserted, Sections 10-11 renumbered to 11-12, Pre-Handoff Verification checklist updated
- [x] `projects/TEMPLATE/priming/priming-package.example.md` — renumbered, Section 10 shown at its "None" default
- [x] `.claude/commands/run-priming-session.md` — mapping-priority note added, section-number references updated (including one pre-existing stale reference to the old Section 10 in Step 3, caught in passing)
- [x] `.claude/commands/anchor-project.md` — Step 7 decomposition note for Section 10 added
- [x] `ANCHOR-PROJECT-DESIGN.md` — §5.11 updated, new §5.15 added, §5.13's stale section skeleton fixed to match the current 12-section shape

Amended 2026-09-19 (second pass, same day) — Conflicting Source Flags, built and self-checked:

- [x] `conventions/priming-command-conventions.md` — PC-011 extended, new Section 6 (`PC-050`-`PC-054`), checklist updated
- [x] `projects/TEMPLATE/priming/priming-package.template.md` — canonical Flag-type list consolidated at document level, Section 4's flag line pointed at it, Pre-Handoff Verification's typing rule extended from Sections 5-7 to 4-7, pre-fill non-substitution guarantee added to the document-level "When reading this file" note
- [x] `projects/TEMPLATE/priming/priming-package.example.md` — concrete `Conflicting Source` example added (VAT rate, voice-note transcript vs. `old-invoice-tracker.xlsx`)
- [x] `.claude/commands/run-priming-session.md` — Step 5a Triage/Extract extended with disagreement detection and a new "Conflicting facts" paragraph; Step 5b extended for the live-PO case; Step 8 report line updated
- [x] `.claude/commands/anchor-project.md` — Step 6 and Step 7 both extended with the non-substitution guarantee and the `Conflicting Source` live-question handling
- [x] `ANCHOR-PROJECT-DESIGN.md` — §5.5 extended, new §5.16 added, status line and Document History updated

Amended 2026-09-19 (third pass, same day) — Section 7/8/9 Consumption Gap and two smaller fixes, built and self-checked:

- [x] `conventions/priming-command-conventions.md` — PC-042 (Sections 7/8 as background material) and PC-043 (Section 9 as live-question content) added to Section 5, checklist updated
- [x] `projects/TEMPLATE/priming/priming-package.template.md` — document-level "When reading this file" note extended to cover Sections 7/8/9; Section 11 (Resume Instructions) gained a third list for unresolved Section 9 rows
- [x] `projects/TEMPLATE/priming/priming-package.example.md` — Section 11 updated to match: PU-001 moved out of the Flags list (it was never a Flag) into its own new Open/Uncategorized list
- [x] `.claude/commands/anchor-project.md` — Step 3 point 6's Reference Material Index fixed to cite the correct source for its Pending-topic count (not Section 3); Step 7 extended with explicit Section 7/8 handoff, explicit Section 9 live-question surfacing, and explicit `Partial`-status handling (previously only a binary Covered/Pending split)
- [x] `ANCHOR-PROJECT-DESIGN.md` — new §5.17 added, status line and Document History updated

Amended 2026-09-19 (fourth pass, same day) — "Out of Scope" Was Silently Swallowing New Facts, built and self-checked:

- [x] `conventions/priming-command-conventions.md` — PC-015 added to Section 2 (Shared Spine Contract), checklist updated
- [x] `.claude/commands/run-priming-session.md` — Step 3's "out of scope" line clarified (asking vs. noticing); Step 5a and Step 5b both extended so a fact touching a locked topic gets recorded into Section 9 rather than dropped
- [x] `projects/TEMPLATE/priming/priming-package.template.md` — Section 9's AI Guide extended to distinguish this case from an ordinary leftover, with the "Domain — already Locked, needs `/assess-change`" phrasing
- [x] `.claude/commands/anchor-project.md` — Section 9 handling extended to recognize that phrasing and route to `/assess-change` instead of a live-discovery question
- [x] `ANCHOR-PROJECT-DESIGN.md` — new §5.18 added, status line and Document History updated

Not yet done:

- [ ] Validation against a real project — either adopting `THORNFIELD-MAINT-001` mid-flight (exercises the bootstrap/reconciliation path directly, though it predates `FW-048` and has no Walking Skeleton tags) or a fresh second simulation end to end, or both
- [ ] `ANCHOR-PROJECT-DESIGN.md` remains the living design document until validation completes and surfaces nothing further to change; it is not yet marked historical
