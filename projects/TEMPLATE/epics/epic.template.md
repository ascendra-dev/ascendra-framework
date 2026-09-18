# Epic Template

> **[AI Guide — Document Level]**
> Epics are authored in conversation with the Product Owner after the BRD is approved. Each epic represents a cohesive delivery unit — a set of BRD requirements that together deliver a complete, user-meaningful capability.
>
> **Authored by:** Product Owner + AI (via `/gen-epics`). Do not generate epics autonomously.
> **Used by:** `/gen-stories` to derive the story list for this epic.
>
> **What this document is not:** a sprint plan (that is `/gen-sprint-plan`), a technical design document, or a task list. If a sentence describes how something is built rather than what it delivers to a user, it belongs in the architecture document or story acceptance criteria — not here.
>
> **One epic, one layer.** Never mix Core and extension-layer (Ext:[extension-name]) requirements in the same epic. Extensions build on Core — each layer's epics are planned and implemented separately.
>
> **Epic size, as a rule of thumb:** an epic typically decomposes into roughly 4–10 stories. If a draft epic would clearly produce many more, propose a split at the `/gen-epics` Step 4 planning table rather than generating an oversized epic and splitting it later.
>
> **Section 7 (Stories) is always blank during epic authoring.** `/gen-stories` populates it after all stories for this epic are written. Do not pre-populate it.
>
> **Before generating stories from this epic:** run the Part 2 Verification checklist. An epic that fails any check produces incorrect or incomplete stories.
>
> **File location:** `projects/{PROJECT_CODE}/epics/EPIC-{NNN}-{slug}.md`
> e.g. `projects/ASCENDRA-PAY-001/epics/EPIC-001-invoice-management.md`

---

## Document Control

> **[AI Guide]** Add a row each time this document is revised. Version 1.0 is the initial draft.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |

---

## Part 1 — Template

---

**Epic ID:** EPIC-{3-digit} — e.g. EPIC-001
**Title:** [Short noun phrase describing the capability — e.g. "Payer Management"]
**Phase:** Phase 1 / Phase 2 / Phase 3
**Layer:** Core / Ext:[extension-name]
**Priority:** Must Have / Should Have / Nice to Have
**Status:** Draft / Approved / In Progress / Done

---

### 1. Goal

> **[AI Guide]** 1–2 sentences from the user's perspective. What does delivering this epic mean for an issuer admin or end user? Answer: "When this epic is done, [persona] can [capability]." Do not describe technical deliverables or list requirements.
>
> **Correct:** "When this epic is done, a Finance Manager can create, edit, and send invoices to any registered Payer without using a spreadsheet."
> **Incorrect:** "This epic implements the Invoice module with CRUD endpoints and PDF generation." — describes technical deliverables, not user capability

[Goal statement]

---

### 2. BRD Requirements Covered

> **[AI Guide]** List every BRD requirement this epic covers. Use the exact description from the BRD — do not paraphrase.
>
> A requirement belongs to exactly one epic. If a requirement spans two epics, assign it to the epic where its acceptance criteria would be written. The other epic names it in Section 4.2 (Out of Scope).

| REQ ID | Description (exact from BRD) | Priority | Walking Skeleton |
|--------|------------------------------|----------|-------------------|
| REQ-xxx | ... | Must Have | Yes / — |

> **[AI Guide — FW-048]** Carry the Walking Skeleton value straight from the BRD's Section 5 — never re-derive or re-judge it here. If any row in this table is `Yes`, this epic carries the Walking Skeleton and must be sequenced first in the Dependency Graph in `epics/index.md` (Section 5, below, still applies for the *reason* it's first — Walking Skeleton status is an additional, harder constraint on top of it, not a replacement for it).

---

### 3. Scope

#### 3.1 In Scope

> **[AI Guide]** Bullet list of capabilities this epic delivers, at feature level. Be specific enough that `/gen-stories` can derive a story title from each item without reading the full BRD. If an item cannot produce a story title, it is too vague.

- ...

#### 3.2 Out of Scope

> **[AI Guide]** Bullet list of capabilities that are NOT in this epic but that a developer might reasonably assume are included. For each item, name the epic that covers it (use EPIC-XXX) or state "Phase X / deferred."
>
> Never leave this section empty. If nothing is excluded, the epic boundaries are not tight enough.

- ...

---

### 4. Personas Involved

> **[AI Guide]** List only personas from BRD Section 2 that directly interact with capabilities in this epic. "Interact" means they trigger an action, receive an output, or their data is affected. Do not list personas who are merely aware of the feature.

| Persona | What they do in this epic |
|---------|--------------------------|
| [Persona from BRD Section 2] | [Specific action or capability] |

---

### 5. Dependencies

> **[AI Guide]** Epics that must be fully implemented and merged before this epic can begin. Use EPIC-XXX IDs. If this epic has no dependencies, write "None."

| EPIC ID | Dependency Reason |
|---------|------------------|
| EPIC-XXX | [Why it must be done first] |

---

### 6. Definition of Done

> **[AI Guide]** This is the epic-level gate. The Product Owner confirms the epic is complete when all of the following are true. Do not modify this list — it applies to every epic. Add epic-specific walkthrough steps in the last item.

- [ ] All stories in this epic are merged to the main branch and passed code review
- [ ] All story acceptance criteria have been verified by QA
- [ ] No open bugs exist against any story in this epic
- [ ] Audit trail entries are confirmed for all user-triggered actions in this epic
- [ ] Product Owner has walked through the capability end to end: [describe the PO walkthrough scenario specific to this epic]

---

### 7. Stories

> **[AI Guide]** Populated by `/gen-stories` after all stories for this epic are written. Leave blank during epic authoring.

| Story ID | Title | Size | Status |
|----------|-------|------|--------|
| | | | |

---

### 8. Notes

> **[AI Guide]** Optional. Include here every UBR (universal business rule), VAL (validation rule), BR (client-specific business rule), and SEC (security requirement) from the domain knowledge document and BRD that `/gen-stories` will need to write complete acceptance criteria for this epic's capabilities. Also include any architectural constraints or open questions the AI must know when generating stories. Omit entirely if none apply.

[Optional]

---

### 9. Density & Judgment Log

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-epics` time, and left out of the document entirely until that command has actually run once. This section holds every run's findings against `practitioner-guide/04-product-structuring-epics.md`'s own tests (the epic-cohesion test, the dependency-ordering test, the story-level interleaving criteria, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. Each invocation either logs a new dated block of findings (only when nothing from a prior run is still open) or resolves whatever is still open (amending that finding's own row in place with the Product Owner's outcome — Confirmed as-is / Revised / Acknowledged tradeoff — never a new row for the same finding, never edited again once resolved). The section header itself is reserved here at generation time — always present — but its table is not; if `/judgment-check` has not yet run against this epic, leave the placeholder pointer line below exactly as it is rather than inventing a table.

> **[AI Guide — numbering note]** Section 9 is per-epic, reserved for `/judgment-check` as shown here — unrelated to `epics/review-record.md`, which is a separate, project-wide file `/review-epics` maintains across all epics and does not use this numbering at all.

*Not yet run — see `/judgment-check` for what this checks and when to run it.*

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before this epic is approved and before story generation begins. An epic that fails any check produces incomplete or incorrect stories.

- [ ] Every REQ ID in Section 2 exists in the approved BRD and has not been modified since this epic was authored
- [ ] The In Scope items in Section 3.1 collectively cover all requirements listed in Section 2 — no requirement is left without a corresponding scope item
- [ ] Every item in Section 3.2 (Out of Scope) names the EPIC or Phase that covers it — nothing excluded without a home
- [ ] All personas in Section 4 appear in BRD Section 2 (Stakeholders & Personas) — no invented personas
- [ ] All EPIC IDs in Section 5 (Dependencies) reference files that exist in the epics/ directory
- [ ] The Layer field is consistent with all requirements in Section 2 — no Core requirement mixed into an Ext epic, and no Ext requirement mixed into a Core epic
- [ ] No implementation detail appears anywhere in Sections 1–5: no field names, API routes, database tables, technology choices, or UI layout decisions
