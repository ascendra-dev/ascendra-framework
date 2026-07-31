# Verify Story

You are verifying a user story's acceptance criteria against the running application. For each acceptance criterion, you derive a test scenario, execute it against the running API or UI, and record the result. This is the Engineering Agent's own quality check — it runs before the PR is raised, not instead of PO review.

The output is a pass/fail record against every AC, with evidence. The story status is not updated by this command — that remains a PO decision.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain the story file path. Example:

```
/verify-story projects/ASCENDRA-PAY/stories/EPIC-001/US-01-003-core-schema.md
```

Extract:
- **Story file path**
- **Story ID** (e.g. `US-01-003`)
- **PROJECT_CODE** (from file path, e.g. `ASCENDRA-PAY`)

---

## Step 2 — Gate check

Read the story file. Check the `**Status:**` field in the story header.

- If the story is `In Progress` or `PR Created` → proceed
- If the story is `Draft` or `Reviewed` → stop:
  > "Story `{ID}` has status `{status}`. Story must be `In Progress` before verification can run. Implement the story first using `/implement-story`."
- If the story is `Merged` or `Done` → warn and ask:
  > "Story `{ID}` is already `{status}`. Run verification anyway? (This may be useful to confirm a regression has not been introduced.)"

---

## Step 3 — Read required files

Read these files before running any tests:

1. The story file in full — you need Section 3 (Acceptance Criteria) and Section 7 (Notes) for test context, plus the header's `**Target:**` field: it names every surface this story's code landed in (`API` / `Web` / `Worker` / `+`-joined combinations), and verification must cover each listed surface with the matching test approach in Step 5 — a story targeting `API+Web` is not verified by API calls alone. For stories generated before the Target field existed, derive the surfaces from the ACs and state the derivation.
2. `projects/{PROJECT_CODE}/standards/test-strategy.md` — project test strategy: what to test, test evidence standards, tooling, coverage targets. If this file does not exist, apply general verification principles: test the happy path, boundary cases, and rejection cases for each AC using whatever test runner the project uses.
3. `reference/defect-severity.md` — defect classification for Step 6
4. **Architecture quick-reference (read first):** `projects/{PROJECT_CODE}/architecture/arch-v1-ref.md` — endpoint paths, auth guards/roles, table/enum names. Covers the vast majority of what verification needs. Read it before opening the full architecture document. **If `arch-v1-ref.md` does not exist** (architecture locked before the ref file was generated): state that explicitly, read the full `arch-v1.md` as the primary source for this run, and recommend regenerating the ref file via `/review-architecture` before the next story. Do not silently proceed as if the ref file had been read.
5. **Architecture document (read only when the ref file is insufficient):** `projects/{PROJECT_CODE}/architecture/arch-v1.md` — read the full document only for: exact response body field shapes not shown in the ref file's abbreviated Tables/Endpoints sections; business-rule detail behind an AC's expected result (Sections 4/5/6 prose); seed data values a test depends on (Section 4.4). When falling back, state explicitly: *"Reading full arch-v1.md for [specific detail] not covered by ref file"* — do not silently fall back. If a detail appears in neither file, state the gap rather than inventing an expected result.
6. **Story implementation plan (`FW-031`):** `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md` — Section 5 (Test Scenarios) is the primary source for Step 4 below; Section 7 (Deviations from Plan) feeds the Plan Conformance check in Step 6. If no plan file exists (a story implemented before `FW-031`), state that explicitly and fall back to deriving scenarios on the fly per Step 4's rules, with no Plan Conformance section in the report.

---

## Step 4 — Read (or derive) test scenarios

**Primary path:** read the Test Scenarios table directly from the story's plan (Section 5) — it was authored before implementation existed, so it can't have unconsciously bent to match whatever got built. Use it as-is; do not re-derive.

**Fallback — only if no plan file exists:** derive a test scenario for each acceptance criterion in Section 3 of the story file:

| AC # | AC text | Test approach | Input | Expected result |
|------|---------|--------------|-------|----------------|
| 1 | [AC text] | API call / UI action / automated job trigger | [what to send/do] | [what should happen] |

Apply these rules when deriving scenarios:
- Happy path AC → test the primary success case
- Boundary/rejection AC → test with the exact invalid input or unmet precondition
- Enforcement/idempotency AC → test the repeat action; verify the system's second response
- Audit/observability AC → test the triggering action; check the audit log or event record

---

## Step 5 — Execute the tests

For each test scenario:

### API story tests

If the API is not already running, start it from the target repo per its own README (services up, migrations applied, seed run, dev server started) — never test against a stale or unmigrated instance. Obtain role JWTs the way this project's architecture actually provides them: log in via the confirmed auth endpoint (Section 5/6.1) using seeded users from Section 4.4 — never fabricate or hand-craft a token.

Execute against this project's own confirmed base path (Section 5 of the architecture doc — e.g. `/api/v1/`, never assume this literally without checking):

```bash
# Example: verify an endpoint returns the expected response
curl -s -X {METHOD} \
  -H "Authorization: Bearer {valid-jwt-for-role}" \
  -H "Content-Type: application/json" \
  -d '{request-body}' \
  http://localhost:{port}/{confirmed-base-path}/{endpoint-path}
```

Record:
- HTTP status code received
- Response body (relevant fields)
- Whether it matches the expected result

For auth-related ACs: test with both an authorised role (expect success) and an unauthorised role (expect 403 Forbidden).

For state-change ACs: query the resource before and after the action and confirm the state transitioned correctly.

### Web story tests

Start the dev server if not running. Navigate to the relevant route. Perform the action described in the AC and observe the result.

Record:
- What appeared on screen / what changed
- Whether it matches the expected result

### Automated job tests

Trigger the job manually, seeding appropriate test data first. Use whatever invocation mechanism this project's architecture actually documents for jobs — an internal HTTP trigger endpoint, a CLI/script entry point, a queue message, or advancing the scheduler. Do not assume an HTTP trigger route exists unless Section 5 (or the worker's own Section 3.1 subsection) defines one; if the architecture documents no manual invocation mechanism, state that gap rather than inventing an endpoint.

```bash
# Example — only if the architecture defines an HTTP trigger endpoint:
curl -X POST http://localhost:{port}/{confirmed-base-path}/internal/jobs/{job-name}/trigger
```

Record:
- Job execution response
- State of affected records after job run
- Whether it matches the expected result

---

## Step 6 — Record defects

If any test returns an unexpected result:

1. **Classify the defect** using `reference/defect-severity.md`
2. Record in the verification report: AC number, expected result, actual result, severity
3. Do not continue to the next test if a Critical or High defect is found — stop, record, and report immediately

---

## Step 7 — Write the verification report

Write a verification report to `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`.

Create the `test-reports/` directory if it does not exist.

Report structure:

```markdown
# Story Verification Report — {Story ID}: {Story Title}

**Story:** {Story ID} — {Title}
**Status at time of test:** {In Progress / PR Created}
**Date:** {today}
**Verified by:** Engineering Agent

---

## Summary

**Overall result:** PASS / FAIL
**ACs tested:** {N}
**ACs passed:** {N}
**ACs failed:** {N}
**Defects found:** {N} ({count by severity: N Critical, N High, N Medium, N Low})

---

## AC Results

| AC # | AC Text | Test | Expected | Actual | Result |
|------|---------|------|---------|--------|--------|
| 1 | {AC text} | {what was done} | {expected} | {actual} | PASS / FAIL |
| 2 | ... | | | | |

---

## Defect Detail

{For each failed AC:}

**DEF-{N}:** {AC number} — {one-line description}
- **Severity:** {Critical / High / Medium / Low}
- **Observed:** {what actually happened}
- **Expected:** {what should have happened}
- **Test input used:** {request body / action taken}

{If no defects: "No defects found."}

---

## Test Evidence

{Paste or summarise key response bodies, state-before/after comparisons, or log entries that provide evidence of test execution. This section is what the PO uses to trust the report.}

---

## Plan Conformance

{Only if a story plan exists — omit this section entirely on the fallback path.}

| Planned artifact | Declared in plan | Actually delivered | Status |
|-------------------|-------------------|---------------------|--------|
| API-1 | {endpoint/table from plan Section 2} | {what exists in the repo} | Match / Deviation / Missing |
| WEB-1 | {screen/component from plan Section 3} | {what exists in the repo} | Match / Deviation / Missing |

**Deviations recorded in the plan (Section 7):** {list each, with its stated rationale — a documented deviation with rationale is not a defect} / "None recorded."

**Undeclared deviations found:** {any artifact that was built differently than planned with no corresponding Section 7 entry — this is a defect, classify and record it above under Defect Detail} / "None found."
```

---

## Step 8 — Report to the user

After writing the report:

**If all ACs passed:**
> "Story `{ID}` verified — all {N} ACs passed.
>
> Verification report written to `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`
>
> **Manual step:** Raise the PR for this story. Story status updates remain a PO decision via `/update-status`."

**If any AC failed:**
> "Story `{ID}` has {N} failing ACs. Verification report written to `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`
>
> **Failing ACs:** [list]
> **Defects:** [list with severity]
>
> Resolve these before raising the PR. Re-run `/verify-story` after fixing."
