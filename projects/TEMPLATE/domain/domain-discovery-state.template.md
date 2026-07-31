# Domain Discovery State — [Domain Name]

> **[AI Guide — Document Level]**
> This file is written by the AI at the end of every domain discovery session where the domain knowledge document is not yet generated.
> It is the sole source of continuity between sessions and the primary input to `/gen-domain-knowledge`.
>
> **When writing this file (`/run-domain-discovery`):**
> - Distil what the PO stated into each section. Do not write a transcript — write what `/gen-domain-knowledge` needs to generate accurate content.
> - On resume: load this file and the domain playbook. Follow the Resume Instructions at the bottom.
>
> **When reading this file (`/gen-domain-knowledge`):**
> - Section 5 (AI Knowledge Corrections) takes precedence over your training knowledge. Apply every correction before generating any content.
> - Section 3 (Domain Confidence Score) gates generation. If below 70%, warn the PO before proceeding.
>
> Do not treat this file as a conversation transcript. It is a distilled record of what the PO stated about the domain.

---

## 1. Session Context

| Field | Value |
|-------|-------|
| Project Code | |
| Domain Name | |
| Domain Playbook Used | `projects/{PROJECT_CODE}/domain/{slug}-domain-playbook.md` |
| Target Output | `projects/{PROJECT_CODE}/domain/{slug}-core.md` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Covered |
|-----------|------|-------------|-----------------|
| 1 | | | |

---

## 3. Domain Confidence Score

> **[AI Guide]** After each session, estimate what percentage of the domain knowledge document can be generated with confidence from this state file. A score below 70% means critical sections are still missing — schedule another session before generating.

**Current score:** [N]%

**What would raise it:**
- [List the sections still incomplete or uncertain]

---

## 4. Playbook Progress

> **[AI Guide]** Mirror the sections of the domain playbook. Mark each as covered or pending.
>
> For every covered section: write a one-paragraph distillation of what the PO stated. This becomes the primary input to `/gen-domain-knowledge`.
> For pending sections: leave blank.

### Section 4 — Domain Scope
**Status:** Covered / Pending

**PO stated:**
[What the PO said about what this domain covers and does not cover.]

---

### Section 5 — Core Entities
**Status:** Covered / Pending

**PO stated:**
[Each entity the PO confirmed, corrected, or added. For each: name, definition, standard attributes, relationships, notes.]

---

### Section 6 — Lifecycle & Status Model
**Status:** Covered / Pending

**PO stated:**
[For each entity with a lifecycle: states, transitions, triggers, terminal states, transition rules.]

---

### Section 7 — Universal Business Rules
**Status:** Covered / Pending

**PO stated:**
[Each universal business rule. Mark any where the PO's answer corrected AI knowledge — note both the standard and the correction.]

---

### Section 8 — Standard Validations
**Status:** Covered / Pending

**PO stated:**
[Each validation and the business reason behind it.]

---

### Section 9 — Common Personas & Roles
**Status:** Covered / Pending

**PO stated:**
[Each role, its typical responsibilities, access, and exclusions.]

---

### Section 10 — Typical Integration Points
**Status:** Covered / Pending

**PO stated:**
[Each integration type, purpose, direction, and whether it is always needed or varies.]

---

### Section 11 — Regulatory Baseline
**Status:** Covered / Pending

**PO stated:**
[Country-agnostic regulatory obligations. Note anything the PO flagged as country-specific — those belong in an extension document.]

---

### Section 12 — Known Variations
**Status:** Covered / Pending

**PO stated:**
[Each variation topic, common options, most common default, and whether unusual options would be a red flag.]

---

### Section 13 — Typical Scope Boundaries
**Status:** Covered / Pending

**PO stated:**
[What is typically in scope for v1 and what is typically deferred, with reasons.]

---

## 5. AI Knowledge Corrections

> **[AI Guide]** Record every instance where the PO's answer differed from AI training knowledge. These are the most important entries — they prevent `/gen-domain-knowledge` from generating incorrect content.

| Topic | AI Assumption | PO Correction | Confidence |
|-------|--------------|---------------|------------|
| | | | High / Medium — verify |

---

## 6. Open Issues

> **[AI Guide]** List anything that could not be resolved in this session. Each issue must be resolved before generating the domain knowledge document or it must be flagged as a gap in Section 12.5 of the generated document.
>
> If none, write "None."

| # | Issue | Section | Owner | Status |
|---|-------|---------|-------|--------|
| DI-001 | | | | Open / Resolved |

---

## 7. Resume Instructions

> **[AI Guide]** This section is the first thing to read on resume.

**Next session starts at:** [Section number and name from the domain playbook]

**Before resuming:**
- [ ] Load: domain playbook at `projects/{PROJECT_CODE}/domain/{slug}-domain-playbook.md`
- [ ] Load: this file (already done if you are reading this)
- [ ] Review section 4 above to rebuild context on what the PO has already stated
- [ ] Review section 5 (AI Knowledge Corrections) — these override AI defaults

**Sections still pending (do not skip):**
- [List pending sections from section 4]

**Open issues to revisit:**
- [Any DI items marked Open in section 6]

---

## 8. Pre-Generation Verification

> **[AI Guide — Verification]** Run these checks before passing this file to `/gen-domain-knowledge`. Do not generate if any check fails — resolve the issue or schedule another discovery session.

- [ ] Section 1 (Session Context): Project Code, Domain Name, and Target Output path are all filled
- [ ] Section 3 (Domain Confidence Score): Score is ≥ 70%. If below 70%, warn the PO and recommend another session before generating
- [ ] Section 4 (Playbook Progress): Every section marked Covered has a substantive "PO stated:" paragraph — not a placeholder or empty entry
- [ ] Section 5 (AI Knowledge Corrections): Reviewed — every instance where the PO's answer differed from AI training knowledge has been recorded here
- [ ] Section 6 (Open Issues): All items are Resolved, or each Open item is explicitly acknowledged and flagged to carry as a gap into the generated document
- [ ] Section 7 (Resume Instructions): Either filled with the next session start point, or explicitly updated to state "Generation ready — no further sessions needed"
