# Generate Domain Discovery Playbook

You are generating a domain discovery playbook for the Ascendra framework. A domain discovery playbook guides a session where the PO (or domain expert) teaches the AI about an unfamiliar domain — before the domain knowledge document is generated. It is the input phase for niche or bespoke domains where AI training knowledge may be insufficient.

This is NOT a requirement discovery playbook. It produces no BRD. It produces domain knowledge inputs.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required (e.g. `HARBORVIEW-INV-001`)
- **Domain name** — optional. If not provided, extract from the brief.

If PROJECT_CODE is missing, ask:
> "Which project is this domain playbook for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

---

## Step 1.5 — Gate check

Check that `projects/{PROJECT_CODE}/brief.md` exists. If not, stop:
> "brief.md not found at `projects/{PROJECT_CODE}/brief.md`. Run `/init-project {PROJECT_CODE}` and complete the brief first."

---

## Step 2 — Read required files

Read the following before doing anything else:

1. `projects/TEMPLATE/domain/domain-playbook.template.md` — the template to follow exactly.
2. `projects/{PROJECT_CODE}/brief.md` — extract the domain name, sector, and problem statement.

---

## Step 3 — Gather remaining inputs

If the domain name could not be extracted from the brief, ask:
> "What is the name of the domain this playbook covers? (e.g. 'Invoice & Payment', 'School Fee Management')"

Do not start generating until the domain name is confirmed.

---

## Step 3.5 — Planning preview

Before writing any files, present the planned playbook structure and wait for confirmation:

From the domain name and brief, identify expected discovery topic areas. If a domain knowledge file or domain discovery state exists for this domain, derive topics from it.

> "Before I generate the domain discovery playbook for **{domain name}**, here is the planned section structure:
>
> **Sections to generate:**
> 1. Playbook scope
> 2. Pre-session checklist
> 3. Session opening
> 4. Core discovery questions — covering: [list expected discovery topic areas, e.g. 'entity types and definitions, lifecycle states, universal business rules, known variations']
> 5. Scope risk flags — for significant complexity or variation topics
> 6. Lifecycle & status model walkthrough
> 7. Universal business rules — confirm and extend
> 8. Standard validations — confirm which apply universally
> 9. Common personas and roles — confirm or correct
> 10. Typical integration points — confirm which apply universally
> 11. Regulatory baseline
> 12. Known variations — what differs between client implementations
> 13. Typical scope boundaries
> 14. Session close
> 15. Playbook verification
>
> Does this structure look right? Any topics to add or remove before I generate the playbook?"

Wait for PO response. Do not write any files until confirmed.

---

## Step 4 — Generate the playbook

Follow the structure of `projects/TEMPLATE/domain/domain-playbook.template.md` exactly. Populate every section.

**Output format:** the generated playbook does not include any `[AI Guide]` blocks or labels from the template. Remove all `[AI Guide]` notes — both the Document Level block at the top and the per-section notes throughout. Replace per-section `[AI Guide]` notes with the actual generated content. Start the document directly with Document Control.

- **Section 1:** Playbook scope — what domain this covers, which documents to load (brief only at this stage — no domain knowledge exists yet), what it does not cover (client requirements, project-specific configuration, technical architecture).
- **Section 2:** Pre-session checklist — brief loaded, no prior domain knowledge to load, check for existing domain discovery state.
- **Section 3:** Session opening — frame the session as a domain education session, not a client session. Confirm domain name and known sub-domains.
- **Section 4:** Domain scope — present a candidate list of modules/capabilities typically found in this domain, drawn from AI training knowledge and the brief's problem statement, client context, and strategic goals; ask the PO to confirm, include, or exclude each before moving to the open follow-up questions. Apply the generalization filter: brief details that are project-specific product decisions (not universal to the domain) get generalized to their generic capability here, with the specific mechanic routed to Section 12 (Known Variations) — never presented as if every implementation works that way.
- **Section 5:** Core entities — present AI knowledge, ask PO to confirm/correct/extend. One sub-section per expected entity.
- **Section 6:** Lifecycle & status model — present the AI's known standard lifecycle (states, transitions, terminal states) for each entity first, ask PO to confirm or correct. One sub-section per entity with meaningful states.
- **Section 7:** Universal business rules — present known rules, ask PO to confirm universality and add any missing.
- **Section 8:** Standard validations — confirm which validations are always enforced and why.
- **Section 9:** Common personas & roles — present standard roles, confirm or correct.
- **Section 10:** Typical integration points — present common integrations, confirm which apply universally vs variably.
- **Section 11:** Regulatory baseline — country-agnostic obligations only.
- **Section 12:** Known variations — what differs between client implementations. These drive the BRD playbook.
- **Section 13:** Typical scope boundaries — what is typically in v1 vs deferred.
- **Section 14:** Session close — summary and readiness check.
- **Section 15:** Playbook verification — run checks after generating.

---

## Step 5 — Run Section 15 verification before writing the file

- [ ] Every section of the domain knowledge template (sections 1–11) is covered by at least one question in this playbook.
- [ ] The playbook clearly distinguishes universal domain knowledge from client-specific requirements.
- [ ] Section 12 (Known Variations) is complete enough to feed into a BRD playbook.

Fix any gaps before writing.

---

## Step 6 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/domain/{domain-slug}-domain-playbook.md`

**Naming convention for the slug:** lowercase, hyphen-separated. Use the same slug that will be used for the domain knowledge document (e.g. `invoice-payment`, `school-fee-management`).

After writing the file, report to the user:

```
DOMAIN PLAYBOOK GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Output:     projects/{PROJECT_CODE}/domain/{domain-slug}-domain-playbook.md
Entities:   [N] entity definitions in Section 5
Variations: [N] known variations in Section 12
─────────────────────────────────────────
```

> **Recommended next step:** Run `/run-domain-discovery {path-to-this-playbook}` to conduct the domain discovery session. This produces a domain discovery state file that `/gen-domain-knowledge` uses to generate an accurate domain knowledge document.
>
> **Alternatively:** If you are confident the AI already has sufficient knowledge of this domain, skip to `/gen-domain-knowledge {PROJECT_CODE}` directly. You will be asked to confirm this choice before the command proceeds.
