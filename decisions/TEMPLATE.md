# FW-XXX — [Decision Title]

> **[AI Guide]** Title format is `FW-XXX — [Short noun phrase describing what was decided]`. Use the next available number from the decisions folder. The title names the thing that was decided, not the problem — e.g. "Brief-Driven Extension Model", not "Fix Project Composition".

**Governing principle:** The sections in this template are the minimum floor, not a ceiling. A simple naming convention needs only the required sections. A decision that redesigns multiple commands, retires existing patterns, introduces a new lifecycle gate, or has complex implementation sequencing should add whatever structure the content demands — sub-sections, design rationale, command-by-command breakdowns, sequencing tables, success criteria, open questions, full design references. The test is: could someone read this ADR six months from now and understand both what was decided and why, without needing to ask anyone? If not, add more.

| Field | Value |
|-------|-------|
| **ID** | FW-XXX |
| **Date** | YYYY-MM-DD |
| **Status** | Decided |
| **Area** | [Scope — e.g. "Framework-wide", "All commands", "`brief.md` and `/run-intake`"] |

> **[AI Guide]** The metadata table always contains ID, Date, Status, and Area. Add the following rows only when relevant — do not add them with empty values:
> - `**Supersedes** | FW-XXX — [Title]` — when this decision replaces a previous one entirely
> - `**Amends** | FW-XXX — [Title]` — when this decision modifies but does not replace a previous one
>
> Status values: `Decided` (default) · `Decided — implementation pending [T-XX]` · `Superseded by FW-XXX`

---

## Decision

> **[AI Guide]** 2–5 sentences. State what was decided in plain language. Focus on the outcome, not the reasoning — reasoning belongs in the sections below. Write it so someone who reads only this section understands the rule or design that now applies.

[State the decision here.]

---

## The Problem That Triggered This Decision

> **[AI Guide]** Explain what broke, what was missing, or what risk was identified. Be specific — name the command, file, or scenario. This section is the justification for why a decision was needed at all. Without it, a future reader cannot tell whether the decision still applies if the context changes.

[Describe the problem here.]

---

> **[AI Guide]** The sections below are free-form. Use as many or as few as the decision needs. Name them to match the content — examples from existing decisions: `## Rationale`, `## Key Design Decisions`, `## How Commands Use This`, `## What Gets Retired`, `## Scope of Application`. Do not use generic names like `## Details` or `## Notes`.
>
> Delete this guide block before writing the file.

---

## Implementation Status

> **[AI Guide]** Include this section only when implementation is not immediate — i.e. when the decision requires follow-up work tracked elsewhere. Use one of these three forms:
>
> - `Applied immediately during this session.` — no separate tracking needed
> - `Pending. Tracked in T-XX.` — work not yet done
> - `Complete. Implemented in T-XX session on YYYY-MM-DD.` — work done
>
> If implementation is complex, add a table showing status per item. Omit this section entirely if the decision is a rule that takes effect on reading (e.g. a naming convention) — those are applied immediately and need no tracking.

[Implementation status here.]

---

## Relationship to Other Decisions

> **[AI Guide]** Include this section only when the relationship is not already captured in the metadata table `Supersedes` / `Amends` rows and there is something worth stating — e.g. a dependency, a coordination note between two decisions, or a sequencing constraint. If the metadata table already says everything, omit this section.

[Cross-decision relationships here.]
