# FW-033 — Standards Compliance in `/implement-story`

| Field | Value |
|-------|-------|
| **ID** | FW-033 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/implement-story.md` |

---

## Decision

`/implement-story` Step 2 now requires reading every file in `projects/{PROJECT_CODE}/standards/` relevant to the story's target(s) as a mandatory, always-read source — not just `arch-v1.md`/`arch-v1-ref.md`. Step 4's Authentication implementation rule no longer names a generic placeholder guard (`JwtAuthGuard`, `RolesGuard`); it instructs using the exact guard/verification mechanism `security-standards.md` (and any auth-provider-specific standard) actually documents. A new **Standards Compliance Self-Check** runs after implementation and before Step 5's automated checks: re-read the applicable standards files and check the real diff against each stated rule, since lint/build/test cannot catch a missing required trigger, an unwired required policy, or a verification mechanism that diverges from the project's own documented convention.

---

## The Problem That Triggered This Decision

While verifying `US-02-001` (`ASCENDRA-PAY-001`) against a real Supabase project, `SupabaseAuthGuard` rejected every genuine login token. The guard verified tokens with a shared secret (HS256); the connected Supabase project signs sessions asymmetrically (ES256, via JWKS) — Supabase's current default. This wasn't a design ambiguity: `security-standards.md` Rule 1 and `supabase-auth-standards.md` Rule 2, both already generated for this project, correctly specified JWKS verification. `arch-v1.md` line 1154 said the same thing. The correct answer was already written down — twice — and the implementation still didn't follow it.

Root cause, confirmed by reading `implement-story.md` directly: Step 2 ("Read mandatory reference files") lists exactly six sources to read — `arch-v1-ref.md`, `arch-v1.md`, Open Decisions, domain knowledge, UI component reference, screen design/mocks. **It never once instructs reading anything in `projects/{PROJECT_CODE}/standards/`.** Step 4's own Authentication rule made it worse — it gave generic NestJS boilerplate (`@UseGuards(JwtAuthGuard, RolesGuard)`) that doesn't even match this project's real guard names, with no explicit instruction to check `security-standards.md` for the actual mechanism. The implementation defaulted to the most common generic JWT pattern instead of the project's own correctly-documented one, because nothing ever pointed at the file that would have prevented it.

The same investigation also found a second, distinct case: migration `0001` never attached the `set_updated_at` trigger `database-standards.md` Rule 8 explicitly requires ("Every table with an `updatedAt` column gets this trigger in the same migration"). Unlike the auth case, `implement-story.md` **does** already state this rule directly (Step 4, Database schema bullet). It was still missed. This is a different failure mode — not a missing pointer, but no self-verification pass catching an already-stated rule that wasn't followed. Every other major command in this framework (BRD, epic, story, screen design) ends with a Part 2 Verification checklist run against its own output before calling it done. `implement-story.md` had no equivalent — Step 5 only runs `lint`/`build`/`test`, none of which can catch "did I follow every applicable standards rule."

Both defects were found and fixed live during `/verify-story` on `US-02-001` — see `projects/ASCENDRA-PAY-001/test-reports/US-02-001-verification.md` and `story-plans/US-02-001-plan.md` Section 7 (Deviations 1 and 13) for the full incident record.

---

## What Changed in `implement-story.md`

- **Step 2, new item 2** (existing items renumbered 3–7, two downstream cross-references to "Step 2 item 6" updated to "item 7"): every file in `projects/{PROJECT_CODE}/standards/` relevant to the story's target(s) — `api-standards.md`, `database-standards.md`, `security-standards.md`, any auth-provider-specific standard, `test-strategy.md`, any per-technology standard — added as an always-read source, explicitly stated as necessary in addition to, not a substitute for, `arch-v1.md`/`arch-v1-ref.md`.
- **Step 4, Authentication rule:** replaced the generic `JwtAuthGuard`/`RolesGuard` placeholder with an instruction to use the exact guard/decorator/verification mechanism `security-standards.md` documents, with explicit emphasis that shared-secret-vs-JWKS is a specific, binding detail, not a default to assume.
- **New section, "Standards Compliance Self-Check"**, inserted between Step 4 and Step 5 (no step renumbering — added as its own delimited block): re-read the applicable standards files and check the actual diff against each stated rule before running automated checks.

---

## Why This Was Needed (and why now, not deferred)

Both defects found here are the kind that only surface when someone actually exercises the system end-to-end against a real dependency (a real Supabase project, a real second run touching the same tables) — exactly what most projects, run by a solo PO without a dedicated QA function, are least likely to do early. Left unfixed, this gap would recur on every future project the moment its architecture relies on a standards-file detail more specific than what's inlined in `arch-v1.md` itself — which is the normal case, not an edge case, since that's the entire reason `standards/` files exist as a separate tier (`FW-025`).

The fix is scoped to the demonstrated failure mode — pointing `/implement-story` at the standards files it already had, and adding a check for rules it already stated — not a general rewrite of how implementation works. `Fix CORS` and `Fix architecture completeness gaps` are handled separately in `FW-034`, since those live in `/gen-architecture`, not here.

---

## Implementation Status

Applied immediately during this session — `.claude/commands/implement-story.md` edited directly; no separate tracking needed.

---

## Relationship to Other Decisions

Companion to `FW-034` (Architecture Generation Completeness Gaps), found and fixed in the same investigation. `FW-033` covers what `/implement-story` fails to read or verify; `FW-034` covers what `/gen-architecture` fails to generate or check in the first place. Both were triggered by the same `/verify-story` session on `US-02-001`.
