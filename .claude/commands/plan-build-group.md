# Plan Build Group

Plans the build of one unit (usually one approved design group) as chunks and checkpoints, so each step of the code can be built, checked and committed on its own. It is for projects whose screens are designed group by group in dependency order, and built in that same order.

It runs once per unit, after the unit's design is approved and before any of its code is written. It first reconciles existing code and documents with the approved design and asks the PO what the build needs that the design doesn't settle. Only then does it write `projects/{CODE}/build/group-{GG}-{slug}.md` from the build plan template, and add the unit to `build/index.md`. It writes no code. `/build-checkpoint` builds the plan one checkpoint at a time.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code and a unit.

- **Project code:** e.g. `ASCENDRA-PAY-004`
- **Unit:** a design group number from the screen inventory (e.g. `1`, `04`), or a short name for a unit with no design group of its own (e.g. `spine`)

If both are missing:

> "To plan a build unit I need:
> 1. **Project code:** the project folder under `projects/` (e.g. `ASCENDRA-PAY-004`)
> 2. **Unit:** a design group number from the screen inventory (e.g. `1`), or a name for a unit without one (e.g. `spine`)"

Wait for PO response.

If only the project code is missing:

> "Which project is this build plan for? Provide the project code (e.g. `ASCENDRA-PAY-004`)."

Wait for PO response.

If only the unit is missing:

> "Which unit should I plan? Give the design group number from the screen inventory (e.g. `1`), or a name for a unit without one (e.g. `spine`)."

Wait for PO response.

**A named unit (not a group number):** ask for its scope and sources all at once:

| Input | What to ask if missing |
|-------|----------------------|
| Scope | "What does this unit build? One or two sentences." |
| Sources | "Which documents hold its requirements? For example, stories in a slice spec (`US-00-04`, `US-00-06`)." |
| Screens | "Does it build any screens? If so, which artboards." |

Ask for every missing input in one message, all at once. Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Project exists | `projects/{CODE}/` exists | > "There's no project `{CODE}` under `projects/`. Check the code." |
| Design approved | For a group: its heading in the screen inventory carries a "Drawn and approved" line | > "Group {GG}'s design isn't marked approved in the screen inventory. Plan it once the design is approved, or tell me to continue anyway." |
| Not already planned | No `projects/{CODE}/build/group-{GG}-*.md` exists | > "Group {GG} already has a build plan: `{path}`. Re-planning would replace it. Do you want to re-plan, or change the existing plan?" |
| Units it depends on | Every unit the inventory's group table lists under "Depends on" that has a plan in `build/index.md` is Closed or listed as built earlier | > "Group {GG} depends on {UNITS}, which aren't closed. Its plan can be written now, but its checkpoints can't start until they close. Continue?" |
| Architecture note exists | `projects/{CODE}/architecture/architecture-note.md` exists | > "There's no architecture note at `projects/{CODE}/architecture/architecture-note.md`. A build plan needs it." |

On any failure, show the message and wait for PO response.

---

## Step 2 — Read files

Read, in this order. Where the project's own `CLAUDE.md` names a different path for any of these, its path wins.

1. `projects/{CODE}/CLAUDE.md`: the project's rules, and its row for `/plan-build-group` if it has one, which names the design decision log and other project-specific inputs.
2. `projects/TEMPLATE/build/build-plan.template.md`: the structure to generate, with every AI Guide.
3. `projects/{CODE}/screens/screen-inventory.md`: the group's heading, its rows, the group table (depends on, split), the navigation section, and the patterns every screen follows.
4. The design decision log the project names (e.g. its handoff): every decision that drew, revised or approved this group, read in full. Where it differs from the domain document, it wins.
5. The group's artboard sources: the screen source files those decisions name, and the board list for the group in the canvas index. They are the specification of every screen: its parts, states, text and sample figures.
6. The design system's README: page recipes, lists and states, forms, money and status vocabulary.
7. `projects/{CODE}/architecture/architecture-note.md`, in full.
8. `projects/{CODE}/architecture/commons-changes.md`: which commons copies have been changed.
9. The domain document sections the inventory rows and decisions cite, and only those.
10. For a named unit: the sources the PO gave in Step 1 (e.g. stories in a slice spec), in full.
11. Every earlier plan in `projects/{CODE}/build/`: only Section 2 (Reconciled), Section 3 (Decisions) and Section 8 (Moves and findings), to stay consistent and pick up anything moved into this unit.
12. Only for a unit with screens, in `reference/ascendra-ui/`: `hard-instructions.md` and `code-style.md`.

Then look at the code itself in each repo the architecture note names: the current branch, uncommitted changes, the modules and tables that exist, the endpoints, the screens and the seed. Record what already exists for this unit's scope.

Do not read the brief, BRD documents or source material unless a decision in item 4 links to them.

---

## Step 3 — Reconcile and find the gaps

Work out four lists. Don't write any file yet.

1. **Reconcile:** every place where existing code, the architecture note or the domain document disagrees with the approved design. Name the file, table or section, and recommend a fix. For example: a table built before a design decision changed the model, or a role held as one value where the design allows several.
2. **Scope:** what the unit builds at the content the design approved, and anything you recommend moving to a later unit. Recommend moving an item only when it needs an outside setup that isn't ready (an account, a provider, a document from outside), when it belongs to work the project has placed later (such as security work before real data), or when it depends on a unit not yet built. Give the destination and the reason for each.
3. **Gaps:** what the build needs that the design doesn't settle. Check each of these: error codes and messages, concurrency and edit conflicts, which commands need an idempotency key, permission names and who holds them, audit entries, data shapes that would be expensive to change later, and the seed figures the artboards need.
4. **Already built:** code that already covers part of this unit, and whether it is kept, corrected or replaced.

Show them in one message, each item numbered and each with a recommendation:

> "Before I write the plan for Group {GG}:
>
> **Reconcile:**
> 1. [What disagrees, where.] My recommendation: [fix].
>
> **Scope:** [In, at the approved content.] **Moves later:**
> 2. [Item.] My recommendation: [move to unit, because …], or [keep].
>
> **Gaps:**
> 3. [Question.] My recommendation: [answer].
>
> **Already built:** [What exists and what happens to it, or 'nothing'.]
>
> Answer by number, or say 'yes to all'."

Wait for PO response.

**Questions:** ask in plain chat, never through a selection widget. Where an answer opens a new question, ask it as its own numbered item with a recommendation.

---

## Step 4 — Chunks and checkpoints

With the PO's answers, split the unit:

**Chunks:** 2 to 5, each one capability a named role can use, with a one-sentence demo that names the role and the job. Cut by capability, never by layer. Order them so each builds only on chunks before it.

**Checkpoints:** within each chunk, built from the inside out, one layer at a time:
- **D, Data:** tables, constraints, row security, migration.
- **R, Rules:** module contract methods, business rules, audit entries.
- **A, API:** endpoints, named actions, idempotency, errors.
- **S, Screen:** one page and its states, from its artboard source, against the real API.
- **F, Flow:** the chunk's demo end to end, as a browser test, with the financial checks if money moves.

Then the unit's close, `B{GG}.X`: its demo, the regression of every closed unit, merge and deploy to staging.

**Limits for every checkpoint:**
- One module and at most one screen.
- About 400 changed lines, not counting generated files and migrations.
- At least one test that fails before the change and passes after it.
- Its hand check fits in one step, or it says "—".

A trivial settings screen may merge D, R and A into one checkpoint; mark the row. Anything that doesn't fit the limits is split, not stretched.

**Design parity:** decide how an artboard is rendered (the project's own harness) and how the built page is rendered at the same size with the seed, for the plan's header. Every S checkpoint is checked this way.

Present the breakdown:

> "Group {GG} in {N} chunks and {M} checkpoints:
>
> **B{GG}.1 [Chunk]:** [demo sentence]
> - B{GG}.1.1 D: [what]. Proved by [tests]
> - B{GG}.1.2 R: [what]. Proved by [tests]
> - …
>
> **B{GG}.X Close:** [demo in one line], regression, staging.
>
> **Seed adds:** [what the seed must produce for the artboards]
>
> Does this breakdown look right? Tell me what to split, merge, move or reorder."

Wait for PO response. Do not write any files until confirmed.

---

## Step 5 — Write the plan

Write `projects/{CODE}/build/group-{GG}-{slug}.md` from the template. `{slug}` is the group's name in lower case with hyphens (e.g. `group-04-payers.md`).

- Follow every AI Guide in the template, then remove the guides.
- Fill every section from Steps 3 and 4 and the PO's answers. Every scope item, rule and decision links to its source (C-043).
- Status: `Draft`. Every checkpoint's Status: `Planned`. No Built blocks.
- Section 8 starts with any item an earlier plan moved into this unit.

---

## Step 6 — Update cross-references

1. **`projects/{CODE}/build/index.md`:** create it if missing, with this structure, then add or update the unit's row:

```markdown
# Build Index

One row per build unit, in build order. Each unit's plan holds its chunks, checkpoints and Built blocks.

| Unit | Plan | Status | Checkpoints | Closed |
|---|---|---|---|---|
| Group 01 Access | [group-01-access.md](group-01-access.md) | Draft | 0 of 14 built | — |
```

Status is `Draft`, `Approved`, `In progress` or `Closed`.

2. **The design decision log** the project names: if it records decisions by number, add one recording that the unit's build plan was written, with a link, and what the PO decided in Step 3.

Commit nothing.

---

## Step 7 — Report

```
BUILD PLAN WRITTEN — {PROJECT_CODE} Group {GG}
─────────────────────────────────────────
Plan:          projects/{CODE}/build/group-{GG}-{slug}.md (Draft)
Chunks:        {N}
Checkpoints:   {M}, plus the close
Reconciled:    {N} items, fixed in {checkpoint IDs}
Moves later:   {items and destinations, or none}
Also changed:  build/index.md{, decision log}
─────────────────────────────────────────
Next: review the plan and set its Status to Approved, then run
/build-checkpoint {PROJECT_CODE} B{GG}.1.1
```
