# Domain Discovery State — Invoice Payment (Core)

---

**Example note:** This example shows the state file as saved after Session 1 of a two-session discovery. Sections 10–13 are pending. The confidence score is below the 70% threshold — a second session is required before generating the domain knowledge document. The example demonstrates what substantive "PO stated:" distillations look like, what an AI Knowledge Correction entry looks like, and how the Pre-Generation Verification gates a partially complete session.

---

## 1. Session Context

| Field | Value |
|-------|-------|
| Project Code | HARBORVIEW-INV-001 |
| Domain Name | Invoice Payment (Core) |
| Domain Playbook Used | `projects/HARBORVIEW-INV-001/domain/invoice-payment-domain-playbook.md` |
| Target Output | `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Covered |
|-----------|------|-------------|-----------------|
| 1 | 2026-05-12 | Ascendra AI + PO | Sections 4–9 (Domain Scope through Common Personas) |

---

## 3. Domain Confidence Score

**Current score:** 55%

**What would raise it:**
- Section 10 (Typical Integration Points) — pending. Payment gateway and email integrations are known to the AI, but not yet confirmed by the PO for this domain.
- Section 11 (Regulatory Baseline) — pending. AI knows about audit trail and retention obligations, but PO has not confirmed which apply in their context or whether any country-specific rules exist.
- Section 12 (Known Variations) — pending. **Critical gap.** This section directly feeds the BRD playbook. Until the PO confirms what varies between clients, the BRD playbook cannot be fully populated and discoveries will miss common variation topics.
- Section 13 (Typical Scope Boundaries) — pending.

---

## 4. Playbook Progress

### Section 4 — Domain Scope
**Status:** Covered

**PO stated:**
The Invoice Payment domain covers the end-to-end process of one business billing another for goods or services rendered, sending that billing document, and subsequently collecting payment. The PO confirmed the domain does NOT cover: payroll, purchase order management, or inventory. They explicitly flagged that subscription billing with automatic recurring charge is a distinct domain — even if the same client organisation uses both, the two must be separate domain documents. Consumer (B2C) payment flows are also excluded — all known client engagements in this domain involve B2B billing only.

---

### Section 5 — Core Entities
**Status:** Covered

**PO stated:**
PO confirmed the AI's proposed entity list (Invoice, Line Item, Payer, Approval Record) with one addition: Opening Balance invoice is a distinct invoice subtype, used only during client onboarding to record outstanding debt carried forward from a prior system. It should be documented in the domain knowledge document as a named subtype with its own validation exception (VAL-003 — unit price > 0 does not apply because the total is pre-calculated from historical data).

Line Item: PO confirmed quantity × unit price = line total. VAT is calculated at invoice level on the subtotal, not at the line item level. Line items are immutable once the invoice is sent — to correct an error, the Finance Manager voids the invoice and creates a new one.

Payer: Always an organisation in the B2B context. PO noted that "Payer" is sometimes called "Client" or "Customer" in client conversations — the domain document should acknowledge these synonyms to prevent terminology drift.

Approval Record: PO confirmed the entity. Stated that in every implementation encountered, the approval record belongs to a single invoice and is created when the first authorised person acts. Multi-level approval (requiring two or more signatories in sequence) exists but is uncommon — it should be a Known Variation, not the domain default.

---

### Section 6 — Lifecycle & Status Model
**Status:** Covered

**PO stated:**
Invoice lifecycle states confirmed: Draft, Pending Approval, Approved, Sent, Overdue, Overdue (Escalated), Paid, Void, Cancelled.

**One correction to AI knowledge — see Section 5:** AI initially presented "Overdue" as a single state, but the PO stated that Overdue and Overdue (Escalated) must be distinct states. Overdue fires automatically when the due date passes with no payment. Overdue (Escalated) fires when the dunning sequence completes with no payment — signalling that automated follow-up is exhausted and Finance Manager manual action is required. Both states can still transition to Paid if payment arrives. Conflating them would prevent the system from distinguishing active chasing from cases requiring human intervention.

Transitions confirmed by the PO: Draft → Sent (direct, below threshold or no threshold configured); Draft → Pending Approval (total at or above the approval threshold); Pending Approval → Approved (Director action); Pending Approval → Rejected (invoice returns to an editable state equivalent to Draft); Approved → Sent (Finance Manager action after approval); Sent → Overdue (automatic, due date passes without payment confirmation); Overdue → Overdue (Escalated) (automatic, dunning sequence complete); Sent / Overdue / Overdue (Escalated) → Paid (payment confirmation received from gateway or manual mark); Sent → Void (Finance Manager manual action).

Terminal states: Paid, Void, Cancelled. PO stated explicitly: a Paid invoice cannot be voided. Once payment is received, the obligation is settled — a credit note or direct refund is the correct post-payment mechanism.

---

### Section 7 — Universal Business Rules
**Status:** Covered

**PO stated:**
All eight AI-proposed UBRs confirmed without removal or correction:
1. Invoice must have at least one line item — confirmed.
2. Invoice total must be greater than zero — confirmed.
3. Reference number is assigned at creation and never changes — confirmed.
4. Voided invoice produces a permanent audit record — confirmed.
5. Invoice cannot be sent to a Payer not in the system — confirmed.
6. Monetary values stored as integers in the smallest currency unit — confirmed. PO added: "this is non-negotiable; decimal float rounding bugs in invoice totals cause real accounting errors and always get traced back to this mistake."
7. VAT applied to invoice subtotal, not to individual line items — confirmed.
8. Invoice sending produces an immutable audit record — confirmed.

No new UBRs added. No corrections to the pre-populated list.

---

### Section 8 — Standard Validations
**Status:** Covered

**PO stated:**
All six AI-proposed validations confirmed:
1. Due date must be today or a future date — confirmed. PO: a past due date on a new invoice is almost always operator error; it should be blocked, not warned.
2. Line item quantity > 0 — confirmed.
3. Line item unit price > 0 for standard invoices; Opening Balance invoices are exempt — confirmed.
4. Payer email must be a valid format — confirmed.
5. Payer phone number must be valid for the configured country — confirmed with a clarification: this validation should be a hard block only if phone number is the configured deduplication key. If the deduplication key is email, an invalid phone number should be a soft warning only — the Payer can still be registered.
6. Reference number uniqueness enforced at the database level — confirmed. PO: "application-level uniqueness checks alone are not safe under concurrent invoice creation; the constraint must be in the database."

---

### Section 9 — Common Personas & Roles
**Status:** Covered

**PO stated:**
Three standard roles confirmed: Finance Manager, Approver/Director, Org Admin.

Finance Manager: Creates invoices, manages Payers, monitors payment status, configures reminder schedules. In small organisations this is often the business owner. PO stated the Finance Manager must never be able to approve their own high-value invoices — self-approval is a control failure the domain document should call out explicitly.

Approver/Director: Reviews and approves or rejects high-value invoices before sending. Access is limited to the approval queue and read-only invoice view. Cannot create invoices, manage Payers, or change system configuration.

Org Admin: Manages users and system-wide settings such as the approval threshold, VAT rate, and email templates. May be the same person as the Finance Manager in small organisations, or a separate IT or operations person in larger ones.

PO added: In some implementations there is a Payer-facing role if a self-service portal is built. In the majority of Phase 1 implementations, however, Payers are not system users — they interact only via the payment link embedded in the invoice email.

---

### Section 10 — Typical Integration Points
**Status:** Pending

---

### Section 11 — Regulatory Baseline
**Status:** Pending

---

### Section 12 — Known Variations
**Status:** Pending

---

### Section 13 — Typical Scope Boundaries
**Status:** Pending

---

## 5. AI Knowledge Corrections

| Topic | AI Assumption | PO Correction | Confidence |
|-------|--------------|---------------|------------|
| Invoice Overdue states | Modelled "Overdue" as a single state — past due date, either paid or still outstanding. No distinction between active dunning and exhausted dunning. | Overdue and Overdue (Escalated) are distinct states. Overdue = past due date, automated reminders still in progress. Overdue (Escalated) = dunning sequence complete, no payment received, Finance Manager must act manually. Both states can transition to Paid if payment eventually arrives. The domain knowledge document must define both states clearly. | High — PO was unambiguous and gave a concrete example of why conflation causes problems in practice |

---

## 6. Open Issues

| # | Issue | Section | Owner | Status |
|---|-------|---------|-------|--------|
| DI-001 | Should credit note handling be documented as a domain pattern (a second financial document type with its own lifecycle), or always treated as a deferred/extension capability and listed only in Known Variations and Typical Scope Boundaries? PO was uncertain — they have seen clients ask for credit notes in v1 but find the lifecycle complexity consistently underestimated. | Section 12 (Known Variations) | PO to confirm before Session 2 | Open |

---

## 7. Resume Instructions

**Next session starts at:** Section 10 — Typical Integration Points

**Before resuming:**
- [ ] Load: domain playbook at `projects/HARBORVIEW-INV-001/domain/invoice-payment-domain-playbook.md`
- [ ] Load: this file (already done if you are reading this)
- [ ] Review section 4 above to rebuild context on what the PO stated in Session 1
- [ ] Review section 5 (AI Knowledge Corrections) — apply the Overdue / Overdue (Escalated) distinction when discussing and generating the lifecycle section

**Sections still pending (do not skip):**
- Section 10 — Typical Integration Points
- Section 11 — Regulatory Baseline
- Section 12 — Known Variations (**critical — do not skip; this section directly feeds the BRD playbook**)
- Section 13 — Typical Scope Boundaries

**Open issues to revisit:**
- DI-001: PO committed to confirm before Session 2 whether credit notes should be a core domain pattern or documented as a Known Variation only (see Section 6)

---

## 8. Pre-Generation Verification

- [x] Section 1 (Session Context): Project Code, Domain Name, and Target Output path are all filled
- [ ] Section 3 (Domain Confidence Score): Score is ≥ 70%. **FAIL — current score is 55%. Sections 10–13 are pending. Do not generate. Schedule Session 2 and update this file before proceeding.**
- [x] Section 4 (Playbook Progress): Every section marked Covered has a substantive "PO stated:" paragraph — Sections 4–9 all have substantive entries
- [x] Section 5 (AI Knowledge Corrections): Reviewed — one correction recorded (Overdue / Overdue (Escalated) distinction)
- [ ] Section 6 (Open Issues): DI-001 is Open. **Must be resolved in Session 2 or explicitly acknowledged as a gap to carry forward into the generated document.**
- [ ] Section 7 (Resume Instructions): Next session start point is filled — **not yet generation-ready. Return here after Session 2 and update this file before running `/gen-domain-knowledge`.**
