# FW-043 — Frontend Auth Session Module Generation

| Field | Value |
|-------|-------|
| **ID** | FW-043 |
| **Date** | 2026-07-19 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md`, `.claude/commands/implement-story.md` |
| **Amends** | `FW-042` (Client↔Server API Contract) — closes a dependency gap `FW-042`'s own scaffold step exposed but did not itself resolve |

---

## Decision

`security-standards.md`'s generation instructions (`/gen-architecture` Step 7) now require naming
the concrete frontend module(s)/function(s) used to retrieve the current session's access token
(e.g. `getBrowserAccessToken()`), not just which auth SDK is used — and, where the frontend
framework has a client/server execution split (e.g. Next.js Client vs. Server Components), require
stating the file split explicitly. `/implement-story`'s Next.js first-run scaffold gains a new
step (item 10) generating this auth session module (`lib/auth.ts`, and `lib/auth.server.ts` where
the split applies) fresh per project — positioned before the API client step (`FW-042`, now item
11), which depends on it.

---

## The Problem That Triggered This Decision

While retrofitting `ascendra-pay-web`'s API client to `FW-042`'s contract, a direct question about
whether `lib/auth.ts`/`lib/auth.server.ts` were also covered by a scaffold-time generation step
surfaced that they were not. Grepping `implement-story.md` for `auth.ts`, `auth.server.ts`,
`getBrowserAccessToken`, and `Supabase` returned zero matches in the Next.js scaffold section — the
only auth-related instruction in the whole file governs the *backend* guard/token-verification
mechanism. `ascendra-pay-web`'s real `lib/auth.ts`/`lib/auth.server.ts` exist purely because some
story built them ad hoc the first time one was needed, exactly the failure shape `FW-041` found for
logging and `FW-042` found for the API contract.

It also directly undermined `FW-042` itself: the `lib/api.ts` scaffold step's request interceptor
was written to "attach `Authorization: Bearer <token>` using this project's actual confirmed auth
provider" — silently assuming a token-getter function already exists, with no step that ever
produces one. Checking the second half of the problem — whether `security-standards.md`'s own
generation instructions even had anything concrete to cite — found the same rubric-not-schema gap
`FW-042` fixed for the API contract: the file's auth bullets cover authentication method, token
verification, RBAC guard patterns, all backend-side; nothing requires pinning the frontend
module/function-name convention.

---

## Why Both Halves Needed Fixing, Not Just One

Adding a scaffold step to `implement-story.md` alone would have had nothing authoritative to cite —
the same problem `FW-042` solved by fixing `gen-architecture.md`'s `api-standards.md` instruction
before wiring `implement-story.md` to it. Fixing only `gen-architecture.md` would have documented
the convention without ever generating it. Both were required, in the same order `FW-042`
established: pin the convention in the standards-generation instruction first, then wire the
scaffold step to cite it.

The two-file split (`lib/auth.ts` / `lib/auth.server.ts`) is stated explicitly rather than left
implicit because it is a correctness requirement, not a style preference, for any frontend
framework with a client/server execution boundary: a server-only import (e.g. Next.js's
`next/headers`) inside a module a Client Component imports breaks the client bundle. `ascendra-pay-web`'s
existing split already follows this correctly — the fix formalizes why, so the next project's
scaffold reproduces it deliberately rather than by luck.

---

## Implementation Status

Applied immediately during this session:
- `gen-architecture.md`'s `security-standards.md` generation instructions gained a bullet requiring
  the frontend session/token-retrieval module convention
- `implement-story.md`'s Next.js scaffold gained item 10 (the auth session module), with items
  10–14 renumbered to 11–15 and the one cross-reference to the old item 10 in the "API calls"
  Web implementation rule updated to item 11

Not yet applied: retrofitting `ascendra-pay-web`'s existing `lib/auth.ts`/`lib/auth.server.ts`
against this now-explicit convention, or updating `ASCENDRA-PAY-001`'s `security-standards.md`/`supabase-auth-standards.md`
to state the convention explicitly — both already work in practice and are left as-is, consistent
with `FW-042`'s own deferred retrofit.

---

## Relationship to Other Decisions

Direct dependency fix for `FW-042`: the API client step `FW-042` added assumed a token-getter
function without ever generating one — this decision closes that gap. Same root-cause shape as
`FW-033`/`FW-034`/`FW-041`/`FW-042`: a generation instruction was a rubric or silently assumed a
prerequisite where a fixed, scaffold-generated convention was actually needed.
