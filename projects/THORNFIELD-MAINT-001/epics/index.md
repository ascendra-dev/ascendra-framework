# Epic Index — THORNFIELD-MAINT-001

## REQ Coverage Map

| REQ-ID | Epic |
|--------|------|
| REQ-001 | EPIC-001 |
| REQ-002 | EPIC-001 |
| REQ-003 | EPIC-001 |
| REQ-004 | EPIC-001 |
| REQ-005 | EPIC-001 |
| REQ-006 | EPIC-001 |
| REQ-012 | EPIC-001 |
| REQ-007 | EPIC-002 |
| REQ-008 | EPIC-002 |
| REQ-009 | EPIC-003 |
| REQ-010 | EPIC-003 |
| REQ-011 | EPIC-003 |
| REQ-013 | EPIC-004 |

## Dependency Graph

Build order: **EPIC-004 → EPIC-001 → {EPIC-002, EPIC-003}**

- **EPIC-004 (CRM Integration)** is the true dependency root, not EPIC-001 — REQ-001 (ticket creation) requires a Tenant to select an existing Tenancy, and Tenancy records only exist in this system once EPIC-004's nightly sync has populated them. Applying the falsifiable dependency test (`practitioner-guide/04-product-structuring.md`): a ticket-creation story in EPIC-001 cannot be meaningfully demoed without EPIC-004's sync having run at least once — this is a real story-level dependency, not "CRM integration feels foundational."
- **EPIC-001 (Maintenance Ticket Lifecycle)** depends on EPIC-004, and is itself a dependency of EPIC-002 and EPIC-003.
- **EPIC-002 (Notifications)** depends on EPIC-001 — a notification story needs a real status-change event to fire on; it cannot be demoed against nothing.
- **EPIC-003 (Compliance & Audit Timeline)** depends on EPIC-001 — the timeline it displays is built from EPIC-001's own state-change history; there is nothing to display until tickets exist and move through states.
- EPIC-002 and EPIC-003 do not depend on each other — both can be built in parallel once EPIC-001 exists.

**Story-Level Interleaving:** None — every dependency above resolves at the epic level (each dependent epic needs its prerequisite epic's Definition of Done met, not one specific story within it).

## Epics

| Epic ID | Title | Phase | Layer | Status |
|---------|-------|-------|-------|--------|
| EPIC-001 | Maintenance Ticket Lifecycle | Phase 1 | Core | Approved |
| EPIC-002 | Notifications | Phase 1 | Core | Approved |
| EPIC-003 | Compliance & Audit Timeline | Phase 1 | Core | Approved |
| EPIC-004 | CRM Integration | Phase 1 | Core | Approved |
