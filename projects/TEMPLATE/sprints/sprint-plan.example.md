## Part 3 — Worked Example

Sprint 1 for Harborview Consulting Ltd — Invoice Management System (BRD v1.0).

---

**Project:** Invoice Management System
**Sprint:** Sprint 01
**Status:** Active
**Epic(s):** EPIC-002 — Client Management, EPIC-003 — Invoice Management
**Stories:** 6 total (3S + 3M)
**Dates:** 2026-07-01 → 2026-07-14
**Generated:** 2026-07-01

---

### 1. Sprint Goal

Finance Manager can create invoices against existing client records and view saved drafts — the core invoice creation loop is in place end to end.

---

### 2. Sprint Roadmap

| Epic | Title | Assigned Sprint | Story Count | Status |
|------|-------|-----------------|--------------|--------|
| EPIC-002 | Client Management | 01 | 2 | Active |
| EPIC-003 | Invoice Management | 01 | 4 | Active |
| EPIC-004 | Dunning | Not yet assigned | — | Not yet assigned |
| EPIC-005 | Payment Collection | Not yet assigned | — | Not yet assigned |
| EPIC-011 | Reporting | Not yet assigned | — | Not yet assigned |

Refreshed from `stories/index.md` Section 8 at generation time — see that file for the live source of truth.

---

### 3. Pre-Sprint Readiness Checklist

| Gate | Result |
|------|--------|
| Architecture Locked | Pass |
| Epics Approved | Pass |
| Stories Reviewed | Pass |
| Dependencies cleared | Pass |
| L-story split decision recorded | N/A |
| Capacity decision recorded | N/A |

---

### 4. Story Board

| Order | Story ID | Title | Epic | Size | Status |
|-------|----------|-------|------|------|--------|
| 1 | US-03-001 | Invoice database schema | EPIC-003 | S | Reviewed |
| 2 | US-02-001 | Client database schema and CRUD API | EPIC-002 | S | Reviewed |
| 3 | US-03-005 | Generate invoice reference number | EPIC-003 | S | Reviewed |
| 4 | US-02-002 | Client list and detail UI | EPIC-002 | M | Reviewed |
| 5 | US-03-002 | Create invoice form | EPIC-003 | M | Reviewed |
| 6 | US-03-003 | Invoice detail page | EPIC-003 | M | Reviewed |

---

### 5. Delivery Sequence

Step 1 → US-03-001: establishes the invoice schema
Step 2 → US-02-001: establishes the client schema and CRUD API, independent of Step 1
Step 3 → US-03-005: reference number generation, needs US-03-001's schema
Step 4 → US-02-002: client list/detail UI, needs US-02-001
Step 5 → US-03-002: create invoice form, needs US-03-001, US-03-005, and US-02-001 (client selector)
Step 6 → US-03-003: invoice detail page, needs US-03-001 and US-03-005

---

### 6. Capacity Analysis

| Metric | Value |
|--------|-------|
| Total stories | 6 |
| XS / S / M / L / XL | 0 / 3 / 3 / 0 / 0 |
| Must Have | 6 |
| Should Have | 0 |

---

### 7. Should Have Decisions

None in this sprint.

---

### 8. Definition of Done

1. All acceptance criteria pass verification (`/verify-story`)
2. PR reviewed and merged to main branch
3. No open Critical or High severity defects
4. Story status updated to `Done` in `stories/index.md`
5. No regression in previously merged stories for this project

---

### 9. Sprint Notes

No client records exist in the database when US-03-002 is reviewed — the invoice form's client selector will appear empty, making that AC unverifiable without seed data. Seed 3–5 test client records via the API before submitting US-03-002 for review.

---

### 10. Sprint Retrospective

[Blank at generation — filled by `/close-sprint`]
