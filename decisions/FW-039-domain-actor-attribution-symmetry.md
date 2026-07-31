# FW-039 — Domain-Specific Actor Attribution Symmetry

| Field | Value |
|-------|-------|
| **ID** | FW-039 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `database-standards.md` (project-level rule), `.claude/commands/gen-architecture.md` (Section 4.2 generation guidance), `.claude/commands/review-architecture.md` (new structural check) |
| **Related** | FW-037/FW-038 — same investigation thread (audit columns), but this decision is about domain-specific lifecycle columns, not the generic `created_by`/`updated_by`/`deleted_by` triad those cover |

---

## Decision

Beyond the generic audit-column triad (`FW-035`–`FW-038`), a table's own domain-specific lifecycle timestamps (`assigned_at`, `submitted_at`, `revoked_at`, `approved_at`, and similar) each get evaluated individually for whether they need a paired `_by` column — **not** a blanket "always pair `_at` with `_by`" rule. Pair it when the state machine attributes the transition to a specific human role; don't pair it when the transition is `System`/webhook-triggered, is a future deadline/schedule field rather than a record of something that happened, or when the event and row creation are the same non-repeating moment already covered by the generic `createdBy`. `database-standards.md` Rule 4a documents this judgment explicitly. `gen-architecture.md`'s Section 4.2 generation guidance and `review-architecture.md`'s structural checks are both updated so this gets caught during generation and review for future projects, not just found by a PO manually reading the schema.

---

## The Problem That Triggered This Decision

Found by direct PO review of `arch-v1.md`'s already-`Locked` data model, not by implementation or `/verify-story`: `staff_capability_assignments` has `assignedBy` (paired with `assignedAt`) but no `revokedBy` for `revokedAt`; `issuer_onboarding_records` has `reviewedBy` (paired with `reviewedAt`) but no `submittedBy` for `submittedAt`. Each table had already established the actor-attribution pattern for one of its lifecycle transitions but silently dropped it for another — an internal asymmetry, not merely an omission. A systematic check across all 20 tables (matching every domain `_at` column with its module) found a third instance: `support_access_grants` has `requestedBy`/`authorizedBy` but no `revokedBy` for `revokedAt`, despite Section 6.2 explicitly naming "Support Access Grantor" as the actor for that transition.

The same check also surfaced several `_at` columns that correctly have **no** `_by` counterpart (`activated_at`, `expires_at`, `cleared_at`, `first_viewed_at`, `next_fire_at`, `sent_at`, `occurred_at`) — confirming this needed a documented judgment call, not a mechanical rule that would have wrongly demanded `_by` columns for system-triggered events and future-scheduling fields.

A second problem surfaced while fixing the first: `revokedBy`/`submittedBy` are genuine `uuid` foreign keys (like the pre-existing `assignedBy`/`reviewedBy` they're paired with) — not the `varchar` actor-label shape `FW-037` gave the generic audit columns, since these columns carry real business meaning as an actual reference, not just a label. But the JWT claims `FW-037` added (`name`, `role`) never carried the caller's actual `issuer_users.id` — only `sub` (Supabase's own `auth.users.id`, which does not satisfy this FK) and the display name. Populating a real domain-specific FK from a session therefore had no working mechanism at all.

---

## What Changed

**`ASCENDRA-PAY-001` (project-level, via `/assess-change`):**
- Added `revokedBy` (`staff_capability_assignments`), `submittedBy` (`issuer_onboarding_records`), `revokedBy` (`support_access_grants`) — all `uuid`, FK'd to `issuer_users.id`, matching the shape of the columns each is paired with.
- Added `issuer_user_id` to the custom-claims hook (alongside `FW-037`'s `name`/`role`) and the `SupabaseJwtClaims` TypeScript interface — the hook already looks up the `issuer_users` row for `name`/`org_id`; emitting its `id` too was a one-line addition, not a new lookup.
- `database-standards.md` Rule 4a documents the pair/don't-pair judgment criteria explicitly, so this file states *why* each domain timestamp does or doesn't have a `_by` counterpart rather than leaving it implicit.
- `story-plans/US-01-003-plan.md`'s API-3 (Remove Staff) updated: `revokedBy` resolves from `user.issuer_user_id`, explicitly distinguished from the `actor` name string used for the generic `updatedBy` on the same row.

**Framework-level (for future projects):**
- `gen-architecture.md`'s Section 4.2 (Data Model) generation guidance now includes Rule 4a's judgment as one of its "universal rules" bullets, and the `database-standards.md`-generation instructions state the same judgment for the file itself — both point at the same criteria so a freshly generated project gets this from its first `/gen-architecture` run, not a later PO review pass.
- `review-architecture.md` gains check 20a (Concern B — Data Model): flags a table where one domain-specific lifecycle transition has a paired `_by` column and a sibling transition on the same table doesn't, unless the unpaired one is correctly exempt under Rule 4a. Added as a lettered sub-check (`20a`) rather than renumbering 21–34, since several of those checks are cross-referenced by number elsewhere in the same file (e.g. check 16 references check 28).
- `SDLC.md` and `PROJECT-LIFECYCLE.md`'s "34 structural checks" references updated to 35.

---

## Implementation Status

Applied immediately during this session — `ASCENDRA-PAY-001`'s schema, claims hook, `database-standards.md` Rule 4a, and the affected story plan; and the framework-level `gen-architecture.md` (Section 4.2 + `database-standards.md` generation instructions) and `review-architecture.md` (check 20a) updates. No separate tracking needed.

---

## Relationship to Other Decisions

Sibling to `FW-037`/`FW-038`, same investigation thread (actor attribution across the schema), but a distinct concern: `FW-037` fixed the generic triad's *shape*, `FW-038` generalized that fix into `/gen-architecture`, and this decision fixes *domain-specific* actor columns that `FW-035`–`FW-038` never covered at all. The `issuer_user_id` claim this decision adds is a direct dependency for any future domain-specific actor FK a project needs — not a replacement for `FW-037`'s `name`/`role` claims, which remain the correct source for the generic label columns.
