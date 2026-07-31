# Generate UAT Checklist

You are generating a User Acceptance Testing (UAT) checklist for a completed sprint. The checklist is the structured test sheet the Product Owner uses to verify every acceptance criterion before signing off the sprint. It must be written for a non-technical reader — plain language, clear steps, unambiguous expected results.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a project path and a sprint number. Examples:

```
/gen-uat-checklist projects/ASCENDRA-PAY 01
/gen-uat-checklist projects/ASCENDRA-PAY 07
```

Extract:
- **Project path** — e.g. `projects/ASCENDRA-PAY`
- **PROJECT_CODE** — from project path (e.g. `ASCENDRA-PAY`)
- **SPRINT_NUMBER** — strip whitespace, then zero-pad to two digits (e.g. `1` → `01`, `7` → `07`, `01` stays `01`). State the normalised value: "Sprint number: {NN}"

If missing, ask:
> "Provide project path (e.g. `projects/ASCENDRA-PAY`) and sprint number (e.g. `01`)."

---

## Step 2 — Gate check: all sprint stories must be Merged

Read `projects/{PROJECT_CODE}/stories/index.md`. Find all story IDs assigned to Sprint {NN} in Section 8 (Sprint Planning table). Expand ranges and comma-separated lists.

For each story ID, check its status in Section 1 (Story Registry):

- If ALL stories are `Merged` or `Done` → proceed
- If ANY story is not yet `Merged` or `Done` → stop:
  > "Cannot generate UAT checklist for Sprint {NN} — the following stories are not yet Merged: [list with current status]. UAT can only begin after all sprint stories are implemented and merged."

---

## Step 3 — Read the UAT checklist template

Read `projects/TEMPLATE/sprints/uat-checklist.template.md` in full — Part 1 (Template) and Part 2 (Verification). This is the document structure to follow exactly in Step 6: header fields, How to use this checklist, Sprint Summary, one per-story section with a scenario table, and UAT Sign-off.

---

## Step 4 — Read all sprint story files

For each story ID in the sprint:

Read the full story file at `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/{story-filename}.md`.

Extract:
- **Story title**
- **Story statement** (Section 1) — for context header
- **Acceptance criteria** (Section 3) — one UAT scenario per AC
- **Notes** (Section 7) — for any edge case context you should explain to the PO

---

## Step 5 — Generate the UAT checklist

Follow `projects/TEMPLATE/sprints/uat-checklist.template.md` exactly — one section per story (in sprint plan delivery order), one scenario row per acceptance criterion, scenario IDs in `{Story-ID}-TC-{N}` format. The template's AI Guide covers translating an AC into plain PO-executable steps (numbered, single-action, observable expected results) — apply it per AC; do not skip, merge, or invent scenarios beyond what the story's ACs state.

---

## Step 6 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}-uat-checklist.md`

Populate every section of the template read in Step 3, in order. Leave the UAT Sign-off fields (Overall result, Failing scenarios, PO Signature, Sprint approved) blank — the PO completes these after running the checklist.

**Output format:** the generated checklist does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Remove the `## Part 1 — Template` label — the checklist content starts directly after `## Document Control`. The `## Part 2 — Verification` checklist is not carried into the output at all, renamed or otherwise — it is an internal pre-write self-check only. This matches `projects/TEMPLATE/sprints/uat-checklist.example.md`, which has neither a `Part 1` label nor any Verification section.

---

## Step 7 — Report to the user

After writing the file:

```
UAT CHECKLIST GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Sprint:            {NN}
Output:            projects/{PROJECT_CODE}/sprints/sprint-{NN}-uat-checklist.md
Stories covered:   {N}
Total scenarios:   {N}
─────────────────────────────────────────
Next steps:
1. Send the checklist to the Product Owner alongside UAT environment access.
2. PO works through each scenario and marks Pass/Fail.
3. Any Fail is raised as a defect. Critical/High defects block sprint closure.
4. When all scenarios pass: PO signs the sign-off section, then run
   /close-sprint projects/{PROJECT_CODE} {NN} to prepare the retrospective.
5. After retrospective is complete, run
   /update-status projects/{PROJECT_CODE} sprint {NN} Complete.
```
