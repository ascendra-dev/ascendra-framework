# Template Conventions

Verifiable rules for all Ascendra document templates. Apply these when authoring a new template and when modifying an existing one. Each rule is independently checkable with a yes/no.

**Governing principle:** These conventions exist for consistency only. A template's primary job is to guide accurate AI generation. Where following a convention would compromise a template's instructional clarity, add the exception to this document under the relevant rule — then apply it consistently across all templates that need it.

---

## Section 1 — AI Guide Notes

**T-001** Every template opens with an `[AI Guide — Document Level]` note immediately after the H1. This note covers:
- What the document is, and when in the lifecycle it is produced
- What command produces it and what gate it enables
- **What this document is not** — a short list of things that belong elsewhere, to prevent scope creep into the template
- The file location pattern for the generated document

**T-002** The `[AI Guide — Document Level]` note always uses the qualified form:
```
> **[AI Guide — Document Level]**
```
Plain `> **[AI Guide]**` is not valid for the document-level note.

**T-003** Section-level AI Guide notes use this qualifier vocabulary. No other qualifiers are valid:

| Qualifier | When to use |
|-----------|-------------|
| `[AI Guide]` | General instruction for filling a section — the default for most sections |
| `[AI Guide — Verification]` | The Pre-Generation Verification section or Part 2 Verification section only |
| `[AI Guide — Document Level]` | The opening note at the H1 level only |
| `[AI Guide — Section Instructions]` | Complex sections in playbooks where the generation logic is multi-step (e.g. discovery session section walkthroughs) |

**T-004** AI Guide notes at the document level include a `**What this document is not:**` block. This block names one or more things a developer or AI might put in this document that belong elsewhere — with a pointer to where they do belong.
- **Exception:** Discovery-state files omit this block. They are working session records, not deliverable artifacts, and the "what this is not" boundary is already implied by their purpose.

**T-005a** Templates read by more than one command with distinct purposes must include explicit audience blocks inside the document-level AI Guide. One block per reading command, using this format:

```
> **When used by `/command-name`:** [What this command reads the template for. What it must and must not do with it.]
```

Rules:
- The block addresses the command's reading purpose only — it does not describe what the generated artifact is for
- If one command writes the artifact and another reads it later (e.g. discovery-state files), use `**When writing this file (`/command`)**` and `**When reading this file (`/command`)**` instead
- Single-audience templates (read only by their gen command) do not need these blocks

Templates with legitimate dual-audience reads as of FW-021:
- `brief.template.md` — read by `/init-project` (structure) and `/run-intake` (guidance + verification)
- `domain-discovery-state.template.md` — written by `/run-domain-discovery`, read by `/gen-domain-knowledge`
- `brd-discovery-state.template.md` — written by `/run-brd-discovery`, read by `/gen-brd`

**T-005** AI Guide notes for high-failure sections include `**Correct:**` and `**Incorrect:**` paired examples. A section is high-failure if it requires a judgment call that commonly produces generic, untestable, or scoped-wrong output. Required in:
- BRD Section 1.3 (Business Goals) — measurability failure common
- Epic Section 1 (Goal) — technical deliverables framing common
- Story Section 1 (Story Statement) — compound stories and feature-area descriptions common
- Brief Part 1 (all sections) — already present via correct/incorrect examples in the brief template

The `**Correct:**` / `**Incorrect:**` lines go inside the AI Guide blockquote, after the rule text and before the fill-in placeholder.

---

## Section 2 — Document Control and Status

**T-010** Templates for deliverable artifacts (brief, BRD, epic, story, architecture, domain knowledge, domain playbook, BRD playbook, sprint plan, UAT checklist, PR description, test execution report) include a `## Document Control` table with this structure:

```markdown
## Document Control

> **[AI Guide]** Add a row each time this document is revised. Version 1.0 is the initial draft.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |
```

Rules:
- The initial row uses `[Date]` and `[Author]` as fill-in placeholders — never pre-filled dates
- The AI Guide note says when to add rows (at every revision)
- Pre-filled real dates in a template are a defect — they appear in every generated document

**T-011** Discovery-state files (domain-discovery-state, brd-discovery-state) do not include a `## Document Control` table. They use `## Session History` (Section 2) as their continuity tracking mechanism, which better reflects their nature as multi-session working documents.

**T-011a** UI mock files (`ui-mock.template.md`) do not include a `## Document Control` table. A UI mock has no independent version history — its lifecycle is entirely inherited from the architecture document it visualises (Section 3.1.3), so a second, parallel version history would duplicate and could drift from the one already on `arch-v{N}.md`.

**T-012** Templates for deliverable artifacts that go through a formal approval gate include a `**Status:**` field in the document header. Status fields and their allowed values:

| Template | Status values |
|----------|---------------|
| brief | `Draft / Brief complete / Approved` |
| BRD | `Draft / Under Review / Approved / Locked` |
| Domain knowledge | *(check domain template for canonical values)* |
| Screen Design | `Draft / Under Review / Approved` |
| Architecture | `Draft / Under Review / Approved / Locked` |
| Epic | `Draft / Approved / In Progress / Done` |
| Story | `Draft / Reviewed / Approved` |
| Sprint plan | `Planning / Active / Complete` |

**T-013** Playbook templates (domain-playbook, brd-playbook) do not include a `**Status:**` field. Playbooks are process tools, not gated artifacts — they do not have an approval lifecycle.

**T-014** Discovery-state files do not include a `**Status:**` field. Their state is tracked via the Domain Confidence Score (domain-discovery-state Section 3) and the Playbook Progress table (Section 3/4).

**T-014a** UI mock files do not include a `**Status:**` field, for the same reason as T-011a — status is inherited from the architecture document (Draft while architecture is Draft/Under Review, frozen when architecture Locks), not tracked independently.

**T-014b** UAT checklist, PR description, and test execution report templates do not include a `**Status:**` field. None of the three carries an independent approval lifecycle: a UAT checklist's state is the Pass/Fail marks the PO fills in per scenario, a PR description's state is the PR's own GitHub status, and a test execution report's recommendation (Section 5) is itself the terminal decision — none needs a separate gate label.

---

## Section 3 — Placeholder Syntax

**T-020** Two placeholder forms are valid. Use each for its specific context only:

| Form | Context |
|------|---------|
| `{PROJECT_CODE}` | The project identifier — uppercase, matches the code in `projects/`. Examples: `HARBORVIEW-INV-001`, `ASCENDRA-PAY` |
| `{project-code}` | The lowercase hyphenated slug used **only** in folder and repository names where lowercase is required — e.g. `{project-code}-api/`, `{project-code}-web/` |

`{PROJECT-CODE}` (hyphen separator in uppercase) is not a valid form. Replace with `{PROJECT_CODE}`.

**T-021** Fill-in placeholders for human content use square brackets: `[Date]`, `[Author]`, `[Epic Title]`. These are replaced by the AI when generating a document.

**T-022** Square brackets serve three distinct purposes in templates. They must not be confused:

| Form | Meaning | Must be replaced? |
|------|---------|-------------------|
| `[Date]`, `[Author]`, `[Description]` | Fill-in placeholder — AI replaces this | Yes |
| `[AI Guide — ...]` | Instruction to AI — not part of generated document | Removed on generation |
| `[Client-Stated]`, `[Domain-Default]`, `[Assumed]` | Source tag — stays in generated BRD as a literal label | No — kept as-is |

AI Guides clarify which category applies where the distinction might be ambiguous.

**T-023** Empty table cells and hint cells in tables: use the form that best matches the section's purpose:
- Fill-in cells use `[Description of what goes here]` as a hint
- Empty cells where the AI is expected to fill in content based on context use empty `|   |` — acceptable but hint form is preferred
- Rows explicitly labelled as examples (e.g. the first row in an extension point table) use concrete illustrative values — these are examples, not fill-ins

---

## Section 4 — Verification Sections

**T-030** Every template for a deliverable artifact includes a verification section. Two structural models are valid — choose based on the document type:

| Model | Used in | Structure |
|-------|---------|-----------|
| `## Part 2 — Verification` | brief, arch, epic, story, sprint plan, UAT checklist, PR description, test execution report | H2 section at the end of the template, after `## Part 1 — Template` |
| Numbered section | domain, brd, domain-playbook, brd-playbook | A numbered section (e.g. `## 12. Verification` or `## 11. Verification`) at the end |

Do not mix models within one template. The model used by a template type must remain consistent across all projects.

**T-031** The verification section AI Guide always uses the `[AI Guide — Verification]` qualifier:
```
> **[AI Guide — Verification]** Run every check below before...
```
Plain `[AI Guide]` is not valid in a verification section.

**T-032** Discovery-state files include a Pre-Generation Verification section as their final section. This section:
- Uses the `[AI Guide — Verification]` qualifier
- Is titled `## N. Pre-Generation Verification` (numbered to fit the document's section sequence)
- Lists checks that block generation — not review checks
- References the command that reads this file (`/gen-domain-knowledge` or `/gen-brd`)

**T-033** Verification checklist items are written as:
- Unchecked markdown boxes: `- [ ] ...`
- Each item names the section it checks and the specific failure condition
- Items do not describe remediation — they only describe the check

---

## Section 5 — Resume Instructions

**T-040** Session-state documents (discovery-state files) include a Resume Instructions section as the penultimate section — after all content sections and before the Pre-Generation Verification section.

**T-041** Resume Instructions sections state:
1. What to load before resuming (file paths, not vague references)
2. Where in the playbook the next session starts
3. What sections are still pending
4. Any open issues or follow-ups the PO committed to

---

## Checklist for Verifying a Template

Use this before marking any new or modified template ready:

- [ ] T-001  `[AI Guide — Document Level]` note is the first thing after the H1, before any `---`
- [ ] T-002  Document-level note uses `[AI Guide — Document Level]` qualifier — not plain `[AI Guide]`
- [ ] T-003  All AI Guide qualifiers are from the approved vocabulary (Document Level / Verification / Section Instructions / plain)
- [ ] T-004  `**What this document is not:**` block present in document-level note (or exception documented)
- [ ] T-005a  Multi-audience templates have explicit "When used by /command" blocks in the document-level AI Guide — one per reading command
- [ ] T-005  Correct/Incorrect examples present in all high-failure sections (BRD 1.3, Epic 1, Story 1, Brief sections)
- [ ] T-010  Document Control table present with `[Date]`/`[Author]` placeholders — no pre-filled real dates
- [ ] T-011  Discovery-state files: no Document Control (uses Session History instead)
- [ ] T-011a  UI mock files: no Document Control (lifecycle inherited from architecture document)
- [ ] T-012  Status field present for gated artifacts; uses the correct allowed values for this document type
- [ ] T-013  Playbook templates: no Status field
- [ ] T-014  Discovery-state files: no Status field
- [ ] T-014a  UI mock files: no Status field (inherited from architecture document)
- [ ] T-020  `{PROJECT_CODE}` used for project identifier; `{project-code}` used only in folder/repo name contexts; `{PROJECT-CODE}` not present
- [ ] T-021  Fill-in placeholders use `[square brackets]`
- [ ] T-022  Square bracket usages are clearly distinguished (fill-in vs instruction vs literal tag)
- [ ] T-030  Verification section present; correct model used for this document type (Part 2 or numbered section)
- [ ] T-031  Verification section uses `[AI Guide — Verification]` qualifier
- [ ] T-032  Discovery-state files: Pre-Generation Verification present as the final section
- [ ] T-033  Verification items are `- [ ]` format; each names a section and a failure condition
- [ ] T-040  Session-state documents include Resume Instructions as penultimate section
- [ ] T-041  Resume Instructions include: files to load, next session start point, pending sections, open follow-ups
