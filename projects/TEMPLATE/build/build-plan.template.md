# Build Plan: Group {GG}, [Group Name]

> **[AI Guide — Document Level]**
> The plan for building one build unit: usually one approved design group, sometimes a named unit with no screens of its own (such as a money spine taken from a slice spec). It splits the unit into **chunks** (one capability a named role can use) and each chunk into **checkpoints** (the smallest step that can be checked: one layer of one chunk, commit-sized). Every checkpoint is built and verified on its own by `/build-checkpoint`, and the PO commits after each.
>
> Produced by `/plan-build-group`, after the PO has answered its reconciliation and gap questions. Approving it (Status: Approved) lets `/build-checkpoint` start. Each checkpoint's Built block is added under its chunk as it is built; the unit's close (checkpoint `B{GG}.X`) completes the document.
>
> **What this document is not:**
> - Not a design. Screens, text and states come from the approved artboards; this plan points to them and never restates them.
> - Not a spec of every field or endpoint. A checkpoint names what it builds and how it is proved; the shape is the builder's choice within the architecture note.
> - Not a place for architecture decisions. A change to what the architecture note decides goes to Section 7, for the note's owner to apply.
> - Not a story list. There are no user stories; the chunk's demo sentence states the job.
>
> **When used by `/plan-build-group`:** the structure to generate, section by section, with every AI Guide followed. Checkpoint tables start with Status `Planned` and no Built blocks.
>
> **When used by `/build-checkpoint`:** read the header, Sections 1 to 4, the chunk holding the checkpoint, the Built blocks of earlier checkpoints (their Differences lines), and Section 8. Write only the checkpoint's Built block and its Status cell, and append to Section 8 if something moves. Never rewrite other sections: a needed change is proposed to the PO.
>
> **File location:** `projects/{PROJECT_CODE}/build/group-{GG}-{slug}.md`, where `{GG}` is the two-digit design group number from the screen inventory, or a short name for a unit without a group (`spine`).

**Status:** Draft

| Field | Value |
|---|---|
| Unit | [Group {GG}, Group Name, or the unit's name] |
| Design approval | [The decision that approved the group's design, linked; canvas version] |
| Depends on | [Build units that must be closed first, linked to their plan files, or "none"] |
| Repos and base | [Each code repo, its branch `group/{GG}-{slug}`, and the commit it starts from] |
| Design parity | [How an artboard is rendered (the project's harness and command) and how the built page is rendered at the same size with the seed] |
| Plan size | [N chunks, N checkpoints] |

## Document Control

> **[AI Guide]** Add a row each time the plan's scope, chunks or checkpoints change. Built blocks don't need a row.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |

---

## 1. Scope

> **[AI Guide]** Everything the unit builds, taken from the screen inventory's rows for the group and the decisions that drew and approved it. One line per item, each linked to its source. A screen is in scope at the content the design approved, unless it is listed under "Moves later" with the unit it moves to and why. Links into screens that aren't built yet are absent, not greyed out.
>
> **Correct:** "SCR-023 Payers: server listing, presets All payers, No students linked, Email bounced; artboards PyrList, PyrFilter."
> **Incorrect:** "Payers screen with a table and filters." (no source, no artboards, no presets)

**Screens:**

| SCR | Screen | Artboards | In this unit |
|---|---|---|---|
| [SCR-NNN] | [Name] | [Artboard names] | [All / Part: what] |

**Rules each screen enforces:** [UBR, VAL and design decision numbers, each with the screen or action that enforces it]

**Records, by owning module:** [Module: records it gains or changes]

**Roles and actions:** [Each named action, the roles that hold it, and whether it needs a second person or only a permission]

**Events, documents and emails:** [What each action produces, and where its document or email is built]

**Moves later:**

| Item | Moves to | Why |
|---|---|---|
| [Item] | [Unit] | [Reason] |

---

## 2. Reconciled

> **[AI Guide]** Every place where existing code, the architecture note or the domain document disagrees with the approved design, and the fix the PO agreed. Name the file or record that disagrees. Write "None found" only after checking the code repos, not just the documents.

| # | What disagrees | Where | Agreed fix | Fixed in |
|---|---|---|---|---|
| 1 | [Description] | [File, table or document section] | [Fix] | [Checkpoint ID] |

---

## 3. Decisions

> **[AI Guide]** What the design doesn't settle but the build needs, as the PO answered it: error codes and messages, concurrency, which commands need an idempotency key, permission names, audit entries, data shapes that are hard to change later. One row per decision; don't re-decide anything the architecture note or a design decision already settled.

| # | Decision | Answer | Reason |
|---|---|---|---|
| 1 | [Question] | [Answer] | [Why] |

---

## 4. Seed and test data

> **[AI Guide]** The sample records the seed must produce so built screens can be compared with their artboards: the same school, people, numbers and figures the artboards show. Seeded through the modules' real contracts, never by inserting rows directly. List what this unit adds to the seed; earlier units' seed data is assumed.

- [Record set: what, how many, which artboard figures it must reproduce]

---

## 5. Chunks

> **[AI Guide]** 2 to 5 chunks, each one capability a named role can use, in build order. A chunk's demo sentence names the role and the job: "The fee clerk adds a payer and finds them in the list." Chunks are cut by capability, never by layer.

| Chunk | Demo | Depends on |
|---|---|---|
| B{GG}.1 [Name] | [Role] [does what] | [Chunk or unit] |

---

## 6. Checkpoints

> **[AI Guide]** One sub-section per chunk, as a bold label. Checkpoints are built from the inside out, one layer at a time:
> - **D, Data:** tables, constraints, row security, migration. Proved by database tests on real PostgreSQL as the app role.
> - **R, Rules:** module contract methods, business rules, audit entries. Proved by unit and database tests; a money rule also by breaking it on purpose.
> - **A, API:** endpoints, named actions, idempotency, errors. Proved by API tests: allowed, denied, another school, a retry.
> - **S, Screen:** one page and its states, built from its artboard source against the real API. Proved by design parity (rendered beside its artboard at the same size) and a browser test of its states.
> - **F, Flow:** the chunk's demo end to end. Proved by a browser test and the financial checks.
>
> Limits: one module and at most one screen; about 400 changed lines, not counting generated files and migrations; at least one test that fails before and passes after. A trivial settings screen may merge D, R and A into one checkpoint; say so in its row. IDs are `B{GG}.{chunk}.{n}`; Status is `Planned`, `Built` or `Moved`.
>
> **Correct:** "B04.1.2 | R | The payer listing query: batches of 50, filters by name, payer number, mobile, email; presets | Database tests: each filter, a batch boundary, another school's payers never returned | Planned"
> **Incorrect:** "B04.1.2 | Backend | Payers backend | Tests | Planned" (several layers, nothing provable)

**B{GG}.1 [Chunk name]**

| ID | Kind | Builds | Proved by | Hand check | Status |
|---|---|---|---|---|---|
| B{GG}.1.1 | [D/R/A/S/F] | [What, in one line] | [Tests and gates] | [One step and its expected result, or "—"] | Planned |

> **[AI Guide]** `/build-checkpoint` adds a Built block under the table for each checkpoint it builds, in this form, at most 10 lines:
>
> **Built B{GG}.1.1** ([Date], [time taken]): [Result in one sentence]
> - Files: [by area, linked]
> - Differences: [from this plan, each choice the plan didn't state, or "none"]
> - Tests: [N passed, where; parity result for S; breaks caught if run]
> - Hand check: [up to 3 steps with expected results]
> - Commit: `[proposed message]`

---

## 7. Close

> **[AI Guide]** Checkpoint `B{GG}.X`, built last by `/build-checkpoint`: the unit's demo, the browser tests of every closed unit (regression), the financial checks, merge of the unit's branches, and deploy to staging. The PO runs the hand check on staging. Also lists what the build changed that the architecture note decides, and changes to commons copies, for their owners.

**Demo:** [Numbered steps, each with its expected result, covering every chunk]

**Architecture note changes proposed:** [Each, or "none"]

**Commons copy changes:** [Each, as logged in commons-changes.md, or "none"]

**Built B{GG}.X:** [Added at close]

---

## 8. Moves and findings

> **[AI Guide]** Anything found while building that a later unit must act on, or that moved out of this unit, with its destination. `/build-checkpoint` appends here; `/plan-build-group` reads this section of every earlier plan.

| Found in | What | Goes to |
|---|---|---|
| [Checkpoint ID] | [Description] | [Unit or document] |
