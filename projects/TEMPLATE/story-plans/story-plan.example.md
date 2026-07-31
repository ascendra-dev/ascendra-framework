Harborview Consulting Ltd — Invoice Management System (BRD v1.1). Same fictional project as `stories/story.example.md`. Two examples: a full API+Web plan for Example 2's story (US-01-002, Create invoice form) showing every section fully worked, and a short excerpt showing just the Worker Plan section's shape for a scheduled-job story, since no full worker example exists in `story.example.md` to draw a whole plan from.

---

### Example 1 — API+Web Plan

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-07-05 | Ascendra AI | Initial version from /gen-story-plan |

**Story ID:** US-01-002

**Epic:** EPIC-003 — Invoice Management

**Target:** API+Web

**Architecture version:** arch-v1.md (Locked on 2026-07-04)

**Plan Status:** Confirmed

---

### 1. AC → Artifact Traceability

| AC # | AC Text (short) | Satisfied by |
|------|-----------------|---------------|
| 1 | Form fields: Payer selector, line items, due date, Save as Draft | WEB-1 |
| 2 | Subtotal/VAT/total recalculate live | WEB-1 |
| 3 | Cannot submit with zero line items | WEB-1, API-1 |
| 4 | Cannot submit without a Payer | WEB-1, API-1 |
| 5 | Due date must be today or future | WEB-1, API-1 |
| 6 | Redirect to invoice detail on save, showing reference number | WEB-1, API-1 |
| 7 | Inline validation errors, no page reload | WEB-1 |

---

### 2. API Plan

#### API-1: Create invoice

**Module:** existing — `src/modules/invoices/` (schema and reference-number generation already exist from US-01-001/US-01-005)
**Endpoint:** `POST /api/v1/invoices` (per arch Section 5, Invoices Endpoints)
**Auth:** `JwtAuthGuard`, `@Roles('finance_manager')`
**Request:** `payerId: string (uuid, required)`, `lineItems: { description: string, quantity: number (min 1), unitPrice: integer (min 0) }[] (min 1 item)`, `dueDate: string (ISO date, must be >= today)`
**Response:** `id: string`, `referenceNumber: string`, `subtotal: integer`, `vat: integer`, `total: integer`, `status: 'draft'`
**Schema touched:** existing `invoices` and `invoice_line_items` tables (per arch Section 4.2) — no new columns
**Business logic:**
1. Validate payer exists and belongs to the caller's organisation
2. Validate at least one line item and a due date of today or later
3. Compute subtotal, VAT (20% flat, per Section 4.2 note), and total server-side — never trust client-computed totals
4. Call the reference-number generation mechanism (US-01-005) inside the same transaction as the invoice insert
5. Persist invoice + line items, status `draft`
**Queries:** insert scoped to caller's `orgId`; no soft-delete concern on create
**Audit / idempotency:** none required — creation is not a repeatable/idempotent-sensitive action per this story's ACs
**Sensitive fields excluded:** none listed for this module in Section 6.4
**Files (predicted):** new: none; modified: `src/modules/invoices/invoices.controller.ts`, `src/modules/invoices/invoices.service.ts`, `src/modules/invoices/dto/create-invoice.dto.ts`

---

### 3. Web Plan

#### WEB-1: Create Invoice page

**Route:** `/invoices/new` (per arch Section 3.1.2, Finance portal route group) — new page
**Surface:** Page — Screen ID SCR-014, Matched Reference "Form with computed summary panel" (per `screen-design.md` Section 3)
**Components:** `ascendra-ui` `SearchableSelect` (Payer picker), `DataTable` in editable-row mode (line items), `DatePicker` (due date), `Button` (Save as Draft) — all reused, no new composition
**Data layer:** `useCreateInvoice` mutation hook, calls API-1; `usePayerList` query hook (existing, reused) to populate the Payer selector
**Types:** `CreateInvoiceRequest` / `CreateInvoiceResponse` matched exactly to API-1's request/response fields
**Client validation:** mirrors API-1 — at least one line item, Payer required, due date >= today; subtotal/VAT/total computed client-side for live display only, never sent to the server as input
**Navigation:** no change — reachable from the existing Invoices list page's "New Invoice" button
**Cross-cutting rules:** none applicable to this screen
**Files (predicted):** new: `app/(finance)/invoices/new/page.tsx`, `hooks/use-create-invoice.ts`; modified: none

---

### 4. Test Scenarios

| AC # | AC Text | Test Approach | Input | Expected Result |
|------|---------|---------------|-------|-----------------|
| 1 | Form fields present | UI action | Load `/invoices/new` | Payer selector, line item table, due date picker, Save as Draft button all render |
| 2 | Live recalculation | UI action | Add two line items (qty 2 @ 500, qty 1 @ 1000) | Subtotal 2000, VAT 400, total 2400 shown before save |
| 3 | Zero line items rejected | UI action + API call | Submit with no line items | Inline error shown; `POST /api/v1/invoices` not called; if called directly, API returns 400 |
| 4 | No Payer rejected | UI action + API call | Submit with a line item but no Payer selected | Inline error shown; API returns 400 if called directly with `payerId` omitted |
| 5 | Past due date rejected | API call | `dueDate` set to yesterday | API returns 400 with a due-date validation message |
| 6 | Redirect + reference number | UI action | Fill valid form, click Save as Draft | Redirected to `/invoices/{id}`, reference number in `INV-YYYY-NNNNNN` format visible |
| 7 | Inline errors, no reload | UI action | Trigger AC-3 and AC-4 together | Both errors shown inline next to their fields; page does not reload |

---

### 5. Risks / Open Questions

None.

---

### 6. Deviations from Plan

None.

---
---

### Example 2 — Worker Plan excerpt

> Only the Worker Plan section is shown — this illustrates the shape for a story whose Target is `Worker`, using the overdue-reminder job referenced as an Out of Scope item in `story.example.md` Example 2. The AC → Artifact Traceability, Test Scenarios, and other sections follow the same shape as Example 1 above and are omitted here for brevity.

### 4. Worker Plan

#### WORKER-1: Send overdue invoice reminder

**Job type:** scheduled (cron: `0 8 * * *`, daily 08:00 UTC) — new
**Trigger:** daily schedule, per the worker's Section 3.1 subsection (`ascendra-invoice-worker`)
**Schema touched:** `invoices` table (read: `status = 'sent'` and `due_date < today`); `reminder_log` table (write, per Section 4.2)
**Business logic:**
1. Query all `sent` invoices past due date with no reminder logged in the last 7 days
2. For each, enqueue a reminder notification (via the notifications module) and write a `reminder_log` row
**Idempotency:** the 7-day `reminder_log` lookback prevents a re-run in the same window from sending duplicate reminders
**Observability:** log job start (invoice count found), per-invoice success/failure (invoice ID, org), and job completion summary
**Files (predicted):** new: `src/jobs/send-overdue-reminders.job.ts`; modified: `src/jobs/registry.ts`
