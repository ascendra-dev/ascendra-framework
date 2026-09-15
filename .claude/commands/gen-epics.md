# Generate Epics

You are generating epics for the Ascendra platform. Epics are cohesive delivery units — each one groups BRD requirements that together deliver a complete, user-meaningful capability. Epics are the layer between the BRD and user stories. They are authored in conversation with the Product Owner (you are the AI half of that conversation) after the BRD is approved.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required. E.g. `HARBORVIEW-INV-001`
- **BRD path** — optional. If provided, e.g. `projects/HARBORVIEW-INV-001/brds/brd-core-v1.md`. If not provided, use the most recent `brd-core-v*.md` in `projects/{PROJECT_CODE}/brds/`.

If PROJECT_CODE is missing, ask:
> "Which project are these epics for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Wait for PO response.

Derive the following paths automatically:
- Brief: `projects/{PROJECT_CODE}/brief.md`

---

## Step 1.5 — Gate check: BRD must be Approved

Read the BRD file now. Check the **Status** field in the BRD's Document Control table.

BRD status `Approved` confirms that `/review-brd` has been completed and the PO has signed off on the requirements. If the BRD has not been through `/review-brd`, run that first — it sets the status to Approved on PO confirmation.

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| BRD file exists | `projects/{PROJECT_CODE}/brds/brd-core-v*.md` is present | "BRD file not found at `{path}`. Generate the BRD first using `/gen-brd {PROJECT_CODE}`." |
| BRD is Approved or Locked | Read the **Status** field in the BRD's Document Control table | "BRD at `{path}` has status `{status}`. Run `/review-brd {PROJECT_CODE}` to complete the BRD review — it will set the status to Approved on PO sign-off." |

---

## Step 2 — Read required files and resolve extension context

Read the following before continuing:

1. `projects/TEMPLATE/epics/epic.template.md` — the template every epic must follow. Read Part 1 and Part 2 (Verification) in full.
2. `projects/{PROJECT_CODE}/brief.md` — project context and Extension Context.
3. `projects/{PROJECT_CODE}/brds/brd-core-v*.md` — the approved BRD (already verified in Step 1.5) — read in full if not already loaded.
4. `[domain knowledge file path referenced in the BRD, if any]` — read it if present.

Epics are Problem Domain artifacts (no architecture dependency — see `decisions/FW-022-problem-solution-domain-boundary.md`). Do not read the architecture document even if one exists; nothing in the epic template depends on it.

**Extension Context:** Read the brief's Extension Context.
- If `Project Type: Standalone`: proceed. Generate epics for all layers in the BRD.
- If `Project Type: Extension`: also read `projects/{EXTENDS_CODE}/epics/index.md` — this is what tells you which Core epics already exist in the parent project. Extension epics cover only the new layer (e.g. `Ext:PK`) — do not regenerate any epic already listed in that index. The Layer filter in Step 3 enforces this.

---

## Step 3 — Gather remaining inputs

PROJECT_CODE and BRD are already resolved from Steps 1–2. Check `$ARGUMENTS` for these additional inputs.

**Phase and key decisions:** If any are missing, ask for them all at once:

| Input | What to ask if missing |
|-------|----------------------|
| **Phase** | "Which phase is this? (e.g. Phase 1, Phase 2)" |
| **Key decisions** | "Are there any REQ assignment decisions, scope exclusions, or cross-epic boundary decisions already made that I must preserve? List them, or type 'none' to derive from the BRD." |

**Layer:** If not provided in `$ARGUMENTS`, ask:

> "Which layer are these epics for?
>
> **(a) Core** — applies regardless of country or sector.
> **(b) Ext:PK** — Pakistan-specific only.
> **(c) Ext:School** — school sector only.
> **(d) Other Ext** — specify the extension code."

Wait for PO response.

---

## Step 4 — Plan the epic set before generating files

Before writing any epic file, produce a **planning table** and wait for confirmation:

1. Group BRD Section 5 requirements by feature area. Each feature area is a candidate epic.
2. Apply the Layer filter — only include requirements matching the requested Layer.
3. Identify cross-cutting dependencies (e.g. a foundation/infrastructure epic that all others depend on).
4. Draft a dependency ordering (which epics must be built before which).
5. Check candidate epic size against the rule of thumb in `epic.template.md`'s Document Level AI Guide (roughly 4–10 stories per epic). If a candidate would clearly produce many more, propose a split into two epics at this table — before any file is generated, not after.
6. Present this as a table:

| Epic ID | Proposed Title | REQ IDs included | Dependencies | Notes |
|---------|---------------|-----------------|--------------|-------|

> "Does this epic breakdown look right? Any REQ reassignments, title changes, or scope adjustments before I generate the files?"

Wait for PO response. Do not write any files until confirmed.

---

## Step 5 — Generate each epic file

After confirmation, generate one file per epic following `projects/TEMPLATE/epics/epic.template.md` exactly.

**Output format:** the generated epic file does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Remove the `## Part 1 — Template` label — the epic header fields (`**Epic ID:**`, `**Title:**`, etc.) start directly after `## Document Control`. The verification checklist is included in the output but renamed from `## Part 2 — Verification` to `## Verification`; the `[AI Guide — Verification]` instruction block within it is removed.

For each epic, populate:

**Header fields:** Epic ID (EPIC-{3-digit}), Title, Phase, Layer, Priority (from BRD), Status (Draft)

**Section 1 — Goal:**
1–2 sentences from the user's perspective. "When this epic is done, [persona] can [capability]." No technical deliverables. No list of requirements.

**Section 2 — BRD Requirements Covered:**
Use exact descriptions from the BRD — word for word. No paraphrasing. Every REQ assigned to this epic appears here.

**Section 3 — Scope:**
- 3.1 In Scope: bullet list specific enough that a story title is derivable from each item without reading the BRD. If a bullet cannot produce a story title, it is too vague — break it down.
- 3.2 Out of Scope: NEVER leave empty. For every capability a developer might assume is included, name the EPIC-XXX that covers it, or state "Phase X / deferred". Reference specific epic IDs using the IDs assigned in Step 3.

**Section 4 — Personas Involved:**
Only personas who directly interact with capabilities in this epic (they trigger, receive, or have data affected). Draw from BRD Section 2. No invented personas.

**Section 5 — Dependencies:**
EPIC IDs that must be fully implemented before this epic can begin. Use the dependency order from Step 3.

**Section 6 — Definition of Done:**
Keep the standard 5 items from the template exactly. For the last item (Product Owner walkthrough), write a specific end-to-end walkthrough scenario that exercises all significant In Scope capabilities.

**Section 7 — Stories:**
Leave blank (empty table rows). `/gen-stories` populates this.

**Section 8 — Notes:**
Include every UBR, VAL, BR, and SEC from the domain knowledge document and BRD that `/gen-stories` will need to write complete acceptance criteria for this epic's capabilities. If none apply: omit section.

**Verification:**
Work through all 7 checklist items. Record any failure before writing the file.

---

## Step 6 — Generate the epics index file

After all epic files are written, create `projects/{PROJECT_CODE}/epics/index.md` with:

1. **Epic Registry table** — Epic ID (linked to file), Title, Priority, Status (Draft), REQ Count, Dependencies
2. **REQ Coverage Map** — one row per epic, listing all REQ IDs assigned to it. Every REQ-ID from the BRD's selected Layer must appear exactly once across all epics. No REQ missing. No REQ in two epics.
3. **Dependency Graph notes** — the build order (ordered list of epics by dependency layer, with explanation of why), followed by a **Story-Level Interleaving** check.

For every epic pair connected by a Section 5 dependency, apply this trigger test: can you name one *specific* story in the depended-on epic — not the whole epic — that must land before a specific story in the dependent epic can be built or meaningfully demoed? If yes, and the dependent epic does not need the rest of the depended-on epic's Definition of Done to be met first, the pair's true build order is finer-grained than a simple epic-before-epic dependency. If the dependent epic can simply wait for the other epic's Definition of Done (Section 6) in full, this is an ordinary epic-level dependency — no note is needed.

When the trigger applies, add this subsection directly beneath the build order list:

```
**Story-Level Interleaving:**
- EPIC-XXX ⇄ EPIC-YYY — [one sentence: which specific story-level capability in EPIC-YYY must land before which specific capability in EPIC-XXX can be built or demoed]
```

If no epic pair meets the trigger, write the subsection anyway with "None — every dependency in this set resolves at the epic level" rather than omitting it. `/gen-stories` and `/review-stories` both read this subsection to decide whether a wave spans one epic or an interleaved pair.

After writing the index, update `projects/{PROJECT_CODE}/brief.md` Section 9 (Related Artifacts) — add a row: `| Epic | projects/{PROJECT_CODE}/epics/index.md | Draft |`. If an Epic row already exists from a prior run, update its status instead of adding a duplicate.

---

## Step 7 — Output location

**Epic files:** `projects/{PROJECT_CODE}/epics/EPIC-{NNN}-{slug}.md`

Slug naming: lowercase, hyphen-separated summary of the epic title. Examples:
- "Platform Foundation & Core Infrastructure" → `platform-foundation`
- "Payer Management & Data Import" → `payer-management`

**Index file:** `projects/{PROJECT_CODE}/epics/index.md`

---

## Step 8 — Report to the user

```
EPICS GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Epics generated:  [N]
REQs covered:     [N] of [N] in the selected Layer
REQs missed:      [list by REQ-ID] / None
Verification:     [N] Part 2 failures / None
Review sequence:  [dependency-ordered epic list]
─────────────────────────────────────────
Next step:
Run /review-epics projects/{PROJECT_CODE} before any story generation begins.
```
