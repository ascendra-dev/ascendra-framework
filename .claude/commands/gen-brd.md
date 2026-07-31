# Generate Business Requirements Document

You are generating a Business Requirements Document (BRD) for the Ascendra platform. The BRD captures what a specific client's system must do — derived from a completed discovery session and grounded in the relevant domain knowledge. It is the approved contract between Product and Engineering before any epics or stories are written.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required. E.g. `HARBORVIEW-INV-001`
- **Playbook file path** — required. E.g. `projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md`

If either is missing, ask for them all at once:
> "To generate the BRD I need:
> 1. **Project code** — e.g. `HARBORVIEW-INV-001`
> 2. **Playbook file** — path to the BRD discovery playbook used in the session (e.g. `projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md`)"

Derive the following paths automatically:
- Brief: `projects/{PROJECT_CODE}/brief.md`
- Discovery state: `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`

---

## Step 1.5 — Gate check

Check all required artifacts exist before proceeding.

| Check | Path | Failure message |
|-------|------|----------------|
| Brief exists | `projects/{PROJECT_CODE}/brief.md` | "brief.md not found. Run `/init-project {PROJECT_CODE}` and complete the brief first." |
| Playbook file exists | Path from arguments | "Playbook not found at `{path}`. Run `/gen-brd-playbook {domain-knowledge-path}` to generate it." |
| Discovery state exists | `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` | "No discovery state found. Run `/run-brd-discovery {PROJECT_CODE} {domain-path} {playbook-path}` to conduct the discovery session first." |

Read the discovery state file. If Section 3 (Playbook Progress) has no topics marked Covered, stop:
> "The discovery state exists but no session data has been recorded yet. Run `/run-brd-discovery {PROJECT_CODE} {domain-path} {playbook-path}` to conduct the discovery session."

---

## Step 2 — Read required files and resolve extension chain

Read the following before continuing:

1. `projects/TEMPLATE/brds/brd.template.md` — the template to follow exactly. Every section must appear in the output.
2. `projects/{PROJECT_CODE}/brief.md` — project context and Extension Context.
3. The playbook file from arguments — read its Section 1.2 (Domain Documents to Load), then read every document listed there in full.
4. `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` — already verified in Step 1.5; read it in full now.

**Extension chain resolution:** Read the brief's Extension Context.
- If `Project Type: Standalone`: no parent BRD to load. Proceed.
- If `Project Type: Extension`: find the most recent approved BRD in `projects/{EXTENDS_CODE}/brds/` and read it as baseline. The current BRD captures only what is new or different from the parent — do not restate requirements already covered in the parent BRD.
- **Cycle detection:** if any project code appears more than once in the chain, stop: "Circular extension reference detected: {chain}. Fix the Extension Context fields before proceeding."
- If the parent project is not in `projects/`, stop: "Parent project `{CODE}` referenced in Extension Context is not in this workspace."

---

## Step 3 — Confirm inputs and determine version

All required documents are now loaded. Before generating:

**Project name:** read from `projects/{PROJECT_CODE}/brief.md` header field `Project Name`. Do not ask the PO.

**BRD version:** check whether `projects/{PROJECT_CODE}/brds/brd-core-v1.md` already exists.
- **Does not exist:** this is version 1. Output file will be `projects/{PROJECT_CODE}/brds/brd-core-v1.md`.
- **Exists:** this is a revision. Read the existing file's Document Control table to find the current version number. Increment the minor version (e.g. v1 → v2). Output file will be `projects/{PROJECT_CODE}/brds/brd-core-v{N}.md`.

---

## Step 4 — Generate the BRD

Follow the structure of `projects/TEMPLATE/brds/brd.template.md` exactly. Populate every section:

**Output format:** the generated BRD does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Start the document directly with `## Document Control`. All section content — checklists, tables, numbered requirements — is included; only the `[AI Guide]` instruction blocks are stripped.

**Section 1 — Project Overview:**
- 1.1 Project name — use the exact name provided
- 1.2 Problem statement — 2–4 sentences from the client's perspective: the problem today, the cost/consequence, and what success looks like
- 1.3 Business goals — specific, measurable outcomes the client expects
- 1.4 Scope — In Scope (feature-area level only, not screens or fields) and Out of Scope (explicitly named exclusions, including anything deferred)
- 1.5 Assumptions and constraints — only what was stated or confirmed

**Section 2 — Stakeholders & Personas:**
Draw from the playbook's Section 3 basics (confirmed org details) and journey walkthrough notes. One row per real persona with distinct goals.

**Section 3 — User Roles (ROL-001 format):**
Draw from playbook Section 8.3 (Access Levels). One subsection per role. Each role must have Can / Cannot / Data visibility defined.

**Section 4 — User Journeys:**
Draw from playbook Section 6 (Journey Walkthroughs) as recorded during discovery. One subsection per primary persona and their end-to-end journey. Step-level only — no screens, no API calls.

**Section 5 — Functional Requirements:**
Organise by feature area. For each requirement:
- **ID:** REQ-001 format (sequential across the whole document)
- **Description:** one specific sentence — what the system must do
- **Priority:** Must Have / Should Have / Nice to Have
- **Source:** `[Client-Stated]` / `[Domain-Default]` / `[Assumed]`
- **Layer:** `Core` / `Ext:PK` / `Ext:School` / `Ext:[code]`

Apply the Layer membership test: *"Would a UK retailer using this system need this requirement?"* If yes → Core. If only because the client is in a specific country → that country's Ext code. If only because of the client's sector → that sector's Ext code.

Do NOT restate domain defaults unless the client overrode them or they drive a specific implementation decision.

**Section 6 — Business Rules (BR-001 format):**
Client-specific rules only. Non-obvious. Decision-driving. No restating of domain knowledge document UBRs.

**Section 7 — Data Requirements:**
Conceptual only — no schema, no field types. Include entity if client has specific attributes or relationships that differ from domain standard.

**Section 8 — Integration Requirements:**
Draw from playbook Section 7 (Integration Confirmation). Every confirmed integration. If none confirmed: "None identified — to be confirmed during technical design."

**Section 9 — Security Requirements (SEC-001 format):**
Cover authentication, authorisation, data protection, audit logging, and compliance. Control-level only — no implementation detail.

**Section 10 — Non-Functional Requirements:**
Only what was explicitly stated by the client. If nothing: "None stated — to be defined in technical design."

**Section 11 — Dependencies:**
Everything that blocks delivery that is owned by someone outside the delivery team. Every confirmed integration with an onboarding lead time is a dependency.

**Section 12 — Open Questions:**
Every unresolved item from playbook Section 9 (Session Close). Every `[Assumed]` tag you applied is a candidate for an open question — review each one.

**Section 15 — Parking Lot:**
Leave as "None identified" at initial generation. Parking Lot entries hold ideas with no domain-model backing and no discovery behind them yet — they typically surface later, during epic review, story writing, or delivery, not during discovery-driven BRD authoring. Do not invent entries here; do not confuse this with Section 5.21 (Future Capabilities), which is for deferred items the discovery session did cover.

---

## Step 5 — Run Section 13 verification before writing the file

Work through every check in Sections 13.1, 13.2, 13.3, and 13.4 of the BRD template:

**13.1 Content Integrity** — required fields, no restated domain defaults, no implementation detail, every deferred Out of Scope item has a REQ-ID (permanent exclusions do not need one)
**13.2 Cross-Section Consistency** — personas in journeys, journeys backed by requirements, integrations have security requirements, roles referenced in requirements
**13.3 Downstream Readiness** — requirements are specific enough for story writing, no deferred-definition language, business rules referenced to requirements, NFRs are measurable
**13.4 Approval Readiness** — open questions have owners, scope blockers are flagged, dependencies have owners, version matches

Record every failure in Section 13.5 (Verification Issues). Resolve all failures before writing — if a failure requires a human decision, record it as a scope blocker in Section 12.

---

## Step 6 — Pre-write summary and confirmation

Before writing any file, tell the user exactly what you are about to generate and wait for their confirmation.

Count from the BRD content you have drafted:
- Functional requirements: count all REQ-XXX IDs in Section 5
- Non-functional requirements: count entries in Section 10
- Open questions: count items in Section 12
- `[Assumed]` tags: count all occurrences (each is a delivery risk)
- Sections with placeholder or TBD content: list any, or note "None"

Output in this format:

> **Ready to write BRD**
>
> **File:** `projects/{PROJECT_CODE}/brds/brd-core-v1.md`
> **Functional requirements:** {N} (across {N} feature areas)
> **Non-functional requirements:** {N}
> **Open questions:** {N}
> **[Assumed] tags:** {N}
> **Sections with TBD content:** {list, or 'None'}
>
> All Section 13 verification checks passed.
>
> **Proceed? (yes / no)**

Wait for the user's response:
- `yes` → continue to Step 7
- `no` → ask what they want to change before proceeding; loop back once clarified

---

## Step 7 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/brds/brd-core-v1.md`

If this is a revision, increment the version in Document Control and in the final section.

**After writing the file**, tell the user:
1. The full path where the file was written
2. Total number of functional requirements written (count of REQ-XXX IDs)
3. Number of open questions remaining
4. Number of `[Assumed]` tags used (these are risks)
5. Whether any Section 13 checks failed and what they are
6. That this BRD must be reviewed and approved by the Product Owner before epic generation begins
7. That the next step after approval is `/gen-epics {path-to-this-brd}`
