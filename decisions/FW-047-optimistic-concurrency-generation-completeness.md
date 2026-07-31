# FW-047 — Optimistic Concurrency Generation Completeness

| Field | Value |
|-------|-------|
| **ID** | FW-047 |
| **Date** | 2026-07-20 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md`, `.claude/commands/implement-story.md` |
| **Amends** | `FW-045` (Optimistic Concurrency API Contract), `FW-046` (Optimistic Concurrency Client Propagation) |

---

## Decision

Closes two classes of gap found while auditing `FW-045`/`FW-046` for completeness against a
hypothetical *next* project, rather than assuming the two fixes already made were sufficient:

**1. Generation completeness — a fresh project would still miss this.** `FW-045`/`FW-046` fixed
`reference/api-contract/contract.md` (the wire rule) and `gen-architecture.md`'s Section 5 (the
per-endpoint contract), but never taught the actual *code-generating* instructions to produce the
supporting pieces:
- `implement-story.md`'s NestJS scaffold step (backend bootstrap) never generated a stale-version
  exception or gave the global filter a branch for it — only `BusinessRuleViolationException` was
  ever mentioned. Now conditional: "if `database-standards.md` uses optimistic locking on any
  table," generate the second typed exception and its own filter branch.
- `gen-architecture.md`'s `api-standards.md` Tier 1 generation and the backend/frontend Tier 2
  exception-handling rubrics never mentioned Rule 8 at all. Now: `api-standards.md` names the
  concrete stale-version exception type (or states "Not applicable" explicitly); the Tier 2 rubrics
  require the filter branch and the matching frontend shape.
- `implement-story.md`'s Standards Compliance Self-Check re-checks Rule 1/Rule 2 compliance against
  the diff but never Rule 8 — the exact check that should have caught the original gap on
  `US-01-003` never existed. Now added: does the handler actually compare `If-Match`, not just
  increment `version`; does a mismatch reject via the typed exception at `409` with its own `code`.

**2. Conditionality — a project that doesn't need this would still get it.** The `FW-046` frontend
scaffold additions (`isStaleVersion`, `withIfMatch`) were worded unconditionally in
`implement-story.md` — every project's scaffold would generate them regardless of whether that
project ever uses optimistic locking. Now gated on the same fact the backend gates on: "if the
backend's `database-standards.md` uses optimistic locking on any table."

A third, upstream gap surfaced while tracing this: `database-standards.md`'s own generation
instruction (the master-table standard-columns bullet) stated `version` as "(if optimistic locking
is used)" with no defined decision process anywhere — unlike the adjacent `tenant/org identifier
(if multi-tenant)` clause, which grounds in an explicit, separately-confirmed architectural fact.
Fixed by making it an explicit default, mirroring the actor-column rule in the same sentence:
optimistic locking is the baseline expectation for any Master table more than one actor can
concurrently mutate (nearly every Master table in a multi-user system); omission requires a stated
reason, never silence.

---

## The Problem That Triggered This Decision

The PO asked directly, after `FW-045`/`FW-046` and their project-level retrofits were both applied
and verified working in `ASCENDRA-PAY-001`/`ascendra-pay-web`: would a *new* project actually
reproduce this automatically, and would a project that doesn't need it stay clean? Tracing both
questions through the actual generation instructions (not assuming the prior two fixes were complete
because the retrofit worked) found the four gaps above. This is the same failure shape `FW-045`
itself was found by: a fix applied at one layer (the wire contract, or in this audit's case, the
per-endpoint contract and the client helper) without confirming every downstream generation step
that's supposed to consume it actually does.

---

## Scope of Application

Applies to `/gen-architecture` and `/implement-story`, for every future project. Does not change
`ASCENDRA-PAY-001`'s already-applied `FW-045`/`FW-046` retrofit, which independently satisfies
everything this decision now makes automatic for the next project.

---

## Implementation Status

Applied immediately during this session:
- `implement-story.md` — backend scaffold (stale-version exception, conditional), frontend scaffold
  (`isStaleVersion`/`withIfMatch`, now conditional), Standards Compliance Self-Check (Rule 8 added)
- `gen-architecture.md` — `api-standards.md` Tier 1 generation (conditional stale-version exception
  naming), backend/frontend Tier 2 exception-handling rubrics (Rule 8 branch requirement), and
  `database-standards.md`'s `version` column bullet (converted from an undefined conditional to an
  explicit default-with-stated-exception, matching the actor-column precedent)

---

## Relationship to Other Decisions

Direct completeness audit of `FW-045`/`FW-046`, same root-cause shape as both (and `FW-034`/`FW-038`/
`FW-039`): a fix that's correct as far as it goes, found incomplete by checking every generation step
downstream of it rather than stopping at the first one that mattered for the triggering project.
