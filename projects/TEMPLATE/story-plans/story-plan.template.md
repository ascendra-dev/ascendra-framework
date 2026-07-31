# Story Implementation Plan Template

> **[AI Guide — Document Level]**
> A story implementation plan is the reviewable bridge between an approved story and the code that implements it. It is produced by `/gen-story-plan` once a story is `Reviewed` and the architecture is `Locked`, and it must be `Confirmed` by the Product Owner before `/implement-story` will write any code (`FW-031`).
>
> **What this document is:**
> - A scoped extraction of the already-Locked architecture, filtered to exactly what this one story needs — which endpoints, which tables, which screens or jobs, in what order
> - The authoritative task list `/implement-story` executes against
> - The authoritative test plan `/verify-story` executes against (Section 5, Test Scenarios)
> - A record of any deviation discovered during implementation (Section 7)
>
> **What this document is not:**
> - A second architecture document — it never invents an endpoint shape, table column, screen, or job that Section 5/4.2/3.1.3 of `arch-v1.md` doesn't already define. An artifact this story needs that the architecture doesn't define is an architecture gap, flagged in Section 6 (Risks/Open Questions) here — not invented on the spot
> - Pseudocode — it states *what* gets built (shapes, names, order), never *how* (decorator syntax, query style, file boilerplate). The *how* is `/implement-story`'s job, using this project's confirmed stack idiom
> - A one-time document — if `/assess-change` touches architecture mid-sprint, the plan is regenerated (Document Control gets a new row), not patched by hand
>
> **File location:** `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md`

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | Ascendra AI | Initial version |

---

## Part 1 — Template

---

**Story ID:** US-[epic]-[seq]

**Epic:** EPIC-[NNN] — [Title]

**Target:** [API / Web / Worker / API+Web — copied from the story header; must match]

**Architecture version:** arch-v[N].md (Locked on [date])

**Plan Status:** Draft / Confirmed

---

### 1. AC → Artifact Traceability

> **[AI Guide]** Every acceptance criterion from the story's Section 3 gets a row. Name the exact artifact(s) below that satisfy it, using the artifact IDs defined in Sections 2–4. Numbering is a separate counter per artifact type, not one shared sequence — the first endpoint is `API-1`, the first screen is `WEB-1`, the first job is `WORKER-1`, regardless of how many of each this story needs; a second endpoint would be `API-2`, never a continuation of the `WEB` count. An AC with no artifact is a gap — flag it in Section 6, do not leave the row blank.

| AC # | AC Text (short) | Satisfied by |
|------|-----------------|---------------|
| 1 | [short paraphrase] | API-1 |
| 2 | ... | WEB-1 |

---

### 2. API Plan

> **[AI Guide]** Include this section only if Target includes `API`. One block per endpoint. Every field here is *extracted* from the Locked architecture (Section 5 for endpoint/DTO shape, Section 4.2 for schema, Section 6 for auth/RBAC/sensitive fields) — never invented. If this story needs an endpoint or table the architecture doesn't already define, do not invent a shape here — flag it in Section 6 instead.

#### API-1: [short endpoint name]

**Module:** [new / existing] — `src/modules/{module-name}/` (per arch Section 3.1.1)
**Endpoint:** `{METHOD} {path}` (per arch Section 5)
**Auth:** [guard / required role or capability, per arch Section 6.1/6.2 — name it exactly as the architecture names it]
**Request:** [field: type (validation rule), ...]
**Response:** [field: type, ...]
**Schema touched:** [table(s)/column(s), new or existing, per arch Section 4.2]
**Business logic:**
1. [ordered step derived from the AC(s) this endpoint satisfies]
2. ...
**Queries:** [tenant-scoping clause / soft-delete exclusion — only if the architecture defines a tenant boundary]
**Audit / idempotency:** [requirement, if the AC calls for one]
**Sensitive fields excluded:** [per arch Section 6.4, if applicable]
**Files (predicted):** new: [...]; modified: [...]

---

### 3. Web Plan

> **[AI Guide]** Include this section only if Target includes `Web`. One block per screen/section. Route, Surface, and Matched Reference are extracted from `screen-design.md` Section 3 / arch Section 3.1.3.3 and the approved mock file — never a fresh interpretation. Types must match the API Plan's DTO shapes above.

#### WEB-1: [short screen name]

**Route:** [path per arch Section 3.1.2] — [new / existing page]
**Surface:** [Page / Dialog / Sheet / Drawer] — Screen ID [SCR-XXX], Matched Reference [per screen-design.md]
**Components:** [ascendra-ui components used — reused vs newly composed]
**Data layer:** [React Query hook name], [query / mutation], calls [API-N above]
**Types:** [new TS interfaces, matched to API-N's DTO shapes]
**Client validation:** [mirrors API-N's server-side validation rules]
**Navigation:** [new nav item / no change, per arch Section 3.1.3.2]
**Cross-cutting rules:** [gating captions / banners applicable, per arch Section 3.1.3.5, if any]
**Files (predicted):** new: [...]; modified: [...]

---

### 4. Worker Plan

> **[AI Guide]** Include this section only if Target includes `Worker`, and only when architecture Section 3.1 declares a worker repo — a story whose ACs describe scheduled/async job content but whose Section 3.1 declares no worker repo is an architecture gap, flagged in Section 6, not a reason to invent a worker mechanism here. One block per job. Extract everything from the worker's own unnumbered `####` subsection in Section 3.1 and Section 4.2 — never invent a job structure.

#### WORKER-1: [short job name]

**Job type:** [scheduled (cron expression) / event or queue-triggered] — [new / existing]
**Trigger:** [exact schedule or triggering event/queue message, per the worker's Section 3.1 subsection]
**Schema touched:** [table(s)/column(s), if this job reads/writes tenant data — per arch Section 4.2]
**Business logic:**
1. [ordered step derived from the AC(s) this job satisfies]
2. ...
**Idempotency:** [how the job stays safe under retries/at-least-once delivery — required for every job]
**Observability:** [what gets logged at job start/completion/failure — org, entity ID, job name]
**Queries:** [tenant-scoping clause / soft-delete exclusion — same rules as the API Plan, only if the architecture defines a tenant boundary]
**Files (predicted):** new: [...]; modified: [...]

---

### 5. Test Scenarios

> **[AI Guide]** One row per AC. This table is authored now, before implementation exists — `/verify-story` reads it directly instead of deriving scenarios after the fact, so the test plan can't unconsciously bend to match whatever got built.
>
> Derivation rule: happy-path AC → primary success case; boundary/rejection AC → the exact invalid input or unmet precondition; enforcement/idempotency AC → the repeat action and the system's second response; audit/observability AC → the triggering action plus the audit log/event check.

| AC # | AC Text | Test Approach | Input | Expected Result |
|------|---------|---------------|-------|-----------------|
| 1 | [AC text] | API call / UI action / job trigger | [what to send/do] | [what should happen] |

---

### 6. Risks / Open Questions

> **[AI Guide]** Anything the story or architecture left ambiguous, stated explicitly rather than silently resolved. Include any AC from Section 1 that has no clean artifact match, and any artifact this story needs that the architecture doesn't yet define. Write "None." if there is genuinely nothing to flag — do not leave the section missing.

[List, or "None."]

---

### 7. Deviations from Plan

> **[AI Guide]** Blank at generation. `/implement-story` fills this in only if execution must diverge from what this plan declares (a technical constraint the plan didn't anticipate, an assumption that turned out wrong). Each entry: what changed, why, and whether it affects any AC's expected result in Section 5. An undocumented deviation is a defect; a documented one is normal engineering judgment.

[Blank at generation]

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before writing the plan file. A plan that fails any check must be corrected before the PO reviews it.

- [ ] Every AC in the story's Section 3 has a row in Section 1, mapped to at least one artifact
- [ ] Every API Plan / Web Plan / Worker Plan artifact traces back to at least one AC (no orphan artifacts)
- [ ] Every endpoint/table/screen/job named in Section 2/3/4 exists in the Locked architecture or approved mock — nothing invented
- [ ] A Worker Plan (Section 4) is present only if Target includes `Worker` and architecture Section 3.1 declares a worker repo — otherwise omitted, not left as an empty placeholder
- [ ] Section 5 has one Test Scenario row per AC
- [ ] Section 6 is populated (or explicitly "None.") — never missing
- [ ] Target field matches the story header's Target field exactly
- [ ] Plan Status is set to `Draft`
