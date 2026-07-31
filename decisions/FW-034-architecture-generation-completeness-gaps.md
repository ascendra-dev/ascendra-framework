# FW-034 — Architecture Generation Completeness Gaps

| Field | Value |
|-------|-------|
| **ID** | FW-034 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md` |

---

## Decision

`/gen-architecture` now generates and checks four things it previously didn't: (1) `api-standards.md`'s rubric includes a CORS/cross-origin policy bullet whenever the topology has separate frontend/backend repos; (2) `security-standards.md`'s authentication bullet explicitly requires stating the JWT signature-verification mechanism (shared secret vs. JWKS), confirmed against what the actual identity provider uses; (3) Section 6.1 (Authentication) generation runs two completeness checks before being finalized — every assumed auth capability (login, logout, password reset, session refresh) must trace to a real BRD requirement, and any session-claims lookup keyed on an external identity provider's user id must have its linking column actually present in Section 4.2, not just described in prose; (4) Section 3.1 generation assigns each locally-run HTTP-serving repo an explicit, distinct dev port when the topology has more than one.

---

## The Problem That Triggered This Decision

Investigating the same `/verify-story` session on `US-02-001` that produced `FW-033`, four further gaps surfaced — all in what `/gen-architecture` generates or checks, not in how `/implement-story` reads it:

1. **CORS never had a standard to begin with.** `api-standards.md`'s generation rubric (Step 7) lists base path, response shapes, error shapes, resource naming, pagination, state transitions — no CORS bullet anywhere, even though Section 7.3 correctly lists `ALLOWED_ORIGINS` as an always-required env var. The env var got declared; nothing ever generated a rule for actually wiring it into bootstrap code.

2. **The auth-verification rubric was underspecified, and got lucky.** `security-standards.md`'s generation rubric only says "Authentication method (JWT Bearer, session, etc.) and access/refresh token TTLs" — no explicit prompt for *how* the signature is verified. The actual generated file happened to say JWKS correctly (confirmed by direct comparison against the file), but that was the generating run's own research, not something the rubric forced. A rubric that gets the right answer by luck on one project is not a rubric — it needed to actually ask the question.

3. **No mechanism ever checked that an assumed capability had a requirement behind it.** `supabase-auth-standards.md` Rule 1 states `ascendra-pay-web` "calls Supabase Auth directly for sign-up/login/logout/password-reset" — but nothing in `gen-architecture.md` ever cross-checks that assumption against the BRD. It turned out none of it did: the full BRD (all 774 lines, confirmed by direct read) has no requirement, journey, or security clause covering login, logout, or password reset for Issuer-side users anywhere. `US-02-001` (signup) was built and worked; there was never a way to log back in afterward. This was closed as its own change (`/assess-change`, Pattern 4, new `EPIC-015`/`REQ-084`–`086`) — `FW-034` fixes the generation-time gap that let it happen unnoticed, not the missing requirement itself.

4. **The custom-claims mechanism was described without its own prerequisite.** The same `supabase-auth-standards.md` describes a custom-claims hook resolving `org_id`/capabilities from `staff_capability_assignments` at login — but `issuer_users` (Section 4.2) had no column linking it to Supabase's own `auth.users.id`, so that hook had no way to find the right row. `gen-architecture.md` never mentions "custom claims," "auth.users," or "JWKS" anywhere in its own instructions; the hook was described in prose with nothing forcing its supporting schema piece to exist. Closed at the architecture level in the same `/assess-change` session (`issuer_users.supabase_user_id`).

5. **Dev port collision.** `ascendra-pay-api` and `ascendra-pay-web` both defaulted to port 3000 — undiscovered until the two were actually run side by side for the first time, during this same verification session. `gen-architecture.md` never mentions dev-server port assignment anywhere, for any repo.

---

## What Changed in `gen-architecture.md`

- **Step 7, `api-standards.md` rubric:** added a CORS/cross-origin policy bullet — required whenever frontend and backend are separate repos or origins (this framework's standard topology), stating where in bootstrap code it must be wired in, not just that the env var exists.
- **Step 7, `security-standards.md` rubric:** extended the authentication bullet to require stating the exact JWT verification mechanism (shared secret vs. JWKS), confirmed against the actual identity provider, not assumed.
- **Section 6.1 (Authentication) generation instructions:** added two completeness checks run before finalizing the subsection — BRD traceability for every assumed auth capability, and linking-column completeness for any external-identity-keyed claims lookup.
- **Section 3.1 (Project Structure) generation instructions:** added dev port assignment guidance for topologies with more than one locally-run HTTP-serving repo.

---

## Why This Was Needed (and why now, not deferred)

Items 1, 2, and 5 recur on every future project with a separate frontend/backend topology — not an edge case, the framework's standard shape. Items 3 and 4 recur on every future project using a managed auth provider with custom claims — again the framework's own recommended default for this kind of multi-tenant SaaS shape, not a one-off choice specific to `ASCENDRA-PAY-001`. Left unfixed, each would independently resurface the same class of defect (a broken or unusable login flow, a blocked cross-origin frontend) on the very next project that reaches real end-to-end verification, at the same late stage — after implementation, during `/verify-story` — rather than being caught at generation time when it's nearly free to fix.

The fix is scoped to the five demonstrated gaps. It does not attempt to add a general-purpose "verify every architectural assumption against the BRD" pass across all of Section 6 — only the specific, recurring shape (auth capabilities, claims-linking columns) that actually broke here. Extend further only when a concrete case demands it.

---

## Implementation Status

Applied immediately during this session — `.claude/commands/gen-architecture.md` edited directly; no separate tracking needed.

---

## Relationship to Other Decisions

Companion to `FW-033` (Standards Compliance in `/implement-story`), found and fixed in the same investigation. The missing-login gap this decision's generation-time fix (item 3) targets was itself closed for `ASCENDRA-PAY-001` via `/assess-change` (Pattern 4) — a new `EPIC-015` and `REQ-084`–`086`, plus the `issuer_users.supabase_user_id` schema fix for item 4 — recorded in `brds/brd-core-v1.md` v1.22 and `arch-v1.md`'s Change History, not in this decision file, since that work is project-specific, not a framework change.
