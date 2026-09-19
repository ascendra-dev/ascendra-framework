# FW-052 — BRD Quick-Reference File

| Field | Value |
|-------|-------|
| **ID** | FW-052 |
| **Date** | 2026-09-19 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/review-brd.md`, `gen-epics.md`, `gen-stories.md`, `assess-change.md` |
| **Amends** | Extends `FW-001`'s `{artifact}-v{N}-ref.md` naming exception to a second artifact type; no other decision changed |

---

## Decision

`/review-brd` generates a companion file, `projects/{PROJECT_CODE}/brds/brd-core-v{N}-ref.md`, on every
approval (initial or re-approval after `/assess-change`) — the same role `arch-v1-ref.md` already plays
for the architecture document. It carries the BRD's Section 5 Requirements table verbatim, plus a
condensed Personas & Roles list and the Section 6 Business Rules list, and nothing else. `/gen-epics`
and `/gen-stories` read it first for REQ-level planning, falling back to the full BRD — with the
fallback stated explicitly, same discipline the architecture-ref consumers already follow — only for
content the ref file doesn't carry (journey narrative, NFRs, a requirement's surrounding context).
`/assess-change` regenerates it in full, never patches it, whenever a Pattern 4 change bumps the BRD
version.

Generation happens at `/review-brd` approval time, not at `/gen-brd`'s initial draft — deliberately
different timing from `arch-v1-ref.md`, which is written by `/gen-architecture` itself before its own
review runs. A BRD's review walkthrough (`/review-brd` Section 17) routinely rewrites REQ text; a ref
file written at draft time would already disagree with the approved document by the time anything reads
it. Generating it at the point the content is actually final avoids that.

---

## The Problem That Triggered This Decision

A phase-by-phase token-economy audit of all 34 commands (2026-09-19, published as an artifact) found
that `/gen-epics`, `/review-epics`, `/gen-stories`, `/review-stories`, and `/gen-screen-design` each
independently full-read the BRD — a wider fan-out than the one that justified building `arch-v1-ref.md`
in the first place. `/gen-stories` in particular re-reads it once per epic, every time it runs, with no
reuse exemption (a related, narrower gap — the missing "already loaded" caveat on that same read — was
fixed directly in `gen-stories.md`/`gen-epics.md` alongside this decision, not left to the ref file
alone to solve).

Most of what those commands need from the BRD is the REQ table: `/gen-epics` groups REQs into epics and
must quote their descriptions "word for word," `/gen-stories` maps epic In-Scope bullets back to the
REQs behind them. Neither routinely needs the BRD's journey narrative, NFRs, integration detail, or
open-questions log to do that job — exactly the same shape of gap `arch-v1-ref.md` already closed for
architecture's table/endpoint/enum consumers.

---

## Why REQs, Personas/Roles, and Business Rules — Nothing Else

The ref file's contents were scoped to exactly what `/gen-epics` and `/gen-stories` were found to
actually use at REQ-planning time: REQ text (quoted verbatim, since paraphrasing a requirement is
explicitly forbidden downstream), the persona/role list (epics name which personas are involved;
stories pick a persona for their statement), and business rules (stories check AC coverage against
them). Journeys, data requirements, integrations, security requirements, NFRs, dependencies, and open
questions were deliberately left out — every command that needs those already reads the full BRD for a
reason unrelated to REQ-level planning (`/gen-architecture`, `/gen-screen-design`, `/review-brd` itself),
so a ref file trying to also carry them would just become a second full BRD with extra maintenance
burden, not a lean one.

---

## Scope of Application

Applies to every project's BRD from this decision forward. Does not retrofit any already-approved BRD
that predates it — `gen-epics.md`/`gen-stories.md` both state the fallback explicitly for that case
("BRD approved before `FW-052`") rather than assuming the ref file always exists.

---

## Implementation Status

Applied immediately during this session:
- `review-brd.md` Step 6 — generates the ref file on approval, before the completion report
- `gen-epics.md` Step 2, `gen-stories.md` Step 2 — read the ref file first, full-BRD fallback stated explicitly, same-session reuse caveat added
- `assess-change.md` — BRD quick-reference sync rule added alongside the existing architecture-ref sync rule; Step 7 post-change verification gained a matching check

Not yet applied: `review-stories.md` and `gen-screen-design.md` were left reading the full BRD as-is —
both load it once per whole session rather than once per epic, so the repeated-read cost this decision
targets doesn't apply to them the same way. Revisit only if a future audit finds otherwise.

---

## Relationship to Other Decisions

Direct sibling of the mechanism `FW-001` already carved out an exception for (`arch-v1-ref.md`) — this
decision extends that same naming and regeneration discipline to a second artifact type rather than
inventing a new one. Stays inside `FW-022`'s Problem Domain boundary: the ref file only ever holds BRD
content, never anything architecture-derived.
