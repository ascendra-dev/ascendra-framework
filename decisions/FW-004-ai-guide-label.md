# FW-004 — [AI Guide] Label Convention

| Field | Value |
|-------|-------|
| **ID** | FW-004 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | All `.template.md` files |

---

## Decision

Notes embedded in template files that provide instructions for how to populate a section are labelled `[AI Guide]` — not `[Guide]`, not `[Note]`, not `[Instructions]`.

```markdown
> **[AI Guide]** 2–4 sentences. Describe the current-state problem...
```

The label `[AI Guide]` is intentional and must not be changed.

---

## Rationale

These notes are **direct commands to the AI agent** — authoritative instructions that tell Claude exactly what to write, what to verify, and what to reject. They are not soft guidance for the PO to optionally read.

The `[AI Guide]` label signals to the AI agent that this block is an operational instruction, not document content. Renaming it to `[Guide]` or `[Note]` weakens that signal and could cause the AI to treat the instructions as commentary rather than commands.

POs also benefit from reading these notes — but that is a secondary benefit. The label remains `[AI Guide]` because the primary audience is the AI executing the command.

---

## What [AI Guide] notes contain

- The correct format for the section (paragraph, table, list)
- The correct/incorrect distinction (with labelled examples)
- Verification rules the AI must check before approving content
- Framing guidance (e.g. "write from the client's perspective, not the system's")

These are the highest-authority instructions in the framework. They override any default AI behaviour for that section.
