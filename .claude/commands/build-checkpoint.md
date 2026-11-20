# Build Checkpoint

Builds exactly one checkpoint of an approved build plan, verifies it, and records it. A checkpoint is one behaviour a reviewer can see work, usually one user action, built from the surface down to the database with its own tests. Building one at a time keeps every change small enough for the developer to review as one commit, and a chunk of checkpoints small enough to review as one pull request.

It runs once per checkpoint, in the plan's order, after `/plan-build-group` has written the plan and the PO and the developer have approved it. It reads the plan, not the wider sources: the plan already carries every fact and decision, copied with its source. If the plan is wrong or missing something, it stops and proposes the change, and never fills the gap from elsewhere. It produces code on the chunk's branch, a Built block under the checkpoint, and the checkpoint's Status set to `Built`. It never commits, pushes or merges, except that the close merges and deploys only when the PO says so.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code and a checkpoint ID. Optional: `--check-tests`.

- **Project code:** e.g. `ASCENDRA-PAY-004`
- **Checkpoint ID:** e.g. `B03.1.2`, or `B03.X` for the unit's close
- **`--check-tests`:** after the tests pass, break the code on purpose to prove they can fail (Step 6). Always done for a checkpoint with a money command block, given or not.

If both required fields are missing:

> "To build a checkpoint I need:
> 1. **Project code:** the project folder under `projects/` (e.g. `ASCENDRA-PAY-004`)
> 2. **Checkpoint ID:** from the unit's build plan (e.g. `B03.1.2`)"

Wait for PO response.

If only the project code is missing:

> "Which project is this checkpoint in? Provide the project code (e.g. `ASCENDRA-PAY-004`)."

Wait for PO response.

If only the checkpoint ID is missing:

> "Which checkpoint should I build? Give its ID from the build plan (e.g. `B03.1.2`). `projects/{CODE}/build/index.md` lists the plans."

Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Plan exists | `projects/{CODE}/build/group-{GG}-*.md` exists, where `{GG}` is the ID's unit | > "There's no build plan for unit {GG} under `projects/{CODE}/build/`. Run `/plan-build-group {CODE} {GG}` first." |
| Plan approved | The plan's Status is `Approved` or `In progress` | > "The plan for unit {GG} is still {STATUS}. The PO and the developer approve it first, by setting its Status to Approved." |
| Checkpoint exists | The ID is a row in the plan's Section B0, or `B{GG}.X` | > "{ID} isn't in the plan for unit {GG}. Check the ID." |
| Not already built | Its Status is `Planned` | > "{ID} is already {STATUS}. Rebuilding it would replace its Built block. Do you want to rebuild it?" |
| Earlier checkpoints built | Every checkpoint above it in Section B0 is `Built` or `Moved`; for `B{GG}.X`, all of them | > "{ID} comes after {EARLIER_IDS}, which aren't built. Build them first, or tell me to continue anyway." |
| Earlier chunks merged | For a chunk's first checkpoint: the previous chunk's pull request is merged | > "Chunk {PREVIOUS} isn't merged yet. Its checkpoints should be accepted and merged first, so this chunk starts from them. Continue anyway?" |
| Units it depends on closed | Every unit in the plan's "Depends on" is `Closed` in `build/index.md` | > "Unit {GG} depends on {UNITS}, which aren't closed. Continue anyway?" |
| Clean start | In each code repo the checkpoint touches: on the chunk's branch (or the main branch, for the chunk's first checkpoint) with no uncommitted changes | > "{REPO} has uncommitted changes on {BRANCH}. Commit the previous checkpoint first, so this one's changes stand on their own." |
| Plan still matches the code | Commits in each repo since the plan's Reconciled commit, other than this unit's own, don't change a module, table, endpoint or shared part that Part A names | > "Since this plan was reconciled, {COMMITS} changed {WHAT}, which the plan relies on in {SECTIONS}. My recommendation: [update the plan as follows / no change needed, because …]. Shall I go on?" |

On any failure, show the message and wait for PO response.

---

## Step 2 — Read files

Record the start time now. It goes in the Built block.

Read, in this order:

1. `projects/{CODE}/CLAUDE.md`: the project's rules, and its row for `/build-checkpoint`, which names the artboard sources and the design-parity harness.
2. **The plan, in parts:** the header, Part A in full, the checkpoint's own section (or Part C for the close), Section B0, the Built blocks of earlier checkpoints in the same chunk, and Section C3. **The checkpoint's blocks are binding**, and so is Part A.
3. `projects/{CODE}/architecture/commons-changes.md`: so no changed commons copy is re-copied.
4. The project's pattern catalogue, in parts: only the entries the checkpoint's blocks name.
5. **For a checkpoint with screens:** the artboard sources of its surfaces, in full, each found from its canvas title (as the plan names it) through the canvas index; the design system README's sections for the parts they use; and, in `reference/ascendra-ui/`, `hard-instructions.md`, `ui-reference.md` and `code-style.md`, with `showcase-reference.md` only to find a specific pattern.
6. **For the close, `B{GG}.X`:** the plan in full, and `build/index.md`.
7. **The code it builds on:** the files earlier Built blocks list, the modules and screens the checkpoint changes, and nearby code that shows the repo's conventions.

Don't read the brief, BRD documents, source material, domain document, screen inventory or design decision log. The plan already carries what they decided. If something the build needs isn't in the plan, that is a plan gap: Step 3.

---

## Step 3 — Check before building

Work out three things, then show them in one message:

1. **Needs first:** what the checkpoint relies on (earlier checkpoints' code, a running database, the seed, a provider account), and whether each is met. Check it yourself where you can. Mark it "PO to confirm" where you can't.
2. **Needed from the PO:** secrets, accounts, or a choice. For each, say exactly where it goes and what it looks like.
3. **Plan gaps:** anything the checkpoint needs that its blocks and Part A don't state, anything in them that is wrong against the code, and anything that would take it past its size or change its scope. For each, give the change to the plan you recommend. An approved plan changes only with the PO's agreement, recorded in its Document Control.

> "Before building {ID} ({goal}):
>
> **Needs first:** [each item: met / not met / PO to confirm]
> **Needed from you:** [each item and where it goes, or 'nothing']
> **Plan gaps:** [each item with the plan change I recommend, or 'nothing']
>
> Shall I start?"

Wait for PO response. Do not write any code until the PO says go.

If all three are empty, say so in one line and still wait.

**Questions:** ask in plain chat, never through a selection widget. Ask one question at a time when a choice needs its own answer, each with your recommendation.

**Secrets:** the PO puts them in a gitignored `.env` in the code repo, never in chat if avoidable, and never in a committed file. Give the PO the keys to add, blank. When the PO says it's ready, check each key is set and not a placeholder without printing any value.

---

## Step 4 — Build

**Branch:** one branch per chunk, `b{GG}.{chunk}-{slug}` (e.g. `b03.1-staff-list`), in each code repo the chunk touches. Create it at the chunk's first checkpoint from the main branch, never from an unmerged branch; afterwards, stay on it. Code from an unmerged branch is used only where the plan's A14 lists it. Make no commits.

Build what the checkpoint's blocks say, and nothing more.

**Rules while building:**
- **The plan's words are the specification.** Text, IDs, rule codes, messages, types and query keys are used exactly as written.
- **IDs in code:** each surface's root carries its surface ID as `data-testid`. The web client's function for an endpoint is named after its action, and a query's key is its query ID's record and purpose, as the data loading block gives them. Each rule's error uses its code. Tests are named as the verification block names them.
- **Folders:** every file goes in the folder of the module that owns it, by the architecture note's layout and the plan's module block. A new module is registered in the project's boundary check (e.g. `scripts/boundaries.config.ts`) with its kind, layer and table prefixes.
- **Effects through events:** no module calls audit or notification directly. An audit entry is raised as a critical event inside the transaction (commons' `emitCritical()`, recorded by commons' audit listener). An email is sent by a reaction in the applications layer to an event emitted only after the commit (the unit of work's after-commit step, or the outbox once it exists). Money between modules stays a direct contract call inside one transaction.
- **This checkpoint only.** Code for a later checkpoint isn't written now, even when it's obvious. If this checkpoint can't work without it, stop and ask.
- **Stay within Part A.** Modules, calls between them, tables and patterns are as Part A decides. If the checkpoint can't be built that way, stop and propose the plan change.
- **Commons:** copy a module only with the commons tool, and only the parts named. If a copy needs a change, make the smallest change and add a row to `commons-changes.md`. Never re-copy a module listed there.
- **Screens:** start from the artboard source, keep its composition, text and states, and replace sample data with the real API. A library part used in the artboard is used here, whole. A part the plan says to extract is built once, where the plan says it lives. A link to a screen not yet built is left out. A deferred part shows exactly what the plan's A3 says: its true current state, or the action disabled with its reason; never a placeholder or sample data. A 400 shows the API's messages as a list in the form's error alert.
- **Seed:** the records the plan's Section A11 lists for this checkpoint are added through the modules' real contracts, never by inserting rows directly.
- **Small choices** the plan doesn't state, and that no reviewer would question (a private helper's name, a file split), are yours. Record each one for the Built block. Anything a reviewer might question is a plan gap: stop and ask.
- **Size:** if the change is growing past the checkpoint's limit, stop and propose a split.
- **Use the domain's terms** in names, as the plan uses them.

---

## Step 5 — Verify

Every gate runs, in this order. A failed gate is fixed in the code, not in the test, before the next gate runs. Change a test only when the test itself is wrong, and say so in the Built block.

1. **Static:** the type check, linting, and the project's boundary checks, in each repo touched.
2. **Tests:** every test the verification block names, against the real thing (a real PostgreSQL as the app role, not a mock) unless the plan says otherwise. At least one of them must fail without this checkpoint's change: confirm it. Then every earlier test in the unit.
3. **Financial checks:** for a checkpoint with a money command block, every check the block lists.
4. **Design parity (screens only):** render each artboard the verification block names (by canvas title, its source found through the canvas index) and the built surface at the same size, the surface with the seed, as the plan's header says. Look at both. List every difference in layout, text, parts and states. Fix what's wrong; a difference you keep goes in the Built block with its reason, and needs to be in Section A13 or proposed for it.
5. **The developer's check:** run its steps yourself where you can, and record the result. The developer runs them again with the commit.
6. **Chunk's last checkpoint:** run the chunk's acceptance steps on the running app, and record the result. List the commits the chunk's pull request will hold.
7. **Close only (`B{GG}.X`):** the demo as a browser test, the browser tests of every closed unit, the financial checks. Then stop and ask before merging:

   > "{UNIT} passes its demo and the regression of {N} closed units. Shall I deploy to staging? Every chunk's pull request is merged first, by you."

   Wait for PO response. Merge and deploy only as the PO says.

If a gate can't pass without breaking something the plan made binding, stop:

> "{GATE} can't pass without [what would break]. My recommendation: [option]. How do you want to handle it?"

Wait for PO response.

---

## Step 6 — Check the tests can fail

Run this step for a checkpoint with a money command block, and for any checkpoint run with `--check-tests`. Otherwise skip it, and the Built block says "Tests not checked by breaking the code."

For each rule the block's "Breaks to prove the tests" lists, or each rule the checkpoint enforces (at most three):

1. Back up the file, then break the rule in the code.
2. Run the tests with a short timeout. Record which tests failed.
3. Restore the file from the backup, and confirm it matches the original byte for byte.

Every break must make at least one test fail. If one doesn't, a test is missing: add it, and say so. After the last break, run the full suite again and confirm everything passes.

---

## Step 7 — Record

Record the end time. In the plan:

1. Under the checkpoint's section, add its Built block, at most 10 lines, in the form the template gives:
   - result in one sentence
   - files by area, as links
   - differences from the plan: every small choice, every change to a commons copy, every PO decision taken during the build; write "none" only if there were none
   - tests: how many passed and where; the parity result for screens; breaks caught if Step 6 ran
   - a proposed commit message per repo
2. Set the checkpoint's Status to `Built`, in its section and in Section B0. Set the plan's Status to `In progress` if it was `Approved`.
3. Append to Section C3 anything a later checkpoint or unit must act on, and to Section C2 anything a document's owner should change.
4. Update the unit's row in `build/index.md`: checkpoints built.
5. **Close only:** fill Part C's Built block; set the plan's Status and its index row to `Closed`, with the date; add a decision to the project's decision log if it records them by number.

Commit nothing. List every changed file in each repo, so the developer can review and the PO can commit.

---

## Step 8 — Report

```
CHECKPOINT BUILT — {PROJECT_CODE} {ID}
─────────────────────────────────────────
Goal:           {what a reviewer can now see working}
Branch:         {repo} b{GG}.{chunk}-{slug} (not committed)
Tests:          {N} passed, on {where}; {N} fail without the change
Parity:         {no differences / N kept, listed / no screens}
Tests checked:  {yes, N breaks caught / no}
Plan changes:   {agreed in Step 3, or none}
Time taken:     {minutes}, of which {minutes} waiting on the PO
Changed:        {N} files in {repos}; plan and build/index.md
Needs the PO:   {anything left open, or none}
─────────────────────────────────────────
Next: the developer reviews the diff and the Built block, then commit
with the proposed message. Then /build-checkpoint {PROJECT_CODE} {NEXT_ID}
{or, at a chunk's end: the PO runs the chunk's acceptance steps, then
open the chunk's pull request}
```
