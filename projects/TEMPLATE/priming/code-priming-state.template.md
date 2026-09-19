# Code Priming State — [Project Name]

> **[AI Guide — Document Level]**
> This file is written and read by `/run-priming-code-session` itself, across however many sessions a codebase takes to walk — it is not read by any other command. It is the resumability ledger for a code-priming walk: which units (entities, or modules/layers where entities aren't cleanly discoverable — see the Graceful Degradation note below) have been Mapped, Triaged, and Extracted, so a session interrupted partway through a large codebase can resume from exactly where it stopped instead of re-walking everything or holding the whole history in conversation context.
>
> **It is not `priming-package.md`, and does not replace it.** `conventions/priming-command-conventions.md` PC-002 requires every priming-producing command to merge into the one project-level `priming-package.md` — this file does not compete with that rule. It is operational bookkeeping for this command's own multi-session resumability, the same way `domain-discovery-state.md` and `brd-discovery-state.md` are working state distinct from the `domain.md`/`brd.md` deliverables they eventually produce. Business-facing facts extracted during a session still get written into `priming-package.md` Sections 4-6 (or Section 10, for the narrow code-structure-only case) — never only into this file.
>
> **File location:** `projects/{PROJECT_CODE}/priming/code-priming-state.md`, alongside `priming-package.md` in the same folder.
>
> **When used by `/run-priming-code-session`:** written and updated after each unit's Map/Triage/Extract pass — the only command that writes it. Read again at the start of any resumed session (Section 8, Resume Instructions) to determine exactly where the walk left off. No other command reads this file.
>
> **Chunking model this file tracks:** entity-centric where possible, not directory-centric — real codebases don't reliably have clean module boundaries, and directory-based chunking would scatter one entity's cross-layer signals (schema/validation/tests) across passes. Three passes per unit: **Map** (structural only — no file contents read), **Triage** (classify domain-likely vs. infrastructure-likely, light reads only), **Extract** (domain-likely units only, full signal cluster via targeted search). This command must never read the full repo — every pass is structural-navigation- or targeted-search-driven.
>
> **Graceful degradation — entity-level chunking is preferred, not assumed.** It's directly achievable wherever a schema/ORM layer exists (an annotated class, a models file, a schema definition — in any language or framework). Some codebases don't have one: raw SQL scattered through service/controller code, a thin or dynamically-typed schema, or genuinely no discoverable entity boundary. For those, Section 4 (Repository Map) stops being just a hint and becomes the required first-level scaffold — group by module/layer first, then attempt entity inference *within* each group, including inferring candidate entities from table names appearing in raw query strings where that's the only signal available. Where entity boundaries still can't be found with confidence for some part of the codebase, that part is walked at module/layer granularity instead — recorded as such in Section 5's `Granularity` column, never silently upgraded to look like a clean entity when it wasn't one.
>
> **The same honesty rule applies below the chunk level too.** If a specific fact for a unit — not just its boundary, but an individual signal like a lifecycle or a rule — can't be found with confidence after a reasonable search, it is marked `Pending` or flagged, never fabricated to look complete. Same standard `/run-priming-session` already holds itself to: an honest gap beats invented precision.
>
> **Flag types — same typed vocabulary `priming-package.md` uses, never an untyped flag:** `AI Knowledge Correction`, `Legacy Design Question`, `Scope Risk`, `Conflicting Source`, or a plain synonym/terminology note.

---

## 1. Session Context

| Field | Value |
|-------|-------|
| Project Code | |
| Codebase location | `projects/{PROJECT_CODE}/source-material/` (same folder `/init-project` creates and `/run-priming-session` already gates on — no separate external-path concept) |
| PO-narrowed scope (if any) | [e.g. "only the billing module" — leave blank if the PO said to use everything present] |
| Target priming package | `projects/{PROJECT_CODE}/priming/priming-package.md` |

---

## 2. Session History

| Session # | Date | Units touched this session |
|-----------|------|-----------------------------|
| 1 | | |

---

## 3. Progress Snapshot

> **[AI Guide]** No numeric confidence score, deliberately — same reasoning `priming-package.md` Section 3 already gives: a single blended number across a whole codebase would imply a precision this walk was never meant to produce. Counts only.

| | Count |
|---|-------|
| Candidate units identified (Map) | |
| — at Entity granularity | |
| — at Module/Layer granularity (degraded) | |
| Classified domain-likely | |
| Classified infrastructure-likely | |
| Domain-likely units fully Extracted | |
| Domain-likely units still Pending | |

---

## 4. Repository Map

> **[AI Guide]** Output of the Map pass — structural only, no file contents read yet. Always produced first, regardless of stack. Two uses, not one: (1) a hint for Section 7's sub-domain-split recommendation, and (2) where entities aren't directly enumerable from a schema/ORM layer, this becomes the **required scaffold** for degraded-granularity walking — group by module/layer here first, then Section 5's entity-inference attempt happens within each group rather than across the whole repo at once.

| Module / Top-level Directory / Layer | Rough Purpose (guess, not yet verified) | Entity layer directly discoverable here? (Y/N) |
|----------------------------------------|--------------------------------------------|--------------------------------------------------|
| | | |

---

## 5. Entity Ledger

> **[AI Guide]** One row per unit — an entity wherever one is cleanly discoverable (primarily from schema/ORM definitions, the highest-trust signal), or a module/layer from Section 4 where it isn't (per the document-level Graceful Degradation note above). `Granularity` states which level this row actually achieved — `Entity` or `Module/Layer` — never implied only by the row's name. Checkpoint granularity is **per row** — a session interrupted mid-walk resumes at the next `Not Started` row, never mid-row. `Pass Status` values: `Not Started` / `Mapped` / `Triaged (domain-likely)` / `Triaged (infrastructure-likely)` / `Extracted`. Infrastructure-likely units stop at Triaged — they get the lightweight batch note in Section 6 instead of individual Extract treatment. `Source Signal` names what the row was actually identified from — an ORM class, a schema file, a raw-SQL table reference, a module grouping — since this varies by stack and by whether Entity or Module/Layer granularity applied. `Signals Found` lists which of the ranked signal types actually turned up something (schema, validation, state machine, tests, comments/commits, flags/config, API surface) — not every row will have all seven, and a `Module/Layer` row may have fewer than an `Entity` row would. `Written To` names the exact `priming-package.md` section/topic the extracted fact landed in (e.g. "Section 5 — Universal Business Rules"), or the Section 10 subsection for the rare code-structure-only fact. Where a specific fact within an otherwise-processed row couldn't be found with confidence, say so in `Notes` rather than leaving the gap unexplained or guessing.

| Unit | Granularity | Source Signal | Pass Status | Signals Found | Written To | Flags | Notes |
|------|--------------|----------------|--------------|-----------------|------------|-------|-------|
| | Entity | | Not Started | | | | |

---

## 6. Infrastructure-Likely Batch

> **[AI Guide]** Infrastructure-likely units (auth plumbing, logging, email delivery, generic admin CRUD, third-party SDK wrappers) get one boundary-level note each here, not individual Extract treatment — per the generalization filter's existing test ("would this be true for any implementation in this domain").

| Unit | One-line boundary note |
|------|-------------------------|
| | |

---

## 7. Sub-Domain Split Candidate

> **[AI Guide]** Non-binding. If the codebase's own module boundaries (Section 4) suggest the eventual domain document should be split per `FW-014` (sub-domain decomposition), name the candidate split here. This is a recommendation handed to `/run-domain-discovery`, not a decision this command makes — the PO still decides, per `FW-014`'s existing best-practice-not-constraint stance. If nothing suggests a split, write "None — single cohesive domain observed."

[Candidate split, or "None — single cohesive domain observed."]

---

## 8. Resume Instructions

> **[AI Guide]** First thing to read when this session resumes.

**Next unit to process:** [Name of the first `Not Started` or partially-processed row in Section 5, or "All units processed — ready to finalize `priming-package.md`."]

**Before resuming:**
- [ ] Load: this file (already done if you are reading this)
- [ ] Load: `projects/{PROJECT_CODE}/priming/priming-package.md`, if it exists, to see what's already merged
- [ ] Review Section 3 (Progress Snapshot) to rebuild context on how far the walk has gotten
- [ ] Review Section 5 (Entity Ledger) for any row with an unresolved Flag — these need a live PO answer before being treated as settled, per `conventions/priming-command-conventions.md` PC-054

**Units still Pending (do not skip silently):**
- [List units from Section 5 not yet `Extracted` or, for infrastructure-likely, not yet in Section 6]

---

## 9. Pre-Handoff Verification

> **[AI Guide — Verification]** Run these checks before merging this session's findings into `priming-package.md` and reporting the session as done. A failed check does not mean the session failed — it means the gap gets recorded in Section 8 (Resume Instructions) rather than silently dropped.

- [ ] Section 1 (Session Context): Project Code and Codebase location are filled
- [ ] Every row in Section 5 (Entity Ledger) has a `Granularity` value (`Entity` or `Module/Layer`) — no row left implying entity-level precision it didn't actually achieve
- [ ] Every row in Section 5 marked `Extracted` has a non-empty `Written To` column — no extracted unit with nowhere recorded
- [ ] Every Flag in Section 5 is tagged with a type from the canonical list (`AI Knowledge Correction` / `Legacy Design Question` / `Scope Risk` / `Conflicting Source` / synonym note) — no untyped flags
- [ ] Every fact actually merged into `priming-package.md` this session traces back to a Section 5 or Section 6 row here — nothing written to the priming package without a corresponding ledger entry
- [ ] Section 7 (Sub-Domain Split Candidate) is filled with a real recommendation or explicitly "None"
- [ ] Section 8 (Resume Instructions) names a real next unit, or states "All units processed — ready to finalize `priming-package.md`"
