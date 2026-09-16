# Init Project

You are creating a new project record in the Ascendra delivery system. This is the first step for every new client project — it creates the directory structure and project brief that all subsequent commands require as context.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain: a project code, project name, client, and domain. Examples:

```
/init-project ASCENDRA-PAY-001 "ASCENDRA PAY — Phase 1: Core" "Ascendra" "Invoice & Payment"
/init-project HARBORVIEW-SCH-001 "Harborview School Fee Management" "Harborview Consulting Ltd" "School Fee Management"
```

Extract:
- **PROJECT_CODE** — uppercase, hyphen-separated, with a trailing 3-digit sequence number. Format: `CLIENT-DOMAIN-NNN` (e.g. `HARBORVIEW-SCH-001`), or `CLIENT-DOMAIN-EXT-NNN` when the project is an extension of another project and the PO wants the extension identity visible in the code itself (e.g. `ASCENDRA-PAY-PKN-001` — the Pakistan extension of an `ASCENDRA-PAY` line). The extension segment is optional and cosmetic — the extension relationship itself is always recorded in the brief's Extension Context fields during `/run-intake` (`FW-016`), never inferred from the code. The sequence number allows a PO to run multiple iterations of the same scope. This becomes the folder name.
  - **Casing:** if the supplied code is not fully uppercase (e.g. `harborview-sch-001` or `Harborview-Sch-001`), silently uppercase it — this is a trivial correction, not an error. The corrected value is what appears in the Step 3 confirmation display, so the PO sees and can catch it there before anything is created.
  - **Structural validity:** the code must resolve to `CLIENT-DOMAIN-NNN` or `CLIENT-DOMAIN-EXT-NNN` — a hyphen-separated code with 3 or 4 segments, ending in a 3-digit sequence number. Anything else (missing the sequence number, no hyphens, 5+ segments) is a real error — do not guess a correction. Stop and ask: "Project code must be in `CLIENT-DOMAIN-NNN` format (e.g. `HARBORVIEW-SCH-001`), optionally `CLIENT-DOMAIN-EXT-NNN` for an extension project. You provided `{as-given}` — what should it be?"
- **Project Name** — full display name (e.g. `Harborview School Fee Management — Phase 1`)
- **Client** — use `Internal` if building for your own organisation (internal team or own product), or the client's full organisation name if building for a third party (e.g. `Harborview Consulting Ltd`)
- **Domain** — the business domain (e.g. `Invoice & Payment`, `School Fee Management`)

If any of these are missing, ask for them all at once:

> "To create a new project I need:
> 1. **Project code** — CLIENT-DOMAIN-NNN format (e.g. `HARBORVIEW-SCH-001`)
> 2. **Project name** — full display name (e.g. `Harborview School Fee Management — Phase 1`)
> 3. **Client** — use `Internal` if building for your own organisation, or the client's full organisation name if building for a third party (e.g. `Harborview Consulting Ltd`)
> 4. **Domain** — the business domain (e.g. `School Fee Management`, `Invoice & Payment`)"

---

## Step 2 — Confirm the project boundary

Before creating anything, ask:

> "In one sentence, what is this project — what product or capability does it deliver, and who uses it?"

Wait for the PO's answer.

If the answer names more than one distinct product or user-facing capability, respond:

> "That sounds like more than one product. A web app and its API are components of one product and belong in the same project. But separate portals for different user groups — or tools with distinct purposes — are separate Ascendra projects, each with its own brief and lifecycle. Which product do you want to start with?"

Wait for clarification. Do not proceed until the answer describes one product or capability. Note: an API + web app for the same product is **one project** — the boundary is about user-facing purpose, not technical components.

Note the deliverable description — you will include it in the Step 2 confirmation display.

---

## Step 3 — Confirm with the PO

Before creating anything, present the parsed values and wait for confirmation:

```
INIT PROJECT — {PROJECT_CODE}
─────────────────────────────────────────
Project Code:   {PROJECT_CODE}
                └─ Folder: projects/{PROJECT_CODE}/

Project Name:   {Project Name}

Client:         {Client}
                └─ "Internal" = building for your own organisation.
                   Client name = building for a third party.

Domain:         {Domain}
─────────────────────────────────────────
Confirm to proceed, or correct any field before the project is created.
```

Wait for the PO to confirm or provide corrections. If corrections are provided, update the values and re-display before proceeding. Do not create any files until the PO confirms.

---

## Step 4 — Read the required templates

Read this file in full before creating anything:

1. `projects/TEMPLATE/brief.template.md` — note the section structure and heading names. You will pre-fill known fields and insert placeholders for all section bodies.

---

## Step 5 — Check whether the project already exists

Check if `projects/{PROJECT_CODE}/` already exists.

**If it does not exist:** create the directory and proceed to Step 5.

**If it does exist**, ask:

> "A project directory for `{PROJECT_CODE}` already exists at `projects/{PROJECT_CODE}/`. Choose an option:
> **(a) Cancel** — keep the existing project untouched. No changes will be made.
> **(b) Add missing files only** — create any files that do not exist yet. Existing files are not touched. Use this to recover from an interrupted init run.
> **(c) Reinitialise** — overwrite `brief.md` with fresh content. Other project files — including `discovery-state.md`, BRDs, stories, and architecture — are not affected."

Wait for the PO's choice before proceeding.

- **(a) Cancel:** Report: "Cancelled. No changes made to `projects/{PROJECT_CODE}/`." Stop here.
- **(b) Add missing files only:** Proceed through Steps 5–8. For each file, check if it already exists before creating — skip files that exist. At Step 9 report which files were created and which were already present.
- **(c) Reinitialise:** Warn first: "This will overwrite `brief.md`. Any content already in it will be lost. Other project files — including `discovery-state.md`, BRDs, stories, and architecture — are not affected. Confirm?" Wait for confirmation, then proceed through Steps 5–7, overwriting that file regardless of whether it exists.

---

## Step 6 — Create the project brief

Create `projects/{PROJECT_CODE}/brief.md` using `projects/TEMPLATE/brief.template.md` as the structure.

Pre-fill the header fields from the confirmed arguments:

| Field | Value |
|-------|-------|
| Project Name | From arguments |
| Project Code | From arguments |
| Client | From arguments |
| Domain | From arguments |
| Created | Today's date |
| Last Updated | Today's date |
| Status | `Active` |
| Content Status | `Draft` |

These are two distinct fields — **Status** is the project's overall lifecycle (stays `Active` through virtually the whole project); **Content Status** is this document's own readiness (`Draft` now, progressing through `/run-intake` and approval). See the brief template's own AI Guide note on this distinction. Do not set both to the same value.

Add the following note below the header fields:

```
> **Recommended:** Run `/run-intake {PROJECT_CODE}` to complete this brief interactively — it guides you section by section, validates your answers, and writes each section on approval.
> **Alternatively:** Fill in each section manually by following the `[AI Guide]` notes in `projects/TEMPLATE/brief.template.md` and referring to the worked example in `projects/TEMPLATE/brief.example.md`.
```

For every section body (Sections 1–9), insert the following placeholder instead of leaving it empty:

```
> *Awaiting intake session. **Recommended:** Run `/run-intake {PROJECT_CODE}` to complete this section interactively. **Alternatively:** fill it manually by following the `[AI Guide]` notes in `projects/TEMPLATE/brief.template.md` and referring to `projects/TEMPLATE/brief.example.md`.*
```

Do not copy `[AI Guide]` notes from the template into `brief.md`. The brief is a clean document — section headings, pre-filled header fields, and placeholders only.

---

## Step 7 — Create the project folder skeleton

Create the following subfolders inside `projects/{PROJECT_CODE}/`. These are created empty now and populated by subsequent commands as the project advances.

```
projects/{PROJECT_CODE}/
├── source-material/ # Raw reference material the PO drops here before/during
│                    # /run-priming-session (an SRS, notes, anything) — empty
│                    # until the PO puts something in it; nothing reads it
│                    # automatically
├── priming/         # Generated by /run-priming-session — priming-package.md
├── domain/          # Domain knowledge, domain playbook, domain discovery state
├── standards/       # Project-specific tech standards — written by /gen-architecture
├── brds/            # BRD, BRD playbook, BRD discovery state
├── epics/           # Generated by /gen-epics
├── screens/         # Generated by /gen-screen-design
├── stories/         # Generated by /gen-stories
├── story-plans/     # Generated by /gen-story-plan
├── architecture/    # Generated by /gen-architecture
├── mocks/           # Generated by /gen-ui-mocks (base pass) and patched by /gen-architecture
├── sprints/         # Generated by /gen-sprint-plan and /gen-uat-checklist
├── test-reports/    # Generated by /verify-story
├── pr/              # Generated by /gen-pr-description
└── releases/        # Generated by /gen-release-notes
```

Project identity (code, name, client, domain, created date) lives only in `projects/index.md` (Step 8 below) — per `FW-027`, no separate `project.json` is created. Extension relationships are captured in the brief's Extension Context fields during `/run-intake`. Repository topology and agent assignments are defined in the architecture document.

---

## Step 8 — Register the project in the index

Check if `projects/index.md` exists.

- **If it exists:** append a new row to the projects table for this project.
- **If it does not exist:** create `projects/index.md` with the following structure and add this project as the first row:

```markdown
# Project Index

| Project Code | Project Name | Client | Domain | Created | Status | Brief | BRD | Architecture | Sprints |
|-------------|-------------|--------|--------|---------|--------|-------|-----|--------------|---------|
| {PROJECT_CODE} | {Project Name} | {Client} | {Domain} | {today} | Active | Draft | — | — | — |
```

The index is the at-a-glance registry of all projects. One row per project. Status columns are updated by `update-status` as the project advances.

---

## Step 9 — Report to the user

```
PROJECT CREATED — {PROJECT_CODE}
─────────────────────────────────────────
Project Name:   {Project Name}
Client:         {Client}
Domain:         {Domain}

Files created:
  projects/{PROJECT_CODE}/brief.md

Folders scaffolded:
  domain/  standards/  brds/  epics/  screens/
  stories/  architecture/  mocks/  sprints/  test-reports/  pr/  releases/

Registry updated:
  projects/index.md
─────────────────────────────────────────
```

Then tell the user:

> **Next steps:**
> 1. Complete the brief — **recommended:** run `/run-intake {PROJECT_CODE}` for a guided session that validates and writes each section. **Alternatively:** fill `projects/{PROJECT_CODE}/brief.md` manually following the `[AI Guide]` notes in `projects/TEMPLATE/brief.template.md` and the worked example in `projects/TEMPLATE/brief.example.md`.
> 2. Once the brief is Approved, run `/gen-domain-knowledge` to generate the domain knowledge base — required for all projects.
> 3. **Optional — niche or complex domains only:** run `/gen-domain-playbook` then `/run-domain-discovery` to deepen domain knowledge before BRD discovery. Skip this step for well-known domains.
> 4. Run `/gen-brd-playbook` to generate the BRD discovery playbook and move to requirement discovery.
