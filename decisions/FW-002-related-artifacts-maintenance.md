# FW-002 — Related Artifacts Table (brief.md Section 9) Maintenance Strategy

| Field | Value |
|-------|-------|
| **ID** | FW-002 |
| **Date** | 2026-06-27 |
| **Status** | Decided — partially implemented. `/gen-domain-knowledge` adds its Section 9 row (verified 2026-07-11). `/gen-brd`, `/gen-architecture`, `/gen-sprint-plan`, `/gen-release-notes`, and `/update-status` do not yet — pending their Batch 2 command-upgrade passes. |
| **Area** | `brief.md` Section 9, all artifact-creating commands, `/update-status` |

---

## Decision

Section 9 of `brief.md` is maintained using a **combination strategy**:

1. **Creating commands add the row** — each command that creates a Section 9 artifact adds a row to Section 9 of `brief.md` immediately after writing the artifact file. Initial status is set to the artifact's starting status (e.g. `Draft`).

2. **`/update-status` updates the status column** — when `/update-status` touches a Section 9 artifact, it also finds and updates that row's Status column in `brief.md` Section 9. For artifact types not in the curated list, it does nothing extra.

3. **Manual always available** — the PO can edit Section 9 directly at any time.

---

## Curated artifact list — what appears in Section 9

Section 9 is a navigation index for major deliverables, not a file listing. Maximum ~8–10 rows for a well-progressed project.

| Artifact | In Section 9 | Added by | Status updated by |
|----------|-------------|----------|------------------|
| Project Brief | ✅ | `/run-intake` (Step 7) ✅ already implemented | `/update-status` |
| Domain Knowledge | ✅ | `/gen-domain-knowledge` | Manual (no lifecycle status) |
| BRD | ✅ | `/gen-brd` | `/update-status` |
| Architecture | ✅ | `/gen-architecture` | `/update-status` |
| Sprint Plans | ✅ | `/gen-sprint-plan` (one row per sprint) | `/update-status` |
| Release Notes | ✅ | `/gen-release-notes` (one row per release) | Manual (no lifecycle status) |
| Epics index | ✅ | `/gen-epics` (one row for `epics/index.md`, not per-epic) | Manual (no single lifecycle status — see epics index itself) |
| Discovery State | ❌ | — | Working document |
| Playbook | ❌ | — | Internal working document |
| Individual epics | ❌ | — | Too granular — `epics/index.md` covers this |
| Individual stories | ❌ | — | Too granular — `stories/index.md` covers this |
| UAT Checklists | ❌ | — | Sprint-level working document |
| Test Execution Reports | ❌ | — | Story-level working document |
| PR Descriptions | ❌ | — | Ephemeral |

---

## Why creating commands, not `/update-status`, add the row

If `/update-status` added the row, Section 9 would only show an artifact after its first status change — not at creation. A BRD in `Draft` state would be invisible in Section 9 until the PO ran `update-status`. The row must appear as soon as the artifact exists, which is at creation time.

---

## Implementation — what each command needs

### Creating commands — add row to Section 9

After writing the artifact file, each command checks if `projects/{PROJECT_CODE}/brief.md` exists. If it does, append a row to the Section 9 table:

```markdown
| {Artifact label} | `{artifact path}` | {initial status} |
```

If `brief.md` does not exist (e.g. command run without init-project), skip silently — do not fail.

Commands requiring this addition:
- `/gen-domain-knowledge` — adds Domain Knowledge row, no lifecycle status → use `Active`
- `/gen-brd` — adds BRD v1 row, initial status `Draft`
- `/gen-architecture` — adds Architecture v1 row, initial status `Draft`
- `/gen-sprint-plan` — adds Sprint N row, initial status `Planning`
- `/gen-release-notes` — adds Release vX.X.X row, no lifecycle status → use `Published`
- `/gen-epics` — adds one Epics index row (`epics/index.md`), no lifecycle status → use `Active` (confirmed already implemented — found during 2026-07-13 audit, this row was missing from the curated list even though `/gen-epics` already wrote it correctly)

### `/update-status` — update status column in Section 9

For BRD, Architecture, and Sprint Plan updates only, after updating the artifact file, also:
1. Read `brief.md`
2. Find the Section 9 table row whose Location matches the artifact path
3. Update its Status column value to the new status
4. If no matching row found, skip silently (do not fail — the PO may have removed it deliberately)

---

## Section 9 ordering

Brief is always first. All other artifacts appear in the order they were added (chronological creation order). Do not sort or reorder.
