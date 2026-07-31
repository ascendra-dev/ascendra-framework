# Run Domain Discovery Session

You are running a domain discovery session for the Ascendra framework. In this session, the PO (or domain expert) teaches you about a specific business domain. You ask structured questions from the domain playbook, record the PO's answers, and write a domain discovery state file. `/gen-domain-knowledge` reads this state file to generate an accurate domain knowledge document.

This is NOT a requirement discovery session with a client. There is no BRD output from this session. The output is domain knowledge inputs only.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **Domain playbook path** — required (e.g. `projects/HARBORVIEW-INV-001/domain/invoice-payment-domain-playbook.md`)
- **PROJECT_CODE** — derived from the playbook path

If the playbook path is missing, ask:
> "Which domain playbook should I use for this session? Provide the path (e.g. `projects/{PROJECT_CODE}/domain/{slug}-domain-playbook.md`). Run `/gen-domain-playbook` first if you haven't generated it yet."

---

## Step 2 — Gate check

Before reading anything else:

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Domain playbook exists | Path from arguments must resolve to an existing file | "Domain playbook not found at `{path}`. Run `/gen-domain-playbook {PROJECT_CODE}` to generate it first." |
| Brief exists | `projects/{PROJECT_CODE}/brief.md` must exist | "Brief not found. Run `/init-project` and `/run-intake` first." |

---

## Step 3 — Read required files

Read every file in full before beginning the session:

1. The domain playbook file — your session guide: questions, entity definitions to confirm, lifecycle walkthroughs.
2. `projects/{PROJECT_CODE}/brief.md` — domain name, sector, problem context.
3. `projects/{PROJECT_CODE}/domain/domain-discovery-state.md` — check if a prior session exists. If Section 4 has entries, this is a **resumed session** — pick up from where it ended.

---

## Step 4 — Pre-session checklist

Before opening the session, confirm:

- [ ] Domain playbook loaded ✓
- [ ] Brief loaded — domain name confirmed: **{domain name}**
- [ ] Session type: **Fresh** (no prior state) or **Resumed** (prior state exists — state which sections were covered)

If resumed, tell the PO:
> "Resuming domain discovery for {domain name}. The following sections were already covered: [list]. Picking up from [next section]. Ready?"

---

## Step 5 — Open the session

Tell the PO:

> "Starting domain discovery session for **{domain name}**.
>
> In this session I'll ask you structured questions about how this domain works universally — not about any specific client's requirements. Your answers will form the domain knowledge document that all future projects in this domain will use.
>
> Where I already have knowledge about this domain, I'll confirm it with you. Where my knowledge might be limited, I'll ask you to teach me. Your answers always take precedence over my assumptions.
>
> Let's begin."

Then follow **Playbook Section 3 (Session Opening)** exactly.

---

## Step 6 — Conduct the discovery session

Work through the playbook section by section:

**Section 4 — Domain Scope:** Ask the scope questions. Record the domain definition and exclusions verbatim.

**Section 5 — Core Entities:** Present AI knowledge of standard entities. Ask the PO to confirm, correct, or extend. For each entity: record name, definition, attributes, relationships, notes.

**Section 6 — Lifecycle & Status Model:** Walk through each entity's lifecycle. Record states, transitions, triggers, terminal states, and transition rules.

**Section 7 — Universal Business Rules:** Present known rules. Confirm universality. Record PO additions and corrections.

**Section 8 — Standard Validations:** Confirm which validations always apply and why.

**Section 9 — Common Personas & Roles:** Present standard roles. Record PO confirmations and corrections.

**Section 10 — Typical Integration Points:** Confirm common integrations and whether they are always needed or variable.

**Section 11 — Regulatory Baseline:** Country-agnostic obligations only. Record what applies universally.

**Section 12 — Known Variations:** Record what differs between client implementations. These directly feed the BRD playbook.

**Section 13 — Scope Boundaries:** Record what is typically in v1 vs deferred and why.

**Rules during the session:**
- Ask one section at a time — do not jump ahead
- When the PO's answer differs from AI knowledge, always record the PO's answer and note the correction explicitly. This is the most important data to capture.
- If the PO is uncertain, record the uncertainty in Open Issues — do not assume
- Do not ask about client-specific requirements — redirect to the domain level if the PO veers that way

---

## Step 7 — Save state progressively

After completing each major section (5, 6, 7, 12), save accumulated answers to `projects/{PROJECT_CODE}/domain/domain-discovery-state.md`. Do not wait until the end of the session.

Update:
- **Section 2 (Session History):** Add/update the current session row
- **Section 4 (Playbook Progress):** Mark each section as covered with a distilled summary
- **Section 3 (Domain Confidence Score):** Update after each completed section
- **Section 5 (AI Knowledge Corrections):** Record every divergence immediately
- **Section 6 (Open Issues):** Record every uncertainty immediately

---

## Step 8 — Session close

Follow **Playbook Section 14 (Session Close)**:
- Summarise the domain in 5–6 bullet points and ask the PO to confirm
- List open issues
- Ask: "Is there anything about this domain I should know before generating the domain knowledge document?"

---

## Step 9 — Write the final state file

Write `projects/{PROJECT_CODE}/domain/domain-discovery-state.md` with all sections complete. Calculate the final domain confidence score.

**Output format:** the state file does not include any `[AI Guide]` blocks or labels from the template. Remove all `[AI Guide]` notes — both the Document Level block and any per-section notes. The file starts directly with the document title and sections.

---

## Step 10 — Report and offer to generate

After writing the state file:

```
DOMAIN DISCOVERY COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Domain:           {domain name}
Confidence score: {N}%
State file:       projects/{PROJECT_CODE}/domain/domain-discovery-state.md
─────────────────────────────────────────
```

> "Confidence score: {N}%.
>
> **Recommended:** Generate the domain knowledge document now while I have full session context loaded — this produces higher quality output than loading the state file in a new session.
>
> Generate domain knowledge document now? (yes / no)"

- **If 'yes':** Proceed immediately with `/gen-domain-knowledge {PROJECT_CODE}` — use the live session context as the primary input (not just the saved state file). Skip the "domain discovery was not run" warning since it was just run.
- **If 'no':** Tell the PO: "State saved at `projects/{PROJECT_CODE}/domain/domain-discovery-state.md`. Run `/gen-domain-knowledge {PROJECT_CODE}` in a new session to generate the domain knowledge document."
- **If score < 70%:** Tell the PO: "Confidence score is {N}%. The following sections need more coverage before generating: [list]. Schedule a follow-up session and run `/run-domain-discovery` again to resume."
