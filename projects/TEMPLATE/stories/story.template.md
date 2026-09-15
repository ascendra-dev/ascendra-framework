# User Story Template

> **[AI Guide — Document Level]**
> Stories are generated after epics are approved and after the architecture document is Locked. Each story covers one testable, independently deliverable slice of an epic's scope.
>
> **Before generating stories:** read the parent epic file and all its BRD requirements. Do not invent scope that is not in the epic's In Scope section.
>
> **Implementation-ready standard:** a story is ready when a developer can start coding it without asking a single follow-up question. Vague acceptance criteria are a quality failure, not a draft state — do not submit them for review.
>
> **What this document is not:** a task list, a technical specification, or a design document. If text describes database schema, API signatures, or UI layout decisions, it belongs in the architecture document or Notes (Section 7) — not in the acceptance criteria.
>
> **One story = one PR.** A story is not done until its PR is merged, code-reviewed, and acceptance criteria verified by QA.
>
> **Generating stories from an epic:** read the epic's In Scope bullet list and derive one story per discrete deliverable capability. Do not write a story that covers two In Scope items — split them. Do not write a story that is not traceable to an In Scope item.
>
> **Produced by:** `/gen-stories`, one epic at a time, after the epic is Approved and the architecture document is Locked. Reviewed by `/review-stories` before sprint lock.
>
> **File location:** `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/{story-id}.md`
> e.g. `projects/ASCENDRA-PAY-001/stories/EPIC-001/US-01-003.md`

---

## Document Control

> **[AI Guide]** Add a row each time this document is revised. Version 1.0 is the initial draft.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |

---

## Part 1 — Template

---

**Story ID:** US-[epic-number]-[sequence]

> **[AI Guide]** e.g. `US-04-003` for the 3rd story generated under EPIC-004. The epic number is the owning epic's own number, 2-digit zero-padded (EPIC-004 → `04`, EPIC-013 → `13`) — it is stable and known the moment a story is generated, unlike sprint membership, which isn't decided until `/gen-sprint-plan` runs and can change afterward (a change request, a re-split, a replanned sprint). Sequence resets to 001 at the start of each epic, not each sprint. Sprint assignment is tracked separately, as a column in the project's stories index (`projects/{PROJECT_CODE}/stories/index.md`) — never encoded in the Story ID itself, since doing so would force a rename every time sprint scope changes. The project code is never part of the Story ID — `projects/{PROJECT_CODE}/stories/...` already disambiguates which project a story belongs to.

**Epic:** EPIC-{3-digit} — [Epic Title]

**Title:** [Short verb-noun phrase — e.g. "Register Payer manually"]

> **[AI Guide]** Verb-noun format only. The verb describes what the user or system does. The noun is the object of that action. Examples: "Register Payer manually", "Generate invoice PDF", "Import Payers from CSV". Do not use passive voice. Do not include the persona in the title.

**Layer:** Core / Ext:[extension-name]
**Target:** API / Web / Worker / [combination, e.g. API+Web]

> **[AI Guide]** Which repo(s) this story's code lands in. Populated by `/gen-stories` from the Locked architecture — Section 3.1 (Project Structure) names the repos; the story's acceptance criteria determine which of them this story touches. Use only repo roles Section 3.1 actually declares (never `Worker` if no worker repo is confirmed). Join multiple targets with `+` in dependency order (e.g. `API+Web`). `/implement-story` reads this field to decide where to implement — it must never be left as the placeholder.

**Status:** Draft / Reviewed / Approved

---

### 1. Story Statement

> **[AI Guide]** One story statement. Three rules:
>
> 1. **One persona** — exactly one role in the "As a..." clause. Never "As a Finance Manager or Finance Officer." If two personas need the same capability, check whether the permission model actually distinguishes them — if not, pick the lower-privilege role.
> 2. **Specific action** — "I want to register a new Payer by entering their name, phone number, and email" is correct. "I want to manage Payers" is not — it is a feature area, not a story.
> 3. **User value in the benefit** — "so that I can bill them without re-entering their details on every invoice" is correct. "so that the system stores the record" is not — it describes a technical outcome, not a user benefit.
>
> The role must be a role defined in BRD Section 3 (User Roles). Do not invent roles.
>
> **Correct:** "As a Finance Manager, I want to register a new Payer by entering their name, phone number, and email, so that I can bill them without re-entering their details on every invoice."
> **Incorrect:** "As a Finance Manager, I want to manage Payer records." — "manage" is a feature area, not a specific action; benefit is missing

As a [role], I want [action], so that [benefit].

---

### 2. FR References

> **[AI Guide]** Comma-separated list of BRD requirement IDs this story satisfies. Format: REQ-007, REQ-008.
>
> Every story must reference at least one approved BRD requirement. A story with no FR reference is out of scope and must not enter a sprint.
>
> A single story can satisfy part of a requirement — it does not need to cover the entire requirement in one story. If one requirement produces multiple stories (e.g. REQ-009 produces a CSV upload story and a separate import log story), all those stories reference REQ-009.
>
> Do not reference requirements from other phases or other epics.

REQ-xxx[, REQ-xxx]

---

### 3. Acceptance Criteria

> **[AI Guide]** Numbered list of testable conditions. Each criterion must be independently verifiable — a QA engineer can write a test case for it without asking a follow-up question.
>
> **Cover:** the happy path, primary validation rules, primary error states, and any explicit constraint from the BRD requirement. Do not leave a BRD constraint unrepresented in AC.
>
> **Do not write AC that describes implementation:** "Uses a database transaction" or "calls the dedup service" are implementation details — they belong in Notes, not AC.
>
> **Count:** minimum 3, maximum 10. Fewer than 3 usually means the story is too small or the AC is too vague. More than 10 usually means the story should be split.
>
> **Format:** each criterion is a complete, present-tense statement of system behaviour. Start with the subject ("The system...", "The form...", "The import log..."), not with "Should" or "Must."

1. [Specific, testable condition]
2. [Specific, testable condition]
3. ...

---

### 4. Out of Scope

> **[AI Guide]** Bullet list of capabilities a developer would naturally implement next after completing this story — but that are explicitly excluded from this story's scope.
>
> **Never leave this section empty.** If nothing appears to be excluded, the story is too broad. Narrow the story further until there is something to exclude.
>
> For each item: name the story that covers it if a story ID has been assigned, or describe the capability if the story has not yet been written. Never say "to be handled later" without naming what handles it.

- [Excluded capability — Story ID or description of covering story]

---

### 5. Size

> **[AI Guide]** Select one value. Apply the scale consistently — size reflects effort for one developer, not calendar time.
>
> - **XS:** a single, contained change — a validation rule, a config flag, a status update
> - **S:** a focused backend endpoint or a simple UI component with no branching logic
> - **M:** a complete feature slice — backend + data layer, or a multi-state UI component
> - **L:** a complex feature slice — multiple endpoints, or a UI flow with branching. Flag L stories for PO review before finalising — consider splitting
> - **XL:** must be split before adding to a sprint. No exceptions
>
> When in doubt between two sizes, choose the larger one.

XS / S / M / L / XL

---

### 6. Dependencies

> **[AI Guide]** Dependencies are managed in two passes.
>
> **Pass 1 — Story writing:** describe each dependency as a plain English concept. Story IDs do not yet exist for stories not yet written — do not invent them.
> Examples: "Payer schema table must exist", "Deduplication key must be configurable", "PSP credentials must be available in environment config."
>
> **Pass 2 — Sprint ordering (batch):** after all stories in the sprint are written and IDs assigned, run the ordering pass. Replace every plain-English description with a resolved story ID reference.
> Examples: US-01-001 (Payer schema), US-01-003 (Deduplication key configuration).
>
> The ordering pass also detects circular dependencies and unresolvable conflicts — these are escalation triggers.
>
> **Infrastructure dependencies** (env vars, external API credentials, third-party accounts) are listed in plain text in both passes and are never replaced with story IDs.
>
> Write "None" only if this story has no prerequisite — it can be the very first story merged.

- [Pass 1: plain English concept | Pass 2: resolved story ID]

---

### 7. Notes

> **[AI Guide]** Optional. Technical constraints, architectural context, or implementation hints the implementer needs that are not already expressed in the acceptance criteria.
>
> Include a note if: there is a non-obvious constraint the developer must know (e.g. idempotency requirement, specific library to use, a known edge case in the data); or if the AC requires context to interpret correctly.
>
> Do not include: implementation opinions without a specific reason, content that duplicates AC, or general best-practice reminders the developer already knows.
>
> Omit this section entirely if there is nothing to say.

---

### 8. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-stories` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/07-sprint-planning.md`'s own tests (the persona tie-break rule, the story sizing/AC-count proxy and its Pattern Novelty / Context Footprint factors, the wave-vs-epic boundary test as it bears on this story's own Dependencies, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 9 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this story, this section does not appear at all; do not pre-create it empty.

### 9. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control. Each row records one finding from Section 8's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this story, this section does not appear at all; do not pre-create it empty.

> **[AI Guide — numbering note]** Sections 8 and 9 are per-story, reserved for `/judgment-check` as shown here — unrelated to `stories/review-record.md`, which is a separate, project-wide file `/review-stories` maintains across all stories and does not use this numbering at all, so there is no numbering collision between the two mechanisms.

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below on every story before it enters a sprint plan. A story that fails any check must be corrected before sprint lock — not after.

- [ ] Story ID follows the correct format and matches the owning epic's number
- [ ] Epic ID is populated and references a file that exists in the project's epics/ directory
- [ ] Layer matches the parent epic's layer
- [ ] Target is populated and names only repo roles declared in architecture Section 3.1 (Project Structure)
- [ ] Story Statement has exactly one persona drawn from BRD Section 3 (User Roles)
- [ ] The action in the Story Statement is a specific capability, not a feature area description
- [ ] FR References trace to requirements in the approved BRD — no story references a requirement that does not exist or was not approved
- [ ] Every acceptance criterion is independently testable — a QA engineer can write a test case without a follow-up question
- [ ] Acceptance criteria cover the happy path, primary validation rules, and primary error states from the BRD requirement
- [ ] No acceptance criterion describes implementation detail (no database calls, no library names, no API signatures)
- [ ] Out of Scope is populated — never empty
- [ ] Every Out of Scope item names the story or concept that covers it — no "to be handled later" without a named owner
- [ ] Size is not XL — if XL, the story is split before entering the sprint
- [ ] Dependencies are present (Pass 1 or Pass 2 format) — never blank
- [ ] The story is independently deliverable — it can be merged without another in-progress story landing at the same time

