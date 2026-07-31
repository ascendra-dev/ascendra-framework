# FW-040 — Ascendra UI Hard Instructions Registry

| Field | Value |
|-------|-------|
| **ID** | FW-040 |
| **Date** | 2026-07-18 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/implement-story.md`, new `reference/ascendra-ui/hard-instructions.md` |

---

## Decision

A new file, `reference/ascendra-ui/hard-instructions.md`, is a living, appendable registry of corrections to how `ascendra-ui` is actually used in practice — populated by PO manual review of implemented screens and by direct investigation against `../ascendra-ui`'s real component source and real page implementations. `/implement-story` now reads this file (Step 2 item 6, Web implementation rules' Components bullet, and the Standards Compliance Self-Check) *in addition to*, never instead of, `ascendra-ui`'s own docs (`ui-reference.md`/`showcase-reference.md`). Where an entry and a docs template disagree, the entry wins — entries exist specifically to record cases where the docs were wrong or incomplete. The underlying condition is unchanged: this file only applies when the project's confirmed tech stack (`system-architecture.md` Section 2) selected Next.js + Ascendra UI; it does not create a dependency on `ascendra-ui` where none was chosen.

---

## The Problem That Triggered This Decision

Manual PO review of the Web screens built for `US-02-001` and `US-01-003` (`ASCENDRA-PAY-001`) found five recurring UI-implementation gaps, all of the same shape: `/implement-story` either invented an implementation pattern `ascendra-ui` already provides, or mapped a requirement onto the wrong component. Investigating each against `../ascendra-ui`'s real source confirmed all five, plus two more found in the process:

- Raw `<h3>` section headers instead of `FieldLegend` (confirmed: every real form page uses `FieldLegend`, none use a raw heading)
- `FieldError` used for validation display instead of `FieldHint` (confirmed: zero real form pages use `FieldError`, all ten use `FieldHint`)
- `FieldContent` wrapping a field's control — a pattern that exists nowhere in any real form/dialog/sheet/drawer page (only in the isolated primitive-preview page), despite `docs/showcase-reference.md`'s own Template 2/5 code samples showing it — **the docs themselves are wrong here**
- `DialogContent showCloseButton` — not a real prop on the component at all; `docs/ui-reference.md`'s prop table lists it anyway
- The Staff List page's "Add Staff" trigger placed in `PageHeaderAction` instead of the table's own `DataTableBarAction` (confirmed: every real `PageHeaderAction` usage across the whole showcase is on a dashboard page, never a table-list page)
- No evidence `/implement-story` ever explores the full Data Table feature surface (sorting, bulk/row actions, column management, saved/preset queries, pagination) before defaulting to the bare minimum shape
- Dialogs/forms visually "rough" — inconsistent spacing versus real reference pages, from not copying a real page's structural nesting exactly

The root cause matches `FW-033`'s pattern exactly: `/implement-story` had a docs-reading step, but nothing caught cases where those docs were incomplete or actively incorrect, and nothing accumulated PO-found corrections for future runs to benefit from. Unlike `FW-033` (which pointed the command at files that already existed), no file existed yet to hold this class of correction — `standards/` is per-project and generated, and `ascendra-ui/docs/` is the library's own (sometimes-wrong) documentation, not a place for this framework's own corrections.

---

## Why a New Reference File, Not an Edit to `ascendra-ui/docs/`

`ascendra-ui/docs/*.md` is generated/maintained by the `ascendra-ui` project itself and gets wholesale-refreshed on a sync (per the web scaffold's "Keeping Ascendra UI current" procedure in `implement-story.md`) — any correction written directly into a copied `docs/` file would be silently overwritten the next sync. A framework-level file under `reference/` (already the home for "fixed, universal content every project consults directly," per the existing `reference/architecture/` and `reference/estimation/` folders) persists across projects and across `ascendra-ui` syncs, and is never touched by anything outside this framework.

---

## Scope of Application

Applied now: `/implement-story` only (the command that writes the UI code these entries correct). Not yet applied: `/gen-ui-mocks`, which also renders `ascendra-ui`-styled HTML for the approved mock — a natural future extension, since a mock built with the same wrong patterns would get PO-approved before `/implement-story` ever runs, defeating the purpose. Deferred rather than done now because it wasn't the reported failure mode this session — `/gen-ui-mocks`'s own mock output wasn't reviewed here, only `/implement-story`'s real screens.

---

## Implementation Status

Applied immediately during this session:
- `reference/ascendra-ui/hard-instructions.md` created with entries AUI-001 through AUI-007
- `.claude/commands/implement-story.md` edited: Step 2 item 6, Web implementation rules' Components bullet, and the Standards Compliance Self-Check

Not yet applied: retrofitting the five confirmed gaps into the already-implemented `US-02-001`/`US-01-003` Web code — a separate, explicit PO decision, tracked outside this ADR.

---

## Relationship to Other Decisions

Same pattern and root cause as `FW-033` (Standards Compliance in `/implement-story`): a command with a reference-reading step that didn't catch a class of real defect, fixed by pointing it at a source of truth plus a self-check pass. `FW-033` covers project-generated `standards/` files; `FW-040` covers the shared `ascendra-ui` library's own real behavior, which no per-project standards file was ever going to capture.
