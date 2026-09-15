# Domain Knowledge Document — [Domain Name]

> **[AI Guide — Document Level]**
> This document captures universal knowledge about a specific domain. It is loaded by the AI before any discovery session in this domain.
>
> **Ubiquitous Language:** Terms defined in Section 2 are the authoritative vocabulary for this domain. Once defined, they are binding across all artifacts that reference this domain — BRD, epics, stories, architecture, and code. Do not use synonyms or informal names for these terms in any artifact. If a term appears in conversation or in a client document under a different name, map it to the canonical term from Section 2 before recording it.
>
> How to use this document during discovery:
> - Terminology in section 2 defines how terms are used throughout this document and in discovery sessions. Use it to interpret client language accurately and to speak precisely.
> - Facts in sections 3–6 are established truth. Do not ask the client about these.
> - Personas in section 7 are standard starting points. Confirm actual names, titles, and permission boundaries with the client during requirement discovery — not during domain knowledge generation.
> - Integrations in section 8 are common, not guaranteed. Confirm which apply to this client.
> - Context-specific rules — country, sector, or client-type — belong in extension documents, not in this document.
> - Variations in section 10 are the things you MUST ask the client about — they cannot be assumed.
> - Scope boundaries in section 11 guide what to propose as in-scope vs what to flag as a future phase.
>
> How to use this document when writing the BRD:
> - Do not restate anything from sections 3–6 in the BRD unless the client has explicitly overridden it.
> - Tag any overridden rule or entity deviation in the BRD as [Client-Stated] with a note on what was overridden.

---

## Document Control

> **[AI Guide]** Increment the version whenever facts, rules, or scope boundaries change. Domain knowledge changes rarely — a version bump signals a meaningful update.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] |  | Initial version |

---

## 1. Domain Overview

### 1.1 What This Document Covers
> **[AI Guide]** 2–3 sentences. Name the domain precisely. Describe the business problem this domain addresses and the type of system it covers. Be specific enough that it is clear what is in scope for this document and what is not.
>
> **Sub-domain scoping:** This document should cover one cohesive sub-domain — a single, focused area of business capability. If the domain spans multiple distinct user-facing portals or independent capability areas (e.g. both an admin portal and a parent portal in a school system), split into separate domain documents — one per sub-domain. Large single documents degrade context window performance and make discovery sessions unfocused. A well-scoped domain document covers one sub-domain completely rather than multiple sub-domains partially.
>
> **Phase expansion:** If a project adds a later phase in the same sub-domain — new features, extended scope, additional entities — update this document and increment the version in Document Control. Do not create a new domain document for the same sub-domain. If a later phase introduces a genuinely different sub-domain, create a new document for it and reference it in Section 1.3. Cross-phase sub-domain dependencies — where a later project builds on domain knowledge from this one — are handled via the child project's brief Extension Context (`Project Type: Extension`, `Extends: {THIS_PROJECT_CODE}`). Commands on the child project resolve the chain automatically and load this document before generating the child's delta document. Do not merge documents.

### 1.2 What This Document Does Not Cover
> **[AI Guide]** Bullet list. Name adjacent domains or capabilities that are explicitly out of scope for this document. This prevents the AI from treating adjacent knowledge as applicable when it is not.
> e.g. "Payroll processing", "Inventory management", "Customer relationship management"

### 1.3 Related Documents
> **[AI Guide]** Leave this table empty at creation time — do not pre-populate with documents that do not yet exist. Update it as extension documents, playbooks, and related domain documents are created.
>
> If no related documents exist yet, write "None at this time." in the table body.

| Document | Relationship |
|----------|-------------|
| None at this time. | — |

---

## 2. Domain Terminology

> **[AI Guide]** These terms are the ubiquitous language for this domain. Use the exact terms defined here in all downstream artifacts — BRD, stories, architecture, code. Never substitute synonyms.
>
> Define terms that have a specific meaning in this domain. Include terms that:
> - A client might use during discovery and the AI must interpret correctly
> - Are used throughout this document in sections 3–6 and must be unambiguous
> - Have a different meaning in this domain than in everyday language
>
> Do not define general business terms the AI already knows. Only include terms where precision matters or where the domain uses a word differently from common usage.
>
> This section is for domain terms, not project-specific terms. Project-specific terminology belongs in the BRD glossary.

| Term | Definition | Notes |
|------|-----------|-------|
|      |           |       |

---

## 3. Core Entities & Relationships

> **[AI Guide]** Define the standard data model for this domain at conceptual level. No schema, no field types, no database concerns.
>
> Include every entity that is central to the domain. For each entity, capture what it is, its standard business-meaningful attributes, and how it relates to other entities.
>
> This section defines the baseline. In the BRD, only capture entities that deviate from this baseline or have client-specific attributes.

### [Entity Name]
> **[AI Guide]** Replace with entity name. Add one subsection per core entity.

**What it is:** ...
**Standard attributes:** ...
**Relationships:** ...
**Notes:** ...

---

## 4. Standard Lifecycle & Status Model

> **[AI Guide]** Define the standard state machine for every entity in this domain that has a meaningful lifecycle. Include all valid states, all valid transitions, what triggers each transition, and which states are terminal.
>
> This is one of the most referenced sections during discovery — the AI uses it to confirm whether a client's workflow matches the standard or deviates from it.

### [Entity Name] Lifecycle

> **[AI Guide]** Replace with entity name. Add one subsection per entity with a lifecycle.

**States:** [list all valid states]
**Standard transitions:**
- [State A] → [State B]: triggered by [what]
- [State B] → [State C]: triggered by [what]

**Terminal states:** [states from which no further transition is possible]
**Rules:** [any constraints on transitions, e.g. "cannot move from X to Y if Z is true"]

---

## 5. Universal Business Rules

> **[AI Guide]** List rules that apply in every implementation of this domain without exception. These are non-negotiable — they are not confirmed with the client, and if the client contradicts them, flag it as a scope risk.
>
> Do not include rules that vary between clients — those belong in section 9 (Known Variations).
>
> Format: UBR-001, UBR-002, etc.

| ID | Rule | Applies To |
|----|------|-----------|
|    |      |           |

---

## 6. Standard Validations

> **[AI Guide]** List validations that are always enforced in this domain regardless of client. These do not need to be captured in the BRD.
>
> Include only validations that have a business reason — not technical constraints (e.g. max field length belongs in technical design, not here).
>
> Format: VAL-001, VAL-002, etc.

| ID | Validation | Applies To | Reason |
|----|-----------|-----------|--------|
|    |           |           |        |

---

## 7. Common Personas & Roles

> **[AI Guide]** List the standard personas and roles that appear in most implementations of this domain. These are starting points — confirm actual names, titles, and permission boundaries with the client during requirement discovery, not during domain knowledge generation.
>
> Do not assume these roles exist. Use them to prompt the right questions: "In your organisation, who is responsible for X?"

| Persona | Typical Responsibilities | Commonly Has Access To | Commonly Excluded From |
|---------|------------------------|----------------------|----------------------|
|         |                        |                      |                      |

---

## 8. Typical Integration Points

> **[AI Guide]** List the external systems and services that are commonly used in this domain. These are not guaranteed — confirm which apply during discovery.
>
> Use this section to prompt the right questions: "Do you use a payment gateway? Which one?" Not to assume the integration is needed.

| System Type | Common Examples | Purpose | Direction | Confirm With Client |
|------------|----------------|---------|-----------|---------------------|
|            |                |         |           |                     |

---

## 9. Regulatory Baseline (Country-Agnostic)

> **[AI Guide]** List compliance and regulatory obligations that apply to this domain regardless of country. These apply to every implementation and do not need to be confirmed with the client.
>
> Country-specific regulations (tax laws, e-invoicing mandates, data retention periods) belong in a country extension document, not here.
>
> If no universal baseline exists, write "No universal regulatory baseline — all compliance requirements are country-specific. See extension documents."

| Obligation | What It Requires | Applies To |
|-----------|-----------------|-----------|
|           |                 |           |

---

## 10. Known Variations

> **[AI Guide]** List the things that commonly differ between client implementations of this domain. These are the questions the AI must ask during discovery — they cannot be assumed from domain knowledge.
>
> For each variation, note the common options and the most common default (if any). The playbook uses this section to generate discovery questions.

| Topic | Common Options | Default (if any) | Must Ask |
|-------|---------------|-----------------|---------|
|       |               |                 | Yes |

---

## 11. Typical Scope Boundaries

> **[AI Guide]** Describe what is typically in scope for a first implementation of a system in this domain, and what is typically deferred to later phases. Use this to guide scoping conversations — not to constrain what the client can ask for.

### Typically In First Implementation
> **[AI Guide]** Bullet list of capabilities that almost every client needs in phase one.

### Typically Deferred to Later Phase
> **[AI Guide]** Bullet list of capabilities that are often wanted but rarely built first. Include a brief reason for each deferral (complexity, dependency, cost).

---

## 12. Document Verification

> **[AI Guide — Verification]**
> Run every check in this section after completing sections 1–11. The document is not production-ready until all items are confirmed.
> Work through each checklist in order. Record the outcome of each check. Do not mark a check as passed if you are uncertain — flag it for human review instead.

---

### 12.1 Internal Consistency

These checks can be run without any external sources. Fail fast — fix before moving on.

- [ ] Every term used in sections 3–6 is defined in section 2 (Domain Terminology). If a rule references a term that is not in section 2, add it or reword the rule.
- [ ] Every entity defined in section 3 that has meaningful states appears in section 4 (Lifecycle). If an entity has no lifecycle, confirm this is intentional and note it.
- [ ] Every entity lifecycle in section 4 has at least one clearly designated initial state (the state an entity enters when first created) and at least one terminal state (a state from which no further transition is possible). A lifecycle with no terminal state is either incomplete or intentionally perpetual — if perpetual, document why.
- [ ] No Universal Business Rule in section 5 contradicts a Standard Validation in section 6, or vice versa.
- [ ] If this is an extension document: no rule in sections 5–6 contradicts a rule in the base domain document. Extension documents add or narrow rules from the base — they do not override without explicit justification. (An extension document adds context-specific rules on top of a base document. Examples: `invoice-payment-pakistan.md` is a country extension of `invoice-payment-core.md`; `invoice-payment-school-fees.md` is a sector extension.)
- [ ] Section 10 (Known Variations) does not list any topic as a variation that is already fixed by a Universal Business Rule in section 5. If something is a UBR, it cannot also be a known variation.
- [ ] Section 11 (Typical Scope Boundaries) does not list anything as "typically in v1" that section 10 flags as a high-complexity variation requiring a special architecture decision.

---

### 12.2 Completeness

These checks confirm the document covers what it should.

- [ ] Every entity in section 3 appears in at least one business rule in section 5 or one validation in section 6. Entities with no rules attached are likely incomplete.
- [ ] Section 10 (Known Variations) covers all topics that commonly differ between client implementations. Cross-check against section 3 (entities with configurable attributes) and section 4 (lifecycles with configurable transitions).
- [ ] Section 11 has entries in both subsections (Typically In First Implementation and Typically Deferred). A section 11 with only one subsection is incomplete.
- [ ] If this is a country or domain extension: confirm section 9 (Regulatory Baseline) covers all obligations specific to this country or domain that are not already in the base document.
- [ ] Run at least 10 edge case scenarios against this document. For each scenario, locate the specific section and rule that resolves it. Any scenario that cannot be resolved without guessing = a gap. Record scenarios and outcomes in section 12.4.

---

### 12.3 Factual Accuracy Log

> **[AI Guide]** For every fact that could become outdated — regulatory thresholds, rates, provider capabilities, external timelines — record the source and verification date here. Facts without a source cannot be trusted.
>
> If you cannot verify a fact against an authoritative source, flag it as **Unverified** and do not treat the document as production-ready until it is resolved.

| Fact | Source | Verified By | Date Verified | Next Review |
|------|--------|-------------|--------------|-------------|
|      |        |             |              |             |

> **[AI Guide]** If this document contains no time-sensitive or regulation-dependent facts, write "None — no time-sensitive facts in this document." in the table body above.

**Common facts requiring verification in this domain:**
> **[AI Guide]** List the facts in this specific document that are time-sensitive or regulation-dependent. Examples: tax rates, regulatory thresholds, provider onboarding timelines, wallet limits, data retention periods. This list is domain-specific — populate it when filling in the template. If none apply, write "None."

---

### 12.4 Edge Case Scenario Log

> **[AI Guide]** Record the edge case scenarios tested against this document. For each scenario, note which section and rule resolved it, and the outcome. Scenarios that could not be resolved are gaps — add them to section 12.5 (Open Issues).

| Scenario | Resolved By (Section + Rule) | Outcome | Gap? |
|----------|------------------------------|---------|------|
|          |                              |         |      |

---

### 12.5 Open Issues

> **[AI Guide]** Record any item that failed a check above, any fact that could not be verified, and any edge case scenario that could not be resolved. Each open issue must be resolved before this document is treated as production-ready.
>
> If no issues exist, write "None."

| ID | Issue | Type | Owner | Status |
|----|-------|------|-------|--------|
|    |       | Consistency / Completeness / Factual / Scenario |  | Open / Resolved |

---

### 12.6 Mock Discovery Validation

> **[AI Guide]** This check requires the linked playbook to exist. Do not attempt it until the playbook is written.
>
> Once the playbook is available:
> 1. Load this document and the linked playbook.
> 2. Conduct a mock discovery session with a simulated client in this domain.
> 3. Review the resulting BRD draft and answer each question below.
> 4. Record failures in section 12.5 (Open Issues) and update this document accordingly.

- [ ] Did the AI ask questions about topics already covered in sections 3–6 of this document? If yes — the AI Guide instructions are unclear, or the content is ambiguous.
- [ ] Did the AI miss any questions it should have asked based on section 10 (Known Variations)? If yes — section 10 has gaps or the playbook does not reference it correctly.
- [ ] Did the resulting BRD restate domain defaults that should have been assumed? If yes — the AI Guide at the document level is not being followed.
- [ ] Did the resulting BRD miss requirements the client would obviously need? If yes — domain docs or playbook are incomplete.
- [ ] Were any facts in the BRD incorrect (wrong rate, wrong lifecycle state, wrong rule)? If yes — factual accuracy issue in section 12.3.

**Mock discovery status:** Not yet run — playbook not yet written.
**Planned after:** Playbook completion.

---

## 13. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-domain-knowledge` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/02-domain-discovery.md`'s own tests (the Section 5-vs-Section 10 universality test, the generalization filter, the confidence-score and Factual Accuracy Log reasoning, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 14 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this document, this section does not appear in the document at all; do not pre-create it empty.

## 14. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control above. Each row records one finding from Section 13's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this document, this section does not appear in the document at all; do not pre-create it empty.
