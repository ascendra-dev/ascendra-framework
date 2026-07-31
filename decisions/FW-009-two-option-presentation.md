# FW-009 — Two-Option Presentation Pattern

| Field | Value |
|-------|-------|
| **ID** | FW-009 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | All commands and documents that offer a command vs manual choice |

---

## Decision

Wherever a command or document offers two ways to complete a task — running a command or doing it manually — the command is explicitly labelled as **Recommended** and the manual option is labelled as **Alternatively**.

```markdown
**Recommended:** Run `/run-intake {PROJECT_CODE}` to complete this brief interactively — it guides you
section by section, validates your answers, and writes each section on approval.
**Alternatively:** Fill in each section manually by following the `[AI Guide]` notes in
`projects/TEMPLATE/brief.template.md` and referring to the worked example in
`projects/TEMPLATE/brief.example.md`.
```

### Rules

1. **Command first, manual second** — the recommended path is always stated first.
2. **State the reason** — explain why the command is recommended (what it does for the PO that manual does not — e.g. guides, validates, writes on approval). Do not just say "recommended."
3. **Manual option is always present** — the manual path is never hidden. A PO should always be able to operate without running a command.
4. **Both options are specific** — name the exact command with its arguments, and name the exact files for the manual path.

---

## Where this pattern applies

- `brief.md` status note (created by `/init-project`)
- `brief.md` section placeholders (created by `/init-project`)
- `/init-project` Step 8 next-steps report
- Any future command or document that offers a choice between running a slash command and doing something manually

---

## Rationale

The framework is designed to be used with commands, but enforcing command-only usage would break the framework for POs who prefer to work directly in files, or in situations where a command cannot run (e.g. offline, context limits). Labelling the command as recommended — with a reason — guides POs without coercing them.
