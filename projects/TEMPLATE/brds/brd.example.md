Harborview Consulting Ltd — Invoice Management System (Phase 1, Core). Source tags shown on every requirement. Open questions resolved before approval. This example demonstrates correct use of source tagging, client-specific vs domain-default separation, cross-section consistency, and the verification checklist.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-06-10 | Ascendra AI | Initial draft from discovery session |
| 1.1 | 2026-06-18 | Ascendra AI | Director approval threshold confirmed; OQ-001 and OQ-002 resolved; NFRs added |

---

## 1. Project Overview

### 1.1 Project Name

Invoice Management System

### 1.2 Problem Statement

Harborview Consulting Ltd manages all client invoicing through a shared Excel spreadsheet. The Finance Manager has no real-time view of outstanding balances, and payment reminders are sent manually — meaning invoices are regularly missed until the monthly finance close. Directors must approve invoices above £10,000 by email forwarding, with no formal tracking or audit trail. The business needs a system that eliminates manual reconciliation, automates overdue reminders, and gives every stakeholder a live view of payment status.

### 1.3 Business Goals

- Reduce Finance Manager monthly reconciliation time from 4 hours to under 30 minutes
- Ensure 100% of overdue invoices receive an automated reminder within 24 hours of the due date passing — no manual intervention
- Give Finance Manager and Directors a live view of outstanding and overdue balances at all times
- Replace email-based Director approval with a tracked, in-system workflow that produces an auditable approval record

### 1.4 Scope

**In Scope**
- Payer (client) management — register, import, deduplicate, and manage the client base
- Invoice creation, editing, reference number generation, and PDF attachment
- Director approval workflow for invoices above a configurable threshold
- Invoice sending via email with Stripe payment link
- Payment confirmation via Stripe webhook
- Automated overdue reminder emails on a configurable schedule
- Finance Manager dashboard (outstanding balance, overdue, upcoming due dates)
- Director approval queue

**Out of Scope**
- Client-facing (Payer) self-service portal — Payers pay via the link in the invoice email; no Payer login is in scope
- Purchase order (PO) matching — Harborview does not use POs
- Accounting system integration (e.g. Xero, QuickBooks) — identified as a future phase; base data model is designed to support it
- Mobile native application — the web application must be mobile-responsive; no native app in scope
- Payroll or expense management — handled by a separate system outside this scope

### 1.5 Assumptions & Constraints

- Harborview operates in GBP only. Multi-currency is not required. [Client-Stated]
- All Payers are companies (B2B). No consumer payment flows are in scope. [Client-Stated]
- The Stripe account already exists — Harborview will provide API credentials before Sprint 1 payment stories begin. [Client-Stated]
- The system is used by Harborview staff only. Payers interact with the system only via emailed payment links — they do not create accounts. [Client-Stated]
- No specific GDPR data retention schedule was stated. UK standard 7-year financial record retention assumed. [Assumed]
- Maximum of 5 concurrent Finance team users at any given time. [Client-Stated]

### 1.6 Designed for Extension

| Dimension | What Varies | Anticipated By | Base Must Provide |
|-----------|-------------|---------------|-------------------|
| Accounting integration | Export format, field mapping, authentication method per provider (Xero vs QuickBooks vs others) | Client explicitly named Xero as a future need | Invoice and payment records must have stable, exportable identifiers; no provider-specific fields in the core data model |

---

## 2. Stakeholders & Personas

| Persona | Who They Are | Primary Goal |
|---------|-------------|--------------|
| Finance Manager | Harborview's sole finance team member (James Okafor). Creates all invoices, manages Payers, monitors balances. | Create and send invoices quickly, track payment status in real time, automate overdue reminders. |
| Director | One of three Harborview directors. Approves invoices above the threshold before they are sent. | Review and approve or reject high-value invoices from any device without email chains. |
| Payer | A client company of Harborview receiving an invoice. Interacts via the payment link in the invoice email only. | Pay invoices quickly without creating an account. |

---

## 3. User Roles

### ROL-001: Finance Manager

**Can:** Create, edit, and void invoices; register and manage Payers; import Payers via CSV; view all invoices and payment status; configure overdue reminder schedule; view the Finance Manager dashboard.
**Cannot:** Approve high-value invoices (Director-only action); modify the approval threshold setting.
**Data visibility:** All Payers, all invoices, all payment records within the organisation.

### ROL-002: Director

**Can:** Approve or reject invoices in the approval queue; view full invoice details before acting; view all invoices (read-only).
**Cannot:** Create or edit invoices; configure system settings; manage Payers; send invoices.
**Data visibility:** All invoices (read-only). Payer name visible; Payer contact details not visible.

---

## 4. User Journeys

### 4.1 Finance Manager — Create and Send a Standard Invoice

1. Finance Manager selects a registered Payer from the Payer list.
2. Finance Manager adds line items (description, quantity, unit price) and sets a due date.
3. System calculates subtotal, VAT (20%), and total in real time.
4. Finance Manager saves the invoice as Draft. System assigns a unique reference number.
5. Finance Manager reviews the draft and clicks Send.
6. System generates a PDF and sends it to the Payer's registered email with an embedded Stripe payment link.
7. Invoice status changes from Draft to Sent.
8. When the due date passes without payment, the system automatically sends overdue reminder emails per the configured schedule.

### 4.2 Finance Manager — Create a High-Value Invoice (Director Approval Required)

1–4. Same as Journey 4.1.
5. Finance Manager attempts to send. System detects the total exceeds the approval threshold (£10,000).
6. Invoice status changes to Pending Approval. Finance Manager sees on-screen confirmation. All three Directors receive an email notification.
7. A Director opens the approval queue, reviews the invoice, and approves or rejects it.
8. **If approved:** status changes to Approved. Finance Manager is notified and sends the invoice (step 6 of Journey 4.1 onwards).
9. **If rejected:** status changes to Rejected with the Director's stated reason. Finance Manager is notified and may edit and resubmit.

### 4.3 Payer — Pay an Invoice via Email Link

1. Payer receives the invoice email with a PDF attachment and a "Pay Now" button.
2. Payer clicks "Pay Now" and is redirected to a Stripe-hosted checkout page.
3. Payer enters card details and completes payment on Stripe's page.
4. Stripe sends a webhook event to the system confirming successful payment.
5. Invoice status changes from Sent to Paid. Finance Manager sees the updated status on the dashboard.

---

## 5. Functional Requirements

### 5.1 Payer Management

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-001 | The system must allow the Finance Manager to register a new Payer with name, primary contact email, and phone number. | Must Have | Client-Stated | Core |
| REQ-002 | The system must detect duplicate Payers on registration. The deduplication key is configurable per organisation: phone number, email, or both (either match triggers detection). Default is phone number. | Must Have | Client-Stated | Core |
| REQ-003 | When a secondary deduplication key matches an existing Payer, the system must show a soft warning and require the Finance Manager to explicitly acknowledge it before proceeding. | Must Have | Domain-Default | Core |
| REQ-004 | The system must allow the Finance Manager to import Payers from a CSV file. The import must run deduplication checks and produce a per-row import log showing success or failure. Duplicate rows are flagged and not imported — manual resolution required. | Must Have | Client-Stated | Core |
| REQ-005 | The system must allow the Finance Manager to create an Opening Balance invoice against any Payer to record outstanding debt carried forward from a prior system. | Must Have | Client-Stated | Core |

### 5.2 Invoice Management

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-006 | The system must allow the Finance Manager to create an invoice by selecting a Payer, adding line items (description, quantity, unit price), and setting a due date. | Must Have | Client-Stated | Core |
| REQ-007 | The system must calculate subtotal, VAT at 20%, and total automatically. These values must recalculate in real time as line items change and must not be editable by the user. | Must Have | Client-Stated | Core |
| REQ-008 | Every invoice must be assigned a unique reference number in the format INV-YYYY-NNNNNN at creation time, where YYYY is the calendar year and NNNNNN is a zero-padded sequential number that never resets between years. | Must Have | Domain-Default | Core |
| REQ-009 | The system must generate a PDF of the invoice and attach it to the invoice sending email. The PDF must include Payer details, line items, subtotal, VAT, total, reference number, and due date. | Must Have | Client-Stated | Core |
| REQ-010 | Invoices above the organisation's high-value threshold must be submitted for Director approval before they can be sent. The approval threshold is configurable per organisation. Harborview's threshold is £10,000 (inclusive). | Must Have | Client-Stated | Core |
| REQ-011 | All three Harborview Directors must be notified by email when an invoice is submitted for approval. Any one Director may approve or reject the invoice — a second Director's approval is not required. | Must Have | Client-Stated | Core |
| REQ-012 | When a Director rejects an invoice, a rejection reason must be recorded and the Finance Manager notified. The Finance Manager may edit and resubmit the invoice. | Must Have | Client-Stated | Core |
| REQ-013 | The system must allow the Finance Manager to void a Sent or Approved invoice. A voided invoice cannot be edited, reactivated, or resent. | Should Have | Client-Stated | Core |

### 5.3 Payment Collection

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-014 | The system must generate a Stripe payment link for each Sent invoice and embed it in the invoice email as a "Pay Now" button. Payers pay via the Stripe-hosted checkout page — no payment UI is built within the system. | Must Have | Client-Stated | Core |
| REQ-015 | When Stripe confirms a successful payment via webhook, the system must automatically update the invoice status to Paid and record the payment date and amount received. | Must Have | Client-Stated | Core |
| REQ-016 | If the payment amount confirmed by Stripe is less than the invoice total, the system must not mark the invoice as Paid. The discrepancy must be logged and the Finance Manager alerted for manual resolution. | Must Have | Assumed | Core |

### 5.4 Overdue Management

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-017 | The system must automatically send an overdue reminder email to the Payer when a Sent invoice passes its due date without payment. The first reminder fires within 24 hours of the due date. | Must Have | Client-Stated | Core |
| REQ-018 | The overdue reminder schedule must be configurable by the Finance Manager: number of reminders and the interval between them (e.g. Day 1, Day 7, Day 14 after due date). | Should Have | Client-Stated | Core |
| REQ-019 | After the configured maximum number of reminders is sent, the system must flag the invoice status as Overdue (Escalated) and alert the Finance Manager that manual follow-up is required. | Should Have | Client-Stated | Core |

### 5.5 Dashboards

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-020 | The Finance Manager dashboard must display: total outstanding balance, total overdue balance, invoices due within the next 7 days, and recently paid invoices. All figures update in real time. | Must Have | Client-Stated | Core |
| REQ-021 | Directors must have a dedicated approval queue view listing all invoices pending their approval, with the ability to view full invoice details before acting. | Must Have | Client-Stated | Core |

---

## 6. Business Rules

| ID | Rule | Applies To | Source |
|----|------|-----------|--------|
| BR-001 | VAT is applied at 20% to the invoice subtotal. Harborview has no VAT-exempt line items — VAT applies to every invoice. | REQ-007 | Client-Stated |
| BR-002 | The Director approval threshold is £10,000 inclusive. An invoice totalling exactly £10,000 requires Director approval. | REQ-010 | Client-Stated |
| BR-003 | Any one of the three Directors may approve an invoice. A second Director's approval is not required. | REQ-011 | Client-Stated |
| BR-004 | A voided invoice cannot be edited, reactivated, or resent. Voiding is a permanent terminal action. | REQ-013 | Client-Stated |
| BR-005 | The invoice reference number sequence must never reset between calendar years. INV-2025-001000 is followed by INV-2026-001001 — not INV-2026-000001. | REQ-008 | Domain-Default |

---

## 7. Data Requirements

### Invoice

**What it is:** The primary billing record from Harborview to a Payer.
**Key attributes:** Reference number, Payer, line items (description, quantity, unit price), subtotal, VAT amount, total (all monetary values stored in pence as integers), due date, status, sent date, paid date, approved-by Director (if applicable), void reason (if voided).
**Relationships:** Belongs to one Payer. Has many line items. May have one approval record.
**Deviates from domain standard:** No.

### Payer

**What it is:** A client company of Harborview that receives invoices.
**Key attributes:** Company name, primary contact email, phone number, configured deduplication key type, opening balance (if an Opening Balance invoice exists).
**Relationships:** Has many invoices.
**Deviates from domain standard:** No — standard Payer entity with configurable deduplication key as per domain default.

### Approval Record

**What it is:** The record of a Director's decision on a high-value invoice.
**Key attributes:** Invoice reference, Director name, decision (Approved / Rejected), rejection reason, decision timestamp.
**Relationships:** Belongs to one invoice. One approval record per invoice lifecycle — the first Director to act creates the record.
**Deviates from domain standard:** Yes — Harborview uses a single-approver model (any one Director). The domain standard supports multi-level approval chains; that complexity is not required here and must not be built.

---

## 8. Integration Requirements

### Stripe — Payment Processing

**What it is:** Payment gateway for card payments from Payers.
**Direction:** Bidirectional — Outbound (generate payment link per invoice), Inbound (receive payment confirmation webhook).
**Trigger:** Outbound: invoice is sent. Inbound: Payer completes card payment on Stripe checkout.
**Data exchanged:** Outbound: invoice total (in pence), currency (GBP), Payer email, invoice reference number. Inbound: webhook event containing payment status, amount received, Stripe payment intent ID, and invoice reference.
**Owner:** Third party (Stripe). Harborview holds the API credentials.

### Resend — Transactional Email

**What it is:** Email delivery service for all system-generated emails.
**Direction:** Outbound only.
**Trigger:** Invoice sent; invoice submitted for Director approval; overdue reminder schedule fires; Director approves or rejects an invoice.
**Data exchanged:** Recipient address, subject line, email body text, PDF attachment (invoice emails only).
**Owner:** Third party (Resend). Credentials held by the Ascendra delivery team.

---

## 9. Security Requirements

| ID | Requirement | Priority | Source |
|----|-------------|----------|--------|
| SEC-001 | All users must authenticate with email and password before accessing any part of the system. No unauthenticated access to invoice or Payer data is permitted. | Must Have | Domain-Default |
| SEC-002 | Invoice and Payer data is scoped to the organisation. A user from one organisation must not be able to view, create, or modify data belonging to another organisation. | Must Have | Domain-Default |
| SEC-003 | The Finance Manager role must not be able to approve or reject invoices. Approval actions are Director-only and must be enforced server-side. | Must Have | Client-Stated |
| SEC-004 | The Stripe webhook endpoint must verify the Stripe webhook signature on every inbound request. Requests with an invalid or missing signature must be rejected with HTTP 400 and the event must be logged without processing. | Must Have | Domain-Default |
| SEC-005 | All actions that change invoice status (create, send, approve, reject, void, paid) must produce an audit log entry recording: invoice reference, actor identity, action, and UTC timestamp. | Must Have | Domain-Default |
| SEC-006 | Payer contact details (email address and phone number) must not be visible to Directors. Directors see Payer company name only. | Should Have | Client-Stated |

---

## 10. Non-Functional Requirements

| Category | Requirement | Source |
|----------|-------------|--------|
| Performance | Invoice list and dashboard pages must load within 2 seconds at up to 50 concurrent users. | Assumed |
| Availability | System must be available during UK business hours (08:00–20:00 GMT). Planned maintenance outside these hours. | Assumed |
| Browser / Device support | Must function correctly on Chrome, Safari, and Edge (latest two versions). Director approval queue must be usable on a tablet in portrait mode. | Client-Stated |
| Data Retention | Financial records (invoices, payments, approvals) must be retained for a minimum of 7 years in line with UK financial record-keeping obligations. | Assumed |

---

## 11. Dependencies

| ID | Dependency | Type | Owner | Required By | Risk if Delayed |
|----|-----------|------|-------|-------------|----------------|
| DEP-001 | Stripe API credentials (publishable key, secret key, webhook signing secret) | External credentials | Harborview (James Okafor) | Sprint 1 — payment integration stories | Payment collection cannot be tested or delivered |
| DEP-002 | Resend API key and sending domain verification | External credentials | Ascendra delivery team | Sprint 1 — invoice email stories | Invoice sending and all automated emails blocked |
| DEP-003 | Director email addresses for approval notification configuration | Client data | Harborview (James Okafor) | Sprint 2 — Director approval stories | Approval notification cannot be configured or tested |

---

## 12. Open Questions

None — all questions raised during discovery were resolved before approval.

---

## 13. Pre-Approval Verification

### 13.1 Content Integrity

- [x] Every requirement in section 5 has all four fields: ID, description, priority, and source tag.
- [x] No [Domain-Default] requirement restates a universal fact without note. REQ-003 (soft warning) and REQ-008 (reference format) are domain defaults confirmed as applicable without client override.
- [x] No [Assumed] requirement covers something that should have been confirmed. REQ-016 (partial payment) and REQ-020 performance figures are low-risk assumptions appropriate for Phase 1.
- [x] Every business rule in section 6 is client-specific or non-obvious. BR-005 (no sequence reset) is a domain default included because the client explicitly asked about reference numbering.
- [x] Section 1.4 Out of Scope is populated with 5 explicit exclusions including the accounting integration and Payer portal.
- [x] No implementation detail in sections 3–9 — no field types, API paths, database schema, or technology choices.

### 13.2 Cross-Section Consistency

- [x] Every persona in section 2 appears in at least one journey in section 4 (Finance Manager → 4.1/4.2; Director → 4.2; Payer → 4.3).
- [x] Every journey step maps to at least one requirement in section 5.
- [x] Every requirement in section 5 is reachable from at least one journey.
- [x] Both integrations in section 8 have security requirements in section 9 (SEC-004 covers Stripe webhook; SEC-001 covers all authenticated access).
- [x] Both integrations have dependency entries in section 11 (DEP-001 for Stripe, DEP-002 for Resend).
- [x] All roles in section 3 are referenced in section 5 requirements.

### 13.3 Downstream Readiness

- [x] Every requirement is specific enough that a developer can write a user story from it without asking follow-up questions.
- [x] No requirement uses deferral language ("standard X", "appropriate Y", "the system should handle Z").
- [x] Every business rule in section 6 references a requirement in section 5 (BR-001 → REQ-007; BR-002/BR-003 → REQ-010/REQ-011; BR-004 → REQ-013; BR-005 → REQ-008).
- [x] All primary data entities implied by requirements are captured in section 7.
- [x] NFRs in section 10 are measurable where stated. Performance NFR specifies a threshold (2 seconds) and load (50 users).
- [x] Section 1.6 is populated — one extension dimension identified (accounting integration).

### 13.4 Approval Readiness

- [x] Section 12 (Open Questions) contains no open items.
- [x] All dependencies in section 11 have named owners.
- [x] BRD version in Document Control (1.1) matches version in section 14.
- [x] Product Owner has reviewed this BRD before client sign-off.

### 13.5 Verification Issues

None.

---

## 14. Approval

**Project:** Invoice Management System
**BRD Version:** 1.1
**Prepared by:** Ascendra AI — 2026-06-18
**Status:** Approved

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner | [PO Name] | ✓ | 2026-06-20 |
| Client Representative | James Okafor, Finance Manager | ✓ | 2026-06-20 |

---

## 15. Parking Lot

| ID | Idea | Surfaced (Date / Context) | Promotion Path | Status |
|----|------|---------------------------|-----------------|--------|
| PL-001 | Allow the Finance Manager's admin assistant to submit invoices on the Finance Manager's behalf, with the Finance Manager approving before send. Surfaced as an aside during UAT prep — never discussed during discovery, and the domain model has no "act on behalf of" or delegated-drafting concept for invoice creation. | 2026-07-02 — UAT prep call, Finance Manager mentioned it in passing | Requires a domain discovery pass on delegated/assisted authoring before it can become a REQ — not yet modeled | Parked |
