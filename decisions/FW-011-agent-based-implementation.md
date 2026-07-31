# FW-011 — Agent-Based Implementation Architecture

| Field | Value |
|-------|-------|
| **ID** | FW-011 |
| **Date** | 2026-06-27 |
| **Status** | Parked — reconsidered; command/chat mode retained until the framework matures |
| **Area** | `/implement-story`, `.claude/commands/agents/`, architecture document |
| **Full design** | Removed from the working tree 2026-07-14 (public-release cleanup) — see git history |

---

## Decision

Story implementation uses a **specialist agent model**, not a single generic command. A master orchestrator dispatches to technology-specific agents in dependency order.

---

## Key design decisions

### 1. Specialist agents at framework level

```
.claude/commands/agents/
├── agent-nestjs.md
├── agent-nextjs.md
├── agent-nextjs-ascendra-ui.md    ← variant with Ascendra UI knowledge
├── agent-spring-boot.md
├── agent-react-native.md
└── ...
```

Each agent holds deep knowledge of one technology stack — patterns, file structure, testing approach, conventions. Agents are reusable across all projects. Projects select which agents apply by declaring them in the architecture document.

### 2. Architecture document owns the execution context mapping

The architecture document defines an **Execution Contexts** section:

```markdown
| Context | Agent | Repo Path | Depends On |
|---------|-------|-----------|------------|
| api     | agent-nestjs | ../project-api | — |
| web     | agent-nextjs-ascendra-ui | ../project-web | api |
```

This is the single authoritative mapping of layer → agent → repo path → dependency order. `project.json` does not hold this information.

### 3. Single PO entry point — `/implement-story`

The PO always runs one command regardless of how many layers a story touches. The master orchestrator reads the story's context field and the architecture's Execution Contexts table, determines which agents are needed and in what order, and runs them sequentially, passing outputs (API contracts, component interfaces) between phases.

### 4. Dependency ordering at runtime

The orchestrator builds a dependency graph from the Execution Contexts table and executes agents in topological order. If the API layer must complete before the Web layer can consume its contract, the orchestrator ensures that ordering automatically.

### 5. Problem space vs solution space

```
Brief → BRD → Epics → Stories    ← problem space — technology agnostic
                    ↓
              Architecture        ← the bridge: defines technology and approach
                    ↓
            Implementation        ← solution space — fully technology specific
```

Implementation tooling is derived from the architecture, not pre-configured at project init.

---

## Full design

The complete background, problem statement, journey of thinking, and all open design items (D1–D8) were written up in `knowledge/archive/agent-implementation-architecture.md`. That file (and its companion staged execution plan, `knowledge/archive/t10-implementation-plan.md`) was removed from the working tree on 2026-07-14 as part of the public-release cleanup — both remain fully intact in git history. If this direction is revisited, retrieve them with `git log --all -- 'knowledge/archive/agent-implementation-architecture.md'` before starting a design session from scratch.

## Parking Note (added 2026-07-11)

Explored in real depth (see the full design doc referenced above), then reconsidered: the scope turned out larger than expected, and the PO chose to keep `/implement-story` as a chat-mode command rather than commit to the specialist-agent model right now. Not superseded by a different design — parked, and may be revisited once the framework is more mature.
