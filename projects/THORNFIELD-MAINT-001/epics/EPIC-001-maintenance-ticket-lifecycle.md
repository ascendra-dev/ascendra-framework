## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version from approved BRD v1.0 |

---

**Epic ID:** EPIC-001
**Title:** Maintenance Ticket Lifecycle
**Phase:** Phase 1
**Layer:** Core
**Status:** Approved

### 1. Goal

A Property Manager can take a tenant-reported maintenance issue from report through triage, contractor assignment, work, and resolution — with a Contractor able to act on their own assigned tickets — without any step being lost or untracked.

### 2. BRD Requirements Covered

| REQ-ID | Description |
|--------|-------------|
| REQ-001 | Tenant reports an issue without an account, receives a tracking link |
| REQ-002 | Severity must be set before assignment |
| REQ-003 | Assignment is a separate action from triage |
| REQ-004 | Contractor must record work done before Resolved |
| REQ-005 | 14-day reopen vs. post-Closed recurrence |
| REQ-006 | Rare multi-trade case via linked secondary ticket |
| REQ-012 | Contractor views/updates only their own assigned tickets, via link |

### 3. Scope

**3.1 In Scope**
- Tenant issue reporting and Tenancy identification
- Severity classification at triage
- Contractor assignment (independent of triage timing)
- Contractor status updates and work-done recording
- Reopen (within 14 days) and recurrence (after Closed) handling
- Linked secondary tickets for rare multi-trade cases

**3.2 Out of Scope**
- Notification delivery mechanics (EPIC-002)
- The compliance timeline view (EPIC-003)
- Tenancy data origination (EPIC-004 — this epic consumes Tenancy records, it does not create them)

### 4. Personas Involved

Tenant, Property Manager, Contractor.

### 5. Dependencies

**Depends on:** EPIC-004 (CRM Integration) — a ticket cannot be created against a Tenancy that doesn't yet exist in this system; EPIC-004's nightly sync must have populated at least one Tenancy before a real ticket-creation story can be demoed.

### 6. Definition of Done

A Property Manager can triage and assign a real tenant-reported ticket; a Contractor can accept, work, and resolve it via their own link; the ticket can be reopened or correctly recur per the 14-day rule; a rare multi-trade case can be modeled as linked tickets — all without any manual, off-system step.

### 7. Stories

*Not yet generated — populated by `/gen-stories`.*

### 8. Notes

- REQ-005's 14-day reopen boundary and the recurrence-vs-reopen distinction are business rules James was explicit about — do not let a story implementation quietly treat "recurs after Closed" as the same case as "recurs within 14 days of Resolved."
- Domain knowledge Section 4 (Lifecycle) is the authoritative state model for this epic's stories.

### 9. Density & Judgment Findings

*No findings — `/judgment-check` was not re-run against this epic file individually; its BRD-level findings (Section 16/17 of `brd-core-v1.md`) already resolved the boundary question this epic's own scope reflects (REQ-012 folded in here rather than left as a separate feature area).*

### 10. Density & Judgment Resolution

*Not applicable — see Section 9.*

---

## Part 2 — Verification

- [x] Goal is a user outcome, not a technical deliverable
- [x] Every REQ-ID in Section 2 matches the approved BRD exactly
- [x] Out of Scope names the covering epic for every exclusion
- [x] Dependencies apply the falsifiable test — a real story-level need, not "feels foundational"
- [x] Definition of Done is demoable end to end
