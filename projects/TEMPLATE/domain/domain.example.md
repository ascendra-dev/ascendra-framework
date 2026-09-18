Invoice Payment (Core) — universal domain knowledge for B2B invoice-based payment systems. This document covers the core sub-domain only; country-specific obligations (tax mandates, e-invoicing regulations) belong in country extension documents. This example was loaded before the Harborview Consulting Ltd discovery session and before any BRD discovery session in the invoice payment domain.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-05-15 | Ascendra AI | Initial version |
| 1.1 | 2026-05-28 | Ascendra AI | Added Opening Balance invoice type; expanded dunning section in Known Variations; added SEC rules from edge case testing |

---

## 1. Domain Overview

### 1.1 What This Document Covers

The invoice payment domain covers the business processes by which one organisation (the Issuer) bills another organisation or individual (the Payer) for goods or services rendered, and subsequently collects payment. It includes invoice creation, approval, delivery, payment collection, and overdue management. It does not cover the underlying service or product delivery that gives rise to the invoice.

### 1.2 What This Document Does Not Cover

- Payroll processing — wages paid to employees are not invoices
- Purchase order (PO) management — the receipt and matching of inbound supplier invoices
- Inventory or stock management — the source of goods or services being invoiced
- Subscription billing with automated recurring charges — recurring SaaS billing has a distinct lifecycle not covered here
- Consumer (B2C) payment flows — this document assumes B2B invoicing; consumer payment regulation differs materially
- Country-specific e-invoicing mandates (e.g. FBR in Pakistan, FIRS in Nigeria) — see country extension documents

### 1.3 Related Documents

| Document | Relationship |
|----------|-------------|
| None at this time. | — |

---

## 2. Domain Terminology

| Term | Definition | Notes |
|------|-----------|-------|
| Invoice | A formal document issued by the Issuer to a Payer requesting payment for goods or services. An invoice is a legally meaningful document in most jurisdictions. | Not the same as a quote or pro-forma — those are not payment demands |
| Issuer | The organisation creating and sending invoices — the party owed money. | In Ascendra-built systems, this is typically the client's organisation |
| Payer | The organisation or individual receiving and paying the invoice — the party owing money. | Also called "client" or "customer" in common usage — use "Payer" in all artifacts |
| Line Item | A single row on an invoice representing one charge — consisting of description, quantity, unit price, and line total. | An invoice must have at least one line item |
| Reference Number | A unique, human-readable identifier assigned to an invoice at creation, used for tracking and communication. | Also called invoice number — use "Reference Number" in all artifacts |
| VAT | Value Added Tax — a consumption tax applied as a percentage of the invoice subtotal. | Rate varies by jurisdiction. Country extension documents define applicable rates |
| Due Date | The date by which payment is expected. Overdue status is triggered when this date passes without full payment. | |
| Opening Balance | A special invoice type created to record debt a Payer carried forward from a prior system, representing historical outstanding amounts rather than a new transaction. | Used during data migration onboarding only |
| Dunning | The process of sending a sequence of automated reminders to a Payer for an overdue invoice. | From the Dutch word "dun" — to press for payment |
| Approval Threshold | A monetary value above which an invoice must be reviewed and approved by an authorised person before it can be sent. | Configurable per organisation |
| Void | The action of permanently cancelling an invoice that was sent but should not be paid, typically due to an error. A voided invoice cannot be edited or resent. | Distinct from a credit note — a credit note creates a new offsetting document; voiding removes the obligation entirely |

---

## 3. Core Entities & Relationships

### Invoice

**What it is:** The primary billing document issued by the Issuer to a Payer.
**Standard attributes:** Reference number, Payer, line items, subtotal, VAT amount, total, issue date, due date, status, sent date, paid date, currency.
**Relationships:** Belongs to one Payer. Has one or more line items. May have one approval record. May have many dunning log entries.
**Notes:** Monetary values are always stored as integers in the smallest currency unit (pence, cents, paisa) — never as decimals.

### Line Item

**What it is:** A single charge within an invoice — the building block of the invoice total.
**Standard attributes:** Description, quantity, unit price, line total (quantity × unit price). VAT is calculated at invoice level, not line item level.
**Relationships:** Belongs to one invoice. Cannot exist without a parent invoice.
**Notes:** Line items are immutable once the invoice is sent. Edits require voiding the invoice and creating a new one.

### Payer

**What it is:** The party receiving and paying invoices. In B2B systems, this is always an organisation.
**Standard attributes:** Organisation name, primary contact name, contact email, phone number, billing address.
**Relationships:** Has many invoices.
**Notes:** A Payer record persists even after all their invoices are closed — they may receive future invoices.

### Approval Record

**What it is:** The record of an authorised person's decision to approve or reject a high-value invoice.
**Standard attributes:** Invoice reference, approver identity, decision (Approved / Rejected), rejection reason, decision timestamp.
**Relationships:** Belongs to one invoice. One approval record per invoice.
**Notes:** Approval record is created when the first approver acts. In single-approver models, this completes the approval. In multi-level models, subsequent approvers add to the same record.

---

## 4. Standard Lifecycle & Status Model

### Invoice Lifecycle

**States:** Draft, Pending Approval, Approved, Sent, Overdue, Overdue (Escalated), Paid, Void, Cancelled

**Standard transitions:**
- Draft → Pending Approval: triggered when Finance Manager attempts to send an invoice that exceeds the approval threshold
- Draft → Sent: triggered when Finance Manager sends an invoice that is below the approval threshold (or where no threshold is configured)
- Pending Approval → Approved: triggered when an authorised approver approves the invoice
- Pending Approval → Rejected: triggered when an authorised approver rejects the invoice. Invoice returns to an editable Draft-like state for correction and resubmission.
- Approved → Sent: triggered when the Finance Manager sends an Approved invoice
- Sent → Overdue: triggered automatically when the due date passes without a confirmed payment
- Sent → Paid: triggered when payment confirmation is received from the payment gateway
- Overdue → Paid: triggered when payment confirmation is received after the due date
- Overdue → Overdue (Escalated): triggered when the dunning sequence completes without payment
- Overdue (Escalated) → Paid: triggered when payment is eventually received after escalation
- Sent → Void: triggered by manual Finance Manager action to cancel an erroneously sent invoice
- Draft → Cancelled: triggered when a draft invoice is discarded before sending

**Terminal states:** Paid, Void, Cancelled

**Rules:**
- An invoice in Paid status cannot be voided — payment has been received; a credit note is the correct mechanism
- An invoice cannot return from Sent to Draft — once sent, it is a binding payment demand
- Rejected invoices return to an editable state equivalent to Draft; they do not have a separate Rejected terminal state

---

## 5. Universal Business Rules

| ID | Rule | Applies To |
|----|------|-----------|
| UBR-001 | An invoice must have at least one line item. An invoice with zero line items cannot be saved or sent. | Invoice creation |
| UBR-002 | The invoice total must be greater than zero. A zero-value invoice cannot be sent. | Invoice creation |
| UBR-003 | The reference number is assigned at creation and never changes. It cannot be edited, reassigned, or reused, even if the invoice is subsequently voided or cancelled. | Reference numbering |
| UBR-004 | A voided invoice produces a permanent audit record. The void reason must be recorded. | Invoice lifecycle |
| UBR-005 | An invoice cannot be sent to a Payer that does not exist in the system. A Payer record must be created before an invoice can reference them. | Invoice creation |
| UBR-006 | Monetary values are always stored and transmitted as integers in the smallest currency unit. No decimal or float representation of money is permitted anywhere in the system. | All monetary calculations |
| UBR-007 | The invoice subtotal is the sum of all line item totals (quantity × unit price). The invoice total is always subtotal + VAT amount, regardless of VAT model. **How VAT itself is computed depends on the rate model confirmed in Section 10 (Known Variations, VAT/Tax model):** under the domain's single-rate default, VAT is applied once to the whole subtotal; under a per-line-item-rate model, VAT is computed per rate group and the group totals summed before adding to the subtotal. What's universal here is the total's shape (subtotal + VAT = total) and that line items are never taxed individually under the single-rate default — not a claim that subtotal-level VAT is the only computation this domain ever uses. | Invoice calculation |
| UBR-008 | An invoice sent to a Payer constitutes a legal payment demand in most jurisdictions. The system must produce an immutable audit record at the point of sending. | Invoice sending |

---

## 6. Standard Validations

| ID | Validation | Applies To | Reason |
|----|-----------|-----------|--------|
| VAL-001 | Due date must be today or a future date at the time of invoice creation. | Invoice creation | A past due date makes the invoice immediately overdue — typically an error |
| VAL-002 | Line item quantity must be greater than zero. | Line item | A zero-quantity line item represents nothing — cannot have a business meaning |
| VAL-003 | Line item unit price must be greater than zero for standard invoices. Opening Balance invoices are exempt — they record historical debt and may have pre-calculated totals. | Line item | |
| VAL-004 | Payer email must be a valid email format. The invoice sending email is sent to this address. | Payer registration | |
| VAL-005 | Payer phone number must be in a valid format for the configured country. | Payer registration | Used as a deduplication key — invalid formats produce false non-matches |
| VAL-006 | Reference number format must be validated on generation — duplicates are not permitted. The uniqueness constraint must be enforced at the database level, not only at the application level. | Reference numbering | Race conditions between concurrent invoice creations can produce duplicates if enforced only in application code |

---

## 7. Common Personas & Roles

| Persona | Typical Responsibilities | Commonly Has Access To | Commonly Excluded From |
|---------|------------------------|----------------------|----------------------|
| Finance Manager | Creates invoices, manages Payers, monitors payment status, configures dunning schedule, runs reports | All invoices, all Payers, all payments, system configuration | Director approval decisions (cannot approve their own invoices) |
| Approver / Director | Reviews and approves or rejects high-value invoices before sending | Approval queue, full invoice details for invoices pending approval | Invoice creation, Payer management, system configuration, Payer contact details |
| Org Admin | Manages user accounts and system-level settings (approval threshold, VAT rate, email templates) | Full system configuration access | Typically does not create invoices (unless also the Finance Manager in small organisations) |

---

## 8. Typical Integration Points

| System Type | Common Examples | Purpose | Direction | Confirm With Client |
|------------|----------------|---------|-----------|---------------------|
| Payment Gateway | Stripe, Safepay, Payfast, Razorpay | Generate payment links; receive payment confirmation via webhook | Bidirectional | Which gateway does the client have an account with? Does it support webhooks? |
| Transactional Email | Resend, SendGrid, Amazon SES | Send invoice emails, approval notifications, overdue reminders | Outbound only | Any existing email platform or preference? |
| PDF Generation | Typically in-process (no external service needed at standard scale) | Generate invoice PDF attachments | Internal | Usually not an external integration — confirm if client has volume that exceeds in-process generation |
| Accounting System | Xero, QuickBooks, Sage | Export invoices and payments for financial reporting and reconciliation | Outbound (export) | Does the client use an accounting system? Is integration needed in Phase 1 or later? |
| Tax / e-Invoicing API | FBR IRIS (Pakistan), FIRS (Nigeria) | Submit invoices to government tax authorities | Outbound | Applicable only in regulated markets — see country extension documents. Do not assume this is needed. |

---

## 9. Regulatory Baseline (Country-Agnostic)

| Obligation | What It Requires | Applies To |
|-----------|-----------------|-----------|
| Audit trail | All status changes on a financial document must be logged with actor identity and timestamp. Audit logs must be immutable — they cannot be deleted or modified after creation. | All invoice status transitions |
| Financial record retention | Invoice records must be retained for a minimum period defined by the relevant jurisdiction. The most common minimum is 7 years. The specific retention period must be confirmed in the country extension document. | All invoices, payments, and approval records |
| Invoice integrity | Once an invoice is sent, its content must not be silently modified. Changes after sending require a formal process (void and reissue, or credit note). | Sent, Overdue, Paid invoices |
| Data isolation | Invoice and Payer data for one organisation must be strictly isolated from data belonging to any other organisation using the same platform. Cross-tenant data leakage is a compliance breach regardless of jurisdiction. | Multi-tenant deployments |

---

## 10. Known Variations

| Topic | Common Options | Default (if any) | Must Ask |
|-------|---------------|-----------------|---------|
| VAT / Tax model | Fixed rate applied to all line items; multiple rates per line item type; tax-exempt clients; no VAT (non-VAT-registered Issuers) | Single fixed rate applied to all line items | Yes — VAT registration status and applicable rate(s) |
| Director approval workflow | No approval threshold (any Finance Manager can send); single threshold, any approver; single threshold, specific approver; multi-level (sequential approvals above escalating thresholds) | No approval threshold | Yes — does the client require approvals? If yes, what threshold and how many approvers? |
| Dunning strategy | No automated reminders (manual only); fixed schedule (Day 1, Day 7, Day 14); configurable schedule per organisation; escalation to a collections note after N reminders | None — varies widely | Yes — does the client want automated reminders? How many and at what intervals? |
| Payer deduplication key | Phone number only; email only; either phone or email (either match triggers detection); both required (only flags if both match) | Phone number | Yes — what field should be used to detect duplicate Payers? |
| Invoice reference format | Sequential number only (00001); year-prefixed sequential (INV-2026-001); year-month-prefixed (INV-2026-06-001); client-specified format | Year-prefixed sequential | Yes — does the client have a required invoice reference format, or is the standard format acceptable? |
| Payment methods | Card only (via payment gateway link); bank transfer (manual reconciliation); multiple methods | Depends on gateway | Yes — how does the client currently receive payment? Which gateway do they use? |
| Credit note handling | Not supported (void and reissue only); credit note issued as a separate negative invoice; credit balance tracked against Payer account | Not supported in v1 | Yes — does the client need to issue credit notes, or is void-and-reissue sufficient? |
| Multi-currency | Single currency only; per-invoice currency selection; Payer-level default currency | Single currency | Yes — does the client bill in more than one currency? |

---

## 11. Typical Scope Boundaries

### Typically In First Implementation

- Payer registration and deduplication
- Invoice creation with line items, VAT calculation, and reference number generation
- Invoice approval workflow (if the client requires it)
- Invoice sending via email with PDF attachment and payment link
- Payment confirmation via payment gateway webhook
- Basic overdue detection and first reminder email
- Finance Manager dashboard showing outstanding and overdue balances
- Audit log for all invoice status changes

### Typically Deferred to Later Phase

- Credit note issuance — adds significant lifecycle complexity; void-and-reissue covers most v1 use cases (complexity, lifecycle dependency)
- Multi-currency support — requires exchange rate management and reporting complexity; defer unless required from day one (complexity)
- Accounting system integration — valuable but not blocking for operations; export can be handled manually in Phase 1 (complexity, external dependency)
- Payer self-service portal — Payers can pay via the link in the email; a full login portal is a separate product (scope, user authentication complexity)
- Configurable invoice templates and custom branding — standard template sufficient for Phase 1 (low business impact for v1)
- Bulk invoice operations — generating or sending invoices in bulk adds state management complexity; Phase 1 covers one-at-a-time (complexity)

---

## 12. Document Verification

### 12.1 Internal Consistency

- [x] Every term used in sections 3–6 is defined in section 2.
- [x] Every entity in section 3 that has meaningful states appears in section 4. Line Item has no independent lifecycle (it is immutable once the invoice is sent — this is documented in section 3 Notes).
- [x] Every entity lifecycle in section 4 has at least one initial state (Draft) and at least one terminal state (Paid, Void, Cancelled).
- [x] No UBR in section 5 contradicts a VAL in section 6.
- [x] This is a core document — no extension contradiction check required.
- [x] Section 10 (Known Variations) does not list any topic fixed by a UBR. VAT rate *and* rate model (single-rate vs. per-line-item) are both Known Variations (Section 10); UBR-007 fixes only the invoice total's shape (subtotal + VAT) and explicitly defers the VAT computation method to whichever rate model Section 10 confirms — corrected 2026-09-16 after `/judgment-check` found the original wording asserted subtotal-level VAT unconditionally, which would have contradicted Section 10's per-line-item-rate option. No conflict now.
- [x] Section 11 does not list anything as "typically in v1" that section 10 flags as high complexity. Credit note handling (high complexity) is correctly listed as typically deferred.

### 12.2 Completeness

- [x] Every entity in section 3 appears in at least one UBR or VAL (Invoice → UBR-001 through UBR-008; Line Item → UBR-001, VAL-002, VAL-003; Payer → UBR-005, VAL-004, VAL-005; Approval Record → UBR-004).
- [x] Section 10 covers all topics that commonly vary: VAT model, approval workflow, dunning, deduplication, reference format, payment methods, credit notes, multi-currency.
- [x] Section 11 has entries in both subsections.
- [x] This is a core document — country-specific regulatory check not applicable. Country obligations are in extension documents.
- [x] Edge case scenarios tested — see section 12.4.

### 12.3 Factual Accuracy Log

| Fact | Source | Verified By | Date Verified | Next Review |
|------|--------|-------------|--------------|-------------|
| 7-year financial record retention | UK Companies Act 2006, s.386 | Ascendra AI (training knowledge) | 2026-05-15 | Review if deploying in a non-UK jurisdiction |
| VAT rate statement (rate varies by jurisdiction) | HMRC / general domain knowledge | Ascendra AI (training knowledge) | 2026-05-15 | N/A — stated as variable, no specific rate asserted |

### 12.4 Edge Case Scenario Log

| Scenario | Resolved By (Section + Rule) | Outcome | Gap? |
|----------|------------------------------|---------|------|
| Finance Manager attempts to send an invoice with a zero total | Section 5, UBR-002 | Blocked — zero-value invoices cannot be sent | No |
| Two Finance Managers create invoices simultaneously — risk of duplicate reference numbers | Section 6, VAL-006 | Uniqueness enforced at database level; application-level check alone is insufficient | No |
| Client wants to issue a refund after an invoice is paid | Section 4 (lifecycle), Section 2 (Void definition) | Void not applicable post-payment; credit note or direct refund is the correct mechanism — add to BRD as a Known Variation | No gap in domain doc; confirm with client during discovery |
| Invoice sent to wrong Payer — Finance Manager wants to cancel | Section 4 (Sent → Void transition) | Supported — Finance Manager voids the invoice; a new invoice is created for the correct Payer | No |
| Payer email changes after invoice is sent | Section 3 (Payer attributes), Section 5 UBR-008 | The invoice email was already sent to the old address; Payer record can be updated for future invoices. No retroactive resend of sent invoices. | No |
| Opening Balance invoice with zero line item price | Section 6, VAL-003 | Opening Balance invoices are exempt from VAL-003 — they record historical debt as a pre-calculated total, not as itemised charges | No |
| Finance Manager attempts to approve their own high-value invoice | Section 7 (Finance Manager persona), Section 5 UBR-008 | Approval is Director-only; server-side role check must enforce this. Finance Manager cannot self-approve. | No |
| Dunning reminder sent but Payer has already paid via bank transfer (manual payment) | Section 4 (Overdue state) | If payment is not recorded in the system, the system cannot know it has been paid. Finance Manager must manually mark the invoice as Paid for bank transfers. | Known gap for manual payment methods — surface as a Known Variation in playbook |

### 12.5 Open Issues

None.

### 12.6 Mock Discovery Validation

- [ ] Did the AI ask questions about topics already covered in sections 3–6?
- [ ] Did the AI miss questions it should have asked based on section 10?
- [ ] Did the resulting BRD restate domain defaults that should have been assumed?
- [ ] Did the resulting BRD miss requirements the client would obviously need?
- [ ] Were any facts in the BRD incorrect?

**Mock discovery status:** Not yet run — playbook not yet written.
**Planned after:** Playbook completion.

---

## 13. Density & Judgment Log

*Generated by `/judgment-check` against `practitioner-guide/02-domain-discovery.md` on 2026-06-02. Findings are questions raised by the guide's own tests, not verdicts. This worked example demonstrates the section's shape — a real project's findings will differ.*

| # | Test (chapter source) | Location | Finding | Outcome | Detail |
|---|---|---|---|---|---|
| 1 | Section 5 vs. Section 10 universality test — "when you are not certain a rule is universal, put it in Section 10" | UBR-007 ("VAT is applied to the subtotal, not to individual line items") vs. Section 10, VAT / Tax model row (lists "multiple rates per line item type" as a common option) | UBR-007 states subtotal-level VAT application as a fixed, exception-free rule, but Section 10 lists per-line-item VAT rates as a real, client-selectable variation — as worded, does UBR-007 foreclose an option Section 10 says clients can choose? | Acknowledged tradeoff | UBR-007 is read as documenting the calculation order and rate application for the domain's default single-rate case, not as a claim that no other model exists — Section 10 already correctly carries "multiple rates per line item type" as a variation to confirm with the client, and no client-facing artifact has actually inherited the contradiction (Harborview uses the single-rate default; see BR-001 in `brd.example.md`). The wording is genuinely ambiguous on a strict read and worth tightening the next time this document's version is bumped, but rewriting it now risks touching a rule other sections' checks and cross-references already treat as settled. |
| 2 | Section 5 vs. Section 10 universality test, applied to the rule's own hedged language | UBR-008 ("An invoice sent to a Payer constitutes a legal payment demand in most jurisdictions...") | The rule's own wording ("in most jurisdictions") signals it isn't universal without exception, while it sits in a section defined as applying "in every implementation without exception" — should the legal-payment-demand characterization move to Section 9 (Regulatory Baseline), leaving only the immutable-audit-record requirement (which does look universal) in Section 5? | Acknowledged tradeoff | Kept in Section 5 as one rule rather than split. The enforceable half of UBR-008 — an immutable audit record must exist at the point of sending — is genuinely universal and already duplicated by Section 9's Audit Trail obligation, so nothing downstream is currently relying on the "legal payment demand" framing as if it were a hard, jurisdiction-independent fact. Splitting it into a Section 5 rule (the audit record) and a Section 9 or Section 10 note (the legal-demand characterization) is the right long-term fix, but it touches the document's own 12.2/12.4 cross-references to UBR-008 and isn't worth doing outside a real revision pass. |
| 3 | "Unverified" Factual Accuracy Log test — the label's "entire enforcement mechanism is the string itself in a table cell," so a fact whose only source is the model's own recall reads as trustworthy when it may not be | Section 12.3, both rows (7-year retention and VAT-rate-varies statement, both "Verified By: Ascendra AI (training knowledge)") | Neither fact was checked against an external authoritative source — "training knowledge" is a recall, not a verification. Should either row, particularly the retention period (used directly as a hard constraint in Harborview's brief), be marked Unverified rather than Verified? | Acknowledged tradeoff | Both facts stay marked Verified rather than relabeled Unverified. Both are long-standing, low-volatility legal facts (UK 7-year financial retention under Companies Act 2006 s.386; VAT rates varying by jurisdiction is stated as variable, not asserted as a specific number) — the practical risk of either being wrong is low, and the retention fact already carries a "Next Review" trigger for a non-UK jurisdiction. Going forward, the practice should default to marking a fact Unverified whenever its only source is model recall rather than an external check, especially for anything carrying real audit exposure — this pair is judged low-risk enough to be the exception, not the new norm. |

*Generated by `/judgment-check` against `practitioner-guide/02-domain-discovery.md` on 2026-09-16, a later independent pass over the same document. Findings are questions raised by the guide's own tests, not verdicts.*

| # | Test (chapter source) | Location | Finding | Outcome | Detail |
|---|---|---|---|---|---|
| 1 | Section 5 vs. Section 10 universality test, re-examined against UBR-007's actual computation, not just its wording | UBR-007 ("VAT is applied to the subtotal, not to individual line items") | The 2026-06-02 pass above called this an ambiguous-wording tradeoff. A closer look this pass found it's not just ambiguous — it's computationally wrong for the case Section 10 promises: VAT genuinely cannot be applied "to the subtotal, not to individual line items" once different line items carry different rates; that case requires computing VAT per rate group before summing. Does this rise from "acceptable as worded" to "needs a real fix"? | Revised | Fixed UBR-007 to state only what's actually universal (the total's shape, subtotal + VAT) and to explicitly defer the computation method to Section 10's confirmed rate model. Section 12.2's cross-check updated to match. No downstream artifact needed to change — Harborview's own BRD/architecture already only use the single-rate case, so this was a domain-document-only fix. Logged as its own new finding in this later pass, not an edit to the 2026-06-02 row above — a changed assessment gets a new row in a new block, never a rewrite of an earlier one. |
