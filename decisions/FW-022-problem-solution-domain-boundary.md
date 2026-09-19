# FW-022 — Problem Domain / Solution Domain Boundary

| Field | Value |
|-------|-------|
| **ID** | FW-022 |
| **Date** | 2026-07-04 |
| **Status** | Decided |
| **Area** | Framework-wide — lifecycle phase classification, scopes T-10 |

---

## Decision

The 28-step lifecycle ([`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md)), plus the 3 Screen Design sub-steps (9a–9c, [`decisions/FW-026-screen-design-phase.md`](FW-026-screen-design-phase.md)), is classified into three regions:

- **Problem Domain** — Brief, Domain Knowledge, BRD, Epics, Screen Design
- **Bridge** — Architecture
- **Solution Domain** — Stories, Sprint Plan, Implement Story, Verify Story, PO PR Review, Sprint QA, UAT, Close Sprint, Release Notes, Deploy, Support

```
┌──────────────────── PROBLEM DOMAIN ────────────────────────┐
│  no architecture dependency at all                          │
│  Brief → Domain Knowledge → BRD → Epics → Screen Design     │
└───────────────────────────────────────────────────────────────┘
                      │
                      ▼
          ┌─────────────────────────┐
          │      ARCHITECTURE        │  ← bridge
          │  first hard dependency   │
          │  point in the gate chain │
          └─────────────────────────┘
                      │
                      ▼
┌────────────── SOLUTION DOMAIN ──────────────────────────────────────┐
│  everything below requires Locked architecture to exist              │
│  Stories → Sprint Plan → Implement → Verify → PR Review →            │
│  Sprint QA → UAT → Close Sprint → Release Notes → Deploy → Support   │
└────────────────────────────────────────────────────────────────────────┘

  cross-cutting, not owned by any region: assess-change · update-status · project-status
```

**Boundary test:** a phase is problem domain only if it can be produced with zero knowledge of the tech stack. A phase is solution domain if it has a hard dependency on Locked architecture, directly or transitively.

This is a mechanical test, not a judgment call — it is answered by the existing gate table in [`decisions/FW-015-framework-design-strategy.md`](FW-015-framework-design-strategy.md):

| Gate | Requires architecture? |
|------|------------------------|
| Brief → Domain Knowledge | No |
| Domain Knowledge → BRD Playbook → BRD | No |
| BRD → Epics | No |
| Epics → Screen Design | No |
| Screen Design → Architecture | No (Screen Design is the input, not the consumer) |
| Architecture → Stories | **Yes — hard gate** (`gen-stories.md` blocks unless architecture Status = Locked) |
| Stories → Sprint Plan → Implementation → … | Yes, transitively |

---

## Why Stories Sit in Solution Domain, Not Problem Domain

This was the point of dispute during the design conversation, and the one non-obvious call in this decision.

Stories look like problem-domain artifacts on the surface — PO-authored in chat mode, acceptance criteria written in business language, and the story template explicitly forbids technical content in the body ("if text describes database schema, API signatures, or UI layout decisions, it belongs in the architecture document... not in the acceptance criteria").

But two facts override the surface appearance:

1. **Ordering is a hard gate, not a filing convenience.** `/gen-stories` refuses to run unless architecture is Locked. Epics has no equivalent dependency — it is generated and reviewed with zero tech-stack knowledge, before architecture even exists.
2. **Planned metadata would make the dependency explicit, if built.** T-10 design item D3 (the parked agent-based implementation design, [`decisions/FW-011-agent-based-implementation.md`](FW-011-agent-based-implementation.md)) would have stories declare an execution-context tag, validated against the architecture's Execution Contexts table — every story carrying architecture-derived routing metadata as a first-class field, not an optional hint. T-10 is currently parked, so this is supporting context for the argument, not a committed near-term fact — fact 1 alone already establishes the ordering dependency.

Conclusion: Stories are the bridge's first *consumer*, not the problem domain's last *producer*. The content staying business-flavored is a quality convention (keep it PO-readable), not evidence of domain membership.

---

## Two Independent Axes

This decision separates two things that are easy to conflate:

- **Domain axis** (what a phase is about): problem domain (defines intent) vs. solution domain (builds/verifies against that intent), with Architecture as the translation point between them.
- **Mode axis** (who drives execution): PO-led/chat-mode vs. agentic.

These do not correlate 1:1. Stories is PO-chat-mode **and** solution domain. PO PR Review and PO-runs-UAT are manual **and** solution domain — permanently, because they are supervision gates (a human checking the machine's output), not authoring gates (a human shaping the spec). Problem domain phases and Architecture stay chat-mode by design, per the framework's core review-gate philosophy; within solution domain, only the execution steps (Implement Story, Verify Story, and likely Sprint Plan generation) are candidates to move to agentic mode under T-10 — the supervision gates do not move.

---

## Scope of Application

- Defines the conceptual boundary referenced when scoping T-10 (FW-011 agent-based implementation architecture): T-10's solution-domain agentic redesign now explicitly includes `/gen-stories` and `/review-stories`, not just `/implement-story` and `/verify-story`.
- `assess-change`, `update-status`, and `project-status` are cross-cutting utilities that operate across all three regions and are not classified into any one of them.
- No commands, templates, or gates change as a result of this decision — it is a classification of the existing lifecycle, not a restructuring.

---

## Relationship to Other Decisions

- **FW-015** (Framework Design Strategy) — supplies the gate table this decision's boundary test is verified against. FW-022 does not change any gate; it names the region each gate sits in.
- **FW-011** (Agent-Based Implementation Architecture, T-10 pending) — this decision sets the scope boundary for T-10's agentic redesign: solution domain from Stories onward, with supervision gates (PR review, UAT, sprint activation/close) excluded from agentic conversion.
- **FW-019** (Review Model Redesign) — `/review-brd`, `/review-epics` sit in problem domain (authoring gates); `/review-stories` sits in solution domain despite following the same walkthrough pattern, per the boundary test above; `/review-architecture` is the bridge's own gate.
