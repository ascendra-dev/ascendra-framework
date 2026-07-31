# Run Intake

You are running a structured intake session to complete the project brief for a given project. The brief is the foundation of all downstream work — domain knowledge, discovery, BRD, architecture. It must be complete and approved before `/gen-domain-knowledge` can run.

The intake session may span multiple interactions. Each section is written to `brief.md` immediately on PO approval — progress is never lost between sessions.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract **PROJECT_CODE** from `$ARGUMENTS` (e.g. `HARBORVIEW-INV-001`).

If no argument is provided, ask:
> "Which project is this intake for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

---

## Step 2 — Load files

Check that `projects/{PROJECT_CODE}/brief.md` exists.

If it does not exist, stop immediately:
> "`projects/{PROJECT_CODE}/brief.md` not found. Run `/init-project {PROJECT_CODE}` first to create the project structure, then re-run `/run-intake`."

Read these files in full before proceeding:
1. `projects/{PROJECT_CODE}/brief.md` — the state: what is already filled
2. `projects/TEMPLATE/brief.template.md` — the question guide: [AI Guide] notes, correct/incorrect examples, and Verification checklist

Do not read `projects/TEMPLATE/brief.example.md` into the session — it is available to the PO as a manual reference only.

---

## Step 3 — Determine session mode

Read the **Content Status** field at the top of `brief.md` (not **Status** — that's the separate project-lifecycle field).

**Amendment mode** — if **Content Status** reads `Approved` (set by `update-status`):
> "The brief for `{PROJECT_CODE}` is Approved. Any change made in this session will be recorded as a tracked amendment in the document history table. Do you want to proceed?"
>
> Wait for confirmation. If the PO confirms, set an internal amendment flag, then ask:
> "Which section do you want to change?"
>
> Go directly to the per-section flow in Step 5 for the named section. The amendment flag ensures Step 9 (amendment tracking) runs after each write.

**Revision mode** — if the status note reads `Brief complete`:
> "The brief for `{PROJECT_CODE}` is already complete. Which section do you want to revise?"
>
> Go directly to the per-section flow in Step 5 for the named section. Do not re-run verification unless the PO asks.

**Intake mode** — if the status note reads `Draft` or `awaiting intake session`:
Continue to Step 4.

---

## Step 4 — Build and report the completion map

A section is **complete** (✅) if it contains real content — paragraph text, a table, or a list that is not a placeholder string.

A section is **blank** (⬜) if:
- It contains the awaiting-intake placeholder text, OR
- The section body is empty or contains only whitespace

Report the map before asking anything:

```
INTAKE — {PROJECT_CODE}
{Project Name} · {Client}

Section status:
  ✅ Header fields     [pre-filled by /init-project]
  ⬜ Extension Context
  ⬜ 1. Problem Statement
  ⬜ 2. Client Context
  ⬜ 3. Strategic Goals
  ⬜ 4. Key Stakeholders
  ⬜ 5. Constraints
  ⬜ 6. Technology Preferences
  ⬜ 7. Success Criteria
  ⬜ 8. Risks and Open Questions
  ⬜ 9. Related Artifacts

Starting with Extension Context, then Section 1.
```

If resuming (some sections already ✅):
> "Resuming intake for {Project Name}. Sections N, N are complete. Picking up from Section N — {Section Name}."
>
> Offer a light review: "Want to review any completed section before we continue, or go straight to Section N?"

---

## Step 5 — Run the intake session

**First — Extension Context (if not yet filled):**

Check if the Extension Context fields in `brief.md` are blank (containing placeholder text or not yet written). If so, ask:

> "Is this a standalone project, or does it build on top of an existing Ascendra project?"

- **Standalone:** Write `Project Type: Standalone`, `Extends: —`, `Extension Adds: —` to the header fields in `brief.md`. Confirm: "Noted — standalone project. Moving to Section 1."
- **Extension:** Ask: "Which project does this extend? Provide the project code (e.g. `ASCPAY-CORE-001`)." Then: "In one sentence, what does this project add on top of that?" Fill the header fields accordingly. Verify the parent project code exists in `projects/` — if not, flag it: "Project `{code}` not found in `projects/`. Confirm the project code or add the parent project to this workspace first."

Write the filled Extension Context fields to `brief.md` immediately after confirming.

**Output format:** write section content directly to `brief.md` — do not copy the `Part 1 — Template` or `Part 2 — Verification` labels from `brief.template.md` into the output file. The brief is a clean document: header fields, section headings, and content only.

**Then work through each blank section in order (1 → 9).** Skip sections already marked ✅, unless the PO asks to revise one.

### Per-section flow

**A — Present the Guide**

Before asking anything, read the section's `[AI Guide]` note(s) from `brief.template.md` — there may be more than one stacked block (e.g. Section 2 has both a `[AI Guide — Client definition]` note and a separate `[AI Guide]` note). Reproduce their **structure and specificity in full** — every numbered question, lettered sub-case, and bulleted checklist item — but rewrite the **voice**: this is guidance for the PO, not an instruction manual for whoever drafts the section. Concretely:

- Drop the `[AI Guide]` / `[AI Guide — Label]` bracket headers themselves — they are template-authoring labels, not PO-facing content
- Where a note instructs the drafter how to write (e.g. "Adjust your language accordingly," "a developer or AI agent joining mid-project would understand..."), rewrite it as a direct statement or question to the PO instead of removing it — the underlying requirement stays, only the addressee changes
- Keep every numbered list, lettered case, and bulleted checklist exactly as structured in the template
- Keep any Correct/Incorrect example as a direct, verbatim quote — these are meant to be read exactly as written

Label it clearly:

```
────────────────────────────────────────
Guide — Section N: {Section Name}
{full guidance, same structure as the template, in PO-facing voice}
────────────────────────────────────────
```

**B — Ask the opening question**

Ask a single open question. Do not list sub-questions. Let the PO speak freely first. Opening questions are in Step 6.

**C — Follow up if thin**

If the PO's response is missing sub-points that the [AI Guide] note says the section must cover, ask one targeted follow-up at a time. Never ask multiple follow-ups at once.

**D — Draft**

Synthesise the PO's responses into the correct format for that section (paragraph, table, or list as appropriate). Present it in a clearly labelled block:

```
────────────────────────────────────────
Draft — Section N: {Section Name}
{content}
────────────────────────────────────────
```

**E — Approval gate**

> "Does this look right? **Approve** to save and move on, **Revise** to give me feedback and I'll redraft, or **Skip** to come back to this section later."

- **Approve** → write the content to `brief.md` immediately, replacing the placeholder or empty body. Update `**Last Updated:**` to today's date. Confirm: "Section N saved." Move to the next blank section.
- **Revise** → take the PO's feedback, redraft, return to step D.
- **Skip** → leave the section blank. Note internally that it was skipped. Move to the next section.

### Mid-session revision

If the PO says they want to go back to a previously completed or skipped section at any point: stop the current section, go to the named section, present its current content and [AI Guide] note, and work through steps D–E again. On approval, write the revised content and return to the section the session was on.

---

## Step 6 — Opening questions per section

| Section | Opening question |
|---------|-----------------|
| 1. Problem Statement | "Describe what's broken or missing today — what the client is living with right now." |
| 2. Client Context | "Tell me about the client — who they are, what they do, and how they operate today." |
| 3. Strategic Goals | "What does the client actually want to achieve? Think outcomes, not features." |
| 4. Key Stakeholders | "Who are the key people involved — names, roles, and how involved will they be in the project?" |
| 5. Constraints | "What are the hard constraints — budget, deadline, mandated technology, regulatory requirements?" |
| 6. Technology Preferences | "What systems or tools do they already use, and do they have any preferences on stack or hosting?" |
| 7. Success Criteria | "How will you know this worked? What does the system let the user do that they couldn't before?" |
| 8. Risks and Open Questions | "What could go wrong, and what's still unknown before delivery starts?" |
| 9. Related Artifacts | Auto-populate — see Step 7. |

---

## Step 7 — Section 9 (Related Artifacts)

Do not ask the PO a question for Section 9. Auto-populate it:

```markdown
### 9. Related Artifacts

| Artifact | Location | Status |
|----------|----------|--------|
| Project Brief | `projects/{PROJECT_CODE}/brief.md` | Active |
```

Present it:
> "Section 9 has been pre-populated with the brief as the first artifact. As the project advances, new artifacts — BRD, architecture, sprint plans — should be added to this table.
>
> To keep it up to date: the recommended way is to run `/update-status` after each artifact is created or changes status — it updates this table automatically. You can also update `brief.md` directly at any time if you prefer to manage it manually.
>
> Does this look right?"

On approval, write to `brief.md`.

---

## Step 8 — Session close

### If any sections were skipped

Do not run verification. Report:

```
INTAKE PAUSED — {PROJECT_CODE}
─────────────────────────────────────────
The following sections are still pending:
  ⬜ Section N: {name}
  ...

Brief status: Draft (unchanged)
Run /run-intake {PROJECT_CODE} to complete the remaining sections.
The brief cannot be approved until all sections are filled.
```

Stop here.

### If all sections are complete

Run every check in the **Verification** section of `projects/TEMPLATE/brief.template.md` against the content of `brief.md`.

For each check that fails, report the specific gap. Work through each flag with the PO — revise the relevant section and write the fix to `brief.md` before moving on. Do not close the session until all checks pass.

Once all verification checks pass:

1. Append a `## Verification` section to the end of `brief.md` (after Section 9), with every checklist item from the template's Part 2 checked off (`[x]`). Strip the `[AI Guide]` note and the `Part 2 —` prefix — the heading is `## Verification`, matching `projects/TEMPLATE/brief.example.md`.

2. Update the **Content Status** header field (not the **Status** field — that's the separate project-lifecycle field, untouched here) from `Draft` to `Brief complete`.

3. Update `**Last Updated:**` to today's date.

4. Report:

```
INTAKE COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
All sections filled and verified.

Next steps:
  1. Review brief.md and approve it:
     update-status projects/{PROJECT_CODE}/brief.md Approved
  2. Then run /gen-domain-knowledge to generate the domain knowledge base.
  3. For niche or complex domains: run /gen-domain-playbook then /run-domain-discovery.
     For well-known domains: run /gen-brd-playbook directly to move to requirement discovery.
```

---

## Step 9 — Amendment tracking (amendment mode only)

If this session was flagged as an amendment (brief was Approved at the start), after writing any change to `brief.md` ask the PO for a one-line summary of what changed and why.

Append an entry to the document history table at the bottom of `brief.md`. If the table does not exist yet, create it first:

```markdown
---

## Document History

| Version | Date | Changed by | Summary |
|---------|------|------------|---------|
| 1.1 | {today} | Product Owner | {PO's summary} |
```

Increment the minor version number for each amendment (1.1, 1.2, etc.).
