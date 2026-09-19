Harborview Consulting Ltd — Invoice Management System (EPIC-001, Core). This example demonstrates: a correctly scoped epic Goal (user outcome, not technical deliverable); a Section 2 BRD Requirements table that uses exact BRD descriptions and IDs matching the approved BRD; a Section 3.2 Out of Scope that names the covering epic for every exclusion; a Section 8 Notes block carrying the domain rules and BRD constraints that `/gen-stories` needs to write complete acceptance criteria; and a filled Verification checklist.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-06-22 | Ascendra AI | Initial version from /gen-epics |

---

**Epic ID:** EPIC-001
**Title:** Payer Management
**Phase:** Phase 1
**Layer:** Core
**Priority:** Must Have
**Status:** Approved

---

### 1. Goal

When this epic is done, the Finance Manager can register client companies (Payers) individually or in bulk from a CSV file, with the system automatically detecting duplicate registrations. Outstanding amounts from prior systems can be recorded as Opening Balance invoices against any Payer, enabling a clean migration handover at go-live.

---

### 2. BRD Requirements Covered

| REQ ID | Description (exact from BRD) | Priority | Walking Skeleton |
|--------|------------------------------|----------|-------------------|
| REQ-001 | The system must allow the Finance Manager to register a new Payer with name, primary contact email, and phone number. | Must Have | Yes |
| REQ-002 | The system must detect duplicate Payers on registration. The deduplication key is configurable per organisation: phone number, email, or both (either match triggers detection). Default is phone number. | Must Have | — |
| REQ-003 | When a secondary deduplication key matches an existing Payer, the system must show a soft warning and require the Finance Manager to explicitly acknowledge it before proceeding. | Must Have | — |
| REQ-004 | The system must allow the Finance Manager to import Payers from a CSV file. The import must run deduplication checks and produce a per-row import log showing success or failure. Duplicate rows are flagged and not imported — manual resolution required. | Must Have | — |
| REQ-005 | The system must allow the Finance Manager to create an Opening Balance invoice against any Payer to record outstanding debt carried forward from a prior system. | Must Have | — |

---

### 3. Scope

#### 3.1 In Scope

- Manual Payer registration: create a new Payer with name, primary contact email, and phone number
- Payer deduplication on registration: primary key check (hard block) and secondary key check (soft warning requiring acknowledgement)
- Deduplication key configuration: Org Admin sets the key type for the organisation (phone number / email / both)
- CSV Payer import: upload a CSV file, run deduplication checks against the configured key, import valid rows
- CSV import log: per-row success/failure report produced after each import run; duplicate rows identified and excluded
- Opening Balance invoice: create a special invoice type recording prior-system outstanding debt against a registered Payer; this invoice does not enter the standard send / approval workflow

#### 3.2 Out of Scope

- Creating invoices against a Payer and managing the invoice lifecycle (EPIC-003 — Invoice Management)
- Director approval workflow for high-value invoices (EPIC-004 — Director Approval)
- Stripe payment link generation and webhook payment confirmation (EPIC-005 — Payment Collection)
- Overdue reminder automation (EPIC-006 — Overdue Management)
- Finance Manager and Director dashboards (EPIC-007 — Dashboard and Reporting)
- Payer self-service portal and portal authentication — out of scope for Phase 1

---

### 4. Personas Involved

| Persona | What they do in this epic |
|---------|--------------------------|
| Finance Manager | Registers Payers manually, runs CSV imports, creates Opening Balance invoices, acknowledges secondary key warnings |
| Org Admin (admin role) | Configures the deduplication key type for the organisation — phone number, email, or either match |

---

### 5. Dependencies

| EPIC ID | Dependency Reason |
|---------|------------------|
| None | Payer Management is the foundational epic — no other epic needs to be complete before it can begin. All invoice epics depend on this one. |

---

### 6. Definition of Done

- [ ] All stories in this epic are merged to the main branch and passed code review
- [ ] All story acceptance criteria have been verified by QA
- [ ] No open bugs exist against any story in this epic
- [ ] Audit trail entries are confirmed for all user-triggered actions in this epic
- [ ] Product Owner has walked through the capability end to end:
  1. Register a Payer manually with a phone number that already exists on another Payer configured with email as the primary key — confirm the soft warning fires for the phone number match, Finance Manager can acknowledge and proceed
  2. Import a CSV file containing at least one valid row and at least one row with a duplicate primary key — confirm the import log shows the valid row as imported and the duplicate row as flagged and excluded
  3. Create an Opening Balance invoice against a newly registered Payer for an amount above the Director approval threshold (£10,000) — confirm it is saved with the correct amount and does not trigger the Director approval workflow

---

### 7. Stories

| Story ID | Title | Size | Status |
|----------|-------|------|--------|
| | | | |

---

### 8. Notes

The following domain rules and BRD-specific constraints must be applied when `/gen-stories` generates acceptance criteria for this epic:

**Deduplication behaviour (REQ-002, REQ-003):**
- The primary key check is a hard block — if the configured key matches an existing Payer, registration cannot proceed. The Finance Manager must correct the entry.
- The secondary key (whichever key is not configured as primary) also runs on every registration. A secondary key match produces a soft warning — the Finance Manager must explicitly acknowledge it (e.g. click "Proceed anyway") before registration saves. The system does not block on a secondary key match alone.
- If the configured key is "both" (either match triggers detection), any match on either phone or email is a hard block.

**CSV import behaviour (REQ-004):**
- Import deduplication runs against the organisation's configured primary key. Rows that match the primary key are flagged as duplicates and not imported. The Finance Manager must resolve these manually (outside the system).
- The import log must be produced even if all rows are rejected — a completely failed import is a valid outcome.
- VAL-004 and VAL-005 (email and phone format validations) apply to each CSV row during import. A row with an invalid email or phone format is also flagged as an error, distinct from a duplicate.

**Opening Balance invoices (REQ-005):**
- Opening Balance invoices are record-only entries. They must not enter the standard invoice sending workflow (no "Send" button).
- Opening Balance invoices are exempt from VAL-003 (unit price > 0 per line item) — the historical debt amount is a pre-calculated total, not an itemised charge.
- Even if the Opening Balance total exceeds the Director approval threshold (£10,000), the Director approval workflow must not trigger — Opening Balance invoices record historical debt and are never sent to the Payer.

**Security (SEC-001, SEC-002):**
- Payer data is scoped to the organisation. A Finance Manager from one organisation must not be able to view, edit, or import Payers belonging to another organisation. This is enforced by filtering all Payer queries by `orgId` from the JWT.

---

## 9. Density & Judgment Log

*Generated by `/judgment-check` against `practitioner-guide/04-product-structuring.md` on 2026-07-12. Findings are questions raised by the guide's own tests, not verdicts. This worked example demonstrates the section's shape — a real project's findings will differ.*

| # | Test (chapter source) | Location | Finding | Outcome | Detail |
|---|---|---|---|---|---|
| 1 | Epic cohesion test — "one scenario, one Definition-of-Done walkthrough" | Section 1 (Goal) and Section 6 (Definition of Done) | Confirm Payer Management still reads as one independently demoable capability, not two (creation vs. deduplication) glued together | Confirmed as-is | Creation and dedup detection are the same capability — dedup is a rule *of* creation (REQ-002 is a sub-behavior of registering a Payer), not a separately demoable feature. One epic is correct. |
| 2 | Section 8's pre-architecture boundary — "a real, sharp edge, not smoothed over" | Section 8 (Notes) | Section 8 names a specific implementation mechanism ("filtering all Payer queries by `orgId` from the JWT") before architecture exists to decide the actual auth mechanism — is this intentional forward-guidance or does it overstep what Section 8 should assert at this stage? | Acknowledged tradeoff | Kept as-is — this is exactly the class of forward-looking technical hint Section 8 is meant to hold, understanding it may be revised once `/gen-architecture` actually decides the auth mechanism (it might not be JWT-based). Flagged here so a PO reading this epic knows not to treat it as locked, the same caution `practitioner-guide/04-product-structuring.md` gives this exact tension. |

---

## Verification

- [x] Every REQ ID in Section 2 exists in the approved BRD v1.1 and has not been modified since this epic was authored — REQ-001 through REQ-005 confirmed against BRD Section 5.1
- [x] The In Scope items in Section 3.1 collectively cover all requirements listed in Section 2 — REQ-001 → manual registration; REQ-002 → dedup check; REQ-003 → secondary key soft warning; REQ-004 → CSV import + log; REQ-005 → Opening Balance invoice
- [x] Every item in Section 3.2 (Out of Scope) names the EPIC or Phase that covers it — no exclusion without a named home
- [x] All personas in Section 4 appear in BRD Section 2 (Stakeholders & Personas) — Finance Manager (James Okafor) and Org Admin (admin role) are both in the BRD
- [x] All EPIC IDs in Section 5 (Dependencies) — Section 5 states None; no EPIC IDs to verify
- [x] The Layer field (Core) is consistent with all requirements in Section 2 — REQ-001 through REQ-005 are all Core requirements
- [x] No implementation detail appears anywhere in Sections 1–5 — no field names, API routes, database tables, technology choices, or UI layout decisions present
