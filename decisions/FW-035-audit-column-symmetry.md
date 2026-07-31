# FW-035 — Audit Column Symmetry (`updated_by`/`deleted_by`)

| Field | Value |
|-------|-------|
| **ID** | FW-035 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md` |

---

## Decision

`/gen-architecture`'s `database-standards.md` generation rubric now enumerates the standard master-table columns explicitly instead of the vague "standard columns required on every master table." The full audit trio is now the default: `created_at`/`updated_at`/`deleted_at` **and their actor counterparts** `created_by`/`updated_by`/`deleted_by` — a row's complete write history (when and by whom, for creation, every update, and every deletion), not just creation. An actor column may be omitted, but only with a stated reason, never silently. Detail tables get the same principle scoped to what applies: immutable detail rows correctly omit the `updated_*`/`deleted_*` pair (nothing ever updates or deletes them), but still get `created_at`/`created_by` — a gap this decision also closes, since `ASCENDRA-PAY-001`'s own `database-standards.md` Rule 5 example omitted `created_by` from detail tables entirely with no stated reason.

---

## The Problem That Triggered This Decision

The PO noticed `ASCENDRA-PAY-001`'s tables consistently carry `created_at`+`created_by` but only `updated_at`/`deleted_at` — no `updated_by`/`deleted_by` counterparts anywhere. Confirmed by direct inspection: `database-standards.md` Rule 4 mentions `createdBy` exactly once; `arch-v1.md` has `createdBy:` on all 20 master-table occurrences and **zero** occurrences of `updatedBy:` or `deletedBy:` anywhere in the document.

This was not an `/implement-story` execution gap (unlike `FW-033`'s findings) — `/implement-story` built exactly what `database-standards.md` specified. The gap is one level up: `gen-architecture.md`'s own rubric for generating `database-standards.md` never enumerates what "standard columns" actually means — it's two bare bullets ("Standard columns required on every master table" / "...every detail table") left entirely to whichever run's own judgment call. On this project, that judgment landed on an asymmetric trio: an actor column for creation, but none for update or delete.

`deleted_by`'s absence is the clearer case: a soft-delete is always a deliberate, meaningful action, and this project carries an explicit 7-year audit retention requirement (BRD Section 10) with a general compliance/regulatory posture (Section 9 Security Requirements, REQ-058/SEC-004 detailed activity logging elsewhere) — "who deleted this record" is a basic audit question with currently no answer anywhere in the schema. `updated_by` is a real but softer case: this project already relies on domain-specific actor columns for the actions that carry business meaning (`assignedBy`, `approvedBy`/`recordedBy`, `reviewedBy`, `requestedBy`/`authorizedBy`) — a generic `updated_by` is a technical safety net on top of those, valuable for "who touched this row, for any reason" but not a substitute for them, and it cannot be trigger-automated the way `updated_at` is (a DB trigger has no visibility into the application's current-user context without session-variable/RLS machinery this project doesn't use) — every future service method issuing an UPDATE must now explicitly pass the current actor's id. The PO confirmed both columns are wanted despite that ongoing cost, for full audit symmetry.

---

## What Changed in `gen-architecture.md`

- **Step 7, `database-standards.md` rubric:** the two vague "standard columns" bullets replaced with an explicit enumeration — `id`, tenant/org identifier, `version`, the full `created_at`/`updated_at`/`deleted_at` + `created_by`/`updated_by`/`deleted_by` set for master tables (omittable only with a stated reason); the same principle for detail tables, explicitly correcting for the immutable-detail-table case (omit `updated_*`/`deleted_*` with that stated reason, but keep `created_at`/`created_by`).

---

## Why This Was Needed (and why now, not deferred)

Same shape as `FW-033`/`FW-034`: a rubric left open to a single generating run's judgment call produces an inconsistency that then propagates identically into every table `/implement-story` builds against it — 20 tables deep before anyone noticed, in this case. Left unfixed, every future project would face the same one-off judgment call, with no guarantee of landing on a complete answer. Enumerating the default explicitly doesn't remove judgment where it's actually needed (a project can still state a reason to omit a column) — it removes the silent, accidental omission that happened here.

---

## Implementation Status

Applied immediately during this session — `.claude/commands/gen-architecture.md` edited directly. The retrofit for `ASCENDRA-PAY-001` itself (its own `database-standards.md`, `arch-v1.md`, and a migration adding the columns to its 5 existing tables) is tracked separately as project work via `/assess-change`, not part of this framework-level decision.

---

## Relationship to Other Decisions

Same investigation family as `FW-033`/`FW-034` (both 2026-07-17) — a rubric or reading gap found via real usage of `ASCENDRA-PAY-001`, fixed at the framework level so it doesn't recur on the next project.
