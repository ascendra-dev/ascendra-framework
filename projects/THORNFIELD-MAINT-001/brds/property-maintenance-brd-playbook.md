**Domain:** Property Maintenance & Repairs
**Project:** THORNFIELD-MAINT-001
**Source:** `domain/property-maintenance-core.md`

---

## Structured Questions (from Domain Sections 4–8)

1. Confirm the exact Severity tier set and names Thornfield wants to use.
2. Confirm the triage → assignment → resolution workflow matches Thornfield's actual practice, and who can perform each step.
3. Confirm notification channel(s) and timing expectations (tenant and contractor).
4. Confirm the existing CRM integration approach for tenancy data.
5. Confirm the 14-day reopen-vs-recurrence boundary is correct for Thornfield's reporting needs.

## Known Variations → Scope Risk Flags (from Domain Section 10)

- **SRF-001 — Severity-to-response-time mapping.** Domain doc marks this Unverified pending compliance advisor input. **Response to client:** "We can't finalize the exact response-time policy for each severity level until your compliance advisor confirms it — we'll model the severity levels and the workflow now, and treat the specific time windows as configurable, to be set once you have that confirmation. This won't block the rest of the BRD."
- **SRF-002 — HHSRS category mirroring.** **Response to client:** "Do you want the system's severity categories to mirror the official HHSRS hazard categories, or is a simpler internal scheme fine? This affects how many severity levels we build and what each is called — but not the core workflow."
- **SRF-003 — Multi-trade tickets.** **Response to client:** "You mentioned multi-trade issues are rare — a handful a year. We'll model that as a linked secondary ticket rather than building full multi-assignee support, unless that's actually more common than expected. Can you confirm the frequency?"

## Journey Walkthroughs

1. Tenant reports an issue → Property Manager triages and assigns → Contractor accepts and resolves → Tenant is notified throughout.
2. Operations Lead pulls a compliance timeline for a disputed ticket.
