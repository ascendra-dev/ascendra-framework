# Test Execution Report Template

> **[AI Guide — Document Level]**
> The test execution report is a sprint-wide QA summary, synthesized in conversation with the Product Owner from every story's `/verify-story` report for this sprint (`projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`). It is optional — Step 23 (Sprint-wide QA Confirmation) is a manual checkpoint with no dedicated slash command; most sprints only need the one-line confirmation recorded in the sprint plan's Sprint Notes section. Generate this fuller report only when the sprint needs a formal, client-facing QA record. It is the gate artifact for UAT when used — the Product Owner reviews it before approving UAT to begin. Nothing enters UAT with an open Critical defect.
>
> **One report per sprint.** The report covers every story in the sprint, aggregated from their individual `/verify-story` reports — it does not re-run or re-derive test results itself.
>
> **What this document is:** a sprint-level QA summary — what was tested, what passed, what failed, what is blocked, and whether the sprint is ready for UAT.
>
> **What this document is not:** a substitute for the per-story `/verify-story` reports (this aggregates them, it doesn't replace them), a persistent defect-tracking system (defects live only in the per-story reports and this aggregation — there is no separate defect log file), or a UAT sign-off document (UAT sign-off is a separate artifact by the Product Owner after UAT runs).
>
> **File location:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}-execution-report.md`
> e.g. `projects/ASCENDRA-PAY-001/sprints/sprint-01-execution-report.md`

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-06-20 | Ascendra | Initial version |
| 1.1 | 2026-06-20 | Ascendra | Structural alignment with template standards; moved to knowledge/product/ |
| 1.2 | 2026-07-13 | Ascendra AI | Removed fictional "QA Agent" role and TC-ID/`tests/`-folder/defect-log apparatus that don't exist in this framework; re-grounded the whole document in the actual source of truth — per-story `/verify-story` reports in `test-reports/`. Fixed `{PROJECT_CODE}` placeholder and sprint filename convention. |

---

## Part 1 — Template

---

**Project:** [Project name]
**Sprint:** Sprint [NN]
**Execution Date:** [YYYY-MM-DD]
**BRD Version:** [e.g. v1.0 — the BRD version this sprint was derived from]
**Sprint Plan:** `sprints/sprint-{NN}.md`

> **[AI Guide]** Execution Date is the date every story's `/verify-story` report for this sprint was complete and this report is ready for Product Owner review. Sprint Plan is a relative link so the reviewer can navigate to the approved scope without searching the file system.

---

### 1. Test Summary

> **[AI Guide]** Populate this table by summing across every story's `/verify-story` report for this sprint — one row of "ACs tested/passed/failed" per report, added up here. Counts must add up: Passed + Failed + Blocked + Not Run = Total.
>
> **Pass Rate** = (Passed ÷ Total) × 100, rounded to one decimal place.
>
> **Capacity status guide:**
> - ≥ 80% pass rate: within the UAT threshold — proceed to QA sign-off checklist
> - 70–79%: borderline — review blockers; consider whether they are resolvable before submission
> - < 70%: below threshold — do not submit. Re-run failed tests (`/verify-story`) after defect fixes, then resubmit
>
> **Not Run:** ACs that have no `/verify-story` report yet for this sprint (e.g. the story was descoped after plan lock, or verification hasn't been run). Not Run is not the same as Blocked — Blocked means verification was attempted and could not complete due to an environment or data issue.

| Metric | Value |
|--------|-------|
| Total ACs | [N] |
| Passed | [N] |
| Failed | [N] |
| Blocked | [N] |
| Not Run | [N] |
| Pass Rate | [N]% |

---

### 2. Story Sign-off

> **[AI Guide]** One row per story in the sprint. AC counts come directly from that story's `/verify-story` report's Summary section — read each report before filling this table.
>
> **Decision options:**
> - **Pass** — all ACs for this story passed per its `/verify-story` report
> - **Conditional Pass** — ACs passed but one or more Medium or Low defects were found. The story can enter UAT; the defects are carried in Section 3 below for PO awareness. Requires PO sign-off before UAT begins
> - **Fail** — one or more Critical or High defects found. Story cannot enter UAT until defects are fixed and re-verified (`/verify-story` re-run)
> - **Blocked** — verification could not be completed due to environment or data issues. Story is not cleared for UAT; blocker must be resolved
>
> **Always record a Decision.** A story cannot be left with no decision at report time — if you cannot decide because information is missing, it is Blocked, not pending.

| Story ID | Title | ACs Total | Passed | Failed | Blocked | Decision |
|----------|-------|-----------|--------|--------|---------|----------|
| [US-xx-xxx] | [Story title] | [N] | [N] | [N] | [N] | Pass / Conditional Pass / Fail / Blocked |

---

### 3. Defects Found

> **[AI Guide]** One row per defect, pulled directly from the Defect Detail section of each story's `/verify-story` report — do not re-derive or re-describe, copy the classification already recorded there. Log every defect carried over — do not omit Low severity ones. If a story has zero defects, omit the row rather than writing "None."
>
> **Defect ID:** use the story's own defect numbering from its `/verify-story` report (e.g. `DEF-1`), qualified by Story ID in the table below to avoid collisions across stories — there is no separate project-wide defect sequence or persistent defect log; this table is the only place defects are aggregated across stories.
>
> **Severity definitions** (same as `reference/defect-severity.md`, used by `/verify-story`):
> - Critical — system crash, data loss, or security breach. Blocks UAT. Must be fixed and re-verified before any UAT begins
> - High — core feature does not work, acceptance criterion completely unmet. Blocks UAT for the affected story. Must be fixed or explicitly deferred by PO
> - Medium — feature partially works; workaround exists. Conditional Pass permitted
> - Low — cosmetic or minor UX issue. Does not block UAT. Fixed in a future sprint
>
> **Status options:** Open / Fixed / Deferred (PO decision) / Won't Fix

| Story | Defect ID | AC # | Description | Severity | Status |
|-------|-----------|------|-------------|----------|--------|
| US-xx-xxx | DEF-x | [AC #] | [One sentence: what failed and what was observed — copied from the story's `/verify-story` report] | Critical / High / Medium / Low | Open / Fixed / Deferred / Won't Fix |

If no defects were found in this sprint, write: **No defects found.**

---

### 4. Blocked Items

> **[AI Guide]** List every AC that could not be verified due to an environment or data issue, per the affected story's `/verify-story` report. A blocked item is not a defect — verification failed to run, not the feature itself.
>
> **Common blockers:**
> - Test/dev environment unreachable or in a broken state
> - Required seed data could not be created (e.g. a dependency story was not merged in time)
> - Third-party API credentials not available in the environment
> - A prerequisite AC (noted in the story's `/verify-story` report) failed in a way that makes subsequent ACs meaningless to test
>
> **Every blocked item must have a named Owner and a Resolution Target date.** An unowned blocker is a blocker that will never be resolved.
>
> If nothing was blocked, write: **No blocked items.**

| Blocked Item | Story Affected | Reason | Owner | Resolution Target |
|--------------|----------------|--------|-------|-------------------|
| [AC # — description] | US-xx-xxx | [Why verification could not run] | [Named person or role] | [YYYY-MM-DD] |

---

### 5. QA Sign-off Checklist

> **[AI Guide]** Complete every item before submitting this report to the Product Owner — this checklist gates submission; the report must not be submitted with any unchecked item. If an item cannot be checked, explain why in the relevant section (Defects Found or Blocked Items) and update the Decision for the affected story.
>
> The only exception is Item 3 (zero High defects): a High defect can be overridden to Deferred status only by explicit Product Owner decision — record it in the Defects Found table with Status "Deferred" and a note of who made the decision and when.

- [ ] Pass Rate is ≥ 80%
- [ ] No Critical defects are Open
- [ ] No High defects are Open (or all High defects have been explicitly Deferred by the Product Owner — confirm in Defects Found table)
- [ ] Every blocked item has a named Owner and a Resolution Target date
- [ ] Every story in the sprint has a recorded Decision in Section 2 — no story left pending

**Sign-off Recommendation:**

> **[AI Guide]** After completing the checklist, write one sentence: "This sprint is ready for UAT." or "This sprint is not ready for UAT — [reason]." The Product Owner reads this first. Do not hedge or qualify. If all checklist items are met, the sprint is ready.

[One-sentence recommendation]

---

### 6. Approval

> **[AI Guide]** Leave Decision and Date blank. The Product Owner completes this. Do not pre-fill. Status transitions to UAT only after the Product Owner signs off here.

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | [Name] | Approved for UAT / Request Changes / Rejected | |

---

### 7. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled when this report is generated, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/09-qa-uat.md`'s own tests (the corrected Deferred Defect Process — condition 3's requirement for explicit, individually-recorded Product Owner agreement on each High defect, never a blanket sign-off — and the trigger test for when this full report is warranted vs. a one-line sprint-plan confirmation) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 8 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this report, this section does not appear at all; do not pre-create it empty.

### 8. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control. Each row records one finding from Section 7's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this report, this section does not appear at all; do not pre-create it empty.
>
> **[AI Guide — numbering note]** Sections 7 and 8 are reserved for `/judgment-check` as shown here, immediately after Section 6 (Approval) and before Part 2 (Verification). No `/review-test-execution-report` command exists in this framework, so there is no other command reserving a conflicting section number here.

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before submitting this report. A report that fails any check will produce an incorrect Product Owner decision.

- [ ] Total in Section 1 equals the sum of ACs tested across every story's `/verify-story` report for this sprint — no story's report missed
- [ ] Passed + Failed + Blocked + Not Run = Total — counts are internally consistent
- [ ] Pass Rate formula is correct: Passed ÷ Total × 100
- [ ] Every story in the sprint plan appears in Section 2 — no story omitted
- [ ] Story AC counts in Section 2 match that story's own `/verify-story` report Summary
- [ ] Every defect in Section 3 is copied from a story's `/verify-story` report (Story + Defect ID uniquely identifies it) and has a recorded Status
- [ ] Severity assignments match what `/verify-story` already recorded — this report does not reclassify
- [ ] Every item in Section 4 has a named Owner (not "TBD") and a Resolution Target date
- [ ] QA Sign-off Checklist in Section 5 is fully completed — no unchecked boxes without explanation
- [ ] Sign-off Recommendation in Section 5 is a single, unambiguous sentence

