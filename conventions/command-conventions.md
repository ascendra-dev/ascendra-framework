# Command Conventions

Verifiable rules for all Ascendra slash commands. Apply these when writing a new command and when modifying an existing one. Each rule is independently checkable with a yes/no.

**Governing principle:** These conventions exist for consistency only. Functional quality, instructional clarity, and the command's intent always take precedence. Where following a convention would compromise a command's purpose, add the exception to this document under the relevant rule — then apply it consistently across all commands that need it. Do not document exceptions inside individual command files.

---

## Section 1 — File Shape

**C-001** The H1 title is the command's display name in Title Case — not the slash command syntax.
- Correct: `# Generate BRD`, `# Review Epics`, `# Init Project`
- Wrong: `# gen-brd`, `# review-epics`

**C-002** The intro block (H1 through the first `---`) contains 1–3 short paragraphs. Each addresses one of: what the command does, when it runs in the lifecycle, what it produces. No step content in the intro.

**C-003** `## Arguments received` with `` `$ARGUMENTS` `` on the next line is always the first H2 section — before Step 1. Present in every command, even if no arguments are expected.

**C-004** Steps use H2 with this exact format: `## Step N — [Descriptive Name]`. The dash and descriptive name are always present. No step is just `## Step N`.

**C-005** `---` separates the intro block from the Arguments section, and separates every step from the next.

---

## Section 2 — Step Sequence and Gate Checks

**C-010** Step 1 parses `$ARGUMENTS`. It names required and optional fields. For each required field that is missing, it provides a blockquote error message asking for that field only.

**C-011** Gate checks are positioned by command type:
- Generation commands (`gen-*`, `init-*`): dedicated `## Step 1.5 — Gate check` between argument parsing and file reads.
- Session commands (`run-*`): dedicated `## Step 2 — Gate check`.
- Review commands (`review-*`): gate check embedded within Step 1.
- **Exception:** `anchor-project` (and any future cross-cutting orchestration command that invokes other commands rather than performing one artifact's own generation/review/session work) follows the session-command placement — dedicated `## Step 2 — Gate check` — since it is fundamentally a resumable, multi-turn session, just not named `run-*`.

**C-012** Gate checks with 3 or more conditions use the table format:
```
| Check | How to verify | Failure message |
|-------|--------------|----------------|
```
Gate checks with 1–2 conditions may use if/else bullets. Do not mix both formats in the same gate check block.

**C-013** File reading is a dedicated step with a numbered list. Each entry: `` `file path` `` — purpose in one phrase. Restrictions on what NOT to load go at the end of the step.
- **Exception:** `anchor-project`'s file needs are inherently sequence-dependent — it may resolve a project via `projects/index.md`, reconcile state against whichever artifacts the project has reached, or inherit an invoked command's own file-reading step, all within one run. Its reads are distributed across the steps that actually need them rather than consolidated into one dedicated step. This exception does not extend to commands whose inputs are fixed and knowable in advance — those still use one dedicated file-reading step.

**C-013a** Template files (`projects/TEMPLATE/`) may appear in a command's file read list only in two cases:
1. The command is a `gen-*` or `init-*` command reading the template for the artifact it is generating
2. The template explicitly addresses this command in a "When used by `/command`" block in its document-level AI Guide

All other commands (`run-*`, `review-*`, `implement-*`, `verify-*`) must not read template files. Rules or criteria sourced from templates must be embedded directly in the command text instead.

**C-014** Input gathering uses a two-column table:
```
| Input | What to ask if missing |
|-------|----------------------|
```
The instruction must say all missing inputs are requested "all at once" — never one at a time across multiple turns.

**C-015** Before writing any primary artifact, a planning summary or table is presented and the PO must confirm. Files are not written without this confirmation. The confirmation prompt uses this exact format:

```
> "Does this [breakdown / scope / structure] look right? [what to do if not]"

Wait for PO response. Do not write any files until confirmed.
```

**C-016** Missing argument prompts follow these phrasing templates:

- **Single missing project code:**
  ```
  > "Which project is this [artifact] for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."
  ```
- **Single missing file path:**
  ```
  > "Which [file type] should I use? Provide the path (e.g. `projects/{CODE}/folder/file.md`)."
  ```
- **Multiple missing args:** one batch blockquote, numbered list, bold label per item:
  ```
  > "To [action] I need:
  > 1. **Label** — description (e.g. `example`)
  > 2. **Label** — description (e.g. `example`)"
  ```

Do not ask for one missing arg, receive a response, then ask for the next. All missing items are surfaced in a single blockquote.

---

## Section 3 — User-Facing Messages

**C-020** All messages from the AI to the PO are in blockquotes: `> "message"`. No user-facing message appears as plain prose.

**C-021** Every pause for PO input has an explicit wait instruction immediately after the message. Standard form:
- "Wait for PO response."

Other forms ("Wait for confirmation.", "Wait for PO confirmation before continuing.") are acceptable where the standard form would be ambiguous, but the standard form is preferred throughout.

**C-022** Sub-items within a step use **bold text**, not `###` or lower heading levels. Conditional branches (fresh session / resumed / complete) are bold labels on their own line, not headings.
- **Exception:** The per-artifact loop in review commands may use `### A` through `### F` for its sub-steps, because the loop body is long enough that heading-level separation aids navigation. H3 is only valid in this context.

**C-023** Multi-option choices (when the PO must select one of several options) use this format:

```
> "Question text:
>
> **(a) Label** — description
> **(b) Label** — description
> **(c) Label** — description"

Wait for PO response.
```

Rules:
- Bold the letter and the label together: `**(a) Label**`, not `**(a)**` alone
- One option per line within the blockquote
- Standalone blockquote — never embedded in a table cell
- `Wait for PO response.` immediately after the closing blockquote
- When a multi-option choice appears alongside other inputs in a gather step, move it out of the input table into its own instruction block after the table. Do not inline multiple options within a table cell.

**C-024** PO confirmations and multi-option questions are always rendered as blockquote text (per C-020/C-023) — never via an interactive selection UI/widget, even when the execution environment offers one (e.g. a clickable-option tool). This keeps every command's interaction identical regardless of how or where it's run.

---

## Section 4 — Structural Checks and Fix Tiers *(review commands only)*

**C-030** Checks are a numbered list. Format: `N. **Title** — description on the same line.`
- Wrong: standalone bold heading (`**Check 1 — Title**`) followed by a paragraph on the next line.
- Right: `1. **Title** — description here on the same line.`

**C-031** Fix tiers use bold label + bullet list of examples:
```
**Tier N — [Label]:**
- example of what triggers this tier
- example of what triggers this tier
```
No tier uses a paragraph sentence instead of bullet examples.

**C-032** In the per-artifact walkthrough, the capability description (real-world framing) is presented to the PO before FLAGS. FLAGS are never surfaced before the PO has confirmed they understand what will be built.

**C-033** FLAGS are presented in a fixed-format code block, not as prose or a table.

---

## Section 5 — Output and Close

**C-040** Every command ends with a final report in a fixed-format code block (not a blockquote, not a prose list). The first line is the command action in ALL CAPS.
```
COMMAND ACTION — {PROJECT_CODE}
─────────────────────────────────────────
Field:  value
Field:  value
─────────────────────────────────────────
```

The separator line is always `─────────────────────────────────────────` (Unicode U+2500 BOX DRAWINGS LIGHT HORIZONTAL, 41 characters). No other separator character is valid — `══════` and similar are not permitted.

- **Exception:** Session commands (`run-*`) that may hand off to another command immediately (e.g. `/run-brd-discovery` → `/gen-brd`) use a code block for the completion summary, then a separate blockquote for the conditional handoff offer. The code block is always present; the blockquote is conditional on session outcome.

**C-041** The next-step pointer appears inside or immediately after the final code block. It names the exact slash command to run next, with its argument syntax.

**C-042** After writing the primary artifact, the command explicitly updates all cross-reference files that track it (e.g. `brief.md` Section 9 Related Artifacts, `epics/index.md`, `stories/index.md`). This update step is stated in the command — not assumed.

**C-043** Any cross-reference to another artifact's file path, written into a *generated project document* (not this framework's own authoring docs — those are governed separately), is a real markdown link (`[relative/path](relative/path)`), computed relative to the citing document's own location — never a plain backtick-quoted path. This applies to `brief.md` Section 9 (Related Artifacts) and any equivalent registry/index table a command populates.

It does **not** apply to `.example.md` files: their citations reference a fictional project (`HARBORVIEW-INV-001`) that doesn't exist in this repo — a link there would be a guaranteed dead link, not an illustrative citation.

A template's (`projects/TEMPLATE/**`) `[AI Guide]` prose citing another framework file for guidance purposes is also exempt (that prose is stripped from generated output and never resolves to a real path). But a template's own **generation skeleton** — the literal section content, using `{PROJECT_CODE}`-style substitution syntax, that gets copied directly into every real generated document — follows C-043 like any other generated content: it's not illustrative, it's the actual pattern every real document is built from, and it resolves to a real path the moment `{PROJECT_CODE}` is substituted for a real project. `brief.template.md` Section 9's own example rows are this case.

---

## Checklist for Verifying a Command

Use this before marking any new or modified command ready:

- [ ] C-001  H1 title is display name in Title Case
- [ ] C-002  Intro: 1–3 paragraphs, no step content
- [ ] C-003  `## Arguments received` + `` `$ARGUMENTS` `` present before Step 1
- [ ] C-004  All steps: `## Step N — [Name]` format
- [ ] C-005  `---` between every section
- [ ] C-010  Step 1 parses arguments and names required/optional fields
- [ ] C-011  Gate check in correct position for this command type
- [ ] C-012  Gate check format consistent within the block (table or bullets, not both)
- [ ] C-013  File reading is a dedicated step with numbered list
- [ ] C-013a  Template reads present only in gen-*/init-* commands, or where the template explicitly addresses this command in a "When used by" block
- [ ] C-014  Input table present; "all at once" instruction present
- [ ] C-015  Pre-artifact confirmation step present; uses blockquote + "Wait for PO response. Do not write any files until confirmed."
- [ ] C-016  Missing arg prompts use the correct phrasing template (project code / file path / batch)
- [ ] C-020  All PO-facing messages in blockquotes
- [ ] C-021  Explicit wait instruction at every PO input point
- [ ] C-022  Sub-items use bold text (not `###`), exceptions noted
- [ ] C-023  Multi-option choices use **(a) Label** format, one per line, standalone blockquote, not embedded in table cell
- [ ] C-024  PO confirmations rendered as blockquote text, never an interactive selection widget
- [ ] C-030  *(review)* Checks are numbered list with inline descriptions
- [ ] C-031  *(review)* Fix tiers use bold label + bullet examples
- [ ] C-032  *(review)* Capability walkthrough before FLAGS
- [ ] C-033  *(review)* FLAGS in fixed-format code block
- [ ] C-040  Final report in fixed-format code block; separator is `─────────────────────────────────────────` only
- [ ] C-041  Next-step pointer in or after final code block
- [ ] C-042  Cross-reference update step explicit in command
- [ ] C-043  Cross-references written into a generated document are real markdown links, not backtick text — never applied to templates or `.example.md` files
