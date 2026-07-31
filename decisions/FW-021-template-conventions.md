# FW-021 — Template Conventions

| Field | Value |
|-------|-------|
| **ID** | FW-021 |
| **Date** | 2026-06-30 |
| **Status** | Decided |
| **Area** | Framework-wide — all document templates in `projects/TEMPLATE/` |

---

## Decision

A formal, verifiable convention set is established for all Ascendra document templates. The full rule set — 19 rules across 5 sections, governing principle, exception model, and verification checklist — lives in `conventions/template-conventions.md`. That file is the authoritative reference; this record captures why the decision was made and what was corrected on first application.

---

## The Problem That Triggered This Decision

Following the command conventions audit (FW-020), a parallel consistency audit was run across all 10 Phase 1–5 templates. The audit found 12 actionable inconsistencies across 4 dimensions:

- **Placeholder syntax:** Three forms of the same variable in use simultaneously — `{PROJECT_CODE}`, `{PROJECT-CODE}`, and `{project-code}` — with no rule distinguishing them. `arch.template.md` contained all three forms.
- **Verification qualifiers:** `[AI Guide — Verification]` used in only 2 of 8 templates with verification sections; the other 6 used plain `[AI Guide]` for the same purpose.
- **Document Control dates:** `story.template.md` and `arch.template.md` contained pre-filled real dates (2026-06-20) rather than `[Date]` placeholders. These dates would propagate into every generated document.
- **Missing Status fields:** The `story` template had no Status field, despite stories going through a formal review gate (`/review-stories`).
- **Missing verification sections:** Both discovery-state templates had no pre-generation verification, leaving no gate before `/gen-domain-knowledge` or `/gen-brd` consumed them.
- **Inconsistent AI guidance:** Only `brief.template.md` had a `**What this document is not:**` block and Correct/Incorrect examples in the document-level AI Guide. Epic, story, and BRD templates had equivalent high-failure sections with no examples.

These inconsistencies created three problems:
1. **AI generation drift** — without a canonical qualifier vocabulary and consistent examples, AI-generated documents varied in quality and framing depending on which template was loaded
2. **No generation gate** — discovery-state files fed directly into generation commands with no verification step, so incomplete session records could silently produce partial artifacts
3. **Template pollution** — pre-filled real dates meant every generated document opened with incorrect version history

---

## Scope of Application

Conventions were audited and applied retroactively to all Phase 1–5 templates during the post-FW-020 session:

**P1 fixes — material errors, applied immediately:**

| Template | Fix applied |
|----------|-------------|
| `arch.template.md` | `{PROJECT-CODE}` → `{PROJECT_CODE}` in file location note |
| `arch.template.md` | Removed Document Control block with pre-filled dates (Change History inside Part 1 is the correct version record) |
| `story.template.md` | Replaced three pre-filled date rows with `[Date]` / `[Author]` placeholder |
| `story.template.md` | Added `**Status:** Draft / Reviewed / Approved` field |
| `domain-discovery-state.template.md` | Added `## 8. Pre-Generation Verification` (6 gate checks before `/gen-domain-knowledge`) |
| `brd-discovery-state.template.md` | Added `## 10. Pre-Generation Verification` (7 gate checks before `/gen-brd`) |

**P2 fixes — AI guidance gaps, applied in the same pass:**

| Template | Fix applied |
|----------|-------------|
| `arch.template.md` | `[AI Guide]` → `[AI Guide — Verification]` in Part 2 |
| `epic.template.md` | `[AI Guide]` → `[AI Guide — Verification]` in Part 2 |
| `story.template.md` | `[AI Guide]` → `[AI Guide — Verification]` in Part 2 |
| `epic.template.md` | Added `**What this document is not:**` to document-level AI Guide |
| `story.template.md` | Added `**What this document is not:**` to document-level AI Guide |
| `brd.template.md` | Added Correct/Incorrect examples to Section 1.3 (Business Goals) |
| `epic.template.md` | Added Correct/Incorrect examples to Section 1 (Goal) |
| `story.template.md` | Added Correct/Incorrect examples to Section 1 (Story Statement) |

**P3 — Codified as conventions only, no template edits required:**

- Canonical qualifier vocabulary (T-003): four valid qualifiers defined; the vocabulary was already implicit but not written down
- Square bracket disambiguation (T-022): three uses distinguished — fill-in placeholder, AI instruction, literal source tag
- Empty vs hint cell choice (T-023): both forms allowed; hint form preferred

---

## Key Conventions Established

**Discovery-state exceptions:** These files are working session records, not deliverable artifacts. Two explicit exceptions apply:
1. No Document Control table — Session History (Section 2) serves their continuity tracking purpose instead
2. No Status field — state is tracked via Confidence Score and Playbook Progress, not a lifecycle field

**Two placeholder forms:** `{PROJECT_CODE}` (uppercase with underscores) is the project identifier; `{project-code}` (lowercase hyphenated) is the slug used only in folder and repository names. `{PROJECT-CODE}` is not a valid form.

**Document Control date rule:** Document Control rows in templates always use `[Date]` / `[Author]` placeholders. Pre-filled real dates are a defect — they propagate into every generated document and are immediately stale.

---

## Relationship to Other Decisions

- **FW-020** (Command Conventions) — Template conventions are the parallel standard for the document layer. Together, FW-020 and FW-021 cover the two primary authoring surfaces of the framework.
- **FW-004** (AI Guide Label) — Establishes the `[AI Guide]` label itself. FW-021 extends that decision by defining the qualifier vocabulary (`— Document Level`, `— Verification`, `— Section Instructions`).
- **FW-003** (Template and Example Conventions) — Earlier decision governing template existence and naming. FW-021 does not supersede FW-003; it governs content structure within templates, not file naming or the template/example split.
