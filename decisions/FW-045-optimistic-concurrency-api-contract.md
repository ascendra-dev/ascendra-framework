# FW-045 — Optimistic Concurrency API Contract

| Field | Value |
|-------|-------|
| **ID** | FW-045 |
| **Date** | 2026-07-20 |
| **Status** | Decided — implemented |
| **Area** | `reference/api-contract/contract.md`, `.claude/commands/gen-architecture.md` |
| **Amends** | `FW-042` (Client↔Server API Contract) — adds a rule to the same fixed, pinned-once contract file |

---

## Decision

`reference/api-contract/contract.md` gains a new Rule 8: any resource whose table carries a
`version` column (per `database-standards.md`'s optimistic-locking rule) is concurrency-controlled
at the wire level via `ETag` (response) / `If-Match` (request) headers, mirroring the existing
`Idempotency-Key` header pattern (Rule 7) rather than a body-shape field — a stale `If-Match`
rejects with `409 Conflict`, `code: "STALE_VERSION"`.

`gen-architecture.md`'s Section 5 (API Contracts) generation instructions gain an explicit
cross-check: every endpoint must be checked against Section 4.2's per-table optimistic-locking
decision, and any endpoint mutating a version-locked table — including bodyless `POST` action
endpoints — must require `If-Match` per the new Rule 8.

---

## The Problem That Triggered This Decision

`database-standards.md`'s generation instructions (`gen-architecture.md`, the `database-standards.md`
derivation step) already correctly derive the `version` column "if optimistic locking is used." But
the Section 5 (API Contracts) generation instructions never read that decision back — its "Universal
rules" list covered DTO requirements and money-field typing, nothing about propagating a table's
locking decision into the request contract of any endpoint that mutates it. The two generation steps
never talked to each other.

Discovered on `ASCENDRA-PAY-001` (`US-01-003`, Add/Remove Issuer Staff Members): its `permissions.repository.ts`
was the first code in that project to `UPDATE` a Master table row. The `version` column was
faithfully incremented, but the Locked architecture's endpoint contract
(`POST /api/v1/staff/:id/remove`) had no request body at all — the endpoint's contract predated any
code that would have surfaced the gap, and nothing in `/gen-architecture`'s own instructions would
ever have produced a different result, on this project or the next one. A system-wide sweep of
`ASCENDRA-PAY-001`'s own `arch-v1.md` Section 5 found the same shape missing on every mutating action
endpoint across every module (Onboarding, IssuerAccount, Catalog, SupportAccess, Invoice, WriteOff,
BillingSchedule), none of it story-specific — a single missed cross-reference in the generation
instructions, not a one-off oversight in one endpoint.

---

## Why a Header (`If-Match`), Not a Request-Body Field

A body field (e.g. `expectedVersion` in the JSON payload) was the more obvious option but was
rejected: several of this contract's own mutating endpoints are deliberately bodyless
`POST /:id/{action}` state-transition endpoints (Rule 5) — forcing a body onto them purely to carry
one version field would be a special case per endpoint shape. A header applies uniformly to every
mutating verb, bodyless or not, and reuses the wire pattern this contract already established for
`Idempotency-Key` (Rule 7) instead of inventing a second convention for a structurally similar
problem. `If-Match`/`ETag` is also not a bespoke invention — it is the real IETF-standard mechanism
for exactly this problem (RFC 7232), the same "adopt an established pattern, don't invent one" stance
already taken for the error shape (Rule 1, RFC 7807 kinship).

---

## Scope of Application

Applies to `/gen-architecture`'s Section 5 (API Contracts) generation, for every future project this
framework generates. Does **not** retrofit any already-generated project's architecture or code —
`ASCENDRA-PAY-001`'s own retrofit (its `api-standards.md`, `arch-v1.md` Section 5, `arch-v1-ref.md`,
and `US-01-003`'s missing AC) is tracked as a separate, explicit `/assess-change` pass against that
project, not part of this framework-level decision.

---

## Implementation Status

Applied immediately during this session:
- `reference/api-contract/contract.md` — Rule 8 added, "Explicitly not standardized" list extended
- `.claude/commands/gen-architecture.md` — Section 5 "Universal rules" extended with the
  optimistic-locking cross-check

Not yet applied: the `ASCENDRA-PAY-001` project-level retrofit (separate pass, per PO decision).

---

## Relationship to Other Decisions

Extends `FW-042`'s fixed-contract-file pattern with one more pinned wire-level rule, same shape as
`Idempotency-Key` (Rule 7). Same root-cause shape as `FW-034`/`FW-038`/`FW-039`: a completeness gap
between two generation steps that should cross-reference each other but didn't, discovered mid-project
and folded back into the framework so no later project reproduces it.
