# BRD Discovery Playbook — Invoice Payment (Core)

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-05-20 | Ascendra AI | Initial generated version from Invoice Payment domain knowledge v1.1 |

---

## 1. Playbook Scope

### 1.1 What This Playbook Covers

This playbook covers a BRD requirement discovery session for organisations that need to build or replace a B2B invoice payment system — one that creates, approves, sends, and tracks invoices, and collects payment from client companies. It runs an initial scoping session that surfaces the client's specific requirements within the Invoice Payment domain. It is designed for clients who are not currently using a purpose-built invoicing platform.

### 1.2 Domain Documents to Load

| Document | Purpose |
|----------|---------|
| `projects/{PROJECT_CODE}/domain/domain.md` | Invoice Payment (Core) domain knowledge — entities, lifecycle, UBRs, validations, personas, integrations, known variations, and scope boundaries. This is the primary reference throughout the session. |

### 1.3 What This Playbook Does Not Cover

- Domain education sessions — this playbook assumes the Invoice Payment domain knowledge document is already loaded and complete
- Country-specific e-invoicing compliance (FBR, FIRS, etc.) — if a client triggers SRF-001, load the relevant country extension document before continuing
- Subscription billing with auto-charge — a distinct lifecycle; use a different playbook
- Purchase order (PO) management — inbound supplier invoice handling; adjacent domain, different playbook
- Consumer (B2C) payment flows — different regulatory and UX context; out of scope for this playbook

---

## 2. Pre-Session Checklist

- [ ] Domain knowledge document(s) listed in section 1.2 are loaded and confirmed.
- [ ] The client's name, organisation name, and domain (industry/sector) are known.
- [ ] The session mode is confirmed: chat discovery (AI leads questions) vs document review (client provides brief upfront).
- [ ] **Check for a discovery state file at `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`.**
  - If it **exists**: load it now. Read the **Resume Instructions** section at the bottom. Follow those instructions instead of continuing with the standard session opening. Do not re-ask questions already covered.
  - If it **does not exist**: this is a fresh session — continue with the checklist below and proceed to section 3.
- [ ] If this is not the first session with this client: review any prior BRD draft and open questions before proceeding.
- [ ] No scope commitments have been made before this session. Discovery precedes scoping.

---

## 3. Session Opening

**Frame the session:**
Briefly explain what you will do: ask about their current invoicing process, what they need the system to handle, and what problems they are trying to solve. Tell them there are no wrong answers — the goal is to understand their situation, not to sell them a specific solution.

**Confirm the basics before asking any domain questions:**
- [ ] Organisation name and type (consulting firm, manufacturer, agency, etc.)
- [ ] Primary business activity — what do they invoice clients for?
- [ ] Approximate size: number of staff, number of active clients they invoice, average invoices per month
- [ ] Who else is involved in this project beyond today's session — who approves large invoices, who handles finance queries, is there an IT team?
- [ ] What prompted them to look at this system now? Pain point with the current approach, growth, compliance issue, a previous system that failed?

> **[AI Guide]** Record the answers to these questions. They inform how you frame subsequent questions and which scope risk flags are most likely to trigger. A large number of invoices per month raises performance and bulk operation questions. Mention of a government tender or public sector client may trigger SRF-001. A very small team may mean Finance Manager and Approver are the same person — flag this in section 4.2.

---

## 4. Core Discovery Questions

> **[AI Guide — Section Instructions]**
> This section contains the questions you must ask. They are organised by topic. Go through every topic — do not skip topics because you assume the answer. Ask the primary question first. Ask follow-up questions only if the primary answer is ambiguous or triggers a follow-up marker. When a client answer triggers a **Scope Risk Flag**, jump to section 5 immediately. Record every answer. Section 10 tells you how to use them in the BRD.

---

### 4.1 VAT and Tax Model

**Primary question:**
> "Are you VAT-registered? Do you charge VAT on your invoices, and if so, what is the rate?"

**Follow-ups if needed:**
- "Do all your services attract the same VAT rate, or do different types of work have different rates?"
- "Do any of your clients have a VAT exemption that means you don't charge them VAT?"
- "Are you invoicing clients outside your home country? If so, do cross-border invoices attract a different tax treatment?"

**Scope risk trigger:** If the client mentions multiple VAT rates, tax-exempt clients, or cross-border VAT rules → SRF-002

**BRD note:** Record the VAT rate as a configurable business rule. If it is a single fixed rate (the standard case), add as BR-xxx [Client-Stated]. If no VAT applies, record explicitly as an [Assumed] exclusion unless the client confirms. Multi-rate VAT triggers SRF-002 before scoping continues.

---

### 4.2 Approvals

**Primary question:**
> "Do any of your invoices need to be reviewed and approved by a manager or director before you send them to your client? If yes, when does that apply?"

**Follow-ups if needed:**
- "Is there a monetary threshold — for example, invoices above a certain value need approval?"
- "Who has the authority to approve? Is it a specific person, or any one of a group of people?"
- "Does any one person have authority, or do you need two approvals for very large invoices?"
- "What happens when you approve an invoice — can you just click a button, or do you need to add a comment or reason?"
- "What if the approver rejects it — what does the Finance Manager do with it?"
- "Does the Finance Manager and the approver ever the same person in your organisation?"

**Scope risk trigger:** If the client requires multi-level approvals (e.g. Director + CFO for invoices above £100,000) → SRF-003

**BRD note:** If approval threshold exists: record as [Client-Stated] BR-xxx. Record the threshold amount, who qualifies as an approver, and whether any one approver suffices or multiple are required. If the Finance Manager is also the only approver, flag this as an open question — the system must be designed to handle or explicitly prohibit self-approval.

---

### 4.3 Overdue Reminders and Dunning

**Primary question:**
> "When a client doesn't pay by the due date, what happens? Does someone manually send a follow-up, or would you like the system to do that automatically?"

**Follow-ups if needed:**
- "If automatic — how many reminders do you want to send, and how often? For example, a reminder on Day 1 after due date, then again on Day 7, then Day 14?"
- "After a certain number of reminders, do you want the system to flag the invoice for you to handle personally?"
- "Do you have clients who are frequently late? Would you want to handle reminders differently for different clients, or is one universal schedule fine?"
- "Do you want to charge late payment interest? Is that something you currently do?"

**Scope risk trigger:** If the client wants per-client reminder schedules, interest calculation, or a collections workflow → SRF-004

**BRD note:** Record the reminder schedule as configurable system settings [Client-Stated]. Record whether the Finance Manager wants an escalation notification when the dunning sequence completes. Late payment interest calculation is not in scope for Phase 1 unless the client confirms it is essential.

---

### 4.4 Payer Management and Deduplication

**Primary question:**
> "How do you manage your client list? Do you have a master list of clients you invoice, or do you create them one at a time as needed?"

**Follow-ups if needed:**
- "Do you have an existing client list in a spreadsheet or another system that we'd need to import?"
- "How do you detect if someone is adding a client that already exists? What do you use to identify a unique client — their company name, email, phone number?"
- "Have you ever accidentally invoiced the same client under two different names or entries? How did you handle that?"
- "When you migrate from your current system, do any of your existing clients have outstanding unpaid amounts you'd want to carry over?"

**Scope risk trigger:** No direct scope risk — but if the client has a very large client base (1,000+) requiring bulk import and sophisticated deduplication, note as a complexity flag.

**BRD note:** Record the deduplication key preference as [Client-Stated]. If the client needs to import an existing list, record as a functional requirement with a CSV import story. If clients have outstanding balances from the previous system, record the Opening Balance requirement [Client-Stated].

---

### 4.5 Invoice Reference Format

**Primary question:**
> "Do your invoices need to use a specific reference number format? For example, do they need to include the year, a sequence number, or a specific prefix?"

**Follow-ups if needed:**
- "Does your accountant or accounting system expect invoices in a particular numbering format?"
- "Do the reference numbers need to continue from where your current system left off?"
- "Should the numbering restart every year, or should it be a continuous sequence?"

**Scope risk trigger:** None — reference format is always client-configurable. However, if the client needs to continue from a specific number in a prior system, record the starting sequence number as a data migration item.

**BRD note:** Record the reference format as [Client-Stated] if specified, or as [Domain-Default] (INV-YYYY-NNNNNN, no annual reset) if the client accepts the standard. If migrating from a prior system, the starting sequence number is a dependency.

---

### 4.6 Payment Methods and Gateway

**Primary question:**
> "How do your clients currently pay you? Bank transfer, card payment, cheque, something else?"

**Follow-ups if needed:**
- "Would you like clients to be able to pay by card directly from the invoice email? Do you have a Stripe account, or another card payment provider?"
- "Do you want the system to track bank transfers too, even if they aren't processed online?"
- "For online card payments — do you already have a payment gateway account, or would you need to set one up?"
- "Do you have any clients who pay in a different currency from your standard billing currency?"

**Scope risk trigger:** If the client mentions multiple currencies or a non-standard payment gateway that may not have a well-documented API → SRF-005 (see multi-currency SRF) or flag as an open question.

**BRD note:** Record the payment method(s) as [Client-Stated]. For card payment via gateway: confirm which gateway, whether the account already exists, and who holds the credentials. Bank transfer tracking requires a manual "Mark as Paid" workflow — note this as a separate story if the client needs it. Multi-currency → SRF-005.

---

### 4.7 Credit Notes

**Primary question:**
> "If you've already sent an invoice and then realise you've overcharged — or the client returns something — how do you handle that today? Do you issue a credit note?"

**Follow-ups if needed:**
- "Would it be enough to cancel the original invoice and create a new corrected one, or do you need a separate credit note document?"
- "Do your clients or your accounting system require a formal credit note document with its own reference number?"

**Scope risk trigger:** If the client requires formal credit note issuance in Phase 1 → SRF-006

**BRD note:** If void-and-reissue is acceptable, record explicitly as [Client-Stated] exclusion: "Credit note issuance is out of scope for Phase 1. The Finance Manager voids the incorrect invoice and creates a corrected one." If the client needs credit notes, record as a Phase 1 requirement and flag SRF-006 — credit note handling adds significant lifecycle complexity.

---

### 4.8 Multi-Currency

**Primary question:**
> "Do you bill all your clients in the same currency, or do you have clients in different countries that you invoice in their local currency?"

**Follow-ups if needed:**
- "If you invoice in multiple currencies, do you need the system to track exchange rates, or is each invoice simply recorded in whatever currency it was raised in?"
- "Does your bank account receive payments in multiple currencies, or do you convert everything to your home currency?"

**Scope risk trigger:** Any multi-currency requirement → SRF-005

**BRD note:** If single currency confirmed: record as [Client-Stated] BR-xxx: "All invoices are raised in [currency]. Multi-currency is out of scope." If multi-currency is required → SRF-005 before continuing.

---

## 5. Scope Risk Flags

> **[AI Guide — Section Instructions]**
> A scope risk flag is triggered when a client answer indicates complexity, dependency, or regulatory overhead that falls outside standard scope.
>
> When a flag triggers:
> 1. Do not continue to the next discovery question.
> 2. Explain the implication to the client clearly and in plain language.
> 3. Record the trigger and the client's response.
> 4. Confirm whether to continue scoping this item in the current session or flag it as a separate conversation.
> 5. Note it as an open question in the BRD if unresolved.
>
> Do not guess the answer for the client. Do not proceed as if the flag didn't trigger.

---

### SRF-001: Government E-Invoicing Mandate

**Triggered when:** The client mentions they invoice a government body, public sector client, or operates in a country where e-invoicing to tax authorities is legally required (e.g. Pakistan — FBR IRIS, Nigeria — FIRS, Egypt — ETA).

**Implication:** Government e-invoicing mandates require the system to submit invoice data to a tax authority API in a specific format at the time of issuing each invoice. This is a separate integration with a regulatory body, not just a payment gateway. It affects the invoice-sending lifecycle, adds an external dependency, and may require country-specific invoice fields. This is materially more complex than standard invoicing.

**Response to client:**
> "You've mentioned [government body / country]. In [country], there's a legal requirement to submit invoices electronically to [tax authority] at the time of issuing. That's a mandatory integration that sits outside standard invoicing scope. Before we continue, I want to flag this explicitly — it affects scope and timeline. Can we confirm whether this applies to all your invoices, or only to invoices issued to government clients specifically?"

**Action:** Record the trigger and the client's answer. If confirmed, note as a dependency in the BRD and flag for a separate country extension scoping session. Load the relevant country extension document before continuing if available. Do not proceed as if this is standard scope.

---

### SRF-002: Multiple VAT Rates

**Triggered when:** The client mentions that different services or clients attract different VAT rates, or that some clients are VAT-exempt.

**Implication:** Multi-rate VAT means the system must apply different tax percentages to different line items (or different invoices), track which rate applies to each, and potentially report on VAT collected by rate. This adds complexity to invoice creation UI, PDF generation, and accounting exports.

**Response to client:**
> "You've mentioned different VAT rates depending on the service or client. That's completely normal, but it does mean the invoicing system needs to handle multiple rates — rather than applying a single percentage to every invoice. I want to make sure we scope that correctly. Can you tell me roughly how many different rates apply, and whether you know the specific rates?"

**Action:** Record the rates and the conditions under which each applies. Note as a complexity flag in the BRD. Confirm whether this is in scope for Phase 1 or deferred.

---

### SRF-003: Multi-Level Approval Chains

**Triggered when:** The client describes approval thresholds that require more than one person to approve a single invoice (e.g. "invoices over £10,000 need the Director, but invoices over £100,000 also need the CFO").

**Implication:** Multi-level approval requires a sequential approval workflow — the invoice moves through multiple approval stages before it can be sent. This is significantly more complex than a single-approver model, requiring an approval chain data model, multiple notification events, and tracking of which approval stage the invoice is at.

**Response to client:**
> "So for invoices above £100,000, you'd need both the Director's approval and the CFO's approval before it can be sent — is that right? That's a two-stage approval chain. That's buildable, but it's more complex than a single approver and it affects how we model the approval record and notifications. I want to flag this so we scope it correctly. Is this genuinely a business requirement, or would a single Director approval at a higher threshold work for your purposes?"

**Action:** Record the approval chain levels and thresholds. If confirmed as required, note as a Phase 1 requirement. If the client is uncertain, add to Open Questions with the client as the owner.

---

### SRF-004: Per-Client or Advanced Dunning Configuration

**Triggered when:** The client wants different reminder schedules for different clients, automatic interest calculation on overdue invoices, or a formal collections workflow (letters, legal escalation).

**Implication:** Per-client dunning configuration requires a dunning policy model stored against each Payer, not a single system-wide setting. Interest calculation requires tracking the principal amount, the interest rate, and the compounding schedule — this is a distinct data model. These add significant complexity to the overdue management feature.

**Response to client:**
> "You'd like to send reminders on a different schedule depending on the client — for example, chase slow payers more aggressively. That's a per-client dunning configuration, which is more complex than a single system-wide schedule. I want to make sure we're both clear on the scope difference. Is this something you need from day one, or would a configurable system-wide schedule work for Phase 1, with per-client configuration added later?"

**Action:** Record the client's answer. If per-client is required in Phase 1, note the complexity and add to scope. If it can be deferred, record as a Phase 1 exclusion with a Phase 2 note.

---

### SRF-005: Multi-Currency

**Triggered when:** The client needs to raise invoices in more than one currency.

**Implication:** Multi-currency requires storing the currency on each invoice, converting to the organisation's home currency for dashboard and reporting totals, handling exchange rate sourcing (live rate vs fixed rate vs rate at invoice date), and potentially reconciling payments received in a foreign currency. This adds material complexity to invoice creation, payment confirmation, and reporting.

**Response to client:**
> "You need to raise invoices in multiple currencies — for example, billing UK clients in GBP and US clients in USD. That adds a layer of complexity around exchange rates and reporting totals. Before we continue: do you need the system to automatically convert foreign currency amounts to your home currency for reporting purposes, or is it enough to track each invoice in the currency it was raised in?"

**Action:** Record the currencies needed and the reporting expectation. Note as a complexity flag. If multi-currency is required in Phase 1, add to scope with the exchange rate approach as an open question. If single currency is acceptable for Phase 1, record as [Client-Stated] exclusion.

---

### SRF-006: Credit Note Issuance in Phase 1

**Triggered when:** The client confirms they need to issue formal credit note documents (not just void-and-reissue) in Phase 1.

**Implication:** Credit notes are distinct financial documents that offset an issued invoice — they require their own reference number, PDF, and lifecycle. A credit note does not cancel the original invoice; it creates a new document that reduces the Payer's outstanding balance. This adds a second document type to the system, a second lifecycle, and a Payer balance model.

**Response to client:**
> "You need to issue formal credit notes — not just cancel and reissue the original invoice. Credit notes are a second type of financial document in the system, with their own reference numbers and lifecycle. That's a meaningful scope addition compared to void-and-reissue. Can you tell me how frequently you issue credit notes today, and whether your accounting system requires them specifically?"

**Action:** Record the client's answer and frequency estimate. If confirmed for Phase 1, note as a scope item with the credit note lifecycle as a new feature area. If the client can accept void-and-reissue for Phase 1, record as [Client-Stated] exclusion.

---

## 6. Journey Walkthroughs

> **[AI Guide — Section Instructions]**
> A journey walkthrough is a structured conversation — not a questionnaire. Guide the client through a complete end-to-end workflow using their own language and process.
>
> How to run a walkthrough:
> 1. Set the scene: "Walk me through what happens from the moment [X occurs] to the moment [Y is complete]."
> 2. Let the client narrate. Do not interrupt with questions during the narrative.
> 3. After the narrative, probe gaps: "What happens if...?", "Who is responsible for...?", "What does [person] see at this point?"
> 4. Record every step the client describes — not just the steps in the template.
>
> The template walkthrough below is a guide for what to probe. The client's actual journey may differ.

---

### 6.1 Standard Invoice — Creation to Payment

**Set the scene with:**
> "Walk me through what happens from the moment you decide to send an invoice to a client to the moment you see that invoice marked as paid. Use your current process — how it actually works today, not how you'd like it to work."

**Standard steps to confirm:**
1. Finance Manager identifies a client to invoice
2. Finance Manager creates a new invoice, adds line items and a due date
3. Invoice is saved as a draft with a reference number
4. Finance Manager reviews and sends the invoice
5. Client receives an email with the invoice PDF and a payment link
6. Client pays via the payment link
7. System receives payment confirmation
8. Invoice status changes to Paid — Finance Manager sees this on their dashboard

**Probe questions for this journey:**
- Who creates the invoice — is it always the Finance Manager, or can other people in the team raise invoices?
- Where do the line items come from? Is there a standard service catalogue, or does the Finance Manager type them in each time?
- After you send an invoice, how do you currently know it has been received by the client?
- What does "paid" look like in your current process — does someone tell you, do you check your bank statement, or is there an automated notification?
- What happens if the client has a question about an invoice — do they email you directly, or is there a process?

**Things to note for BRD:**
Record every actor named during the narration (they become user roles or personas). Note any step where the client says "I have to check manually" or "I don't always know" — these are automation opportunity requirements. Note any approval step mentioned in passing — it belongs in Section 4.2 if not already covered.

---

### 6.2 High-Value Invoice — Approval Flow

**Set the scene with:**
> "Tell me about the last time you sent an invoice to a client for a large amount — something that needed sign-off before it went out. Walk me through how that actually worked."

**Standard steps to confirm:**
1. Finance Manager creates an invoice above the approval threshold
2. Finance Manager attempts to send — system flags that approval is required
3. Approver (Director) is notified that an invoice needs their review
4. Director opens the invoice and reviews it
5. Director approves or rejects the invoice
6. Finance Manager is notified of the decision
7. If approved: Finance Manager sends the invoice
8. If rejected: Finance Manager edits and resubmits

**Probe questions for this journey:**
- How are directors currently notified that an invoice needs approval — email, phone call, chat message?
- Can a Director approve an invoice from their phone, or do they need to be at a desk?
- What happens if the Director is on holiday and unavailable — is there a substitute approver?
- How long does the approval typically take? Is there a deadline?
- When a Director rejects an invoice, what information do they give you? Do you always know why?

**Things to note for BRD:**
Record the notification mechanism (email is standard — confirm this). Note whether a mobile-responsive approval interface is needed. Note any substitute approver process — if the client requires it, it is a requirement; if they accept a manual workaround, record that explicitly as an [Assumed] exclusion.

---

### 6.3 Overdue Invoice — Reminder to Resolution

**Set the scene with:**
> "Think of a client who regularly pays late. Walk me through what happens today from the moment an invoice goes past its due date to the moment it's finally paid — or escalated to someone else."

**Standard steps to confirm:**
1. Due date passes without payment
2. System detects the overdue status
3. Automated reminder sent to client on Day 1
4. Further reminders sent per the configured schedule
5. After maximum reminders: Finance Manager is alerted for manual follow-up
6. Client eventually pays — invoice marked as Paid

**Probe questions for this journey:**
- Currently, how do you know when an invoice is overdue? Do you check manually or does something alert you?
- What does the overdue reminder email say today? Do you want the system to use the same language?
- Do you ever call clients instead of emailing them for overdue invoices? Should the system flag which clients need a phone call?
- What is your hardest-to-collect invoice situation — what happened and how did you eventually resolve it?
- Are there clients you never chase automatically — for example, a key account where you'd prefer to handle reminders personally?

**Things to note for BRD:**
Note any manual intervention points — these may become notification requirements. Note any exception clients (key accounts handled differently) — if needed in Phase 1, this is a per-client dunning configuration (SRF-004). Note the client's current reminder language if they want to preserve it — this is an email template configuration item.

---

## 7. Integration Confirmation

> **[AI Guide]**
> Go through the typical integration points defined in the domain document (section 8) one by one. Do not assume any integration is needed — confirm each explicitly.
>
> For each integration: ask whether it applies, who owns it, and whether it already exists or needs to be set up.
> Flag any integration with an onboarding lead time as an external dependency in the BRD.

| Integration | Question to ask | If yes: record in BRD | External dependency? |
|------------|----------------|----------------------|---------------------|
| Payment Gateway (Stripe, Safepay, etc.) | "Do you want clients to be able to pay invoices by card online? Do you have a payment gateway account?" | Integration requirements section. Confirm gateway name, whether account exists, and who holds credentials. | Yes — client must provide API credentials before payment stories can be built. Lead time: request credentials at end of session. |
| Transactional Email (Resend, SendGrid, etc.) | "Which email address should invoice emails, approval notifications, and reminders be sent from? Do you have an email sending service, or should we set one up?" | Integration requirements section. Confirm sender address, domain verification status. | Yes — sending domain must be verified before invoice email can be sent. Lead time: domain verification can take 24–72 hours. |
| PDF Generation | "Invoice emails will include a PDF of the invoice as an attachment — is that what you'd expect?" | Note as a standard requirement — no external integration needed at this scale. | No — PDF generation is in-process. |
| Accounting System (Xero, QuickBooks, etc.) | "Do you use an accounting system like Xero or QuickBooks? Would you need invoices and payments to sync into it automatically?" | If confirmed for Phase 1: integration requirements section with export format and authentication method. If deferred: note as Phase 2. | Yes if in Phase 1 — each accounting system has different API authentication. Flag as a separate scoping conversation. |
| Tax / e-Invoicing API | "Do you invoice any government clients, or are you based in a country where invoices must be submitted to a tax authority?" | If yes → SRF-001. | Yes if applicable — government APIs have strict onboarding processes. See SRF-001. |

---

## 8. Output Confirmation

> **[AI Guide]**
> At the end of the session, confirm every output type below explicitly with the client. Clients assume standard outputs are included. Some are not.
>
> Go through this list out loud. Do not skip it or assume answers from context.

### 8.1 Reports & Exports

| Output | Confirm needed? | Consumer | Format | Frequency |
|--------|----------------|---------|--------|-----------|
| Outstanding invoices list (all invoices not yet paid) | Yes / No | Finance Manager | In-system view | Live / real-time |
| Overdue invoices list (invoices past due date) | Yes / No | Finance Manager | In-system view | Live / real-time |
| Invoices due in the next 7 days | Yes / No | Finance Manager | In-system view / dashboard widget | Live / real-time |
| Payment history (all paid invoices with amounts and dates) | Yes / No | Finance Manager | In-system view | Live / real-time |
| Aged debtor report (outstanding amounts grouped by how overdue they are) | Yes / No | Finance Manager / Director | In-system or exportable | Monthly or on demand |
| CSV / Excel export of invoices or payments | Yes / No | Finance Manager (for accounting) | CSV | On demand |

### 8.2 Notifications

| Notification | Confirm needed? | Channel | Recipient |
|-------------|----------------|---------|-----------|
| Invoice sent confirmation | Yes / No | Email (to Finance Manager) | Finance Manager |
| Invoice approval request | Yes / No | Email | All Approvers / Directors |
| Approval decision notification (approved / rejected) | Yes / No | Email | Finance Manager |
| Overdue reminder sent | Yes / No | Email (to Payer) | Payer |
| Dunning sequence completed — manual follow-up required | Yes / No | In-system alert or email | Finance Manager |
| Payment received | Yes / No | Email or in-system alert | Finance Manager |

### 8.3 Access Levels

- [ ] Confirm who needs full system access (create, send, configure) — typically Finance Manager and Org Admin
- [ ] Confirm which roles are limited to specific actions only — Approver / Director can only approve, not create or configure
- [ ] Confirm whether any roles need read-only / reporting access only — Directors viewing invoice status without approval authority
- [ ] Confirm whether Payer contact details (email, phone) should be visible to Approvers, or name only
- [ ] Confirm whether there is a single Finance Manager or multiple — if multiple, whether they can see each other's drafts

---

## 9. Session Close

**Closing steps:**

1. **Summarise scope:** Restate in 5–8 bullet points what the system will do, based on what the client described:
   > "Based on what you've told me, here is what we're building: [bullet summary]. Does this match your understanding?"

2. **Name what is out of scope:** State explicitly 2–3 things that are not in scope based on what was confirmed or deferred in this session:
   > "I also want to confirm what is NOT in scope, so there are no surprises later: [list]. Is that correct?"

3. **List open questions:** Read out every item flagged as unresolved:
   > "These are the items we weren't able to confirm today — I need an answer on each one before the BRD is finalised: [list]. Who will answer each of these, and by when?"

4. **Confirm next step:**
   > "I'll produce a draft BRD from today's session. Once you've reviewed it, we'll book time to go through it together and confirm before we move to the epic and story phase. You should receive the draft within [timeframe]."

> **[AI Guide]** Record the client's corrections and additions before closing. Do not produce the BRD draft from memory of what you expected them to say — use what they actually confirmed. Any item the client says "I'll check" or "I'm not sure" is an Open Question, not an [Assumed] requirement.

---

## 9.5 Saving State on Pause

> **[AI Guide]**
> Write a discovery state file whenever:
> - The client asks to pause and resume later
> - The session ends naturally but the BRD is not yet complete
> - More than one session is required to complete discovery
>
> The state file is the only thing that survives into the next session. Write it as if the next session starts cold — because it does.
>
> **How to save state:**
> 1. Write the file to `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`. Create the file if it does not exist. Overwrite it if it does — the file always reflects the latest saved state.
> 2. After writing, confirm to the client: "Your session has been saved. When you return, load this project's discovery state file and the AI will resume from where we left off."
>
> **What to write:**
> - Every topic from section 4 that was covered — with a one-line distillation of the client's answer. Not a transcript. The AI on resume must be able to draft BRD requirements from this distillation alone.
> - Every topic not yet covered — clearly marked as pending.
> - Every scope risk flag that triggered — what triggered it, what the client said, current status.
> - Every open question — who owns it, due date if stated.
> - Captured client answers in enough detail for BRD drafting — written as if you were briefing a colleague who was not in the room.
> - Resume instructions: the exact section and topic to start from next session, and anything the client committed to follow up on.
>
> **What not to write:**
> - A transcript or log of the conversation. The state file is a distillation, not a record.
> - Implementation details or your own inferences. Only what the client said.
> - Domain knowledge facts. Those come from the domain doc — not the state file.

---

## 10. BRD Drafting Instructions

### 10.1 Before You Start

- Load `projects/TEMPLATE/brds/brd.template.md`.
- Load the domain knowledge document(s) from section 1.2 of this playbook.
- Have the session notes from sections 4, 6, 7, 8, and 9 ready.

### 10.2 Source Tagging Rules

Every requirement you write must be tagged. Apply the tags as follows:

- `[Client-Stated]` — the client said this explicitly, in their own words, during the session.
- `[Domain-Default]` — this is a universal domain fact the client confirmed applies without deviation. Only include in the BRD if the client explicitly overrode it or the requirement drives a specific implementation decision that must be documented.
- `[Assumed]` — you have inferred this from context. Use sparingly. Every `[Assumed]` tag is a risk. If in doubt, add it to Open Questions instead.

Do not restate facts from the domain knowledge document as BRD requirements. The domain doc is pre-loaded knowledge — the BRD captures only what is project-specific.

### 10.3 Populating Each BRD Section

| BRD Section | Source in session notes |
|-------------|------------------------|
| 1. Project Overview | Session opening answers (organisation, problem, size) + session close summary |
| 2. Stakeholders & Personas | Roles named during journey walkthroughs + discovery questions; Payer as a persona if confirmed |
| 3. User Roles | Permissions confirmed in section 8.3 (Access Levels); map to Finance Manager and Director roles from domain doc |
| 4. User Journeys | Journey walkthrough notes (section 6); each walkthrough becomes one journey in the BRD |
| 5. Functional Requirements | Core discovery question answers (section 4) + journey probe answers; group by feature area (Payer management, Invoice management, Payment collection, Overdue management, Dashboard) |
| 6. Business Rules | Client-specific rules surfaced in discovery (VAT rate, approval threshold, dunning schedule); rules that deviate from or override UBRs in domain doc |
| 7. Data Requirements | Entities confirmed during journey walkthroughs; any client-specific attributes or deviations from domain standard entities |
| 8. Integration Requirements | Integration confirmation table (section 7); only confirmed integrations |
| 9. Security Requirements | Permission model from section 8.3; any compliance flags from scope risk flags that triggered in section 5 |
| 10. Non-Functional Requirements | Only if explicitly stated by the client during discovery — do not invent NFRs |
| 11. Dependencies | External integrations with onboarding lead times; open items from scope risk flags |
| 12. Open Questions | Every unresolved item from section 9 close |

### 10.4 Quality Check Before Handing Over

- [ ] Run the Pre-Approval Verification in BRD section 13.
- [ ] Every open question has an owner and a due date.
- [ ] Every scope risk flag that triggered has been recorded — either as a dependency, open question, or explicit out-of-scope exclusion.
- [ ] No `[Assumed]` tag covers something that should have been confirmed during the session. If found, move to Open Questions.

---

## 11. Playbook Verification

- [x] Every Known Variation in the domain document's Section 10 is covered by at least one question in section 4 of this playbook:
  - VAT / Tax model → 4.1
  - Approver workflow → 4.2
  - Dunning strategy → 4.3
  - Payer deduplication key → 4.4
  - Invoice reference format → 4.5
  - Payment methods → 4.6
  - Credit note handling → 4.7
  - Multi-currency → 4.8
- [x] Every Typical Integration Point from domain Section 8 appears in the Integration Confirmation table (section 7): Payment Gateway, Transactional Email, PDF Generation, Accounting System, Tax / e-Invoicing API.
- [x] Every significant complexity topic from the domain has a Scope Risk Flag in section 5:
  - Government e-invoicing mandate → SRF-001
  - Multiple VAT rates → SRF-002
  - Multi-level approval chains → SRF-003
  - Per-client / advanced dunning → SRF-004
  - Multi-currency → SRF-005
  - Credit note issuance in Phase 1 → SRF-006
- [x] The Journey Walkthroughs in section 6 cover every primary invoice lifecycle flow from domain Section 4: standard invoice creation-to-payment (6.1), high-value approval (6.2), and overdue-to-resolution (6.3).
- [x] The Output Confirmation in section 8 includes all standard reports, notifications, and permission model items from the domain document.
- [ ] Run at least one mock discovery session using `/run-mock-discovery` before treating this playbook as production-ready. Record the outcome in the domain document's section 12.6 (Mock Discovery Validation). **Status: Not yet run.**
