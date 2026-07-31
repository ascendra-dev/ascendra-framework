# FW-046 — Optimistic Concurrency Client Propagation

| Field | Value |
|-------|-------|
| **ID** | FW-046 |
| **Date** | 2026-07-20 |
| **Status** | Decided — implemented |
| **Area** | `reference/api-contract/contract.md`, `.claude/commands/implement-story.md` |
| **Amends** | `FW-045` (Optimistic Concurrency API Contract) — extends the same mechanism from the backend contract to the frontend client that has to actually use it |

---

## Decision

`reference/api-contract/contract.md` Rule 8 gains a clarification: a single-resource `GET` carries
the version in an `ETag` response header, as already specified — but a collection `GET` (Rule 4's
list shape) cannot carry one `ETag` for many rows, so each object in `data[]` instead carries its own
`version` field in the body. Both are the same value at different wire locations, not two different
mechanisms.

`.claude/commands/implement-story.md`'s Next.js scaffold and Web implementation rules gain matching
instructions: `lib/api/client.ts` exports a shared `withIfMatch(version)` helper (an explicit per-call
opt-in, not blind interceptor logic like `Idempotency-Key` — only some resources are version-tracked,
and the value is resource-specific); `lib/api/error.ts`'s typed error class gains an
`isStaleVersion` getter; any domain's `.api.ts` function for a `Requires If-Match` endpoint takes
`version` as a parameter and calls `withIfMatch(version)`; the domain's response types carry
`version` on every version-tracked object; and a version-guarded mutation hook takes the whole object
the caller already has (never a bare id) and invalidates that resource's query on a
`STALE_VERSION` error.

---

## The Problem That Triggered This Decision

`FW-045` fixed the backend half of this gap (`gen-architecture.md`'s Section 5 generation, and
`contract.md`'s Rule 8 itself) after `ASCENDRA-PAY-001`'s `US-01-003` surfaced it. Implementing that
project's own retrofit — adding real `If-Match` handling to `POST /api/v1/staff/:id/remove` in
`ascendra-pay-api` — immediately exposed the mirror-image gap on the client side:
`implement-story.md`'s Next.js scaffold instructions (the `lib/api/client.ts` generation step) list
exactly what interceptors to build — `Authorization`, `Idempotency-Key` — with no mention of `ETag`/
`If-Match` at all, the same shape of omission `FW-045` already fixed once on the backend. Without a
fix here, every future project's frontend would need to reinvent how a mutation obtains and attaches
a resource's version from scratch, the same way `ascendra-pay-web`'s `staff.api.ts`/`staff.hook.ts`
would have had to.

A second, smaller gap surfaced in the same pass: Rule 8 as written only described a single-resource
`ETag`, but `ascendra-pay-api`'s actual `GET /api/v1/staff` is a list endpoint with no single-resource
`GET /api/v1/staff/:id` built yet — there was nowhere for an `ETag` header to attach to a specific
row. The pragmatic fix (`version` as a body field on each list item) is correct and necessary
regardless of which single-resource endpoints exist, since a list is the common shape most read paths
actually use — but Rule 8 didn't say this was valid until now.

---

## Why an Explicit Helper, Not a Blind Interceptor

`Idempotency-Key` is generated fresh on every mutating request with no dependency on prior state —
a pure interceptor concern. `If-Match` is not: it requires state (which version was last read for
*this specific resource*) that only the calling code — the one that fetched the object being
mutated — actually has. A `lib/api/client.ts`-level interceptor guessing at this from the URL alone
would mean maintaining a hidden resource-version cache keyed by path, fragile and hard to reason
about compared to the calling code simply passing through a value it already has in hand. `withIfMatch`
stays a thin, explicit, reusable formatting helper (get the header name and quoting right, once) —
not a mechanism that tries to infer *when* to apply it.

---

## Scope of Application

Applies to `/implement-story`'s Next.js scaffold and Web implementation rules, for every future
project this framework generates with a version-guarded resource. Does **not** retrofit
`ascendra-pay-web`'s existing code — that retrofit (adding `withIfMatch` to `lib/api/client.ts`,
`isStaleVersion` to `lib/api/error.ts`, `version` to `staff.types.ts`, and threading it through
`staff.api.ts`/`staff.hook.ts`/`RemoveStaffDialog`) is tracked as a separate, explicit
`/assess-change`-style pass against that project, not part of this framework-level decision.

---

## Implementation Status

Applied immediately during this session:
- `reference/api-contract/contract.md` Rule 8 — list-vs-single-resource clarification added
- `.claude/commands/implement-story.md` — `lib/api/client.ts`/`lib/api/error.ts` scaffold steps and
  the API-calls/Hooks Web implementation rules extended

Not yet applied: the `ASCENDRA-PAY-001`/`ascendra-pay-web` project-level retrofit (separate pass, per
PO decision).

---

## Relationship to Other Decisions

Direct extension of `FW-045`, same root-cause shape (`FW-034`/`FW-038`/`FW-039`/`FW-045`): a
completeness gap between two generation steps — here, the backend contract fix and the frontend
scaffold instructions that were supposed to consume it — discovered by actually implementing the
first fix and finding the second half missing.
