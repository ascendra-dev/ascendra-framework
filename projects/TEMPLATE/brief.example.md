Harborview Consulting Ltd — Invoice Management System. Active project, intake complete. This example demonstrates: the boundary between strategic goals (Section 3 — outcomes the client wants to be in) and BRD requirements (features the system must do); the correct distinction between Constraints (Section 5 — non-negotiable) and Technology Preferences (Section 6 — inform architecture without constraining it); the use of Risks vs Open Questions as distinct item types in Section 8; and how Section 9 is updated as artifacts are produced across the project lifecycle.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-06-01 | Ascendra AI | Initial version from intake session |
| 1.1 | 2026-06-15 | Ascendra AI | Open question OQ-001 resolved; Section 9 updated with BRD and Architecture |

---

**Project Name:** Invoice Management System
**Project Code:** HARBORVIEW-INV-001
**Client:** Harborview Consulting Ltd
**Domain:** Finance
**Status:** Active
**Created:** 2026-06-01
**Last Updated:** 2026-06-15
**Project Type:** Standalone
**Extends:** —
**Extension Adds:** —

---

### 1. Problem Statement

Harborview Consulting Ltd manages all client invoicing through a shared Excel spreadsheet. As the business has grown to 25 staff across three directors and a finance function, the spreadsheet has become error-prone and impossible to audit in real time — the Finance Manager has no way to see which invoices are overdue without manually checking every row each week. Payment reminders are sent by hand, meaning invoices regularly go unnoticed until the monthly finance close. The Directors must approve high-value invoices by forwarding a spreadsheet row by email, with no formal record that approval was given or when.

---

### 2. Client Context

Harborview is a professional services consultancy based in London, UK. They bill approximately 40–60 invoices per month to a stable client base of around 30 companies. Their finance function is run by one Finance Manager, James Okafor, who handles everything from invoice creation to chasing overdue payments. Three directors — Sarah Chen (who leads the project engagement), plus two others — must approve any invoice above £10,000 before it is sent. That approval process currently involves forwarding a spreadsheet row by email and waiting for a reply, with no tracking of who approved, when, or whether a reminder was sent. Clients pay by bank transfer or card; Harborview has an existing Stripe account but it is not connected to the invoicing process. Harborview has no IT infrastructure and no in-house technical staff — the delivery team manages all hosting and operations.

---

### 3. Strategic Goals

1. Eliminate manual invoice tracking — Harborview wants full AR visibility in real time: Finance Manager and Directors can see what is outstanding, what is overdue, and what has been paid without exporting to a spreadsheet or logging into Stripe.
2. Remove manual effort from payment chasing — the Finance Manager currently spends several hours each month manually sending overdue reminders; that time drops to zero through automated dunning with no manual intervention required.
3. Replace email-based Director approval with a traceable, in-system process — any Director can approve a high-value invoice from any device, and a formal approval record exists for every invoice above the threshold.

---

### 4. Key Stakeholders

| Name | Role | Decision Authority | Involvement |
|------|------|--------------------|-------------|
| Sarah Chen | CFO / Project Owner | Final sign-off on BRD, sprint plans, and UAT | Available for weekly review calls; primary decision-maker |
| James Okafor | Finance Manager | Daily user | Day-to-day contact; will run UAT |
| 3 x Directors (incl. Sarah Chen) | Invoice approvers | End user only | Not involved in delivery — approve high-value invoices only |
| Harborview Clients (Payers) | Invoice recipients | End user only | Not involved in delivery |

---

### 5. Constraints

- **Budget:** Fixed — no overruns permitted. Scope additions require a formal change request and revised estimate.
- **Deadline:** Must go live before 2026-10-01 — Harborview's Q4 financial period starts then and they want to begin the new quarter on the new system. The deadline cannot move.
- **Technology:** Must use Stripe — Harborview has an existing Stripe relationship with a negotiated rate. No alternative payment providers.
- **Regulatory:** UK GDPR applies to storage of client contact and financial data. Invoice records must be retained for 7 years (UK Companies Act 2006, Section 386).
- **Other:** Cloud hosting only — Harborview has no on-premise infrastructure and no capacity to manage servers.

---

### 6. Technology Preferences

- **Existing systems:** Stripe (card payments), Gmail / Google Workspace (internal email and calendar — no integration required; Harborview uses Gmail independently)
- **Preferred stack:** No preference — client defers to delivery team
- **Hosting:** Cloud-managed on delivery team's infrastructure
- **Authentication:** Standalone email + password. Harborview does not use SSO and has no Google Workspace integration requirement.

---

### 7. Success Criteria

1. Finance Manager can create and send an invoice in under 5 minutes from login — verified in UAT.
2. Payment statuses update automatically via Stripe webhook — Finance Manager does not need to check Stripe manually or mark invoices as paid for card payments.
3. Overdue reminders are sent without manual intervention — Finance Manager can confirm from the dashboard that reminders went out without triggering them.
4. Directors can approve high-value invoices from any device — no spreadsheet or email forwarding required; the approval is recorded in the system with the Director's name and timestamp.
5. Finance Manager can view the full AR position and outstanding balance at any time without exporting data or leaving the system.

---

### 8. Risks and Open Questions

| Item | Type | Owner | Notes |
|------|------|-------|-------|
| Stripe API keys not yet provided | Open Question | Sarah Chen | Required before Sprint 1 payment stories can begin. Due: 2026-06-20. Resolved: keys received 2026-06-18 and stored in environment config. |
| DNS configuration for Resend email sending (SPF / DKIM records) | Open Question | Harborview (via Sarah Chen) | DNS propagation can take up to 72 hours. Must be resolved before invoice email stories are tested in QA. Due: 2026-07-01. Status: outstanding. |
| Director email addresses for approval notification configuration | Open Question | James Okafor | All three Director email addresses required before the Director approval story can be configured. Due: 2026-06-25. Status: outstanding. |
| Historical open invoices at go-live | Risk | James Okafor | James will manually enter outstanding invoices as Opening Balance records at go-live. Risk of data entry errors affecting the opening AR balance. Mitigation: provide import template and validation rules; James reviews totals against spreadsheet before go-live. |
| Director availability for UAT | Risk | Sarah Chen | Three directors travel frequently. If UAT window is not agreed upfront, director approval user journeys cannot be tested. Mitigation: agree UAT dates at project kick-off — not Sprint 3. |

---

### 9. Related Artifacts

| Artifact | Location | Status |
|----------|----------|--------|
| Project Brief | `projects/HARBORVIEW-INV-001/brief.md` | Approved |
| Domain Knowledge | `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md` | Approved |
| BRD v1.1 | `projects/HARBORVIEW-INV-001/brds/brd-core-v1.1.md` | Approved |
| Architecture v1 | `projects/HARBORVIEW-INV-001/architecture/arch-v1.md` | Locked |
| Epics Index | `projects/HARBORVIEW-INV-001/epics/index.md` | Approved |
| Sprint 1 Plan | `projects/HARBORVIEW-INV-001/sprints/sprint-01.md` | Active |

---

## Verification

- [x] Problem Statement describes the current-state problem, not the solution — no sentence begins with "We are building..."
- [x] Client Context includes only information relevant to system design — no company history or marketing content
- [x] Every Strategic Goal is an outcome, not a feature — each goal could in principle be achieved without the system (manual AR visibility, manual payment chasing, manual Director approval by phone all existed before the system was proposed)
- [x] Every stakeholder has a named Decision Authority from the defined options
- [x] Every Constraint in Section 5 is genuinely non-negotiable — preferences (stack, hosting) are in Section 6
- [x] Every named regulation in Section 5 is specific: UK GDPR (not "privacy laws") and UK Companies Act 2006, Section 386 (not "7-year retention requirement")
- [x] Technology Preferences in Section 6 are distinguished from hard mandates in Section 5 — Stripe is in Section 5 (mandate); stack and hosting are in Section 6 (preferences)
- [x] Every Success Criterion is observable — each can be verified by a Product Owner in UAT without asking the client for a subjective opinion
- [x] Every item in Section 8 has a named owner — no row with "TBD" in the Owner column
- [x] Related Artifacts in Section 9 reflects the current state — no stale "Draft" statuses on approved documents
- [x] Project Type is Standalone — Extension check not applicable
