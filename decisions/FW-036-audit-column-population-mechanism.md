# FW-036 — Audit Column Population Mechanism

| Field | Value |
|-------|-------|
| **ID** | FW-036 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md`, `.claude/commands/implement-story.md` |
| **Amends** | FW-035 — Audit Column Symmetry |

---

## Decision

`FW-035` fixed *what* audit columns exist (`created_by`/`updated_by`/`deleted_by` alongside their timestamp counterparts). This decision fixes the gap directly adjacent to it: neither `gen-architecture.md` nor `implement-story.md` ever stated *how* those columns actually get populated. Both now state it explicitly: there is no ORM-level or database-level mechanism that fills these in automatically (no lifecycle hooks, and a trigger cannot see the application's current-user context) — every insert/update/soft-delete must set the relevant actor column by hand, threading the acting user's id from the verified session through controller → service → repository. One named exception: an entity's own "self-signup" flow, where the actor's own record doesn't exist yet when the transaction starts — pre-generate that record's id and self-reference it (the same pattern already used for a self-referential tenant/org id), never derived from the identity provider's own user id, which does not satisfy the actor foreign key.

---

## The Problem That Triggered This Decision

The PO asked directly: how are the various `_by` columns actually being populated — automatically at the ORM level, or by hand? Checking the real code (`onboarding.controller.ts`, `onboarding.service.ts`, `onboarding.repository.ts`) rather than answering from memory: `createdBy` was `NULL` on every single row `US-02-001` had ever created, including the Owner's own signup — where the actor is completely unambiguous and already verified by `SupabaseAuthGuard`. The controller never called `@CurrentUser()`, the service signature carried no actor id, and none of the four inserts in `createIssuerWithDefaults` set `createdBy`.

Root cause, same shape as `FW-033`/`FW-034`/`FW-035`: `database-standards.md` Rule 4 (even after `FW-035`) stated that the actor columns must exist, but never stated the mechanism for filling them in. No standards file, and no implementation-rule bullet in `implement-story.md`, said "thread the current user through explicitly" — so the story was built exactly to the (still incomplete) letter of the standard.

A second, genuine subtlety surfaced while designing the fix, not just the gap itself: `createdBy` is a foreign key into the project's own `issuer_users` table, not into the identity provider's `auth.users` table. For most actions this is fine — `@CurrentUser()`'s session already carries a resolved `issuer_users`-scoped identity by the time a story reaches it. But for the one story where an account doesn't exist yet (signup), naively trying to use the verified session's own `sub` claim as `createdBy` would be a type/foreign-key mismatch, not a valid fix. The correct pattern is the same self-reference trick already used for `orgId` (`issuers.orgId` self-references the row being created) — pre-generate the new actor's id before the transaction and use it consistently across every row that transaction creates, including the actor's own row.

---

## What Changed

- **`gen-architecture.md`, `database-standards.md` generation rubric (Step 7):** added a bullet requiring `database-standards.md` to state the population mechanism explicitly — no automatic mechanism exists, threading is manual, with the self-signup bootstrapping exception named.
- **`implement-story.md`, Database schema implementation rule:** added the same requirement as a concrete implementation instruction — every insert/update/soft-delete this story performs must set the relevant actor column, with the self-signup exception spelled out.
- **`ASCENDRA-PAY-001`'s own `database-standards.md` Rule 4:** the population-mechanism paragraph added directly (project-level fix, applied alongside the framework fix, not deferred to a separate `/assess-change` session since it's a direct extension of `FW-035`'s own edit to the same rule).
- **`ascendra-pay-api` code fix**, applied immediately: `OnboardingRepository.createIssuerWithDefaults` now pre-generates the Owner's `issuer_users.id` before the transaction and sets `createdBy` to it on all four inserts (`issuers`, `issuer_users` self-referentially, `branches`, `issuer_onboarding_records`). `PermissionsService.seedOwnerCapabilities` now also sets `createdBy` on the capability rows (alongside the pre-existing, domain-specific `assignedBy`). Verified against a real local Postgres: all 8 rows a signup call produces now carry the same, correct `created_by` value. Unit tests extended to assert this.

---

## Why This Was Needed (and why now, not deferred)

Same recurring shape as the whole `FW-033`–`FW-036` family: a rule that specifies a column's existence but not its behavior produces code that's technically compliant and practically useless — an audit trail with every actor column permanently `NULL` gives none of the auditability the column existed for in the first place. Left unfixed, every future story touching any master table would have hit this exact gap independently, since nothing in the standards or implementation rules would have told it otherwise. Fixing it now, in the same pass as `FW-035`, closes both halves of the same defect together rather than leaving the population-mechanism half to be rediscovered later.

---

## Implementation Status

Applied immediately during this session — `gen-architecture.md`, `implement-story.md`, `ASCENDRA-PAY-001`'s `database-standards.md`, and `ascendra-pay-api`'s `onboarding.repository.ts`/`permissions.service.ts` all edited directly, with unit tests and a live-database check confirming the fix. No separate tracking needed. `updatedBy`/`deletedBy` remain unexercised by any current code, since no story yet performs an UPDATE or soft-delete against these tables — future stories that do must follow this same standard, now that both `gen-architecture.md` and `implement-story.md` state it.

---

## Relationship to Other Decisions

Direct continuation of `FW-035` (same investigation thread as `FW-033`/`FW-034`, all 2026-07-17) — `FW-035` fixed column *existence*, this fixes column *population*. Together they close the full audit-column gap this project's own schema surfaced.
