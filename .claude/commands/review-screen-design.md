# Review Screen Design

You are running the Screen Design review for an Ascendra client project. This is the mandatory quality gate between `/gen-screen-design` and the Product Owner approving the document. No architecture generation begins until this review is complete and Screen Design is Approved.

The review is a sequential walkthrough of the document's four sections — Portals, Navigation Map, Screen Inventory, and Data Requirements. For each: the AI runs structural checks, surfaces FLAGS, applies fixes, and gets PO confirmation. Screen Inventory gets the closest scrutiny, because every Surface/UI Pattern citation must resolve to a real, verifiable `ascendra-ui` catalog entry — an invented shape here is exactly the failure mode this phase exists to prevent.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments and gate check

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required (e.g. `HARBORVIEW-INV-001`)

If PROJECT_CODE is missing, ask:
> "Which project is this screen design review for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

**Gate check — document must exist and not already be Approved:**

Read `projects/{PROJECT_CODE}/screens/screen-design.md`.
- If it does not exist → stop: > "No screen design found. Run `/gen-screen-design {PROJECT_CODE}` first."
- Read the `**Status:**` field. If `Draft` → proceed. If `Approved` → stop: > "Screen Design is already Approved. To modify it, run `/assess-change` and describe the change needed."

---

## Step 2 — Read required files

1. `projects/{PROJECT_CODE}/screens/screen-design.md` — read in full
2. The approved BRD in `projects/{PROJECT_CODE}/brds/` — source of truth for journeys, roles, and FRs
3. `ascendra-ui/docs/ui-reference.md` Part 2 (composite patterns) — needed to verify every Matched Reference citation is real, not to re-derive them
4. Every mock file in `projects/{PROJECT_CODE}/mocks/` produced by `/gen-screen-design`'s base-mode invocation of `/gen-ui-mocks`

---

## Step 3 — Review Record: initialise or resume

Check for a `## Review Record` section in `screen-design.md`.

---

**Fresh session (no Review Record section):**

Append to `screen-design.md`:

```markdown
## Review Record

**Review started:** {today's date}
**Review status:** In Progress

| Section | Description | Flags | Status |
|---------|-------------|-------|--------|
| 1 | Portals | 0 | Pending |
| 2 | Navigation Map | 0 | Pending |
| 3 | Screen Inventory | 0 | Pending |
| 4 | Data Requirements | 0 | Pending |

## Open Flags

| # | Section | Screen ID | Issue | Tier | Status |
|---|---------|-----------|-------|------|--------|

## Changes Applied

| Section | Change |
|---------|--------|

## Review Summary

*(Filled on completion)*
```

Update the document `**Status:**` header to `Under Review`. Proceed to Step 4 starting from Section 1.

---

**Resumed session (Review Record exists, status `In Progress`):**

Find the first section with status `Pending` or `In Progress`.

> "Resuming screen design review for {PROJECT_CODE}. Completed so far: [list Reviewed sections]. Picking up from Section [N]: [Description]."

Jump to Step 4 at the resume point.

---

**Review already complete (status `Complete`):**

> "The screen design review is already complete. Current document status: `{status}`. Run `/update-status projects/{PROJECT_CODE}/screens/screen-design.md Approved` if it is not yet Approved."

---

## Step 4 — Section-by-section walkthrough

**Fix tiers — apply within each section as FLAGS are surfaced:**

**Tier 1 — Apply immediately, no pause:**
- Typo or label error in any field
- Route/naming inconsistency between Portals and Screen Inventory
- Missing visibility condition on a Nav Map row that is obviously "baseline — always visible"

**Tier 2 — Present and confirm before applying:**
- Missing screen for a BRD journey step
- Surface reassigned (e.g. a screen marked Page that reads as an inline action)
- UI Pattern or Matched Reference corrected to an actually-real catalog entry
- Missing Data Requirements row for an existing screen

**Tier 3 — PO decides before executing:**
- Portal boundary change (merge or split a portal)
- Screen removed as out of scope, or added beyond what the BRD states

**Tier 4 — Halt:**
- Multiple screens with no real Matched Reference (document was generated without doing the pattern-matching work — regenerate via `/gen-screen-design`)
- Multiple BRD journeys with no covering screen (document is structurally incomplete)

---

### Section 1 — Portals

**Frame:** > "This project has [N] portal(s): [list, one line each — who uses it, what it's for]. Does this match how you'd expect users to be grouped into separate apps or sessions?"

Wait for PO confirmation.

**Checks:**
1. **Genuine navigation shells** — no two rows represent the same app/session split only by persona; two personas sharing an app with only visibility differences are one portal, not two.
2. **Delegated access handled** — if BRD Section 6.3 is populated, the delegated session is noted as entering an existing portal, not listed as its own row.

---

### Section 2 — Navigation Map

**Checks:**
1. **Portal reference validity** — every row's Portal matches a real row in Section 1.
2. **Visibility condition present** — every row states a condition, never left blank.
3. **Grouping stated** — every row states flat or grouped, consistent with the portal's actual item count.

---

### Section 3 — Screen Inventory

**Frame:** > "This screen design covers [N] screens across [N] journeys from the BRD: [list journey → screen mapping briefly]. Open the base mocks now alongside this table — do the navigation and screen shapes match what you'd expect a user to see?"

Wait for PO confirmation before presenting FLAGS.

**Checks:**
1. **Journey coverage** — every journey step in BRD Section 4 maps to at least one Screen ID. Flag any uncovered step.
2. **Surface assigned** — every row has a Surface (Page/Dialog/Sheet/Drawer), never blank.
3. **UI Pattern assigned** — every row has a UI Pattern, never blank.
4. **Matched Reference is real** — for each row, verify the cited catalog entry or primitive composition actually exists in `ui-reference.md` Part 1 or Part 2. A citation naming a pattern that isn't in the catalog, or a generic label with no citation, is a Tier 2 flag minimum — Tier 4 if this affects multiple rows.
5. **Surface reasoning holds** — the Surface choice is consistent with its source journey/FR language (e.g. a screen whose journey says "without leaving the page" should not be marked Page).
6. **Tabs justified** — any row proposing Tabs has a source journey/FR genuinely describing 2+ independent sub-views under one entry point. Flag speculative Tabs proposals.
7. **No Command Palette or Toast as a per-screen match** — flag any row that assigned either as a Surface or UI Pattern.
8. **Base mock exists** — every Portal in Section 1 has a corresponding file in `projects/{PROJECT_CODE}/mocks/`.

---

### Section 4 — Data Requirements

**Checks:**
1. **Every screen covered** — every Screen ID in Section 3 has a corresponding row here. Flag any screen missing its data requirements.
2. **Fields trace to the BRD** — fields/actions listed are traceable to a BRD entity, journey step, or business rule — not invented to fill the row.

---

**Present FLAGS (per section):**

```
FLAGS — Section {N}: {Description}
[Numbered list. For each flag: check number, Screen ID if applicable, precise issue, fix tier.]
[If none: "No flags — all checks passed."]
```

Apply Tier 1 fixes immediately. Present Tier 2/3 flags for PO confirmation before applying. Halt on Tier 4 and explain what's needed before the review can continue.

**Update the Review Record** after each section: set its Status to `Reviewed`, its Flags count, and log any unresolved flag in the Open Flags table with status `Open`.

> "Section {N} reviewed ✓. Moving to Section {N+1}: {Description}."

---

## Step 5 — After all sections are reviewed

**Resolve open flags**

Check the Open Flags table. If any are `Open`:

> "The following items were not resolved during the walkthrough: [list]. These must be resolved before Screen Design can be approved."

Work through each with the PO. Update the Open Flags table to `Resolved` as each closes.

**Approval confirmation:**

> "All four sections reviewed. [N] flags raised, all resolved. [N] changes applied.
>
> Ready to approve Screen Design? Type `approve` to set Status to Approved and complete the review, or `not yet` to leave it in Under Review for further changes."

Wait for PO response.

---

**If PO types `approve`:**

Update `screen-design.md`:
- `**Status:** Approved`

Update the Review Record:
- `**Review status:** Complete`
- Add `**Review completed:** {today's date}`
- Fill the Review Summary:

```markdown
## Review Summary

**Completed:** {today's date}
**Screens reviewed:** [N]
**Approved without changes:** [N]
**Approved with changes:** [N] — [one-line summary]
**Flags raised:** [N] — all resolved before approval
```

```
SCREEN DESIGN REVIEW COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Status:         Approved
Document:       projects/{PROJECT_CODE}/screens/screen-design.md
Sections:       4 reviewed
Flags raised:   [N total] — all resolved
Changes:        [N applied]
─────────────────────────────────────────
Next step:
Run /gen-architecture {PROJECT_CODE}. Screen & Navigation Map
(Portals, Navigation Map, Screen Inventory) will be carried
over from this approved document, not re-derived.
```

---

**If PO types `not yet`:**

> "Screen Design remains in Under Review status. Progress is saved in the Review Record — return to this review at any time. Run `/review-screen-design {PROJECT_CODE}` to resume."
