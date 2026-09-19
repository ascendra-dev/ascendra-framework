# FW-028 — Artifact Naming Conventions

| Field | Value |
|-------|-------|
| **ID** | FW-028 |
| **Date** | 2026-07-13 |
| **Status** | Decided |
| **Area** | Framework-wide — every generated artifact's ID format and filename pattern |

---

## Decision

A consolidated, authoritative reference for every ID format (Project Code, Epic ID, Story ID, REQ ID, Defect ID) and generated-file naming pattern (one row per artifact type, across all lifecycle phases) is established at [`conventions/artifact-naming-conventions.md`](../conventions/artifact-naming-conventions.md). That file is the authoritative reference; this record captures why it was created.

This does not replace the existing requirement ([`template-conventions.md`](../conventions/template-conventions.md) T-001) that every template states its own file-location pattern locally, in its own document-level AI Guide. The new file is a cross-check reference, not a new place commands read from — a template's local statement remains the one a generating command actually follows.

---

## Rationale

Before this decision, each artifact's naming pattern existed in up to three scattered places — the template's own AI Guide, the generating command's Output path, and [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md)'s Output line — with nothing tying them together. This was found to be a real, recurring source of bugs, not a theoretical risk. In one review pass (2026-07-13) three separate naming-drift issues were found this way, each only caught by manually grepping across files:

1. Story IDs were originally specified as sprint-keyed (`US-{sprint}-{seq}`) even though `/gen-stories` runs before sprint assignment exists — fixed to epic-keyed (`US-{epic}-{seq}`), stable at generation time.
2. `sprint-plan.template.md` documented an output filename (`sprint-{NN}-plan.md`) that didn't match what `gen-sprint-plan.md` actually wrote (`sprint-{NN}.md`) — the two had fully diverged.
3. `story.template.md`'s own Story ID header field displayed `PRJ-{PROJECT_CODE}-US-[epic-number]-[sequence]` — a form used by zero commands and zero real generated files anywhere in the framework, with the actually-used short form relegated to a footnote.

Each of these was a template or command internally self-consistent enough to look correct in isolation; the drift only became visible when cross-checked against every other place the same artifact was named. A single consolidated reference makes that cross-check a lookup instead of a grep sweep, and gives any future new artifact type an obvious place to register its pattern and check it against what already exists.

---

## What changes

- New file: [`conventions/artifact-naming-conventions.md`](../conventions/artifact-naming-conventions.md) — ID formats (Section 1); every generated file naming pattern across the full lifecycle, including cross-reference/tracking artifacts easy to overlook (`epics/index.md`, `stories/index.md`, `epics/review-record.md`, `stories/review-record.md`) and the one genuinely global artifact, `projects/index.md` (Sections 2, 2a); and the four scope keys that key a filename — per-story, per-sprint, per-version, and fixed-name-once (Section 3).
- `story.template.md`'s Story ID field simplified to the short form actually used everywhere (`US-{epic}-{seq}`) — the unused `PRJ-{PROJECT_CODE}-...` long form removed.
- `gen-brd-playbook.md` had an internal inconsistency (`brds/discovery-state.md` vs `brds/brd-discovery-state.md` in two different sections of the same file) — corrected to match every other command's usage.
- `release-notes.template.md` was missing the "File location" AI Guide line every other template of its kind has — added.

## When adding a new artifact type

Register its ID format and/or filename pattern in [`conventions/artifact-naming-conventions.md`](../conventions/artifact-naming-conventions.md) before or alongside building the command that generates it. Check it against the existing scope keys (Section 3) rather than defaulting to "per sprint."
