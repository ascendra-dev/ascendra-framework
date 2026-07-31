## Part 3 — Worked Example

Sprint 1 execution for Harborview Consulting Ltd — Invoice Management System. Sprint plan: `sprints/sprint-01.md`.

---

**Project:** Invoice Management System
**Sprint:** Sprint 01
**Execution Date:** 2026-07-15
**BRD Version:** v1.0
**Sprint Plan:** `sprints/sprint-01.md`

---

### 1. Test Summary

| Metric | Value |
|--------|-------|
| Total ACs | 14 |
| Passed | 12 |
| Failed | 1 |
| Blocked | 1 |
| Not Run | 0 |
| Pass Rate | 85.7% |

---

### 2. Story Sign-off

| Story ID | Title | ACs Total | Passed | Failed | Blocked | Decision |
|----------|-------|-----------|--------|--------|---------|----------|
| US-03-001 | Invoice database schema | 2 | 2 | 0 | 0 | Pass |
| US-02-001 | Client database schema and CRUD API | 2 | 2 | 0 | 0 | Pass |
| US-03-005 | Generate invoice reference number | 2 | 2 | 0 | 0 | Pass |
| US-02-002 | Client list and detail UI | 3 | 3 | 0 | 0 | Pass |
| US-03-002 | Create invoice form | 4 | 3 | 1 | 0 | Conditional Pass |
| US-03-003 | Invoice detail page | 1 | 0 | 0 | 1 | Blocked |

---

### 3. Defects Found

| Story | Defect ID | AC # | Description | Severity | Status |
|-------|-----------|------|-------------|----------|--------|
| US-03-002 | DEF-1 | 3 | Inline validation error on the Client field persists on screen after the user selects a valid client — the error message does not clear on input change | Medium | Open |

---

### 4. Blocked Items

| Blocked Item | Story Affected | Reason | Owner | Resolution Target |
|--------------|----------------|--------|-------|-------------------|
| AC 1 — Invoice detail page renders reference number from sequence | US-03-003 | Test data seeding script failed to create invoice records in the test environment due to a missing foreign key in the client table (pre-existing data integrity issue, not a code defect). Test environment must be reset and reseeded before this AC can be verified | Burhan Ali Shah | 2026-07-16 |

---

### 5. QA Sign-off Checklist

- [x] Pass Rate is ≥ 80% — 85.7%
- [x] No Critical defects are Open
- [x] No High defects are Open — DEF-1 (US-03-002) is Medium
- [x] Every blocked item has a named Owner and a Resolution Target date — US-03-003 AC 1 blocked; Owner: Burhan Ali Shah; Resolution: 2026-07-16
- [x] Every story in the sprint has a recorded Decision in Section 2

**Sign-off Recommendation:**

This sprint is ready for UAT — one Medium defect (US-03-002, DEF-1) is logged for awareness, and the single blocked AC (US-03-003, AC 1) has a named owner and a resolution target of 2026-07-16, one day before the UAT session.

---

### 6. Approval

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | Zaka Shah | Approved for UAT | 2026-07-15 |
