# The Lifecycle Map — Project Lifecycle & the Anchor-Project Alternative

## What this page is

[`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md) is the framework's authoritative operational reference — every step, the command that runs it, the template it reads, the gate that closes it, for all 33 commands, across 28 numbered steps and 12 stages. It is not optional background reading; it is the document `/anchor-project` itself walks to decide what runs next, and the one you walk by hand ([`How to Resume a Mid-Project Session`](../PROJECT-LIFECYCLE.md#how-to-resume-a-mid-project-session)) if you aren't using anchor. This page doesn't replace it or restate its detail — it's a navigable map of the same 12 stages, laid out so you can see the whole shape of a project at once, with the one thing the source table can't show inline: **where the `/anchor-project` alternative actually changes what a Product Owner has to do, stage by stage.**

Read [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md) itself for the exact arguments, gate checks, and output paths of any one command. Come back to this page when you want the map, not the manual.

## Two ways to run this framework

Every phase chapter in this guide is written against typing each command directly — that's the default assumption throughout, and it stays fully available no matter how experienced you get. `/anchor-project {PROJECT_CODE}` is the other way: one command that resolves or bootstraps a project, then invokes whichever command owns the current stage and executes its instructions exactly as written. Nothing about a command's own behavior, file formats, or gates changes depending on which path invoked it.

| | Typing commands directly | `/anchor-project {PROJECT_CODE}` |
|---|---|---|
| **What you hold in your head** | The full 28-step sequence — what ran last, what's next, which gate is open | Nothing — anchor reconciles its own state against what's actually on disk |
| **Session-to-session continuity** | Manual — re-derive it from `PROJECT-LIFECYCLE.md`'s Gate Summary and your own memory | Automatic — `.anchor-state.md` tracks it, refreshed every resume |
| **Question density at each gate** | Full — every discovery and review command's own dense prompts, as written | Compressed, without lowering the rigor behind them |
| **Drift checking across artifacts** | None built in — you'd have to think to check it yourself | Proactive — version lineage, constraint propagation, Ubiquitous Language, tiered free/cheap/heavy-on-trigger |
| **Priming material handoff** | You re-explain source material to each command yourself | Folded in automatically the moment anchor reaches the stage that needs it |
| **Running record of the project** | None, unless you keep your own notes | `journal.md`, feeding a `testimonial.md` at milestones (`FW-050`) |
| **What actually gets produced** | Identical | Identical — anchor is a sequencing and interaction layer, never a discount on rigor |

**Who should use which:** new to the framework, running your first few projects, or you'd simply rather hold less in your head — start with anchor, that's exactly what it's built for. Already fluent in the sequence and want to control every question and every gate yourself — type the commands directly; nothing about that path is diminished by anchor's existence. Every command remains usable standalone either way, and you can switch paths mid-project without penalty — anchor's own reconciliation step is designed to pick up state left by direct command use.

**Where anchor stops driving:** once a project's Walking Skeleton (`FW-048`) is complete, anchor narrows to release prep and formal scope changes — day-to-day story work (Stage 8's implement → verify → PR loop) reverts to direct command use even on an anchored project. See [`decisions/FW-049-anchor-project-and-priming-session.md`](../decisions/FW-049-anchor-project-and-priming-session.md)'s Solution Domain Boundary section for why.

Design and rationale: [`decisions/FW-049-anchor-project-and-priming-session.md`](../decisions/FW-049-anchor-project-and-priming-session.md) and [`decisions/FW-050-project-journal-and-framework-learning.md`](../decisions/FW-050-project-journal-and-framework-learning.md). The command itself: [`.claude/commands/anchor-project.md`](../.claude/commands/anchor-project.md).

## The 12 stages, at a glance

Three regions, per [`FW-022`](../decisions/FW-022-problem-solution-domain-boundary.md): **Problem Domain** (Stages 1–5) has zero architecture dependency. **Bridge** (Stage 6) is the first hard dependency point. **Solution Domain** (Stages 7–12) requires Locked architecture, directly or transitively, for everything in it.

### Problem Domain

| Stage | Purpose | Key commands | Gate | Repeats? |
|---|---|---|---|---|
| **1 — Intake** | Create the project record and a complete, approved brief | `/init-project` → `/run-intake` | ✅ Brief `Approved` | Once |
| **2 — Domain Discovery** | Build reusable domain knowledge — entities, lifecycles, business rules | (optional) `/gen-domain-playbook` → `/run-domain-discovery` → `/gen-domain-knowledge` | — | Once per domain |
| **3 — BRD & Scope Lock** | Run requirement discovery, then lock scope in an approved BRD | `/gen-brd-playbook` → (optional) `/run-mock-discovery` → `/run-brd-discovery` → `/gen-brd` → `/review-brd` | ✅ BRD `Approved` | Once |
| **4 — Product Structuring** | Break the approved BRD into delivery-sized epics | `/gen-epics` → `/review-epics` | ✅ Epics `Approved` | Once |
| **5 — Screen Design** | Derive Portals, Navigation Map, and Screen Inventory from the BRD alone | `/gen-screen-design` → `/review-screen-design` | ✅ Screen Design `Approved` | Once (skipped if no UI in scope) |

### Bridge

| Stage | Purpose | Key commands | Gate | Repeats? |
|---|---|---|---|---|
| **6 — Architecture** | Define the complete technical blueprint before any code is written | `/gen-architecture` → `/review-architecture` | ✅ Architecture `Locked` | Once |

### Solution Domain

| Stage | Purpose | Key commands | Gate | Repeats? |
|---|---|---|---|---|
| **7 — Sprint Planning** | Generate, review, and sprint-assign stories one wave at a time | `/gen-stories` → `/review-stories` → `/gen-sprint-plan` | ✅ Stories `Reviewed`; ✅ Sprint `Active` | Story steps per wave; sprint-plan per sprint |
| **8 — Development** | Plan, implement, verify, and merge each sprint story | `/gen-story-plan` → `/implement-story` → `/verify-story` → `/gen-pr-description` | ✅ Story Plan `Confirmed`; ✅ PR reviewed | Per story |
| **9 — Testing & QA** | Confirm the sprint is technically sound as a whole before UAT | Manual QA confirmation | ✅ QA confirmation | Per sprint |
| **10 — UAT** | Get PO/client sign-off that every Must Have scenario passes | `/gen-uat-checklist` → manual PO UAT | ✅ UAT sign-off | Per sprint |
| **11 — Sprint Closure & Release** | Close the sprint and generate release artefacts | `/close-sprint` → `/gen-release-notes` → manual deploy | ✅ Sprint `Complete`; ✅ Deployment | Per sprint / per release |
| **12 — Support** | Monitor production, triage defects, close or retain the project | Manual monitoring + Hotfix Cycle (below) | ✅ Project closure | Once, ongoing until closed |

**Cross-cutting, at any stage:** `/assess-change` (scope/impact changes), `update-status` (records every gate decision), `/project-status` (live dashboard), `/judgment-check` (density check before a `/review-X` walkthrough), `/run-priming-session` (Problem Domain only — head start from dropped source material). Full treatment of all five: [`11-cross-cutting.md`](11-cross-cutting.md).

## Where `/anchor-project` changes the day, stage by stage

The commands and gates above are identical either way. What anchor actually removes is different at each stage:

- **Stages 1–5 (Problem Domain):** anchor's biggest lift. These stages are the most conversationally dense in the framework — a brief, a domain discovery session, a full BRD discovery, epic review, screen review — five separate PO-facing walkthroughs in sequence. Anchor compresses the question density across all five without skipping any of them, and folds in anything already captured via `/run-priming-session` the moment each stage needs it.
- **Stage 6 (Bridge):** anchor still invokes `/gen-architecture` and `/review-architecture` unmodified — it does not shrink the 35-check review. What it removes is the PO having to know, unprompted, that this is the one stage where a fresh, otherwise-empty session matters most; anchor sequences it that way by default.
- **Stages 7–11 (Solution Domain):** this is where anchor's role visibly narrows once the Walking Skeleton is complete. Story-by-story work — `/gen-story-plan` → `/implement-story` → `/verify-story` → `/gen-pr-description` — reverts to direct command use even on an anchored project, since this loop runs per story, often across multiple developers in parallel, and anchor's single-threaded sequencing model doesn't fit that shape. Anchor comes back for release prep (Stage 11) and any formal scope change routed through `/assess-change`.
- **Stage 12 (Support):** anchor is not driving here by design (see "Where anchor stops driving" above) — this stage is inherently PO/team-run, not sequence-driven.

## The Hotfix Cycle (Stage 12, as-needed)

Triggered by a Critical or High severity defect in production. Bypasses sprint planning and UAT — narrow scope, validated before deployment: hotfix branch → `/implement-story` → `/verify-story` → PR against `main` → targeted test run → deploy → smoke test. Full sequence: [`PROJECT-LIFECYCLE.md`'s Hotfix Cycle section](../PROJECT-LIFECYCLE.md#hotfix-cycle).

## Where to find more

- [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md) — the full 28-step reference this page maps: exact arguments, gate checks, output paths, the complete Gate Summary and Status Values Reference.
- [`SDLC.md`](../SDLC.md) — the same 12 stages described conceptually, by responsible agent, rather than by command.
- [`decisions/FW-022-problem-solution-domain-boundary.md`](../decisions/FW-022-problem-solution-domain-boundary.md) — why the three domain regions are drawn where they are.
- [`decisions/FW-049-anchor-project-and-priming-session.md`](../decisions/FW-049-anchor-project-and-priming-session.md) and [`decisions/FW-050-project-journal-and-framework-learning.md`](../decisions/FW-050-project-journal-and-framework-learning.md) — the design behind `/anchor-project`.
- [`00-orientation.md`](00-orientation.md) — read first if you haven't; this page assumes its terminology map.
