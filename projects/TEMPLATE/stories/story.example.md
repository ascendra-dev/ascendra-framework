Harborview Consulting Ltd — Invoice Management System (BRD v1.1). Three examples showing different story types: backend logic (reference number generation), UI form (invoice creation), and integration (Stripe webhook). REQ IDs match the approved BRD example. Dependencies are shown in Pass 2 format (resolved IDs) as they would appear after the sprint ordering pass. Each story in production is its own file with its own Document Control and Verification section. A single Verification section is shown after Example 3 as a representative filled-in instance.

---

### Example 1 — Backend Logic Story

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-07-05 | Ascendra AI | Initial version from /gen-stories |

**Story ID:** US-01-005

**Epic:** EPIC-003 — Invoice Management

**Title:** Generate invoice reference number

**Layer:** Core
**Walking Skeleton:** Yes
**Status:** Approved

---

#### 1. Story Statement

As a Finance Manager, I want each new invoice to automatically receive a unique reference number, so that invoices are traceable and consistent with our accounting records.

---

#### 2. FR References

REQ-008

---

#### 3. Acceptance Criteria

1. A new invoice record is assigned a reference number in the format INV-YYYY-NNNNNN at the moment it is created, where YYYY is the calendar year of creation and NNNNNN is a zero-padded 6-digit global sequence
2. The global sequence increments across all years and never resets — INV-2025-000999 is followed by INV-2026-001000, not INV-2026-000001
3. The reference number is stored on the invoice record at creation time — it is not computed on read
4. No two invoices share the same reference number, enforced by a unique constraint on the database column, not just an application-level check
5. The reference number field is not exposed as a writable input on the invoice creation API endpoint — it is assigned by the system only

---

#### 4. Out of Scope

- Invoice creation form and the remainder of the invoice creation flow (US-01-002)
- PDF generation using the reference number (US-01-007)
- Invoice sending (US-01-008)

---

#### 5. Size

S

---

#### 6. Dependencies

- US-01-001 — Invoice database schema: the `invoices` table and `reference_number` column must exist before the reference number generation logic can be written

---

#### 7. Notes

Use a single global database sequence (PostgreSQL `SEQUENCE`). Wrap the sequence increment and the invoice `INSERT` in one transaction — if the insert fails and rolls back, the consumed sequence number must not be reused. A gap in the global sequence due to a transaction rollback is acceptable; a year-boundary reset in the sequence is not.

---
---

### Example 2 — UI Story

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-07-05 | Ascendra AI | Initial version from /gen-stories |

**Story ID:** US-01-002

**Epic:** EPIC-003 — Invoice Management

**Title:** Create invoice form

**Layer:** Core
**Walking Skeleton:** Yes
**Status:** Approved

---

#### 1. Story Statement

As a Finance Manager, I want to create an invoice by selecting a Payer, adding line items, and setting a due date, so that I can bill Payers without leaving the system.

---

#### 2. FR References

REQ-006

---

#### 3. Acceptance Criteria

1. The form contains: a Payer selector (searchable dropdown populated from the registered Payer list), a line item table (columns: description, quantity, unit price), a due date picker, and a Save as Draft button
2. Subtotal, VAT (20% applied to the subtotal), and total recalculate and display in real time as line items are added, edited, or removed
3. The form cannot be submitted with zero line items — an inline validation message is shown and the Save button is disabled
4. The form cannot be submitted without a Payer selected — an inline validation message is shown
5. The due date must be today or a future date — past dates are rejected with an inline validation message
6. On successful save, the user is redirected to the invoice detail page showing the saved draft and its assigned reference number
7. On any validation error, errors appear inline next to the relevant field — the page does not reload

---

#### 4. Out of Scope

- Sending the invoice to the Payer (US-01-008)
- Director approval workflow for invoices above £10,000 (US-01-004)
- PDF generation (US-01-007)
- Editing a saved draft (US-01-003)

---

#### 5. Size

M

---

#### 6. Dependencies

- US-01-001 — Invoice schema: the `invoices` and `invoice_line_items` tables must exist before this form can persist data
- US-01-005 — Reference number generation: must be in place so the invoice detail page can display the reference number after save
- US-02-001 — Payer management: the Payer list must be queryable before the Payer selector can be populated

---

#### 7. Notes

Quantity and unit price are numeric inputs. Subtotal, VAT, and total are computed display fields — read-only, not form inputs. Do not send computed values from the client; recalculate on the server before persisting. All monetary values are stored as integers in pence.

---
---

### Example 3 — Integration Story

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-07-05 | Ascendra AI | Initial version from /gen-stories |

**Story ID:** US-01-012

**Epic:** EPIC-005 — Payment Collection

**Title:** Stripe webhook payment confirmation

**Layer:** Core
**Walking Skeleton:** Yes
**Status:** Approved

---

#### 1. Story Statement

As a Finance Manager, I want invoice status to update to Paid automatically when a Payer pays via Stripe, so that payment tracking requires no manual intervention for card payments.

---

#### 2. FR References

REQ-014, REQ-015

---

#### 3. Acceptance Criteria

1. The system exposes a `POST /webhooks/stripe` endpoint that accepts `checkout.session.completed` and `payment_intent.succeeded` Stripe event types
2. The endpoint verifies the Stripe webhook signature on every request — events with an invalid or missing signature are rejected with HTTP 400 and not processed
3. On a valid payment event matching a Sent invoice, the invoice status is updated to Paid and a `paid_at` timestamp is recorded
4. If the payment amount received from Stripe is less than the invoice total, the invoice status is not changed — the discrepancy is logged and the Finance Manager is alerted
5. A successful status change produces an audit log entry: invoice reference, actor = "stripe-webhook", action = "paid", UTC timestamp
6. Processing a Stripe event ID that has already been processed produces no state change — the endpoint returns HTTP 200 without modifying data (idempotency)
7. The endpoint returns HTTP 200 to Stripe within 5 seconds to prevent Stripe from retrying the event

---

#### 4. Out of Scope

- Manual payment marking by the Finance Manager for bank transfer payments (US-01-011)
- Overdue reminder suppression when payment is received (US-01-015)
- Stripe payment link generation on invoice send (US-01-010)

---

#### 5. Size

M

---

#### 6. Dependencies

- US-01-001 — Invoice schema: the `invoices` table with `status` = `paid` and `paid_at` column must exist
- US-01-008 — Invoice sending: invoices must be reachable in Sent status before payment events can match them
- Stripe API keys and webhook signing secret: must be available in environment config (`STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`)

---

#### 7. Notes

Use `stripe.webhooks.constructEvent()` for signature verification — do not implement manual HMAC verification. Store processed Stripe event IDs in a `stripe_events` table (columns: `event_id`, `processed_at`) to enforce idempotency at the database level. Test locally using `stripe listen --forward-to localhost:3001/webhooks/stripe`.

---

## Verification

> Shown here for Example 3 (most complex story). The same checklist applies to Examples 1 and 2 — all checks pass on those stories as well.

- [x] Story ID follows the correct format (US-01-012) and matches Sprint 1
- [x] Epic ID is populated (EPIC-005) and references a file that exists in the project's epics/ directory
- [x] Layer (Core) matches the parent epic's layer
- [x] Story Statement has exactly one persona (Finance Manager) drawn from BRD Section 3 (User Roles)
- [x] The action in the Story Statement is a specific capability ("invoice status to update to Paid automatically when a Payer pays via Stripe") — not a feature area description
- [x] FR References (REQ-014, REQ-015) trace to requirements in the approved BRD — both exist in BRD Section 5.3 (Payment Collection)
- [x] Every acceptance criterion is independently testable — a QA engineer can write a test case for each without a follow-up question
- [x] Acceptance criteria cover the happy path (AC-3), primary validation (AC-2 webhook signature), and primary error states (AC-4 partial payment, AC-6 duplicate event)
- [x] No acceptance criterion describes implementation detail — AC items describe system behaviour, not database calls or library methods (implementation detail is in Section 7 Notes, where it belongs)
- [x] Out of Scope is populated — three related capabilities named with their covering story IDs
- [x] Every Out of Scope item names the story that covers it — no "to be handled later" without a named owner
- [x] Size is M — not XL, no split required
- [x] Dependencies are present in Pass 2 format — two resolved story IDs and one infrastructure dependency in plain text
- [x] The story is independently deliverable — it can be merged once its two story dependencies land; no in-flight stories must land simultaneously

---

### 8. Density & Judgment Findings

*Generated by `/judgment-check` against `practitioner-guide/07-sprint-planning.md` on 2026-07-20, run against Example 2 (US-01-002, Create invoice form) — the example that produced genuine, checkable findings; Examples 1 and 3 raised nothing checkable against this chapter's tests. Shown once, representatively, per this file's own convention of showing a single worked instance rather than one copy per example.*

| # | Test (chapter source) | Location | Finding |
|---|---|---|---|
| 1 | Proactive story-sizing test — Pattern Novelty / Context Footprint factors, and the template's own Size guide criterion that L covers "a UI flow with branching" | US-01-002, Section 5 (Size: M) read against Section 3 (AC-2 real-time computed totals, AC-3/4/5 three independent validation branches, AC-6 post-save redirect) | Three separate validation branches plus a real-time computed-field pattern and a post-save redirect touch enough distinct concerns that the story could read as "a UI flow with branching" (the template's own L criterion) rather than M — is 7 ACs comfortably inside the 3–10 range enough to confirm M, or does the branching push this past it? |
| 2 | Wave-vs-epic boundary test — "It's legitimately more than one epic only when the pair carries a documented story-level interleaving note" | US-01-002, Section 6 (Dependencies) — "US-02-001 — Payer management: the Payer list must be queryable before the Payer selector can be populated" | This dependency crosses from EPIC-003 (Invoice Management) into EPIC-002 (Payer/Client Management) — does this rise to the level of a story-level interleaving note that belongs in `epics/index.md` Section 3, or is it an ordinary build-order dependency (generate and review EPIC-002 before EPIC-003) that resolves without one? |

---

### 9. Density & Judgment Resolution

| Date | Finding | PO Response | Detail |
|------|---------|-------------|--------|
| 2026-07-20 | US-01-002 sizing (M vs. L) | Confirmed as-is | The three validation branches (Payer required, line items required, due date not in the past) are guard conditions on one form submission, not divergent user-facing flows — the template's "UI flow with branching" L criterion means genuinely different outcomes or screens depending on role or state, which this story doesn't have. Real-time computed totals are a single derived-field pattern, not a novel one for this codebase's context (every line-item form needs it). 7 ACs squarely mid-range confirms M. |
| 2026-07-20 | US-01-002 → US-02-001 cross-epic dependency | Confirmed as-is | This is a coarse "EPIC-002 substantially built before this specific EPIC-003 story" dependency, not alternating interleaving — US-01-002 only needs the Payer list queryable, which EPIC-002's own first story (schema + CRUD) already establishes on its own. Normal wave sequencing (generate and review EPIC-002 stories, then EPIC-003, order by dependency at the sprint-planning stage) resolves it without a documented interleaving note; reserving that mechanism for cases where individual stories from two epics must genuinely alternate keeps it meaningful rather than routine. |
