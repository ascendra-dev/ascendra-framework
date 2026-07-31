# Generate Domain Knowledge Document

You are generating a domain knowledge document for the Ascendra framework. This document captures universal, reusable knowledge about a business domain — it is not project-specific. It becomes the foundation that all future discovery sessions, playbooks, and BRDs for this domain are built on.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:

- **PROJECT_CODE** — required. The project this domain knowledge belongs to (e.g. `HARBORVIEW-INV-001`).

If PROJECT_CODE is missing, ask:
> "Which project is this domain knowledge for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

---

## Step 1.5 — Gate check: brief must be approved

Read `projects/{PROJECT_CODE}/brief.md`.

If the file does not exist, stop:
> "`projects/{PROJECT_CODE}/brief.md` not found. Run `/init-project {PROJECT_CODE}` then `/run-intake {PROJECT_CODE}` first."

If the brief's **Content Status** field (not **Status** — that's the separate project-lifecycle field) is not `Approved`, stop:
> "The brief for `{PROJECT_CODE}` is not yet approved. Review and approve it first:
> `update-status projects/{PROJECT_CODE}/brief.md Approved`"

Once the brief is confirmed as Approved, proceed. The brief is now loaded — extract: domain name, sector, problem statement, client constraints, and the Extension Context fields (Project Type, Extends, Extension Adds).

---

## Step 2 — Check for domain discovery state

Check whether `projects/{PROJECT_CODE}/domain/domain-discovery-state.md` exists.

**If it exists:** Read it in full. This is the primary input — the PO has already run a domain discovery session. Use the PO's stated answers from the state file to populate the domain knowledge document. Where the state file is silent on a section, supplement with AI knowledge. Skip to Step 3.

**If it does not exist:** Show this warning and wait for confirmation before proceeding:

```
DOMAIN KNOWLEDGE — {PROJECT_CODE}
─────────────────────────────────────────
⚠  Domain discovery was not run.

This document will be generated from AI training knowledge only.

For well-known domains (invoicing, payroll, school fees, HR): this is
sufficient and you can proceed.

For niche, regulated, or unfamiliar domains: the result may be thin or
inaccurate without prior domain discovery.

Recommended: Run /gen-domain-playbook {PROJECT_CODE} first, then
/run-domain-discovery — this ensures the domain knowledge reflects your
specific domain context before it is used in discovery and BRD generation.

Proceed with AI knowledge only? (yes / no)
─────────────────────────────────────────
```

- **yes** → continue to Step 3.
- **no** → stop. Tell the user: "Run `/gen-domain-playbook {PROJECT_CODE}` to generate the domain discovery playbook, then `/run-domain-discovery` to conduct the session. Re-run this command afterwards."

---

## Step 3 — Read required files

Read the following before doing anything else:

1. `projects/TEMPLATE/domain/domain.template.md` — the template structure. Every section must appear in the output. Follow the `[AI Guide]` notes exactly.
2. `projects/{PROJECT_CODE}/brief.md` is already loaded from Step 1.5 — do not re-read.
3. **Extension chain resolution:** Check the brief's Extension Context fields loaded in Step 1.5.
   - If `Project Type: Standalone`: no parent documents to load. Proceed.
   - If `Project Type: Extension`: read the parent project's brief at `projects/{EXTENDS_CODE}/brief.md`. Check its Extension Context. Repeat until a `Standalone` project is reached. This resolves the full ancestor chain.
   - **Cycle detection:** if any project code appears more than once in the chain, stop immediately: "Circular extension reference detected: {chain}. Fix the Extension Context fields before proceeding."
   - **Load order:** root (standalone) → direct parent → this project. Read domain knowledge files from `projects/{CODE}/domain/` for each ancestor in this order. If an ancestor project is not found in `projects/`, stop: "Project `{CODE}` referenced in the extension chain is not in this workspace. Add it or correct the Extension Context in the relevant brief."

(Domain discovery state was already read in Step 2 if it existed — do not re-read it.)

---

## Step 4 — Gather remaining inputs

Check what is now known from the arguments, brief, and discovery state. If any of the following are still missing, ask for them **all at once before doing any work**:

| Input | What to ask if missing |
|-------|----------------------|
| **Domain name** | "What is the name of the domain? (e.g. 'Invoice & Payment', 'School Fee Management', 'HR & Payroll')" |
| **Domain scope** | "In 2–3 sentences, what business problem does this domain address and what type of system does it cover? This becomes Section 1.1." |

Whether this is a standalone or extension document is determined by the brief's Extension Context (resolved in Step 3) — do not ask the PO.

Do not start generating until both inputs above are confirmed.

---

## Step 4.5 — Planning preview

Before writing any files, present a content summary and wait for confirmation:

> "Before I generate the domain knowledge document for **{domain name}**, here is what I will cover:
>
> **Type:** [Standalone / Extension — derived from brief Extension Context]
> **Sections to generate:**
> 1. Domain overview and scope boundary
> 2. Ubiquitous language (domain terminology)
> 3. Core entities and relationships — [list entity names if derivable from discovery state or AI knowledge]
> 4. Standard lifecycle and status model
> 5. Universal business rules
> 6. Standard validations
> 7. Common personas and roles
> 8. Typical integration points
> 9. Regulatory baseline (country-agnostic)
> 10. Known variations — [list the key variation axes if identifiable]
> 11. Typical scope boundaries
> **Source:** [Domain discovery state (primary) / AI knowledge only]
>
> Does this scope look right? Type 'proceed' or raise any concerns before I generate."

Wait for PO response. Do not write any files until confirmed.

---

## Step 5 — Generate the document

Follow the structure of `projects/TEMPLATE/domain/domain.template.md` exactly. Populate every section following the `[AI Guide]` notes.

**Output format:** the generated document starts directly with the Document Control section. Do not include any Part labels or template meta-headings from the template file in the output.

**Priority rule:** Where the domain discovery state file provides an answer, use it. Where it is silent, use AI knowledge. Where the PO's stated answer differs from AI knowledge, the PO's answer takes precedence — note the divergence in the document.

- **Section 1:** Domain overview (1.1 what it covers, 1.2 what it does not cover, 1.3 related documents — leave empty at creation time)
- **Section 2:** Domain terminology — only terms where precision matters or where the domain uses a word differently from common usage. These become the ubiquitous language for this domain.
- **Section 3:** Core entities and relationships — conceptual only, no schema or field types.
- **Section 4:** Standard lifecycle and status model — state machine for every entity with meaningful states, all valid transitions, and terminal states.
- **Section 5:** Universal business rules (UBR-001 format) — rules that apply in every implementation without exception.
- **Section 6:** Standard validations (VAL-001 format) — business-reason validations always enforced.
- **Section 7:** Common personas and roles — starting points, not guaranteed. To be confirmed during requirement discovery.
- **Section 8:** Typical integration points — common but not assumed. Confirm with client during requirement discovery.
- **Section 9:** Regulatory baseline (country-agnostic) — obligations applying regardless of country.
- **Section 10:** Known variations — the things that MUST be asked during requirement discovery. These drive the BRD playbook.
- **Section 11:** Typical scope boundaries — what is usually in a first implementation vs deferred.
- **Section 12:** Document verification — run all checks in sections 12.1, 12.2, 12.3, 12.4, 12.5. Leave 12.6 as "Not yet run — BRD playbook not yet written."

**If this is an extension document:** Sections 5 and 6 only add or narrow rules from the base document — they do not override without explicit justification. State what each rule adds or narrows and why.

---

## Step 6 — Run Section 12 verification before writing the file

Work through every checklist item in Section 12.1 (Internal Consistency) and Section 12.2 (Completeness). Record any failures in Section 12.5 (Open Issues). Do not write the file if there are unresolved consistency failures — fix them first.

For Section 12.3 (Factual Accuracy Log), list every fact that is time-sensitive or regulation-dependent and flag it with "Unverified — requires human review before production use" if you cannot confirm it from an authoritative source.

---

## Step 7 — Write the output file

**Output path:**
- Base document: `projects/{PROJECT_CODE}/domain/{domain-slug}-core.md`
- Extension (country): `projects/{PROJECT_CODE}/domain/{domain-slug}-{country-code}.md`
- Extension (sector): `projects/{PROJECT_CODE}/domain/{domain-slug}-{sector-slug}.md`

**Naming convention for the slug:** lowercase, hyphen-separated, no spaces. Examples: `invoice-payment`, `school-fee-management`, `hr-payroll`.

**After writing the file**, update `projects/{PROJECT_CODE}/brief.md` Section 9 (Related Artifacts) if the file exists — add a row:

| Artifact | Location | Status |
|----------|----------|--------|
| Domain Knowledge | `projects/{PROJECT_CODE}/domain/{filename}.md` | Active |

If `brief.md` does not exist, skip this step silently.

---

## Step 8 — Report to the user

```
DOMAIN KNOWLEDGE GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Output:       projects/{PROJECT_CODE}/domain/{domain-slug}-core.md
Sections:     [N] of 12 completed
Verification: [N] issues in Section 12.5 / None
Source:       Domain discovery state / AI knowledge only
─────────────────────────────────────────
```

> **Recommended next step:** Run `/gen-brd-playbook {output-path}` to generate the BRD discovery playbook for this domain.
>
> **Before that, optionally:** Run `/run-mock-discovery {output-path} {brd-playbook-path}` after generating the BRD playbook to validate domain knowledge and playbook together before a real client session. This is strongly recommended for niche or complex domains.
