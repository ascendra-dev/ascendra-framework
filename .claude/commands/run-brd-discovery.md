# Run Discovery Session

You are running a live requirement discovery session for an Ascendra client project. You conduct the session interactively — asking questions, recording answers, probing edge cases, and building a complete picture of what the system must do. The session is grounded in the domain knowledge document (what is universal about this domain) and the playbook (what questions to ask this type of client).

At session end: write the discovery state file. If discovery is complete and confidence is sufficient, immediately offer to generate the BRD in the same session — this is always recommended over stopping and resuming, because the full context is live.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a project code, a domain knowledge file path, and a BRD playbook file path. Example:

```
/run-brd-discovery HARBORVIEW-INV-001 projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md
```

Extract:
- **PROJECT_CODE** — e.g. `HARBORVIEW-INV-001`
- **Domain knowledge file** — path to the domain knowledge document
- **Playbook file** — path to the BRD discovery playbook

If any are missing, ask for them all at once:

> "To run the discovery session I need:
> 1. **Project code** — e.g. `HARBORVIEW-INV-001`
> 2. **Domain knowledge file** — path (e.g. `projects/HARBORVIEW-INV-001/domain/invoice-payment-core.md`)
> 3. **BRD playbook file** — path (e.g. `projects/HARBORVIEW-INV-001/brds/invoice-payment-brd-playbook.md`)"

---

## Step 2 — Gate check (required before anything else)

Check all three conditions. Stop and report any failure — do not proceed with a partial context load.

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Brief exists | `projects/{PROJECT_CODE}/brief.md` must exist | "brief.md not found at `projects/{PROJECT_CODE}/brief.md`. Run `/init-project` first, then complete the brief before running discovery." |
| Domain knowledge file exists | Path from arguments must resolve to an existing file | "Domain knowledge file not found at `{path}`. Run `/gen-domain-knowledge` to generate it first." |
| Playbook file exists | Path from arguments must resolve to an existing file | "Playbook file not found at `{path}`. Run `/gen-brd-playbook {domain-knowledge-path}` to generate it first." |

---

## Step 3 — Read all required files

Read every file in full before beginning the session:

1. `projects/{PROJECT_CODE}/brief.md` — project context: client name, domain, problem statement, goals, scope, constraints
2. The domain knowledge file — understand the domain thoroughly before asking a single question
3. The playbook file — your session guide: questions, scope risk flags, journey walkthroughs, output confirmation checklist
4. `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` — check if it exists:
   - **Does not exist (first session):** create it now from `projects/TEMPLATE/brds/brd-discovery-state.template.md`. Pre-fill Section 1 (Session Context): Project Code = `{PROJECT_CODE}`, Domain Name = from domain knowledge Section 1.1, Domain Playbook Used = playbook file path from arguments, Target Output = `projects/{PROJECT_CODE}/brds/brd-{domain-slug}-v1.md`.
   - **Exists:** read it in full. If Section 2 (Session History) has prior entries, this is a **resumed session** — pick up from where the previous session ended using the playbook progress in Section 3.

---

## Step 4 — Pre-session checklist

Before opening the session with the user, check:

- [ ] Brief Section 2 (Problem Statement) is filled in — if blank, stop: "The brief is incomplete. Please fill in the problem statement, goals, and scope in `projects/{PROJECT_CODE}/brief.md` before starting discovery."
- [ ] Domain knowledge file loaded ✓
- [ ] Playbook loaded ✓
- [ ] Session type confirmed: **Initial** (no prior sessions) or **Resumed** (prior sessions exist — state which sections were completed)

If resumed, tell the user:
> "Resuming from session {N}. The following playbook sections were already covered: [list]. I'll continue from [next section]. Ready?"

---

## Step 5 — Open the session

Tell the user:

> "Starting discovery session for **{Project Name}** ({PROJECT_CODE}).
>
> Domain: {domain}
> Client: {client from brief}
>
> I'll work through the discovery questions section by section. You can answer in your own words — I'll extract what I need. Let me know if you want to skip any topic or come back to something.
>
> Let's begin with the basics."

Then follow **Playbook Section 3 (Session Opening)** exactly.

---

## Step 6 — Conduct the discovery session

Work through the playbook section by section:

**Playbook Section 4 (Core Discovery Questions):** Ask each subsection's primary question. If the client's answer matches a scope risk trigger, go to the matching SRF in Playbook Section 5 immediately — do not continue to the next question until the risk is resolved.

**Playbook Section 6 (Journey Walkthroughs):** After all core questions are answered, walk through each primary journey. Set the scene, walk step by step, probe at each step for edge cases and exceptions.

**Playbook Section 7 (Integration Confirmation):** Work through the integration table. Confirm in/out for each integration. Record who owns each confirmed integration.

**Playbook Section 8 (Output Confirmation):** Walk through the reports, notifications, and access levels checklist. Correct any assumptions.

**Rules during the session:**
- Do not ask more than 2–3 questions at a time — the client should not feel interrogated
- Use the client's own words once you learn their terminology
- When an answer is ambiguous or contradicts something said earlier, probe immediately — do not defer
- When a scope risk flag triggers, stop the normal flow and work through the SRF response before continuing
- Track the requirement confidence score mentally as you go: coverage of known-variation topics from domain knowledge Section 10

---

## Step 7 — Save state progressively

After completing each major playbook section (Section 4, 6, 7, 8), save the accumulated answers to `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`. Do not wait until the end of the session.

Update the following sections of the discovery state file as you go:
- **Section 2 (Session History):** Add/update the current session row
- **Section 3 (Playbook Progress):** Mark each covered topic Covered/Pending with a one-line distillation of the client's answer
- **Section 4 (Scope Risk Flags):** Record every flag that triggered — what the client said, their response, current status
- **Section 5 (Integration Confirmation):** Update each integration row as it is confirmed in or out
- **Section 6 (Output Confirmation):** Update after Section 8 of the playbook is run
- **Section 7 (Open Questions):** Record every ambiguity, contradiction, or unanswered question immediately with an owner

---

## Step 8 — Session close

After all playbook sections are completed, follow **Playbook Section 9 (Session Close)**:
- Summarise what was covered
- Name any topics that are out of scope
- List open questions that remain
- Confirm client's next step

---

## Step 9 — Write the final discovery state

Write `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` with all sections complete:
- Section 1: Project and session context (pre-filled at session start)
- Section 2: Session history (this session added)
- Section 3: Playbook progress (all topics marked Covered/Pending with one-line client answer distillations)
- Section 4: Scope risk flags (all triggered flags with client responses and status)
- Section 5: Integration confirmation (all integrations confirmed in/out/pending)
- Section 6: Output confirmation (reports, notifications, access levels)
- Section 7: Open questions (each with owner)
- Section 8: Captured client answers — one block per topic covered, written so a colleague can draft the BRD from this section alone
- Section 9: Resume instructions (if discovery is incomplete — next section to start from, pending topics, client commitments)

**Output format:** the state file does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` or `[AI Guide — Verification]` notes from the template. The file starts directly with the document title and sections. Section content (tables, lists, narrative answers) is included; only the `[AI Guide]` instruction blocks are stripped.

---

## Step 10 — Offer to generate the BRD in this session

After writing the discovery state file, calculate the requirement confidence score from Section 3 (Playbook Progress): count topics marked Covered divided by total topics, expressed as a percentage.

- **Score ≥ 85% and all sections covered:** The session is complete.

```
DISCOVERY COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Project:          {Project Name}
Domain:           {domain}
Confidence score: {N}%
State file:       projects/{PROJECT_CODE}/brds/brd-discovery-state.md
─────────────────────────────────────────
```

> "Confidence score: {N}%.
>
> **Recommended:** Generate the BRD now while I have the full session context loaded. This avoids reloading state in a new session and produces a higher-quality BRD since I can directly reference what you told me.
>
> Generate BRD now? (Type 'yes' to proceed or 'no' to stop here.)"

- **If 'yes':** Proceed immediately with BRD generation without ending the session. Read `projects/TEMPLATE/brds/brd.template.md` now and follow all steps from `/gen-brd` exactly — using the live session context (not just the saved state file) as the discovery input. Write the BRD to `projects/{PROJECT_CODE}/brds/brd-core-v1.md`.

- **If 'no':** Tell the user: "Discovery state saved. Run `/gen-brd {PROJECT_CODE}` in a new session to generate the BRD."

- **Score < 85% or sections incomplete:** Tell the user: "Confidence score is {N}%. Discovery is not yet sufficient for BRD generation. The following topics need more coverage: [list gaps from Section 4]. Schedule a follow-up session and run `/run-brd-discovery` again to resume."

---

## Resuming across sessions

If `projects/{PROJECT_CODE}/brds/brd-discovery-state.md` already exists with completed sections: skip those sections. Start from the first uncovered topic in Section 3 (Playbook Progress). Reference the saved answers throughout — do not re-ask covered questions.

If the user opens a resumed session and wants to revisit a previously covered topic: allow it, update the saved answer, and note the revision in Section 7 (Open Questions) with the reason for the change. If the revision contradicts an answer already reflected in a Scope Risk Flag (Section 4), update that flag's status too rather than leaving it stale.
