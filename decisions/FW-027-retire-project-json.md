# FW-027 — Retire `project.json`

| Field | Value |
|-------|-------|
| **ID** | FW-027 |
| **Date** | 2026-07-13 |
| **Status** | Decided |
| **Amends** | `FW-006` (superseded — see below) |
| **Area** | `project.json`, `/init-project`, `/implement-story`, `/gen-pr-description`, `CLAUDE.md`, `projects/index.md` |

---

## Decision

`project.json` is removed entirely. `/init-project` no longer creates it. Its five identity fields (`projectCode`, `projectName`, `client`, `domain`, `created`) are fully covered by `projects/index.md`'s registry row for the project — a `Created` column is added there to close the one gap that existed.

`FW-006` (project.json Is Identity Only) is superseded by this decision — not because its reasoning was wrong, but because the file it governs never acquired a real consumer.

---

## Rationale

A pass through every command file (2026-07-13) found `project.json` referenced in exactly three places:
- `/init-project` — writes it, once, at project creation
- `/implement-story` and `/gen-pr-description` — mention it only to say repo paths come from the architecture document, *not* from `project.json`

No command reads it back for any functional decision. Every command derives `PROJECT_CODE` directly from the file path argument it's given, never by reading `project.json` to "identify" the project — the stated purpose in `FW-006` ("a stable, lightweight way to identify which project [commands are] working on") never actually materialized in practice.

The one thing that was ever meant to live in `project.json` beyond identity — the `composes` field for project composition (`FW-010`) — was already removed and relocated to the brief's Extension Context fields by `FW-016`. After that change, every remaining field in `project.json` was pure duplication of what `projects/index.md` already records for the same project, except `created`, which had no column there.

Keeping a write-only file with zero readers is worse than not having it: it's one more artifact a new command author might assume they should read from, or one more place identity data could silently drift from `projects/index.md`.

---

## What changes

| Before | After |
|--------|-------|
| `/init-project` creates `projects/{PROJECT_CODE}/project.json` | No longer created |
| Identity fields split across `project.json` and `projects/index.md` | Single source of truth: `projects/index.md`, now with a `Created` column |
| `/implement-story`, `/gen-pr-description` note "not from `project.json`" | Dangling reference removed — nothing to disclaim once the file doesn't exist |
| `FW-006` describes `project.json`'s schema | Marked superseded; schema fully absorbed into `projects/index.md` |

---

## Also corrected while retiring this

Two other decision docs still described the already-dead `composes`-in-`project.json` model (`FW-010`, superseded by `FW-016`) as if it were live:
- `FW-014` (Subdomain Decomposition) — referenced "the project composition model (`composes` field in `project.json`)" for cross-phase dependency
- `FW-015` (Framework Design Strategy) — referenced "the `composes` field in `project.json`" for cross-project domain knowledge reuse

Both corrected to point at `FW-016`'s brief-driven extension model instead. `FW-007` (Artifact Ownership Boundaries) had a table row for `project.json` — removed.

---

## Migration

Existing projects with a `project.json` file (e.g. `ASCENDRA-PAY-001`) have it deleted; nothing reads it, so nothing breaks. Its `created` value is carried into `projects/index.md`'s new `Created` column before deletion, so no information is lost.
