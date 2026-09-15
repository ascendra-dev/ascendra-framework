# Business Requirements Document

> **[AI Guide — Document Level]**
> This template is populated during or after a client discovery session.
> Two sources feed this document:
> - **Domain knowledge** — facts already known from the relevant domain doc(s). Do not ask the client about these; confirm overrides only.
> - **Client discovery** — answers the client gave during the session. These are always explicitly stated.
>
> Every requirement must be tagged with its source: `[Client-Stated]`, `[Domain-Default]`, or `[Assumed]`.
> Do not restate domain defaults unless the client has overridden them.
> Do not include implementation details, UI decisions, or engineering choices.

---

## Document Control

> **[AI Guide]** Add a row each time this document is revised. Version 1.0 is the initial draft produced from the discovery session.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | Ascendra AI | Initial draft from discovery session |

---

## 1. Project Overview

### 1.1 Project Name
> **[AI Guide]** The name of the project as agreed with the client. Use the format the client used.

### 1.2 Problem Statement
> **[AI Guide]** 2–4 sentences. What problem does the client have today? What is the cost or consequence of that problem? What does success look like when this system exists? Write from the client's perspective, not a technical one.

### 1.3 Business Goals
> **[AI Guide]** Bullet list. Specific, measurable outcomes the client expects. If the client gave vague goals, sharpen them into something testable. Each goal should answer: how will we know this project succeeded?
>
> **Correct:** "Reduce the time finance officers spend on monthly reconciliation from 4 hours to under 30 minutes."
> **Incorrect:** "Improve the invoicing process." — not measurable; no baseline; cannot be verified at UAT

### 1.4 Scope

**In Scope**
> **[AI Guide]** Bullet list of capabilities that will be built. Stay at feature-area level — not individual screens or fields. These should map directly to the epics that will be created from this BRD.

**Out of Scope**
> **[AI Guide]** Bullet list of things explicitly excluded from this project. Include anything the client mentioned wanting but agreed to defer, and any obvious adjacencies that are NOT included. This is as important as in-scope.

### 1.5 Assumptions & Constraints
> **[AI Guide]** Bullet list. Assumptions are things treated as true that have not been verified. Constraints are fixed boundaries (budget, deadline, regulatory, technical) the solution must operate within. If none were stated, leave this section with "None identified."

### 1.6 Designed for Extension
> **[AI Guide]** Fill this section if any future extension dimensions were identified during discovery or are reasonably anticipated. Leave it as "None identified" if this project is fully self-contained with no known future variants.
>
> An extension dimension is an axis along which this system may need to be adapted — for a different market, sector, regulatory context, or capability — in a future project. Naming these dimensions here:
> - Signals to the architecture team where to design seams and abstraction points
> - Prevents over-engineering the base: we design for the extension, we do not build it
> - Feeds directly into the architecture document's Extension Points section
>
> Do not design the extension here. Name the dimension and what the base must provide to support it.

| Dimension | What Varies | Anticipated By | Base Must Provide |
|-----------|-------------|---------------|-------------------|
| e.g. Market — Pakistan | FBR e-invoicing, IRIS submission | Planned expansion | Invoice model extensible for regulatory reference fields |

---

## 2. Stakeholders & Personas

> **[AI Guide]** List the people who interact with the system. For each persona include:
> - **Name** — what you will call this role throughout the document
> - **Who they are** — their actual job or relationship to the system
> - **Primary goal** — what they are trying to accomplish using this system
>
> Keep it to real personas with distinct goals. Do not create personas for every minor variation in a role.

| Persona | Who They Are | Primary Goal |
|---------|-------------|--------------|
|         |             |              |

---

## 3. User Roles

> **[AI Guide]** Define what each role is permitted and not permitted to do in the system. This is distinct from personas — a persona describes who a person is and what they want; a role defines the permission boundary the system enforces.
>
> Include a role if it has a distinct permission boundary from other roles. A developer must be able to make an authorisation decision based on this section alone.
>
> For each role capture what they can do, what they explicitly cannot do, and whose data they can see.
>
> Format: ROL-001, ROL-002, etc.

### ROL-001: [Role Name]
> **[AI Guide]** Replace with role name. Add one subsection per role.

**Can:** ...
**Cannot:** ...
**Data visibility:** ...

---

## 4. User Journeys

> **[AI Guide]** Describe the end-to-end flow each primary persona takes through the system to achieve their goal. One journey per persona. Keep it at step level — not screen or field level.
>
> A user journey belongs here if:
> - It spans more than one feature area
> - It shows how the system hands off between personas (e.g. one person submits, another approves)
> - It reveals sequencing or dependency that functional requirements alone would not show
>
> Format each journey as a numbered step sequence. Note where the system acts vs where the user acts. Note any decision points or branches (e.g. approval granted vs rejected).
>
> Do not describe UI interactions. Do not describe API calls. Stay at the business action level.

### 4.1 [Persona Name] — [Journey Name]
> **[AI Guide]** Replace heading with persona and journey name (e.g. "Finance Manager — Submit and Track an Invoice"). Add one subsection per primary journey.

1. ...
2. ...
3. ...

---

## 5. Functional Requirements

> **[AI Guide]** Organise requirements by feature area (these will become epics). Within each feature area, list discrete requirements — one capability per row.
>
> A requirement belongs here if:
> - A business stakeholder would make a decision about it
> - Getting it wrong would cause a scope rework, not just a code fix
> - It has a business rule or constraint attached
>
> A requirement does NOT belong here if:
> - It is a domain default that the client has not overridden
> - It is an implementation or UI detail
> - It would be identical for any client in this domain
>
> Format per requirement:
> - **ID** — REQ-001, REQ-002, etc. (sequential across the whole document)
> - **Description** — one sentence, plain English, what the system must do
> - **Priority** — Must Have / Should Have / Nice to Have
> - **Source** — [Client-Stated] | [Domain-Default] | [Assumed]
> - **Layer** — `Core` | `Ext:PK` | `Ext:School` | `Ext:[code]`
>
> **How to assign Layer**: A requirement is `Core` if it applies regardless of country or sector. Apply the membership test: *"Would a UK retailer need this?"* If yes → `Core`. If the requirement exists only because the client is in Pakistan → `Ext:PK`. If it exists only because the client is a school → `Ext:School`. If unsure, ask the test.
>
> Layer determines implementation order: all `Core` requirements must be implemented and tested before any `Ext` requirement that depends on them. `Ext` requirements within the same layer (e.g. all `Ext:School`) can be built in parallel with `Ext:PK` — they do not depend on each other.

### 5.1 [Feature Area Name]
> **[AI Guide]** Replace this heading with the feature area name (e.g. "Invoice Creation", "Payment Processing"). Add as many feature area sections as needed.

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
|    |             |          |        |       |

### 5.2 [Feature Area Name]

| ID | Description | Priority | Source | Layer |
|----|-------------|----------|--------|-------|
|    |             |          |        |       |

### 5.21 Future Capabilities

> **[AI Guide]** Section 5.21 is a fixed, reserved subsection number — unlike 5.1, 5.2, 5.3... above (assigned dynamically, one per real feature area), 5.21 is always 5.21 regardless of how many feature-area subsections this project actually has. The number is set deliberately high on the assumption a BRD rarely reaches 21 real feature areas. If a project's own feature areas ever do reach 5.21, move this subsection to the next unused number (e.g. 5.22) and note the renumbering in Document Control.
>
> This subsection holds capabilities the client already discussed and wants eventually — just not in this phase. Use it only when the answer to this test is yes: *can you write one factual sentence describing exactly how it will behave, backed by something already in the domain knowledge document or this discovery session?* If yes, it belongs here with a real REQ-ID. If the honest answer requires guessing, it is not ready for this subsection — it belongs in Section 15 (Parking Lot) instead, undiscovered and untagged, until a discovery pass gives it the same grounding.
>
> Because these ARE already-discovered, they carry the same shape as a real requirement — not the low-ceremony form of a Parking Lot row. Format per entry, using the same fields as Section 5 plus a Title for scanning:
> - **ID** — REQ-XXX, drawn from the same sequential numbering as every other requirement in this document. Do not run a separate numbering track for this subsection.
> - **Title** — a short label for scanning during epic review (e.g. "Accounting System Export").
> - **Description** — one factual sentence, exactly as Section 5 requires: what the system must do, backed by something already in the domain doc or this discovery session — not a guess.
> - **Priority** — Must Have / Should Have / Nice to Have. The priority it will carry once built, not "low" because it's deferred.
> - **Source** — `[Client-Stated]` | `[Domain-Default]` | `[Assumed]`, same as Section 5.
> - **Layer** — `Core` | `Ext:PK` | `Ext:School` | `Ext:[code]`, same membership test as Section 5.
>
> **Promotion rule:** when the client confirms this capability is now in scope — a new phase kicks off, a change request lands, or the need resurfaces during epic review — cut the row from this table and paste it into a real Section 5.N feature-area subsection (create one if none fits), unchanged unless the underlying facts have changed since it was written. The REQ-ID travels with it; it is now a live requirement like any other. If the client instead confirms the capability will never be built, delete the row — it never became a live requirement, so nothing downstream references its ID. Because the domain/BRD grounding already exists, promoting a Section 5.21 item does not require a fresh discovery pass first, unlike promoting a Section 15 (Parking Lot) entry.
>
> Typically empty at initial BRD generation ("None identified"). Like Section 15, entries usually accumulate later — during epic review, story writing, or a later phase kickoff — not during discovery-driven authoring.

| ID | Title | Description | Priority | Source | Layer |
|----|-------|-------------|----------|--------|-------|
|    |       |             |          |        |       |

---

## 6. Business Rules

> **[AI Guide]** List rules that govern system behaviour. Only include rules that are:
> - Client-specific (this client's rules, not universal domain rules)
> - Non-obvious (a developer would not infer them from context)
> - Decision-driving (they affect how a feature works, not just a label or format)
>
> Do NOT restate rules already captured in the domain knowledge document unless the client has overridden them.
>
> Format: BR-001, BR-002, etc.

| ID | Rule | Applies To | Source |
|----|------|-----------|--------|
|    |      |           |        |

---

## 7. Data Requirements

> **[AI Guide]** Describe the key data entities at a conceptual level only. No schema, no field types, no indexes.
>
> Include an entity if:
> - It is central to the system's purpose
> - The client has specific attributes or relationships that differ from the domain standard
>
> For each entity describe:
> - What it represents
> - Its key attributes (business-meaningful ones only)
> - Its relationships to other entities
>
> If the entity matches the domain standard exactly, note that and move on — do not restate it.

### [Entity Name]
> **[AI Guide]** Replace with the entity name. Add one subsection per key entity.

**What it is:** ...
**Key attributes:** ...
**Relationships:** ...
**Deviates from domain standard:** Yes / No — [explain if yes]

---

## 8. Integration Requirements

> **[AI Guide]** List every external system, service, or platform this solution must connect to. This is distinct from data requirements — an integration is a live connection to something outside the system boundary.
>
> Include an integration if:
> - The system must send data to or receive data from an external party
> - A third-party service is required for the solution to function (e.g. payment gateway, email provider, tax API)
>
> For each integration capture:
> - **What it is** — name and purpose of the external system
> - **Direction** — Inbound / Outbound / Bidirectional
> - **Trigger** — what causes this integration to fire
> - **Data exchanged** — at business level only (e.g. "invoice amount and recipient details"), not field names
> - **Owner** — who is responsible for the external system (client, third party, us)
>
> If no integrations were discussed, write "None identified — to be confirmed during technical design."

### [Integration Name]
> **[AI Guide]** Replace with integration name (e.g. "Stripe — Payment Processing"). Add one subsection per integration.

**What it is:** ...
**Direction:** ...
**Trigger:** ...
**Data exchanged:** ...
**Owner:** ...

---

## 9. Security Requirements

> **[AI Guide]** Capture security requirements as discrete, testable statements. Security is treated as a first-class section — not a subcategory of NFRs — because missed security requirements create liability, not just rework.
>
> Cover: authentication, authorisation, data protection, audit logging, and compliance obligations.
>
> Include a security requirement if:
> - It defines who can or cannot access something
> - It defines how sensitive data must be handled or protected
> - It is mandated by regulation or the client's own compliance obligations
>
> Do not include implementation detail (e.g. which encryption library to use). Stay at the control level.
>
> Format: SEC-001, SEC-002, etc.

| ID | Requirement | Priority | Source |
|----|-------------|----------|--------|
|    |             |          |        |

---

## 10. Non-Functional Requirements

> **[AI Guide]** Capture only constraints the client has explicitly stated or that are imposed by their regulatory or operational context. Do not invent NFRs. If none were discussed, write "None stated — to be defined in technical design."

| Category | Requirement | Source |
|----------|-------------|--------|
| Performance |  |  |
| Availability |  |  |
| Scalability |  |  |
| Browser / Device support |  |  |
| Data Retention |  |  |

---

## 11. Dependencies

> **[AI Guide]** List everything this project depends on that is owned by someone outside the delivery team. This is distinct from constraints (fixed limits) and assumptions (things treated as true).
>
> A dependency belongs here if:
> - It is required before development or testing can proceed
> - It is owned by the client, a third party, or another team
> - A delay would block or materially impact delivery
>
> If none exist, write "None identified."

| ID | Dependency | Type | Owner | Required By | Risk if Delayed |
|----|-----------|------|-------|-------------|----------------|
|    |           |      |       |             |                |

---

## 12. Open Questions

> **[AI Guide]** Anything that came up during discovery but was not resolved. Each item needs an owner and a due date before this BRD can be signed off. If nothing is open, write "None."

| ID | Question | Owner | Due |
|----|----------|-------|-----|
|    |          |       |     |

---

## 13. Pre-Approval Verification

> **[AI Guide — Verification]**
> Run every check below before marking this BRD ready for human review. This is a quality gate — not a formality.
> The BRD must pass all checks before being presented to the Product Owner for approval.
> Record any failure in section 13.5 (Open Issues). Do not proceed to section 14 (Approval) until all issues are resolved or explicitly accepted.
>
> This section is an internal quality record. It is retained in the document — not removed before client presentation.

---

### 13.1 Content Integrity

Confirms the right content is in the right sections and nothing has been misplaced.

- [ ] Every requirement in section 5 has all four fields: ID, description, priority, and source tag. No requirement is missing any of these.
- [ ] No requirement tagged `[Domain-Default]` restates a universal domain fact without a note explaining what the client specifically confirmed or overrode. Domain defaults that were not overridden should not appear in section 5 at all.
- [ ] No requirement tagged `[Assumed]` covers something that should have been confirmed with the client during discovery. If unsure, flag as an open question in section 12.
- [ ] Every business rule in section 6 is client-specific or non-obvious. No rule restates a fact already covered in the domain knowledge document.
- [ ] Section 1.4 (Scope > Out of Scope) is populated. A BRD with no explicit exclusions is incomplete — something has always been excluded, even if the client didn't say so.
- [ ] Every item in Section 1.4 (Out of Scope) is either explicitly a permanent exclusion, or has a corresponding REQ-ID in Section 5 tagged `[Client-Stated — Phase 2]` (or equivalent). A deferred capability with no REQ-ID cannot later extend an epic without a full BRD change — check this now, while it is cheap to fix, not after epics exist.
- [ ] No implementation detail appears anywhere in sections 3–9: no field types, no API endpoints, no database design, no UI decisions, no technology choices.
- [ ] Every entry in Section 5.21 (Future Capabilities) has all five fields a real requirement has (ID, Title, Description, Priority, Source, Layer) and real domain/BRD grounding for its Description — no entry sits here on a guess. If the grounding is not real, move it to Section 15 (Parking Lot) instead.
- [ ] Every entry in Section 15 (Parking Lot) has a source/context and an explicit Promotion Path note — no entry is silently treated as if it were already scoped, discovered, or REQ-worthy.

---

### 13.2 Cross-Section Consistency

Confirms the sections are coherent with each other.

- [ ] Every persona listed in section 2 (Stakeholders) appears in at least one user journey in section 4. A persona with no journey is either redundant or the journey is missing.
- [ ] Every user journey step in section 4 maps to at least one functional requirement in section 5. A journey step with no backing requirement means scope has not been captured.
- [ ] Every functional requirement in section 5 is reachable from at least one user journey in section 4. A requirement with no journey is either in the wrong section or scope has been added without a workflow context.
- [ ] Every integration listed in section 8 has at least one corresponding security requirement in section 9. An integration with no security requirement is an oversight — all external connections have security implications.
- [ ] Every integration listed in section 8 has a corresponding dependency entry in section 11 if the integration depends on an external party providing credentials, access, or approval.
- [ ] Every user role defined in section 3 is referenced in at least one functional requirement in section 5. A role with no requirements is either not in scope or the requirements are missing.

---

### 13.3 Downstream Readiness

Confirms the BRD is specific enough to produce user stories from without additional discovery.

- [ ] Every requirement in section 5 is specific enough that a developer could write a user story from it without needing to ask follow-up questions. Vague requirements must be rewritten or moved to section 12 (Open Questions).
- [ ] No requirement uses language that defers definition: "standard X", "appropriate Y", "the system should handle Z", "users can manage W". Each of these is a placeholder, not a requirement.
- [ ] Every business rule in section 6 references the requirement(s) in section 5 it governs. A floating business rule with no functional requirement home cannot be traced to stories.
- [ ] Data requirements in section 7 identify at least the primary entities the system must store. If entities are implied by functional requirements but not listed in section 7, add them.
- [ ] Non-functional requirements in section 10 are stated as measurable constraints, not aspirational statements. "The system must be fast" is not an NFR. "All pages must load within 2 seconds at 50 concurrent users" is.
- [ ] Section 1.6 (Designed for Extension) is populated if any future extension dimension was identified during discovery or is reasonably anticipated — the architecture team uses this section to design extension seams. If no extension dimensions exist, the section explicitly states "None identified." A blank Section 1.6 is not acceptable.

---

### 13.4 Approval Readiness

Confirms the BRD is in a state where a Product Owner can responsibly sign it.

- [ ] Every open question in section 12 has an owner and a due date assigned. An open question without an owner will never be resolved.
- [ ] Any open question in section 12 that would materially change scope if answered one way vs the other is explicitly flagged as a **scope blocker**. These must be resolved before approval — not after.
- [ ] Every dependency in section 11 has an owner identified. Dependencies without owners are unmanaged risks.
- [ ] The BRD version in the Document Control table matches the version in section 14 (Approval). A mismatch means the wrong version is being approved.
- [ ] The Product Owner has reviewed the BRD before it is presented to the client for sign-off. AI-drafted BRDs are not presented to the client without a human review step.

---

### 13.5 Verification Issues

> **[AI Guide]** Record every item that failed a check above. The BRD is not ready for approval until every issue is either resolved or explicitly accepted with a written rationale. If no issues exist, write "None."

| ID | Failed Check | Description | Resolution | Status |
|----|-------------|-------------|-----------|--------|
|    |             |             |           | Open / Accepted / Resolved |

---

## 14. Approval

> **[AI Guide]** Populate the project name and date. Leave signatory fields for humans to complete. Do not fabricate names or dates.

**Project:** [Project Name]
**BRD Version:** 1.0
**Prepared by:** Ascendra AI — [Date]
**Status:** Draft / Under Review / Approved

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner |  |  |  |
| Client Representative |  |  |  |

---

## 15. Parking Lot

> **[AI Guide]** This section holds ideas that surfaced outside the discovery process — during epic review, story writing, a client aside, or ongoing delivery — that are NOT yet requirements. Two things distinguish a Parking Lot entry from a Section 1.4/5.21 deferred capability: (1) it has no backing in the domain knowledge document — no entity, lifecycle, or rule models it yet, and (2) no discovery session (client or PO) has actually explored it. Adding an entry here is informal and low-ceremony — do not tag it `[Client-Stated]` or `[Domain-Default]`, do not give it a Priority or Layer, and do not treat it as if it were scoped. If the idea is already well-understood and just deprioritized (i.e. the domain model or client conversation already backs it), it belongs in Section 5.21 (Future Capabilities) with a real REQ-ID instead — Parking Lot is only for genuinely undiscovered ideas.
>
> **Promotion rule:** a Parking Lot entry can never become a REQ-ID via `/assess-change` alone. It must first go through `/run-domain-discovery` and/or `/run-brd-discovery` (or an equivalent documented live discussion) so the idea gets the same modeling rigor every other requirement received. Once discovered, it is promoted into Section 5 (or Section 5.21 if still deferred) with a real REQ-ID, and its Parking Lot row is marked `Promoted → REQ-XXX`. If nothing to record, write "None."

| ID | Idea | Surfaced (Date / Context) | Promotion Path | Status |
|----|------|---------------------------|-----------------|--------|
|    |      |                           | Requires domain + BRD discovery before it can become a REQ | Parked / Promoted |

---

## 16. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-brd` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/03-brd-discovery-and-scope-lock.md`'s own tests (REQ density, the Scope/Priority/Layer distinction, the Section 5.21-vs-15 test, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 17 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this BRD, this section does not appear in the document at all; do not pre-create it empty.

## 17. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control. Each row records one finding from Section 16's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this BRD, this section does not appear in the document at all; do not pre-create it empty.

> **[AI Guide — reserved numbering]** Sections 16 and 17 are reserved for `/judgment-check` as shown here. `/review-brd` creates its own **Section 18 (Review Record)** at the end of the document on first run — never Section 16, to avoid colliding with these two. If `/judgment-check` has not run yet when `/review-brd` creates Section 18, Sections 16–17 simply don't exist yet; Section 18 still follows immediately after Section 15 (Parking Lot) in that case.
