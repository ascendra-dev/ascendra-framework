# Generate Story Implementation Plan

You are producing the implementation plan for exactly one user story — the reviewable bridge between an approved story and the code that will implement it. This plan states *what* gets built (which endpoints, tables, screens, or jobs, in what order, tested how) by extracting it from the already-Locked architecture and the story's own acceptance criteria. It never invents new architecture, and it never states *how* to build anything in stack-specific syntax — that remains `/implement-story`'s job. Per `FW-031`, `/implement-story` will not run against this story until the plan this command produces is `Confirmed`.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **Story file path** — required, e.g. `projects/ASCENDRA-PAY-001/stories/EPIC-001/US-01-001-grant-baseline-issuer-staff-access.md`

Derive from the story path:
- **PROJECT_CODE** — second segment of the path
- **Story ID** (e.g. `US-01-001`)
- **Architecture path** — `projects/{PROJECT_CODE}/architecture/arch-v1.md`

If the story file path is missing, ask:
> "Which story should a plan be produced for? Provide the path (e.g. `projects/ASCENDRA-PAY-001/stories/EPIC-001/US-01-001-grant-baseline-issuer-staff-access.md`)."

Wait for PO response.

Read the story file in full. Extract the **Target** field from the header.
- If present → use it (`API` / `Web` / `Worker` / a `+`-joined combination).
- If missing (a pre-`FW-030` story): derive it from the story's acceptance criteria using the same rule `/implement-story` uses as its fallback (backend/schema/endpoint content → `API`; frontend page/screen/component content → `Web`; scheduled/async job content → `Worker`; more than one → combine with `+`). Write the derived value back into the story file's header (inserting `**Target:**` after the `**Layer:**` line) so it becomes the permanent record, and state explicitly that you did this.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Story is Reviewed or later | `**Status:**` in the story header is `Reviewed`, `In Progress`, `PR Created`, `Merged`, or `Done` (a plan may be regenerated after a mid-sprint architecture change) | "Story `{ID}` has status `Draft`. Stories must pass review before a plan can be produced. Run `/review-stories` and then `update-status {story-path} Reviewed`." |
| Story is not Deprecated | `**Status:**` is not `Deprecated` | "Story `{ID}` is Deprecated and should not be planned or implemented." |
| Architecture is Locked | `projects/{PROJECT_CODE}/architecture/arch-v1.md`'s `**Status:**` is `Locked` | "Architecture is not yet Locked (current status: `{status}`). A story plan extracts from Locked architecture only — it cannot be produced against a document still in flux. Run `/review-architecture {PROJECT_CODE}`." |

---

## Step 2 — Read reference files

1. **Architecture quick-reference (read first):** `projects/{PROJECT_CODE}/architecture/arch-v1-ref.md`. If it does not exist (architecture locked before the ref file was generated), state that explicitly, read the full `arch-v1.md` as the primary source for this run, and recommend regenerating the ref file via `/review-architecture`.
2. **Architecture document (fallback):** `projects/{PROJECT_CODE}/architecture/arch-v1.md` — read the full document for: exact endpoint/DTO field shapes not shown in the ref file, table/column definitions (Section 4.2), enum values (Section 4.3), auth/RBAC/delegated-access/sensitive-field detail (Section 6), screen/navigation detail (Section 3.1.2/3.1.3) for Web-target stories, worker job structure (Section 3.1's unnumbered `####` worker subsection) for Worker-target stories. When falling back, state explicitly which section you're reading and why.
3. **Open Decisions check (Section 8):** if this story's capability area has an open, unresolved decision recorded in Section 8, stop and flag it:
   > "Architecture Section 8 has an open decision affecting this story: '{decision text}'. Planning now risks scoping against a choice that may still change. Resolve it first, or confirm you want a plan built against a stated assumption."
   Wait for PO response before proceeding on override.
4. **Domain knowledge** — any file referenced in the story's Notes section or its parent epic.
5. **Screen design and approved mock** (Web-target stories only): find this story's screen(s) in `projects/{PROJECT_CODE}/screens/screen-design.md` Section 3 (or architecture Section 3.1.3.3), read Surface/UI Pattern/Matched Reference, then open the actual approved mock file at `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html` and locate the real markup. This is the visual ground truth the Web Plan section extracts from — not a fresh interpretation.

---

## Step 3 — Build the AC → Artifact Traceability table

For each acceptance criterion in the story's Section 3, identify exactly which artifact(s) satisfy it, using the IDs you define in Steps 4–6. Each artifact type has its own independent counter — the first endpoint is `API-1`, the first screen is `WEB-1`, the first job is `WORKER-1`, whether or not the others exist in this story — never one shared sequence across types (a second endpoint is `API-2`, not a continuation of `WEB`'s count). An AC that doesn't cleanly map to anything you can define yet is not a silent gap — carry it into Step 8 (Risks / Open Questions) instead of forcing a match.

---

## Step 4 — Build the API Plan (if Target includes `API`)

For each endpoint this story needs, extract — never invent:

- **Module and schema** from arch Section 3.1.1 / 4.2: is this a new module/table, or does it extend an existing one already defined there?
- **Endpoint shape** from arch Section 5: method, path, auth requirement, request/response fields. If this story requires an endpoint or field the architecture doesn't already define anywhere in Section 5, that is an architecture gap, not a planning decision — do not invent a shape for it. Record it in Step 8 (Risks / Open Questions) and note that `/assess-change` may be needed if this is a genuine scope gap.
- **Business logic** as ordered, plain-language steps derived from the AC(s) this endpoint satisfies — describe the sequence of what happens, not code.
- **Query notes**: tenant-scoping and soft-delete requirements, only if arch Section 6.2 defines a tenant/organisation boundary for this project.
- **Audit/idempotency requirements** the ACs call for.
- **Sensitive-field exclusions** from Section 6.4, if this endpoint touches any listed field.
- **Delegated-access implications** from Section 6.3, only if this project defines such a mechanism and this endpoint is reachable under a delegated session.
- **Predicted file paths**, following Section 3.1.1's naming convention — stated as a prediction `/implement-story` may adjust, not a guarantee.

---

## Step 5 — Build the Web Plan (if Target includes `Web`)

For each screen/section this story needs, extract — never invent:

- **Route and surface** from Screen Design Section 3 / arch Section 3.1.3.3 and the approved mock: exact folder path, Page/Dialog/Sheet/Drawer, Screen ID, Matched Reference.
- **Components** from the actual mock markup — which `ascendra-ui` components appear, reused vs newly composed.
- **Data layer**: how this screen fetches/mutates data per the confirmed State Management / Data Fetching choice (`system-architecture.md` Step 5) — if it uses hooks (e.g. React Query, SWR), name the hook from `hooks/{domain}.hook.ts` (`FW-044`) and whether it's a query or mutation; if the confirmed approach is hookless (e.g. Server Components only), describe the actual fetch mechanism instead. Never assume React Query if a different (or no) approach was confirmed. Either way, name which `API-N` block above it calls.
- **Types**: new TypeScript interfaces matched exactly to the corresponding `API-N` block's request/response shapes — this is where API/Web shape drift gets caught before either side is coded.
- **Client-side validation** mirroring the corresponding `API-N` block's validation rules.
- **Navigation changes** from arch Section 3.1.3.2, if this story adds or changes a nav item.
- **Cross-cutting UI rules** from Section 3.1.3.5 (gating captions, banners) applicable to this screen, if any.
- **Predicted file paths**, following Section 3.1.2's convention.

---

## Step 6 — Build the Worker Plan (if Target includes `Worker`)

Only applies when architecture Section 3.1 declares a worker repo — if a story's ACs describe scheduled/async job content but Section 3.1 declares no worker repo, that is an architecture gap: record it in Step 8, do not invent a worker mechanism inline in the API plan. For each job this story needs, extract — never invent:

- **Job type and trigger** from the worker's own unnumbered `####` subsection in Section 3.1: scheduled (cron expression) or event/queue-triggered, new job or existing one extended.
- **Schema touched**, if the job reads/writes tenant data — same source (Section 4.2) and same tenant-scoping/soft-delete rules as the API Plan, since a worker is not exempt from them.
- **Business logic** as ordered, plain-language steps derived from the AC(s) this job satisfies.
- **Idempotency**: how the job stays safe under retries/at-least-once delivery — every job needs this, per the story's own enforcement ACs if present.
- **Observability**: what gets logged at job start, completion, and failure (org, entity ID, job name) — background failures have no user-facing error to surface otherwise.
- **Predicted file paths**, following the worker subsection's documented layout.

---

## Step 7 — Build the Test Scenarios table

One row per AC, following the same derivation rules `/verify-story` currently applies at test time — moved here so the test plan is authored before implementation exists, not derived afterward:
- Happy-path AC → the primary success case
- Boundary/rejection AC → the exact invalid input or unmet precondition
- Enforcement/idempotency AC → the repeat action and the system's expected second response
- Audit/observability AC → the triggering action plus the audit log/event check

---

## Step 8 — Risks / Open Questions

State explicitly anything the story or architecture left ambiguous: any AC from Step 3 with no clean artifact match, any artifact needed that the architecture doesn't define, any assumption you had to make to produce Steps 4–7. Write "None." if there is genuinely nothing to flag.

---

## Step 9 — Present the plan summary and wait for confirmation

Before writing any file, present a summary table and wait for confirmation:

| AC # | Satisfied by | Artifact summary |
|------|--------------|-------------------|
| 1 | API-1 | [one-line description] |
| 2 | WEB-1 | [one-line description] |

> "Does this plan structure look right? Any adjustments before I write the file?"

Wait for PO response. Do not write any files until confirmed.

---

## Step 10 — Write the plan file

Follow `projects/TEMPLATE/story-plans/story-plan.template.md` exactly. Remove `[AI Guide]` notes and the `## Part 1 — Template` label from the output, same convention as every other generated artifact. Set **Plan Status:** `Draft` — the summary confirmation in Step 9 is a quick sanity check on structure, not the formal PO review; that happens against the written file via `update-status ... Confirmed`.

**Output path:** `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md`

Create the `story-plans/` directory if it does not exist.

If a plan already exists for this story (regenerating after `/assess-change` touched architecture, or the sprint mid-course), add a new row to the Document Control table describing what changed, and reset **Plan Status** to `Draft` — a Confirmed plan does not survive a regeneration silently.

---

## Step 11 — Report to the user

```
STORY PLAN GENERATED — {Story ID}: {Story Title}
─────────────────────────────────────────
Target:        {API / Web / Worker / API+Web / other combination}
Plan file:      projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md
API artifacts:  {count} (or "N/A — Target does not include API")
Web artifacts:  {count} (or "N/A — Target does not include Web")
Worker artifacts: {count} (or "N/A — Target does not include Worker")
Risks flagged:  {count} (or "None")
─────────────────────────────────────────
AC coverage:
  AC1 → {artifact(s)}
  AC2 → {artifact(s)}
─────────────────────────────────────────
Next step:
Review the plan above — this is the task list /implement-story will execute against
and the test plan /verify-story will execute against. When satisfied, confirm it:

  update-status projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md Confirmed

/implement-story will not proceed against this story until the plan is Confirmed.
```

Do not set Plan Status to `Confirmed` yourself. That remains a Product Owner decision via `/update-status`.
