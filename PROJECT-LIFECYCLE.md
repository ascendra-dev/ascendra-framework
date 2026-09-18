# Ascendra Project Lifecycle — Command Workflow Reference

## Purpose

This document is the authoritative reference for the Ascendra delivery workflow from first client contact to production release. It defines every step, the slash command that executes it, the template it produces from, the gate that closes it, and the status that records it. Every one of the framework's 33 commands (`ls .claude/commands/`) is accounted for somewhere in this document — see the Completeness Check at the end.

Read this document when: starting a new project, resuming a project mid-delivery, onboarding a new agent to an in-flight project, or verifying that all gates are correctly closed before advancing.

**Relationship to `SDLC.md`:** `SDLC.md` describes the twelve delivery *stages* and the responsible *agents*. This document describes the specific *slash commands* that implement those stages and the exact sequence of execution. They are complementary — read both.

**Relationship to `decisions/FW-022-problem-solution-domain-boundary.md`:** every phase below is explicitly tagged with its domain region. **Problem Domain** (Phases 1–4: Intake, Discovery, Product Structuring, Screen Design) has zero architecture dependency — every artifact in it can be produced with no knowledge of the tech stack. **Bridge** (Phase 5: Architecture) is the first hard dependency point in the gate chain. **Solution Domain** (Phases 6–8: Sprint Delivery, Sprint Closure & Release, Support) requires Locked architecture, directly or transitively, for everything in it. `assess-change`, `update-status`, `project-status`, and `judgment-check` are cross-cutting utilities that operate across all three regions and belong to none of them. `/anchor-project` is different in kind from all four — it doesn't add a utility alongside the sequence below, it drives the sequence itself; see "Running This Sequence via `/anchor-project`" immediately after this table.

**Repeatable / optional steps are called out explicitly** — most of this lifecycle runs once per project, but a few sections repeat (Phase 6 runs once per **wave**, not once per project) or are conditional (Domain Discovery, Mock Discovery, Screen Design are all skippable under stated conditions). See the "Repeats / Optional" column in the table below.

---

## Lifecycle at a Glance

28 numbered steps plus lettered sub-steps for optional or non-linearly-inserted steps (a convention already established at 5a and 9a–9c, extended here to the Discovery phase's optional sub-flow). Table row order is the actual execution order — where a lettered step's position doesn't match its number sequentially (e.g. 2a/2b run *before* Step 2), the table position is authoritative, not the label.

| # | Step | Command | Repeats / Optional | Manual Gate | Template |
|---|------|---------|---------------------|-------------|---------|
| | **▼ PROBLEM DOMAIN — zero architecture dependency (`FW-022`)** | | | | |
| **Phase 1 — Intake** |
| 1 | Create project record | `/init-project` | Once | — | `brief.template.md` |
| 1a | Complete the brief interactively | `/run-intake` | Once (re-run for amendments) | — | `brief.template.md` |
| 1b | Approve the brief | `update-status` (brief → `Approved`) | Once | ✅ PO gate | — |
| **Phase 2 — Discovery** |
| 2a | Generate domain discovery playbook | `/gen-domain-playbook` | **Optional** — niche/complex domains only | — | `domain-playbook.template.md` |
| 2b | Run domain discovery session | `/run-domain-discovery` | **Optional**, follows 2a | — | `domain-discovery-state.template.md` |
| 2 | Generate domain knowledge | `/gen-domain-knowledge` | Once per domain (reusable across projects) | — | `domain.template.md` |
| 3 | Generate BRD discovery playbook | `/gen-brd-playbook` | Once | — | `brd-playbook.template.md` |
| 3a | Run mock discovery session | `/run-mock-discovery` | **Optional but recommended** — validates 2/3 before real client cost | — | — (updates domain knowledge Section 12.6) |
| 4 | Run discovery session → write state → offer BRD | `/run-brd-discovery` | Once | — | `brd-discovery-state.template.md` → `brd.template.md` |
| 5 | Generate BRD (if not done at step 4) | `/gen-brd` | Once | — | `brd.template.md` |
| 5a/6 | Review BRD + approve | `/review-brd` (approves inline on PO typing 'approved') | Once | ✅ PO gate | — |
| **Phase 3 — Product Structuring** |
| 7 | Generate epics | `/gen-epics` | Once (all epics, full BRD scope) | — | `epic.template.md` |
| 8/9 | Review epics + approve | `/review-epics` (approves inline) | Once | ✅ PO gate | — |
| **Phase 4 — Screen Design** |
| 9a | Generate screen design | `/gen-screen-design` | Once — **skipped entirely** if BRD shows no UI in scope | — | `screen-design.template.md` |
| 9b/9c | Review screen design + approve | `/review-screen-design` (approves inline) | Once | ✅ PO gate | — |
| | **▼ BRIDGE — first hard architecture dependency (`FW-022`)** | | | | |
| **Phase 5 — Architecture** |
| 10 | Generate architecture document | `/gen-architecture` | Once (invokes `/gen-ui-mocks` base mode internally) | — | `arch.template.md` |
| 11/12 | Review architecture + lock | `/review-architecture` (locks inline; invokes `/gen-ui-mocks` patch mode) | Once | ✅ PO gate | — |
| | **▼ SOLUTION DOMAIN — everything below requires Locked architecture (`FW-022`)** | | | | |
| **Phase 6 — Sprint Delivery** | *(repeats per **wave**, not per epic 1:1 — see Phase 6 intro)* |
| 13 | Generate stories for a wave | `/gen-stories` | **Repeats per wave** | — | `story.template.md` |
| 14/15 | Review stories + approve + sprint-assign | `/review-stories` (approves inline; Sprint Assignment decides sprint boundary) | **Repeats per wave** | ✅ PO gate | — |
| 16 | Generate sprint plan | `/gen-sprint-plan` | **Repeats per sprint** | — | `sprint-plan.template.md` |
| 17 | Activate sprint | `update-status` (sprint → `Active`) | Repeats per sprint | ✅ PO gate | — |
| | *↳ Steps 17a–21 repeat for every story in the sprint, in delivery sequence order.* | | | | |
| 17a | Generate story implementation plan | `/gen-story-plan` | **Repeats per story** | ✅ PO gate | `story-plan.template.md` |
| 18 | Implement story | `/implement-story` | **Repeats per story** | — | Story plan (Confirmed) → `arch-v1-ref.md` → `arch-v1.md` (law) |
| 19 | Verify story acceptance criteria | `/verify-story` | **Repeats per story** | — | Story plan Section 5 (Test Scenarios) — falls back to deriving from the story's own ACs if no plan exists |
| 20 | Generate PR description + PO reviews PR | `/gen-pr-description` + Manual | **Repeats per story** | ✅ PO gate | `pr-description.template.md` |
| 21 | Update story status | `update-status` (story → `Merged`/`Done`) | Repeats per story | — | — |
| 22 | Handle scope changes mid-sprint | `/assess-change` | **As-needed, cross-cutting** | ✅ PO gate | — |
| **Phase 7 — Sprint Closure & Release** | *(repeats per sprint)* |
| 23 | Sprint-wide QA confirmation | Manual confirmation | Repeats per sprint | — | `test-execution-report.template.md` (optional) |
| 24 | Generate UAT checklist | `/gen-uat-checklist` | Repeats per sprint | — | `uat-checklist.template.md` |
| 25 | PO runs UAT | Manual | Repeats per sprint | ✅ PO gate | — |
| 26 | Fill sprint retrospective | `/close-sprint` (helper) | Repeats per sprint | — | `sprint-plan.template.md` |
| 27 | Close sprint | `update-status` (sprint → `Complete`) | Repeats per sprint | ✅ PO gate | — |
| 28 | Generate release notes | `/gen-release-notes` | Repeats per release (not always 1:1 with sprint) | — | `release-notes.template.md` |
| — | Deploy to production | Manual | Repeats per release | ✅ PO gate | — |
| **Phase 8 — Support** | *(runs once, after the final sprint)* |
| — | Monitor + triage production defects | Manual | Ongoing | — | `defect-severity.md` |
| — | Hotfix (if Critical/High defect) | See Hotfix Cycle section | As-needed | ✅ PO gate per PR + deploy | — |
| — | Close project or hand over | `update-status` (brief → `Complete`) | Once | ✅ PO gate | — |

**Cross-cutting utilities — run at any time, in any phase:**

- `/assess-change` — assesses a described change's blast radius across the document pipeline and applies coordinated updates. Not tied to any one phase; used whenever scope needs to change after a gate has closed.
- `update-status` — the single mechanism that records every gate decision. Referenced throughout the table above.
- `/project-status projects/{PROJECT_CODE}` — live dashboard showing all epic statuses, story completion by status and size, per-sprint progress bars, and an ATTENTION section flagging anything that needs action (stale statuses, unresolved dependencies, L-stories not split, sprints at risk of slipping). Run it to get a situational picture before any gate, at session start, or whenever you need to know where the project stands.
- `/judgment-check {artifact-path}` — supplementary density/judgment-quality check against one already-generated artifact, sourced from the matching `practitioner-guide/` chapter rather than the artifact's own template. Run it before that artifact's `/review-X` walkthrough, not after — it surfaces things worth raising during that review, never a gate on its own. Piloted on BRD, Epics, and Architecture, then extended to every other artifact type this table covers — no type reports "not yet extended" any more.

**`/run-priming-session {PROJECT_CODE}` is different from the four above** — it isn't usable at any phase, only Problem Domain (Brief/Domain/BRD), and it isn't a check against an existing artifact — it's a free-form, optional, repeatable session (with or without dropped `source-material/` files) that materializes facts into `priming/priming-package.md`, giving `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery` a head start instead of a blank page. Runs after `/init-project`, since it needs the folder structure that command creates. Full design: `ANCHOR-PROJECT-DESIGN.md` §5.

---

## Running This Sequence via `/anchor-project`

Everything above — every step, command, gate, and template — describes running the framework by typing each command yourself. `/anchor-project {PROJECT_CODE}` is a second way to run the identical sequence: it resolves or bootstraps a project, determines the current stage the same way this document's own "How to Resume a Mid-Project Session" section below describes doing it by hand, then invokes the right command for that stage and executes its instructions exactly as written — nothing about a command's own behavior, file formats, or gates changes depending on whether it was invoked directly or through anchor. What anchor adds on top: state continuity across sessions (so resuming doesn't require re-deriving where you left off), a compression layer on PO-facing questions and review output, and cross-artifact drift checking nothing else in the pipeline runs proactively. Where document ingestion is needed, anchor hands off to `/run-priming-session` rather than doing it itself — a standalone companion command, not a mode of anchor — and decomposes its output into each phase's own expected input at the moment anchor actually reaches that phase, never speculatively ahead of time. It stops driving once a project's Walking Skeleton (`FW-048`) is complete — see `ANCHOR-PROJECT-DESIGN.md` §9 — narrowing afterward to release prep and formal scope changes, with day-to-day story work reverting to direct command use. Full design and rationale: `ANCHOR-PROJECT-DESIGN.md` at the repo root; the command itself: `.claude/commands/anchor-project.md`.

Every command remains fully usable standalone, with or without anchor — this is an additive way to run the sequence, not a replacement for the one documented above.

---

## Phase 1 — Intake

*Domain: Problem Domain (`FW-022`) — no architecture dependency.*

**Purpose:** Create the project record and a complete, approved brief that all subsequent commands read as context.

**Entry condition:** None — this is the first phase for every new project.

### Step 1 — `/init-project`

**What it does:**
- Creates the project directory: `projects/{PROJECT_CODE}/`
- Creates `brief.md` from `projects/TEMPLATE/brief.template.md`, pre-filled with the project name, code, client, and domain from arguments, with every section body left as an explicit placeholder pointing at `/run-intake`
- Creates the full folder skeleton (`domain/`, `standards/`, `brds/`, `epics/`, `screens/`, `stories/`, `architecture/`, `mocks/`, `sprints/`, `test-reports/`, `pr/`, `releases/`)
- Registers the project in `projects/index.md` (creates the registry if absent) — this is the single source of project identity (code, name, client, domain, created date); there is no separate `project.json` (`FW-027`)

**Arguments:** `<PROJECT_CODE> "<Project Name>" "<Client>" "<Domain>"`

**Output:** `projects/{PROJECT_CODE}/brief.md` (placeholders only), full folder skeleton, a row in `projects/index.md`

**Gate:** None — the brief is a working document at this point, not yet approved.

**Next step:** `/run-intake`

---

### Step 1a — `/run-intake`

**What it does:**
- Reads the existing `brief.md` (state) and `brief.template.md` (question guide — `[AI Guide]` notes, correct/incorrect examples, verification checklist)
- Conducts the intake session section by section, writing each section to `brief.md` immediately on PO approval — progress is never lost across sessions
- **Amendment mode:** if the brief is already `Approved`, any change made is recorded as a tracked amendment in the document's history table, not a silent overwrite
- At completion, sets the brief's status note to `Brief complete — ready for /gen-domain-knowledge` and recommends running the approval gate below

**Arguments:** `<PROJECT_CODE>`

**Gate check:** `brief.md` must exist (i.e. `/init-project` has run).

**Output:** Fully completed `projects/{PROJECT_CODE}/brief.md`

**Gate:** None yet — approval is the next step.

---

### Step 1b — Brief Approval

**Command:** `update-status projects/{PROJECT_CODE}/brief.md Approved`

**What happens:** `brief.md`'s **Content Status** field set to `Approved`. This is a hard gate — `/gen-domain-knowledge` explicitly checks for it and refuses to run otherwise.

**Gate:** ✅ PO manual decision. Required before `/gen-domain-knowledge` can run.

> **`brief.md` has two distinct status fields, resolved 2026-07-13:** **Status** (`Active`/`On Hold`/`Complete`/`Cancelled`) tracks the project's overall lifecycle, set once at `/init-project` and kept in sync with `projects/index.md`. **Content Status** (`Draft`/`Under Review`/`Brief complete`/`Approved`/`Locked`/`Superseded`) tracks this document's own readiness — the field `/run-intake` and this gate actually use. The two never share a value and `update-status` now routes to the correct one automatically based on which value is given. See `brief.template.md`'s own AI Guide note for the full explanation.

---

## Phase 2 — Discovery

*Domain: Problem Domain (`FW-022`) — no architecture dependency.*

**Purpose:** Build the domain knowledge base (optionally via a dedicated discovery session for niche domains), structure the requirement-discovery session, run it with the client (or via AI-synthesis for internal/no-external-client projects), and produce the approved BRD.

**Entry condition:** Brief must be `Approved`.

### Step 2a — `/gen-domain-playbook` *(optional — niche or complex domains only)*

**What it does:**
- Reads the brief for domain name, sector, and problem statement
- Reads `projects/TEMPLATE/domain/domain-playbook.template.md`
- Produces a playbook that guides a *domain education* session (the PO/domain expert teaching the AI about the domain) — distinct from a BRD discovery playbook, which discovers client requirements. Produces no BRD input directly; feeds `/run-domain-discovery`.

**Arguments:** `<PROJECT_CODE> [<domain-name>]`

**Gate check:** `brief.md` must exist.

**Output:** `projects/{PROJECT_CODE}/domain/{domain-slug}-domain-playbook.md`

**When to skip:** for well-known domains (invoicing, payroll, school fees, HR, etc.) where AI training knowledge is already sufficient — go straight to Step 2.

---

### Step 2b — `/run-domain-discovery` *(optional, follows 2a)*

**What it does:**
- Reads the domain playbook from Step 2a, the brief, and any prior `domain-discovery-state.md`
- Conducts the session section by section, saving state progressively (not just at the end)
- Calculates a Domain Confidence Score; at completion, offers to generate the domain knowledge document immediately using live session context (recommended over a fresh session reading only the saved state)

**Arguments:** `<domain-playbook-path>`

**Gate check:** Domain playbook must exist (Step 2a). Brief must exist.

**Output:** `projects/{PROJECT_CODE}/domain/domain-discovery-state.md`

---

### Step 2 — `/gen-domain-knowledge`

**What it does:**
- Reads `brief.md` — extracts domain, sector, and constraints
- **Checks whether `domain-discovery-state.md` exists (Steps 2a/2b above):** if it does, uses the PO's stated answers as the primary input. **If it does not**, shows an explicit warning and waits for confirmation before proceeding on AI training knowledge alone — sufficient for well-known domains, risky for niche/regulated ones (`FW-013`: Domain Discovery is optional, unlike Screen Design)
- Reads `projects/TEMPLATE/domain/domain.template.md`
- Produces a comprehensive domain knowledge document: core entities, standard lifecycle models, universal business rules, validation rules, common personas, common configurations, known edge cases — reusable across projects, not client-specific
- Can also generate a country- or sector-specific **extension** of an existing base document (same command, different arguments) — output path varies accordingly (see Output below)

**Arguments:** `<PROJECT_CODE> [<existing-domain-knowledge-file>]` — the second argument extends an existing doc rather than starting fresh

**Gate check (before generating):** Brief must be `Approved` (not merely exist — see Step 1b).

**Output:** `projects/{PROJECT_CODE}/domain/{domain-slug}-core.md` (base), or `{domain-slug}-{country-code}.md` / `{domain-slug}-{sector-slug}.md` (extension)

**Gate:** None — domain knowledge is reference material, not a locked artifact.

**Next step:** `/gen-brd-playbook`

---

### Step 3 — `/gen-brd-playbook`

**What it does:**
- Reads the domain knowledge document, the brief (for Extension Context), and resolves the extension chain if this project extends another
- Reads `projects/TEMPLATE/brds/brd-playbook.template.md`
- Converts the domain knowledge document's facts into structured questions, scope risk flags, and journey walkthroughs — the active process document used live during the real discovery session (Step 4)
- Every Known Variation (domain doc Section 10) must map to at least one question; every Typical Integration Point (Section 8) must appear in the integration confirmation table

**Arguments:** `<domain-knowledge-file-path>`

**Gate check:** Domain knowledge file must exist.

**Output:** `projects/{PROJECT_CODE}/brds/{domain-slug}-brd-playbook.md`

---

### Step 3a — `/run-mock-discovery` *(optional but recommended)*

**What it does:**
- Simulates *both* sides of a discovery session internally — AI-as-facilitator following the BRD playbook, AI-as-simulated-client using the domain knowledge document — to catch playbook/domain-knowledge gaps before real client cost
- Evaluates five structured checks (redundant questions, missed variations, restated domain defaults, missing requirements, factual accuracy) and updates the domain knowledge document's Section 12.6 with results

**Arguments:** `<domain-knowledge-path> <brd-playbook-path>`

**Gate check:** Both files must exist (Steps 2 and 3).

**Output:** Updated `projects/{PROJECT_CODE}/domain/{domain-slug}-core.md` Section 12.6 (Mock Discovery Validation)

**When to skip:** if confident in the domain knowledge and playbook already — this is a quality gate, not a hard requirement.

---

### Step 4 — `/run-brd-discovery`

**What it does:**
- Reads `brief.md`, the domain knowledge file, and the BRD playbook
- Conducts the discovery session interactively — for projects with no external client (e.g. an internal SaaS product), this runs as **AI-synthesis-and-review** per the brief's stated discovery operating mode: the AI proposes an answer grounded in the domain knowledge document's defaults, and the PO reviews/corrects/overrides
- Writes `discovery-state.md` progressively as sections are completed
- **At session end:** offers to generate the BRD immediately in the same session (recommended — richer live context) or defer to a standalone `/gen-brd` run later

**Arguments:** `projects/{PROJECT_CODE} <domain-knowledge-file> <brd-playbook-file>`

**Gate check (before starting):** Brief, domain knowledge, and BRD playbook must all exist.

**Output:** `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` (and optionally the BRD, in the same session)

**Gate:** None — discovery-state is a working document. BRD approval (Step 5a/6) is the gate.

---

### Step 5 — `/gen-brd` *(standalone — only if not generated inline at Step 4)*

**What it does:**
- Reads `brd-discovery-state.md`, the BRD playbook, domain knowledge files, and `brief.md`
- Reads `projects/TEMPLATE/brds/brd.template.md`
- Produces the full Business Requirements Document: user roles, journeys, functional requirements, business rules, data requirements, integrations, security requirements — every requirement traced to the discovery session record, none invented

**Arguments:** `<discovery-state-file> <domain-knowledge-file>` or `projects/{PROJECT_CODE}` (auto-discovers files)

**Gate check (before generating):** `brd-discovery-state.md` must exist with its Playbook Progress section fully completed.

**Output:** `projects/{PROJECT_CODE}/brds/brd-core-v1.md`

---

### Step 5a/6 — `/review-brd`

**What it does:**
- Guided walkthrough, not a report — walks through Functional Requirements feature-area by feature-area (framed in plain business language), then Business Rules, Integration Requirements, and Security Requirements as batches
- Progress recorded in the BRD's own Section 16 (Review Record) — resumable across sessions
- **Handles approval inline:** when the PO types `'approved'`, the command itself sets Section 14 Status to `Approved`, fills the Review Summary, and adds the Document Control row — there is no separate `update-status` call in the normal path

**Arguments:** `<brd-file-path>` or `<PROJECT_CODE>`

**Gate check (before reviewing):** BRD status must be `Draft` or `Under Review` (not already `Approved`).

**Output:** Updated BRD with Section 16 review record filled and, on approval, Status set to `Approved`

**Gate:** ✅ PO manual decision (typing `'approved'` inside the command). `update-status projects/{PROJECT_CODE}/brds/brd-core-v1.md Approved` remains available as a manual fallback if approval needs to happen outside the review session, but is not the primary path. Required before `/gen-epics` can run.

---

## Phase 3 — Product Structuring

*Domain: Problem Domain (`FW-022`) — no architecture dependency.*

**Purpose:** Break the approved BRD into delivery units (epics) — the full v1 scope, generated upfront, since epics carry no architecture dependency and are safe to fully elaborate before any sprint exists.

**Entry condition:** BRD must be `Approved`.

### Step 7 — `/gen-epics`

**What it does:**
- Reads the BRD and domain knowledge
- Reads `projects/TEMPLATE/epics/epic.template.md`
- Groups BRD requirements into epics — each a cohesive capability deliverable independently — **for all layers in the BRD at once** (Standalone projects); for Extension projects, only the new layer, cross-checked against the parent project's existing epics
- Produces one epic file per epic, plus an epics index with the REQ coverage map and dependency graph (layers) that later phases (Phase 6's wave sequencing) read directly

**Arguments:** `projects/{PROJECT_CODE}/brds/brd-core-v1.md`

**Gate check (before generating):** BRD status must be `Approved`.

**Output:** `projects/{PROJECT_CODE}/epics/EPIC-{NNN}-{slug}.md`, `projects/{PROJECT_CODE}/epics/index.md`

---

### Step 8/9 — `/review-epics`

**What it does:**
- Reviews all epics one at a time against the BRD and epic template — structural checks (REQ coverage, scope boundaries, dependency ordering, size, no gold-plating) embedded directly in the command
- Progress recorded in `epics/review-record.md` — resumable across sessions
- **Handles approval inline:** updates each epic's Status from `Draft` to `Approved` directly as part of the review — there is no separate `update-status` call per epic in the normal path

**Arguments:** `projects/{PROJECT_CODE}`

**Gate check:** BRD must be `Approved`.

**Output:** Updated epic files with review fixes applied and Status set to `Approved`

**Gate:** ✅ PO manual decision (confirmed during the review walkthrough). `update-status projects/{PROJECT_CODE}/epics/EPIC-{NNN}.md Approved` remains available per-epic as a manual fallback, but is not the primary path. All epics must be `Approved` before `/gen-screen-design` can run.

---

## Phase 4 — Screen Design

*Domain: Problem Domain (`FW-022`) — no architecture dependency.*

**Purpose:** Derive Portals, Navigation Map, and Screen Inventory from the BRD alone — pattern-matched against `ascendra-ui`'s real component catalog — before architecture starts. See `decisions/FW-026-screen-design-phase.md` for why this sits here rather than inside Architecture. **Not optional**, unlike Domain Discovery — screens are project-specific by definition, with no "well-known enough to skip" case — except when the BRD has no UI in scope at all (pure API/backend project).

**Entry condition:** All epics must be `Approved`.

### Step 9a — `/gen-screen-design`

**What it does:**
- Reads the BRD (Personas, Roles, Journeys, FRs) and domain knowledge — no tech-stack dependency
- Reads `projects/TEMPLATE/screens/screen-design.template.md`
- Derives Portals, Navigation Map, and Screen Inventory; pattern-matches every screen to a Surface (Page/Dialog/Sheet/Drawer) and UI Pattern (Form/Dashboard/Report/Table-List/Detail), citing a specific `ascendra-ui` catalog entry or primitive composition for each
- Records per-screen data requirements (input to `/gen-architecture`'s Data Model and API Contracts steps)
- Presents the full table to the PO for confirmation, then writes the file
- **Invokes `/gen-ui-mocks` in base mode** — not run standalone; this is the first of `/gen-ui-mocks`'s two invocations

**Arguments:** `{PROJECT_CODE}`

**Gate check (before generating):** All epics `Approved`. **Skipped entirely** (not applicable) if the BRD shows no persona with a logged-in UI session — pure API/backend project goes straight to `/gen-architecture`.

**Output:** `projects/{PROJECT_CODE}/screens/screen-design.md`, plus `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html` per portal

---

### Step 9b/9c — `/review-screen-design`

**What it does:**
- Sequential walkthrough of every screen against the BRD, mirroring `/review-epics`'s structure
- Checks: every journey covered by a screen, every Surface/Pattern citation resolves to a real catalog entry or primitive, Portals are genuine navigation shells, base mocks match the table
- **Handles approval inline:** on the PO typing `'approve'`, sets Status to `Approved` directly

**Arguments:** `{PROJECT_CODE}`

**Gate check:** Document must exist and not already be `Approved`.

**Output:** Updated `screen-design.md` with review fixes applied and Status set to `Approved`

**Gate:** ✅ PO manual decision (confirmed during the review). `update-status projects/{PROJECT_CODE}/screens/screen-design.md Approved` remains available as a manual fallback. Screen Design must be `Approved` before `/gen-architecture` can run (when applicable).

---

## Phase 5 — Architecture

*Domain: Bridge (`FW-022`) — the first hard architecture dependency point in the gate chain.*

**Purpose:** Define the complete technical blueprint before any story is written or code is implemented.

**Entry condition:** All epics must be `Approved`. Screen Design must be `Approved` (when the BRD has a UI in scope — Phase 4, Step 9c).

### Step 10 — `/gen-architecture`

**What it does:**
- Reads the BRD, epics index, domain knowledge, `screen-design.md` (when present), and all architecture standards files
- Reads `projects/TEMPLATE/architecture/arch.template.md`
- Determines the tech stack, standards files, internal layering pattern, and repo/scaffold topology **on the fly per project** — no fixed "Ascendra standard stack" (`FW-025`). NestJS + Next.js/Ascendra UI is the common default and fallback, not a mandate; a different confirmed stack is fully legitimate and must be followed by every downstream command (`FW-029`)
- Produces the complete architecture document: system overview, tech stack, module breakdown, project structure (backend + frontend + any additional confirmed repo — mobile/worker), full data model, all API endpoint contracts with DTOs, security design (auth, RBAC, delegated access, sensitive fields), infrastructure plan, environment variables, deviation declarations
- Section 3.1.3.1–3.1.3.3 (Portals, Navigation Map, Screen Inventory) are carried over from the approved `screen-design.md`, not re-derived. Only 3.1.3.4 (Action Gating) and 3.1.3.5 (Cross-Cutting UI Rules) are generated fresh here, once Section 6 (security model) exists
- **Invokes `/gen-ui-mocks` in patch mode** once Section 6 is written — the second of its two invocations, adding gating captions and cross-cutting banners to the base mocks from Step 9a
- Runs Part 2 verification checklist before writing the file

**Arguments:** `<brd-file> "<project-name>" <PROJECT_CODE>`

**Gate check (before generating):** All epics `Approved`. Screen Design `Approved`, when the BRD has a UI in scope.

**Output:** `projects/{PROJECT_CODE}/architecture/arch-v1.md` and `arch-v1-ref.md` (compact quick-reference, generated in the same step)

---

### Step 11/12 — `/review-architecture`

**What it does:**
- Reads the architecture document, BRD, domain knowledge, and all architecture standards files
- Runs 35 AI-verifiable structural checks across 5 concerns (module-table coverage, BRD requirement coverage, money column types, `orgId` not in DTOs, state machine completeness, user role completeness, state transition patterns, domain-specific actor attribution symmetry, open decisions, and more)
- Presents each concern as a guided walkthrough with real-world framing before surfacing FLAGS, per-concern — not a single upfront report
- Applies Tier 1 and Tier 2 fixes on PO confirmation
- **Handles lock inline:** on PO approval, updates Status to `Locked` and fills the Approval section directly

**Arguments:** `projects/{PROJECT_CODE}/architecture/arch-v1.md` or `projects/{PROJECT_CODE}`

**Gate check (before reviewing):** Architecture status must be `Draft` or `Under Review` (not already `Locked`).

**Output:** Updated `arch-v1.md` with fixes applied and Status set to `Locked` (on PO approval)

**Gate:** ✅ PO manual decision. The lock confirmation invites an optional one-line rationale (`lock: {what you specifically verified}`) and asks once if the PO declines to give one with a bare `lock` — either way, `lock` alone is always sufficient and never blocked on a rationale being supplied. A given rationale is recorded in Section 10 (Approval) as `**Locked because:**`; a bare `lock` simply omits that line rather than writing a placeholder. `update-status projects/{PROJECT_CODE}/architecture/arch-v1.md Locked` remains available as a manual fallback (it never prompts for a rationale). Required before `/implement-story` can run for any story; also required (soft check) for `/gen-stories`.
>
> **Resolved 2026-09-15:** an earlier version of this document described a lock-rationale prompt that had gone missing from `review-architecture.md`, and this section previously flagged the discrepancy as an open, undecided question. It's been restored — see above — as a lightweight, optional capture (never a hard requirement) so the framework retains a "why," not just the "what," on the single most consequential PO decision in the pipeline.

---

## Phase 6 — Sprint Delivery

*Domain: Solution Domain (`FW-022`) — requires Locked architecture, directly.*

**Purpose:** Generate, review, sprint-assign, implement, verify, and merge — one **wave** at a time. A wave is normally one epic; it is more than one epic only when `epics/index.md` documents a story-level interleaving between them (their true build order doesn't reduce to a clean epic-level sequence), or when a small epic is deliberately combined with the next to fill sprint capacity. Sprint boundaries are decided during Step 14/15's Sprint Assignment, never upfront and never baked into a Story ID.

**Entry condition:** Architecture must be `Locked`. **Repeats for every wave** — not for every sprint 1:1, since a wave and a sprint are not always the same size.

### Step 13 — `/gen-stories`

**What it does:**
- Reads the epic file, BRD, domain knowledge, architecture document, and `epics/index.md` Section 3 (Dependency Graph) — the last of these to check whether this epic carries a documented story-level interleaving note with another epic
- Reads `projects/TEMPLATE/stories/story.template.md` and the project's chosen `reference/estimation/` model (copied to `projects/{PROJECT_CODE}/standards/estimation.md` on first run)
- Generates one file per story: story statement, FR references, acceptance criteria, out of scope, size, dependencies, notes — technology-agnostic, no field names, table names, API routes, or library names in ACs
- Runs Part 2 verification on each story before writing
- Closing message: if this epic is interleaved with another, instructs generating that paired epic next before any review; otherwise instructs running `/review-stories` for this epic now — **never** "generate every remaining epic first"

**Arguments:** `<epic-file> [<starting-sequence>]` — no sprint number; sprint membership isn't known until Step 14/15 and is never encoded in the Story ID (`US-{epic}-{seq}`, stable from generation — see `conventions/artifact-naming-conventions.md`)

**Gate check (before generating):** Epic status must be `Approved`.

**Output:** `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/US-{epic}-{seq}-{slug}.md`

**Repeats:** once per wave — do not generate stories for every remaining epic before the current wave is reviewed and sprint-assigned.

---

### Step 14/15 — `/review-stories`

**What it does:**
- Reviews stories against the template, epic scope, BRD, and estimation rules — 10 structural checks per story, 6 epic-level checks per epic
- Pass 2 dependency resolution: replaces plain-English epic references with resolved story IDs
- Processes every epic that currently has stories, in dependency order — under the wave-based flow this is naturally scoped to the current wave, since no later epic has stories yet
- **Handles approval inline:** updates each reviewed story's Status from `Draft` to `Reviewed` directly, once all flags are resolved
- **Sprint Assignment (Part F, per epic/wave):** the step that actually decides sprint boundaries — nothing upstream or downstream does this. Using this wave's real story count against the project's capacity target (e.g. 8–10 for T-Shirt Sizing): assigns a sprint number if at/near target; offers to pull in the next unblocked epic if below target (or top up with earlier leftover Reviewed-but-unassigned stories); asks the PO for a split boundary along intra-epic delivery order if above target. Writes the result into `stories/index.md` Section 8 — this is the only place that table gets populated

**Arguments:** `projects/{PROJECT_CODE}` — processes whatever epics currently have stories; there is no separate single-epic argument

**Output:** Updated story files (Status → `Reviewed`), `stories/index.md` Section 8 populated with sprint assignment

**Gate:** ✅ PO manual decision (confirmed during the per-story/per-epic walkthrough). `update-status projects/{PROJECT_CODE}/stories/EPIC-{NNN}/US-{epic}-{seq}.md Reviewed` remains available per-story as a manual fallback. Stories must be `Reviewed` and sprint-assigned before appearing in a sprint plan and before `/implement-story` can run.

---

### Step 16 — `/gen-sprint-plan`

**Tip:** Run `/project-status projects/{PROJECT_CODE}` before generating the sprint plan — the dashboard confirms at a glance that all sprint stories are `Reviewed`, dependency epics are `Done`, and no L-stories are flagged for split, the same checks this command runs internally.

**What it does:**
- Reads `projects/{PROJECT_CODE}/stories/index.md` and `epics/index.md`
- Reads `projects/TEMPLATE/sprints/sprint-plan.template.md`
- Extracts stories assigned to this sprint from Section 8 — an assignment Step 14/15's Sprint Assignment must already have written; this command does not decide sprint scope itself
- Runs readiness gate checks: architecture `Locked`, epics `Approved`, stories `Reviewed`, dependencies cleared, L-story split decisions recorded
- Produces the sprint plan document: goal, sprint roadmap (from the same Section 8 data), pre-sprint checklist, story board, delivery sequence, capacity analysis, Should Have decisions, Definition of Done

**Arguments:** `projects/{PROJECT_CODE} {sprint-number}`

**Gate checks:** Architecture `Locked`; all sprint epics `Approved`; all sprint stories `Reviewed`; dependency epics `Done` or `Active`.

**Output:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md`

**Repeats:** once per sprint.

---

### Step 17 — Sprint Activation

**Command:** `update-status projects/{PROJECT_CODE} sprint {NN} Active`

**Gate:** ✅ PO manual decision. Required before implementation begins. All pre-sprint checklist items must be resolved.

**During the sprint — track progress at any time:** `/project-status projects/{PROJECT_CODE}` — stories by status, progress bars per sprint, remaining workload by size, and an ATTENTION section surfacing blockers.

---

> **Steps 17a–21 repeat for every story in the sprint, in delivery sequence order from the sprint plan.** Complete the full loop (plan → implement → verify → PR → merge) for one story before starting the next.

### Step 17a — `/gen-story-plan` (`FW-031`)

**What it does:**
- Gate-checks story `Reviewed` and architecture `Locked`
- Extracts — never invents — the endpoints, tables, and screens this one story needs from the already-Locked architecture (Section 5/4.2 for API, Section 3.1.2/3.1.3 + the approved mock for Web), filtered down to exactly this story's scope
- Builds an AC → Artifact traceability table, a Test Scenarios table (one row per AC, consumed later by `/verify-story`), and flags any Risk/Open Question the story or architecture leaves ambiguous
- Never states *how* to build anything in stack-specific syntax — that remains `/implement-story`'s job

**Arguments:** `<story-file-path>`

**Gate check:** Story `Reviewed` or later; architecture `Locked`; no unresolved Open Decision (Section 8) in this story's capability area.

**Output:** `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md`, **Plan Status:** `Draft`. PO reviews it, then runs `update-status projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md Confirmed` — `/implement-story` hard-gates on this, no override.

---

### Step 18 — `/implement-story`

**What it does:**
- Gate-checks story `Reviewed`, architecture `Locked`, all declared dependencies `Merged`/`Done`, **and the story's implementation plan `Confirmed` (`FW-031`, hard gate, no PO override)** before proceeding
- Reads the confirmed stack from architecture Section 2 first — NestJS/Next.js is this project's confirmed choice and the framework's common default, not a universal assumption (`FW-025`, `FW-029`); a different confirmed stack means following that project's own arch-v1.md structure instead of the NestJS/Next.js-specific syntax
- Reads the Confirmed plan as its task list — which artifacts (`API-N`/`WEB-N`) to build, in what order — cross-checked against the story's own `Target` field (API / Web / Worker / multiple), resolved from architecture Section 3.1 as a sibling directory (`../{repo-name}`) — never from a per-project config file (`FW-027`)
- **Bootstraps the repo if it doesn't exist yet** — scaffolds it (e.g. `nest new`, or cloning `ascendra-ui` for Next.js) the first time any story touches it, resolving the circular "scaffolding story needs the repo to exist" problem
- Creates a story branch (`story/{STORY-ID}-{slug}`) per `git-standards.md` before writing code
- First-run bootstrap: creates `README.md`/`CLAUDE.md` in the target repo if missing
- Reads the approved UI mock's actual markup for Web stories (Step 2 of the command) — builds to match the PO-approved visual shape, not a fresh interpretation
- Implements exactly the plan's declared artifacts, satisfying every story AC — nothing more. Any mid-implementation deviation from the plan is recorded in the plan's own Section 7 (Deviations from Plan) with rationale, never applied silently
- Runs lint, build, and test suite in the target directory(ies); commits on the story branch
- Reports files created/modified, checks passed, commit, plan artifacts built, deviations, and AC coverage — per repo touched

**Arguments:** `<story-file-path>`

**Gate check (before implementing):** Story `Reviewed`; architecture `Locked`; dependencies `Merged`/`Done`; implementation plan `Confirmed`; no unresolved Open Decision (Section 8) in this story's capability area.

**Output:** Code committed to the target repo(s) on the story branch. Status updated to `In Progress` automatically.

---

### Step 19 — `/verify-story`

**What it does:**
- Reads the story's acceptance criteria (Section 3) and its implementation plan's Test Scenarios table (Section 4) — the primary source, authored before implementation existed, so it can't have unconsciously bent to match whatever got built. Falls back to deriving scenarios on the fly from the AC text only if no plan file exists (a pre-`FW-031` story)
- Reads `projects/{PROJECT_CODE}/standards/test-strategy.md`
- For each scenario: exercises the running application via its confirmed base path (API endpoint, UI screen, or automated job — black-box, stack-agnostic by design) and records the result
- Adds a **Plan Conformance** section cross-checking every plan-declared artifact against what was actually delivered, and surfaces any undeclared deviation (an artifact built differently than planned with no matching Section 6 entry) as a defect
- Reports: overall Pass/Fail, per-AC result, evidence (response bodies, UI states, log entries), any defects found
- Does NOT update the story status — that remains a PO decision

**Arguments:** `<story-file-path>`

**Gate check:** Story must be `In Progress` (or `PR Created`/`Merged`/`Done`, with a confirmation prompt for the latter two — useful for regression checks).

**Output:** Test execution summary in the conversation. Optionally writes `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`

---

### Step 20 — `/gen-pr-description` + PO PR Review

**What `/gen-pr-description` does:**
- Runs after `/verify-story` reports all ACs passing — reads the story, the passing verification report, and the real code diff (`git diff main` in the target repo)
- Reads `projects/TEMPLATE/pr/pr-description.template.md`
- Produces the PR title and full description: Summary, Type of Change, What Changed (from the real diff, not the ACs), How to Test (from the verification report's evidence), Screenshots (Web stories), Pre-Merge Checklist
- Presents real-world framing of the PR for PO confirmation before writing the file

**Arguments:** `<story-file-path>`

**Gate check:** Story `In Progress` or `PR Created`; a passing verification report must exist.

**Output:** `projects/{PROJECT_CODE}/pr/US-{epic}-{seq}-pr-description.md` — used as the PR body (e.g. `gh pr create --body-file ...`)

**Manual step — PO PR Review.** PO reviews the PR using the generated description. Three outcomes: **Approve** → story moves to `Merged`; **Request Changes** → story moves to `Changes Requested`, Engineering Agent revises on the same branch; **Reject** → Engineering Agent closes PR, deletes branch, reimplements from scratch on a new branch.

---

### Step 21 — Story Status Update

**Command:** `update-status <story-file> Merged` or `update-status <story-file> Done`

`Merged` = PR merged to main branch. `Done` = acceptance criteria verified, sprint complete.

---

### Step 22 — `/assess-change` *(mid-sprint, as-needed — cross-cutting)*

**What it does:**
- Assesses a described change against the BRD, epics, stories, and architecture using an index-first reading strategy (never loads more than 5 epic files or 15 story files in one session)
- Classifies into one of 4 patterns (Fix / Gap / Scope Shift / Requirement Change) plus a Parking Lot variant, each with its own formality level
- Determines blast radius: which artifacts are affected, which approved documents need re-approval
- Presents the impact to the PO before touching anything; on confirmation, applies all updates across the document pipeline top-down in one coordinated session
- Also the mechanism for propagating a sprint's delivery/verification/UAT learnings into a not-yet-built epic — `/close-sprint`'s own retrospective is process-only and does not do this itself

**Arguments:** Description of the change, project path

**Gate:** ✅ PO must confirm blast radius before any document is modified. Formal changes (Pattern 3/4) require an additional explicit confirmation before touching signed-off artefacts.

---

## Phase 7 — Sprint Closure & Release

*Domain: Solution Domain (`FW-022`) — requires Locked architecture, transitively.*

**Purpose:** Confirm the sprint is technically sound, get UAT sign-off, close the sprint, and generate release artefacts. **Repeats per sprint.**

**Entry condition:** All sprint stories must be `Merged`.

### Step 23 — Sprint-wide QA Confirmation

**Manual confirmation step — no slash command.** Before generating the UAT checklist, confirm the sprint is clean as a whole:

1. **Run the full test suite** in the target repository (the project's confirmed toolchain — `npm test -- --run` for this project's Node/TypeScript stack). All tests must pass.
2. **Check coverage** against `test-strategy.md`'s threshold. Add missing tests if coverage dropped.
3. **Review all `/verify-story` reports** for this sprint. Confirm zero open Critical or High defects.
4. **Record the outcome** in the sprint plan's Sprint Notes section. If a full test execution report is required (client-facing project), generate it using `projects/TEMPLATE/test-reports/test-execution-report.template.md`, aggregating from every story's `/verify-story` report for this sprint.

If any Critical or High defect is open: return to Step 18 (fix), 19 (verify), 20 (PR), 21 (merge), then re-confirm here.

---

### Step 24 — `/gen-uat-checklist`

**What it does:**
- Reads all story files for the sprint
- Reads `projects/TEMPLATE/sprints/uat-checklist.template.md`
- Produces a structured UAT test sheet: one section per story, one row per AC — test steps, expected result, PO result (Pass/Fail), notes. Written for the Product Owner to execute manually — no technical jargon
- References the sprint's approved mocks and Navigation Map for context, not just the raw AC text

**Arguments:** `projects/{PROJECT_CODE} {sprint-number}`

**Gate check:** All sprint stories must be `Merged`.

**Output:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}-uat-checklist.md`

---

### Step 25 — PO UAT

**Manual step.** PO works through the UAT checklist story by story, marking each AC Pass or Fail. Any Fail raises a defect, classified using `reference/defect-severity.md`. Critical/High defects return to implementation (18), re-verify (19), PR (20), merge (21), QA re-confirmation (23), then back here. Medium/Low defects are batched into a future sprint at PO's discretion.

---

### Step 26 — `/close-sprint` *(helper)*

**What it does:**
- Reads the sprint plan document and story statuses
- Fills the Retrospective section: what went well, what to improve, velocity, carry-forward items
- **Checks for forward-looking learnings:** asks whether anything from this sprint's delivery/verification/UAT should correct a not-yet-built epic — if so, points at `/assess-change` before the next wave's `/gen-stories` runs against possibly-stale scope. This retrospective itself is process-only (velocity, carry-forward) and does not correct requirements on its own.
- Does NOT mark the sprint `Complete` — that remains a PO decision via `update-status`

**Arguments:** `projects/{PROJECT_CODE} {sprint-number}`

**Gate check:** UAT must be complete (PO confirms before running).

**Output:** Updated `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md` (retrospective filled)

---

### Step 27 — Sprint Closure

**Command:** `update-status projects/{PROJECT_CODE} sprint {NN} Complete`

**Gate:** ✅ PO manual decision. UAT signed off, retrospective filled, no open defects.

---

### Step 28 — `/gen-release-notes`

**What it does:**
- Reads the completed sprint plan and all story files for the sprint
- Reads `projects/TEMPLATE/releases/release-notes.template.md`
- Produces release notes: version number, release date, what changed (grouped by feature area, not by story), known limitations, deployment notes — written for a non-technical audience, described by user outcome not system behaviour

**Arguments:** `projects/{PROJECT_CODE} {sprint-number} <version>`

**Gate check:** Sprint must be `Complete`.

**Output:** `projects/{PROJECT_CODE}/releases/v{version}-release-notes.md` — keyed by version, not sprint number, since a release doesn't always map 1:1 to a sprint

---

### Production Deployment

**Manual step.** PO initiates deployment after release notes are approved. Governed by `standards/ci-cd-pipeline.md` and `standards/environment-strategy.md`. Gate 13 in this document's own Gate Summary table.

---

## Phase 8 — Support

*Domain: Solution Domain (`FW-022`).*

**Purpose:** Monitor the live system, triage production defects, and formally close the project.

**Entry condition:** Successful production deployment. Runs once per project — not per sprint.

**Duration:** Default 30 days. Override via BRD Section 10 (Non-Functional Requirements) or explicit client agreement.

### Defect Triage

Production defects are classified on receipt using `reference/defect-severity.md`:

| Severity | Response | Path |
|---------|---------|------|
| Critical or High | Hotfix Cycle triggered immediately | Hotfix Cycle (below) |
| Medium or Low | Batched into the next sprint; does not block anything | Future sprint |

### Project Closure

At the end of the support period, one of two outcomes:

**Close:** Run `update-status projects/{PROJECT_CODE}/brief.md Complete`. Produce a closure document (summary of what was delivered, open known issues, handover notes). Archive the project directory.

**Retainer:** Agreement made for ongoing development or support. A new project brief or addendum is created. The project directory remains active.

---

## Hotfix Cycle

A hotfix is triggered by a Critical or High severity defect found in production during Support. It bypasses sprint planning and UAT because the scope is narrow and the fix is validated before deployment.

**Trigger:** PO classifies a production defect as Critical or High and confirms a hotfix should proceed.

**Steps (strictly sequential):**

1. **Create a hotfix branch** from `main`: `hotfix/{PROJECT_CODE}-{issue-description}`
2. **Implement the fix** — run `/implement-story` targeting this branch. Scope strictly limited to the defect AC.
3. **Verify the fix** — run `/verify-story` targeting the hotfix story. Confirm the defect scenario passes and no regressions in the affected module.
4. **PR against `main`** — Product Owner reviews and approves (same gate as Step 20).
5. **Targeted test run** — full suite plus specifically the regression tests for the affected module. Full E2E suite not required.
6. **Production deployment** — PO approves with rollback plan in hand (same gate as Phase 7's deployment).
7. **Smoke tests** pass automatically after deployment.
8. **Confirm the fix is preserved going forward.** Story branches are cut from and merged directly into `main` (no persistent sprint-integration branch exists in this framework's git model — see `implement-story.md`) — so merging the hotfix PR into `main` in step 4 already means every story branch cut from `main` afterward includes the fix automatically. The one case that needs a manual check: any story branch for the current sprint that was already cut from `main` *before* the hotfix merged. Merge or rebase `main` into that branch before its own PR merges, so the hotfix isn't silently reverted by an older branch's diff when it lands.

**What is skipped:** Sprint planning, full regression suite, UAT.

**What is NOT skipped:** PO approval at PR, PO approval at deployment, targeted QA verification, rollback plan.

---

## Command Reference

Every command in `.claude/commands/` (33 total), with its domain region and phase.

| Command | Domain | Phase | Inputs | Output | Template |
|---------|--------|-------|--------|--------|---------|
| `/init-project` | Problem | Intake | Arguments | `brief.md` (placeholders), folder skeleton | `brief.template.md` |
| `/run-priming-session` | Problem | Intake *(optional, repeatable)* | `source-material/` files and/or live conversation | `priming/priming-package.md` | `priming-package.template.md` |
| `/run-intake` | Problem | Intake | `brief.md` (placeholders) | Completed `brief.md` | `brief.template.md` |
| `/gen-domain-playbook` | Problem | Discovery *(optional)* | `brief.md` | `domain/{slug}-domain-playbook.md` | `domain-playbook.template.md` |
| `/run-domain-discovery` | Problem | Discovery *(optional)* | Domain playbook, brief | `domain/domain-discovery-state.md` | `domain-discovery-state.template.md` |
| `/gen-domain-knowledge` | Problem | Discovery | `brief.md` (Approved), discovery state (optional) | `domain/{slug}-core.md` | `domain.template.md` |
| `/gen-brd-playbook` | Problem | Discovery | Domain knowledge | `brds/{slug}-brd-playbook.md` | `brd-playbook.template.md` |
| `/run-mock-discovery` | Problem | Discovery *(optional)* | Domain knowledge, BRD playbook | Updated domain knowledge Section 12.6 | — |
| `/run-brd-discovery` | Problem | Discovery | `brief.md`, domain knowledge, BRD playbook | `brds/brd-discovery-state.md` → optionally BRD | `brd-discovery-state.template.md` → `brd.template.md` |
| `/gen-brd` | Problem | Discovery | Discovery state, domain knowledge | `brds/brd-core-v1.md` | `brd.template.md` |
| `/review-brd` | Problem | Discovery | BRD (Draft/Under Review), domain knowledge | Updated BRD (Approved inline) | — |
| `/gen-epics` | Problem | Product Structuring | BRD (Approved) | `epics/EPIC-{NNN}.md`, `epics/index.md` | `epic.template.md` |
| `/review-epics` | Problem | Product Structuring | Epics, BRD | Updated epic files (Approved inline) | — |
| `/gen-screen-design` | Problem | Screen Design | BRD, epics (Approved) | `screens/screen-design.md`, mock HTML | `screen-design.template.md` |
| `/review-screen-design` | Problem | Screen Design | Screen design, BRD | Updated screen design (Approved inline) | — |
| `/gen-ui-mocks` | Problem/Bridge | Screen Design + Architecture *(invoked, not standalone)* | Screen design (base) / Architecture Section 6 (patch) | `mocks/{portal-slug}-mock.html` | `ui-mock.template.md` |
| `/gen-architecture` | Bridge | Architecture | BRD, epics (Approved), screen design, domain knowledge | `architecture/arch-v1.md` + `arch-v1-ref.md` | `arch.template.md` |
| `/review-architecture` | Bridge | Architecture | Architecture (Draft/Under Review), BRD, domain knowledge | Updated `arch-v1.md` (fixes + Locked inline) | — |
| `/gen-stories` | Solution | Sprint Delivery | Epic (Approved), BRD, architecture (Locked) | `stories/EPIC-{NNN}/US-{epic}-{seq}.md` | `story.template.md` |
| `/review-stories` | Solution | Sprint Delivery | Story files, BRD, epics | Updated story files (Reviewed inline), `stories/index.md` Section 8 | — |
| `/gen-sprint-plan` | Solution | Sprint Delivery | `stories/index.md`, epics index | `sprints/sprint-{NN}.md` | `sprint-plan.template.md` |
| `/gen-story-plan` | Solution | Sprint Delivery | Story (Reviewed), architecture (Locked) | `story-plans/US-{epic}-{seq}-plan.md` (Draft) | `story-plan.template.md` |
| `/implement-story` | Solution | Sprint Delivery | Story (Reviewed), architecture (Locked), story plan (Confirmed) | Code + commit in target repo(s) | Story plan → `arch-v1-ref.md` → `arch-v1.md` |
| `/verify-story` | Solution | Sprint Delivery | Story (In Progress), running app | Test execution report | Story plan Section 4 — falls back to deriving from ACs if none |
| `/gen-pr-description` | Solution | Sprint Delivery | Story (In Progress/PR Created), passing verification report | `pr/US-{epic}-{seq}-pr-description.md` | `pr-description.template.md` |
| `/gen-uat-checklist` | Solution | Sprint Closure | Sprint stories (Merged) | `sprints/sprint-{NN}-uat-checklist.md` | `uat-checklist.template.md` |
| `/close-sprint` | Solution | Sprint Closure | Sprint plan, story statuses | Updated sprint plan (retro filled) | `sprint-plan.template.md` |
| `/gen-release-notes` | Solution | Sprint Closure | Sprint (Complete), story files | `releases/v{version}-release-notes.md` | `release-notes.template.md` |
| `/assess-change` | Cross-cutting | Any phase | Change description, project path | Updated artifacts | — |
| `update-status` | Cross-cutting | Any gate | Artifact path + new status | Updated artifact + index | — |
| `/project-status` | Cross-cutting | Any time | Project path | Status dashboard | — |
| `/judgment-check` | Cross-cutting | Before any `/review-X` (all artifact types) | Artifact path | Density & Judgment Log section in that artifact | — |
| `/anchor-project` | All three (orchestrates the full sequence) | Any time | `PROJECT_CODE` (optional — resolves or bootstraps) | Drives whichever command owns the current stage; also writes `.anchor-state.md` (not a project artifact) | — (invokes each phase's own template via that phase's own command) |

---

## Template Reference

| Template | Location | Used by |
|---------|----------|---------|
| `brief.template.md` | `projects/TEMPLATE/` | `/init-project`, `/run-intake` |
| `priming-package.template.md` | `projects/TEMPLATE/priming/` | `/run-priming-session` |
| `domain-playbook.template.md` | `projects/TEMPLATE/domain/` | `/gen-domain-playbook` |
| `domain-discovery-state.template.md` | `projects/TEMPLATE/domain/` | `/run-domain-discovery` |
| `domain.template.md` | `projects/TEMPLATE/domain/` | `/gen-domain-knowledge` |
| `brd-playbook.template.md` | `projects/TEMPLATE/brds/` | `/gen-brd-playbook` |
| `brd-discovery-state.template.md` | `projects/TEMPLATE/brds/` | `/run-brd-discovery` |
| `brd.template.md` | `projects/TEMPLATE/brds/` | `/gen-brd`, `/run-brd-discovery` |
| `epic.template.md` | `projects/TEMPLATE/epics/` | `/gen-epics` |
| `screen-design.template.md` | `projects/TEMPLATE/screens/` | `/gen-screen-design` |
| `ui-mock.template.md` | `projects/TEMPLATE/mocks/` | `/gen-ui-mocks` |
| `arch.template.md` | `projects/TEMPLATE/architecture/` | `/gen-architecture` |
| `system-architecture.template.md`, `technical-standard.template.md`, `architectural-pattern.template.md` | `projects/TEMPLATE/standards/` | `/gen-architecture` |
| `story.template.md` | `projects/TEMPLATE/stories/` | `/gen-stories` |
| `story-plan.template.md` | `projects/TEMPLATE/story-plans/` | `/gen-story-plan` |
| `estimation/*.md` (T-shirt/Fibonacci/Time-based) | `reference/estimation/` | `/gen-stories`, `/review-stories` |
| `sprint-plan.template.md` | `projects/TEMPLATE/sprints/` | `/gen-sprint-plan`, `/close-sprint` |
| `uat-checklist.template.md` | `projects/TEMPLATE/sprints/` | `/gen-uat-checklist` |
| `test-execution-report.template.md` | `projects/TEMPLATE/test-reports/` | Step 23 (manual, aggregates `/verify-story` reports) |
| `pr-description.template.md` | `projects/TEMPLATE/pr/` | `/gen-pr-description` |
| `release-notes.template.md` | `projects/TEMPLATE/releases/` | `/gen-release-notes` |

---

## Status Values Reference

All statuses are set via `update-status` (or inline by the corresponding `review-*` command, per Phases 2–6 above). Gates are enforced — commands check status before proceeding.

### Brief — two distinct fields (see Phase 1, Step 1b)

**Content Status** (document readiness):
| Status | Meaning |
|--------|---------|
| `Draft` | Placeholders only, awaiting `/run-intake` |
| `Under Review` | Being revised or reviewed |
| `Brief complete` | `/run-intake` finished; awaiting PO approval |
| `Approved` | Gates `/gen-domain-knowledge` |
| `Locked` | No further changes without a formal change request |
| `Superseded` | Replaced by a later brief version |

**Status** (project's overall lifecycle, kept in sync with `projects/index.md`):
| Status | Meaning |
|--------|---------|
| `Active` | Project in delivery |
| `On Hold` | Paused — reason recorded in brief Section 8 |
| `Complete` | Project delivered and support period ended |
| `Cancelled` | Project cancelled — all work preserved for reference |

### BRD
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Being generated or revised | `/gen-epics` blocked |
| `Under Review` | `/review-brd` in progress | `/gen-epics` blocked |
| `Approved` | Reviewed and approved (inline by `/review-brd`) | `/gen-epics` unblocked |
| `Locked` | No further changes without `/assess-change` | Formal change request required |

### Epic
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Generated — not yet reviewed | `/gen-stories` blocked; `/gen-screen-design` blocked |
| `Approved` | Reviewed and approved (inline by `/review-epics`) | `/gen-stories` unblocked; `/gen-screen-design` unblocked |
| `In Progress` | At least one story In Progress | — |
| `Done` | All stories Merged and verified | Dependency epics unblocked |
| `Deprecated` | Descoped — no further work | — |

### Screen Design
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Generated — not yet reviewed | `/gen-architecture` blocked (when UI in scope) |
| `Under Review` | `/review-screen-design` in progress | `/gen-architecture` blocked |
| `Approved` | Reviewed and approved (inline by `/review-screen-design`) | `/gen-architecture` unblocked |

### Architecture
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Being generated or revised | `/implement-story` blocked |
| `Under Review` | `/review-architecture` in progress | `/implement-story` blocked |
| `Locked` | Approved and locked (inline by `/review-architecture`) — is now law | `/implement-story` unblocked; `/gen-sprint-plan` unblocked |

### Story
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Generated — not yet reviewed | `/implement-story` blocked; `/gen-story-plan` blocked |
| `Reviewed` | Reviewed and sprint-assigned (inline by `/review-stories`) | `/gen-story-plan` unblocked; sprint inclusion permitted. `/implement-story` still requires a separate Confirmed story plan (`FW-031`) |
| `In Progress` | Engineering Agent implementing | — |
| `PR Created` | PR open — awaiting PO review | — |
| `Changes Requested` | PO requested revisions | Agent revises on same branch |
| `Merged` | PR merged to main branch | When all sprint stories Merged: Step 23 QA confirmation, then `/gen-uat-checklist` |
| `Done` | ACs verified — final state | Sprint Complete when all stories Done |
| `Deprecated` | Requirement removed | — |

### Story Plan (`FW-031`)
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Draft` | Generated by `/gen-story-plan` — awaiting PO review | `/implement-story` blocked (hard gate, no override) |
| `Confirmed` | PO reviewed | `/implement-story` unblocked for this story |

### Sprint
| Status | Meaning | Gate effect |
|--------|---------|-------------|
| `Planning` | Sprint plan being assembled | No implementation |
| `Active` | Sprint in progress | Implementation running |
| `Complete` | All stories Done, retro filled | `/gen-release-notes` unblocked |

---

## Gate Summary

| Gate | Artifact | Approved by | Blocks |
|------|---------|-------------|--------|
| 1 | Brief | PO | `/gen-domain-knowledge` |
| 2 | BRD | PO (inline via `/review-brd`) | `/gen-epics` |
| 3 | Epics (all) | PO (inline via `/review-epics`) | `/gen-screen-design` |
| 4 | Screen Design | PO (inline via `/review-screen-design`) | `/gen-architecture` (when UI in scope) |
| 5 | Architecture (review + lock) | PO (inline via `/review-architecture`) | `/implement-story`, `/gen-sprint-plan` |
| 6 | Stories (per wave) | PO (inline via `/review-stories`) | `/implement-story` for that wave |
| 7 | Sprint Activation | PO | Implementation |
| 7a | Story Plan (per story) | PO | `/implement-story` for that story (hard gate, no override — see Status Values Reference above) |
| 8 | PR (per story) | PO | Story → Merged |
| 9 | QA Confirmation (sprint-wide) | PO + AI | UAT checklist generation |
| 10 | UAT sign-off | PO | Sprint → Complete |
| 11 | Sprint Complete | PO | `/gen-release-notes` |
| 12 | Release Notes | PO | Production deployment |
| 13 | Production Deployment | PO | Goes live |
| 14 | Hotfix PR (if triggered) | PO | Hotfix deployed |
| 15 | Hotfix Deployment (if triggered) | PO | Hotfix goes live |

---

## How to Resume a Mid-Project Session

**This entire procedure is what `/anchor-project {PROJECT_CODE}` automates** — steps 1–5 below map directly onto its Step 3 (Load and reconcile anchor state), which runs this same walk of the Gate Summary table and writes the result to `.anchor-state.md` so it doesn't need re-deriving by hand on the next resume. Manual and automated are both fully valid; use this procedure directly when you want full control or aren't using anchor for this project.

1. Run `/project-status projects/{PROJECT_CODE}` — see current state of all epics, stories, and sprints
2. Check `projects/{PROJECT_CODE}/brief.md` and `projects/index.md` for document/project statuses
3. Find the first step in this document where the gate has not been closed
4. Read the artifacts for that step and the step before it to rebuild context
5. Resume from that step

**Context loading order (minimum):**
- `projects/{PROJECT_CODE}/brief.md`
- `projects/{PROJECT_CODE}/brds/brd-core-v1.md` (approved sections)
- `projects/{PROJECT_CODE}/architecture/arch-v1-ref.md` (if Locked — read this first)
- `projects/{PROJECT_CODE}/architecture/arch-v1.md` (if Locked — full document; only when ref file is insufficient)
- The current sprint plan (if Active)
- The story you are about to work on

---

## Artifact Location Map

```
projects/{PROJECT_CODE}/
├── brief.md                                  # Project brief (Intake)
├── domain/
│   ├── domain-discovery-state.md             # Domain discovery session state (optional)
│   ├── {domain-slug}-domain-playbook.md      # Domain discovery playbook (optional)
│   └── {domain-slug}-core.md                 # Domain knowledge (+ extension variants)
├── brds/
│   ├── {domain-slug}-brd-playbook.md         # BRD discovery playbook
│   ├── brd-discovery-state.md                # BRD discovery session state
│   └── brd-core-v1.md                        # Business Requirements Document
├── epics/
│   ├── index.md                              # Epic registry + REQ coverage + dependency graph
│   ├── review-record.md                      # Review session tracking (not a deliverable)
│   └── EPIC-{NNN}-{slug}.md                  # One file per epic
├── screens/
│   └── screen-design.md                      # Portals, Navigation Map, Screen Inventory
├── mocks/
│   └── {portal-slug}-mock.html               # One file per portal
├── architecture/
│   ├── arch-v1.md                            # Architecture document (locked)
│   └── arch-v1-ref.md                        # Compact quick-reference
├── standards/                                # Confirmed tech stack, standards files (PO/AI-authored)
├── stories/
│   ├── index.md                              # Story registry + Section 8 sprint planning view
│   ├── review-record.md                      # Review session tracking (not a deliverable)
│   └── EPIC-{NNN}/
│       └── US-{epic}-{seq}-{slug}.md         # One file per story
├── sprints/
│   ├── sprint-{NN}.md                        # Sprint plan + retrospective
│   └── sprint-{NN}-uat-checklist.md          # UAT checklist per sprint
├── test-reports/
│   └── US-{epic}-{seq}-verification.md       # Story verification reports
├── pr/
│   └── US-{epic}-{seq}-pr-description.md     # PR description per story
└── releases/
    └── v{version}-release-notes.md           # Release notes per version

projects/index.md                             # Global project registry (not per-project)
```

---

## Completeness Check

All 33 commands in `.claude/commands/`, cross-checked against this document:

| Command | Documented at |
|---------|--------------|
| `init-project.md` | Step 1 |
| `run-priming-session.md` | Optional, repeatable, Problem Domain only — see the note directly after the Cross-cutting utilities list, below the Lifecycle at a Glance table |
| `run-intake.md` | Step 1a |
| `gen-domain-playbook.md` | Step 2a |
| `run-domain-discovery.md` | Step 2b |
| `gen-domain-knowledge.md` | Step 2 |
| `gen-brd-playbook.md` | Step 3 |
| `run-mock-discovery.md` | Step 3a |
| `run-brd-discovery.md` | Step 4 |
| `gen-brd.md` | Step 5 |
| `review-brd.md` | Step 5a/6 |
| `gen-epics.md` | Step 7 |
| `review-epics.md` | Step 8/9 |
| `gen-screen-design.md` | Step 9a |
| `review-screen-design.md` | Step 9b/9c |
| `gen-ui-mocks.md` | Invoked by Steps 9a and 10 (not standalone) |
| `gen-architecture.md` | Step 10 |
| `review-architecture.md` | Step 11/12 |
| `gen-stories.md` | Step 13 |
| `review-stories.md` | Step 14/15 |
| `gen-sprint-plan.md` | Step 16 |
| `gen-story-plan.md` | Step 17a |
| `implement-story.md` | Step 18 |
| `verify-story.md` | Step 19 |
| `gen-pr-description.md` | Step 20 |
| `assess-change.md` | Step 22 (cross-cutting) |
| `gen-uat-checklist.md` | Step 24 |
| `close-sprint.md` | Step 26 |
| `gen-release-notes.md` | Step 28 |
| `update-status.md` | Every gate step (cross-cutting) |
| `project-status.md` | Utility (cross-cutting) |
| `judgment-check.md` | Utility (cross-cutting) — before any `/review-X` |
| `anchor-project.md` | "Running This Sequence via `/anchor-project`" (orchestrates every step above; not tied to one) |

**Result: all 33 commands accounted for.** Four were entirely missing before the original rewrite (`gen-domain-playbook`, `run-domain-discovery`, `run-mock-discovery`, `gen-brd-playbook` — the last was previously referenced under a nonexistent `/gen-playbook` name), and `run-intake` was missing along with the brief-approval gate it feeds. `gen-story-plan.md` was added under `FW-031`, splitting `/implement-story`'s prior all-in-one design+execution responsibility into a PO-reviewable planning step (Step 17a) and a plan-executing step (Step 18). `judgment-check.md` was also previously missing from this specific table despite already being live and listed in the Command Reference above — an inconsistency within this same document, now corrected alongside adding `anchor-project.md` and, later the same day, `run-priming-session.md` once ingestion moved out of anchor into its own command. No manual/no-command steps (Sprint-wide QA Confirmation, PO UAT, Production Deployment, Defect Triage, Project Closure) have a corresponding command file, by design — they're deliberate human checkpoints.
