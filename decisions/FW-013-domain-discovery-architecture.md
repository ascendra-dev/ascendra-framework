# FW-013 — Domain Discovery Architecture

| Field | Value |
|-------|-------|
| **ID** | FW-013 |
| **Date** | 2026-06-27 |
| **Status** | Decided — implemented |
| **Area** | Domain phase commands, TEMPLATE/domain/, TEMPLATE/brds/, CLAUDE.md |

---

## Decision

The domain knowledge phase gains a structured discovery path (optional) that mirrors the BRD phase's playbook → discovery → generation pattern. Domain discovery is optional for well-known domains and required for niche or unfamiliar domains. `/gen-domain-knowledge` warns when domain discovery was not run and pauses for confirmation.

---

## Updated Lifecycle — Domain Phase

```
[Optional] /gen-domain-playbook
               ↓
[Optional] /run-domain-discovery
               ↓
           /gen-domain-knowledge  ← warning shown if no domain discovery ran
               ↓
           /gen-brd-playbook  (renamed from /gen-playbook)
               ↓
[Optional] /run-mock-discovery  ← validates domain knowledge + BRD playbook together
               ↓
           /run-brd-discovery  (real client session)
               ↓
           /gen-brd
```

---

## New and Changed Commands

### `/gen-domain-playbook` (new)
- **Purpose:** Generate a structured interview guide so the PO can teach the AI an unfamiliar domain.
- **Inputs:** `brief.md` + `domain-playbook.template.md`
- **Output:** `projects/{CODE}/domain/{slug}-domain-playbook.md`
- **Template:** `projects/TEMPLATE/domain/domain-playbook.template.md`
- **Template structure:** 11 sections mapping to each section of the domain knowledge document — domain scope, entities, lifecycle, business rules, validations, personas, integrations, regulatory baseline, known variations, scope boundaries, session close.

### `/run-domain-discovery` (new)
- **Purpose:** Run a structured session where the PO teaches the AI the domain using the domain playbook.
- **Inputs:** `domain/{slug}-domain-playbook.md` + `brief.md`
- **Output:** `projects/{CODE}/domain/domain-discovery-state.md`
- **Template:** `projects/TEMPLATE/domain/domain-discovery-state.template.md`
- **State structure:** Session history, playbook progress (one-line answer per section), domain confidence score, open issues, resume instructions.

### `/gen-domain-knowledge` (updated)
- **Change:** Accepts `domain/domain-discovery-state.md` as optional input. When it exists, enriches the output with PO-stated knowledge. When it does not exist, shows warning and pauses:

```
DOMAIN KNOWLEDGE — {PROJECT_CODE}
─────────────────────────────────────────
⚠  Domain discovery was not run.

This document will be generated from AI training knowledge only.

For well-known domains (invoicing, payroll, school fees, HR): this is
sufficient and you can proceed.

For niche, regulated, or unfamiliar domains: the result may be thin or
inaccurate without prior domain discovery.

Recommended: Run /gen-domain-playbook → /run-domain-discovery first
to ensure the domain knowledge reflects your specific domain context.

Proceed with AI knowledge only? (yes / no)
─────────────────────────────────────────
```

### `/gen-brd-playbook` (renamed from `/gen-playbook`)
- **Change:** Rename only. No logic change.
- **Output path:** `brds/{slug}-brd-playbook.md` (was `playbooks/{slug}-discovery.md`)
- **Template:** `projects/TEMPLATE/brds/brd-playbook.template.md` (moved + renamed from `playbooks/playbook.template.md`)

### `/run-mock-discovery` (new)
- **Purpose:** Simulate a client discovery session internally to validate that domain knowledge and BRD playbook work correctly together before a real client session.
- **Inputs:** `domain/{slug}-core.md` + `brds/{slug}-brd-playbook.md`
- **Output:** Updates `domain/{slug}-core.md` Section 12.6 in place. No new file.
- **Method:** AI plays both roles — facilitator (using BRD playbook) and simulated client (using domain knowledge to model typical client answers). Evaluates result against the five structured checks in Section 12.6.
- **Position:** After `/gen-brd-playbook`, before `/run-brd-discovery`.

---

## New Templates

| Template | Location |
|---|---|
| `domain-playbook.template.md` | `projects/TEMPLATE/domain/` |
| `domain-discovery-state.template.md` | `projects/TEMPLATE/domain/` |
| `brd-playbook.template.md` | `projects/TEMPLATE/brds/` (moved + renamed from `playbooks/playbook.template.md`) |

---

## CLAUDE.md Command Table Changes

Add:
- `/gen-domain-playbook` — Generate a domain discovery playbook for an unfamiliar domain
- `/run-domain-discovery` — Run a domain discovery session to teach the AI the domain
- `/gen-brd-playbook` — Generate a BRD discovery playbook from the domain knowledge document
- `/run-mock-discovery` — Simulate a client discovery session to validate domain knowledge and BRD playbook

Remove:
- `/gen-playbook` — replaced by `/gen-brd-playbook`

---

## Design Rationale

The BRD phase has always had three upstream phases: playbook → discovery → generation. Domain knowledge had zero. This asymmetry was acceptable for well-known domains where AI training knowledge is deep and accurate. For niche domains it produced thin or incorrect output with no structured correction path.

The optional discovery path resolves this without burdening well-known domain projects with unnecessary steps. The warning message makes the trade-off explicit — the PO decides with full information.

Mock discovery (12.6 validation) was previously a manual checklist that was effectively never run. Making it a command turns it into an executable gate that catches domain knowledge and playbook gaps before real client cost.

---

## Full implementation scope

Implemented in full — see the Status field above and `.claude/commands/gen-domain-playbook.md`, `run-domain-discovery.md`, `gen-domain-knowledge.md`, `gen-brd-playbook.md`, `run-mock-discovery.md`.
