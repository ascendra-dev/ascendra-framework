# FW-007 — Artifact Ownership Boundaries

| Field | Value |
|-------|-------|
| **ID** | FW-007 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | All commands that create project artifacts |

---

> **Corrected 2026-07-13:** the ownership table below had drifted from actual filenames/commands since this decision was written — `/gen-playbook` was renamed to `/gen-brd-playbook` and its output co-located into `brds/` (`FW-012`, `FW-013`), the domain knowledge filename gained a `-core` suffix, story/test-report IDs became epic-keyed (`FW-028`), and `/gen-pr-description` was added (Batch 2). Table corrected to match current reality; the ownership *rule* itself is unchanged.

---

## Decision

Each artifact is owned by exactly one command — the command that first creates it. No other command may create that artifact. Commands may read any artifact; only the owning command creates it.

### Ownership table

| Artifact | Owning command | Notes |
|----------|---------------|-------|
| `brief.md` | `/init-project` | Created with placeholders; content filled by `/run-intake` |
| `discovery-state.md` | `/run-brd-discovery` | Created on first run if it does not already exist |
| `domain/{slug}-core.md` | `/gen-domain-knowledge` | |
| `brds/{slug}-brd-playbook.md` | `/gen-brd-playbook` | Co-located inside `brds/`, not a separate `playbooks/` folder (`FW-012`) |
| `brds/brd-core-v1.md` | `/gen-brd` | |
| `architecture/arch-v1.md` | `/gen-architecture` | |
| `architecture/arch-ref-v1.md` | `/gen-architecture` | Companion to arch-v1.md, generated in same step |
| `mocks/{portal-slug}-mock.html` | `/gen-ui-mocks` | Visualises architecture Section 3.1.3; reviewed inside `/review-architecture` Concern A, no independent Status/Document Control |
| `epics/EPIC-{NNN}-{slug}.md` | `/gen-epics` | |
| `epics/index.md` | `/gen-epics` | |
| `stories/EPIC-{NNN}/{story}.md` | `/gen-stories` | |
| `stories/index.md` | `/gen-stories` | |
| `sprints/sprint-{NN}.md` | `/gen-sprint-plan` | |
| `sprints/sprint-{NN}-uat-checklist.md` | `/gen-uat-checklist` | |
| `test-reports/US-{epic}-{seq}-verification.md` | `/verify-story` | |
| `releases/v{version}-release-notes.md` | `/gen-release-notes` | |
| `pr/US-{epic}-{seq}-pr-description.md` | `/gen-pr-description` | Command added after this decision was written (Batch 2, 2026-07); ownership rule applies unchanged |

---

## Key boundary: `discovery-state.md`

`discovery-state.md` is owned by `/run-brd-discovery`, not `/init-project`. `/init-project` does not create it. `/run-brd-discovery` checks for its existence on first run and creates it from the template if it does not exist.

**Why:** `discovery-state.md` is the live record of a discovery session. Creating an empty stub in `/init-project` before any discovery has happened creates a misleading document with no content. The file should only exist once discovery begins.

---

## Key boundary: `brief.md` content vs creation

`/init-project` **creates** `brief.md` with header fields pre-filled and section placeholders. `/run-intake` **fills** the section content. These are distinct responsibilities — init-project owns the file, run-intake owns the content.

`/init-project` does not copy `[AI Guide]` notes from the template into `brief.md`. The brief is a clean document.
