# FW-017 — Estimation References Model

| Field | Value |
|-------|-------|
| **ID** | FW-017 |
| **Date** | 2026-06-29 |
| **Status** | Decided |
| **Area** | Story generation — sizing model selection |

---

## Decision

The framework ships three estimation standards. At the start of the first `/gen-stories` run for a project, the AI asks the PO to choose one. The chosen standard is written to `projects/{PROJECT_CODE}/standards/estimation.md` and all subsequent story-related commands read from that file. No community contribution. No template for custom standards.

---

## The Problem That Triggered This Decision

`gen-stories.md` and `review-stories.md` currently reference `knowledge/process/estimation-rules.md` directly — a framework-level file the command reads as if the sizing model were a fixed law. This:

1. Treats Ascendra's opinionated sizing model as a universal standard, which it is not
2. Creates an inconsistency — all other commands reference only `projects/TEMPLATE/` and project artifacts; estimation is a special-case exception
3. Gives the PO no choice, even though sizing methodology is an organisational decision

---

## The Three Reference Standards

The universe of sizing approaches in active use is small and closed. Three models cover it:

| File | Model | Recommended |
|------|-------|-------------|
| `reference/estimation/tshirt-sizing.md` | XS / S / M / L / XL with a 4-factor sizing model | Yes — Ascendra default |
| `reference/estimation/fibonacci-points.md` | 1 / 2 / 3 / 5 / 8 / 13 story points | No |
| `reference/estimation/time-based.md` | Hours per story, banded by complexity | No |

No template for custom standards is shipped. The three models cover all practical cases. A team that uses a non-standard approach is outside the expected profile for this framework.

---

## Selection Flow

First run of `/gen-stories` for a project: if `projects/{PROJECT_CODE}/standards/estimation.md` does not exist, the command surfaces the question before generating any stories:

> "No estimation standard is set for this project. Which sizing model will you use?
> 1. T-shirt sizing — XS / S / M / L / XL [recommended]
> 2. Fibonacci story points — 1 / 2 / 3 / 5 / 8 / 13
> 3. Time-based — hours per story
>
> The selected standard will be written to `standards/estimation.md` and used for all story sizing in this project."

After PO selection: the AI copies the chosen reference file to `projects/{PROJECT_CODE}/standards/estimation.md`. All subsequent `/gen-stories` and `/review-stories` runs read from that file silently.

The question is asked once per project, not once per epic.

---

## What Commands Change

| Command | Current reference | New reference |
|---------|------------------|---------------|
| `gen-stories.md` | `knowledge/process/estimation-rules.md` | `projects/{PROJECT_CODE}/standards/estimation.md` (with selection step on first run) |
| `review-stories.md` | `knowledge/process/estimation-rules.md` | `projects/{PROJECT_CODE}/standards/estimation.md` |
| `assess-change.md` | `knowledge/process/estimation-rules.md` | `projects/{PROJECT_CODE}/standards/estimation.md` |

---

## Why No Community Contribution

The estimation space is not open-ended. Three models cover the practical universe. Community contribution here would produce noise — minor variations of existing models that add complexity without adding capability. The framework's role is to provide the options and let the PO choose; not to build a marketplace for sizing methodologies.

---

## Implementation Status

Complete (verified 2026-07-13). `reference/estimation/tshirt-sizing.md`, `fibonacci-points.md`, and `time-based.md` all exist; `gen-stories.md`, `review-stories.md`, and `assess-change.md` all read from `projects/{PROJECT_CODE}/standards/estimation.md` as decided, with the first-run selection step in place. `projects/ASCENDRA-PAY-001/standards/estimation.md` exists as a real example of the selected standard.
