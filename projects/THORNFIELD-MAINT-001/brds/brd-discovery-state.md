## 1. Project & Session Context

**Project Code:** THORNFIELD-MAINT-001
**Client contact:** Priya Shah (Operations Lead), with James Whitfield (Senior Property Manager) joining for the workflow questions

## 2. Session History

| Session | Date | Duration | Topics Covered |
|---------|------|----------|-----------------|
| 1 | 2026-09-16 | ~70 min | All structured questions, all journeys, SRF-001–003 |

## 3. Playbook Progress

### Section 4 — Core Discovery Questions

| Topic | Status |
|-------|--------|
| Severity tier set and names | Covered |
| Triage → assignment → resolution workflow | Covered |
| Notification channels and timing | Covered |
| Existing CRM integration approach | Covered |
| 14-day reopen-vs-recurrence boundary | Covered |

### Section 6 — Journey Walkthroughs

| Journey | Status |
|---------|--------|
| Report → triage → assign → resolve → notify | Covered |
| Operations Lead pulls compliance timeline | Covered |

## 4. Scope Risk Flags

| ID | Flag | Status | Resolution |
|----|------|--------|-----------|
| SRF-001 | Severity-to-response-time mapping unresolved | Resolved this session | Priya confirmed: model severity levels now, keep exact response-time targets per severity as a configurable value, not hard-coded — she'll supply the confirmed numbers once the compliance advisor responds, expected within 2 weeks, well before this phase's sprint work reaches the notification-timing story |
| SRF-002 | HHSRS category mirroring | Resolved this session | Priya: "No, keep it simple — our own four levels, not HHSRS categories. HHSRS is for council enforcement, not us." Genuinely new information (not just an unusual value — see below) |
| SRF-003 | Multi-trade ticket frequency | Resolved this session | James confirmed: 3–5 times a year, genuinely rare. Linked secondary ticket approach confirmed as sufficient, not a first-class multi-assignee model |

## 5. Integration Confirmation

Existing CRM (client uses a generic CRM product for tenancy/landlord records, name withheld by client pending their own vendor conversation): confirmed as **read-only lookup** for this phase — the maintenance system reads tenancy/tenant/landlord identifiers from the CRM via a scheduled sync (nightly is acceptable to Priya), never writes back. A real-time API integration was discussed but Priya deprioritized it — "the CRM isn't going anywhere and we're not in a rush, nightly is fine, don't want to hold up the maintenance work waiting on a vendor API conversation."

## 6. Output Confirmation

Session confidence: 91%. All core topics covered with clear, specific answers. The one still-open item (SRF-001's actual numeric values) is deliberately deferred, not unresolved — Priya was explicit and specific about the plan (configurable field, numbers to follow), which is a resolved decision about *how* to handle the gap, not an open gap itself.

## 7. Open Questions

| Question | Owner | Due | Notes |
|----------|-------|-----|-------|
| Confirmed numeric response-time targets per severity level | Priya Shah (via compliance advisor) | Within 2 weeks of this session | Tracked as a config value to populate later, not a blocker to BRD generation — see Requirement REQ-011 |

## 8. Captured Client Answers (BRD Input)

### Severity tier set and names

Priya: "We want to keep our own four levels — Emergency, Urgent, Routine, Planned — not switch to anything HHSRS-based. That's what James's team already thinks in, no point retraining everyone on new terminology."

### Triage → assignment → resolution workflow

James: "A ticket comes in, one of us picks it up, sets the severity, and either assigns it to a contractor straight away or comes back to it — we don't want the system forcing an assignment at the same moment as triage, sometimes I triage a batch first thing and assign after I've checked contractor availability."

### Notification channels and timing

Priya: "Email and SMS both, tenant chooses which when they first report it. Contractors — just SMS, they don't check email on site."

### Existing CRM integration

Covered under Section 5 above.

### 14-day reopen-vs-recurrence boundary

James: "Yes, two weeks is right — that's roughly how we've always drawn the line informally, glad to see it written down properly."

## 9. Resume Instructions

Session complete in one pass — no resume needed.

## 10. Pre-Generation Verification

- [x] All playbook topics Covered
- [x] All Scope Risk Flags resolved or explicitly deferred with an owner and plan
- [x] Session confidence ≥ 85% (91%)
- [x] No unresolved scope-blocking Open Question — the one Open Question is a config-value-to-follow, not a scope decision
