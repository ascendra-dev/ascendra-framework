# Ascendra Framework — AI-Native Software Delivery

## What This Is

**Ascendra Framework** is a PO-led software delivery framework that uses AI as an execution partner. A single skilled PO — someone who can think across BA, SA, architecture, engineering, and QA — works alongside Claude to take a client project from brief to production.

The framework is not a running platform. It is a structured set of documents, templates, standards, and slash commands that shape how discovery, design, and delivery are done. The AI does the heavy lifting inside each phase. The PO reviews, directs, and approves at every gate.

Tagline: **AI-Native Software Delivery**

---

## Who This Is For

A skilled delivery professional who can already think across business analysis, solution architecture, engineering, and QA. Specifically:

- Has delivered software projects end to end, including client-facing discovery sessions
- Can evaluate AI-generated artifacts and redirect the AI when output is wrong or shallow
- Understands what a domain model is, what a BRD is, and what an architecture decision is
- Has run at least one client requirement discovery session before

**The framework is a force multiplier — it amplifies existing capability. It does not teach delivery.** A practitioner who has not yet built cross-functional delivery experience will produce faster output of the wrong kind.

One skilled PO can run multiple projects in parallel using this framework.

---

## Core Philosophy

- One PO + AI beats a large team doing handoffs
- Every document produced by AI is reviewed before the next step starts
- Ubiquitous Language is enforced from day one — every term in code, docs, and conversation comes from the domain glossary
- The framework is technology-agnostic — each project defines its own tech stack in `projects/{CODE}/standards/`
- Generated artifacts (domain knowledge, playbooks, BRDs, epics, stories, architecture) live in `projects/{CODE}/`, not in the framework
- A **project** is one user-facing product or capability — one thing a user or system interacts with to accomplish a goal. A product may have multiple technical components (API, web app, mobile app, workers); those are defined in the architecture, not counted as separate projects. The boundary is at the **user-facing purpose level**: a teacher portal and a parent portal are two projects; a web app and its API are one project.

---

## Workspace Setup

1. Clone this repo: `git clone https://github.com/zakashah/ascendra-framework`
2. Open the repo root as your primary Claude Code session directory (commands only load from the active root)
3. Create a project: run `/init-project` — it creates `projects/{CODE}/` with all required folders and stubs
4. Project identity (code, name, client, domain, created date) is recorded in `projects/index.md`, not a per-project file. Extension relationships are captured in the brief during `/run-intake`. Repository topology is defined in the architecture document.
5. Add project-specific tech standards to `projects/{CODE}/standards/` before running `/gen-architecture`

---

## Folder Structure

```
ascendra-framework/
├── .claude/commands/       # 31 slash commands — the framework's executable interface
├── SDLC.md                 # Conceptual stage map — the twelve delivery stages
├── PROJECT-LIFECYCLE.md    # Operational command workflow — every step, command, gate
├── conventions/            # Framework-authoring meta-rules — command, template, and artifact-naming conventions
├── reference/              # Fixed, universal content every project consults directly (estimation models, Core/Extension methodology, defect severity) — not templated or filled in per project
├── decisions/              # Framework-level ADRs — see decisions/index.md for the index
└── projects/
    ├── index.md            # Project registry
    ├── TEMPLATE/           # ⚠ FRAMEWORK DEPENDENCY — do not delete or rename
    │   ├── brief.template.md
    │   ├── brds/
    │   │   ├── brd.template.md
    │   │   ├── brd-playbook.template.md
    │   │   └── brd-discovery-state.template.md
    │   ├── domain/
    │   │   ├── domain.template.md
    │   │   ├── domain-playbook.template.md
    │   │   └── domain-discovery-state.template.md
    │   ├── standards/
    │   │   ├── system-architecture.template.md
    │   │   ├── technical-standard.template.md
    │   │   └── architectural-pattern.template.md
    │   ├── epics/
    │   │   └── epic.template.md
    │   ├── screens/
    │   │   └── screen-design.template.md
    │   ├── stories/
    │   │   └── story.template.md
    │   ├── story-plans/
    │   │   └── story-plan.template.md
    │   ├── architecture/
    │   │   └── arch.template.md
    │   ├── mocks/
    │   │   ├── ui-mock.template.md
    │   │   └── ui-mock.example.md
    │   ├── sprints/
    │   │   ├── sprint-plan.template.md
    │   │   └── uat-checklist.template.md
    │   ├── test-reports/
    │   │   └── test-execution-report.template.md
    │   ├── pr/
    │   │   └── pr-description.template.md
    │   └── releases/
    │       └── release-notes.template.md
    └── {PROJECT_CODE}/     # ⚠ GITIGNORED — your private client project workspace
        ├── brief.md
        ├── domain/              # Generated: domain knowledge, playbooks, discovery state
        ├── standards/           # User-defined: tech stack, API, DB, security, test strategy
        ├── brds/                # Generated: BRD versions
        ├── screens/             # Generated: screen design (Portals, Nav Map, Screen Inventory)
        ├── architecture/        # Generated: architecture docs
        ├── mocks/               # Generated: UI mocks (HTML, one file per portal; base pass + architecture patch)
        ├── epics/               # Generated: epic files + index
        ├── stories/             # Generated: story files + index
        ├── story-plans/         # Generated: PO-confirmed implementation plans, one per story (FW-031)
        ├── test-reports/        # Generated: story verification reports
        ├── pr/                  # Generated: PR descriptions, one per story
        ├── sprints/             # Generated: sprint plans + UAT checklists
        └── releases/            # Generated: release notes
```

---

## Slash Commands

| Command | What It Does |
|---------|--------------|
| `/init-project` | Create a new project folder with all required subfolders |
| `/run-intake` | Complete the project brief interactively with the Product Owner |
| `/gen-domain-knowledge` | Generate domain knowledge docs from a discovery session |
| `/gen-domain-playbook` | Generate a discovery playbook for a domain |
| `/run-domain-discovery` | Run a live domain discovery session |
| `/run-mock-discovery` | Run a mock discovery session to validate the domain playbook |
| `/gen-brd-playbook` | Generate a BRD discovery playbook from the domain knowledge |
| `/run-brd-discovery` | Run a live BRD requirements discovery session |
| `/gen-brd` | Generate a Business Requirements Document |
| `/review-brd` | Review and approve the BRD before epic generation |
| `/gen-epics` | Generate epics from an approved BRD |
| `/review-epics` | Review all epics against the process before story generation |
| `/gen-screen-design` | Generate Screen Design (Portals, Nav Map, Screen Inventory) from the approved BRD, pattern-matched against `ascendra-ui` |
| `/review-screen-design` | Review and approve Screen Design before architecture generation begins |
| `/gen-stories` | Generate user stories for an approved epic |
| `/review-stories` | Review all stories in an epic before sprint lock |
| `/gen-story-plan` | Generate a PO-confirmable implementation plan for one story — the task list `/implement-story` requires before writing any code |
| `/gen-architecture` | Generate an architecture document |
| `/gen-ui-mocks` | Generate persistent HTML UI mocks — base pass from Screen Design, patched by `/gen-architecture` with gating and cross-cutting content |
| `/review-architecture` | Review and lock the architecture document |
| `/gen-sprint-plan` | Generate a sprint plan |
| `/gen-uat-checklist` | Generate a UAT checklist |
| `/implement-story` | Implement a user story in code |
| `/verify-story` | Verify a story implementation against its acceptance criteria |
| `/gen-pr-description` | Generate a PR description from a story's verification report and real code diff |
| `/close-sprint` | Fill a sprint's retrospective, calculate velocity, and identify carry-forward items after UAT sign-off |
| `/gen-release-notes` | Generate release notes |
| `/assess-change` | Assess the impact of a requested scope or architecture change |
| `/update-status` | Update the status of a project artifact |
| `/project-status` | Live dashboard: epic/story status, per-sprint progress, and an ATTENTION section flagging blockers — run at any time |
| `/judgment-check` | Supplementary density/judgment-quality check against one artifact, sourced from the matching `practitioner-guide/` chapter — run before that artifact's `/review-X` walkthrough. Piloted on BRD, Epics, and Architecture; since extended to all remaining artifact types (brief, domain knowledge, screen design, stories/sprint plan, story plan/PR description, UAT checklist/test execution report, release notes). |

---

## Problem Domain / Solution Domain

The lifecycle splits into three regions: **Problem Domain** (Brief, Domain Knowledge, BRD, Epics — technology-agnostic, no architecture dependency), the **Architecture bridge** (first hard dependency point in the gate chain), and **Solution Domain** (Stories through Deploy/Support — everything gated on Locked architecture, directly or transitively). Stories sit in Solution Domain despite being PO-authored in chat mode, because `/gen-stories` hard-gates on Locked architecture and stories carry architecture-derived metadata — authorship mode and domain are independent axes. Full reasoning and the phase-by-phase boundary test: `decisions/FW-022-problem-solution-domain-boundary.md`.

---

## Ubiquitous Language

Every project maintains a domain glossary in `projects/{CODE}/domain/`. Once terms are defined in that document, they become binding across all artifacts — code, database schema, API endpoint names, documentation, and conversation with Claude.

**Rule:** If a term in a story, BRD, or command prompt does not match the domain glossary, correct it before proceeding. Never let synonyms drift in.

**Documented exceptions are valid.** If a term's glossary entry says "also called X in Y context" (e.g. a PSP-industry term for an external API), use the exception term only in that specific context and the canonical term everywhere else.

---

## Working Convention

- Execute one step at a time and pause for review before the next step
- Each new concept is explained when it first appears
- Manual steps (account creation, installs, external API setup) are flagged explicitly and execution pauses
- No Python — TypeScript throughout

---

## Project Registry

`projects/index.md` is the registry — one row per project (Project Code, Project Name, Client, Domain, Created, Status, and gate statuses). This is the single source of project identity — per `FW-027`, there is no separate `project.json` file. `projects/TEMPLATE/` is a committed framework dependency; all other project folders are gitignored (client work stays private). To list active projects, run `ls projects/`.

Repository paths are an architecture decision, not an identity field: `/implement-story` resolves the target repo from the architecture document's Section 3.1 (Project Structure) and applies convention-over-configuration — each confirmed repo is a sibling directory to this framework repo (`../{repo-name}`).

## TEMPLATE Project

`projects/TEMPLATE/` is a hard framework dependency. Each subfolder contains the `.template.md` file for the artifact that folder holds, and every generating command reads its template from here. **Do not delete or rename the TEMPLATE project.**
