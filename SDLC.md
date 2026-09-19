# Software Delivery Lifecycle (SDLC)

## Purpose

Defines the twelve stages of every software project delivered by Ascendra. Stages are strictly sequential — no stage begins until the previous stage's exit criteria are met and any required gate is closed.

**How to read this document:** This is the conceptual stage map — it describes *what* happens and *why* at each stage, entry and exit criteria, and what is produced. For the operational detail (which slash command runs, which template is used, which status value to set), see [`PROJECT-LIFECYCLE.md`](PROJECT-LIFECYCLE.md).

**Mode of operation:** All AI work is executed by Claude in chat mode using slash commands (`/gen-brd`, `/gen-epics`, etc.), not by autonomous agents. "Product Owner" is the human approver at every gate — the person with authority over scope, quality, and release decisions.

> **On agent-based implementation (T-10):** a specialist-agent model for Stage 8 (Development) was explored in depth and parked — the framework retains chat-mode command execution until it's more mature. See [`decisions/FW-011-agent-based-implementation.md`](decisions/FW-011-agent-based-implementation.md) (the full design doc it references is preserved in git history, not the working tree) if this direction is revisited later. Treat this line and Stage 8 below as the current, confirmed model, not a placeholder.

**Stages 1–6 run once per project.** Stage 7's story-generation half repeats per **wave** (normally one epic — see Stage 7 for what a wave is and why it isn't always one sprint); its sprint-plan half, and Stages 8–11, repeat per sprint, in sequence, until all epics are done and the project enters Stage 12.

**Cross-cutting, not tied to any one stage:** `/assess-change` (assesses and applies a scope change's blast radius across the document pipeline), `update-status` (records every gate decision — referenced throughout the stages below), `/project-status` (live dashboard of epic/story/sprint state, run at any time). See [`PROJECT-LIFECYCLE.md`](PROJECT-LIFECYCLE.md) for full detail on all three.

---

## Stage 1 — Intake

**Entry:** A new project is identified and ready to begin.

**Executed by:** Product Owner (with AI assistance via `/init-project`, then `/run-intake`)

**What happens:**
- Project directory is created with a unique project code, via `/init-project`; brief placeholders come from `projects/TEMPLATE/brief.template.md`
- Project is registered in `projects/index.md` — the single source of project identity (`FW-027`), no separate `project.json`
- Brief is completed interactively via `/run-intake`: problem statement, business goals, scope, constraints, budget/timeline — one section at a time, written to `brief.md` immediately on Product Owner approval so progress is never lost across sessions
- Product Owner approves the completed brief (`update-status ... Approved`) — this sets `brief.md`'s **Content Status** field (document readiness), a hard gate `/gen-domain-knowledge` checks for. This is distinct from `brief.md`'s separate **Status** field (the project's overall lifecycle — `Active`/`On Hold`/`Complete`/`Cancelled`, kept in sync with `projects/index.md`); the two fields never share a value.

**Exit criteria:**
- `brief.md` fully complete and Content Status = `Approved`
- Project registered in index

**Artifacts produced:** `projects/{CODE}/brief.md`

---

## Stage 2 — Domain Discovery

**Entry:** Brief Approved.

**Executed by:** Product Owner (with AI assistance via `/gen-domain-knowledge`, and — optionally — `/gen-domain-playbook`, `/run-domain-discovery`)

**What happens:**
- **Domain knowledge** is generated via `/gen-domain-knowledge`: universal facts about the business domain, standard entities, lifecycle models, business rules, and known variations that must be asked during discovery. For well-known domains (invoicing, payroll, school fees, HR), AI training knowledge is sufficient on its own. For niche, regulated, or unfamiliar domains, an **optional** dedicated sub-flow runs first: `/gen-domain-playbook` produces a playbook that guides a domain-education session, then `/run-domain-discovery` conducts it (tracking its own Domain Confidence Score) and writes a discovery-state file that `/gen-domain-knowledge` uses as its primary input (`FW-013` — this optionality is specific to domain knowledge; Screen Design, later, has no equivalent skip).

**Exit criteria:**
- Domain knowledge document written
- Domain Confidence Score at the required threshold, if the optional sub-flow ran

**Artifacts produced:** `projects/{CODE}/domain/{domain}-core.md`, `projects/{CODE}/domain/{domain}-domain-playbook.md` and `domain-discovery-state.md` (if the optional sub-flow ran)

---

## Stage 3 — BRD & Scope Lock

**Entry:** Domain Discovery complete.

**Executed by:** Product Owner (with AI assistance via `/gen-brd-playbook`, `/run-brd-discovery`, and — optionally — `/run-mock-discovery`), AI (via `/gen-brd` or as continuation of `/run-brd-discovery`, then `/review-brd`), Product Owner (approves)

**What happens:**
- **BRD discovery playbook** is generated via `/gen-brd-playbook` from the domain knowledge document: structured questions, scope risk flags, and journey walkthroughs tailored to this domain — the active document used live during the real requirement-discovery session below. Not the same command as `/gen-domain-playbook` from Stage 2 — the two produce different playbooks for different sessions.
- **Optionally**, `/run-mock-discovery` simulates both sides of a discovery session internally (AI-as-facilitator and AI-as-simulated-client) to catch playbook/domain-knowledge gaps before real client cost — recommended, not required.
- **Requirement discovery session** is conducted interactively via `/run-brd-discovery`, using the BRD playbook — AI asks questions, records answers, tracks its own Requirement Confidence Score, flags scope risks in real time. For projects with no external client, this runs as AI-synthesis-and-review: the AI proposes an answer grounded in the domain knowledge document's defaults, and the Product Owner reviews/corrects/overrides.
- At session end, discovery state is saved to `brd-discovery-state.md`; if confidence ≥ 85%, AI immediately offers to generate the BRD in the same session (recommended — full context is live)
- BRD is produced from discovery notes, following `projects/TEMPLATE/brds/brd.template.md`: user roles, user journeys, functional requirements, business rules, data requirements, integrations, security requirements
- `/review-brd` walks the Product Owner through the BRD feature-area by feature-area in plain business language — not a report to read cold. Changes are applied immediately on agreement; progress is tracked in the BRD's own review record, resumable across sessions.
- **The review command sets approval inline:** when the Product Owner types `'approved'`, `/review-brd` itself sets BRD status to `Approved` — there is no separate manual status-setting step in the normal path
- On approval, scope is locked — no changes without a formal change request via `/assess-change`

**Exit criteria:**
- Requirement confidence score ≥ 85% (from `/run-brd-discovery`, distinct from Domain Discovery's own confidence score)
- All BRD playbook sections covered
- Discovery state file written
- BRD status set to `Approved`
- All open questions in BRD Section 12 have owners or are resolved

**Artifacts produced:** `projects/{CODE}/brds/{domain}-brd-playbook.md`, `projects/{CODE}/brds/brd-discovery-state.md`, `projects/{CODE}/brd-v1.md`

---

## Stage 4 — Product Structuring

**Entry:** BRD Approved.

**Executed by:** AI (via `/gen-epics`, `/review-epics`), Product Owner (approves each epic during the review)

**What happens:**
- BRD requirements are grouped into epics — cohesive delivery units, each representing a complete, independently useful capability — for **all layers of the approved BRD at once** (epics have no architecture dependency, so full v1 scope is safe to elaborate before any sprint exists)
- Each epic is generated following `projects/TEMPLATE/epics/epic.template.md`: goal, BRD requirements covered, scope in/out, personas, dependencies, definition of done
- `/review-epics` walks through all epics one at a time against the BRD, checking coverage gaps, scope boundary issues, and dependency ordering. **Sets each epic's status to `Approved` inline** as part of the walkthrough — no separate manual status-setting step in the normal path. No story generation begins until an epic is `Approved`.
- Note: **stories are not generated in this stage** — they are generated one wave at a time after architecture is locked (Stage 7), so they can be written with full technical context and without committing the whole backlog to detail before any sprint has delivered learnings

**Exit criteria:**
- All epics approved
- REQ coverage map confirms every BRD requirement is assigned to exactly one epic
- Dependency order recorded in `epics/index.md`

**Artifacts produced:** `projects/{CODE}/epics/EPIC-{NNN}.md` (one per epic), `projects/{CODE}/epics/index.md`

---

## Stage 5 — Screen Design

**Entry:** All epics Approved.

**Executed by:** AI (via `/gen-screen-design`, `/review-screen-design`), Product Owner (approves)

**What happens:**
- Portals, Navigation Map, and Screen Inventory are derived from the BRD alone — Personas, Roles, Journeys, FRs — with no dependency on the confirmed tech stack, following `projects/TEMPLATE/screens/screen-design.template.md`
- Every screen is pattern-matched against `ascendra-ui`'s real component catalog: assigned a Surface (Page/Dialog/Sheet/Drawer) and UI Pattern (Form/Dashboard/Report/Table-List/Detail), each cited to a specific named catalog entry or primitive composition — never an invented shape
- Per-screen data requirements are recorded — this becomes the input for Architecture's Data Model and API Contracts
- `/gen-ui-mocks` is invoked in base mode (not run standalone) right after `screen-design.md` is written — real `ascendra-ui` component-source fidelity, not just design tokens, so the Product Owner can review actual navigation and screen layout before architecture begins
- `/review-screen-design` walks through every screen against the BRD, mirroring `/review-epics`'s structure. **Sets status to `Approved` inline** on the Product Owner's confirmation — no separate manual status-setting step in the normal path. Skipped entirely (not applicable) if the BRD has no UI in scope.
- See [`decisions/FW-026-screen-design-phase.md`](decisions/FW-026-screen-design-phase.md) for why this runs before Architecture rather than after

**Exit criteria:**
- Screen Design status set to `Approved` (or phase skipped — no UI in scope)
- Every BRD journey maps to at least one screen
- Every screen has a cited Surface + UI Pattern match

**Artifacts produced:** `projects/{CODE}/screens/screen-design.md`, `projects/{CODE}/mocks/{portal-slug}-mock.html` (one per portal)

---

## Stage 6 — Architecture

**Entry:** All epics Approved. Screen Design Approved (when the BRD has a UI in scope).

**Executed by:** AI (via `/gen-architecture`), Product Owner (approves and locks)

**What happens:**
- Architecture document is produced following `projects/TEMPLATE/architecture/arch.template.md`
- Tech stack, standards files, internal layering pattern, and repo topology are recommended **on the fly per project** — there is no fixed "Ascendra standard stack" (`FW-025`); NestJS + Next.js/Ascendra UI is the common default and fallback, not a mandate
- Covers: system overview, tech stack, all module definitions, complete database schema (tables, columns, enums, indexes, seed data — cross-referenced against Screen Design's per-screen data requirements), all API endpoint contracts with DTOs, security design (auth, RBAC, delegated access, sensitive fields), infrastructure plan, and all environment variables
- Screen & Navigation Map (Portals, Navigation Map, Screen Inventory) is carried over from the approved Screen Design, not re-derived; only Action Gating and Cross-Cutting UI Rules are generated fresh here, once the security model exists — `/gen-ui-mocks` is then invoked in patch mode (its second of two invocations) to add gating captions and cross-cutting banners to the base mocks from Stage 5
- All decisions conform to architecture standards in `projects/{CODE}/standards/`
- Deviations from the standard stack are declared in Section 9 of the architecture document — never made silently
- `/review-architecture` runs 35 structural checks across 5 concerns, presented as a guided per-concern walkthrough. On the Product Owner typing `lock`, **sets architecture status to `Locked` inline** — as the command works today this is a bare `lock`/`not yet` confirmation with no rationale captured (an earlier version of this document implied otherwise; unresolved whether that was a deliberate cut or lost by accident)
- Once Locked, the architecture document is law — the Engineering Agent implements exactly what it defines. Undocumented deviations are not permitted.

**Exit criteria:**
- Architecture status = `Locked`
- No open decisions in Section 8 of the architecture document

**Artifacts produced:** `projects/{CODE}/architecture/arch-v1.md`

---

## Stage 7 — Sprint Planning

**Entry:** Architecture Locked. Story generation repeats per **wave**; sprint-plan generation repeats per sprint.

**Executed by:** AI (via `/gen-stories`, `/review-stories`, `/gen-sprint-plan`), Product Owner (approves stories during review, activates sprint)

**What happens:**
- Stories are generated **one wave at a time** — normally one epic, following `projects/TEMPLATE/stories/story.template.md` and the project's chosen `reference/estimation/` model (copied to `projects/{PROJECT_CODE}/standards/estimation.md` on first run). A wave is more than one epic only when `epics/index.md` documents a story-level interleaving between them (their true build order doesn't reduce to a clean epic-level sequence) — never the whole remaining backlog at once. This is deliberate, not just a convenience: generating stories far ahead of delivery bakes in assumptions before any sprint has produced real learnings to correct them.
- Stories are technology-agnostic — they define *what* the system must do, not *how*; no field names, table names, routes, or library names in acceptance criteria
- `/review-stories` reviews stories for AC quality, scope accuracy, size, and dependency resolution. **Sets each story to `Reviewed` inline** as part of the walkthrough. It also runs **Sprint Assignment** — the step that actually decides sprint boundaries, using this wave's real story count against sprint capacity: assigns a sprint if at/near target, pulls in the next unblocked epic (or tops up with earlier leftover stories) if below target, or asks the Product Owner for a split boundary if a single epic exceeds capacity. Sprint membership is never known or fixed at story-generation time, and is never encoded in the Story ID.
- Sprint plan is generated: goal, readiness checklist, story board, delivery sequence, capacity analysis, Should Have decisions
- Product Owner confirms all readiness gate checks pass and activates the sprint

**Exit criteria:**
- All sprint stories in `Reviewed` status and sprint-assigned
- Sprint plan document produced
- Sprint status set to `Active`

**Artifacts produced:** `projects/{CODE}/stories/EPIC-{NNN}/US-{epic}-{seq}.md` (one per story, epic-numbered — stable at generation, since sprint membership isn't), updated `stories/index.md`, `projects/{CODE}/sprints/sprint-{NN}.md`

---

## Stage 8 — Development

> **On agent-based implementation (T-10):** described below is the current, confirmed single-command model. A master-orchestrator/specialist-agent redesign was explored and parked (see [`decisions/FW-011-agent-based-implementation.md`](decisions/FW-011-agent-based-implementation.md)) — not scheduled, may be revisited once the framework is more mature.

**Entry:** Sprint Active. Repeats for each story in the sprint.

**Executed by:** AI (via `/gen-story-plan`, `/implement-story`, `/verify-story`, `/gen-pr-description`), Product Owner (confirms each story's plan, approves each PR)

**What happens:**
- Stories are planned and implemented one at a time in the delivery sequence from the sprint plan
- `/gen-story-plan` runs first for each story (`FW-031`): it extracts — never invents — exactly which endpoints, tables, and screens this one story needs from the already-Locked architecture, builds an AC-to-artifact traceability table and a per-AC test plan, and writes it out for Product Owner review. This splits what was previously one all-in-one command into a reviewable design step and a plan-executing step — the PO catches a wrong interpretation before any code exists, which is far cheaper to fix than after. `/implement-story` hard-gates on this plan being Confirmed; there is no override
- `/implement-story` reads the confirmed plan as its task list, and the locked architecture document for how to build it in this project's confirmed stack idiom; determines the confirmed stack from Section 2 first (NestJS/Next.js is the common default and this project's actual choice, never a universal assumption — `FW-025`, `FW-029`) and the target repo (API / Web / Worker, or more than one) from Section 3.1
- If the target repo doesn't exist yet, `/implement-story` scaffolds it itself the first time any story touches it (e.g. `nest new`, or cloning `ascendra-ui` for Next.js) — resolving what would otherwise be a circular dependency (a dedicated "scaffolding story" can't run before the repo it's meant to create already exists)
- Creates a story branch (`story/{STORY-ID}-{slug}`) per `standards/git-standards.md` before writing any code; implements exactly the ACs, no more; lint, build, and tests pass before committing
- `/verify-story` tests each acceptance criterion against the running application (black-box — HTTP/UI, not stack-specific) using the plan's own Test Scenarios table where one exists, and adds a Plan Conformance check (declared artifacts vs. what was actually delivered) alongside the usual AC pass/fail; produces a verification report
- `/gen-pr-description` generates the PR title and description from the real code diff and the passing verification report — not from the ACs alone — for the Product Owner to use raising the PR
- Product Owner approves → story merged; requests changes → agent revises on same branch; rejects → agent reimplements from scratch

**Exit criteria:**
- All sprint stories in `Merged` status
- All PRs closed
- Build passing

**Artifacts produced:** Story implementation plans in `projects/{CODE}/story-plans/`, code committed and merged to main branch (per repo touched), PR descriptions in `projects/{CODE}/pr/`, verification reports in `projects/{CODE}/test-reports/`

---

## Stage 9 — Testing & QA

**Entry:** All sprint stories Merged. Runs once per sprint, before UAT.

**Executed by:** AI + Product Owner

**What happens:**
- Full test suite is run across the entire codebase: unit tests, integration tests, and any E2E tests
- Results of all `/verify-story` reports are reviewed — confirm zero open Critical or High defects
- Test coverage is confirmed to meet the project threshold (target ≥ 80% per the test strategy in `projects/{CODE}/standards/test-strategy.md`)
- Any defect found at this stage is classified using `reference/defect-severity.md`: Critical/High → resolved before UAT; Medium/Low → logged and triaged (may proceed to UAT with PO approval)
- QA confirmation is recorded (a note in the sprint plan's Sprint Notes section suffices; a full test execution report using `projects/TEMPLATE/test-reports/test-execution-report.template.md`, aggregated from every story's `/verify-story` report, is required for client-facing projects)

**Exit criteria:**
- Full test suite passing
- Zero open Critical or High defects
- Coverage threshold met

**Artifacts produced:** QA confirmation note in sprint plan; optionally `test-execution-report.md`

---

## Stage 10 — UAT

**Entry:** QA confirmation recorded.

**Executed by:** AI (generates checklist via `/gen-uat-checklist`), Product Owner (executes UAT)

**What happens:**
- UAT checklist is generated: one test scenario per acceptance criterion across all sprint stories, written in plain language for the Product Owner (or client)
- Product Owner works through each scenario, marks Pass or Fail, records observations
- Any Fail raises a defect: Critical/High → returns to Stage 8 (fix, re-verify, re-PR) then Stage 9, then Stage 10 again; Medium/Low → logged, deferred to a future sprint at PO's discretion
- UAT sign-off: Product Owner signs the checklist indicating all Must Have scenarios passed

**Exit criteria:**
- All Must Have scenario results: Pass
- Zero open Critical or High defects
- UAT checklist signed off

**Artifacts produced:** `projects/{CODE}/sprints/sprint-{NN}-uat-checklist.md` (completed and signed)

---

## Stage 11 — Sprint Closure & Release

**Entry:** UAT sign-off complete.

**Executed by:** AI (via `/close-sprint`, `/gen-release-notes`), Product Owner (closes sprint, approves release, triggers deployment)

**What happens:**
- Sprint retrospective is filled in: what went well, what to improve, carry-forward items, velocity
- `/close-sprint` also checks for **forward-looking learnings**: whether anything from this sprint's delivery, verification, or UAT should correct a not-yet-built epic before the next wave's stories are generated against possibly-stale scope. If so, it points at `/assess-change` — the retrospective itself is process-only and does not correct requirements on its own.
- Sprint status set to `Complete`
- Release notes are generated following `projects/TEMPLATE/releases/release-notes.template.md`: plain-language description of every user-visible change, known limitations, deployment notes — keyed by version, not sprint number, since a release doesn't always map 1:1 to a sprint
- Product Owner approves release notes
- Production deployment is executed; post-deployment smoke tests confirm the release is healthy
- Client (or PO if internal project) is notified of go-live

**Exit criteria:**
- Sprint status = `Complete`
- Production deployment verified healthy
- Release notes approved and delivered

**Artifacts produced:** Updated sprint plan (retrospective complete), `projects/{CODE}/releases/v{version}-release-notes.md`, deployment record

---

## Stage 12 — Support

**Entry:** Successful production release. Entered once per project (not per sprint), after the final sprint's production deployment.

**Executed by:** Product Owner (monitors, triages), AI (assists with defect investigation on request)

**What happens:**
- Post-release monitoring period begins (default 30 days; client agreement or BRD Section 10 may override)
- Defects reported in production are logged and classified:
  - **Critical / High** → Hotfix Cycle triggered immediately (see below)
  - **Medium / Low** → batched into a future sprint; do not trigger a hotfix
- At the end of the support period: project is either closed (brief status set to `Complete`) or handed over to a retainer agreement
- Formal closure document is produced if the project is closing

**Exit criteria:**
- Support period elapsed
- Zero open Critical or High defects
- Project status set to `Complete` or retainer agreement in place
- Closure document produced if applicable

**Artifacts produced:** Support log, closure document (if closing)

---

## Hotfix Cycle

A hotfix is triggered by a Critical or High severity defect found in production during Stage 12 (Support). It bypasses sprint planning and UAT because the fix is narrow, targeted, and validated before deployment.

**Trigger:** Product Owner classifies a production defect as Critical or High and confirms a hotfix should proceed before work begins.

**Steps (strictly sequential):**

1. Create a `hotfix/{PROJECT_CODE}-{issue-description}` branch from `main` — not from any sprint branch
2. Run `/implement-story` targeting this branch; scope is strictly limited to the defect AC. No other changes.
3. Run `/verify-story` to confirm the defect scenario is fixed and no regressions are introduced in the affected module
4. PR is raised against `main`. Product Owner reviews and approves (same gate as Stage 8 PR review)
5. Full test suite is run on the hotfix branch; targeted regression tests cover the affected module and any integration points touched by the change. Full E2E suite is not required.
6. Product Owner approves production deployment (same gate as Stage 11 deployment — rollback plan required)
7. Production deployment executes; post-deployment smoke tests run automatically
8. The fix is confirmed preserved going forward: story branches are cut from and merged directly into `main` (no persistent sprint-integration branch exists in this framework's git model), so the hotfix merging into `main` in step 4 already reaches every branch cut afterward. The one manual check: any current-sprint story branch already cut from `main` *before* the hotfix merged needs `main` merged or rebased into it before its own PR, so an older branch's diff doesn't silently revert the hotfix on merge.

**What is skipped:** Sprint planning, UAT, full regression suite.

**What is not skipped:** Product Owner approval at PR, Product Owner approval at deployment, targeted QA verification, rollback plan.

---

## Sequential Enforcement

No stage may begin until the preceding stage's exit criteria are met and any required gate is closed. Gate checks are enforced by the AI at the start of every downstream command — each command reads the upstream artifact's status and stops with a clear error if the gate has not been closed. See [`PROJECT-LIFECYCLE.md`](PROJECT-LIFECYCLE.md)'s Gate Summary table for the complete gate reference.
