Harborview Consulting Ltd — Invoice Management System (BRD v1.0, Approved). This example demonstrates: a single-portal Staff App with a flat navigation map; a Screen Inventory row set covering both EPIC-002 (Payer Management) and EPIC-003 (Invoice Management); mandatory, cited Surface/UI Pattern/Matched Reference fields on every row (no generic labels); an overlay screen (Dialog) correctly distinguished from a routed Page; a fully completed Per-Screen Data Requirements section and Verification checklist; and journey citations that cite a specific numbered step within a BRD journey (e.g. "Journey 4.1, step 1"), not an invented sub-heading.

---

# Screen Design — Invoice Management System

**Project:** HARBORVIEW-INV-001
**Status:** Approved
**BRD source:** `projects/HARBORVIEW-INV-001/brds/brd-core-v1.md` (v1.0, Approved)

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | 2026-07-10 | Ascendra AI | Initial version |

---

## Part 1 — Template

### 1. Portals

| Portal | Personas | Access model | Base route |
|---|---|---|---|
| Staff App | Finance Manager, Account Owner | Standing session | `/app` |

### 2. Navigation Map

**Staff App** — flat (no distinct feature-area grouping yet at this scale):

| Nav item | Portal | Visibility condition | Grouping |
|---|---|---|---|
| Dashboard | Staff App | baseline — always visible | flat |
| Payers | Staff App | baseline — always visible | flat |
| Invoices | Staff App | baseline — always visible | flat |

### 3. Screen Inventory

> Journey citations reference a specific numbered step within a BRD Section 4 journey (e.g. "Journey 4.1, step 1" — the first numbered step under `### 4.1`), since BRD journeys number their steps as a plain list, not as sub-headings. There is no `4.1.1` heading anywhere in the BRD — never cite one.

| Screen ID | Portal | Route | Reachable by | Purpose | Source | Surface | UI Pattern | Matched Reference | Notes |
|---|---|---|---|---|---|---|---|---|---|
| SCR-001 | Staff App | `/app/payers` | baseline | Browse all Payer records | Journey 4.1, step 1, REQ-010 | Page | Table-List | Data Table + Empty State | Empty State candidate — new account |
| SCR-002 | Staff App | overlay on SCR-001 | baseline | Create or edit a Payer record | Journey 4.1, step 1 (Payer must exist before selection), REQ-011 | Sheet | Form | closest: Customer Profile | |
| SCR-003 | Staff App | `/app/invoices` | baseline | Browse all invoices | Journey 4.1, step 4 (post-save landing), REQ-020 | Page | Table-List | Data Table + Page Bar | Empty State candidate — new account |
| SCR-004 | Staff App | `/app/invoices/new` | baseline | Draft and send a new invoice against an existing Payer | Journey 4.1, steps 1–5, REQ-021/022 | Page | Form (Complex) | closest: Create Product Listing (multi-section, mixed grid) | Line-item totals computed inline |
| SCR-005 | Staff App | `/app/invoices/{id}` | baseline | Invoice detail — line items, status, totals | Journey 4.1, step 7; Journey 4.2, steps 6–9 (approval status visible here too), REQ-023 | Page | Detail | Item + Card | |

### 4. Per-Screen Data Requirements

| Screen ID | Fields shown/collected | Actions available | Notes |
|---|---|---|---|
| SCR-001 | Payer name, billing contact, payment terms, invoice count | Filter, sort, open detail, create new | |
| SCR-002 | Company name, billing contact name/email, payment terms | Save | REQ-011 payment terms required |
| SCR-003 | Invoice #, Payer, Amount, Status, Due date | Filter, sort, open detail, create new | |
| SCR-004 | Payer (select existing), line items (description, qty, unit price), due date | Save draft, send | Tax and total computed from line items |
| SCR-005 | Invoice #, status, Payer, line items, totals, sent/draft timestamps | Edit (if Draft), send | Only Draft invoices are editable |

---

## Part 2 — Verification

- [x] Every journey in BRD Section 4 maps to at least one Screen ID in Section 3 — 4.1 and 4.2 both covered
- [x] Every Portal in Section 1 is a genuinely distinct navigation shell — one portal, matching the single Staff persona group at this scale
- [x] Every nav item in Section 2 has a matching Portal and a stated visibility condition
- [x] Every Screen Inventory row has a Surface, a UI Pattern, and a Matched Reference — no row left blank or generic
- [x] Every Matched Reference names a real, existing `ascendra-ui` catalog entry or primitive composition — verified against `ui-reference.md` Part 2 (Forms/Dialogs/Sheets/Drawers) and Part 1 (Table primitives)
- [x] Tabs are proposed only where the source genuinely describes 2+ independent sub-views — not used in this example, no screen needed them
- [x] Every screen in Section 3 has a corresponding row in Section 4 (Data Requirements)
- [x] No Command Palette or Toast entries appear as per-screen Surface/Pattern matches
