## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version from approved BRD v1.0 |

---

**Epic ID:** EPIC-003
**Title:** Compliance & Audit Timeline
**Phase:** Phase 1
**Layer:** Core
**Status:** Approved

### 1. Goal

The Operations Lead can produce a complete, trustworthy, timestamped account of any ticket's handling, on demand, without depending on a Property Manager's memory or notes — the thing Thornfield needs if a tenant or the Housing Ombudsman disputes how a repair was handled.

### 2. BRD Requirements Covered

| REQ-ID | Description |
|--------|-------------|
| REQ-009 | Timestamped entry for every state change, visible to Operations Lead for any ticket |
| REQ-010 | Full timeline retrievable in under a minute |
| REQ-011 | Configurable, not hard-coded, response-time target per Severity |

### 3. Scope

**3.1 In Scope**
- Recording a timestamped entry for every state change EPIC-001 produces
- A view, scoped to Operations Lead, of any ticket's full timeline regardless of which Property Manager handled it
- A configurable per-Severity response-time target, editable by an Operations Lead without a code change

**3.2 Out of Scope**
- The state changes themselves — owned by EPIC-001, this epic only records and surfaces them
- Any specific numeric response-time value — REQ-011 is explicit that Thornfield's compliance advisor has not yet confirmed these (domain knowledge Section 12.3); this epic delivers the configuration mechanism, not the values

### 4. Personas Involved

Operations Lead.

### 5. Dependencies

**Depends on:** EPIC-001 — the timeline is built from EPIC-001's own state-change history. Nothing to display exists before a ticket has moved through at least one transition.

### 6. Definition of Done

An Operations Lead can open any real ticket and see its complete, accurate timeline in under a minute; a Severity's response-time target can be changed via configuration, by an Operations Lead, without a developer or a deployment.

### 7. Stories

*Not yet generated — populated by `/gen-stories`.*

### 8. Notes

- This epic is the clearest candidate for `ascendra-commons`'s `audit-logging` module (which itself depends on `domain-events`), if the confirmed backend stack matches — see `reference/ascendra-commons/adoption-guide.md`. The client's own stated need (a trustworthy, tamper-evident-feeling timeline for a compliance dispute) is a real, specific signal, not a speculative fit — flagged for the architecture phase to evaluate.
- REQ-011's configurability requirement is a direct consequence of the domain knowledge document's own Unverified flag on the actual response-time numbers (Section 12.3) — do not let a story implementation quietly hard-code a placeholder number "for now."

### 9. Density & Judgment Findings

*Not yet run against this epic file individually.*

### 10. Density & Judgment Resolution

*Not applicable — see Section 9.*

---

## Part 2 — Verification

- [x] Goal is a user outcome, not a technical deliverable
- [x] Every REQ-ID in Section 2 matches the approved BRD exactly
- [x] Out of Scope names the covering epic for every exclusion
- [x] Dependency is a real story-level need, stated with reasoning
- [x] Definition of Done is demoable end to end
