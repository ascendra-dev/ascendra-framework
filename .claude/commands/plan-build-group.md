# Plan Build Group

Plans the build of one unit (usually one approved design group) and writes it as one living document, `projects/{CODE}/build/group-{GG}-{slug}.md`, from the build plan template. The document holds every implementation decision and the build order, written for three readers at once: the PO, the developer and the AI that builds it. `/build-checkpoint` builds it one checkpoint at a time.

The command takes every decision itself, and writes each as question, answer and why, so the reviewers check decisions instead of filling gaps. It stops for the PO only on what it may not decide alone: a change to the approved design, content for a surface the design implies but doesn't draw, and a product question the sources don't answer. It writes no code.

It is the only build step that reads the wider sources (design decisions, source material, domain document). Everything the build needs is copied into the plan, so `/build-checkpoint` reads the plan and the code, not the sources. Planning two or three groups in one session shares the common reads; plan them in dependency order, and have each one reviewed before the next is planned.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code and a unit.

- **Project code:** e.g. `ASCENDRA-PAY-004`
- **Unit:** a design group number from the screen inventory (e.g. `3`, `04`), or a short name for a unit with no design group of its own (e.g. `spine`)

If both are missing:

> "To plan a build unit I need:
> 1. **Project code:** the project folder under `projects/` (e.g. `ASCENDRA-PAY-004`)
> 2. **Unit:** a design group number from the screen inventory (e.g. `3`), or a name for a unit without one (e.g. `spine`)"

Wait for PO response.

If only the project code is missing:

> "Which project is this build plan for? Provide the project code (e.g. `ASCENDRA-PAY-004`)."

Wait for PO response.

If only the unit is missing:

> "Which unit should I plan? Give the design group number from the screen inventory (e.g. `3`), or a name for a unit without one (e.g. `spine`)."

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
| Design approved | For a group: its heading in the screen inventory, or the design decision log, records it as approved | > "Group {GG}'s design isn't recorded as approved. Plan it once the design is approved, or tell me to continue anyway." |
| Not already planned | No `projects/{CODE}/build/group-{GG}-*.md` exists | > "Group {GG} already has a build plan: `{path}` ({STATUS}). Do you want me to re-plan it from the start, or change the existing plan? A change to an Approved plan is recorded in its Document Control and needs approval again." |
| Earlier units planned | Every unit the group depends on (the inventory's group table) has a plan in `build/index.md` | > "Group {GG} depends on {UNITS}, which have no plan yet. This plan would have to guess what they build. Plan them first, or tell me to continue anyway." |
| Architecture note exists | `projects/{CODE}/architecture/architecture-note.md` exists | > "There's no architecture note at `projects/{CODE}/architecture/architecture-note.md`. A build plan needs it." |

On any failure, show the message and wait for PO response.

---

## Step 2 — Read files

Read, in this order. Where the project's own `CLAUDE.md` names a different path or a narrower part for any of these, the project wins. Note what you read from each, and which parts: it becomes Section A1 of the plan. Read parts, not whole documents, wherever this list says so: a large document read in full costs context and attention for little the group uses.

1. `projects/{CODE}/CLAUDE.md`: the project's rules, and its row for `/plan-build-group`, which names the design decision log, the artboard sources, the design system, and how source material is mapped to groups.
2. `projects/TEMPLATE/build/build-plan.template.md`, in full: the structure to generate, every AI Guide, the Block library and the Plan checks.
3. The project's pattern catalogue, if it has one (e.g. `build/pattern-catalogue.md`), in full: the patterns every plan names, and its extraction schedule.
4. `projects/{CODE}/screens/screen-inventory.md`, in parts: the group's heading, its rows, the group table (depends on, split), the navigation section, the patterns every screen follows, and any inventory decision those rows cite.
5. The design decision log the project names (e.g. its handoff), in parts: its foundation decisions (modules, architecture, cross-cutting concerns, documentation), and every decision that names this group, its screens or its records. Where it differs from the domain document, it wins.
6. The group's artboard sources, in full, and the group's board list in the canvas index. They are the specification of every surface: its parts, states, text and sample figures. The canvas index gives each artboard's **canvas title** and page: the plan names artboards by that title, never by file name, because the reviewers find artboards on the canvas by their titles.
7. The design system's README, in full: page recipes, lists and states, forms, money and status vocabulary.
8. `projects/{CODE}/architecture/architecture-note.md`, in full, and `projects/{CODE}/architecture/commons-changes.md`. The note's module list, module kinds, layers and folder layout are the only source of module names, kinds and file paths.
9. `reference/api-contract/contract.md`, in full, and the project's API standards if it has them. `reference/form-validation-messages.md`, in full.
10. **Source material, in parts,** as the project's row maps it. Where it gives no map: the glossary; the sections for the modules this group touches; the scenarios, events and acceptance tests that name this group's records; and the security and tenancy section. Read the roadmap only when the group creates a new module, to check what later milestones will need from it.
11. **The domain document, in parts:** the glossary in full (its terms are binding), and the entities, lifecycles and rules for this group's records. The design decision log wins where they differ.
12. **Every earlier plan** in `projects/{CODE}/build/`, in parts: the header, Section A3's surface table, and Sections A5, A7, A8, A12 and C3. They say what already exists, which names are taken, and what was moved into this unit.
13. For a unit with screens, in `reference/ascendra-ui/`: `hard-instructions.md` and `code-style.md`.
14. For a named unit: the sources the PO gave in Step 1, in full.

Then look at the code in each repo the architecture note names, **on its main line**: its latest commit, the modules and tables that exist, the endpoints, the screens, the shared parts already extracted, and the seed. Record what already exists for this unit's scope. Unmerged branches are reference only: note any that hold work in this unit's scope, and lift code from one only as a decision, listed in A14.

Don't read the brief or BRD documents unless a decision in item 5 links to them.

---

## Step 3 — Walk the design

Before deciding anything, list what the group's artboards contain. Don't write any file yet.

1. **Surfaces:** every page, listing, card, menu, sheet, drawer, form, dialog and tab, across every artboard of the group. Give each an ID by the template's scheme, checking earlier plans for surfaces that already exist (extend them, keeping their ID). Note every notice, toast and state too, but under the surface it appears on: they get no ID of their own. A surface that fits none of the template's kinds is a decision (a new kind and its block), never a quiet invention.
2. **Actions:** every button and menu item, and what it does.
3. **Displayed values:** every column, field, fact and figure a person sees, for the field-source table.
4. **Text:** every title, subtitle, message, notice, hint and toast, copied exactly. Note every number in the text (days, limits, amounts): each is a setting or a constant.
5. **Implied but not drawn:** every drawn item whose result has no artboard, such as a menu item with no dialog.
6. **Drawn but not buildable yet:** every surface or value that needs a module, provider or group not yet built.

---

## Step 4 — Decide

Take every decision, in this order, which is the order a reviewer meets them. Each is written into the plan: a fact into its table with a source stamp, and a choice with a real alternative into Section A15 as question, answer and why, naming the alternative rejected. Never leave a decision for the reader.

**Part A, once for the group:**

1. **Scope:** what is built, what is deferred (only when it needs an outside setup that isn't ready, belongs to work the project has placed later, or depends on a unit not yet built), and what is implied. A designed surface is built whole; only the part that later work feeds is deferred. For each deferral, decide what shows in its place now: its **true current state** (real data, even if empty), or the action **disabled with its reason**; **hidden** only for a whole page or sidebar item of an unbuilt group. Never a placeholder or sample data. Where a seam stands in for the later work, name it and its honest default.
2. **Rules:** every business rule, its code, its statement copied from its source, and every place it is enforced. The API always enforces; the screen only hints; the database adds a constraint where the rule fits one row.
3. **Actions and roles:** every action, its roles, its surfaces and its endpoint.
4. **Lifecycle and allowed actions:** every status, its badge, its allowed actions, and every refusal's exact text and where it shows. The server works out allowed actions and reasons and sends them with each record; the screen only shows them.
5. **Modules:** every module touched, including one that only gains configuration (role map actions, email catalogue entries), named only from the architecture note, with its two kinds (where its code comes from; what it is for), its layer and its folder. The top layer is "Applications", never a module name. A module the note lacks is a decision, carried to the note in C2. Every call between modules is a direct contract call (downward only), a critical event (inside the transaction), an after-commit event, or a registration (the higher module registers a check or listener with the lower one, for an upward need).
6. **Data:** the tables and their changes, and the source of every displayed value. A value held by a provider or another module is fetched once per request at most, never once per row; prefer recording it in our own table when we see it change.
7. **Settings and constants:** by the template's rule for loading them.
8. **What each command causes:** audit, events, emails, retry safety and concurrency, for every endpoint that changes data. No module calls audit or notification directly: an audit entry is a critical event, raised inside the transaction; an email is sent by a reaction in the applications layer to an after-commit event. Money between modules stays a direct contract call inside one transaction.
9. **Seed:** a record for every state the artboards show.
10. **Patterns and components:** reuse, extract now (the design shows it twice or more) or one-off, by the catalogue.
11. **Differs from the design:** every difference and every proposal, each to be agreed in Step 6.
12. **Reconciled:** every place existing code or documents disagree with the approved design, and the fix.
13. **Considered for every group:** answer each row of Section A2.

**Part B, per surface, from the surface down:**

14. **Page:** route and parameters, who can open it, recipe, title, subtitle, header actions, notices, tabs (each tab's contents in the blocks that follow). Each surface kind takes its block by the template's mapping.
15. **Each listing:** static or server. Static when the whole set fits in the browser (state its limit) and searching and filtering there is enough. Server when the set is large, or when it has preset queries or filters over data the browser can't hold. Then its full block: columns (order, frozen, shown by default, cell, format, reuse), and for a server listing every query, every filter field, sort, export and each query's server side.
16. **Each menu, sheet, drawer, form, dialog and card:** its full block, with exact text, and every response handled (success, validation, each rule's code, no permission, version conflict, network error), with what shows, where it goes, and what refreshes. A 400 shows the API's messages as a bulleted list in the form's error alert, never under fields.
17. **Data loading:** every list a page shows gets its own query and endpoint, even one that returns fixed values today, so its source can change later without touching the screen. Per page, decide one combined query or parallel queries: parallel when the surfaces make sense apart, so each has its own loading, empty, error and no-permission state. Each query has a query ID; its key is the ID's record and purpose, and the client function for each endpoint is named after its action. Files sit in the owning module's folder.
18. **Endpoints, modules and migrations:** their full blocks.

**Then the cut:**

19. **Checkpoints:** one visible behaviour each, usually one user action, from the surface down to the database. Order them so each builds only on earlier ones: the listing first, then the action that creates records, then the actions that change them. A checkpoint with nothing to see stands alone only when it is security- or data-critical. Split anything past about 600 changed lines (not counting tests, generated files and migrations) by surface or action, never by layer.
20. **Chunks:** 2 to 5, each one capability a named role can use, with a one-sentence demo and acceptance steps for the PO.
21. **Verification:** per checkpoint, the automated tests in plain language, named by rule code, at least one failing without the change; the developer's check; the design-parity pairs. Per chunk, the PO's acceptance steps. At the close, the demo, regression, financial checks and deploy.

---

## Step 5 — Check the decisions

Before asking the PO anything, run every one of the template's Plan checks against the decisions from Step 4, as if the plan were written. Fix every failure you can decide yourself by going back to Step 4. Every failure that needs the PO (a text no artboard draws, a difference from the design, a product question) joins Step 6's list. Nothing is written yet.

This step exists so that everything needing the PO is found before the PO is asked, and is asked at once, rather than found while writing.

---

## Step 6 — Ask the PO what only the PO can decide

Show, in one message, only what the command may not decide alone, each numbered with a recommendation. That includes everything Step 5 found:

- **Design changes:** every Section A13 difference, including what shows for each deferred item.
- **Implied but not drawn:** the proposed content of each such surface (title, text, buttons, outcome).
- **Product questions:** anything the build needs that no source answers and that changes what a person sees or what the product does. A technical choice is not a product question: decide it in Section A15.
- **Reconcile items** whose fix changes approved behaviour.

Then the plan at a glance, for information: the chunks and their checkpoints in one line each, and the deferred items.

> "Before I write the plan for Group {GG}, these need your decision:
>
> **Design changes:**
> 1. [Where (artboard canvas title): what's drawn, what would be built, why.] My recommendation: [answer].
>
> **Implied but not drawn:**
> 2. [Surface, implied by …: proposed title, text, buttons, outcome.] My recommendation: [answer].
>
> **Product questions:**
> 3. [Question.] My recommendation: [answer].
>
> **At a glance:** [N] chunks, [M] checkpoints.
> - B{GG}.1 [Chunk]: [demo]. B{GG}.1.1 [goal]; B{GG}.1.2 [goal]
> - …
> - Deferred: [items and where they go]
>
> Answer by number, or say 'yes to all'. Everything else is decided in the plan, with its reasons, for you and the developer to review."

Wait for PO response. Do not write any files until confirmed.

If nothing needs a decision, say so, show the glance, and still wait.

**Questions:** ask in plain chat, never through a selection widget. Where an answer opens a new question, ask it as its own numbered item with a recommendation.

---

## Step 7 — Write the plan

Write `projects/{CODE}/build/group-{GG}-{slug}.md` from the template. `{slug}` is the group's name in lower case with hyphens (e.g. `group-03-staff-and-roles.md`).

- Fill every section from Steps 2 to 6 and the PO's answers. Record each agreed design change in Section A13 with the PO's agreement and date.
- Copy into each checkpoint every block it needs from the Block library, filled in full. Remove the Block library and every AI Guide.
- Copy every fact the build needs, with its source stamp. Links are real markdown links (C-043), and never replace the copied fact.
- Status: `Draft`. Every checkpoint's Status: `Planned`. No Built blocks. The header's Reconciled line gives today's date and the latest commit of each repo.
- Section C3 starts with any item an earlier plan moved into this unit.
- Run the Plan checks again on the written plan, since the PO's answers may have changed it. Fix every failure, then tick each check. A check that still needs a PO decision means Step 5 missed it: ask it as in Step 6 before finishing, and note in C3 that Step 5 missed it.

---

## Step 8 — Update cross-references

1. **`projects/{CODE}/build/index.md`:** create it if missing, with this structure, then add or update the unit's row:

```markdown
# Build Index

One row per build unit, in build order. Each unit's plan holds its chunks, checkpoints and Built blocks.

| Unit | Plan | Status | Checkpoints | Closed |
|---|---|---|---|---|
| Group 03 Staff and roles | [group-03-staff-and-roles.md](group-03-staff-and-roles.md) | Draft | 0 of 9 built | — |
```

Status is `Draft`, `Approved`, `In progress` or `Closed`.

2. **The design decision log** the project names: if it records decisions by number, add one recording that the unit's plan was written, with a link, and what the PO decided in Step 6.
3. **Section C2's proposals** (a new module for the architecture note, a new surface kind or block for the template) are listed in the report for their owners. Change neither document here.

Commit nothing.

---

## Step 9 — Report

```
BUILD PLAN WRITTEN — {PROJECT_CODE} Group {GG}
─────────────────────────────────────────
Plan:          projects/{CODE}/build/group-{GG}-{slug}.md (Draft)
Chunks:        {N}
Checkpoints:   {M}, plus the close
Decisions:     {N} in A15; {N} design changes agreed
Deferred:      {items and where they go, or none}
Reconciled:    {N} items, fixed in {checkpoint IDs}
Sources read:  {N}, listed in A1
Also changed:  build/index.md{, decision log}
─────────────────────────────────────────
Next: the PO and the developer review the plan and set its Status to
Approved, then run /build-checkpoint {PROJECT_CODE} B{GG}.1.1
```
