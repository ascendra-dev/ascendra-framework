## 1. Session Context

**Project:** THORNFIELD-MAINT-001
**Domain:** Property Maintenance & Repairs (Residential Letting)
**Playbook:** `property-maintenance-domain-playbook.md`
**Sessions held:** 1
**Status:** Complete

## 2. Session History

| Session | Date | Duration | Sections Covered |
|---------|------|----------|-------------------|
| 1 | 2026-09-16 | ~55 min | Sections 4–13 (all) |

## 3. Domain Confidence Score

**Score: 78%**

Not a naive ratio of sections covered — weighted toward Section 11 (Regulatory Baseline), the highest-risk section for this domain per the intake self-test. Sections 4–8 and 13 (the generic ticketing/entity/lifecycle model) are high-confidence — this is a well-understood workflow pattern the AI already knew well and the session mostly confirmed rather than corrected. Section 11 (Regulatory Baseline) is the one section genuinely holding the score down: the session surfaced that the AI's initial framing of the Fitness for Human Habitation Act as specifying a precise response-time table was wrong, and the corrected, honest position is that the precise timing framework is industry convention, not statute — which is exactly right to know, but it also means this section carries two Unverified rows (see Section 11 below) that a compliance advisor, not this session, needs to close out. Per the project's own brief Section 8, this is already a named Open Question owned by Priya Shah.

**Given this domain includes a genuine regulatory dimension, the practitioner guide's raised bar applies: do not treat 78% as sufficient on its own.** Section 11's two Unverified rows are treated as hard blockers for BRD generation, not soft notes — see Section 6 (Open Issues) below.

## 4. Playbook Progress

### Section 4 — Domain Scope

**Status:** Covered
**PO stated:** Reactive, tenant-reported maintenance only, for this phase — proactive/planned maintenance (annual boiler service, gutter clearing) is explicitly out of scope. Contractor invoicing and payment stay in Thornfield's existing finance process; this system tracks *that* a job was completed and by whom, not the money.

### Section 5 — Core Entities

**Status:** Covered
**PO stated:** A `MaintenanceTicket` (raised by a Tenant against a specific `Tenancy`/unit), a `Contractor` (external, may work multiple trades), and — confirmed during the session — a `Property` sits above `Tenancy` (a Property can have more than one Tenancy, e.g. a converted house split into flats), which matters for the shared/communal-area question in the playbook.

### Section 6 — Lifecycle & Status Model

**Status:** Covered
**PO stated:** Reported → Triaged → Assigned → Accepted (by contractor) → In Progress → Resolved → Closed. A ticket can be reopened from Resolved back to In Progress if the tenant reports the same issue recurring within 14 days — after Closed, a recurrence is a new ticket referencing the old one, not a reopen (James was clear this distinction matters for his own reporting). Confirmed: a single ticket can have more than one contractor only in a genuine multi-trade case (the leak/drywall example from the playbook) — James said this is rare enough (a handful a year) that it's tracked as a linked secondary ticket, not a first-class multi-assignee model on one ticket.

### Section 7 — Universal Business Rules

**Status:** Covered
**PO stated:** Every ticket must have a severity classification before it can be assigned (a property manager cannot skip triage). A ticket cannot move to Resolved without the contractor recording what was done. **Explicitly NOT included here:** any specific response-time number — see Section 11, this stays out of Universal Business Rules on purpose.

### Section 8 — Standard Validations

**Status:** Covered
**PO stated:** A ticket must reference a valid, currently-active Tenancy. A severity classification is required at triage, chosen from a fixed set (see Section 12 — the set itself is fixed, but which severity a given issue type maps to is where the variation lives).

### Section 9 — Common Personas & Roles

**Status:** Covered
**PO stated:** Tenant (reports and tracks their own tickets only, via a link — no account), Property Manager (triages, assigns, sees everything across their managed properties), Operations Lead (sees everything across the whole book, pulls compliance/audit views), Contractor (sees and updates only tickets assigned to them, no account creation friction — a magic-link-per-job was explicitly requested by James based on what he knows contractors will actually tolerate).

### Section 10 — Typical Integration Points

**Status:** Covered
**PO stated:** SMS/email notification to tenants and contractors on status change (confirmed as a real need, not a nice-to-have — this is Success Criterion 2 from the brief). The existing CRM holds tenancy data Thornfield doesn't want to re-enter — flagged as an integration point to resolve at BRD stage, not decided here.

### Section 11 — Regulatory Baseline

**Status:** Covered — with two rows marked Unverified, not resolved in this session
**PO stated / AI correction recorded:**

| Fact | Source | Status |
|------|--------|--------|
| The Homes (Fitness for Human Habitation) Act 2018 implies a covenant that a rented property is fit for human habitation throughout the tenancy — it does **not** itself specify a numeric response-time table for any disrepair category | AI training knowledge, corrected during session (see Section 5, AI Knowledge Corrections) | Verified — this is the Act's actual mechanism, not an assumption |
| Industry-convention response bands (commonly: Emergency ≈ 24 hours, Urgent ≈ 3–5 days, Routine ≈ 28 days) are widely used across UK letting agents as good practice, but are **not themselves a statutory requirement** — they're how a letting agent typically operationalizes the Act's general "reasonable time" standard | AI training knowledge | **Unverified** — plausible and commonly cited, but not confirmed against a specific authoritative source in this session; needs the compliance advisor's sign-off before being treated as Thornfield's actual policy |
| Whether HHSRS hazard categories should independently drive Thornfield's own severity classification, separate from the Act | AI training knowledge | **Unverified** — HHSRS is the framework local authorities use for enforcement; whether Thornfield should mirror its categories internally, or use its own simpler scheme, was not settled in this session |
| Whether social vs. private tenancy status changes anything — Thornfield's book is private-sector only | PO-stated (Priya, via James) | Verified for Thornfield's actual scope — moot for this project since Thornfield manages no social housing, but noted so a future project reusing this domain doc doesn't assume the finding generalizes |
| The Housing Ombudsman's Complaint Handling Code requires timely response and clear record-keeping for any complaint, including one arising from a maintenance issue — it governs *how Thornfield must respond to a complaint*, which is a related but distinct obligation from the Fitness for Human Habitation Act's habitability standard itself | AI training knowledge | Verified at a general level — the distinction between the two regimes is correct; specific Code timescales not independently checked in this session |

### Section 12 — Known Variations

**Status:** Covered
**PO stated:** Which severity a given issue type maps to, and the exact response-time expectation for each severity level, are **not** fixed platform-wide — Thornfield's own policy may differ from another letting agent's, and per the brief's Section 8 open question, an individual landlord's instruction can be stricter than Thornfield's own default. This is recorded here as a Known Variation (Topic: severity-to-response-time mapping; Common Options: the industry-convention bands above, a stricter per-landlord override, a simpler agent-specific 3-tier model; Default: Thornfield's own bands once the compliance advisor confirms them; Must Ask: does any individual landlord contract specify its own timescales) — explicitly not a Universal Business Rule, for the reason recorded in Section 11 above.

### Section 13 — Typical Scope Boundaries

**Status:** Covered
**PO stated:** Reactive maintenance only, this phase — matches Section 4. Proactive/planned maintenance and contractor invoicing are named as "Typically Deferred to Later Phase," not permanently excluded.

## 5. AI Knowledge Corrections

**Correction 1 (Section 11):** The AI's first-pass answer, offered before the PO's own input, stated the Fitness for Human Habitation Act 2018 "requires urgent repairs — no heating, no hot water — to be actioned within 24 hours." This is wrong in a specific, important way: the Act does not itself specify any numeric window. Priya (relayed via James, pending direct compliance-advisor confirmation) corrected this: the 24-hour figure is a widely-used industry convention for operationalizing the Act's general "reasonable time" standard, not a number the Act itself states. The correction was accepted and Section 11 rewritten to reflect it, with the specific-number claim demoted from a stated fact to an Unverified, sourced-to-industry-convention note. This is exactly the failure mode `practitioner-guide/02-domain-discovery.md` warns about: a plausible, specific-sounding regulatory "fact" that would have been wrong if asserted as a Universal Business Rule.

## 6. Open Issues

| Issue | Owner | Status |
|-------|-------|--------|
| Confirm the actual response-time bands Thornfield should adopt, with the compliance advisor, before these are written into the BRD as a business rule | Priya Shah | Open — blocks BRD Section 6 (Business Rules) on this specific point, does not block the rest of BRD discovery |
| Confirm whether Thornfield should mirror HHSRS hazard categories internally or use its own simpler severity scheme | Priya Shah | Open — same blocking scope as above |
| Existing CRM integration approach (read-only lookup vs. one-way sync vs. no integration this phase) | Priya Shah / Thornfield's CRM vendor | Open — does not block BRD generation, resolve during BRD discovery |

## 7. Resume Instructions

Session complete — one pass covered all playbook sections. If a second session is needed (e.g. once the compliance advisor responds), resume by revisiting Section 11 only; do not re-ask Sections 4–10 or 12–13, already confirmed.

## 8. Pre-Generation Verification

- [x] Every playbook section has a Covered status in Section 4 above
- [x] Every "Covered" section has a substantive "PO stated" paragraph, not a placeholder
- [x] Domain Confidence Score is stated with its reasoning, not just a bare number
- [x] Every regulatory fact is either Verified (with why) or explicitly marked Unverified — none asserted as plain fact
- [ ] All Unverified regulatory facts resolved — **intentionally not checked**: two rows in Section 11 remain Unverified by design, tracked in Section 6 Open Issues, not silently dropped
