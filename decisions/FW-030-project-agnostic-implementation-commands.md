# FW-030 — Project-Agnostic Implementation Commands and Deterministic Story Targeting

| Field | Value |
|-------|-------|
| **ID** | FW-030 |
| **Date** | 2026-07-15 |
| **Status** | Decided |
| **Area** | `implement-story.md`, `verify-story.md`, `gen-stories.md`, `review-stories.md`, `story.template.md`, `epic.template.md` |
| **Extends** | `FW-029` (Stack-Agnostic Implementation Commands) |

---

## Decision

`FW-029` made implementation commands agnostic on the **technology axis** — no framework's syntax stated as universal. This decision extends the same rule to the **project/domain axis**, and hardens three run-time behaviours found weak in a critical review of `implement-story.md` on 2026-07-15:

1. **No project-specific domain content in framework commands.** Guard names, role names, capability names, tenancy rules, and data-representation standards (e.g. money as integer minor units) belong to a project's architecture document — a framework command may cite them only as conditioned examples ("if Section 6.x defines…", "e.g. where the architecture stores money as…"), never as unconditional rules.
2. **Story targeting is decided at generation time, not inferred at implementation time.** The story template gains a required `**Target:**` header field (`API` / `Web` / `Worker` / `+`-joined combinations in dependency order). `/gen-stories` populates it from the Locked architecture's Section 3.1 repo topology (which it already gates on); `/review-stories` verifies it against the ACs; `/implement-story` reads it as authoritative, keeping content-based derivation only as a fallback for pre-FW-030 stories and as a cross-check.
3. **Repo existence is structural, not nominal.** `/implement-story` no longer treats a directory merely existing as proof of a completed scaffold — it checks for `.git`, the confirmed stack's manifest, and the Section 3.1 layout, and stops on a partial scaffold rather than building on it or silently re-scaffolding.

---

## Rationale

Found 2026-07-15 during a critical review of `implement-story.md` (still never run against a real story). The FW-029 pass fixed stack leakage but missed a second leakage axis: the command stated ASCENDRA-PAY-specific content as framework law — `PayerJwtAuthGuard` / `SuperAdminJwtAuthGuard` guard names, the "Support Access Grant" / "Internal Reviewer" delegated-access vocabulary, unconditional `orgId` multi-tenant scoping, and money-as-`@IsInt()`. None of these terms exist in `arch.template.md`; they exist only in one project's architecture. Running the command against a second project risked importing the first project's security model into it. The story template carried the same class of leak (`Layer: Core / Ext:PK / Ext:School`), now genericised to `Ext:[extension-name]`.

The same review found three run-time weaknesses: target repo was inferred from story prose with no structured contract despite `/gen-stories` already running against a Locked architecture; repo existence was checked with a bare `ls`, so an interrupted scaffold would be built on blind; and two bootstrap branches had no path for a confirmed Next.js frontend *without* Ascendra UI (a combination `/gen-architecture` explicitly permits and records as a Section 9 deviation). A missing `arch-v1-ref.md` also had no stated behaviour — now an explicit fall-back to the full document with a recommendation to regenerate via `/review-architecture`.

---

## What changes

- `implement-story.md`: Section 6.2/6.3 rules rephrased as "apply what this project's architecture defines" with the old names demoted to conditioned examples; tenant scoping and money typing conditioned on the architecture defining them; Step 3 reads the story's `Target` field first (content derivation kept as pre-FW-030 fallback); repo-exists check made structural with a stop-and-report on partial scaffolds; new bootstrap branch for Next.js without Ascendra UI; `npx @nestjs/cli` fallback for a missing global Nest CLI; explicit fallback when `arch-v1-ref.md` is missing.
- `story.template.md`: `Target` header field added with AI Guide; Layer placeholder genericised; verification checklist gains a Target item (now 15 items).
- `gen-stories.md`: header spec derives and populates `Target` from Section 3.1; checklist count updated to 15; stories index registry gains a `Target` column.
- `review-stories.md`: check item 10 verifies Target against Section 3.1 and against what the ACs describe.
- `verify-story.md` (audited same day — no domain leaks found, four alignment gaps fixed): reads the story's `Target` field and must verify every listed surface, not just one; same explicit fallback as `implement-story.md` when `arch-v1-ref.md` is missing; API tests start the app per the repo README and obtain role JWTs via the confirmed auth endpoint with Section 4.4 seeded users, never fabricated tokens; the job-trigger HTTP example is conditioned on the architecture actually defining such an endpoint, with a state-the-gap rule when no manual invocation mechanism is documented.
- `epic.template.md`: Layer value set genericised from `Ext:PK / Ext:School` to `Ext:[extension-name]` (same leak class).

## When authoring or editing any command

Both agnosticism axes apply together: a command may use this framework's home project as a worked example on either axis (stack or domain), but every such reference must be conditioned and paired with the generic rule the example illustrates. Anything decidable from an artifact that already exists at authoring time (a Locked architecture, an approved BRD) is stamped into the artifact at generation time — never re-inferred downstream from prose.
