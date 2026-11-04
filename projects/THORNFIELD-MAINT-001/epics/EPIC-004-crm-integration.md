## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version from approved BRD v1.0 |

---

**Epic ID:** EPIC-004
**Title:** CRM Integration
**Phase:** Phase 1
**Layer:** Core
**Status:** Approved

### 1. Goal

Every other epic can trust that Tenancy, Tenant, and Landlord data in this system reflects Thornfield's existing CRM, without this system ever becoming a second, divergent source of truth for that data.

### 2. BRD Requirements Covered

| REQ-ID | Description |
|--------|-------------|
| REQ-013 | Nightly read-only sync of Tenancy/Tenant/Landlord data from the existing CRM |

### 3. Scope

**3.1 In Scope**
- A nightly, read-only sync job pulling Tenancy, Tenant, and Landlord identifying data from Thornfield's existing CRM
- Making synced Tenancy records available for a Tenant to select when reporting an issue (EPIC-001)

**3.2 Out of Scope**
- Any write-back to the CRM
- Real-time sync — explicitly deferred (BRD REQ-015, Section 5.21)

### 4. Personas Involved

None directly — this epic has no user-facing screen or interaction of its own; it exists to make EPIC-001 possible.

### 5. Dependencies

**Depends on:** Nothing within this project — this is the dependency root. Every other epic depends on it, directly or transitively.

### 6. Definition of Done

A real Tenancy record from Thornfield's CRM is visible and selectable inside this system within 24 hours of existing in the CRM, with no manual re-entry step.

### 7. Stories

*Not yet generated — populated by `/gen-stories`.*

### 8. Notes

- Despite being small (one REQ), this is the true dependency root — see `epics/index.md`'s Dependency Graph reasoning. Do not deprioritize this epic in sprint sequencing on the assumption that "it's only one requirement, it can wait" — nothing in EPIC-001 can be meaningfully demoed without it.
- The specific CRM product was not named during discovery (client preferred not to commit before their own vendor conversation) — the architecture phase must confirm the actual integration mechanism (API, file export, or other) before this epic can be story-planned.

### 9. Density & Judgment Findings

*Not yet run against this epic file individually.*

### 10. Density & Judgment Resolution

*Not applicable — see Section 9.*

---

## Part 2 — Verification

- [x] Goal is a user outcome, not a technical deliverable (here: an enabling outcome for other epics, stated as such rather than as a technical mechanism)
- [x] Every REQ-ID in Section 2 matches the approved BRD exactly
- [x] Out of Scope names the covering epic or BRD Section 5.21 item for every exclusion
- [x] Dependencies correctly state this epic as the root, not assumed foundational without the falsifiable test applied
- [x] Definition of Done is demoable end to end
