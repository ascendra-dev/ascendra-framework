# FW-005 — Brief as Intake State

| Field | Value |
|-------|-------|
| **ID** | FW-005 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | `brief.md`, `/run-intake` command |

---

## Decision

`brief.md` itself is the state for intake sessions. There is no separate session log, session state file, or progress tracker.

### Completion detection

`/run-intake` determines section completion using dual detection:

- A section is **blank** if it contains the awaiting-intake placeholder text, OR if the section body is empty or whitespace only.
- A section is **complete** if it contains real content (paragraph, table, or list that is not a placeholder string).

Dual detection handles the case where a PO deletes a placeholder without running `/run-intake`.

### Status journey

```
Draft  →  Brief complete  →  Approved
```

| Status | Set by | Meaning |
|--------|--------|---------|
| `Draft` | `/init-project` | Brief created, intake not yet run |
| `Brief complete` | `/run-intake` | All 9 sections filled and verified |
| `Approved` | PO via `/update-status` | Brief signed off — downstream commands unblocked |

### Amendment tracking

Amendment tracking (Document History table entries) only activates after the brief is `Approved`. Changes made while the brief is `Draft` or `Brief complete` are not tracked — they are part of the normal intake and revision process.

Once `Approved`, any change made via `/run-intake` prompts for a one-line amendment summary and adds a versioned row to the Document History table.

### Session modes

`/run-intake` detects the brief status and enters the appropriate mode:

| Mode | Trigger | Behaviour |
|------|---------|-----------|
| Intake | Status = `Draft` or contains placeholder | Works through blank sections in order |
| Revision | Status = `Brief complete` | Asks which section to revise — no verification re-run unless requested |
| Amendment | Status = `Approved` | Confirms intent, asks which section, tracks every write in Document History |

---

## Rationale

A separate session state file creates two sources of truth — the brief and the session log — which can diverge. Using `brief.md` directly means the state is always exactly what the document contains. Resuming a session is automatic: the command reads the document, detects what's filled, and picks up from the first incomplete section.
