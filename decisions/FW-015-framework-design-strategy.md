# FW-015 — Framework Design Strategy

| Field | Value |
|-------|-------|
| **ID** | FW-015 |
| **Date** | 2026-06-28 |
| **Status** | Decided |
| **Area** | Framework-wide — positioning, enforcement, onboarding |

---

## Decision

The Ascendra Framework adopts a four-pillar design strategy:

1. **Prerequisite definition** — the framework explicitly defines who it is for and who it is not for
2. **Project boundary** — a project is one independently deliverable unit; engagement-level planning lives outside the framework
3. **Enforcement architecture** — lifecycle gates are enforced by commands, not by documentation
4. **Onboarding path** — the EXAMPLE project is the primary onboarding mechanism; documentation is secondary

---

## The Problem That Triggered This Decision

A scenario was raised: a PO runs `/init-project` for a school management platform and describes attendance, fee collection, teacher portal, parent portal, student portal, admin portal, and LMS — all in one project. The framework has no mechanism to redirect this. Guidelines exist but depend on user judgment to enforce.

The deeper question surfaced: if a user does not understand or cannot follow the framework correctly — not because they are unwilling, but because they lack the mental model — the framework produces poor output. The user concludes the framework failed. The actual failure was a missing prerequisite or a missing onboarding path.

Adding more documentation does not solve this. The solution requires a deliberate strategy.

---

## Pillar 1 — Prerequisite Definition

### What the framework is

A **force multiplier** for skilled delivery professionals. It amplifies existing capability — it does not grant capability the user does not have.

### What the framework is not

A learning tool for junior practitioners. A project management methodology anyone can follow. A replacement for delivery experience.

### The target user

A PO who can already think across BA, SA, architecture, engineering, and QA. Specifically:

- Has delivered software projects end to end, including client-facing discovery
- Understands what a domain is, what a requirement is, what an architecture decision is
- Can evaluate AI output and direct it when it goes wrong
- Has run at least one client discovery session before

**If a user does not meet this profile, the framework will produce faster output of the wrong kind.** This is not a framework failure — it is a prerequisite violation. The framework must say this plainly, not apologetically.

### Where this lives

- CLAUDE.md "Who This Is For" section — updated to reflect the above profile
- EXAMPLE project preamble — states prerequisites before the first command

---

## Pillar 2 — Project Boundary

### Definition

A project in Ascendra is a **single, independently deployable deliverable**. It produces one thing a user or system can interact with.

**Valid projects:**
- A web portal (admin portal, parent portal, teacher portal are three separate projects)
- A mobile app
- A microservice or API
- An operations management tool
- A standalone integration or data pipeline

**Not a valid project:**
- An entire platform ("school management system") — that is a client engagement
- A delivery phase ("Phase 1") — that is a milestone within a project
- Multiple portals — those are multiple projects

### The engagement / project distinction

| Level | What it is | Lives |
|-------|-----------|-------|
| **Engagement** | The full client relationship — all deliverables, phasing strategy, commercial terms, roadmap | Outside the framework |
| **Project** | One deliverable — one portal, one app, one service | Inside the framework |

Engagement-level planning happens before the PO opens Claude. The framework has no concept of engagements. It only knows projects. The PO and client agree on the engagement shape offline, then the PO picks the first deliverable and runs `/init-project`.

### How multiple deliverables relate

A client engagement that produces five deliverables becomes five Ascendra projects:

```
ALNOOR-FEE-001    Fee Management Portal
ALNOOR-PAR-001    Parent Mobile App
ALNOOR-TEA-001    Teacher Portal
ALNOOR-STU-001    Student LMS
ALNOOR-ADM-001    Admin Operations Panel
```

Each runs independently through the full framework lifecycle. Cross-project domain knowledge reuse is handled via the brief-driven extension model (`FW-016` — Extension Context fields in `brief.md`; `FW-010`'s original `composes`-in-`project.json` model is superseded, see `FW-027`). No domain knowledge is regenerated if it already exists in an extended project.

### Enforcement

`/init-project` asks one boundary question before creating the project folder:

> "In one sentence, what is the single deliverable this project produces — a portal, an app, a service, a tool?"

If the answer names more than one thing, the command redirects:

> "That sounds like more than one deliverable. Each deliverable is a separate Ascendra project. Which one do you want to start with?"

The folder is not created until the boundary is confirmed.

---

## Pillar 3 — Enforcement Architecture

### The principle

Documentation is the weakest form of enforcement. A guideline that depends on user memory to enforce itself will be violated. The strongest enforcement is: **the command refuses to proceed**.

### Lifecycle gate enforcement

Every lifecycle transition is enforced by the command that initiates the next phase. A user cannot skip a gate — not because a document says not to, but because the command checks the prerequisite and stops.

| Gate | Enforced by | Refuses if |
|------|------------|-----------|
| Brief → Domain Knowledge | `/gen-domain-knowledge` | No brief at `Approved` status |
| Domain Knowledge → BRD Playbook | `/gen-brd-playbook` | No approved domain knowledge document |
| BRD Playbook → Discovery | `/run-brd-discovery` | No BRD playbook at the project path |
| Discovery → BRD | `/gen-brd` | No completed discovery state file |
| BRD Draft → BRD Approved | `/review-brd` | BRD not at `Draft` or `Under Review` status (already Locked) |
| BRD → Epics | `/gen-epics` | No BRD at `Approved` status |
| Epics → Architecture | `/gen-architecture` | No approved epics |
| Architecture → Stories | `/gen-stories` | No architecture at `Approved` status |
| Stories → Implementation | `/implement-story` | Story not at `Ready` status |

### Override path

A PO with a legitimate reason to bypass a gate (e.g. AI already has sufficient domain knowledge and no discovery session is needed) can explicitly confirm the override. The command asks:

> "No domain knowledge document found for this project. Do you want to proceed anyway? The AI will generate from training knowledge only — no discovery session has been run. Confirm with 'yes, proceed without discovery'."

The override is explicit, not silent. It cannot happen by accident.

### What enforcement does NOT cover

Command-level enforcement catches missing prerequisites. It does not catch incorrect content — a brief that describes the wrong problem, a BRD that is shallow, a story that is ambiguous. Content quality is the PO's responsibility. The verification checklists embedded in each template (Section 12 in domain docs, Part 2 in brief, etc.) are the content quality mechanism. They run at artifact completion and flag gaps before the PO approves.

---

## Pillar 4 — Onboarding Path

### The principle

A framework is learned by doing, not by reading. A PO who reads CLAUDE.md and then starts a real project has a mental model built from text. A PO who runs the EXAMPLE project first has a mental model built from experience.

### The EXAMPLE project

The EXAMPLE project is a pre-built, realistic scenario included in the framework repository. It demonstrates the full lifecycle — intake through implemented stories — for a simple, known domain. Every command is pre-run. Every artifact is pre-generated at realistic quality. Every approval gate is shown in context.

A new PO's first action is not `/init-project`. It is:

1. Read `projects/EXAMPLE/` — see the complete output
2. Trace back from each artifact to the command that produced it
3. Read the brief, the domain knowledge, the BRD, the epics, the architecture — see how each flows from the previous
4. Then run `/init-project` on their first real project

After this pass, the PO has answered the questions that documentation cannot answer:
- What does a good brief actually look like?
- What level of detail belongs in domain knowledge vs. the BRD?
- How granular should epics be?
- What does a production-ready story contain?

### Documentation role

CLAUDE.md, command files, and template AI Guides are reference material — consulted during execution, not studied in advance. They are authoritative but not the primary learning path.

---

## Success Criteria

### Framework level

1. A skilled PO completes a production-ready deliverable — from intake to deployed stories — without a team
2. Every artifact passes its own verification checklist without manual gap-filling
3. A PO running two projects simultaneously produces higher-quality output than a traditional team running one
4. Domain knowledge built for one project is reused in a subsequent project without regeneration

### User level

1. A new user who works through the EXAMPLE project can start their first real project without external help
2. No lifecycle gate can be bypassed without an explicit, logged override
3. A PO can hand a project folder to another PO mid-delivery and the incoming PO can resume without a briefing session

### What is explicitly out of scope

The framework does not measure:
- Client satisfaction with the delivered product — that depends on PO judgment, not framework compliance
- Time saved versus a traditional team — this varies by domain complexity and PO experience
- Whether the right thing was built — that is a discovery and BRD quality question, not a framework question

---

## Implementation Status

| Pillar | Implementation | Status |
|--------|---------------|--------|
| Prerequisite definition | CLAUDE.md "Who This Is For" section | Done |
| Project boundary | `/init-project` boundary question + redirect | Done |
| Enforcement architecture | Per-command gate enforcement | Done — applied across all Phase 1-5 commands during the T-27 convention pass (`FW-020`) and the Batch 2/3 command-upgrade passes |
| Onboarding path | EXAMPLE project build | Pending — dedicated session, not yet scheduled |

Three of four pillars are implemented (verified 2026-07-13). The EXAMPLE project (Pillar 4) remains the one open item — this decision is still the design anchor for that pillar.
