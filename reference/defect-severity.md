# Defect Severity

## Purpose
Defines the four severity levels for defects found during testing, with examples and SLA targets for resolution. `/verify-story` uses this to classify every defect it logs; the Product Owner uses it to prioritise fixes and decide on deferrals.

---

## Severity Levels

### Critical
**Definition:** The system is unusable, data is at risk, or a security vulnerability exists. No workaround is possible.

**Examples:**
- User cannot log in
- Data is lost or corrupted on save
- Payment processes for the wrong amount
- A security control is bypassed (unauthorised access to another user's data)
- System crashes or returns 500 on a core user journey

**SLA:** Fix within 4 hours. Hotfix deployed to production same day. PO sign-off required before hotfix goes live.

**Effect on release:** Blocks UAT sign-off. Blocks production deployment. No exceptions.

---

### High
**Definition:** A core feature is broken and there is no viable workaround. The user cannot complete a primary task.

**Examples:**
- Cannot create, edit, or delete a primary record (invoice, customer, order)
- A filter or search returns consistently wrong results
- A required form field fails to save correctly
- Email notifications are not sent
- Role-based access control is not enforced on a screen

**SLA:** Fix within 1 business day.

**Effect on release:** Blocks UAT sign-off until resolved or formally deferred with Product Owner agreement.

---

### Medium
**Definition:** A feature is degraded. The user can still accomplish their goal via a workaround, but the experience is impaired.

**Examples:**
- Export to PDF/CSV fails, but data is visible on screen
- Sorting or pagination behaves incorrectly in some cases
- A non-critical calculation is off (e.g., a totals row on a report)
- A form field does not retain its value after a validation error
- A success message does not appear after a successful action

**SLA:** Fix within the current sprint.

**Effect on release:** Does not block UAT sign-off. Must be documented in the test execution report, if one is produced for this sprint (see `test-execution-report.template.md`).

---

### Low
**Definition:** Cosmetic issue or minor inconvenience. No functional impact.

**Examples:**
- A label is misaligned or truncated
- The wrong date format is displayed (e.g., MM/DD instead of DD/MM)
- A tooltip contains a typo
- An icon is missing or incorrect
- Spacing or colour is inconsistent with the design

**SLA:** Fix in the next sprint or place in backlog.

**Effect on release:** Does not block any gate.

---

## Classification Rules

- Severity is set at the time of logging — by `/verify-story` during implementation, or by the Product Owner during UAT. Final authority always rests with the Product Owner; either can request a reclassification with justification.
- When in doubt between two levels, always choose the higher severity.
- A defect found in production is automatically escalated one severity level (Medium → High, High → Critical).
- Security-related defects are always classified as Critical regardless of functional impact.
- Cross-tenant data access defects (Org A user reading or writing Org B data) are always Critical regardless of whether data was actually modified.
- When a defect is fixed, add or update a test that would catch the same defect if it regressed. This test is included in the same PR as the fix.

---

## Deferred Defect Process

A defect may be set to **Deferred** only when all of the following conditions are met:

1. The defect is **Medium or Low** severity — Critical and High defects cannot be deferred
2. If this project has an external client (not an internal/self-built product), the client has been **informed** of the defect and its deferral
3. The **Product Owner has approved** the deferral
4. A **target sprint** for the fix is agreed and recorded in the defect log

Deferred defects are batched into the agreed sprint. They do not disappear — track all Deferred defects (e.g. in the sprint plan's carry-forward list or the test execution report) and re-test them when the target sprint is delivered.

**A High severity defect may never be deferred without explicit Product Owner agreement.** The default is that High defects block UAT sign-off.

---

## Defect Log Format

Every defect logged (by `/verify-story`, or by the Product Owner during UAT) should include:

```
ID:          DEF-{N}
Title:       {Short description — what is wrong, not what was expected}
Severity:    Critical | High | Medium | Low
Story:       {Story ID that introduced or covers this behaviour}
Steps:       {Numbered reproduction steps}
Expected:    {What should have happened}
Actual:      {What actually happened}
Environment: {Dev | UAT | Production}
Status:      Open | In Progress | Fixed | Verified | Deferred
Fixed By:    {PR / commit reference — populated when status moves to Fixed}
Regression:  {Test added: yes/no — if no, reason}
```
