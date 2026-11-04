**Project Name:** Thornfield Maintenance Ticketing System
**Project Code:** THORNFIELD-MAINT-001
**Client:** Thornfield Property Management Ltd
**Domain:** Property Maintenance & Repairs
**Status:** Active
**Content Status:** Approved
**Created:** 2026-09-16
**Last Updated:** 2026-09-16
**Project Type:** Standalone
**Extends:** —
**Extension Adds:** —

### 1. Problem Statement

Thornfield currently manages maintenance requests by phone call and a shared email inbox, with no shared record of what's open, who it's assigned to, or when it was actually reported. Tenants frequently go a week or more without any update, and property managers have no reliable way to confirm which urgent repairs — no heating, no hot water, damp causing a health risk — were reported when, which creates real exposure under the Homes (Fitness for Human Habitation) Act 2018 if a tenant or the Housing Ombudsman later disputes the timeline. If the problem were solved, every request would have a timestamped record from submission to resolution, and no urgent repair would go untracked.

### 2. Client Context

Thornfield Property Management Ltd is a mid-sized letting and management agency in Greater Manchester and Leeds, managing roughly 600 residential tenancies on behalf of about 80 landlords. The team is four property managers and one operations lead, coordinating work across 15–20 contractors spanning plumbing, electrical, and general trades. They currently use a generic CRM for tenant and landlord records, but it has no maintenance-specific workflow — maintenance is tracked entirely outside it, by phone and a shared inbox. As a regulated letting agent, Thornfield is subject to the Housing Ombudsman's Complaint Handling Code and the Homes (Fitness for Human Habitation) Act 2018, both of which shape how urgent repairs must be handled and evidenced.

### 3. Strategic Goals

1. Reduce average time from a tenant reporting a maintenance issue to a contractor being assigned and starting work.
2. Give the Operations Lead full visibility into every open and overdue ticket without having to chase property managers individually.
3. Be able to demonstrate, for any ticket, a complete and accurate timeline of when it was reported and how it was handled — for Ombudsman or tenant disputes.
4. Reduce the time property managers spend on status-chasing phone calls from tenants and contractors.

### 4. Key Stakeholders

| Name | Role | Decision Authority | Involvement |
|------|------|--------------------|-------------|
| Priya Shah | Operations Lead | Final sign-off | Weekly calls, UAT participation, approves BRD and sprint plans |
| James Whitfield | Senior Property Manager | Daily user | Heavy input on the triage/assignment workflow — will use the system every day |
| Contractor representative (TBC) | External Contractor | Input only | Consulted once on the contractor-facing status-update flow |

### 5. Constraints

- **Budget:** Capped at a range agreed separately with Priya Shah; not fixed to the last pound.
- **Deadline:** Not contractually fixed, but Thornfield wants the heating-related urgent-repair flow live before the start of the winter heating season — the highest-volume urgent category.
- **Technology:** None.
- **Regulatory:** Homes (Fitness for Human Habitation) Act 2018 (statutory timelines for certain categories of disrepair); Housing Ombudsman Complaint Handling Code (record-keeping and response-time expectations for complaint-adjacent issues).
- **Other:** None.

### 6. Technology Preferences

- **Existing systems:** A generic CRM (tenant/landlord records) that Thornfield wants to keep using for tenancy data — no hard integration requirement identified yet, but the CRM should not be replaced by this project.
- **Preferred stack:** No preference — client defers to delivery team.
- **Hosting:** Cloud, no provider preference.
- **Authentication:** Thornfield staff use Google Workspace internally and would like staff login to use it if practical. Tenants and contractors have no existing account system with Thornfield — standalone auth (or a token-link flow, given many tenants won't want to create an account for one-off use).

### 7. Success Criteria

1. A property manager can triage and assign a new maintenance request to a contractor within 10 minutes of it arriving — verified in UAT.
2. A tenant receives a notification within 15 minutes of any status change to their reported issue.
3. The Operations Lead can pull a complete, timestamped history for any single ticket in under a minute, without asking a property manager for context.
4. A contractor can update a ticket's status themselves, without phoning the office, for at least 80% of jobs.

### 8. Risks and Open Questions

| Item | Type | Owner | Notes |
|------|------|-------|-------|
| Contractors may resist a new system and keep calling/texting the office instead | Risk | Priya Shah | Mitigation: keep the contractor-facing flow to a single status update per visit, no account creation required |
| Whether the statutory urgent-repair response window depends only on the disrepair category, or also varies by tenancy type (social vs. private) | Open Question | Priya Shah (with Thornfield's compliance advisor) | Must be answered before the urgent-repair business rule can be written correctly — flagged for domain/BRD discovery |
| Whether Thornfield wants this built for their own use only, or eventually offered to other letting agents they know | Open Question | Priya Shah | Affects whether multi-tenancy is designed for now or left for a future phase |

### 9. Related Artifacts

| Artifact | Location | Status |
|----------|----------|--------|
| Project Brief | `projects/THORNFIELD-MAINT-001/brief.md` | Draft |

---

## Verification

- [x] Problem Statement describes the current-state problem, not the solution — no sentence begins with "We are building..."
- [x] Client Context includes only information relevant to system design — no company history or marketing content
- [x] Every Strategic Goal is an outcome, not a feature — each goal could in principle be achieved without the system
- [x] Every stakeholder has a named Decision Authority from the defined options
- [x] Every Constraint in Section 5 is genuinely non-negotiable — preferences have been moved to Section 6
- [x] Every named regulation in Section 5 is specific (Homes (Fitness for Human Habitation) Act 2018; Housing Ombudsman Complaint Handling Code) — not generic ("housing law")
- [x] Technology Preferences in Section 6 are distinguished from hard mandates in Section 5
- [x] Every Success Criterion is observable — a Product Owner can verify it in UAT without asking the client for a subjective opinion
- [x] Every item in Section 8 has a named owner — no row with "TBD" in the Owner column
- [x] Related Artifacts in Section 9 reflects the current state of all project documents — no stale "Draft" statuses on approved documents
- [x] Project Type is Standalone — Extends/Extension Adds not applicable
