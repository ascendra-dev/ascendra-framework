# Run Mock Discovery Session

You are running a mock discovery session to validate that the domain knowledge document and BRD discovery playbook work correctly together. You simulate both roles: the AI-as-facilitator using the playbook, and a simulated client using the domain knowledge to model typical client behaviour. At the end, you evaluate the session against five structured checks and update Section 12.6 of the domain knowledge document.

This is a quality gate — run it before the first real client session to catch gaps while correction is cheap.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **Domain knowledge path** — required (e.g. `projects/{PROJECT_CODE}/domain/invoice-payment-core.md`)
- **BRD playbook path** — required (e.g. `projects/{PROJECT_CODE}/brds/invoice-payment-brd-playbook.md`)
- **PROJECT_CODE** — derived from the domain knowledge path

If either path is missing, ask for them both at once:
> "To run a mock discovery session I need:
> 1. **Domain knowledge file path** — e.g. `projects/{PROJECT_CODE}/domain/invoice-payment-core.md`
> 2. **BRD playbook file path** — e.g. `projects/{PROJECT_CODE}/brds/invoice-payment-brd-playbook.md`"

---

## Step 2 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Domain knowledge file exists | Path from arguments must resolve to an existing file | "Domain knowledge file not found at `{path}`. Run `/gen-domain-knowledge` first." |
| BRD playbook file exists | Path from arguments must resolve to an existing file | "BRD playbook not found at `{path}`. Run `/gen-brd-playbook` first." |

---

## Step 3 — Read required files

Read both files in full before beginning:

1. The domain knowledge document — you will use this to model the simulated client's knowledge and needs.
2. The BRD playbook — you will follow this as the facilitator.

---

## Step 4 — Explain what you are about to do

Tell the PO:

> "Running mock discovery session for **{domain name}** ({PROJECT_CODE}).
>
> I will simulate both sides of a discovery session:
> - **As facilitator:** I follow the BRD playbook exactly — asking questions, handling scope risks, running journey walkthroughs.
> - **As simulated client:** I model a typical client in this domain using the domain knowledge document — answering questions as a real client would.
>
> At the end I will evaluate five specific checks and update Section 12.6 of the domain knowledge document.
>
> This session is internal — you do not need to answer anything. I will run it and report findings."

---

## Step 5 — Run the mock session

Conduct the simulated discovery session internally, working through the BRD playbook:

**Playbook Section 3 (Session Opening):** Simulate a client confirming their organisation, sector, size, and trigger for the project. Use the domain knowledge document's Section 7 (Personas) and Section 1 (Overview) to model realistic client responses.

**Playbook Section 4 (Core Discovery Questions):** For each question, generate a realistic client answer consistent with the domain knowledge document. Vary the answers to test edge cases — do not always choose the most common option.
- Note every question where you needed to invent information not covered in the domain knowledge document — these are gaps.
- Note every question that asked about something already answered in Sections 3–6 of the domain knowledge document — these are AI Guide failures.

**Playbook Section 5 (Scope Risk Flags):** Trigger at least one scope risk flag during the simulation. Evaluate whether the prescribed response is clear and sufficient.

**Playbook Section 6 (Journey Walkthroughs):** Walk through each journey as a client would narrate it. Use Section 4 (Standard Lifecycle) of the domain knowledge document to model the client's workflow.
- Note every walkthrough where the standard steps were unclear or incomplete.

**Playbook Section 7 (Integration Confirmation):** Confirm or deny each integration using the domain knowledge document's Section 8.

**Playbook Section 8 (Output Confirmation):** Confirm outputs using the domain knowledge document's personas and lifecycle.

---

## Step 6 — Evaluate against the five checks

After completing the mock session, answer each check honestly:

**Check 1:** Did you (as facilitator) ask questions about topics already covered in Sections 3–6 of the domain knowledge document?
- If YES → the AI Guide instructions are unclear, or the content is ambiguous. Identify which sections.

**Check 2:** Did the playbook miss any questions it should have asked, based on Section 10 (Known Variations) of the domain knowledge document?
- If YES → Section 10 has gaps, or the playbook does not cover all variations. Identify which variations were missed.

**Check 3:** Would the resulting BRD (if drafted from this session) restate domain defaults that should have been assumed from the domain knowledge document?
- If YES → the AI Guide at the document level is not being followed correctly. Identify which defaults were being restated.

**Check 4:** Would the resulting BRD miss requirements the client would obviously need?
- If YES → the playbook is incomplete in specific areas. Identify what was missed.

**Check 5:** Were any facts generated in the mock session incorrect (wrong lifecycle state, wrong rule, wrong entity attribute)?
- If YES → factual accuracy issue in the domain knowledge document. Identify which facts.

---

## Step 7 — Update Section 12.6 of the domain knowledge document

Update `projects/{PROJECT_CODE}/domain/{domain-slug}-core.md` Section 12.6 with the results:

Replace the placeholder text with:
- The date the mock session was run
- The result of each of the five checks (passed / failed — with detail if failed)
- A list of all failures recorded in Section 12.5 (Open Issues) — add new rows for each failure found

**Mock discovery status:** Set to "Run — [date]. [N] checks passed, [N] failed."

---

## Step 8 — Report to the PO

Tell the PO:

```
MOCK DISCOVERY COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Domain: {domain name}

Results:
  Check 1 — Redundant questions:    [Passed / Failed]
  Check 2 — Missing variations:     [Passed / Failed]
  Check 3 — Domain default restate: [Passed / Failed]
  Check 4 — Missing requirements:   [Passed / Failed]
  Check 5 — Factual accuracy:       [Passed / Failed]

Issues found: {N}
─────────────────────────────────────────
```

If failures were found:
> "Section 12.5 (Open Issues) in the domain knowledge document has been updated with {N} new items. Review and resolve these before running a real client discovery session."
>
> "**Recommended:** Address the open issues in the domain knowledge document or BRD playbook, then re-run `/run-mock-discovery` to confirm they are resolved."

If all checks passed:
> "All five checks passed. The domain knowledge document and BRD playbook are consistent.
>
> **Recommended next step:** Run `/run-brd-discovery {PROJECT_CODE} {domain-knowledge-path} {brd-playbook-path}` to begin the real client discovery session."
