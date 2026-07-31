# FW-037 — Audit Column Actor-Label Design

| Field | Value |
|-------|-------|
| **ID** | FW-037 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `database-standards.md`, `supabase-auth-standards.md`, `arch.template.md`'s audit-column guidance, all already-generated `arch-v1.md` Section 4.2/6.1 content |
| **Amends** | FW-035 — Audit Column Symmetry; FW-036 — Audit Column Population Mechanism |

---

## Decision

The `createdBy`/`updatedBy`/`deletedBy` audit columns (`FW-035`) change from `uuid`, foreign-keyed to a single user table (`issuer_users.id`), to `varchar` — a plain, self-contained actor-label string with no foreign key and no companion type-discriminator column. Population stays a manual, explicit responsibility (`FW-036`'s core rule is unchanged), threaded as a single `actor: string` parameter from controller → service → repository. Each repository method already knows, from its own operation, which of the three columns it writes — no type/flag parameter is ever needed to decide that. Resolution of `actor` by caller type: an authenticated session resolves it from the verified token's `name` claim (newly added to the custom-claims hook, alongside a previously-described-but-never-wired `role` claim); a Payer acting through a tokenized link resolves it to the real Payer's name (already loaded for that action, not a fresh lookup); a worker/webhook/system-triggered write uses a fixed literal from a centralized constants file (e.g. `'System (billing schedule)'`).

---

## The Problem That Triggered This Decision

`FW-035`/`FW-036` fixed audit-column *existence* and *population discipline*, but never questioned the column's *shape* — a `uuid` foreign key into `issuer_users.id`. That shape assumes every actor in the system is an Issuer Staff member. It isn't: Section 6.2's own State Machine table names `System` (scheduled jobs, gateway webhooks) and `Payer` (tokenized-link actions — invoice viewed, payment initiated) as legitimate actors for several transitions, and `ascendra_staff` (Compliance/Onboarding Reviewer, Platform Support Staff) is a second, deliberately separate identity table with no `orgId` at all (`security-standards.md` Rule 3). A single-table FK cannot represent any of these — for every action taken by a worker, a webhook, or a Payer, the column can only ever be `NULL`, silently losing exactly the "who" information the column exists to capture.

The gap surfaced during `/gen-story-plan` for `US-01-003` (Add/Remove Issuer Staff), while resolving how the Remove Staff endpoint would populate `deletedBy`. Working through it surfaced a second, more serious problem underneath the first: `OnboardingRepository.createIssuerWithDefaults` (`US-02-001`, already `Done`) never populates `issuer_users.supabaseUserId` at signup, despite that column existing specifically so the custom-claims hook (`supabase-auth-standards.md` Rule 3) can resolve a logged-in user's `org_id`/`capabilities`. Without that link, the hook cannot find *any* user's row — meaning `org_id`/`capabilities` currently fail to resolve for every real login, not just the new one this story needs (`name`). `CapabilityGuard`-protected endpoints are effectively unreachable by a real, non-manually-tokened session today. This decision's root-cause fix (below) closes that gap as a prerequisite, not a side effect.

---

## Rationale / Research Basis

Considered three shapes for the "who" value, in order:

1. **`uuid` FK to `issuer_users.id`** (the `FW-035` status quo) — rejected: cannot represent `System`/`Payer`/`ascendra_staff` actors at all; the column silently goes `NULL` exactly where the domain says a non-Issuer-Staff actor legitimately acted.
2. **`actorType` enum + `actorId` uuid (opaque, no DB-level FK)** — the pattern general audit-log design research converges on (see Sources), specifically to avoid embedding PII directly in an immutable audit trail. Rejected for this project on cost/benefit grounds: it doubles the audit surface (6 columns per Master table instead of 3) for a lookup that, on inspection, is only ever exercised on a cold path (a human opening one row's history, not a hot list endpoint) — the join-cost concern motivating the search for an alternative doesn't actually apply at this system's read frequency, but the schema-complexity cost of the type+id shape is real and paid on every table regardless.
3. **Plain `varchar` actor-label, no FK, no type column** — adopted. A self-contained value, decided by the calling code at the moment of the action, needs nothing else (no lookup, no join, no type discriminator) to be meaningful — the only shape that satisfies "no relation to any single user table" *and* "no per-row type flag" simultaneously, since a bare untyped `id` is provably unresolvable across heterogeneous actor kinds (there is no way to know which table to join against without a type signal of some kind, and a type column was explicitly rejected).

The known cost of option 3, weighed openly: a person's name is copied into every row they touch, rather than referenced once. Research on audit-log design (see Sources) flags this as a real tradeoff — repeated PII duplicates a value across many rows, complicates correcting a name after the fact, and duplicates the same PII this project's own `arch-v1.md` Section 6.4 already treats as sensitive. Accepted anyway for this project's current scope: no right-to-erasure or similar regulatory requirement is in scope for this v1 build, and the tradeoff has a clean, deferred migration path (add an id-based lookup later, only if a real requirement forces it) rather than needing to be solved now.

**Sources consulted:**
- [Database Design for Audit Logging](https://www.red-gate.com/blog/database-design-for-audit-logging/)
- [How to Design a Schema for Audit Logging in MySQL](https://oneuptime.com/blog/post/2026-03-31-mysql-design-schema-for-audit-logging/view)
- [Designing an Audit Log System: Immutable Events, Efficient Querying, and Compliance at Scale](https://letsbuildsolutions.com/blog/system-design/designing-an-audit-log-system-immutable-events-efficient-querying-and-compliance-at-scale/)
- [Guide to Building Audit Logs for Application Software](https://medium.com/@tony.infisical/guide-to-building-audit-logs-for-application-software-b0083bb58604)

---

## What Changed

**1. Column shape (supersedes `FW-035`'s `uuid` FK):**
```typescript
createdBy: varchar('created_by', { length: 200 }),
updatedBy: varchar('updated_by', { length: 200 }),
deletedBy: varchar('deleted_by', { length: 200 }),
```
No `.references()`, no companion type/discriminator column. Applies uniformly to every Master table's three audit dimensions, project-wide.

**2. Population contract (extends `FW-036`, which still governs — population is still always manual, never automatic):** a single `actor: string` parameter is threaded controller → service → repository on every insert/update/soft-delete. Each repository method is operation-specific and already implies which column it writes (an insert method sets `createdBy`; an update method sets `updatedBy`; a soft-delete method sets `deletedBy` alongside `deletedAt`) — the caller never passes a flag to select the column; the method being called decides it.

**3. Actor resolution by caller type:**
- Authenticated session (Issuer Staff, Ascendra-internal): `actor` = the verified token's `name` claim.
- Payer, tokenized-link action (`SEC-007`): `actor` = the real Payer's name, from the `payers` row already loaded for that action (e.g. via the invoice's payer relation) — not a generic `'Payer'` placeholder.
- Webhook-triggered write: a fixed literal matching the State Machine's own vocabulary, e.g. `'System (payment gateway webhook)'`.
- Worker/scheduled job: a fixed literal, e.g. `'System (billing schedule)'`, centralized in `src/common/constants/system-actor.ts` rather than typed ad hoc per call site.

**4. Custom-claims hook extended** (`supabase-auth-standards.md` Rule 3): now also emits `name` (so controllers never need a DB lookup to populate `actor`) and `role` — a claim the architecture already described conceptually (Section 6.1: `role: 'reviewer'` for Compliance/Onboarding Reviewer and Platform Support Staff sessions) but which was never actually added to the `SupabaseJwtClaims` TypeScript interface or the hook itself. Same staleness-until-next-token-refresh caveat already accepted for `capabilities` (Rule 4) applies to `name`/`role` — not a new risk category, an extension of an already-accepted one.

**5. Root-cause fix (prerequisite for all of the above to work against a real login):** `OnboardingRepository.createIssuerWithDefaults` now sets `issuer_users.supabaseUserId` on the Owner's row at signup. Without this, the custom-claims hook cannot resolve `org_id`/`capabilities`/`name`/`role` for *any* real session — this was already broken before this decision, for reasons unrelated to audit columns; fixing it here because this is the change that first needed to exercise the claims chain end-to-end.

---

## Implementation Status

Applied immediately during this session:
- `database-standards.md` Rule 4, `supabase-auth-standards.md` Rule 3 — updated.
- `arch-v1.md` Section 4.2 (every Master table), Section 6.1 (JWT payload shape), Change History — updated; `arch-v1-ref.md` regenerated.
- `ascendra-pay-api`: schema files, a new migration, `OnboardingRepository`, `PermissionsService`/`PermissionsRepository`, `SupabaseAuthGuard`'s claims type, new `system-actor.ts` constants — patched.
- `story-plans/US-01-002-plan.md`, `US-02-001-plan.md` — Deviation entries added recording the retroactive column-shape and `supabaseUserId` fixes.
- `story-plans/US-01-003-plan.md` — API-2/API-3 rewritten to the `actor: string` contract; stays `Draft` pending PO confirmation.

No BRD, epic, or story ACs changed — this is an implementation-only correction (Pattern 1, per `/assess-change`'s classification), the same category as `FW-035`/`FW-036`.

---

## Relationship to Other Decisions

Direct amendment of `FW-035` (column shape) and `FW-036` (population contract stays manual/explicit, just carrying a string instead of a uuid now). Does not reopen `FW-033`/`FW-034`. Surfaced and resolved during `/gen-story-plan`/`/assess-change` work on `US-01-003`, the same discovery pathway `FW-035`/`FW-036` themselves came from.
