## Part 3 — Worked Example

Sprint 1 UAT checklist for Harborview Consulting Ltd — Invoice Management System.

---

# UAT Checklist — Sprint 01 — Invoice Management System

**Sprint:** 01
**Project:** Invoice Management System
**Date generated:** 2026-07-15
**Completed by (PO):** Zaka Shah
**UAT date:** 2026-07-16

---

### How to use this checklist

Work through each story section in order. For every test scenario:
1. Follow the Test Steps exactly as written.
2. Observe the result.
3. Mark the Result column: **Pass**, **Fail**, or **N/A** (if the scenario does not apply to your test data).
4. If the result is **Fail**: note what you observed in the Notes column. Do not continue to the next section — raise the defect first.

A failing test stops this sprint from being marked Complete. Every Fail must be resolved (either fixed and re-tested, or formally deferred with a reason recorded).

---

### Sprint Summary

| Stories in this sprint | Must Have | Should Have | Total ACs | Status |
|----------------------|-----------|-------------|-----------|--------|
| 6 | 6 | 0 | 14 | All Merged ✓ |

---

### US-03-002 — Create invoice form

**What this story delivers:** A Finance Manager can create a new invoice against an existing client, add line items, and save it as a draft.

| # | Test Scenario | Test Steps | Expected Result | Result | Notes |
|---|--------------|-----------|----------------|--------|-------|
| US-03-002-TC-1 | Happy path — create and save a draft invoice | 1. Navigate to Invoices → New Invoice.<br>2. Select "Harborview Test Client" from the client dropdown.<br>3. Click "Add Line Item" and enter: Description "Consulting", Quantity 1, Unit Price 500.00.<br>4. Set due date to today + 30 days.<br>5. Click Save as Draft. | Client name and billing address auto-populate. Line total shows 500.00; total updates to 600.00 with VAT. Success toast appears: "Invoice saved as Draft." New invoice appears in the invoice list with total £600.00. | Pass / Fail / N/A | |
| US-03-002-TC-2 | Rejection — Client field required | 1. Navigate to Invoices → New Invoice.<br>2. Leave the Client field empty.<br>3. Add a line item: Description "Consulting", Quantity 1, Unit Price 500.00.<br>4. Click Save as Draft. | Form does not submit. Inline error appears below the Client field: "Client is required." No invoice is created. | Pass / Fail / N/A | |

---

### US-03-003 — Invoice detail page

**What this story delivers:** A Finance Manager can open any invoice and see its reference number, line items, and totals.

| # | Test Scenario | Test Steps | Expected Result | Result | Notes |
|---|--------------|-----------|----------------|--------|-------|
| US-03-003-TC-1 | Happy path — view invoice detail | 1. From the invoice list, click the invoice created in US-03-002-TC-1.<br>2. Observe the detail page. | Reference number in format INV-YYYY-NNNNNN is displayed. Line items, subtotal, VAT, and total match what was entered at creation. | Pass / Fail / N/A | |

---

### UAT Sign-off

All scenarios in this checklist have been executed. Results are recorded above.

**Overall result:** Pass / Fail (circle one)

**Failing scenarios (if any):**

| Scenario ID | Issue description | Severity | Decision |
|------------|------------------|---------|---------|
| | | | Fix before release / Defer to Sprint {N} |

**PO Signature:** _______________________   **Date:** ___________________

**Sprint approved for release:** Yes / No

---

_Generated via `/gen-uat-checklist` from approved sprint stories. All test scenarios are derived directly from story acceptance criteria._
