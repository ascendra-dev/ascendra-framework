## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version from discovery session with Thornfield Property Management Ltd |

---

## 1. Project Overview

**Project Name:** Thornfield Maintenance Ticketing System
**Client:** Thornfield Property Management Ltd

### 1.4 Scope

**In Scope**
- Reactive, tenant-reported maintenance ticket lifecycle: report, triage, assignment, contractor work, resolution, closure
- Severity classification (Emergency / Urgent / Routine / Planned)
- Tenant and contractor notifications (email/SMS)
- A full, timestamped audit timeline per ticket, viewable by the Operations Lead
- Read-only nightly sync of tenancy/tenant/landlord data from Thornfield's existing CRM

**Out of Scope**
- Proactive/planned maintenance (scheduled servicing, inspections)
- Contractor invoicing and payment
- Multi-tenant support for letting agents other than Thornfield
- Real-time (as opposed to nightly) CRM integration

### 1.5 Assumptions & Constraints

- Assumption: Thornfield's existing CRM will remain the system of record for tenancy/tenant/landlord data; this project never writes back to it.
- Constraint: Budget capped per separate agreement with Priya Shah. Soft target to have the urgent-repair flow live before the winter heating season.
- Constraint (regulatory): Homes (Fitness for Human Habitation) Act 2018; Housing Ombudsman Complaint Handling Code.

### 1.6 Designed for Extension

None identified. This is a standalone, single-organization deployment for Thornfield; no extension dimension was named or anticipated during discovery.

---

## 2. Stakeholders & Personas

| Persona | Description |
|---------|-------------|
| Tenant | Reports issues and tracks their own tickets via a link sent at report time — no account creation. |
| Property Manager | Triages incoming tickets, assigns Contractors, sees every ticket for properties they manage. |
| Operations Lead | Sees every ticket across the whole portfolio; pulls compliance/audit timelines. |
| Contractor | Accesses only tickets assigned to them, via a per-job link — no account creation. |

---

## 3. User Roles

| Role | Access Level |
|------|-------------|
| Tenant | Own tickets only, via link — report new, view status, no severity/assignment visibility |
| Property Manager | Full read/write on tickets for their managed properties |
| Operations Lead | Full read on all tickets across the portfolio; read/write on nothing ticket-specific (no triage/assignment duties themselves, per discovery) |
| Contractor | Read/write on only their own assigned tickets, via link — status update and work-done notes only |

---

## 4. User Journeys

### 4.1 Tenant — Report and Track an Issue

1. Tenant reports an issue via a public form (no login), selecting the affected Tenancy (looked up from the CRM sync by an identifier they're given at move-in, e.g. a property/unit code) and describing the problem.
2. System sends the Tenant a tracking link by their chosen channel (email or SMS).
3. Tenant can revisit the link at any time to see the ticket's current status.
4. Tenant receives a notification whenever the ticket's status changes.

### 4.2 Property Manager — Triage and Assign

1. Property Manager reviews new Reported tickets and sets a Severity.
2. Property Manager assigns the ticket to a Contractor — this can happen at the same time as triage, or later, at the Property Manager's discretion.
3. System sends the assigned Contractor a per-job link by SMS.
4. Contractor accepts the job via the link, updates status to In Progress when starting work, and records what was done before marking it Resolved.
5. System notifies the Tenant at each status change.
6. Property Manager reviews and closes Resolved tickets, or reopens one if the same issue recurs within 14 days.

### 4.3 Operations Lead — Compliance Timeline

1. Operations Lead opens any ticket, including ones outside their own day-to-day involvement.
2. System shows a complete, timestamped timeline: report time, triage time and severity assigned, assignment time, every status change, and resolution/closure time.
3. Operations Lead exports or references this timeline in response to a tenant or Ombudsman query.

---

## 5. Functional Requirements

### 5.1 Maintenance Ticket Lifecycle

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-001 | The system must let a Tenant report a maintenance issue without creating an account, identifying their Tenancy from a code provided at move-in, and must send them a tracking link by their chosen channel (email or SMS) immediately on submission. | Must Have | Client-Stated | Core |
| REQ-002 | The system must require a Severity (Emergency, Urgent, Routine, or Planned) to be set before a ticket can be assigned to a Contractor — a Property Manager cannot skip this step. | Must Have | Client-Stated | Core |
| REQ-003 | The system must allow a Property Manager to assign a triaged ticket to a Contractor independently of when it was triaged — assignment is a separate action, not a mandatory same-step follow-on to triage. | Must Have | Client-Stated | Core |
| REQ-004 | The system must require a Contractor to record what work was done before a ticket can move to Resolved. | Must Have | Client-Stated | Core |
| REQ-005 | The system must allow a Resolved ticket to be reopened (returned to In Progress) if the same issue is reported again within 14 days of being marked Resolved; a report of the same issue after a ticket has been Closed must instead create a new ticket that references the original. | Must Have | Client-Stated | Core |
| REQ-006 | The system must support a rare multi-trade case (a single reported issue requiring more than one trade) by allowing a ticket to link to one or more secondary tickets, rather than assigning more than one Contractor to a single ticket. | Should Have | Client-Stated | Core |
| REQ-012 | The system must let a Contractor view and update only tickets assigned to them, via a per-job link, with no account creation required. | Must Have | Client-Stated | Core |

### 5.2 Notifications

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-007 | The system must notify a Tenant, by their chosen channel (email or SMS, selected when they first report an issue), whenever their ticket's status changes. | Must Have | Client-Stated | Core |
| REQ-008 | The system must notify an assigned Contractor by SMS only — never email — when they are assigned a ticket, since contractors do not reliably check email while on site. | Must Have | Client-Stated | Core |

### 5.3 Compliance & Audit Timeline

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-009 | The system must record a timestamped entry for every state change a ticket goes through — reported, triaged (with severity set), assigned, accepted, in progress, resolved, closed — and must make this full history visible to the Operations Lead for any ticket, regardless of which Property Manager handled it. | Must Have | Client-Stated | Core |
| REQ-010 | The system must let the Operations Lead retrieve a single ticket's complete timeline in under a minute without needing to ask a Property Manager for context, per the client's stated success criterion. | Must Have | Client-Stated | Core |
| REQ-011 | The system must classify Severity against a response-time target that is configurable per Severity level, not hard-coded — Thornfield's compliance advisor has not yet confirmed the specific numeric windows (domain knowledge document Section 9/12.3 marks this Unverified), and the values must be changeable by an Operations Lead once confirmed, without a code change. | Must Have | Client-Stated | Core |

### 5.4 CRM Integration

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
| REQ-013 | The system must look up Tenancy, Tenant, and Landlord identifying details from Thornfield's existing CRM via a nightly read-only sync — this system never writes back to the CRM. | Must Have | Client-Stated | Core |

### 5.21 Future Capabilities

> Domain-grounded, discovery-backed ideas explicitly deferred this phase — see `practitioner-guide/03-brd-and-scope-lock.md`'s test: each row below has a real factual sentence behind it, backed by the domain knowledge document and this project's own discovery session, not a guess.

| ID | Title | Description | Priority | Source | Layer |
|----|-------|-------------|----------|--------|-------|
| REQ-014 | Proactive/Planned Maintenance | The system could extend to scheduled, non-reactive maintenance (e.g. annual boiler servicing) — the domain knowledge document (Section 11) names this as a typical later-phase capability in this domain, and Thornfield's own discovery session (brief Section 8 and BRD discovery Section 3) confirmed it is real but explicitly out of scope for this phase. | Should Have | Client-Stated | Core |
| REQ-015 | Real-Time CRM Integration | The system could move from nightly to real-time CRM sync if Thornfield's needs change — Priya explicitly named this as a deliberate, considered deferral, not an unconsidered gap, during BRD discovery (Section 5, Integration Confirmation). | Nice to Have | Client-Stated | Core |

---

## 6. Business Rules

1. A ticket cannot move from Triaged to Assigned without a Severity set (REQ-002).
2. A ticket cannot move to Resolved without a work-done record from the assigned Contractor (REQ-004).
3. A ticket reported again within 14 days of a Resolved status reopens the same ticket; after Closed, it creates a new, linked ticket (REQ-005).

---

## 7. Data Requirements

- Tenancy, Tenant, and Landlord identifying data: sourced from the existing CRM via nightly sync (REQ-013), never originated in this system.
- Maintenance Ticket, its full status-change history, and any linked secondary tickets: originated and owned by this system.

---

## 8. Integration Requirements

| Integration | Direction | Notes |
|-------------|-----------|-------|
| Existing CRM | Read-only, nightly sync | REQ-013. No write-back. Real-time sync explicitly deferred (REQ-015). |
| SMS/Email provider | Outbound only | REQ-007, REQ-008. |

---

## 9. Security Requirements

- Tenant and Contractor access is link-based (no password), so each link must be unguessable and scoped to exactly one ticket (Tenant) or one assigned job (Contractor) — never a general-purpose account.
- Property Manager and Operations Lead access requires authentication; Operations Lead's read access spans the whole portfolio, Property Manager's is scoped to their managed properties.

---

## 10. Non-Functional Requirements

- Notification delivery: attempted within 5 minutes of the triggering status change (supports REQ-007/REQ-008 and the client's own 15-minute success criterion with margin).

---

## 11. Dependencies

- SMS/email provider selection — resolved at architecture stage, not a BRD-level decision.
- Thornfield's compliance advisor's confirmation of REQ-011's specific numeric values — does not block architecture or story generation, since REQ-011 is written to require configurability, not a specific number.

---

## 12. Open Questions

| Question | Owner | Status |
|----------|-------|--------|
| Confirmed numeric response-time targets per Severity level | Priya Shah, via compliance advisor | Open — tracked against REQ-011, does not block BRD approval since REQ-011 already specifies configurability as the requirement |

---

## 13. Pre-Approval Verification

- [x] Every requirement has ID, Description, Priority, Source, Layer
- [x] Every Description is one capability, not two glued together, and not missing the detail that makes it usable (REQ-writing density test applied per `practitioner-guide/03-brd-and-scope-lock.md`)
- [x] Every Out of Scope item that is genuinely deferred (not permanently excluded) has a corresponding Section 5.21 entry with a real REQ-ID
- [x] No Universal-Business-Rule-shaped claim about the specific regulatory response-time numbers — correctly deferred to configuration (REQ-011), matching the domain knowledge document's own Unverified flag
- [x] Every persona in Section 2 appears in at least one journey (Section 4) and at least one requirement
- [x] Every integration in Section 8 has a corresponding security consideration in Section 9 where relevant

---

## 14. Approval

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | (Simulation) | Approved | 2026-09-16 |
| Client Representative | Priya Shah | Approved | 2026-09-16 |

---

## 15. Parking Lot

*None.*

---

## 16. Density & Judgment Findings

*Generated by `/judgment-check` against `practitioner-guide/03-brd-and-scope-lock.md` on 2026-09-16. Findings are questions raised by the guide's own tests, not verdicts — see Section 17 for how each was actually handled.*

| # | Test (chapter source) | Location | Finding |
|---|---|---|---|
| 1 | REQ-writing density — "one REQ = one independently testable capability" | REQ-005 (reopen/recurrence) | REQ-005 covers two paths (reopen within 14 days; new linked ticket after Closed) in one requirement — is this one capability with two branches, or two capabilities glued together? |
| 2 | BRD→Epic traceability — "choose feature-area boundaries the way you'd choose epic boundaries" | Section 5.2 (Notifications) | Notifications read as a cross-cutting concern threaded through 5.1 and 5.4 rather than an independently demoable capability on their own — is a standalone feature area the right boundary, or should this fold into the epics it actually serves? |
| 3 | BRD→Epic traceability, same test | Former Section 5.4 (Contractor Access, one REQ) | A feature area with exactly one requirement, whose content (a Contractor's own lifecycle interaction) already overlaps REQ-004 in 5.1 — is this really an independently-shippable capability, or does it belong inside the Lifecycle feature area? |

---

## 17. Density & Judgment Resolution

| Date | Finding | PO Response | Detail |
|------|---------|-------------|--------|
| 2026-09-16 | REQ-005 density | Confirmed as-is | The 14-day reopen boundary and the post-Closed recurrence path are two sides of one business rule, not two shippable features on their own — neither half is meaningful without the other defining the boundary between them. Matches REQ-002's precedent for "one rule, multiple necessary facets." |
| 2026-09-16 | Section 5.2 Notifications boundary | Acknowledged tradeoff | Keeping Notifications as its own feature area, even though it is cross-cutting in implementation — the client (Priya) treats a notification failure as a distinct, independently trackable failure mode from a workflow failure, and Success Criterion 2 in the brief is notification-specific. Flagged for attention when `/gen-epics` runs: this may or may not become its own epic depending on how the cohesion test in `practitioner-guide/04-product-structuring.md` applies once REQ-007/REQ-008 are seen alongside the rest of the epic breakdown. |
| 2026-09-16 | Former Section 5.4 Contractor Access | Revised | Folded REQ-012 into Section 5.1 (Maintenance Ticket Lifecycle) and removed the standalone 5.4 heading — a Contractor's own access and update behavior is part of the same lifecycle capability as REQ-004, not a separate one. Section numbering closed up (former 5.5 CRM Integration is now 5.4). |
