# UAT Checklist Template

> **[AI Guide — Document Level]**
> The UAT checklist is produced by `/gen-uat-checklist` once every story in a sprint is `Merged`. It is the structured test sheet the Product Owner uses to verify every acceptance criterion before signing off the sprint — written for a non-technical reader, one scenario per acceptance criterion, translated into plain executable steps.
>
> **What this document is:** a sprint-wide, PO-executable test sheet derived directly from story acceptance criteria — one section per story, one row per AC.
>
> **What this document is not:** a QA execution report (that's the separate, optional `test-execution-report.template.md`, aggregated from `/verify-story` reports — a different audience and a different purpose); a defect tracker (failing scenarios are recorded inline in the Sign-off section, not in a persistent defect log); a substitute for `/verify-story` (that already verified these ACs technically — UAT re-verifies them from the Product Owner's own hands, against the real running system).
>
> **File location:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}-uat-checklist.md`
> e.g. `projects/ASCENDRA-PAY-001/sprints/sprint-01-uat-checklist.md`

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-06-20 | Ascendra | Initial version |
| 1.1 | 2026-07-13 | Ascendra AI | Replaced entirely — previous content was an unused, mislabeled "individual test case" artifact (QA Agent role, project-wide `TC-xxx` files in a `tests/` folder) that no command ever read or wrote. This version matches what `/gen-uat-checklist` actually generates. |

---

## Part 1 — Template

---

# UAT Checklist — Sprint [NN] — [Full Project Name]

**Sprint:** [NN]
**Project:** [Project Name]
**Date generated:** [today]
**Completed by (PO):** ___________________
**UAT date:** ___________________

---

### How to use this checklist

> **[AI Guide]** This block is fixed instructional text for the PO — reproduce it exactly, do not paraphrase.

Work through each story section in order. For every test scenario:
1. Follow the Test Steps exactly as written.
2. Observe the result.
3. Mark the Result column: **Pass**, **Fail**, or **N/A** (if the scenario does not apply to your test data).
4. If the result is **Fail**: note what you observed in the Notes column. Do not continue to the next section — raise the defect first.

A failing test stops this sprint from being marked Complete. Every Fail must be resolved (either fixed and re-tested, or formally deferred with a reason recorded).

---

### Sprint Summary

> **[AI Guide]** One row, totals across the whole sprint. Must Have/Should Have counts come from the stories index; Total ACs is the sum of every story's AC count in this sprint.

| Stories in this sprint | Must Have | Should Have | Total ACs | Status |
|----------------------|-----------|-------------|-----------|--------|
| [N] | [N] | [N] | [N] | All Merged ✓ |

---

### [Story ID] — [Story Title]

> **[AI Guide]** One section per story in the sprint, in the same order as the sprint plan's delivery sequence. "What this story delivers" is a plain-language rewrite of the story statement — no "As a [persona]..." format, just what the feature does for the user. One scenario row per acceptance criterion — do not skip, merge, or add scenarios beyond what the story's ACs state.
>
> **Scenario ID format:** `{Story-ID}-TC-{N}` — e.g. story `US-04-003` produces scenarios `US-04-003-TC-1`, `US-04-003-TC-2`, etc. Sequential within the story, not across the project.
>
> **Translating an AC into UAT steps:**
>
> **AC text example:** "The system rejects a payment attempt when the invoice status is not Open."
>
> **UAT test steps:**
> 1. Navigate to the Payments screen.
> 2. Find an invoice that has already been paid (status: Paid).
> 3. Attempt to record a new payment against it.
> 4. Submit the form.
>
> **Expected result:** An error message is displayed. The payment is not recorded. The invoice status remains unchanged.
>
> **Rules for writing test steps:**
> - Plain, numbered steps — no technical jargon, no code, no API calls
> - Each step is a single action ("Click X", "Navigate to Y", "Enter Z")
> - The expected result is observable in the UI or system state — never "the system calls the API"
> - Scheduled job ACs: write the expected result as what the user observes the next business day, not how the job runs
> - Email/notification ACs: write the expected result as what the recipient receives, not how the email is sent

**What this story delivers:** [One sentence, plain language]

| # | Test Scenario | Test Steps | Expected Result | Result | Notes |
|---|--------------|-----------|----------------|--------|-------|
| [Story-ID]-TC-1 | [AC text in plain language] | [Numbered steps] | [What should happen] | Pass / Fail / N/A | |

---

*{Repeat the story section above for every story in the sprint}*

---

### UAT Sign-off

> **[AI Guide]** Leave Overall result, Failing scenarios table, PO Signature, and Sprint approved fields blank at generation — the Product Owner completes these after running the checklist. Do not pre-fill.

All scenarios in this checklist have been executed. Results are recorded above.

**Overall result:** Pass / Fail (circle one)

**Failing scenarios (if any):**

| Scenario ID | Issue description | Severity | Decision |
|------------|------------------|---------|---------|
| | | | Fix before release / Defer to Sprint {N} |

**PO Signature:** _______________________   **Date:** ___________________

**Sprint approved for release:** Yes / No

---

## Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-uat-checklist` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/09-qa-uat.md`'s own tests (the AC-to-scenario translation density test — numbered steps, observable expected results, no technical jargon — and the rest of that chapter's checkable material) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Density & Judgment Resolution below for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this checklist, this section does not appear at all; do not pre-create it empty.

## Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control. Each row records one finding from Density & Judgment Findings' history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this checklist, this section does not appear at all; do not pre-create it empty.
>
> **[AI Guide — numbering note]** This checklist's own sections are unnumbered by convention ("How to use this checklist," "Sprint Summary," "UAT Sign-off" above) — these two headings follow that same unnumbered style rather than the numbered-section convention used elsewhere in this framework. No `/review-uat-checklist` command exists to reserve a conflicting heading here.

---

_Generated via `/gen-uat-checklist` from approved sprint stories. All test scenarios are derived directly from story acceptance criteria._

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before writing this document.

- [ ] Every story in the sprint (per the sprint plan's delivery sequence) has its own section, in that order
- [ ] Every scenario row maps to exactly one acceptance criterion from that story — no AC skipped, no scenario invented beyond the ACs
- [ ] Scenario IDs follow `{Story-ID}-TC-{N}`, sequential within the story
- [ ] Test steps are plain-language, numbered, single-action — no technical jargon or API/code references
- [ ] Every Expected Result is observable (UI state or system state) — never "it works" or a description of internal mechanics
- [ ] Sprint Summary totals (Must Have, Should Have, Total ACs) match the stories index for this sprint
- [ ] UAT Sign-off fields (Overall result, Failing scenarios, Signature, Sprint approved) are left blank for the PO to complete
