# Review BRD

You are running the BRD review for an Ascendra project. This is the mandatory quality gate between `/gen-brd` producing the Business Requirements Document and the Product Owner approving it. No epic generation begins until this review is complete and the BRD is Approved.

The BRD review is a guided walkthrough — not a report. You walk through functional requirements feature area by feature area, framing each in plain business language so the Product Owner can confirm it matches what was agreed with the client. Changes are applied immediately on agreement. Progress is recorded in Section 17 (Review Record) of the BRD — this is the single source of truth for review state, resume points, and the permanent change record.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Locate the BRD and run the gate check

If `$ARGUMENTS` contains a BRD file path, use it. If `$ARGUMENTS` contains a project code or project path, locate the most recent `brd-core-v*.md` in `projects/{PROJECT_CODE}/brds/`. If `$ARGUMENTS` is empty, ask:

> "Which project is this BRD review for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Read the BRD file. Check the **Status** field in Section 14 (Approval).

- If `Draft` or `Under Review` → proceed
- If `Approved` → stop:
  > "This BRD is already Approved. To make changes, run `/assess-change` describing what needs to change."
- If the BRD file does not exist → stop:
  > "No BRD found. Generate the BRD first using `/gen-brd {PROJECT_CODE}`."

---

## Step 2 — Read files before doing anything else

Read every file below in full before continuing.

1. The BRD file (located in Step 1) — already read in Step 1; load any sections not yet fully in context
2. The domain knowledge file referenced in the BRD, or in `projects/{PROJECT_CODE}/domain/` — read in full

If no domain knowledge file is found, proceed — frame requirements from the BRD content and domain knowledge from training for that domain.

---

## Step 3 — Review Record: initialise or resume

Check the BRD for **Section 17 (Review Record)**.

**Fresh session (Section 17 does not exist):**

This is a new review. Create Section 17 at the end of the BRD with the following structure. Pre-populate the Requirement Log with every reviewable item from Sections 5, 6, 8, and 9, all set to `Pending`.

```markdown
## 17. Review Record

**Review started:** {today's date}
**Review status:** In Progress

### Requirement Log

| ID | Section | Feature Area / Type | Review Status | Change Made |
|----|---------|---------------------|--------------|-------------|
| REQ-001 | 5.1 | [Feature Area Name] | Pending | — |
| REQ-002 | 5.1 | [Feature Area Name] | Pending | — |
| REQ-003 | 5.2 | [Feature Area Name] | Pending | — |
| BR-001  | 6   | Business Rule       | Pending | — |
| INT-001 | 8   | Integration         | Pending | — |
| SEC-001 | 9   | Security            | Pending | — |

### Flags

| # | Item | Status |
|---|------|--------|
| — | None | — |

### Review Summary

*(Filled on completion)*
```

After creating Section 17, continue with the pre-walkthrough checks below before starting Step 4.

**Pre-walkthrough checks (fresh session only):**

**Verification issues — Section 13.5:**

Read Section 13.5 (Verification Issues) of the BRD. If it has open or unresolved issues:

> "The BRD has [N] unresolved verification issue(s) logged during generation:
> [List each: ID, failed check, description]
> I will surface these again when we reach the relevant requirements."

Note each open issue — bring it up in context during the walkthrough when the relevant requirement is reached. If a verification issue remains unresolved by the end of the walkthrough, add it to the Flags table in Section 17.

If Section 13.5 is empty or all issues are marked Resolved: proceed.

**Open questions — Section 12:**

Read Section 12 (Open Questions) of the BRD. If it has questions marked as **scope blockers**:

> "Before we begin the walkthrough, Section 12 has [N] scope blocker(s) that must be resolved before this BRD can be approved:
> [List each scope blocker with its question and current status]
> Would you like to resolve these now, or walk through the requirements first and address them at the end?"

If the PO resolves them now: update Section 12, then proceed to Step 4.
If the PO prefers to continue first: note them as open and surface in Step 5.

If no scope blockers exist: proceed to Step 4.

**Parking Lot — Section 15:**

If Section 15 (Parking Lot) has any rows, mention them once, briefly — they are informational only and are not walked through line-by-line like Sections 5/6/8/9, since nothing there has been discovered yet:

> "Section 15 (Parking Lot) has [N] undiscovered idea(s) noted: [list IDs]. These aren't part of this review — they need a discovery pass before they can become requirements."

---

**Resumed session (Section 17 exists, status `In Progress`):**

Read the Requirement Log to find the **first row with status `Pending`**. That is the resume point.

> "Resuming BRD review. Progress so far: [N] confirmed, [N] changed, [N] removed, [N] still pending.
> Resuming from [REQ-ID / BR-ID / SEC-ID] — [brief description]."

Jump directly to Step 4 at the resume point. Pre-walkthrough checks were completed in the prior session — skip them.

---

**Review already complete (Section 17 exists, status `Complete`):**

> "The BRD review is already complete per Section 17. The BRD is ready for approval — type 'approved' to update the status, or raise any final concerns."

---

## Step 4 — Walkthrough

Walk through the BRD in this order:
1. Section 5 — Functional Requirements (feature area by feature area, one requirement at a time)
2. Section 6 — Business Rules (as a batch)
3. Section 8 — Integration Requirements (as a batch)
4. Section 9 — Security Requirements (as a batch)

**After every outcome** (confirmed, changed, removed, or flagged), update the corresponding row in the Section 17 Requirement Log immediately. Do not wait until the end of the session.

---

### Section 5 — Functional Requirements

Walk through one feature area (subsection) at a time.

**For each feature area**, open with a one-line orientation:

> "**[Feature Area Name]** — [N] requirements: [REQ-IDs]. Together these allow [persona(s)] to [capability summary in one plain sentence]."

Then walk through each requirement one at a time.

For each requirement, present it as a real-world capability:

> **[REQ-XXX]** — [Description from BRD]
> In practice: [plain-English framing of what will exist when this is built]
> Does this match what was agreed with the client?

**Framing guide by requirement type:**

| What the requirement covers | How to frame it |
|-----------------------------|----------------|
| User-initiated action | "The [persona] will be able to [action] — [describe the experience and outcome]." |
| System-enforced rule or constraint | "The system will [enforce / prevent / calculate] [condition] — [describe when this triggers and what the user sees or cannot do]." |
| Automated background process | "The system will automatically [action] when [trigger] — [describe what happens without user intervention]." |
| External integration | "[External system] will [send / receive] [what] when [trigger] — [describe what this enables for the business]." |
| Reporting or visibility | "The [persona] will be able to see [what] — [describe what decisions or actions this supports]." |

If the requirement is source-tagged `[Assumed]`, note this before presenting:
> "This requirement is tagged as an assumption — it was not explicitly confirmed by the client. Confirm it is correct, or we can revise or remove it."

**After presenting each requirement, wait for PO response:**

| PO response | Action | Section 17 update |
|-------------|--------|-------------------|
| Confirms | Move to next requirement | Status → `Confirmed` |
| Requests a change | Discuss → agree → apply to BRD immediately → confirm in one line: "Updated: [REQ-XXX] now reads '[new text]'." | Status → `Changed` · Change Made → brief description of what changed |
| Removes the requirement | Remove from Section 5 → note removal: "Removed: [REQ-XXX] — [reason]." | Status → `Removed` · Change Made → reason |
| Requirement too vague to frame | Flag it → ask PO to clarify → apply clarified version inline | Status → `Changed` once clarified; or add to Flags table if unresolved |

After all requirements in a feature area are walked through:

> "[Feature Area Name] complete — [N] confirmed, [N] changed, [N] removed. Moving to [next feature area]."

---

### Section 6 — Business Rules

Present all business rules as a batch:

> "The BRD defines [N] business rule(s). I'll list each — confirm if it is correct, or tell me what needs to change.
> [List each BR-ID and its rule text]"

For any rule challenged: apply the agreed change inline, confirm in one line, update the Section 17 row, continue.

If Section 6 is empty: confirm and skip.

---

### Section 8 — Integration Requirements

Present all integrations as a batch:

> "The BRD lists [N] integration(s). Confirm each is still in scope, or flag what has changed.
> [For each: name, direction, purpose in one line]"

For any integration challenged or removed: apply inline, confirm, update Section 17, continue.

If an integration has no corresponding security requirement in Section 9: add to the Section 17 Flags table — this must be resolved before approval.

If Section 8 is empty or states "None identified": confirm and skip.

---

### Section 9 — Security Requirements

Present all security requirements as a batch:

> "The BRD defines [N] security requirement(s). Confirm each is correct and complete, or flag what needs to change.
> [List each SEC-ID and its requirement text]"

For any requirement challenged: apply inline, confirm, update Section 17, continue.

Flag any integration from Section 8 that still has no corresponding security requirement — add to Section 17 Flags table.

If Section 9 is empty and Section 8 has integrations: flag it. Otherwise confirm and skip.

---

## Step 5 — Verification summary

After all sections are walked through, present the summary to the PO and update Section 17:

```
BRD REVIEW — {PROJECT_CODE}
─────────────────────────────────────────
Document:         {brd-filename}
Date reviewed:    {today}
─────────────────────────────────────────

REVIEWED
- Functional requirements: [N] confirmed, [N] changed, [N] removed
- Business rules:          [N] confirmed, [N] changed
- Integration requirements:[N] confirmed, [N] changed
- Security requirements:   [N] confirmed, [N] changed

FLAGS — UNRESOLVED
[Numbered list of any items in the Section 17 Flags table still Open. If none: "None."]

OPEN QUESTIONS (Section 12)
[List any scope blockers still unresolved. If none: "None."]
─────────────────────────────────────────
```

Update the Section 17 Flags table to reflect current status of all flagged items.

**If FLAGS or unresolved scope blockers remain:**

> "The BRD cannot be approved while these items are unresolved. Work through each one and I will apply the agreed changes. Type 'approved' when all are resolved."

Work through each remaining item. Apply agreed changes inline. Update Section 17 Flags table as each is resolved.

**If all items are resolved:**

> "All requirements reviewed and confirmed. Type 'approved' to approve the BRD and enable epic generation, or raise any final concerns now."

---

## Step 6 — On PO approval

When the PO types 'approved':

1. **Update Section 17 (Review Record):**

   - Set `**Review status:** Complete`
   - Add `**Review completed:** {today's date}`
   - Fill the Review Summary block:

   ```markdown
   ### Review Summary

   **Completed:** {today's date}
   **Functional requirements:** [N] confirmed, [N] changed, [N] removed
   **Business rules:** [N] confirmed, [N] changed
   **Integration requirements:** [N] confirmed, [N] changed
   **Security requirements:** [N] confirmed, [N] changed
   **Flags raised:** [N] — all resolved before approval
   ```

2. **Update Section 14 (Approval):**
   - Set `**Status:** Approved`
   - Set `**Date:** {today's date}`
   - Leave the signatory rows blank — the PO fills those manually

3. **Add a row to the Document Control table:**
   ```
   | {next version} | {today's date} | Ascendra AI | BRD review complete — see Section 17 for full review record. |
   ```
   Increment version: 1.0 → 1.1 if changes were made; leave at 1.0 if no changes.

4. **Report to the user:**

```
BRD REVIEW COMPLETE
─────────────────────────────────────────
Status:     Approved ✅
Document:   {brd-path}
Record:     Section 17 — full review log retained in document
─────────────────────────────────────────
Next step:
Run /gen-epics {PROJECT_CODE} to generate epics from the approved BRD.
```
