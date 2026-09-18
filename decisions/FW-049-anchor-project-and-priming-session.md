# FW-049 — Anchor Project and Priming Session

| Field | Value |
|-------|-------|
| **ID** | FW-049 |
| **Date** | 2026-09-18 |
| **Status** | Decided — implemented, pending validation against a real project |
| **Area** | `.claude/commands/anchor-project.md`, `.claude/commands/run-priming-session.md`, `.claude/commands/init-project.md`, `projects/TEMPLATE/priming/priming-package.template.md` + `.example.md`, `conventions/command-conventions.md` (two exceptions: `C-011`, `C-013`), `observations/index.md` + `TEMPLATE.md`, `CLAUDE.md`, `PROJECT-LIFECYCLE.md`, `practitioner-guide/00-orientation.md` |
| **Depends on** | `FW-048` (Walking Skeleton Milestone) — `/anchor-project`'s Solution Domain handoff trigger is `FW-048`'s `Walking Skeleton Complete` field; this decision would not be checkable without it |

---

## Decision

Two new commands give the framework a second, optional way to run its own lifecycle, without changing how any existing command behaves for a PO who never touches either one.

**`/anchor-project {PROJECT_CODE}`** resolves or bootstraps a project, reconciles its own lightweight state against the real artifacts on every resume (never trusting stale state blindly), determines the current stage the same way `PROJECT-LIFECYCLE.md`'s own "How to Resume a Mid-Project Session" already documents doing by hand, then invokes whichever existing command owns that stage and executes its instructions exactly as written. It adds three things no existing command provides: state continuity across sessions, a compression layer on PO-facing interaction (dense questions and review output made terser without lowering rigor), and cross-artifact drift checking. It drives a project through Problem Domain and Architecture, then through implementation of exactly its Walking Skeleton (`FW-048`), and narrows afterward to release preparation and formal scope changes — day-to-day story implementation across multiple developers is explicitly outside its remit, because its state model is single-session, single-actor by design.

**`/run-priming-session {PROJECT_CODE}`** is a standalone companion command, not a mode of anchor: a free-form, optional, repeatable session — with or without raw material the PO drops into a new `source-material/` project folder — that materializes facts into `priming/priming-package.md`, scoped to Problem Domain only (Brief, Domain, BRD). It gives `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery` a head start; none of them are modified to know it exists. Its own output template is composed from the sections those commands' existing discovery-state templates already define, not a new parallel schema — so it can never drift out of sync with what they actually require.

Neither command is a hard gate. Every command in the framework, including both of these, remains fully usable standalone.

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
- **`FW-024`** (Parking Lot) — `priming-package.template.md`'s Section 9 (Open/Uncategorized) reuses the same discipline: nothing discovered gets dropped just because it doesn't fit a defined field.

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

Not yet done:

- [ ] Validation against a real project — either adopting `THORNFIELD-MAINT-001` mid-flight (exercises the bootstrap/reconciliation path directly, though it predates `FW-048` and has no Walking Skeleton tags) or a fresh second simulation end to end, or both
- [ ] `ANCHOR-PROJECT-DESIGN.md` remains the living design document until validation completes and surfaces nothing further to change; it is not yet marked historical
