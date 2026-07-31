# FW-016 — Brief-Driven Extension Model

| Field | Value |
|-------|-------|
| **ID** | FW-016 |
| **Date** | 2026-06-28 |
| **Status** | Decided |
| **Supersedes** | FW-010 (Project Composition) |
| **Amends** | FW-006 (project.json identity only — `composes` field removed) |
| **Area** | Framework-wide — project extension, domain knowledge loading, command behaviour |

---

## Decision

Project extension relationships are expressed in the **project brief**, not in `project.json`. Every command reads the brief's Extension Context fields and resolves the extension chain before proceeding. The `composes` field is removed from `project.json`. `project.json` is identity only (restoring FW-006).

> **Implementation note (added retroactively):** as actually built, "Extension Context" is not a literal `### Extension Context` heading — it's three flat header fields (`Project Type`, `Extends`, `Extension Adds`) living alongside Project Name/Code/Client in the brief's header block, exactly like `brief.template.md` and `run-intake.md` implement it today. This document originally illustrated a dedicated heading with `Type` / `Extension dimension` / separate "what this adds" and "what this does not change" fields — that illustration was simplified away during implementation and never updated here. All six consuming commands (`run-intake`, `gen-architecture`, `gen-brd`, `gen-brd-playbook`, `gen-domain-knowledge`, `gen-epics`) already agree on the real field names; this document has been corrected to match them below. Nothing functional was broken — only this record was stale.

---

## What This Is — and What It Is Not

This decision does not remove the concept of composition. Extension chains still exist. ASCPAY-EDU-PK-001 building on ASCPAY-PK-001 building on ASCPAY-CORE-001 is still a dependency graph. What changes is how that graph is expressed and resolved:

| | FW-010 (old) | FW-016 (new) |
|--|---|---|
| Where expressed | `composes` array in `project.json` | Extension Context section in `brief.md` |
| PO maintains | Full ancestor list, ordered | Direct parent only |
| Chain resolution | Framework reads JSON config | AI reads briefs transitively |
| Human-readable | No | Yes — brief is reviewed at every command |
| Cycle detection | Not specified | Required in every command's chain resolution logic |

---

## The Problem with FW-010

FW-010 put the `composes` field in `project.json` — a machine-readable config file. This created several problems:

- **Contradicted FW-006** — project.json was decided as identity-only; composition added structural coupling to a file that should not carry it
- **PO responsibility** — the PO had to maintain the full ordered ancestor list and keep it in sync with the actual extension chain
- **Invisible to commands** — config fields in JSON are not reviewed at every command; they are set once and forgotten
- **Wrong abstraction** — composition is a *relationship between projects*, which belongs in a document the PO writes and reviews, not in a config file

---

## The Solution: Extension Context in the Brief

During intake (`/run-intake`), the PO fills three Extension Context fields in `brief.md`'s header block (alongside Project Name, Project Code, Client, Domain):

**For a standalone project:**
```markdown
**Project Type:** Standalone
**Extends:** —
**Extension Adds:** —
```

**For an extension project:**
```markdown
**Project Type:** Extension
**Extends:** ASCPAY-CORE-001
**Extension Adds:** Pakistan FBR e-invoicing compliance, IRIS integration
```

`Extension Adds` is a single free-text sentence — what this project adds on top of the parent, gathered by `/run-intake` asking "In one sentence, what does this project add on top of that?" It is not a structured category tag.

The PO specifies only the **direct parent**. The AI resolves the full chain transitively.

---

## How Commands Use Extension Context

Every command that opens a new lifecycle phase reads the brief's Extension Context before proceeding. Chain resolution follows these steps:

1. Read the current project's brief → check the Extension Context fields
2. If `Project Type: Extension`: read the referenced project's brief → check its Extension Context fields too
3. Repeat until a `Project Type: Standalone` project is reached (the chain root)
4. Load artifacts in order: chain root first, direct parent last, current project is built on top

**Cycle detection:** If a project appears more than once in the chain, stop immediately and report the circular reference to the PO before proceeding.

**Referenced project not in workspace:** If the brief references a project code that does not exist in `projects/`, the command stops and asks the PO to either add the project to the workspace or provide direct artifact file paths.

---

## What Each Command Loads from the Extension Chain

Not every command loads everything. Loading rules per command:

| Command | What to load from parent projects |
|---------|----------------------------------|
| `/gen-domain-knowledge` | Domain knowledge documents — base first, extension layer last. Generate current project's domain as a delta only — do not replicate what the parent already covers. |
| `/gen-brd-playbook` | Domain knowledge from chain (for Known Variations). BRD from direct parent as baseline context. |
| `/gen-brd` | BRD from parent as baseline — to understand what is already required. Current BRD captures only what is new or different. |
| `/gen-architecture` | Architecture document from parent — specifically the Extension Points section. Current architecture documents how this project plugs into those seams. |
| `/gen-epics` | Nothing. Epics are always project-specific — inherited requirements are already in the BRD. |
| `/gen-stories` | Nothing. Stories are always project-specific. |
| `/implement-story` | Architecture only — to know which codebase and service each story targets. |

---

## Extension Domain Documents Are Deltas

When `/gen-domain-knowledge` runs for an extension project, the generated domain document is explicitly additive. It must not replicate content from the parent domain. It documents only:

- New entities introduced by this extension (e.g. FBR Invoice, IRIS Submission)
- New business rules that apply in this context but not in the parent
- Overrides to parent rules — explicitly named and justified
- New lifecycle states or transitions

The domain template's Section 12.1 extension document rules apply: *"Extension documents add or narrow rules from the base — they do not override without explicit justification."*

---

## Architecture Decides Scaffolding

The Extension Context in the brief establishes the relationship. It does not decide what gets built. That decision belongs to the architecture phase.

When the architecture document for an extension project is generated, the architect decides:
- Does this extension add to an existing service, or does it require a new one?
- Does it extend the existing web app, or does it require a new frontend?
- Does it require a new mobile app?

If the architecture decides a new service or app is needed, that becomes a **separate Ascendra project** — with its own brief, its own Extension Context pointing to the parent, its own lifecycle. If the architecture decides to extend existing code, the extension stories are implemented into the existing codebase.

The framework does not pre-decide scaffolding. The architecture document does.

---

## Impact on project.json

**Superseded 2026-07-13 by `FW-027`.** At the time this decision was written, the plan was: the `composes` field is removed and `project.json` returns to identity-only (FW-006 restored). That intermediate state was never the end point — `project.json` was subsequently retired entirely by `FW-027`, since it never acquired a real reader (every command derives `PROJECT_CODE` from its file-path argument). Project identity now lives only in `projects/index.md`. The substantive point this decision established — the extension relationship lives in `brief.md`, not in a machine-readable composition config — is unaffected and remains current.

---

## Implementation Status

| Item | Status |
|------|--------|
| Extension Context fields in brief template | Done — implemented as flat header fields (`Project Type`/`Extends`/`Extension Adds`), not the dedicated heading originally sketched above; this document corrected to match on 2026-07-11 |
| Extension Points section in architecture template | Done — `arch.template.md` Section 3.2 |
| "Designed for Extension" section in BRD template | Done — `brd.template.md` Section 1.6 |
| Extension chain resolution logic in all commands | Done — `run-intake`, `gen-architecture`, `gen-brd`, `gen-brd-playbook`, `gen-domain-knowledge`, `gen-epics` all implement it consistently |
| Remove `composes` from project.json | Superseded — moot, since `project.json` was retired entirely by `FW-027` (2026-07-13); see the corrected "Impact on project.json" section above |
