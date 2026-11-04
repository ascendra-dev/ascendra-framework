Property Maintenance & Repairs (Core) — universal domain knowledge for reactive, tenant-reported maintenance ticketing systems used by UK residential letting/management agents. This document covers the core reactive-maintenance sub-domain only; proactive/planned maintenance and contractor invoicing are named as typically-deferred, not modeled here. Generated primarily from `domain-discovery-state.md` (session held 2026-09-16) — AI training knowledge supplements only where the session is silent.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version, from discovery session with Thornfield Property Management Ltd |

---

## 1. Domain Overview

### 1.1 What This Document Covers

The lifecycle of a tenant-reported maintenance issue, from initial report through triage, contractor assignment, work, and resolution, for a UK residential letting/management agency managing properties on behalf of third-party landlords. Includes the regulatory context that shapes how urgency must be classified and evidenced.

### 1.2 What This Document Does Not Cover

Proactive/planned maintenance (scheduled servicing, inspections). Contractor invoicing, payment, or financial reconciliation. Property/landlord/tenancy onboarding (assumed to exist in the client's existing systems). Commercial (non-residential) property maintenance.

### 1.3 Related Documents

`property-maintenance-domain-playbook.md`, `domain-discovery-state.md` (both this folder).

---

## 2. Domain Terminology

| Term | Definition | Notes |
|------|-----------|-------|
| Maintenance Ticket | The record of one tenant-reported issue, from report to closure. | Also called "repair," "job," or "request" in common usage — use "Maintenance Ticket" in all artifacts. |
| Property | A physical building or address a landlord owns. | May contain more than one Tenancy (e.g. a house split into flats). |
| Tenancy | One tenant's (or household's) occupied unit within a Property, under a specific agreement. | A Maintenance Ticket is always raised against a Tenancy, not directly against a Property. |
| Severity | The urgency classification assigned to a ticket at triage. | Fixed set of levels; which issue types map to which level is a Known Variation — see Section 10. |
| Contractor | An external tradesperson or firm assigned to carry out the work. | Not a Thornfield employee; accesses only tickets assigned to them. |
| Recurrence | A new ticket referencing a prior Closed ticket for the same underlying issue. | Distinct from "reopen" — see Section 4. |

---

## 3. Core Entities & Relationships

### Property

**What it is:** A physical building or address managed on behalf of a landlord.
**Standard attributes:** Address, landlord reference.
**Relationships:** Has one or more Tenancies.

### Tenancy

**What it is:** One occupied unit within a Property, under one tenancy agreement.
**Standard attributes:** Unit identifier, tenant contact details, tenancy start date, active/inactive status.
**Relationships:** Belongs to one Property. Has one active Tenant at a time. Has zero or more Maintenance Tickets.

### Maintenance Ticket

**What it is:** The record of one tenant-reported maintenance issue.
**Standard attributes:** Description, severity, status, reported date/time, resolved date/time, assigned Contractor.
**Relationships:** Belongs to one Tenancy. Assigned to zero or one Contractor at a time (a genuine multi-trade case is modeled as a linked secondary ticket, not multiple assignees on one ticket). May reference a prior ticket as its Recurrence source.

### Contractor

**What it is:** An external tradesperson or firm.
**Standard attributes:** Name, trade(s), contact details.
**Relationships:** Assigned to zero or more Maintenance Tickets.

---

## 4. Standard Lifecycle & Status Model

### Maintenance Ticket Lifecycle

`Reported → Triaged → Assigned → Accepted → In Progress → Resolved → Closed`

- **Reported → Triaged:** a Property Manager assigns a Severity. A ticket cannot be assigned to a Contractor before this happens (Universal Business Rule, Section 5).
- **Resolved → In Progress (reopen):** valid only if the same issue recurs within 14 days of Resolved.
- **Closed → (new ticket, Recurrence):** if the same issue recurs after a ticket is Closed, this is a new ticket referencing the old one, not a reopen of the old one. The 14-day boundary and the Closed-vs-Resolved distinction are both load-bearing — a UK letting agent's reporting typically needs to distinguish "we didn't fix it right the first time" (reopen) from "it broke again independently" (recurrence).

---

## 5. Universal Business Rules

1. A Maintenance Ticket must have a Severity classification before it can move from Triaged to Assigned — a Property Manager cannot skip triage.
2. A Maintenance Ticket cannot move to Resolved without the assigned Contractor recording what work was done.
3. A Maintenance Ticket must reference a currently-active Tenancy.

**Deliberately not included here:** any specific response-time number tied to a Severity level. See Section 10 (Known Variations) — this is a variable business policy, not a universal rule, and the reasoning for keeping it out of this section is recorded there in full.

---

## 6. Standard Validations

- A Maintenance Ticket must reference a Tenancy that is currently active (not a past or future tenancy).
- A Severity value must be one of the fixed set (Section 10 names the set; which issue maps to which value varies).
- A ticket cannot be assigned to a Contractor whose trade doesn't match the ticket's issue category (a plumbing issue does not get assigned to an electrical-only contractor).

---

## 7. Common Personas & Roles

| Persona | Typical Access |
|---------|----------------|
| Tenant | Reports and tracks only their own tickets, typically via a link, not a full account. |
| Property Manager | Triages, assigns, and sees every ticket across the properties they manage. |
| Operations Lead | Sees every ticket across the whole managed portfolio; pulls compliance/audit views. |
| Contractor | Sees and updates only tickets assigned to them; low-friction access (e.g. a per-job link) is standard in this domain, since contractors resist account-creation overhead. |

---

## 8. Typical Integration Points

- **Notification (SMS/email):** near-universal in this domain — tenants and contractors expect a status-change notification without needing to check a portal.
- **Existing tenancy/CRM system:** a letting agent typically already holds tenancy and landlord data elsewhere; a maintenance system usually needs to reference it (read or sync), not re-implement it.

---

## 9. Regulatory Baseline (Country-Agnostic Where Possible, UK-Specific Here)

- **Homes (Fitness for Human Habitation) Act 2018 (England):** implies a covenant, in most residential tenancies, that the property is fit for human habitation at the start of and throughout the tenancy. **It does not itself specify a numeric response-time table for any category of disrepair** — this is a common and important misreading to avoid; do not model a specific hours/days figure as sourced directly to this Act.
- **Housing Ombudsman Complaint Handling Code (England):** governs how a letting/managing agent must handle a *complaint* (which a mishandled maintenance issue often becomes) — timely acknowledgement, investigation, and response, with clear record-keeping. A related but distinct obligation from the Fitness for Human Habitation Act's habitability standard itself.
- **Industry-convention severity response bands** (commonly cited: Emergency ≈ 24 hours, Urgent ≈ 3–5 days, Routine ≈ 28 days): widely used across UK letting agents as a way of operationalizing the Act's "reasonable time" standard into practice, but **these are convention, not statute** — **Unverified** in this project specifically until Thornfield's compliance advisor confirms the exact bands Thornfield should adopt (see `domain-discovery-state.md` Section 6, Open Issues).
- **Housing Health and Safety Rating System (HHSRS):** the hazard-scoring framework local authorities use for enforcement action. Whether a letting agent should mirror HHSRS categories internally for its own severity classification, versus using a simpler agent-specific scheme, is **Unverified** — not settled by this domain, genuinely varies by agent.
- This project's client is a private-sector-only letting agent; the social-vs-private-tenancy question from the playbook was resolved as moot for Thornfield specifically, but is preserved here since a future project reusing this domain document might manage social housing, where the applicable standard may differ.

---

## 10. Known Variations

| Topic | Common Options | Default | Must Ask |
|-------|----------------|---------|----------|
| Severity levels used | A 3-tier (Urgent/Routine/Planned) or 4-tier (Emergency/Urgent/Routine/Planned) model are both common | 4-tier, per Thornfield's own stated practice | Confirm the exact tier count and names this project's client actually uses |
| Severity-to-response-time mapping | Industry-convention bands (≈24hr / 3–5 day / 28 day); a stricter per-landlord override; an agent-specific simpler scheme | Industry-convention bands, pending compliance sign-off | Does any individual landlord's contract specify its own, stricter timescale that overrides the agency default? |
| Whether HHSRS categories are mirrored internally | Yes (mirrors local-authority framework); no (agent-specific scheme) | No — treat as a separate, Unverified question per client | Ask the client's compliance advisor directly; do not assume |
| Multi-trade tickets | A single ticket with multiple assignees; a linked secondary ticket per additional trade | Linked secondary ticket (Thornfield's stated low-frequency case) | Confirm frequency and client's actual preference — a high-frequency multi-trade case might justify a first-class multi-assignee model instead |

---

## 11. Typical Scope Boundaries

### Typically In First Implementation

Reactive, tenant-reported ticket lifecycle (report through closure); triage and severity classification; contractor assignment and status updates; tenant/contractor notifications; an auditable timeline per ticket.

### Typically Deferred to Later Phase

Proactive/planned maintenance (scheduled servicing); contractor invoicing and payment; multi-tenant support for agencies other than the initial client.

---

## 12. Document Verification

### 12.1 Internal Consistency

- [x] Every entity in Section 3 appears in at least one business rule (Section 5) or validation (Section 6): Maintenance Ticket (Rules 1–3, all Validations), Tenancy (Rule 3, Validation 1), Contractor (Validation 3).
- [x] Every lifecycle state in Section 4 is reachable and has a defined trigger.

### 12.2 Completeness

- [x] Personas (Section 7) cover every actor named in the discovery session (Tenant, Property Manager, Operations Lead, Contractor).
- [x] Regulatory Baseline (Section 9) names every regulation raised in discovery, not paraphrased generically.

### 12.3 Factual Accuracy Log

| Fact | Status |
|------|--------|
| Fitness for Human Habitation Act 2018 does not specify a numeric response-time table | Verified — corrected during discovery session (see `domain-discovery-state.md` Section 5) |
| Industry-convention response bands (24hr/3-5day/28day) | **Unverified** — pending Thornfield's compliance advisor |
| HHSRS category applicability to Thornfield's internal scheme | **Unverified** — pending Thornfield's compliance advisor |

### 12.4 Edge Case Scenario Log

- A ticket recurring within 14 days of Resolved reopens the same ticket; recurring after Closed creates a new, linked ticket — both paths modeled in Section 4.
- A genuine multi-trade issue is modeled as a linked secondary ticket, not a multi-assignee ticket, per the client's stated low frequency for this case.

### 12.5 Open Issues

Two Regulatory Baseline facts remain Unverified (Section 12.3) — carried forward from `domain-discovery-state.md` Section 6 as BRD-blocking on the specific business rule they affect (severity response-time mapping), not blocking for the rest of BRD discovery.

### 12.6 Mock Discovery Validation

Not run for this project — the live discovery session in `domain-discovery-state.md` already surfaced and corrected a real AI-knowledge error (the response-time misattribution), which is the exact class of gap `/run-mock-discovery` exists to catch. Given a real session already found and fixed a genuine error, and the two remaining gaps are correctly identified as needing an external compliance advisor (not something a simulated client conversation would resolve either), a mock discovery pass was judged low-value for this project — recorded here as a deliberate decision, not an oversight.
