# FW-020 — Command Conventions

| Field | Value |
|-------|-------|
| **ID** | FW-020 |
| **Date** | 2026-06-30 |
| **Status** | Decided |
| **Area** | Framework-wide — all slash commands in `.claude/commands/` |

---

## Decision

A formal, verifiable convention set is established for all Ascendra slash commands. The full rule set — 22 rules across 5 sections, governing principle, exception model, and verification checklist — lives in [`conventions/command-conventions.md`](../conventions/command-conventions.md). That file is the authoritative reference; this record captures why the decision was made and what was corrected on first application.

---

## The Problem That Triggered This Decision

During the T-27 review model redesign, newly written and redesigned commands (`review-brd.md`, `review-epics.md`) were observed to have structural inconsistencies compared to established commands (`gen-brd.md`, `init-project.md`). Specifically:

- Structural checks formatted as standalone bold headings + paragraph, rather than numbered list items with inline descriptions
- Fix tier descriptions in paragraph prose rather than bold-label-plus-bullet-examples
- Gate checks embedded inconsistently — some in Step 1, some in a dedicated step, some mixed with file reading
- Final reports in prose or blockquotes rather than a fixed-format code block
- Missing `## Arguments received` block in several commands

These inconsistencies created three problems:
1. **Verifiability gap** — no checklist existed to confirm a command met the expected structure
2. **Reader confusion** — the same structural element looked different depending on which command you were reading
3. **Maintenance risk** — future commands written without a reference would drift further from the established pattern

---

## Scope of Application

Conventions were audited and applied retroactively to all Phase 1–5 commands during the T-27 session:

| Command | Deviations corrected |
|---------|---------------------|
| `gen-architecture.md` | Missing `## Arguments received` block |
| `gen-epics.md` | Missing `## Arguments received` block; final report was prose list |
| `gen-stories.md` | Gate check in Step 1 (not Step 1.5); final report was prose list |
| `gen-domain-knowledge.md` | Final report was prose + blockquote |
| `gen-domain-playbook.md` | No gate check; final report was prose + blockquote |
| `gen-brd-playbook.md` | Gate check and file reads mixed in Step 1; final report was prose + blockquote |
| `run-domain-discovery.md` | No code block before handoff offer |
| `run-brd-discovery.md` | No code block before handoff offer |
| `review-epics.md` | Checks as bold headings; fix tiers as paragraphs |
| `review-stories.md` | All deviations (full redesign under T-27) |
| `review-architecture.md` | Checks as bold headings; no argument parsing step; no real-world framing |

All new commands written after FW-020 must be verified against the checklist in [`conventions/command-conventions.md`](../conventions/command-conventions.md) before being marked ready. For commands not yet reviewed in the lifecycle (Phases 6–9), conventions are applied as part of the review pass.

---

## Relationship to Other Decisions

- **FW-019** (Review Model Redesign) — T-27 was the first session in which conventions were defined and retroactively applied. The review model redesign and the convention set were developed together.
- **T-25** (Post-Phase-5 lifecycle review) — Conventions apply to all Phase 6–9 commands during each phase's review pass.
- **T-10** (Agent-based implementation) — When agent command files are created under `.claude/commands/agents/`, the same conventions apply.
