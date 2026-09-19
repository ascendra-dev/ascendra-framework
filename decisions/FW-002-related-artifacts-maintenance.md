# FW-002 — Related Artifacts Table (brief.md Section 9) Maintenance Strategy

| Field | Value |
|-------|-------|
| **ID** | FW-002 |
| **Date** | 2026-06-27 |
| **Status** | Decided — fully implemented 2026-09-19. All seven creating commands (`/run-intake`, `/gen-domain-knowledge`, `/gen-brd`, `/gen-architecture`, `/gen-sprint-plan`, `/gen-release-notes`, `/gen-epics`) add their Section 9 row; `/update-status` syncs the Status column for BRD, Architecture, and Sprint Plan. Location values are real markdown links (`conventions/command-conventions.md` C-043), not plain text, per that rule's later addition. |
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
| Project Brief | ✅ | `/run-intake` (Step 7) | N/A — the brief's own Status field is separate from its Section 9 self-row; no command syncs this row (corrected 2026-09-19 — this cell previously claimed `/update-status`, but the Implementation section below never actually specified that, and none was built) |
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

## Implementation — status as of 2026-09-19

### Creating commands — add row to Section 9

After writing the artifact file, each command checks if `projects/{PROJECT_CODE}/brief.md` exists. If it does, append a row to the Section 9 table:

```markdown
| {Artifact label} | [`{artifact path}`]({artifact path}) | {initial status} |
```

The Location column is a real markdown link, relative to `brief.md`'s own location — never plain backtick text (`conventions/command-conventions.md` C-043, added after this decision's original text; this file's row format is corrected to match). If `brief.md` does not exist (e.g. command run without init-project), skip silently — do not fail.

All six commands below are built and self-checked:
- [x] `/gen-domain-knowledge` — adds Domain Knowledge row, no lifecycle status → use `Active`
- [x] `/gen-brd` — adds BRD v{N} row, initial status `Draft`; on a revision, updates the existing row in place rather than duplicating
- [x] `/gen-architecture` — adds Architecture v1 row, initial status `Draft`
- [x] `/gen-sprint-plan` — adds Sprint {NN} row, initial status `Planning`; one row per sprint, never overwritten by a later sprint
- [x] `/gen-release-notes` — adds Release v{version} row, no lifecycle status → use `Published`; one row per release
- [x] `/gen-epics` — adds one Epics index row (`epics/index.md`), no lifecycle status → use `Active` (confirmed already implemented — found during 2026-07-13 audit, this row was missing from the curated list even though `/gen-epics` already wrote it correctly)

### `/update-status` — update status column in Section 9

**Built.** For BRD, Architecture, and Sprint Plan updates only, after updating the artifact file, also:
1. Read `brief.md`
2. Find the Section 9 table row whose Location matches the artifact path
3. Update its Status column value to the new status
4. If no matching row found, skip silently (do not fail — the PO may have removed it deliberately)

Domain Knowledge, Release Notes, and Epics index rows are never synced by `/update-status` — each carries no lifecycle status of its own (`Active`/`Published` is fixed at creation), matching the curated table above.

---

## Section 9 ordering

Brief is always first. All other artifacts appear in the order they were added (chronological creation order). Do not sort or reorder.
