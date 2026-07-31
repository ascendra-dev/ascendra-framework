# Domain Discovery Playbook — Invoice Payment (Core)

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-05-10 | Ascendra AI | Initial generated version |

---

## 1. Playbook Scope

### 1.1 What This Playbook Covers

This playbook covers a domain discovery session for the Invoice Payment (Core) domain — the business processes by which one organisation bills another for goods or services rendered and subsequently collects payment. This is a domain education session, not a client requirement session. The PO teaches the AI what is universally true about invoice payment systems; client-specific variations are captured separately during BRD discovery.

### 1.2 Documents to Load

| Document | Purpose |
|----------|---------|
| `projects/{PROJECT_CODE}/brief.md` | Domain name, sector, and problem context for this engagement |

### 1.3 What This Playbook Does Not Cover

- Client-specific requirements, thresholds, approval structures, or configurations (captured in requirement discovery via `/run-brd-discovery`)
- Project-specific entities or rules (captured in the BRD)
- Technology choices or architecture (captured in the architecture document)
- Country-specific e-invoicing mandates or tax regulations — these are covered in country extension documents, not in the core domain document
- Payroll, purchase order management, or inventory — adjacent domains not covered by this playbook

---

## 2. Pre-Session Checklist

- [ ] Brief loaded — domain name and problem context confirmed.
- [ ] Check for an existing domain knowledge document at `projects/{PROJECT_CODE}/domain/`. If one exists, this session is updating it — note which sections need to change.
- [ ] Check for a prior domain discovery state at `projects/{PROJECT_CODE}/domain/domain-discovery-state.md`. If it exists, this is a resumed session — load it and follow the Resume Instructions.
- [ ] The PO (or domain expert) understands this session is about the Invoice Payment domain universally — not about this specific client's implementation.

---

## 3. Session Opening

Tell the PO:
> "We're going to define the universal rules and structure of the Invoice Payment domain. I'll ask questions about how invoice billing and payment collection works in general — not about this specific client. The answers will form a reusable domain knowledge document that all future invoice payment projects will use. Where I already have knowledge about this domain, I'll confirm it with you. Where I don't, I'll ask you to teach me."

Confirm with the PO:
- [ ] The domain name: "Invoice Payment (Core)" — or a correction if the PO uses a different name
- [ ] The primary business problem: billing one organisation for services rendered and collecting payment
- [ ] Exclusions: payroll, purchase order matching, subscription billing with auto-charge, and consumer (B2C) payment flows are all adjacent domains excluded from this session

---

## 4. Domain Scope

**Primary question:**
> "In 2–3 sentences, what does the Invoice Payment domain cover? What business problem does it address and what type of system does it describe?"

**Follow-ups if needed:**
- "What does this domain explicitly NOT cover? Name adjacent domains or capabilities that should not be in this document — for example, is subscription billing in scope?"
- "Is there a meaningful sub-domain here that should be a separate document? For example, should B2C consumer payments be split out from B2B invoicing?"

**State note:** Record the domain scope statement and exclusions verbatim. These become Section 1.1 and 1.2 of the domain knowledge document. If the PO names a sub-domain that should be its own document, flag it for a follow-on playbook.

---

## 5. Core Entities

**Opening prompt:**
> "Based on my knowledge of the Invoice Payment domain, the core entities are: Invoice, Line Item, Payer, and Approval Record. Does this match your understanding? Are there entities I'm missing, or entities on this list that don't apply?"

---

### 5.1 Invoice

- "What is an Invoice? Give me a business-level definition — not a technical one."
- "What are its standard business-meaningful attributes? Think about what a Finance Manager looks at when they open an invoice — not database fields."
- "How does an Invoice relate to a Payer? To Line Items?"
- "Is there anything important about how monetary values are stored — for example, should they be stored as decimals or as integers?"

**State note:** Record definition, attributes, and any PO corrections. The AI knows that Invoice is the primary billing record; if the PO uses a different term (e.g. "Bill", "Sales Invoice"), record the synonym and use the PO's term going forward within this session.

---

### 5.2 Line Item

- "What is a Line Item? What does it represent on an invoice?"
- "What are the standard attributes of a Line Item? Is VAT applied at the line item level or at the invoice total level?"
- "Can a Line Item exist without a parent Invoice?"
- "What happens to a Line Item once an invoice has been sent — can it be edited?"

**State note:** Record whether VAT is calculated per line item or at invoice total level — this affects UBR-007. Confirm immutability of line items once sent.

---

### 5.3 Payer

- "What is a Payer? In a B2B context, is this always an organisation or can it be an individual?"
- "What are the standard attributes of a Payer — what information do you need to know about them before you can send them an invoice?"
- "Can a Payer have multiple invoices at different stages at the same time?"
- "What happens to a Payer record once all their invoices are closed — should it persist for future invoices?"

**State note:** Confirm B2B vs B2C applicability. Record whether Payer persistence matters (it does — Payer records persist for future invoices).

---

### 5.4 Approval Record

- "When a high-value invoice requires sign-off before sending, what record do we create to track that decision? What do we call it?"
- "What are the key attributes of that record — who approved it, when, what their decision was, and if they rejected it, why?"
- "Can one invoice have multiple approval records — for example, if it requires sign-off from two different people?"

**State note:** Record the standard name (Approval Record) and confirm whether the domain supports single-approver or multi-level approval as the default model. Note that multi-level is a known variation — the record structure must support both.

---

## 6. Lifecycle & Status Model

**Opening prompt:**
> "For the Invoice entity — the only entity in this domain with a meaningful independent lifecycle — I'll walk through the standard states and transitions. Tell me where my understanding is wrong or incomplete."

---

### 6.1 Invoice Lifecycle

- "What states can an Invoice be in? I know of: Draft, Pending Approval, Approved, Sent, Overdue, Paid, Void, and Cancelled. Am I missing any?"
- "What triggers the transition from Draft to Sent?"
- "What triggers the transition from Draft to Pending Approval — and what happens after a Director approves or rejects it?"
- "What triggers Sent → Overdue? Is it automatic on the due date or does someone trigger it manually?"
- "What are the terminal states — states an Invoice can enter but never leave?"
- "Can a Paid invoice be voided? What is the correct mechanism for handling an invoice that was paid in error?"
- "Can a Sent invoice be edited? If a Finance Manager made an error, what is the correct process?"

**State note:** Record the complete state list, all transitions with triggers, all terminal states, and the rules on what cannot happen (Paid → Void; Sent → Draft). If the PO mentions 'Overdue (Escalated)' as a distinct state from 'Overdue', record it. Flag any transition where the PO's answer contradicts what I know — record both the standard and the correction.

---

## 7. Universal Business Rules

**Opening prompt:**
> "I'll read out the business rules I know apply universally in the Invoice Payment domain. Tell me which ones you'd add, remove, or correct."

Pre-populated list to present:
1. An Invoice must have at least one Line Item. An invoice with zero line items cannot be saved or sent.
2. The invoice total must be greater than zero. A zero-value invoice cannot be sent.
3. The reference number is assigned at creation and never changes — it cannot be edited, reassigned, or reused even if the invoice is voided.
4. A voided invoice produces a permanent audit record — the void reason must be recorded.
5. An invoice cannot be sent to a Payer that does not exist in the system.
6. Monetary values are always stored as integers in the smallest currency unit. No decimal or float representation of money is permitted in the system.
7. VAT is applied to the invoice subtotal, not to individual line items. Total = subtotal + VAT amount.
8. An invoice sent to a Payer constitutes a legal payment demand. The system must produce an immutable audit record at the point of sending.

**For each rule proposed or raised:**
- "Does this rule apply in every implementation of this domain without exception?"
- "If a client wanted to do something different — would that be a valid variation or a red flag?"

**For any rule the PO adds:**
- "Is this universal (always true) or does it vary between clients?"
- "Can you state it as a single sentence beginning with 'An [entity] must...' or 'The system must...'?"

**State note:** Record only rules that are universal. Rules that vary between clients belong in Section 12 (Known Variations). If the PO removes a rule from the list, record the removal and the reason.

---

## 8. Standard Validations

**Opening prompt:**
> "What validations are always enforced in invoice payment systems — things the system must always check before allowing an action? I have a starting list; tell me what to add, change, or remove."

Pre-populated list to present:
1. Due date must be today or a future date at invoice creation.
2. Line item quantity must be greater than zero.
3. Line item unit price must be greater than zero for standard invoices (Opening Balance invoices may be exempt).
4. Payer email must be a valid email format.
5. Payer phone number must be a valid format for the configured country.
6. Invoice reference number uniqueness must be enforced at the database level — not only in application code.

**For each validation:**
- "Why is this validation enforced? What business consequence does it prevent?"
- "Does this apply in every implementation or only in some? For example, does the phone number format check apply if the client doesn't use phone numbers as a deduplication key?"

**State note:** Record only validations with a clear business reason that apply universally. Technical constraints (field length limits, data type checks) belong in technical design, not here.

---

## 9. Common Personas & Roles

**Opening prompt:**
> "The standard roles in the Invoice Payment domain are typically: Finance Manager, Approver (or Director), and Org Admin. In some implementations there is also a Payer-facing role for a self-service portal. Does this match your experience? Are there roles I'm missing?"

**For each role confirmed or added:**
- "What are their typical responsibilities?"
- "What do they typically have access to?"
- "What are they typically excluded from — for example, can a Finance Manager approve their own invoices?"

**State note:** Record these as starting points — they will be confirmed with the client during requirement discovery. Do not assert that all roles will exist for every client. The Payer role is often not a system user at all — Payers interact via email links only.

---

## 10. Typical Integration Points

**Opening prompt:**
> "What external systems or services are commonly used in Invoice Payment systems? I'll propose the ones I know — correct or extend the list."

Pre-populated list to present:
1. Payment Gateway (Stripe, Safepay, Payfast, Razorpay) — generate payment links; receive payment confirmation via webhook
2. Transactional Email (Resend, SendGrid, Amazon SES) — send invoice emails, approval notifications, overdue reminders
3. PDF Generation — generate invoice PDF attachments (typically in-process, not an external service)
4. Accounting System (Xero, QuickBooks, Sage) — export invoices and payments for reconciliation
5. Tax / e-Invoicing API (FBR IRIS for Pakistan, FIRS for Nigeria) — submit invoices to government tax authorities in regulated markets

**For each integration:**
- "What is the purpose of this integration — what does it do for the business?"
- "Is data flowing in, out, or both?"
- "Is this always needed or does it vary by client?"

**State note:** Record as common but not guaranteed for every client. These become confirmation questions during BRD discovery. Note which integrations require client credentials and which have onboarding lead times.

---

## 11. Regulatory Baseline

**Opening prompt:**
> "What compliance or regulatory obligations apply to invoice payment systems regardless of which country the client is in?"

Pre-populated list to confirm:
1. Audit trail — all status changes on a financial document must be logged with actor identity and timestamp. Logs must be immutable.
2. Financial record retention — invoice records must be retained for a minimum period defined by the jurisdiction (commonly 7 years).
3. Invoice integrity — once sent, invoice content must not be silently modified.
4. Data isolation — in multi-tenant systems, one organisation's invoice data must not be accessible to another organisation.

**For each obligation raised:**
- "Does this apply universally, or only in specific countries or sectors?"
- "What does it require the system to do?"

**State note:** Record only country-agnostic obligations here. Country-specific requirements (e.g. FBR mandatory e-invoicing in Pakistan) belong in a country extension document, not in the core domain document.

---

## 12. Known Variations

**Opening prompt:**
> "What aspects of the Invoice Payment domain commonly differ between client implementations? These are the things we must always ask during requirement discovery — we cannot assume them."

Pre-populated list to confirm and extend:
1. VAT / Tax model — fixed rate on all items vs multiple rates vs no VAT vs tax-exempt clients
2. Director / Approver workflow — no approval required vs single threshold with any one approver vs multi-level approval at escalating thresholds
3. Dunning strategy — no automated reminders vs fixed schedule vs configurable schedule per organisation
4. Payer deduplication key — phone number, email, either match, or both required
5. Invoice reference format — sequential number, year-prefixed, year-month-prefixed, or client-specified
6. Payment methods — card via payment gateway link vs bank transfer (manual reconciliation) vs multiple methods
7. Credit note handling — not supported (void-and-reissue only) vs credit note as a separate negative invoice
8. Multi-currency — single currency only vs per-invoice currency selection

**For each variation:**
- "What are the common options?"
- "Is there a most common default?"
- "Would it be a red flag if a client wanted a very unusual option here?"

**State note:** These directly become discovery questions in the BRD playbook. The more complete this section, the fewer gaps in discovery. Every variation listed here must appear as a question topic in the BRD playbook.

---

## 13. Typical Scope Boundaries

**Questions:**
- "What capabilities does almost every client need in a first implementation of an invoice payment system?"
- "What capabilities are commonly wanted but rarely built first? Why are they typically deferred?"

Pre-populated starting list to confirm:
- **Typically in v1:** Payer registration and deduplication; invoice creation with line items, VAT, and reference number generation; invoice approval workflow; invoice sending via email with PDF and payment link; payment confirmation via webhook; basic overdue detection and first reminder; Finance Manager dashboard; audit log
- **Typically deferred:** Credit note issuance; multi-currency support; accounting system integration; Payer self-service portal; configurable invoice templates; bulk invoice operations

**State note:** Record both what is typically in scope and what is typically deferred. Include the reason for deferral — complexity, external dependency, low v1 business impact.

---

## 14. Session Close

**Closing steps:**

1. **Summarise the domain:** Restate the domain scope, key entities, lifecycle, and top Known Variations in 5–6 bullet points. Ask the PO to confirm or correct:
   > "Here is my summary of what we covered. Please correct anything I've got wrong: [bullet summary]"

2. **List open items:** Read out anything flagged as uncertain or needing follow-up during the session.

3. **Confirm readiness:**
   > "Is there anything about the Invoice Payment domain I should know before generating the domain knowledge document? Any edge cases, unusual client situations, or common misconceptions that should be documented?"

**Domain knowledge drafting note:** After writing the state file, run `/gen-domain-knowledge` with the path to the state file. The command uses this state as primary input, enriched by AI knowledge where the PO did not address a topic.

---

## 15. Playbook Verification

- [x] Every section of the domain knowledge template (sections 1–11) is covered by at least one question in this playbook.
  - Section 1 (Domain Overview) → Playbook section 4
  - Section 2 (Terminology) → Implicitly surfaced in sections 5 and 6 as PO names entities and states
  - Section 3 (Core Entities) → Playbook section 5
  - Section 4 (Standard Lifecycle) → Playbook section 6
  - Section 5 (Universal Business Rules) → Playbook section 7
  - Section 6 (Standard Validations) → Playbook section 8
  - Section 7 (Common Personas) → Playbook section 9
  - Section 8 (Typical Integration Points) → Playbook section 10
  - Section 9 (Regulatory Baseline) → Playbook section 11
  - Section 10 (Known Variations) → Playbook section 12
  - Section 11 (Typical Scope Boundaries) → Playbook section 13
- [x] The playbook explicitly separates universal domain knowledge from client-specific requirements — no questions ask about "what the client wants."
- [x] Section 12 (Known Variations) questions are clear enough that the answers will directly feed into BRD playbook question topics. All 8 known variations have corresponding BRD discovery topics.
