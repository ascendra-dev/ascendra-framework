# Review Epics

You are running the epic review for an Ascendra project. This review happens after all epics for a project are generated and before story generation begins. No epic proceeds to story generation without passing this review.

The review is a sequential walkthrough — one epic at a time. For each epic you run structural checks, present the capability in plain PO language so the Product Owner can confirm what will be built, then surface FLAGS and targeted questions. Changes are applied immediately on agreement.

Progress is recorded in `projects/{PROJECT_CODE}/epics/review-record.md` — this is the single source of truth for review state and the permanent change record. It enables the review to be stopped and resumed across sessions without losing progress.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Locate project and run gate checks

Resolve the project path from `$ARGUMENTS`. If not provided, ask:
> "Which project are these epics being reviewed for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Wait for PO response.

**Gate check — BRD must be Approved:**

Read the most recent `brd-core-v*.md` in `projects/{PROJECT_CODE}/brds/`. Check the Status field.

- If `Approved` or `Locked` → proceed
- If not → stop:
  > "The BRD is not yet Approved. Run `/review-brd {PROJECT_CODE}` and get PO approval before reviewing epics."

**Epics must exist:**

Read `projects/{PROJECT_CODE}/epics/index.md`. Confirm at least one epic is listed.

- If no epics → stop:
  > "No epics found. Generate epics first using `/gen-epics {PROJECT_CODE}`."

---

## Step 2 — Read shared context

Read these files in full before starting the walkthrough. They are loaded once and used across all epic reviews in this session.

1. `projects/{PROJECT_CODE}/brds/brd-core-v*.md` — the BRD (already read in Step 1 — load any sections not yet in context)
2. `projects/{PROJECT_CODE}/epics/index.md` — REQ coverage map and dependency table
3. `[domain knowledge file path(s) referenced in the BRD, or in projects/{PROJECT_CODE}/domain/]`

Do **not** load individual epic files here — each epic is loaded only when its turn comes in the walkthrough.

---

## Step 3 — Gather inputs

Check `$ARGUMENTS`. If any of the following are missing, ask for them **all at once** before doing any work.

| Input | What to ask if missing |
|-------|----------------------|
| **Layer** | "Which layer is being reviewed? (Core / Ext:XX / other)" |
| **Key decisions** | "Were any REQ assignment decisions, scope exclusions, or cross-epic boundary decisions made during epic generation that I must preserve? List them, or type 'none'. Common types: a REQ moved from the obvious epic to a less obvious one, a scope item explicitly excluded by PO, a cross-epic boundary call." |

The review sequence is derived from the dependency order in `epics/index.md` — epics with no unmet dependencies first. If you need a different order, state it now. Otherwise the review begins immediately after this step.

---

## Step 4 — Review Record: initialise or resume

Check for `projects/{PROJECT_CODE}/epics/review-record.md`.

---

**If review-record.md does NOT exist — fresh session:**

Create `projects/{PROJECT_CODE}/epics/review-record.md` with the structure below. Pre-populate the Epic Log with every epic from the index in dependency order, all set to `Pending`.

```markdown
# Epic Review Record — {PROJECT_CODE}

**Review started:** {today's date}
**Review status:** In Progress
**Layer:** {layer}
**Key decisions preserved:** {key decisions from Step 3, or "None"}

## Epic Log

| Epic | Title | Status | Changes Made | Flags Raised |
|------|-------|--------|-------------|-------------|
| EPIC-001 | [Title] | Pending | — | — |
| EPIC-002 | [Title] | Pending | — | — |

## Flags

| # | Epic | Issue | Status |
|---|------|-------|--------|

## Review Summary

*(Filled on completion)*
```

Proceed to Step 5 starting from the first epic.

---

**If review-record.md EXISTS with status `In Progress` — resumed session:**

Read the Epic Log. Find the **first row with status `Pending`** — that is the resume point.

Confirm to the PO:
> "Resuming epic review. Progress so far: [N] reviewed, [N] pending.
> Resuming from [EPIC-ID]: [Title]."

Jump to Step 5 at the resume epic.

---

**If review-record.md EXISTS with status `Complete`:**

> "All epics have already been reviewed per `epics/review-record.md`. Epic statuses should be set to Approved — run `/update-status` if any are still at Draft."

---

## Step 5 — Per-epic walkthrough

For each epic in sequence (starting from the resume point if resuming):

---

### A — Load this epic's context

Read this epic's file only. Do not load other epic files. The shared context (BRD, index, domain knowledge) is already in context from Step 2.

---

### B — Run structural checks

Run all 13 checks against this epic using the shared context. Apply key decisions from Step 3 — do not flag items that were explicitly decided during generation.

1. **REQ Coverage** — Section 2 contains exactly the REQs assigned to this epic in the index coverage map — no more, no fewer. Missing REQ = gap. Extra REQ = misassignment.
2. **Verbatim Accuracy** — Each REQ description in Section 2 matches the BRD word for word. No paraphrasing, truncation, or rewording.
3. **Scope Completeness** — Every REQ in Section 2 has at least one corresponding In Scope bullet in Section 3.1. No REQ without In Scope coverage.
4. **Story Granularity** — Each In Scope bullet is specific enough that a story title can be derived without reading the BRD. Flag bullets too vague (no story title derivable) or too broad (one bullet covers multiple independent deliverables that should each be a separate story).
5. **Out of Scope Integrity** — Every item in Section 3.2 names a specific EPIC-XXX, Phase X, or architectural exclusion. No exclusion without a stated home.
6. **Persona Accuracy** — All personas in Section 4 appear in BRD Section 2. No invented personas. No persona missing who directly triggers, receives from, or has data affected by a capability in this epic.
7. **Dependency Correctness** — All EPIC-IDs in Section 5 reference epics that exist in the index. Each dependency reason is accurate. No missing dependencies — this epic does not use data or infrastructure from another epic that is unlisted.
8. **Domain Rule Coverage** — Section 8 Notes references all UBRs, VALs, BRs, and SEC requirements from the domain knowledge document that `/gen-stories` will need to write complete acceptance criteria for this epic's capabilities. Flag any applicable domain rule absent from Notes.
9. **Definition of Done** — The PO walkthrough scenario in Section 6 exercises all significant In Scope capabilities. No major In Scope capability is untested by the scenario.
10. **Template Compliance** — Section 7 Stories is blank. Verification checklist present. No implementation detail in Sections 1–6 (no field names, API routes, table names, technology choices, UI layout decisions). Layer field matches the layer under review. Priority matches BRD priority for each REQ in Section 2.
11. **Layer Boundary** — No capability belonging to a different layer appears as an In Scope item. Any interface defined in this layer that an extension will implement is identified as interface-only, with the extension epic named in Section 3.2.
12. **Epic Size** — This epic's REQ count (Section 2) is not a clear outlier against the REQ Count column for every other epic in `epics/index.md`'s Epic Registry table, and its In Scope bullet count (Section 3.1) is proportionate. Rule of thumb (`epic.template.md`'s Document Level AI Guide): an epic typically decomposes into roughly 4–10 stories. An epic clearly beyond that relative to its siblings should have been split at `/gen-epics` Step 4 — flag it for a split now.
13. **No Gold-Plating** — Every In Scope item (3.1) and every Section 8 Notes entry traces back to a REQ actually listed in Section 2 or an explicitly-approved domain rule (UBR/VAL/BR/SEC) from the domain knowledge document. No invented scope beyond what the BRD or an approved domain rule calls for.

---

### C — Capability walkthrough

Before presenting FLAGS, describe the epic as a user-facing outcome in plain PO language:

> "**[EPIC-ID]: [Title]**
> When this epic is complete: [describe in 2–4 sentences what the persona(s) will be able to do — what they can see, what actions they can take, what the system enforces on their behalf. Write from the user's perspective, not as a feature list. Mention the personas and the business value.]
> This epic covers [N] BRD requirements and will produce approximately [N] stories.
> Does this match your expectation for this feature area?"

Wait for PO confirmation before presenting FLAGS. If the PO redirects the capability description, that is a scope or coverage issue — surface it as a FLAG.

---

### D — Present FLAGS and apply fixes

After the capability is confirmed, present FLAGS:

```
FLAGS — EPIC-{NNN}: {Title}
[Numbered list. For each flag: check number, section, precise issue, and fix tier.]
[If none: "No flags — all checks passed."]
```

**Fix tiers:**

**Tier 1 — Apply immediately, no pause:**
- Wording correction or typo in any text field
- Cross-reference error (wrong EPIC-ID, wrong REQ-ID in Section 5)
- Missing domain rule in Section 8 Notes that is clearly applicable
- Priority field value wrong but derivable from the BRD without ambiguity

**Tier 2 — Present and confirm before applying:**
- In Scope bullet rewrite (scope wording change or granularity correction)
- Persona list addition or removal
- Section 8 Notes addition for a domain rule identified during the check
- Section 6 Definition of Done walkthrough revision to cover a missing capability
- In Scope bullet or Section 8 Notes entry removed because it does not trace to a REQ in Section 2 or an approved domain rule (gold-plating)

**Tier 3 — PO decides before executing:**
- REQ reassignment between epics (moves coverage from one epic to another)
- Scope boundary change (capability moved in or out of scope for this epic)
- Layer boundary correction that changes what this epic is responsible for

**Tier 4 — Halt:**
- Multiple REQs missing from coverage (epic is structurally incomplete)
- Scope fundamentally misaligned with BRD (epic covers the wrong capability area)
- Layer boundary violated across multiple In Scope items (epic needs to be regenerated)
- Epic Size is a clear outlier against the rest of the epic set (rule of thumb: ~4–10 stories) — epic needs to be split into two and regenerated

---

### E — Targeted PO questions

After FLAGS are addressed, ask **1–2 questions** that structural analysis cannot answer. Make them specific to the risks in this particular epic — do not use a generic checklist. Pick from examples like these, or write a more specific one:

- "Is there any capability you discussed with the client in this area that is not covered by the In Scope list?"
- "The Definition of Done walkthrough — does it exercise the full flow you'd demo to the client, or is something missing?"
- "The Out of Scope list for this epic — does it explicitly name the things a developer might assume are in scope but are not?"
- "Is any In Scope item here something you'd expect a different epic to own?"

Wait for PO response. Apply any changes raised. Confirm before moving on.

---

### F — Update review record

After the PO confirms the epic:

Update `epics/review-record.md`:
- Set this epic's Status → `Reviewed`
- Changes Made → brief note per change (or `—` if none)
- Flags Raised → count (or `0`)

For any FLAG that was not fully resolved, add a row to the Flags table with status `Open`.

Confirm to the PO:
> "EPIC-{NNN} reviewed ✓. Moving to [next EPIC-ID]: [Title]."

---

## Step 6 — After all epics are reviewed

**Resolve open flags**

Check the Flags table in `epics/review-record.md`. If any flags are `Open`:

> "The following items were not resolved during the walkthrough:
> [List each open flag with its epic and issue]
> These must be resolved before epics can be approved."

Work through each with the PO. Apply agreed changes to the relevant epic files. Update the Flags table to `Resolved` as each is closed.

**Update epic statuses**

Once all flags are resolved:

1. Update the **Status** field in each epic file from `Draft` to `Approved`
2. Update the **Status column** in `epics/index.md` for all approved epics

**Complete the review record**

Update `epics/review-record.md`:
- Set `**Review status:** Complete`
- Add `**Review completed:** {today's date}`
- Fill the Review Summary:

```markdown
## Review Summary

**Completed:** {today's date}
**Epics reviewed:** [N]
**Approved without changes:** [N]
**Approved with changes:** [N] ([list which epics and one-line summary of what changed])
**Flags raised:** [N] — all resolved before approval
```

**Report to the user**

```
EPIC REVIEW COMPLETE
─────────────────────────────────────────
Epics reviewed:        [N]
Approved clean:        [N]
Changes applied:       [N]
Record:                projects/{PROJECT_CODE}/epics/review-record.md
─────────────────────────────────────────
All epics are now Approved. Next step:
Run /gen-architecture {PROJECT_CODE}
```
