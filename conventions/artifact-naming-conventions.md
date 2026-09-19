# Artifact Naming Conventions

Authoritative, consolidated reference for every ID format and generated-file naming pattern in the Ascendra framework. Each pattern is still documented locally where it's used (a template's own "File location" AI Guide line, per [`template-conventions.md`](template-conventions.md) T-001, and the generating command's Output path) — this file exists so those scattered copies have one place to be checked against, not to replace them. When adding a new artifact type, register its pattern here and confirm it doesn't collide with an existing one.

**Governing principle:** the project code never appears inside an ID or a generated filename. `projects/{PROJECT_CODE}/...` already disambiguates which project an artifact belongs to — repeating it inside the ID itself (e.g. a rejected `PRJ-{PROJECT_CODE}-US-...` form once present in `story.template.md`) is redundant and never actually used anywhere in the framework.

---

## Section 1 — ID Formats

IDs used *inside* documents (headers, cross-references) — distinct from filenames, Section 2.

| ID | Format | Scope / reset point | Owning template |
|----|--------|---------------------|-----------------|
| Project Code | `CLIENT-DOMAIN-NNN` (uppercase, hyphenated, 3-digit trailing sequence) | Global — set once at `/init-project` | `init-project.md` |
| Epic ID | `EPIC-{NNN}` (3-digit zero-padded) | Per project | `epic.template.md` |
| Story ID | `US-{epic}-{seq}` (epic: 2-digit zero-padded; seq: 3-digit, resets to 001 per epic) | Per epic — **never per sprint**. Sprint membership isn't known at generation time (decided later, in `/review-stories`'s Sprint Assignment step) and can change after generation (replanning, `/assess-change`) — encoding it in the ID would force a rename every time | `story.template.md` |
| REQ ID | `REQ-{NNN}` (3-digit, sequential across the whole BRD) | Per BRD document | `brd.template.md` |
| Defect ID | `DEF-{n}` (sequential within one story's `/verify-story` report only) | Per story — **not** a project-wide sequence. When aggregated across stories (e.g. in a test execution report), qualify with Story ID to disambiguate — there is no persistent, cross-story defect log | `test-execution-report.template.md` |

---

## Section 2 — Generated File Naming Patterns

All paths are relative to `projects/{PROJECT_CODE}/`.

| Artifact | Filename pattern | Folder | Scope | Owning command | Owning template |
|----------|-----------------|--------|-------|-----------------|------------------|
| Brief | `brief.md` | (root) | Once per project | `/init-project` | `brief.template.md` |
| Domain discovery state | `domain-discovery-state.md` | `domain/` | Once per project | `/run-domain-discovery` | `domain-discovery-state.template.md` |
| Domain knowledge (base) | `{domain-slug}-core.md` | `domain/` | Once per project | `/gen-domain-knowledge` | `domain.template.md` |
| Domain knowledge (extension) | `{domain-slug}-{country-code}.md` / `{domain-slug}-{sector-slug}.md` | `domain/` | Per extension dimension | `/gen-domain-knowledge` | `domain.template.md` |
| Domain playbook | `{domain-slug}-domain-playbook.md` | `domain/` | Once per project | `/gen-domain-playbook` | `domain-playbook.template.md` |
| BRD discovery state | `brd-discovery-state.md` | `brds/` | Once per project | `/run-brd-discovery` | `brd-discovery-state.template.md` |
| BRD playbook | `{domain-slug}-brd-playbook.md` | `brds/` | Once per project | `/gen-brd-playbook` | `brd-playbook.template.md` |
| BRD | `brd-core-v{N}.md` — filename **increments** on every revision (`v1` → `v2`), per `FW-001` | `brds/` | Once per project, new file per version | `/gen-brd` | `brd.template.md` |
| Screen Design | `screen-design.md` | `screens/` | Once per project | `/gen-screen-design` | `screen-design.template.md` |
| UI mock | `{portal-slug}-mock.html` | `mocks/` | Per portal | `/gen-ui-mocks` | `ui-mock.template.md` |
| Epic | `EPIC-{NNN}-{slug}.md` | `epics/` | Per epic | `/gen-epics` | `epic.template.md` |
| Epics index | `index.md` (fixed name) | `epics/` | Once per project — registry + REQ coverage map + dependency graph | `/gen-epics` | — (no separate template; structure defined inline in the command) |
| Epics review record | `review-record.md` (fixed name) | `epics/` | Once per project — resumable review-session tracking, not a deliverable artifact | `/review-epics` | — (no separate template) |
| Architecture | `arch-v1.md` (+ `arch-v1-ref.md` companion) — filename **stays fixed**; revisions tracked via Document Control/Change History only, per `FW-001` | `architecture/` | Once per project | `/gen-architecture` | `arch.template.md` |
| Standard (system architecture) | `system-architecture.md` | `standards/` | Once per project | `/gen-architecture` | `system-architecture.template.md` |
| Standard (technical / pattern) | `{standard-name}.md` / `{pattern-name}.md` | `standards/` | Per standard | `/gen-architecture` | `technical-standard.template.md`, `architectural-pattern.template.md` |
| Story | `US-{epic}-{seq}-{slug}.md` | `stories/EPIC-{NNN}/` | Per story | `/gen-stories` | `story.template.md` |
| Stories index | `index.md` (fixed name) | `stories/` | Once per project — registry + Sprint Planning table (Section 8, written by `/review-stories`'s Sprint Assignment step) | `/gen-stories` (created/updated) | — (no separate template; structure defined inline in the command) |
| Stories review record | `review-record.md` (fixed name) | `stories/` | Once per project — resumable review-session tracking, not a deliverable artifact | `/review-stories` | — (no separate template) |
| Sprint plan | `sprint-{NN}.md` | `sprints/` | Per sprint | `/gen-sprint-plan` | `sprint-plan.template.md` |
| UAT checklist | `sprint-{NN}-uat-checklist.md` | `sprints/` | Per sprint | `/gen-uat-checklist` | `uat-checklist.template.md` |
| Test execution report | `sprint-{NN}-execution-report.md` | `sprints/` | Per sprint (optional; aggregates every story's `/verify-story` report for that sprint) | Step 23 (manual) | `test-execution-report.template.md` |
| Story verification report | `US-{epic}-{seq}-verification.md` | `test-reports/` | Per story | `/verify-story` | — (derives scenarios from the story's own ACs, no separate template) |
| PR description | `US-{epic}-{seq}-pr-description.md` | `pr/` | Per story (one story = one PR) | `/gen-pr-description` | `pr-description.template.md` |
| Release notes | `v{version}-release-notes.md` | `releases/` | Per version — **not per sprint**; a release doesn't always map 1:1 to a sprint (hotfix release, or one release bundling several sprints) | `/gen-release-notes` | `release-notes.template.md` |

---

## Section 2a — Global (not per-project)

One artifact sits above the per-project scope every row in Section 2 shares — it isn't relative to `projects/{PROJECT_CODE}/` at all.

| Artifact | Filename pattern | Location | Scope | Owning command |
|----------|------------------|----------|-------|-----------------|
| Project registry | `index.md` (fixed name) | `projects/` (repo root level) | Once per framework instance — one row per project (Project Code, Name, Client, Domain, Created, and gate statuses, per `FW-027`) | `/init-project` (creates/appends) |

---

## Section 3 — Scope Keys, Explained

Four different things can key an artifact's filename, and mixing them up is the most common source of drift (see Rationale below):

- **Per story** — `US-{epic}-{seq}` embedded in the filename. Used by: story files themselves, verification reports, PR descriptions.
- **Per sprint** — `sprint-{NN}` embedded in the filename. Used by: sprint plans, UAT checklists, test execution reports. A sprint's *number* is only known after `/review-stories`'s Sprint Assignment step — never bake a sprint number into anything generated before that point.
- **Per version** — `v{version}` embedded in the filename. Used only by release notes, deliberately decoupled from sprint number for the reason in Section 2's Release Notes row.
- **Fixed name, once per project (or once globally)** — no variable key at all; the folder (or, for the project registry, the row) is what disambiguates. Used by: indexes (`epics/index.md`, `stories/index.md`), review-session tracking records (`epics/review-record.md`, `stories/review-record.md`), and the project registry (`projects/index.md`). These are cross-reference/tracking artifacts, not versioned deliverables — there is never more than one live copy, so there's nothing for a filename key to distinguish.

If a new artifact type doesn't obviously fit one of these four, that's a signal to think through its actual scope before picking a pattern — don't default to "per sprint" just because most things are.

---

## Rationale

Built 2026-07-13 (`FW-028`) after three separate naming-drift bugs were found and fixed in one pass: Story IDs were originally keyed by sprint number instead of epic number (forced renames on every replan); `sprint-plan.template.md` and its command disagreed on the output filename (`sprint-{NN}-plan.md` vs `sprint-{NN}.md`); and `story.template.md`'s own Story ID field displayed an unused `PRJ-{PROJECT_CODE}-US-...` long form that contradicted every command and every real generated file. Each was only caught by manually grepping across files — there was no single reference to check against. This file is that reference.
