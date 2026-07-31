# FW-042 — Client↔Server API Contract

| Field | Value |
|-------|-------|
| **ID** | FW-042 |
| **Date** | 2026-07-19 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md`, `.claude/commands/implement-story.md`, new `reference/api-contract/contract.md` |
| **Amends** | `FW-041` (Logging and Exception Handling Completeness) — same "generate fresh per project from a fixed reference" pattern, applied to the API request/response contract |

---

## Decision

A new file, `reference/api-contract/contract.md`, fixes the client↔server wire contract — error
response shape, list-response shape, HTTP method semantics, query parameter conventions, and
header conventions (`Authorization`, `X-Request-Id`, `Idempotency-Key`) — as one literal schema
every project generates against, replacing what was previously a rubric (`api-standards.md` must
have *a* `code` field, must distinguish business-rule violations) with no guarantee two projects'
generated shapes actually match field-for-field. `/gen-architecture`'s `api-standards.md`
generation now cites this file directly instead of re-deriving a shape from scratch. `/implement-story`'s
NestJS and Next.js first-run scaffold steps both cite it explicitly — the backend's exception
filter and the frontend's API client are generated fresh per project, tech-stack-appropriate, the
same way `AllExceptionsFilter`/`pino.config.ts` already are (`FW-041`) — **not** vendored from a
shared library. `ascendra-ui` is untouched by this decision; it remains a UI component library only.

---

## The Problem That Triggered This Decision

Reviewing `ASCENDRA-PAY-001`'s real code (`ascendra-pay-api`'s `AllExceptionsFilter`,
`ascendra-pay-web`'s `lib/api.ts`/`lib/api-error.ts`) against a separate library the PO maintains
(`ascendra-ui`, which also ships its own `lib/api/*`) found the two had converged on almost
nothing in common: different auth assumptions (`next-auth` vs. this project's actual Supabase
auth), a `{success, data}`/`{success: false, error: {...}}` envelope on one side vs. a flat,
unwrapped body on the other, different pagination field names, and no `requestId`/error-tracking
support at all on one side.

Tracing the root cause through this framework's own generation instructions: `gen-architecture.md`
Step 7's `api-standards.md` instruction only ever asked for a *rubric* — "include a machine-readable
`code` field alongside `message`" — never a pinned schema. Two different projects could each
satisfy that rubric with genuinely different field names (`statusCode` vs `status`, `error` vs
`errorCode`) and both be "correct." Unlike `FW-040` (where the gap was projects drifting from an
*external* library's real behavior), this gap was structural: nothing in the framework's own
generation instructions was ever going to produce the same shape twice, because nothing pinned one
shape as canonical in the first place.

---

## Why a Fixed Reference File, Generated Fresh Per Project — Not a Shared Library

Two shapes of "make this consistent across projects" were considered:

1. **Vendor a shared client library** (the `FW-040` pattern — `ascendra-ui` ships real component
   code, copied wholesale into every consuming project and re-synced on drift).
2. **Fix the contract, generate the implementation fresh per project** (the `FW-041` pattern —
   `logging-standards.md` is a fixed reference; `AllExceptionsFilter`/`pino.config.ts` are written
   fresh by `/implement-story` for every project, tech-stack-appropriate, never copied from
   anywhere).

This decision takes the second shape, deliberately. The two problems `FW-040` and `FW-041` solve
are not the same problem: a UI component library is a large, stateful, visually-tested artifact
where re-deriving it from scratch per project would be enormous waste and a correctness risk of its
own — vendoring and syncing is the right call there. An API client implementing a fixed
request/response contract is comparatively small, and — critically — the *auth provider* half of
it is inherently project-specific (`system-architecture.md` Section 2), which a vendored library
would have to solve with a pluggable injection seam. Generating it fresh, per project, directly
against whatever that project's confirmed stack actually is, avoids that indirection entirely: the
same rubric-based reasoning `FW-041` already established for the NestJS side (a global exception
filter and Sentry initialization are generated fresh per project, not vendored) applies symmetrically
to the Next.js side of the same contract. `ascendra-ui` stays exactly what it already is — nothing
about this decision touches it.

---

## Scope of Application

Applies to `/gen-architecture` (Step 7's `api-standards.md` generation) and `/implement-story`
(the NestJS and Next.js first-run scaffold steps, and the Standards Compliance Self-Check). Does
**not** apply to `ascendra-ui` — no file in that project is touched or referenced by this decision.
Does **not** retrofit `ASCENDRA-PAY-001`'s existing `ascendra-pay-api`/`ascendra-pay-web` code —
both already work and are left as-is; a future project (or a future, separate, explicit retrofit
decision) is what actually exercises these updated generation instructions for the first time.

---

## Implementation Status

Applied immediately during this session:
- `reference/api-contract/contract.md` created
- `.claude/commands/gen-architecture.md` Step 7 edited to cite the fixed contract
- `.claude/commands/implement-story.md` edited: NestJS scaffold step, Next.js scaffold step, and
  the Standards Compliance Self-Check

Not yet applied: retrofitting `ascendra-pay-api`/`ascendra-pay-web` to this exact contract — a
separate, explicit PO decision, tracked outside this ADR (both already satisfy most of this
contract in practice, since it was derived from their real, working shape, but neither has been
re-verified field-for-field against the now-fixed schema).

---

## Relationship to Other Decisions

Extends `FW-041`'s "generate fresh per project from a fixed reference file, never leave it to
whichever story happens to touch bootstrap code first" pattern from the logging/exception-handling
subsystem to the API request/response contract subsystem. Deliberately does **not** follow
`FW-040`'s vendored-library pattern — see "Why a Fixed Reference File..." above for the reasoning
split. Same root-cause shape as `FW-033`/`FW-034`/`FW-041`: a command's generation instruction was a
rubric where a fixed schema was actually needed, discovered by comparing generated output against
a second, independently-built implementation of the same problem.
