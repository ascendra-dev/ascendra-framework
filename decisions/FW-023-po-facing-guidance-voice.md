# FW-023 — PO-Facing Guidance Voice

| Field | Value |
|-------|-------|
| **ID** | FW-023 |
| **Date** | 2026-07-05 |
| **Status** | Decided |
| **Area** | All live-session commands that present template-derived guidance to a human before asking a question |

---

## Decision

When a command presents a template's `[AI Guide]` content to a live human (PO or simulated client) before asking a question, it must preserve the guidance's full **structure and specificity** — every numbered question, lettered sub-case, bulleted checklist item, and Correct/Incorrect example, verbatim where the example is a quote — but rewrite its **voice**. The `[AI Guide]` / `[AI Guide — Label]` bracket headers are template-authoring labels and must never be shown to a human. Any sentence written as an instruction to the drafter (e.g. "Adjust your language accordingly," "a developer or AI agent joining mid-project would understand...") must be rewritten as a direct statement or question addressed to the PO — the underlying requirement stays, only the addressee changes.

This is not a synthesis-into-prose rule. Flattening structured guidance into a short paraphrase loses the checklist items and examples the PO needs — that was tried and rejected during manual testing (see below). The fix is re-voicing, not summarising.

---

## The Problem That Triggered This Decision

During the Phase 1–5 manual test pass, `/run-intake` Step 5-A instructed: *"show the full `[AI Guide]` note for that section from `brief.template.md`... Label it clearly."* This is a verbatim-reproduction rule, applied identically to every section regardless of how that section's guide happens to be authored in the template.

Section 1 (Problem Statement) has one flowing `[AI Guide]` block with a numbered list and a Correct/Incorrect pair — verbatim display of that reads naturally. Section 2 (Client Context) has *two* stacked blocks — `[AI Guide — Client definition]` (three lettered sub-cases) and a separate `[AI Guide]` (a bulleted checklist) — and several sentences written as meta-instructions to whoever drafts the section, not to the PO (e.g. "Adjust your language accordingly. In case (a) write as if describing a third party..."). Verbatim display of that reads like raw internal documentation being read aloud.

The first attempted fix over-corrected: it replaced verbatim display with "synthesise into 2–4 natural sentences," which flattened the numbered questions, lettered cases, and checklist items into unstructured prose. This lost real value — the PO could no longer see the discrete points to cover or the Correct/Incorrect contrast clearly. The correct fix keeps the structure and only changes who the sentences are addressed to.

---

## Scope of Application

Checked every command that references `[AI Guide]` content (`grep -l "AI Guide" .claude/commands/*.md`). Two distinct categories exist, only one of which this decision governs:

**Category 1 — Generated document output (not affected by this decision):** `gen-brd`, `gen-epics`, `gen-stories`, `gen-domain-knowledge`, `gen-domain-playbook`, `gen-brd-playbook`, `run-brd-discovery` (final state file), `run-domain-discovery` (final state file). These already correctly *strip* all `[AI Guide]` blocks from generated output — generated artifacts are clean documents, not live conversation, and this was already right.

**Category 2 — Live session presentation to a human (governed by this decision):**
- `/run-intake` — fixed under this decision (Step 5-A)
- `/run-brd-discovery` — checked, already compliant: Step 6 asks the playbook's questions directly in conversational form ("Ask each subsection's primary question... Walk through each primary journey...") without ever dumping a raw `[AI Guide]`-labeled block. No change needed.
- `/run-domain-discovery` — checked, already compliant: Step 6 uses the same direct, conversational pattern ("Present AI knowledge of standard entities. Ask the PO to confirm, correct, or extend..."). No change needed.
- `/run-mock-discovery` — not in scope: it is a self-test simulating both roles internally, not a live PO-facing session.

So `/run-intake` was the only command with this defect. `run-brd-discovery` and `run-domain-discovery` were already written in the correct voice — this decision documents *why* that pattern is correct so it doesn't regress, and gives future command authors (including any live-session command T-10's Solution Domain work introduces) a named rule to follow instead of re-deriving it.

---

## Implementation Status

Applied immediately during this session — `run-intake.md` Step 5-A rewritten. No other command required changes.

---

## Relationship to Other Decisions

- **FW-004** (AI Guide Label Convention) — establishes the `[AI Guide]` label itself for template-authoring use. FW-023 does not change that convention; it governs what happens when a *command* reads that content for a live human, which FW-004 didn't specify.
- **FW-019** (Review Model Redesign) — established "real-world framing... in plain PO language" for the review commands, which independently arrived at a compatible principle from a different angle (walkthrough commands, not intake). FW-023 generalises the same underlying rule — never expose internal authoring scaffolding to the PO — into an explicit, named convention so it applies uniformly rather than being re-discovered per command.
