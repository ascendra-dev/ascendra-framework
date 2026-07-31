# BRD Discovery State — Invoice Management System

---

**Example note:** This example shows the state file as saved after Session 2 of a two-session discovery — all topics and journeys covered. Two open questions remain (OQ-001 and OQ-002) and are listed in Section 7. No scope risk flags triggered during either session. The Resume Instructions are updated to "Generation ready" after Session 2. This represents the state from which `/gen-brd` would produce the BRD.

---

## 1. Project & Session Context

| Field | Value |
|-------|-------|
| Project Name | Invoice Management System |
| Project Code | HARBORVIEW-INV-001 |
| Client | Harborview Consulting Ltd |
| Domain | Invoice Payment (Core) |
| Sector | Professional services consultancy (UK) |
| Domain docs loaded | `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md` |
| BRD playbook used | `projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Covered |
|-----------|------|-------------|-----------------|
| 1 | 2026-06-05 | Ascendra AI + James Okafor (Finance Manager) | Section 3 (Opening), Section 4 topics 4.1–4.5, Journey 6.1 |
| 2 | 2026-06-07 | Ascendra AI + James Okafor + Sarah Chen (Director, as observer) | Section 4 topics 4.6–4.8, Journeys 6.2–6.3, Section 7 (Integration Confirmation), Section 8 (Output Confirmation) |

---

## 3. Playbook Progress

### Section 4 — Core Discovery Questions

| # | Topic | Status | Client Answer (distilled) |
|---|-------|--------|--------------------------|
| 4.1 | VAT and Tax Model | Covered | VAT-registered. Single rate: 20% applied to all invoice subtotals. No exempt clients, no cross-border VAT variation. All clients are UK-based companies. |
| 4.2 | Approvals | Covered | Director approval required for invoices at or above £10,000. Any one of the three Directors may approve — single approver sufficient. Finance Manager cannot approve. Rejection must include a reason; Finance Manager can edit and resubmit. No escalation chain above Director level. |
| 4.3 | Overdue Reminders and Dunning | Covered | Wants automated overdue reminders. First reminder within 24 hours of due date. Would like to configure the schedule (number of reminders and intervals). After the sequence completes, Finance Manager should be alerted that manual follow-up is required. No interest calculation needed. One universal schedule for all clients (no per-client variation). |
| 4.4 | Payer Management and Deduplication | Covered | Manages ~30 active client companies. Has an existing CSV export from their spreadsheet to import. Deduplication key: phone number (confirmed — they track one phone number per client and rely on it). Has not had duplicate client issues in practice, but wants a soft warning on registration if a match is detected. Several clients have outstanding balances from the current system that need to be carried over. |
| 4.5 | Invoice Reference Format | Covered | No fixed format required from accounting side. Accepts AI-proposed standard: INV-YYYY-NNNNNN, continuous sequence (no annual reset). Migrating from spreadsheet — no prior reference numbers to continue from; can start at INV-2026-000001. |
| 4.6 | Payment Methods and Gateway | Covered | Clients currently pay by bank transfer (manually reconciled) and occasionally by card. Wants card payments online via Stripe. Has an existing Stripe account — will provide API keys. Bank transfer tracking: yes, Finance Manager should be able to manually mark an invoice as Paid for bank transfers. No multi-currency — all clients billed in GBP. |
| 4.7 | Credit Notes | Covered | Does not currently issue formal credit notes. Void-and-reissue is acceptable for Phase 1. Confirmed: credit note issuance is out of scope. |
| 4.8 | Multi-Currency | Covered | Single currency only: GBP. All clients and all payments are in GBP. Confirmed out of scope. |

### Section 6 — Journey Walkthroughs

| # | Journey | Status | Key Points from Client |
|---|---------|--------|------------------------|
| 6.1 | Standard Invoice — Creation to Payment | Covered | Finance Manager creates invoice by picking a client, typing line items, setting due date. Currently does this in Excel then emails a PDF manually. Expects to see the invoice PDF in the email the client receives. Client pays by bank transfer (most common) or will now pay via Stripe link. Finance Manager currently knows about payment by checking bank statement — wants to see this automatically in the system. |
| 6.2 | High-Value Invoice — Approval Flow | Covered | Director approval currently done by forwarding the Excel invoice by email and waiting for a reply. No formal tracking. Director (Sarah Chen, present in Session 2) confirmed: she reviews on her phone frequently and needs the approval to work on mobile. Any one of the three Directors can approve — no escalation. Directors want email notification when approval is needed. Rejection: Finance Manager needs to know the reason clearly. |
| 6.3 | Overdue Invoice — Reminder to Resolution | Covered | Currently sends overdue reminders manually — "when I remember to" (James's words). Wants Day 1, Day 7, Day 14 reminders as the default schedule, but wants to be able to change it. After Day 14 reminder, Finance Manager wants a system alert that manual follow-up is now required. No automation beyond that point. Key account exception: James mentioned one client (not named) he would prefer to call rather than email when overdue. Confirmed: per-client dunning is NOT required in Phase 1 — he will call them manually after the Day 14 escalation alert. |

---

## 4. Scope Risk Flags

None triggered in either session.

| Flag ID | Flag Name | Triggered By | Client Response | Status |
|---------|-----------|-------------|----------------|--------|
| — | — | — | — | — |

Context: SRF-001 (government e-invoicing) was implicitly checked — Harborview invoices private companies only, UK-based. Not applicable. SRF-002 (multi-rate VAT) not triggered — single 20% rate confirmed. SRF-003 (multi-level approval) not triggered — any one Director is sufficient. SRF-004 (per-client dunning) was close: James mentioned one preferred-call client but confirmed per-client dunning is not needed in Phase 1. SRF-005 (multi-currency) not triggered — GBP only. SRF-006 (credit notes in Phase 1) not triggered — void-and-reissue accepted.

---

## 5. Integration Confirmation

| Integration | Status | Notes |
|------------|--------|-------|
| Payment Gateway — Stripe | Confirmed In | Existing Stripe account. James will provide publishable key, secret key, and webhook signing secret before Sprint 1 payment stories begin. |
| Transactional Email — Resend | Confirmed In | No existing email service. Ascendra delivery team will set up Resend account and verify the sending domain. James confirmed the sending address should be invoices@harborviewconsulting.co.uk — domain verification required. Lead time: 24–72 hours for DNS propagation. |
| PDF Generation | Confirmed In | In-process. No external service required. James confirmed invoices should include Harborview logo — logo file to be provided before Sprint 1 PDF story. |
| Accounting System (Xero) | Confirmed Out (Phase 1) | James mentioned Xero as a future need. Confirmed it is not in scope for Phase 1 — he will reconcile manually using the system's invoice records. Noted as a Phase 2 item. |
| Tax / e-Invoicing API | Confirmed Out | UK-based, invoicing private companies only. No government e-invoicing mandate applies. |

---

## 6. Output Confirmation

**Status:** Completed (Session 2)

| Output | Confirmed? | Notes |
|--------|-----------|-------|
| Outstanding invoices list (live view) | Yes | Finance Manager dashboard widget — total outstanding balance and list of unpaid invoices |
| Overdue invoices list (live view) | Yes | Finance Manager dashboard widget — total overdue balance |
| Invoices due in next 7 days | Yes | Finance Manager dashboard widget |
| Recently paid invoices | Yes | Finance Manager dashboard — last 5–10 paid invoices |
| Director approval queue | Yes | Directors need a view showing invoices pending their approval with full invoice details |
| Aged debtor report | No | James said "maybe later" — not needed in Phase 1 |
| CSV / Excel export | No | James confirmed he will use the system for records; no export needed in Phase 1 |
| Invoice sent confirmation (to Finance Manager) | No | James: "I know I sent it — I don't need an email confirmation" |
| Approval request notification (to Directors) | Yes | Email to all three Directors when an invoice is submitted for approval |
| Approval decision notification (to Finance Manager) | Yes | Email when a Director approves or rejects — Finance Manager wants to know immediately |
| Overdue reminder sent (to Payer) | Yes | Email to Payer per configured schedule |
| Dunning sequence complete alert (to Finance Manager) | Yes | In-system alert + email. James wants to know when he needs to call someone. |
| Payment received notification (to Finance Manager) | Yes | In-system dashboard update. No separate email needed — James checks the dashboard. |

---

## 7. Open Questions

| # | Question | Raised In | Owner | Due | Notes |
|---|---------|---------|-------|-----|-------|
| OQ-001 | Email addresses for all three Directors — needed to configure the approval notification recipients in the system. James does not have them to hand. | Session 2 | James Okafor (Finance Manager) | 2026-06-10 | James committed to email them to the delivery team by 2026-06-10. Without these, the Director approval notification cannot be configured or tested. |
| OQ-002 | Resend API key, sending domain DNS records, and Harborview logo file for PDF invoices — needed before Sprint 1 email and PDF stories can begin. | Session 2 | Ascendra delivery team (Resend setup) + James Okafor (logo file) | Before Sprint 1 email stories begin | Two parts: (1) Ascendra creates Resend account and provides DNS records to James for verification. (2) James provides logo file (PNG preferred, 300×100px or similar). |

---

## 8. Captured Client Answers (BRD Input)

---

### VAT and Tax Model

Harborview is VAT-registered. VAT is charged at 20% on all invoice subtotals. There are no VAT-exempt services and no VAT-exempt clients. All clients are UK-based companies — no cross-border or zero-rated scenarios. James confirmed: "all our invoices have VAT at 20%, no exceptions." This is a fixed configurable business rule, not a line-item-level variation.

---

### Approvals

Director approval is required for any invoice totalling £10,000 or more (inclusive — James confirmed: "at £10,000 and above, it needs sign-off"). Any one of the three Directors may approve — a second Director's approval is not required. Finance Manager cannot approve their own invoices. When rejected, the Director must provide a reason; the Finance Manager can edit and resubmit.

Sarah Chen (Director, Session 2 observer) confirmed she typically reviews approval requests on her iPhone. The approval action must work on a mobile browser without needing a desktop.

James asked: "what if all three Directors are on holiday at the same time?" Acknowledged as a practical edge case — the system will not handle substitute approvers in Phase 1. Finance Manager will need to contact a Director directly in that scenario. Recorded as an explicit out-of-scope exclusion.

---

### Overdue Reminders and Dunning

Automated overdue reminders confirmed. First reminder within 24 hours of the due date passing. Preferred default schedule: Day 1, Day 7, Day 14 after due date. James wants to be able to change this schedule — the number of reminders and the intervals should be configurable system settings, not hardcoded.

After the final reminder in the sequence, James wants a system alert that tells him manual follow-up is now required. He described this as: "I want the system to tell me it's given up so I know to pick up the phone."

No interest calculation. No per-client reminder schedules (James mentioned one key account he calls personally but confirmed he will call them after the escalation alert — no system variation needed).

---

### Payer Management and Deduplication

Harborview has approximately 30 active client companies. James has an existing spreadsheet with a list of all clients that can be exported to CSV for import. Deduplication key: phone number (James: "each company has one main number we use — that's how we'd know if someone added the same client twice").

Soft warning on duplicate detection: if the phone number already exists, show a warning but allow the Finance Manager to proceed if they confirm. Do not block outright — there may be legitimate cases where two entries share a phone (e.g. a client group with a shared switchboard).

Opening balances: several clients have unpaid amounts from before the new system. James wants to carry these forward as Opening Balance invoices — not re-send them, just record the amount outstanding against those clients. Confirmed: Opening Balance invoices do not go through the standard invoice lifecycle (no sending, no Director approval). They are record-only entries.

---

### Invoice Reference Format

No fixed format required from the accounting side. James accepted the standard format: INV-YYYY-NNNNNN (year-prefixed, zero-padded 6-digit global sequence, no annual reset). Starting from scratch — no prior sequence to continue. First invoice will be INV-2026-000001.

---

### Payment Methods and Gateway

Two payment methods in scope for Phase 1:
1. Card payment via Stripe — Payers receive a "Pay Now" link in the invoice email that opens a Stripe-hosted checkout. Harborview has an existing Stripe account.
2. Bank transfer (manual reconciliation) — Finance Manager marks the invoice as Paid manually when the bank transfer clears. James: "most of my clients still pay by bank transfer; I just need to be able to tick it off in the system."

No multi-currency. All invoices and payments in GBP only.

---

### Credit Notes

James does not currently issue formal credit notes — if there's an error, he reissues the invoice. Confirmed: void-and-reissue is acceptable for Phase 1. Credit note issuance is explicitly out of scope.

---

### Multi-Currency

Single currency: GBP. All clients are UK-based. Confirmed out of scope.

---

### Journey — Standard Invoice to Payment

Current process: James creates the invoice in Excel, exports to PDF, and emails the PDF manually. He then watches his bank statement or waits for the client to tell him they've paid. Pain point: "I have no idea which invoices are outstanding at any given moment without going back through my emails."

New system expectation: select a client from a list, add line items, set a due date, save as draft. Review the draft and click Send. Client receives an email with the PDF attached and a "Pay Now" button. When the client pays (via Stripe), the status updates automatically. For bank transfers, James marks it as Paid manually.

James confirmed: the invoice email should come from invoices@harborviewconsulting.co.uk. He sends roughly 40–60 invoices per month.

---

### Journey — High-Value Invoice Approval

Current process: James forwards the Excel invoice to a Director by email and waits for a reply. "Sometimes I have to chase them for the reply — there's no visibility at all."

New system expectation: when James tries to send an invoice at or above £10,000, the system should stop him and tell him it needs Director approval. All three Directors should get an email. Director clicks into the system, reviews the invoice, and approves or rejects. If approved, James gets notified and sends the invoice. If rejected, James needs to see the reason clearly.

Sarah Chen (Director, present in Session 2) added: Directors want to see the full invoice details on the approval screen, not just the total. She wants to be able to approve from her iPhone without logging into a desktop.

---

### Journey — Overdue Invoice to Resolution

Current process: "I send reminders when I remember to. Sometimes I forget for a couple of weeks." James estimates he loses time to reminders about once a month when he finds an invoice that has been overdue for longer than he realised.

New system expectation: the system sends the first reminder automatically the day after the due date. Then again at Day 7 and Day 14. After Day 14, James wants an alert in the system telling him to follow up personally. He does not want the system to do anything further after that — no legal letters, no escalation to a collections process.

James identified one long-term client he prefers to call directly when overdue, rather than sending an automated email. Confirmed: no per-client configuration needed in Phase 1. After the Day 14 escalation alert, James calls them manually.

---

## 9. Resume Instructions

**Next session starts at:** N/A — Generation ready after Session 2.

**Status:** All topics from Section 4 (4.1–4.8) and all journeys from Section 6 (6.1–6.3) are covered. Integration Confirmation and Output Confirmation are complete. Two open questions remain in Section 7 — both have owners and due dates. These will be carried into BRD Section 12 (Open Questions) and must be resolved before the BRD is approved.

**Before generating the BRD:**
- [ ] Load: `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md`
- [ ] Load: `projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md`
- [ ] Load: this file (already done if you are reading this)
- [ ] Load: `projects/TEMPLATE/brds/brd.template.md`
- [ ] Apply BRD Drafting Instructions from BRD playbook Section 10

**Topics still pending:**
None.

**Things the client committed to follow up on:**
- OQ-001: Director email addresses — James to email by 2026-06-10
- OQ-002: Resend DNS verification records needed from Ascendra; logo file needed from James

**Scope risk items still open:**
None.

---

## 10. Pre-Generation Verification

- [x] Section 1 (Project & Session Context): Project Name, Project Code, Client, Domain, and BRD playbook path are all filled
- [x] Section 3 (Playbook Progress): Every topic marked Covered has a substantive one-line client answer — all 8 discovery topics and 3 journeys have substantive entries
- [x] Section 4 (Scope Risk Flags): No flags triggered. Context documented confirming each SRF was implicitly checked and did not apply.
- [x] Section 5 (Integration Confirmation): All 5 integrations are marked — 3 Confirmed In, 2 Confirmed Out. No Pending integrations.
- [x] Section 6 (Output Confirmation): Completed in Session 2. Every standard output confirmed Yes or No with notes.
- [x] Section 7 (Open Questions): OQ-001 and OQ-002 are listed with owners and due dates. Both will map to BRD Section 12 — they must not be dropped.
- [x] Section 8 (Captured Client Answers): Every covered topic has a substantive block. 8 discovery topics and 3 journeys all have substantive entries. No blocks are empty or placeholder.
