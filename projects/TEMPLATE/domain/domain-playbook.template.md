# Domain Discovery Playbook — [Domain Name]

> **[AI Guide — Document Level]**
> This playbook runs a domain discovery session where the PO (or domain expert) teaches the AI about an unfamiliar domain. It is the input phase before generating a domain knowledge document.
>
> This is NOT a requirement discovery session with a client. The goal here is to capture universal domain knowledge — what is always true about this domain regardless of which client is using it.
>
> **When generating this playbook (`/gen-domain-playbook`):**
> - Pre-populate all `[list]` placeholders with your domain knowledge for this specific domain.
> - Expand sections 5 and 6 into one sub-section per entity. The session runner must see actual entity names and states — no placeholders should remain in the generated file.
>
> **When running this playbook (`/run-domain-discovery`):**
> - Load `projects/{PROJECT_CODE}/brief.md` for context on the domain name, sector, and problem space.
> - Do not load any prior domain knowledge document — the purpose of this session is to build that document from scratch (or validate AI knowledge against PO expertise).
> - Work through each section in order. Each section maps to a section in the domain knowledge template.
> - Ask structured questions. Record what the PO states — not what you already know.
> - Where your AI knowledge and the PO's answers agree, confirm agreement and move on. Where they diverge, the PO's answer takes precedence — record it explicitly.
> - At the end of the session, write the domain discovery state file. `/gen-domain-knowledge` reads this file to generate an accurate domain knowledge document.

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] |  | Initial version |

---

## 1. Playbook Scope

### 1.1 What This Playbook Covers
> **[AI Guide]** 2–3 sentences. Name the domain and describe the type of discovery this session conducts. State that this is a domain education session — not a client requirement session.

### 1.2 Documents to Load
> **[AI Guide]** List every document to load before running this playbook.

| Document | Purpose |
|----------|---------|
| `projects/{PROJECT_CODE}/brief.md` | Domain name, sector, problem context |

### 1.3 What This Playbook Does Not Cover
> **[AI Guide]** Bullet list. State explicitly that this playbook does not capture client-specific requirements — those belong in the BRD discovery phase.

- Client-specific requirements (captured in requirement discovery via `/run-brd-discovery`)
- Project-specific entities, rules, or configurations (captured in the BRD)
- Technology choices or architecture (captured in the architecture document)

---

## 2. Pre-Session Checklist

> **[AI Guide]** Run every item below before beginning.

- [ ] Brief loaded — domain name and problem context confirmed.
- [ ] Check for an existing domain knowledge document at `projects/{PROJECT_CODE}/domain/`. If one exists, this session is updating it — note which sections need to change.
- [ ] Check for a prior domain discovery state at `projects/{PROJECT_CODE}/domain/domain-discovery-state.md`. If it exists, this is a resumed session — load it and follow the Resume Instructions.
- [ ] The PO (or domain expert) understands this session is about the domain universally — not about this specific client's implementation.

---

## 3. Session Opening

> **[AI Guide]** Frame the session for the PO before asking any questions.

Tell the PO:
> "We're going to define the universal rules and structure of the [domain name] domain. I'll ask questions about how this domain works in general — not about this specific client. The answers will form a reusable domain knowledge document that all future projects in this domain will use. Where I already have knowledge about this domain, I'll confirm it with you. Where I don't, I'll ask you to teach me."

Confirm with the PO:
- [ ] The domain name (as it will appear in all documents)
- [ ] The primary business problem this domain addresses
- [ ] Any known sub-domains or adjacent domains that should be excluded from this session

---

## 4. Domain Scope

> **[AI Guide]** This section populates Section 1 (Domain Overview) of the domain knowledge document.
>
> **Generation:** Replace `[list]` in the opening prompt with the modules/capabilities you know are typically part of this domain — draw on both AI training knowledge and the brief's problem statement, client context, and strategic goals. Be comprehensive: list every module/capability a mature product in this domain would typically include, not just the obvious core ones.
>
> **Generalization filter — apply before adding anything sourced from the brief:** the brief will contain project-specific product decisions (a particular configurability model, a specific licensing plan, named personas) mixed in with genuinely universal domain concepts. Only add a candidate item at the level of generality that would hold true for *any* implementation of this domain, not just this project. If the brief reveals a specific mechanic or design choice riding on top of a generic capability (e.g. brief says "tiered template options + full customization" — the generic capability is "invoice templating," the tiering/customization model is a project-specific choice, not a domain truth), list only the generic capability here and route the specific mechanic to Section 12 (Known Variations) instead — do not silently drop it and do not silently present it as if every implementation works that way.
> **Session:** Present the pre-populated list first, then ask the PO to confirm, include, or exclude each item, before moving to the open follow-up questions.

**Opening prompt:**
> "Based on my knowledge of this domain and your brief, this domain typically covers: [list of modules/capabilities]. Which of these should be in scope for this document, which should be excluded, and is there anything I'm missing?"

**Follow-ups if needed:**
- "What does this domain explicitly NOT cover? Name adjacent domains or capabilities that should not be in this document."
- "Are there sub-domains within this area that should be separate documents? (e.g. admin portal vs parent portal)"
- If a brief detail was generalized rather than added directly (per the filter above): "Your brief mentions [specific mechanic] — I've kept [generic capability] in scope here, but noted [specific mechanic] as a known variation to confirm during BRD discovery rather than assuming it's how every implementation works. Does that split make sense?"

**State note:** Record the confirmed scope list, exclusions, and the resulting domain scope statement verbatim. These become Section 1.1 and 1.2 of the domain knowledge document. Record any generalized project-specific mechanics as candidate entries for Section 12 (Known Variations), carried forward when that section is reached.

---

## 5. Core Entities

> **[AI Guide]** This section populates Section 3 (Core Entities & Relationships) of the domain knowledge document.
>
> **Generation:** Replace `[list]` in the opening prompt with the entities you know for this domain. Create one sub-section per entity using the question pattern below.
> **Session:** Present the pre-populated entity list first, then ask the PO to confirm, correct, or extend.

**Opening prompt:**
> "Based on my knowledge of this domain, the core entities are: [list]. Does this match your understanding? Are there entities I'm missing, or entities on this list that don't apply?"

**For each entity confirmed from the AI's original list:** present your own proposed definition, attributes, and relationships first — do not ask the PO to originate these from scratch. Consolidate into one confirmation, not a series of open questions:
> "Here's my understanding of [entity] — **Definition:** [business-level definition, not technical]. **Attributes:** [standard business-meaningful attributes — not database fields]. **Relationships:** [how it relates to other entities]. Does this match, or would you correct or extend anything?"

Follow up only on what the PO flags as wrong, missing, or incomplete — do not re-ask what has already been confirmed. If the PO corrects something, ask one targeted follow-up on that point only.

**For an entity the PO added** (one not in the AI's original list — there is no AI knowledge to propose): ask directly, since this is the PO teaching the AI:
- "What is [entity]? Give me a business-level definition — not a technical one."
- "What are its standard business-meaningful attributes?"
- "How does it relate to other entities?"

**For every entity, regardless of source:** "Are there any important notes or constraints about this entity?"

**State note:** Record each confirmed, corrected, and newly identified entity with its definition and relationships.

---

## 6. Lifecycle & Status Model

> **[AI Guide]** This section populates Section 4 (Standard Lifecycle & Status Model) of the domain knowledge document.
>
> **Generation:** Create one sub-section per entity that has a meaningful lifecycle, using the question pattern below. Pre-populate each sub-section heading with the entity name.
> **Session:** Walk through each entity's lifecycle in order.

**Opening prompt:**
> "For each entity with a meaningful lifecycle, I'll walk through the standard states and transitions. Tell me where my understanding is wrong or incomplete."

**For each entity known from AI training:** present the proposed lifecycle first, not open questions:
> "Here's the standard lifecycle for [entity] — **States:** [list]. **Transitions:** [State A] → [State B] triggered by [trigger], and so on for each transition. **Terminal states:** [list, if any]. **Transition rules:** [any conditions gating a transition]. Does this match, or would you correct or extend anything?"

Follow up only on what the PO flags as wrong or incomplete.

**For an entity with no known standard lifecycle** (novel entity, or the AI has no prior knowledge of how it behaves): ask directly:
- "What states can [entity] be in?"
- "What triggers the transition from [State A] to [State B]?"
- "Are there any states [entity] can enter but never leave? (Terminal states)"
- "Are there any transition rules — conditions that must be true before a transition is allowed?"

**State note:** Record states, transitions, triggers, terminal states, and transition rules for each entity. Flag any lifecycle where the PO's answer contradicts standard domain knowledge — record both the standard and the PO's correction.

---

## 7. Universal Business Rules

> **[AI Guide]** This section populates Section 5 (Universal Business Rules) of the domain knowledge document.
>
> **Generation:** Pre-populate the opening prompt with the universal business rules you know apply to this domain.
> **Session:** Read out the pre-populated rules. Add any the PO raises that are not already listed.

**Opening prompt:**
> "I'll read out the business rules I know apply universally in this domain. Tell me which ones you'd add, remove, or correct."

**Before reading out any pre-populated rule:** skip any rule tied to an entity the PO removed in Section 5 (Core Entities) — do not ask about it.

**For each rule proposed or raised:**
- "Does this rule apply in every implementation of this domain without exception?"
- "If a client wanted to do something different — would that be a valid variation or a red flag?"

**For rules the PO adds:**
- "Is this rule universal (always true) or does it vary between clients?"
- "Can you state it as a single sentence beginning with 'A [entity] must...' or 'The system must...'?"

**State note:** Record only rules that are universal — applicable in every implementation. Rules that vary between clients belong in Section 10 (Known Variations), not here.

---

## 8. Standard Validations

> **[AI Guide]** This section populates Section 6 (Standard Validations) of the domain knowledge document.

**Opening prompt:**
> "What validations are always enforced in this domain — things the system must always check before allowing an action?"

**Before reading out any pre-populated validation:** skip any validation tied to an entity the PO removed in Section 5 (Core Entities) — do not ask about it.

**For each validation:**
- "Why is this validation enforced? What business consequence does it prevent?"
- "Does this apply in every implementation or only in some?"

**State note:** Record only validations with a business reason that apply universally. Technical constraints (field length, data type) belong in technical design, not here.

---

## 9. Common Personas & Roles

> **[AI Guide]** This section populates Section 7 (Common Personas & Roles) of the domain knowledge document.
>
> **Generation:** Replace `[list]` in the opening prompt with the standard roles you know for this domain.
> **Session:** Present the pre-populated role list first, then ask the PO to confirm, correct, or extend.

**Opening prompt:**
> "The standard roles in this domain are typically: [list]. Does this match your experience? Are there roles I'm missing?"

**For each role confirmed or added:**
- "What are their typical responsibilities?"
- "What do they typically have access to?"
- "What are they typically excluded from?"

**State note:** Record these as starting points — they will be confirmed with the client during requirement discovery. Do not assert these roles will definitely exist for every client.

---

## 10. Typical Integration Points

> **[AI Guide]** This section populates Section 8 (Typical Integration Points) of the domain knowledge document.
>
> **Generation:** Pre-populate the opening prompt with the integration points you know are common in this domain.
> **Session:** Present the pre-populated list, then ask the PO to confirm, remove, or extend.

**Opening prompt:**
> "What external systems or services are commonly used in this domain? I'll propose the ones I know — correct or extend the list."

**For each integration:**
- "What is the purpose of this integration?"
- "Is data flowing in, out, or both?"
- "Is this always needed or does it vary by client?"

**State note:** Record as common but not guaranteed. These become confirmation questions during requirement discovery.

---

## 11. Regulatory Baseline

> **[AI Guide]** This section populates Section 9 (Regulatory Baseline) of the domain knowledge document.

**Opening prompt:**
> "What compliance or regulatory obligations apply to this domain regardless of which country the client is in?"

**For each obligation raised:**
- "Does this apply universally, or only in specific countries or sectors?"
- "What does it require the system to do?"

**State note:** Record only country-agnostic obligations here. Country-specific requirements belong in a country extension document.

---

## 12. Known Variations

> **[AI Guide]** This section populates Section 10 (Known Variations) of the domain knowledge document.
>
> **First, carry forward anything generalized out of Section 4** (project-specific mechanics that were abstracted to a generic capability there, per that section's generalization filter) — these are pre-populated variation candidates, present them before asking the open question below.

**Opening prompt:**
> "What aspects of this domain commonly differ between client implementations? These are the things we must always ask during requirement discovery — we cannot assume them."

**For each variation:**
- "What are the common options?"
- "Is there a most common default?"
- "Would it be a red flag if a client wanted a very unusual option here?"

**State note:** These directly become discovery questions in the BRD playbook. The more complete this section, the fewer gaps in discovery.

---

## 13. Typical Scope Boundaries

> **[AI Guide]** This section populates Section 11 (Typical Scope Boundaries) of the domain knowledge document.

**Questions:**
- "What capabilities does almost every client need in a first implementation?"
- "What capabilities are commonly wanted but rarely built first? Why?"

**State note:** Record both what is typically in scope for v1 and what is typically deferred. Include the reason for deferral (complexity, dependency, cost).

---

## 14. Session Close

> **[AI Guide]** Close the session with a summary and confirmation before writing the state file.

**Closing steps:**

1. **Summarise the domain:** Restate the domain scope, key entities, and lifecycle in 5–6 bullet points. Ask the PO to confirm or correct.
2. **List open items:** Read out anything that was flagged as uncertain or needing follow-up.
3. **Confirm readiness:** Ask: "Is there anything about this domain I should know before generating the domain knowledge document?"

**Domain knowledge drafting note:** After writing the state file, run `/gen-domain-knowledge` with the path to the state file. The command uses this state as primary input, enriched by AI knowledge where the PO did not address a topic.

---

## 15. Playbook Verification

> **[AI Guide]** Run these checks after writing a new domain playbook — before using it.

- [ ] Every section of the domain knowledge template (sections 1–11) is covered by at least one question in this playbook.
- [ ] The playbook explicitly separates universal domain knowledge from client-specific requirements — no questions ask about "what the client wants."
- [ ] Section 12 (Known Variations) questions are clear enough that the answers will directly feed into BRD playbook question topics.
