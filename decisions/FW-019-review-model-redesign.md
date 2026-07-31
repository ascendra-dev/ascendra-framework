# FW-019 — Review Model Redesign

| Field | Value |
|-------|-------|
| **ID** | FW-019 |
| **Date** | 2026-06-29 |
| **Status** | Decided |
| **Area** | Framework-wide — BRD, architecture, epic, and story review phases |

---

## Decision

The review commands (`/review-epics`, `/review-stories`, `/review-architecture`) are redesigned from audit-report generators into sequential, artifact-at-a-time guided walkthroughs. A new `/review-brd` command is added as a missing gate between BRD generation and epic generation. Structural quality checks are embedded in the commands — external process knowledge files are retired. Each artifact is presented with real-world framing so the PO understands what will be built, not just what was written.

`/verify-story` is unchanged — it is already the correct model for its phase.

---

## The Problem That Triggered This Decision

Two distinct problems were identified, both surfacing through the review commands:

### Problem 1 — Structural quality checking is architecturally broken

`/review-stories` loads all epics and all stories before running checks. On large projects (>35 stories), context degrades progressively — the model loses fidelity on early stories by the time it processes the last epic. The block-based session workaround built into the command is a band-aid on an architectural mistake.

The check logic lives in external process files (`knowledge/process/epic-review-process.md`, `knowledge/process/story-review-process.md`) that must be loaded before any review begins, adding context cost and creating a second maintenance location for the same logic. This violates the pattern established by Phases 1–2 commands, which reference only `projects/TEMPLATE/` and project artifacts.

After review, the PO receives a FLAGS report — which is itself a document they must read and interpret. The review process produces a document to approve another document. This is not a well-designed gate.

### Problem 2 — PO comprehension is not addressed

The current model produces artifacts, runs structural checks, and asks the PO to approve. A skilled PO reads a BRD or a 40-story backlog and says "yes" — but approval is not the same as comprehension. The PO may be approving something they have not truly visualised.

There is no mechanism to bridge the gap between a written requirement or story and what it actually produces in the real world. A PO reading "REQ-007: The system shall allow payers to be registered" cannot easily picture whether that means a web form, a CSV import, an API endpoint, or all three — and which stories cover which output. Without this bridge, approvals are nominal.

---

## Root Cause

The review commands conflate two separate concerns:

| Concern | What it needs |
|---------|--------------|
| **Structural quality** — does the artifact comply with template, BRD, domain rules, and layer boundaries? | Machine check, bounded context, immediate FLAGS |
| **PO comprehension** — does the PO understand what this artifact means and what it will produce? | Guided walkthrough, real-world framing, conversational confirmation |

The current commands attempt to serve both with a single mechanism (load everything, run checks, produce report) and serve neither well.

---

## The Redesigned Review Model

### Principle 1 — One artifact at a time

No review command loads the full project artifact set. Context is bounded per artifact:

| Reviewing | Load |
|-----------|------|
| One BRD requirement | That requirement section + relevant discovery state topics |
| One epic | That epic file + BRD sections it covers + domain knowledge relevant to it |
| One story | That story + its parent epic + the BRD REQs it references + applicable domain rules |

Project size has no bearing on context usage. A 100-story project reviews at the same fidelity as a 10-story project.

### Principle 2 — Structural checks embedded, not externalised

Check logic moves into the command files. External process files (`epic-review-process.md`, `story-review-process.md`) are retired. The checks do not change — only their location. The benefit: one place to read, one place to maintain, no external file load cost.

### Principle 3 — Walkthrough, not report

The AI does not produce a FLAGS document the PO reads after the fact. Instead, FLAGS surface as part of the walkthrough — the PO encounters an issue in context, as part of reviewing the artifact, not as a separate output. The PO's review and the quality check run interleaved, not sequentially.

### Principle 4 — Real-world framing

For every artifact, the AI classifies what it will produce and frames it in PO language. Classification is shown to the PO for confirmation — it is a best-effort inference, not a guaranteed output.

| Output type | Real-world framing the AI presents |
|-------------|-----------------------------------|
| UI screen / form | "You will see: a form with fields [Name (required), Email (required), Phone (optional)] and a Submit button. On success, the payer appears in the payer list." |
| REST endpoint | "This produces: `POST /api/payers` — accepts name, email, phone. Returns the created payer. Used by the registration form." |
| Database schema | "This creates: a `payers` table with columns [id, org\_id, name, email, phone, created\_at, updated\_at]. Referenced by invoices and payments." |
| Background job | "This runs automatically: nightly at midnight, scans unpaid invoices past due date, sends a reminder email to each payer." |
| Business rule enforcement | "This adds a constraint: a payer with the same email cannot be registered twice in the same organisation. The system rejects the duplicate with a clear error message." |
| Combined (screen + endpoint) | Both described — AI notes the story produces a UI and the API call it triggers |

If the real-world description cannot be derived from the acceptance criteria — because the ACs are too vague — that surfaces as a FLAG: the story is incomplete.

### Principle 5 — Clarification is part of the review

The PO can ask questions during the walkthrough. The AI answers from the artifact content. If the answer is not in the artifact, that is a FLAG — the artifact needs to be updated before the walkthrough continues. The review is a dialogue, not a one-way presentation.

---

## Command-by-Command Redesign

### `/review-brd` — new command

**What it is:** A structured walkthrough of the approved BRD, requirement by requirement, before epic generation begins. Currently this gate does not exist — the PO reads the BRD and approves it directly from `/gen-brd`. This is the highest-value artifact in the lifecycle and the weakest review point.

**How it works:**
1. Reads one requirement section at a time
2. Confirms the requirement maps back to a topic covered in the discovery session
3. Frames it as a user-facing capability: "When REQ-007 is built, the Finance Manager can register a new payer from the dashboard — they fill in name and email, submit, and the payer is saved. Does this match what you discussed in the discovery session?"
4. PO confirms or challenges
5. If challenged: AI revises the requirement inline, confirms with PO, then continues
6. After all requirements: verification summary — any requirements not traceable to discovery topics are flagged as potential scope additions (not gaps in discovery)

**Gate:** BRD must be at `Draft` or `Under Review` status. After walkthrough completion and PO approval, status updates to `Approved`.

---

### `/review-epics` — redesigned

**Current model:** Loads all epics + BRD + domain knowledge → runs 11 checks → produces FLAGS report per epic → PO reviews report.

**Redesigned model:** One epic at a time. For each epic:

1. **Structural check** (embedded, not from external file): REQ coverage accuracy, in-scope granularity, out-of-scope completeness, persona correctness, dependency accuracy, domain rule presence in Notes, layer boundary. FLAGS surface immediately.
2. **Capability walkthrough**: "When EPIC-003 is complete, the Finance Manager can manage all payers — register, search, view history, deactivate. This epic covers [N] requirements from the BRD and will produce [N] stories. Does this match your expectation for this feature area?"
3. **PO interaction**: Confirm or challenge. If challenged, structural revision or scope adjustment. AI executes changes, confirms, continues.
4. **PO review pack**: The questions only a human can answer — surfaced at the end of the epic walkthrough, not as a separate document. One or two questions maximum, targeted.

**Context load:** That epic + its BRD sections + relevant domain knowledge. No other epics loaded.

---

### `/review-stories` — redesigned

**Current model:** Loads all stories + all epics + BRD + domain knowledge → 10 story-level checks + 5 epic-level checks → FLAGS report → PO reviews report → block-based workaround for large projects.

**Redesigned model:** One story at a time, within one epic at a time. For each story:

1. **Structural check** (embedded): AC completeness, domain rule coverage, dependency correctness, size validity, out-of-scope integrity, persona accuracy, FR traceability, layer match. FLAGS surface immediately — AI applies Tier 1 fixes inline, presents Tier 2 revisions for confirmation, halts on Tier 3 structural issues.
2. **Real-world framing**: AI classifies the output type and presents what this story will produce (see Principle 4 above). PO confirms classification is correct.
3. **Clarification window**: PO can ask "what happens if X?" — AI answers from ACs. If the answer is absent from ACs, it is a FLAG.
4. **Confirm and advance**: PO confirms the story or requests a change. AI applies, confirms, moves to next story.

**Epic-level checks** (run after all stories in an epic are walked through): cross-story coverage completeness, no scope duplication, no AC duplication, dependency chain coherence, sprint capacity. These are structural and run as a batch at epic boundary, not story-by-story.

**Context load per story:** That story + parent epic + referenced BRD REQs + applicable domain rules. No other stories loaded.

---

### `/review-architecture` — lighter redesign

The architecture is a single document, not a collection, so context overload is not the primary issue. The redesign focuses on adding the real-world framing layer:

- **Section 3 (Module Structure):** "These are the [N] modules the system will be built from. Each handles one bounded area. Does this breakdown match how you think about the system?"
- **Section 4 (Data Model):** For each table — "This stores [entity]. Key fields: [list]. Used by [modules]." PO confirms the data model reflects the business entities from the domain knowledge.
- **Section 5 (API Endpoints):** For each endpoint group — "These endpoints are what the frontend will call to [capability]. They match requirements [REQ-IDs]."
- **Section 9 (Deviations):** Each deviation explained plainly — "We're not using [standard component] because [reason]. Instead: [alternative]. This was flagged for your approval." PO confirms each deviation.

Structural checks remain (tech stack completeness, module ownership, endpoint coverage, environment variable completeness). FLAG mechanism unchanged.

---

### `/verify-story` — no change

`/verify-story` is already correctly modelled: it tests a running system against acceptance criteria, classifies defects by severity, and produces a test execution report. It is an implementation phase command, not a document review command. The redesign of review commands does not affect it.

---

## What Gets Retired

| Artifact | Disposition |
|----------|-------------|
| `knowledge/process/epic-review-process.md` | Check logic embedded in redesigned `/review-epics`. File retired. |
| `knowledge/process/story-review-process.md` | Check logic embedded in redesigned `/review-stories`. File retired. |
| Block-based session architecture in `/review-stories` | Made redundant by one-story-at-a-time loading. Removed. |
| FLAGS report as primary PO-facing output | Replaced by interleaved walkthrough. FLAGS become inline dialogue, not a separate document. |

---

## What Does Not Change

- The structural checks themselves — what is checked does not change, only where the logic lives and how it is surfaced
- The tier system for fixes (Tier 1 inline, Tier 2 present and confirm, Tier 3 PO decides, Tier 4 halt) — this remains the right classification
- The status update mechanism — artifacts move from Draft → Reviewed → Approved through the same transitions
- `/verify-story` — unchanged

---

## Dependency and Sequencing Notes

The redesign introduces `/review-brd` as a new gate between `/gen-brd` (Approved) and `/gen-epics`. This means:

**Updated lifecycle:**
`/gen-brd` → `/review-brd` → PO approves BRD → `/gen-epics` → `/review-epics` → `/gen-stories` → `/review-stories` → sprint lock

The BRD approval status currently set by the PO after reading the document is now set by `/review-brd` after the walkthrough. This changes the gate enforcement check in `/gen-epics`: it checks BRD is Approved, which now implies `/review-brd` was completed.

`knowledge/process/defect-severity.md` — at the time of this decision (2026-06-29), still referenced by `/verify-story`, not retired, not affected by this decision. **Update 2026-07-13:** `defect-severity.md` was subsequently rewritten and moved to `reference/defect-severity.md` as part of the public-release cleanup that removed the rest of `knowledge/process/` (which described a fictional multi-agent model). `/verify-story` now reads it from the new path. This has no bearing on the review-model redesign itself.

---

## Implementation Status

Complete. Implemented in T-27 session on 2026-06-30.

---

## Implementation Notes

Key design decisions made during T-27 execution that extend the design above:

### review-architecture: deeper redesign than planned

FW-019 described a "lighter redesign" for `/review-architecture` — add real-world framing per section, keep structural checks unchanged. During implementation, this was revised to a full concern-by-concern sequential walkthrough, consistent with the redesign applied to the other review commands. Reasons:

- The architecture document's sections are deeply interdependent (modules, data model, endpoints, security all cross-reference each other). Reviewing sections independently would surface the same flags in multiple places and create redundant PO confirmations.
- Five thematic concerns (Module Structure, Data Model, API Contracts, Security & Access Control, Infrastructure & Compliance) group the checks in a way that mirrors how a PO mentally models an architecture — cleaner than section-by-section.
- Adding a walkthrough and state tracking at this level is consistent with the FW-019 principle of "walkthrough, not report" applied uniformly.

The five concerns and their 14 checks are documented in the implemented command at `.claude/commands/review-architecture.md`.

### Review state tracking pattern

A consistent pattern was established across all four review commands for resumable session state:

| Command | State location |
|---------|---------------|
| `/review-brd` | Section 15 appended to `brds/brd-core-v1.md` |
| `/review-epics` | `epics/review-record.md` (separate file) |
| `/review-stories` | `stories/review-record.md` (separate file) |
| `/review-architecture` | Section 11 appended to `architecture/arch-v1.md` |

BRD and architecture use in-document sections because there is one canonical file for each. Epics and stories use separate files because the reviewed artifacts are a collection (many files), and a single index file is the natural tracking location.

### Fix tiers: defined once, applied per concern

In `/review-architecture`, fix tiers are defined once at the top of Step 4 (before any concern begins) and referenced within each concern's FLAGS handling. They are not repeated per concern. This avoids duplication across five concern sections while keeping the tiers visible and unambiguous throughout the walkthrough.
