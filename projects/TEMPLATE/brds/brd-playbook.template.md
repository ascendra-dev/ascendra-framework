# BRD Discovery Playbook — [Domain Name]

> **[AI Guide — Document Level]**
>
> **When generating this playbook (`/gen-brd-playbook`):**
> - Pre-populate every section from the domain knowledge document. No `[placeholder]` text should remain in the generated file.
> - Section 4 topics must map 1:1 to the domain knowledge document's Section 10 (Known Variations). Every known variation must have at least one question.
> - Section 5 Scope Risk Flags must cover every significant complexity or regulatory topic in the domain. Every SRF referenced in Section 4 must appear here.
> - Section 6 journeys must cover every primary lifecycle flow from the domain knowledge document's Section 4.
> - Section 7 must list every integration from the domain knowledge document's Section 8.
> - If the project is an Extension: add conditional markers in Sections 1.2 and 4 indicating which questions apply only when extension-specific conditions are met. Cross-reference the extension domain document for those sections.
>
> **When running this playbook (`/run-brd-discovery`):**
> This playbook runs a requirement discovery session with a client. It is an active process document — not a reference document.
>
> Before starting:
> - Load the relevant domain knowledge document(s) and this playbook simultaneously.
> - The domain doc gives you facts. This playbook tells you what to do with them.
>
> How this playbook works:
> - **Core Discovery Questions** are the questions you must ask. They surface information the domain doc cannot assume.
> - **Scope Risk Flags** are triggers. When a client answer matches a trigger, stop and respond with the prescribed escalation before continuing.
> - **Journey Walkthroughs** are guided conversations — not one-off questions. Walk the client through a complete workflow to surface requirements they would not think to state.
> - **Output Confirmation** is a checklist. Go through it explicitly at the end of the session — clients assume things are included that are not, and assume things are excluded that are obvious.
>
> What this playbook is not:
> - It is not a script. Adapt the language. Use the client's own terms once you have learned them.
> - It is not exhaustive. If a client says something unexpected, probe it — do not skip it because it is not in the list.
> - It is not a BRD. Session notes go into the BRD after the session closes.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] |  | Initial version |

---

## 1. Playbook Scope

### 1.1 What This Playbook Covers
> **[AI Guide]** 2–3 sentences. Name the domain and the type of client this playbook is designed for. State what kind of discovery session this runs (e.g. initial scoping, detailed requirements, handover review).

### 1.2 Domain Documents to Load
> **[AI Guide]** List every document that must be loaded before running this playbook. These determine what you already know — and therefore what you do not ask.

| Document | Purpose |
|----------|---------|
|          |         |

### 1.3 What This Playbook Does Not Cover
> **[AI Guide]** Bullet list. Name adjacent domains or scenarios that are out of scope for this playbook. Prevents the AI from applying this playbook to the wrong type of engagement.

---

## 2. Pre-Session Checklist

> **[AI Guide]** Run every item below before beginning. Do not start the session if any item is unresolved.

- [ ] Domain knowledge document(s) listed in section 1.2 are loaded and confirmed.
- [ ] The client's name, organisation name, and domain (industry/sector) are known.
- [ ] The session mode is confirmed: chat discovery (AI leads questions) vs document review (client provides brief upfront).
- [ ] **Check for a discovery state file at `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`.**
  - If it **exists**: load it now. Read the **Resume Instructions** section at the bottom. Follow those instructions instead of continuing with the standard session opening. Do not re-ask questions already covered.
  - If it **does not exist**: this is a fresh session — continue with the checklist below and proceed to section 3.
- [ ] If this is not the first session with this client: review any prior BRD draft and open questions before proceeding.
- [ ] No scope commitments have been made before this session. Discovery precedes scoping.

---

## 3. Session Opening

> **[AI Guide]** Use the opening to frame the session for the client. The goal is to make them comfortable, set expectations, and confirm the basics before asking domain questions.
>
> Adapt the language to the client. Do not read this verbatim.

**Frame the session:**
Briefly explain what you will do: ask about their current process, what they need the system to handle, and what problems they are trying to solve. Tell them there are no wrong answers — the goal is to understand their situation, not to sell them a specific solution.

**Confirm the basics before asking any domain questions:**
- [ ] Organisation name and type
- [ ] Primary business activity (what do they sell, provide, or manage?)
- [ ] Approximate size: number of staff, number of clients/customers, transaction volumes if relevant
- [ ] Who else is involved in this project beyond today's session (decision makers, finance, IT, operations)?
- [ ] What prompted them to look at this system now? (Pain point, growth, compliance, previous system failure?)

> **[AI Guide]** Record the answers to these questions. They inform how you frame subsequent questions and which scope risk flags are most likely to trigger.

---

## 4. Core Discovery Questions

> **[AI Guide — Section Instructions]**
> This section contains the questions you must ask. They are organised by topic.
>
> How to use this section:
> - Go through every topic. Do not skip topics because you assume the answer.
> - Ask the primary question first. Ask follow-up questions only if the primary answer is ambiguous or triggers a follow-up marker.
> - When a client answer triggers a **Scope Risk Flag**, jump to section 5 immediately. Do not continue to the next question until the scope risk is addressed.
> - Record every answer. The BRD drafting instructions in section 10 tell you how to use them.

---

### 4.1 [Topic Name]

> **[AI Guide]** Replace with topic name (e.g. "Invoice Creation", "Payment Collection", "Approvals"). Add one subsection per major topic. Topics should map to the Known Variations in the domain knowledge document — one topic per variation area.

**Primary question:**
> [Write the question the AI asks the client. Plain English. One question per block.]

**Follow-ups if needed:**
- [Sub-question A]
- [Sub-question B]

**Scope risk trigger:** [Describe the answer that triggers a scope risk flag. E.g. "If client mentions X, go to scope risk flag SRF-001."] → SRF-001

**BRD note:** [Instruction for the AI on what to do with this answer when drafting the BRD. E.g. "Record as a functional requirement under feature area X if client confirms Y."]

---

### 4.2 [Topic Name]

**Primary question:**
> [...]

**Follow-ups if needed:**
- [...]

**Scope risk trigger:** [...]

**BRD note:** [...]

---

## 5. Scope Risk Flags

> **[AI Guide — Section Instructions]**
> A scope risk flag is triggered when a client answer indicates complexity, dependency, or regulatory overhead that falls outside standard scope.
>
> When a flag triggers:
> 1. Do not continue to the next discovery question.
> 2. Explain the implication to the client clearly and in plain language.
> 3. Record the trigger and the client's response.
> 4. Confirm whether to continue scoping this item in the current session or flag it as a separate conversation.
> 5. Note it as an open question in the BRD if unresolved.
>
> Do not guess the answer for the client. Do not proceed as if the flag didn't trigger.

---

### SRF-001: [Flag Name]

> **[AI Guide]** Replace with flag name (e.g. "IRIS e-invoicing mandate"). Add one subsection per scope risk.

**Triggered when:** [The specific client answer or condition that activates this flag]

**Implication:** [What this means for scope, timeline, or cost — in plain language]

**Response to client:**
> [What to say to the client when this flag triggers. Honest and direct. Not alarmist.]

**Action:** [What to record and what to do next: continue in session / defer to follow-up / escalate to senior]

---

### SRF-002: [Flag Name]

**Triggered when:** [...]

**Implication:** [...]

**Response to client:**
> [...]

**Action:** [...]

---

## 6. Journey Walkthroughs

> **[AI Guide — Section Instructions]**
> A journey walkthrough is a structured conversation — not a questionnaire. You guide the client through a complete end-to-end workflow using their own language and process.
>
> How to run a walkthrough:
> 1. Set the scene: "Walk me through what happens from the moment [X occurs] to the moment [Y is complete]."
> 2. Let the client narrate. Do not interrupt with questions during the narrative.
> 3. After the narrative, probe gaps: "What happens if...?", "Who is responsible for...?", "What does [person] see at this point?"
> 4. Record every step the client describes — not just the steps in the template.
>
> The template walkthrough below is a guide for what to probe. The client's actual journey may differ.

---

### 6.1 [Journey Name]

> **[AI Guide]** Replace with journey name (e.g. "Invoice Creation to Payment"). Add one subsection per key journey. Journeys should cover the primary end-to-end flows for the domain.

**Set the scene with:**
> "[Prompt text to start the walkthrough conversation — a specific, grounded scenario the client will relate to]"

**Standard steps to confirm:**
> **[AI Guide]** List the typical steps for this journey as known from the domain document. Use these to probe gaps in the client's narrative — not to lead them toward a predetermined answer.

1. [Step]
2. [Step]
3. [Step]

**Probe questions for this journey:**
- What happens if [exception condition]?
- Who approves / authorises [step]?
- What does [secondary actor] see or do at [stage]?
- How does [actor] know when [event] has occurred?
- What is the failure state and who handles it?

**Things to note for BRD:**
> **[AI Guide]** [What to capture from this walkthrough that feeds directly into BRD sections. E.g. "Note any approvals mentioned — these are functional requirements in the Approvals feature area."]

---

### 6.2 [Journey Name]

**Set the scene with:**
> "[...]"

**Standard steps to confirm:**
1. [...]

**Probe questions:**
- [...]

**Things to note for BRD:**
> **[AI Guide]** [...]

---

## 7. Integration Confirmation

> **[AI Guide]**
> Go through the typical integration points defined in the domain document (section 8) one by one. Do not assume any integration is needed — confirm each explicitly.
>
> For each integration: ask whether it applies, who owns it, and whether it already exists or needs to be set up.
> Flag any integration with an onboarding lead time as an external dependency in the BRD.

| Integration | Question to ask | If yes: record in BRD | External dependency? |
|------------|----------------|----------------------|---------------------|
| [Name] | [Does the client use / need this?] | [What to capture] | Yes / No — [note lead time if yes] |

---

## 8. Output Confirmation

> **[AI Guide]**
> At the end of the session, confirm every output type below explicitly with the client. Clients assume standard outputs are included. Some are not. Some assumed exclusions are obvious inclusions.
>
> Go through this list out loud. Do not skip it or assume answers from context.

### 8.1 Reports & Exports

> **[AI Guide]** List the standard reports and exports for this domain. For each one, confirm: is it needed? who uses it? what format? what frequency?

| Output | Confirm needed? | Consumer | Format | Frequency |
|--------|----------------|---------|--------|-----------|
| [Report name] | Yes / No | | | |

### 8.2 Notifications

> **[AI Guide]** List the standard notifications for this domain. For each one, confirm: is it needed? which channel? who receives it?

| Notification | Confirm needed? | Channel | Recipient |
|-------------|----------------|---------|-----------|
| [Notification name] | Yes / No | | |

### 8.3 Access Levels

> **[AI Guide]** Confirm the permission model explicitly. Ask who needs admin access, who should only see their own data, and whether there are any reporting-only roles.

- [ ] Confirm which roles need full system access
- [ ] Confirm which roles are limited to their own records only
- [ ] Confirm whether any roles need read-only / reporting access only
- [ ] Confirm whether any data must be hidden from certain internal roles (e.g. salary information, management fees)

### 8.4 Non-Functional Requirements

> **[AI Guide]** Unlike Reports/Notifications/Access Levels above, NFRs are not a fixed checklist — they must be actively pulled from two specific sources that nothing else in this playbook prompts for, so do not skip this step just because "nothing NFR-related came up" during the session:
> 1. The client's brief (Client Context section) — check explicitly for any stated volume, scale, growth trajectory, or timeline expectation.
> 2. The domain knowledge document's Section 9 (Regulatory Baseline) — check explicitly for any retention, availability, or compliance-driven obligation that implies a measurable constraint (e.g. a record retention period).
>
> If a source states the fact but not a concrete figure (e.g. "a defined period" with no number), confirm the actual number with the client now, or record it as an Open Question if genuinely unresolved. Do not let a vague source statement become a vague NFR.

- [ ] Checked the brief's Client Context for stated volume/scale/growth expectations
- [ ] Checked the domain document's Section 9 (Regulatory Baseline) for retention/compliance obligations implying a measurable constraint
- [ ] Any figure not already concrete is either confirmed with the client now, or recorded as an Open Question

---

## 9. Session Close

> **[AI Guide]**
> Close the session by summarising what you heard and explicitly naming the open questions. Give the client the chance to correct misunderstandings or add anything missed.
>
> Do not rush this step. A client who leaves with an uncorrected misunderstanding will request changes after the BRD is written.

**Closing steps:**

1. **Summarise scope:** Restate in 5–8 bullet points what the system will do, based on what the client described. Ask if this matches their understanding.

2. **Name what is out of scope:** State explicitly 2–3 things that are not in scope, including anything the client mentioned that was deferred or excluded. Ask the client to confirm these exclusions.

3. **List open questions:** Read out every item that was flagged as unresolved during the session. Confirm who will answer each one and by when.

4. **Confirm next step:** State what happens next — BRD draft will be produced, or next session is required.

> **[AI Guide]** Record the client's corrections and additions before closing. Do not produce the BRD draft from memory of what you expected them to say — use what they actually confirmed.

---

## 9.5 Saving State on Pause

> **[AI Guide]**
> Write a discovery state file whenever:
> - The client asks to pause and resume later
> - The session ends naturally but the BRD is not yet complete
> - More than one session is required to complete discovery
>
> The state file is the only thing that survives into the next session. Write it as if the next session starts cold — because it does.
>
> **How to save state:**
> 1. Write the file to `projects/{PROJECT_CODE}/brds/brd-discovery-state.md`. Create the file if it does not exist. Overwrite it if it does — the file always reflects the latest saved state.
> 2. After writing, confirm to the client: "Your session has been saved. When you return, load this project's discovery state file and the AI will resume from where we left off."
>
> **What to write:**
> - Every topic from section 4 that was covered — with a one-line distillation of the client's answer. Not a transcript. The AI on resume must be able to draft BRD requirements from this distillation alone.
> - Every topic not yet covered — clearly marked as pending.
> - Every scope risk flag that triggered — what triggered it, what the client said, current status.
> - Every open question — who owns it, due date if stated.
> - Captured client answers in enough detail for BRD drafting — written as if you were briefing a colleague who was not in the room.
> - Resume instructions: the exact section and topic to start from next session, and anything the client committed to follow up on.
>
> **What not to write:**
> - A transcript or log of the conversation. The state file is a distillation, not a record.
> - Implementation details or your own inferences. Only what the client said.
> - Domain knowledge facts. Those come from the domain doc — not the state file.

---

## 10. BRD Drafting Instructions

> **[AI Guide — Section Instructions]**
> Use this section after the session closes to turn session notes into a BRD. Follow the instructions below in order.

### 10.1 Before You Start

- Load `projects/TEMPLATE/brds/brd.template.md`.
- Load the domain knowledge document(s) from section 1.2 of this playbook.
- Have the session notes from sections 4, 6, 7, 8, and 9 ready.

### 10.2 Source Tagging Rules

Every requirement you write must be tagged. Apply the tags as follows:

- `[Client-Stated]` — the client said this explicitly, in their own words, during the session.
- `[Domain-Default]` — this is a universal domain fact the client confirmed applies without deviation. Only include in the BRD if the client explicitly overrode it or the requirement drives a specific implementation decision that must be documented.
- `[Assumed]` — you have inferred this from context. Use sparingly. Every `[Assumed]` tag is a risk. If in doubt, add it to Open Questions instead.

Do not restate facts from the domain knowledge document as BRD requirements. The domain doc is pre-loaded knowledge — the BRD captures only what is project-specific.

### 10.3 Populating Each BRD Section

| BRD Section | Source in session notes |
|-------------|------------------------|
| 1. Project Overview | Session opening answers + session close summary |
| 2. Stakeholders & Personas | Roles named during journey walkthroughs + discovery questions |
| 3. User Roles | Permissions confirmed in section 8.3 (Access Levels) |
| 4. User Journeys | Journey walkthrough notes (section 6) |
| 5. Functional Requirements | Core discovery question answers (section 4) + journey probe answers |
| 6. Business Rules | Client-specific rules surfaced in discovery; rules that deviate from UBRs in domain doc |
| 7. Data Requirements | Entities confirmed during journey walkthroughs; any client-specific attributes stated |
| 8. Integration Requirements | Integration confirmation table (section 7) |
| 9. Security Requirements | Permission model from section 8.3; compliance from section 5 (scope risk flags if triggered) |
| 10. Non-Functional Requirements | Only if explicitly stated by the client during discovery |
| 11. Dependencies | External integrations with onboarding lead times; open items from scope risk flags |
| 12. Open Questions | Every unresolved item from section 9 close |

### 10.4 Quality Check Before Handing Over

Before marking the BRD as draft-ready:

- [ ] Run the Pre-Approval Verification in BRD section 13.
- [ ] Every open question has an owner and a due date.
- [ ] Every scope risk flag that triggered has been recorded — either as a dependency, open question, or explicit out-of-scope exclusion.
- [ ] No `[Assumed]` tag covers something that should have been confirmed during the session. If found, add to Open Questions.

---

## 11. Playbook Verification

> **[AI Guide]**
> Run these checks after writing a new playbook — before using it in a live session.

- [ ] Every Known Variation in the domain document's section 10 is covered by at least one question in section 4 of this playbook. A known variation with no corresponding question will never be surfaced in discovery.
- [ ] Every Typical Integration Point from the domain document's section 8 appears in the Integration Confirmation table (section 7 of this playbook).
- [ ] Every significant complexity topic from the domain has a corresponding Scope Risk Flag in section 5.
- [ ] The Journey Walkthroughs in section 6 cover every primary journey from the domain document's section 8 (if the domain doc defines journeys) or from section 4 (standard lifecycle).
- [ ] The Output Confirmation in section 8 includes every standard report, notification, and permission model mentioned in the domain doc.
- [ ] Run at least one mock discovery session using `/run-mock-discovery` before treating this playbook as production-ready. Record the outcome in the domain document's section 12.6 (Mock Discovery Validation).
