## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version from approved BRD v1.0 |

---

**Epic ID:** EPIC-002
**Title:** Notifications
**Phase:** Phase 1
**Layer:** Core
**Status:** Approved

### 1. Goal

A Tenant or Contractor always knows the current state of a ticket that concerns them, without checking a portal — they're told, on their preferred channel, the moment something changes.

### 2. BRD Requirements Covered

| REQ-ID | Description |
|--------|-------------|
| REQ-007 | Tenant notified on every status change, via their chosen channel |
| REQ-008 | Contractor notified by SMS only on assignment |

### 3. Scope

**3.1 In Scope**
- Tenant notification on every ticket status change (email or SMS, tenant's choice)
- Contractor notification on assignment (SMS only)

**3.2 Out of Scope**
- The events being notified on (ticket creation, status transitions) — owned by EPIC-001
- Notification content beyond what REQ-007/REQ-008 specify — no marketing, no digest/summary notifications this phase

### 4. Personas Involved

Tenant, Contractor.

### 5. Dependencies

**Depends on:** EPIC-001 — a notification is triggered by a real status-change event; there is nothing to notify on until EPIC-001 produces one. This is genuinely cross-cutting in implementation (this epic's stories will likely be event listeners reacting to EPIC-001's own state transitions), which is exactly why it was flagged for attention during `/judgment-check` on the BRD (see `brd-core-v1.md` Section 17) rather than silently folded into EPIC-001 — kept separate because the client treats a notification failure as an independently trackable failure mode from a workflow failure, and it has its own real Definition of Done below.

### 6. Definition of Done

Every status change on a real ticket in EPIC-001 produces the correct notification, on the correct channel, to the correct recipient, verifiable via a real test message — without needing EPIC-001's own stories to be re-touched to add the notification call.

### 7. Stories

*Not yet generated — populated by `/gen-stories`.*

### 8. Notes

- This is the first, clearest candidate for `ascendra-commons`'s `notification` module, if the confirmed backend stack matches — see `reference/ascendra-commons/adoption-guide.md`. Flagged here for the architecture phase to evaluate against a real signal (this epic exists specifically because of it), not proposed speculatively.
- REQ-008's "SMS only, never email" for Contractors is a real client instruction (James: contractors don't check email on site) — do not let a story default to a generic "send via all configured channels" pattern for this recipient.

### 9. Density & Judgment Findings

*Not yet run against this epic file individually.*

### 10. Density & Judgment Resolution

*Not applicable — see Section 9.*

---

## Part 2 — Verification

- [x] Goal is a user outcome, not a technical deliverable
- [x] Every REQ-ID in Section 2 matches the approved BRD exactly
- [x] Out of Scope names the covering epic for every exclusion
- [x] Dependency is a real story-level need, stated with reasoning, not asserted
- [x] Definition of Done is demoable end to end
