# Generate BRD Discovery Playbook

You are generating a BRD discovery playbook for the Ascendra framework. A BRD playbook is an active process document — it runs a requirement discovery session with a client. It converts the facts in the domain knowledge document into structured questions, scope risk flags, and journey walkthroughs. The playbook is used live, alongside the domain knowledge document, during the discovery session.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **Domain knowledge file path** — required. E.g. `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md`
- **PROJECT_CODE** — extract from the path: it is the second segment (`projects/{PROJECT_CODE}/domain/...`)

If no domain knowledge file path is provided, ask:
> "Which domain knowledge file is this playbook for? Provide the path (e.g. `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md`)."

---

## Step 1.5 — Gate check

Check that the domain knowledge file exists at the provided path. If it does not exist, stop:
> "Domain knowledge file not found at `{path}`. Run `/gen-domain-knowledge {PROJECT_CODE}` first to generate it."

---

## Step 2 — Read required files and resolve extension chain

Read the following before continuing:

1. The domain knowledge file from Step 1 — read in full.
2. `projects/TEMPLATE/brds/brd-playbook.template.md` — the template to follow exactly. Every section must appear in the output.
3. `projects/{PROJECT_CODE}/brief.md` — read for domain context and Extension Context.

**Extension chain resolution:** Read the brief's Extension Context.
- If `Project Type: Standalone`: no parent documents to load. Proceed.
- If `Project Type: Extension`: read the parent project's brief at `projects/{EXTENDS_CODE}/brief.md`. Check its Extension Context. Repeat until a Standalone project is reached.
- **Cycle detection:** if any project code appears more than once in the chain, stop immediately: "Circular extension reference detected: {chain}. Fix the Extension Context fields before proceeding."
- **Load order:** root → direct parent → current project. Read domain knowledge documents from `projects/{CODE}/domain/` for each ancestor in this order. If an ancestor is not found in `projects/`, stop: "Project `{CODE}` referenced in the extension chain is not in this workspace."

---

## Step 3 — Confirm inputs before generating

By this point all inputs are loaded:
- Domain knowledge file(s): primary project's document + any ancestor chain documents, read in base-first order
- Brief: loaded for domain context and extension classification
- Template: loaded

If any document is still missing, resolve it before continuing. Do not start generating until everything is loaded.

---

## Step 3.5 — Planning preview

Before writing any files, present the planned discovery coverage and wait for confirmation:

From the domain knowledge document loaded in Step 2, identify:
- Section 10 (Known Variations): these become the discovery topics in Section 4
- Section 8 (Typical Integration Points): these populate Section 7
- Primary lifecycle flows from Section 4: these become journey walkthroughs in Section 6

> "Before I generate the BRD discovery playbook, here is the planned coverage:
>
> **Domain:** {domain name}
> **Discovery topics — from Section 10 Known Variations:**
> [List each known variation as a numbered item — e.g. '1. Approval workflow (single vs multi-level)', '2. Tax calculation model']
>
> **Scope risk flags:** [N] — one per significant complexity or regulatory variation
> **Journey walkthroughs:** [N] — one per primary lifecycle flow
> **Integration confirmation:** [N] integration points from Section 8 of the domain knowledge
>
> Does this coverage match what you expect to cover in the discovery session? Type 'proceed' or add any topics."

Wait for PO response. Do not write any files until confirmed.

---

## Step 4 — Generate the playbook

Follow the structure of `projects/TEMPLATE/brds/brd-playbook.template.md` exactly. Populate every section:

**Output format:** the generated playbook does not include the `[AI Guide — Document Level]` block from the top of the template. Start the document directly with `## Document Control`. Per-section `[AI Guide]` notes in Sections 1 and 2 are generation hints — replace them with the actual generated content. Per-section `[AI Guide]` and `[AI Guide — Section Instructions]` notes in Sections 3–9.5 are runtime session instructions for `/run-brd-discovery` — retain them in the generated output.

- **Section 1:** Playbook scope — 1.1 what it covers, 1.2 domain documents to load (list every file that must be loaded, including conditionals like "load X only if client is confirmed as Y"), 1.3 what it does not cover.

- **Section 2:** Pre-session checklist — the checklist the AI runs before starting. The discovery state file check references `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`.

- **Section 3:** Session opening — frame the session, confirm the basics. Adapted to the specific domain's typical client type.

- **Section 4:** Core discovery questions — one subsection per topic. Topics map directly to **Section 10 (Known Variations)** in the domain knowledge document. Every known variation must have at least one corresponding question here. For each question include: primary question, follow-ups if needed, scope risk trigger reference (→ SRF-NNN), and BRD note (what to do with the answer when drafting the BRD).

- **Section 5:** Scope risk flags — one SRF per significant complexity or regulatory risk. Every SRF referenced in Section 4 must appear here. For each: triggered when, implication, response to client, action.

- **Section 6:** Journey walkthroughs — one journey per primary end-to-end flow in the domain. Journeys must cover the primary lifecycle from Section 4 of the domain knowledge document. For each journey: set-the-scene prompt, standard steps (from domain lifecycle), probe questions, things to note for BRD.

- **Section 7:** Integration confirmation table — one row per typical integration point from **Section 8** of the domain knowledge document. Every integration in the domain doc must appear here.

- **Section 8:** Output confirmation — 8.1 reports and exports, 8.2 notifications, 8.3 access levels, 8.4 non-functional requirements (pulled explicitly from the brief's stated volume/scale expectations and the domain document's Section 9 regulatory baseline — not a fixed checklist like 8.1-8.3). Base 8.1-8.3 on the domain document's typical outputs and personas.

- **Section 9:** Session close — summary, out-of-scope naming, open questions list, next step confirmation.

- **Section 9.5:** Saving state on pause — discovery state saves to `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`.

- **Section 10:** BRD drafting instructions — follow the template exactly. The source-tagging rules (`[Client-Stated]`, `[Domain-Default]`, `[Assumed]`), the section-to-source mapping table, and the quality check must all be present.

- **Section 11:** Playbook verification checklist — run these checks after writing all sections.

**If this playbook covers conditional extension documents** (e.g. country or sector extensions): clearly indicate in Sections 1.2 and 4 which questions apply only when a specific condition is met. Cross-reference the extension knowledge document for those sections.

---

## Step 5 — Run Section 11 verification before writing the file

Work through every checklist item in Section 11:

- [ ] Every Known Variation in the domain document's section 10 is covered by at least one question in section 4.
- [ ] Every Typical Integration Point from section 8 of the domain document appears in the integration confirmation table (section 7).
- [ ] Every significant complexity topic has a corresponding Scope Risk Flag in section 5.
- [ ] Journey walkthroughs cover every primary journey from the domain document's lifecycle.
- [ ] Output Confirmation covers every standard report, notification, and permission model mentioned in the domain doc.

Record any gaps. Fix them before writing the file.

---

## Step 6 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/brds/{domain-slug}-brd-playbook.md`

Use the same slug as the domain knowledge document (e.g. `invoice-payment-core.md` → `invoice-payment-brd-playbook.md`).

After writing the file, report to the user:

```
BRD PLAYBOOK GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Output:      projects/{PROJECT_CODE}/brds/{domain-slug}-brd-playbook.md
Questions:   [N] discovery questions across Section 4 subsections
Risk flags:  [N] scope risk flags in Section 5
Journeys:    [N] journey walkthroughs in Section 6
Verification:[N] Section 11 failures / None
─────────────────────────────────────────
```

> **Recommended next step:** Run `/run-mock-discovery {domain-knowledge-path} {this-playbook-path}` to validate the domain knowledge and playbook together before a real client session. This catches gaps before real client cost.
>
> **Alternatively:** Proceed directly to `/run-brd-discovery` if you are confident in the domain knowledge and playbook.

Remind the user to update Section 12.6 of the domain knowledge document (Mock Discovery Validation) after the first mock or live session run.
