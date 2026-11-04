Thornfield Property Management Ltd — Maintenance Ticketing System (BRD v1.0, Approved).

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-09-16 | Ascendra AI | Initial version |

---

## Part 1 — Template

### 1. Portals

**Applying the Portal boundary test (`practitioner-guide/05-screen-design.md`):** a Portal is a navigation shell — one app/session. Tenant and Contractor access is link-based, single-purpose, no persistent session or navigation shell at all — neither qualifies as a Portal under that test; each is a standalone linked page, not modeled as a Portal here. Property Manager and Operations Lead share one authenticated session and nav shell — one Portal, matching the base rule (same app, navigation differs only by visibility), with no auth-flow split to justify separating them the way the harder-cases.md Ascendra Pay example needed to for Staff vs. Family.

| Portal | Personas | Access model | Base route |
|---|---|---|---|
| Staff App | Property Manager, Operations Lead | Standing session (Google Workspace SSO, per brief Section 6) | `/app` |

### 2. Navigation Map

**Staff App** — flat:

| Nav item | Portal | Visibility condition | Grouping |
|---|---|---|---|
| Dashboard | Staff App | baseline | flat |
| Tickets | Staff App | baseline | flat |

### 3. Screen Inventory

| Screen ID | Portal | Route | Reachable by | Purpose | Source | Surface | UI Pattern | Notes |
|---|---|---|---|---|---|---|---|---|
| SCR-001 | Staff App | `/app/tickets` | baseline | Browse all tickets, filter by status/severity | Journey 4.2, step 1, REQ-002 | Page | Table-List | |
| SCR-002 | Staff App | `/app/tickets/{id}` | baseline | Ticket detail — triage, assign, timeline | Journey 4.2, steps 1–2; Journey 4.3, steps 2–3, REQ-002/003/009/010 | Page | Detail | Operations Lead's compliance view is this same screen, full-portfolio scoped |
| SCR-003 | (no portal — linked page) | `/report` | public link | Tenant reports an issue | Journey 4.1, step 1, REQ-001 | Page | Form | Not part of a Portal — no nav shell, single-purpose link |
| SCR-004 | (no portal — linked page) | `/track/{token}` | tracking link | Tenant views their ticket's status | Journey 4.1, steps 3–4, REQ-007 | Page | Detail | Same reasoning as SCR-003 |
| SCR-005 | (no portal — linked page) | `/job/{token}` | per-job link | Contractor accepts, updates status, records work | Journey 4.2, steps 3–4, REQ-004/012 | Page | Form | Same reasoning as SCR-003 |

### 4. Per-Screen Data Requirements

| Screen ID | Fields shown/collected | Actions available | Notes |
|---|---|---|---|
| SCR-001 | Ticket ID, Tenancy, Severity, Status, Assigned Contractor | Filter, sort, open detail | |
| SCR-002 | Full ticket detail, full timestamped timeline, Severity, Contractor assignment | Set Severity, assign Contractor, close/reopen | REQ-009/010's timeline is this screen's core content |
| SCR-003 | Tenancy code, issue description, notification channel choice | Submit | |
| SCR-004 | Ticket status, timeline (Tenant-visible subset) | None — read-only | |
| SCR-005 | Ticket detail, status update, work-done notes | Accept, update status, record work done | |

---

## Part 2 — Verification

- [x] Every journey in BRD Section 4 maps to at least one Screen ID
- [x] Every Portal is a genuinely distinct navigation shell — Tenant/Contractor correctly excluded as link-based, not forced into a Portal shape they don't have
- [x] Every Screen Inventory row has a Surface and UI Pattern
- [x] Every screen in Section 3 has a corresponding row in Section 4
