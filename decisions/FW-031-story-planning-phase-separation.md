# FW-031 — Story Planning Phase Separation

| Field | Value |
|-------|-------|
| **ID** | FW-031 |
| **Date** | 2026-07-15 |
| **Status** | Decided — implemented |
| **Area** | New: `gen-story-plan.md`, `story-plan.template.md`. Changed: `implement-story.md`, `verify-story.md`, `update-status.md`, `init-project.md`, `CLAUDE.md`, `PROJECT-LIFECYCLE.md`, `SDLC.md` |
| **Extends** | `FW-029`, `FW-030` |

---

## Decision

`/implement-story` previously went straight from gate-checks to writing code, with no artifact the Product Owner could review *before* code existed — the one phase in the framework that didn't honor the core philosophy "every document produced by AI is reviewed before the next step starts." This decision splits that single command's two responsibilities across two commands:

- **`/gen-story-plan`** (new) — owns the **planning** responsibility: extracting, from the already-Locked architecture and the story's own ACs, exactly which endpoints/tables/screens this one story needs, in what order, and how each AC will be tested. Output is a PO-reviewable, persisted artifact: `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md`.
- **`/implement-story`** (trimmed) — owns the **execution** responsibility only: given a Confirmed plan, build exactly what it declares, using this project's confirmed stack idiom (unchanged from `FW-029`/`FW-030`).
- **`/verify-story`** (trimmed) — reads its Test Scenarios from the plan (Section 4) instead of deriving them after the fact, and adds a Plan Conformance check (declared vs. delivered) alongside the existing AC pass/fail.

**`/implement-story` hard-gates on the plan being `Confirmed`** — no PO override, unlike the softer dependency/Open-Decision gates it already had. A plan is cheap enough to produce and review that there is no legitimate case for skipping it.

---

## What actually moves, and what doesn't

The split is **extraction-vs-invention**, not **planning-vs-no-planning** — most of what a story plan contains was already decided, at `/gen-architecture` time, and Locked. `/gen-story-plan` does not re-decide architecture; it scopes the already-Locked architecture down to one story and makes the narrow set of decisions that genuinely aren't pinned down anywhere else:

| Moves to `/gen-story-plan` (the *what*) | Stays in `/implement-story` (the *how*) |
|---|---|
| Which of the architecture's already-defined endpoints/tables/screens this story touches | NestJS/Next.js (or whatever stack is confirmed) decorator syntax, query style, file-path conventions — unchanged `FW-029` content |
| AC → artifact traceability | The actual file writes, scaffolding, branch/commit mechanics |
| Build order across artifacts | Bootstrap-if-missing repo logic (`FW-030`'s structural check) |
| Per-AC test scenario (approach, input, expected result) | Running lint/build/test, fixing failures |
| Flagging genuine ambiguity (an AC with no clean artifact, an artifact the architecture doesn't define) | — |

If the architecture doesn't already define a shape a story needs, `/gen-story-plan` does not invent one — it flags the gap in its own Section 5 (Risks / Open Questions) rather than let the two commands drift out of agreement on what the "real" design is.

**Deviation handling:** implementation sometimes has to diverge from a plan (a technical constraint the plan-writer didn't anticipate). Rather than silently drifting or treating any deviation as an automatic failure, the plan template carries a Section 6 (Deviations from Plan), mirroring the pattern `arch.template.md` already uses for Section 9 (Deviation Declarations) — `/implement-story` records what changed and why; `/verify-story`'s Plan Conformance check treats a documented deviation as normal engineering judgment and an undocumented one as a defect.

---

## Rationale

Raised 2026-07-15 during a critical review of `implement-story.md` and `verify-story.md`: neither command gave the PO a reviewable checkpoint before code was written, in contrast to every other phase of the framework (BRD, epics, screen design, architecture, stories all have a distinct gen-then-review pair). The PO explicitly requested visibility into *what* a story would produce before implementation started — pages, endpoints, queries, DTOs, guards — and a way to verify against that expectation afterward, which is exactly what a persisted, PO-confirmed plan provides.

Gate strictness was a deliberate PO call, not a framework default: hard-block, no override — consistent with how BRD/architecture/epics gates already work, and justified here because a plan is materially cheaper to produce than a BRD or architecture document, so there's no legitimate "not worth it for this story" case the way there sometimes is for the dependency-cleared or Open-Decision gates (which do allow override).

---

## What changes

- **New:** `projects/TEMPLATE/story-plans/story-plan.template.md`, `projects/TEMPLATE/story-plans/story-plan.example.md`, `.claude/commands/gen-story-plan.md`
- `implement-story.md`: new hard gate (Step 1) requiring a Confirmed plan; Step 2 reframed as *how*-reference now that the plan states *what*; Step 3's target-repo determination cross-checks against the plan instead of re-deriving; Step 4 reads the plan as its task list and records deviations in the plan's Section 7; Step 6 report references plan artifacts built and any deviations
- `verify-story.md`: Step 3 reads the plan file; Step 4 consumes the plan's Test Scenarios table (falls back to on-the-fly derivation only if no plan exists — pre-`FW-031` stories); report gains a Plan Conformance section
- `update-status.md`: new artifact type ("story plan"), statuses `Draft`/`Confirmed`, lightweight confirm-only transition (no rationale prompt, unlike BRD/architecture locks)
- `init-project.md`: `story-plans/` added to the project folder skeleton
- `CLAUDE.md`, `PROJECT-LIFECYCLE.md`, `SDLC.md`: command count 29 → 30; new Step 17a in the lifecycle table and Stage 8 description; Story Plan added to the Status Values Reference

### Correction pass (same day, before first real use)

A PO review of the initial build caught three gaps before any story was planned against it:

1. **Missing `Worker` coverage.** The template and command initially had only API Plan / Web Plan sections — a story targeting `Worker` (scheduled/async jobs) had nowhere to go. Added Section 4 (Worker Plan) to the template — job type/trigger, schema touched, business logic, idempotency, observability, predicted files, extracted from the worker's own Section 3.1 subsection, same never-invent rule as API/Web — and a matching Step 6 in `gen-story-plan.md`. This shifted Test Scenarios to Section 5 and Deviations from Plan to Section 7; every cross-reference to those section numbers in `implement-story.md` and `verify-story.md` was updated to match (they reference the *generated plan's* section numbers directly, per `command-conventions.md` C-013a — commands other than `gen-*`/`init-*` must not read template files, so these numbers have to be kept in sync by hand whenever the template's structure changes).
2. **Missing `.example.md`.** `FW-003` establishes that every deliverable-artifact template gets a co-located worked example — checked against all fourteen other template folders, all but the free-form `standards/` templates have one. `story-plan.example.md` was missing. Added, reusing the same fictional Harborview Consulting Ltd domain `stories/story.example.md` already uses, for continuity — one full API+Web example (built from that file's own "Create invoice form" story) plus a short Worker Plan excerpt.
3. **Convention non-compliance in `gen-story-plan.md` itself.** Checked against `conventions/command-conventions.md` and found two violations: gate checks were embedded in Step 1 instead of a dedicated `Step 1.5 — Gate check` (required for `gen-*` commands per C-011); and there was no pre-write planning-summary confirmation before the file was written (required by C-015, and already the pattern in `gen-stories.md` Step 4). Restructured to match: Step 1 (parse only) → Step 1.5 (gate check, table format) → Steps 2–8 (read, build) → Step 9 (present summary, wait for confirmation) → Step 10 (write, still `Draft`) → Step 11 (report).

## When adding a future command in this category

Any command that currently combines "decide what to build" and "build it" in one step should be evaluated against this same split: can the *what* be extracted from an already-Locked, already-reviewed upstream artifact and presented for a cheap PO check before the *how* runs? If yes, prefer two commands over one, following the extraction-vs-invention principle above — the new command scopes and previews, it does not re-decide what's already been decided upstream.
