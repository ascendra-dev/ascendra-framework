# Build Checkpoint

Builds exactly one checkpoint of an approved build plan, verifies it, and records it. A checkpoint is the smallest step that can be checked: one layer of one chunk (data, rules, API, screen or flow), or the unit's close. Building one at a time, with the PO committing after each, keeps every change small enough to review and easy to revert.

It runs once per checkpoint, in the plan's order, after `/plan-build-group` has written the plan and the PO has approved it. It produces code on the unit's branch in each code repo it touches, a Built block of at most 10 lines under the checkpoint in the plan, and the checkpoint's Status set to `Built`. It never commits, pushes or merges, except that the close checkpoint merges and deploys only when the PO says so.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code and a checkpoint ID. Optional: `--check-tests`.

- **Project code:** e.g. `ASCENDRA-PAY-004`
- **Checkpoint ID:** e.g. `B04.1.2`, or `B04.X` for the unit's close
- **`--check-tests`:** after the tests pass, break the code on purpose to prove they can fail (Step 6). Always done for an R checkpoint that enforces a money rule, given or not.

If both required fields are missing:

> "To build a checkpoint I need:
> 1. **Project code:** the project folder under `projects/` (e.g. `ASCENDRA-PAY-004`)
> 2. **Checkpoint ID:** from the unit's build plan (e.g. `B04.1.2`)"

Wait for PO response.

If only the project code is missing:

> "Which project is this checkpoint in? Provide the project code (e.g. `ASCENDRA-PAY-004`)."

Wait for PO response.

If only the checkpoint ID is missing:

> "Which checkpoint should I build? Give its ID from the build plan (e.g. `B04.1.2`). `projects/{CODE}/build/index.md` lists the plans."

Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Plan exists | `projects/{CODE}/build/group-{GG}-*.md` exists, where `{GG}` is the ID's unit | > "There's no build plan for unit {GG} under `projects/{CODE}/build/`. Run `/plan-build-group {CODE} {GG}` first." |
| Plan approved | The plan's Status is `Approved` or `In progress` | > "The plan for unit {GG} is still {STATUS}. Approve it first: review it, then set its Status to Approved." |
| Checkpoint exists | The ID is a row in the plan's Section 6, or `B{GG}.X` | > "{ID} isn't in the plan for unit {GG}. Check the ID." |
| Not already built | Its Status is `Planned` | > "{ID} is already {STATUS}. Rebuilding it would replace its Built block. Do you want to rebuild it?" |
| Earlier checkpoints built | Every checkpoint above it in the plan is `Built` or `Moved`; for `B{GG}.X`, all of them | > "{ID} comes after {EARLIER_IDS}, which aren't built. Build them first, or tell me to continue anyway." |
| Units it depends on closed | Every unit in the plan's "Depends on" is `Closed` in `build/index.md` | > "Unit {GG} depends on {UNITS}, which aren't closed. Continue anyway?" |
| Clean start | In each code repo the checkpoint touches: on the branch `group/{GG}-{slug}` (or on the base commit, for the unit's first checkpoint) with no uncommitted changes | > "{REPO} has uncommitted changes on {BRANCH}. Commit the previous checkpoint first, so this one's changes stand on their own." |

On any failure, show the message and wait for PO response.

---

## Step 2 — Read files

Record the start time now. It goes in the Built block.

Read, in this order:

1. `projects/{CODE}/CLAUDE.md`: the project's rules, and its row for `/build-checkpoint` if it has one.
2. The plan: its header, Sections 1 to 4, the chunk holding this checkpoint, the Differences line of every earlier Built block, and Section 8. **The checkpoint's row is binding: what it builds and how it is proved.** Sections 2 and 3 are binding. The chunk's demo is context.
3. `projects/{CODE}/architecture/architecture-note.md`, in full: the decisions every checkpoint stays within.
4. `projects/{CODE}/architecture/commons-changes.md`: so no changed commons copy is re-copied.
5. The sources the checkpoint's scope items link to in the plan's Section 1, and only those: the inventory row, the design decision, the domain section.
6. **For an S checkpoint:** the artboard source of the screen and every state the row names, in full, and the design system README's sections for the parts it uses. Then, in `reference/ascendra-ui/`: `hard-instructions.md`, `ui-reference.md`, `code-style.md` and `field-hint-guide.md`; `showcase-reference.md` only to find a specific pattern.
7. **For the close, `B{GG}.X`:** the plan in full, and `build/index.md`.
8. The code the checkpoint builds on: the files earlier Built blocks list for this chunk, and the module's published contract.

Don't read the brief, BRD documents or source material. The plan and the design already carry what they decided.

---

## Step 3 — Check before building

Work out three things, then show them in one message:

1. **Needs first:** what the checkpoint relies on (earlier checkpoints' code, a running database, the seed, a provider account), and whether each is met. Check it yourself where you can. Mark it "PO to confirm" where you can't.
2. **Needed from the PO:** secrets, accounts, or a choice. For each, say exactly where it goes and what it looks like.
3. **Not stated by the plan:** anything that would cross the checkpoint's limits (a second module, a second screen, more than about 400 changed lines), change the plan's scope, add a commons module or an outside service, or differ from the architecture note. For each, give your recommendation, including splitting the checkpoint.

> "Before building {ID} ({KIND}: {what it builds}):
>
> **Needs first:** [each item: met / not met / PO to confirm]
> **Needed from you:** [each item and where it goes, or 'nothing']
> **Not stated by the plan:** [each item with my recommendation, or 'nothing']
>
> Shall I start?"

Wait for PO response. Do not write any code until the PO says go.

If all three are empty, say so in one line and still wait.

**Questions:** ask in plain chat, never through a selection widget. Ask one question at a time when a choice needs its own answer, each with your recommendation.

**Secrets:** the PO puts them in a gitignored `.env` in the code repo, never in chat if avoidable, and never in a committed file. Give the PO the keys to add, blank. When the PO says it's ready, check each key is set and not a placeholder without printing any value.

---

## Step 4 — Build

**Branch:** at the unit's first checkpoint, create `group/{GG}-{slug}` in each code repo from the plan's base commit. Afterwards, stay on it. Make no commits.

Build what the checkpoint's row says, and nothing more.

**Rules while building:**
- **One layer, one chunk.** Code for a later checkpoint isn't written now, even when it's obvious. If this checkpoint can't work without it, stop and ask.
- **Stay within the architecture note.** If the checkpoint can't be built that way, stop and ask. Don't change the design silently.
- **Commons:** copy a module only with the commons tool, and only the parts named. If a copy needs a change, make the smallest change and add a row to `commons-changes.md`. Never re-copy a module listed there.
- **Screens:** start from the artboard source, keep its composition, text and states, and replace sample data with the real API. A library part used in the artboard is used here, whole. A screen that links to one not yet built leaves the link out.
- **Seed:** records the plan's Section 4 asks for are added through the modules' real contracts, never by inserting rows directly.
- **Small choices** the plan leaves open are yours. Record each one for the Differences line. Don't stop for them.
- **Stop and ask** before anything Step 3 would have listed, if it only turns up now. Above all, if the change is growing past the checkpoint's limits: propose a split instead of carrying on.
- **Use the domain's terms** in names, as the plan and the design use them.

---

## Step 5 — Verify

Every gate runs, in this order. A failed gate is fixed in the code, not in the test, before the next gate runs. Change a test only when the test itself is wrong, and say so in the Built block.

1. **Static:** the type check, linting, and the project's boundary checks, in each repo touched.
2. **Tests:** the checkpoint's own tests, as its row names, against the real thing (a real PostgreSQL as the app role, not a mock) unless the plan says otherwise. At least one of them must fail without this checkpoint's change: confirm it. Then every earlier test in the unit.
3. **Financial checks:** for any checkpoint that touches money records, Outstanding is recomputable from records alone and Allocations never exceed Payments.
4. **Design parity (S only):** render the artboard and the built page at the same size, the page with the seed, as the plan's header says. Look at both. List every difference in layout, text, parts and states. Fix what's wrong; any difference you keep goes in the Built block with its reason.
5. **Close only (`B{GG}.X`):** the unit's demo as a browser test, the browser tests of every closed unit, the financial checks. Then stop and ask before merging:

   > "{UNIT} passes its demo and the regression of {N} closed units. Shall I merge `group/{GG}-{slug}` into {MAIN} in {REPOS} and deploy to staging? The merge stays uncommitted for your review until you say."

   Wait for PO response. Merge and deploy only as the PO says.

If a gate can't pass without breaking something the plan made binding, stop:

> "{GATE} can't pass without [what would break]. My recommendation: [option]. How do you want to handle it?"

Wait for PO response.

---

## Step 6 — Check the tests can fail

Run this step for an R checkpoint that enforces a money rule, and for any checkpoint run with `--check-tests`. Otherwise skip it, and the Built block says "Tests not checked by breaking the code."

For each rule the checkpoint enforces (at most three):

1. Back up the file, then break the rule in the code.
2. Run the tests with a short timeout. Record which tests failed.
3. Restore the file from the backup, and confirm it matches the original byte for byte.

Every break must make at least one test fail. If one doesn't, a test is missing: add it, and say so. After the last break, run the full suite again and confirm everything passes.

---

## Step 7 — Record

Record the end time. In the plan:

1. Under the checkpoint's chunk table, add its Built block, at most 10 lines, in the form the template gives:
   - result in one sentence
   - files by area, as links
   - differences from the plan: every choice the plan didn't state, every change to a commons copy, every PO decision taken during the build; write "none" only if there were none
   - tests: how many passed and where; the parity result for S; breaks caught if Step 6 ran
   - hand check: up to 3 steps with expected results
   - a proposed commit message per repo
2. Set the checkpoint's Status to `Built`. Set the plan's Status to `In progress` if it was `Approved`.
3. Append to Section 8 anything a later checkpoint or unit must act on.
4. Update the unit's row in `build/index.md`: checkpoints built.
5. **Close only:** fill Section 7's Built block; record proposed architecture note changes there (don't edit the note); set the plan's Status and its index row to `Closed`, with the date; add a decision to the project's decision log if it records them by number.

Commit nothing. List every changed file in each repo, so the PO can review and commit.

---

## Step 8 — Report

```
CHECKPOINT BUILT — {PROJECT_CODE} {ID}
─────────────────────────────────────────
Kind:           {D / R / A / S / F / Close}: {what it builds}
Branch:         {repo} group/{GG}-{slug} (not committed)
Tests:          {N} passed, on {where}; {N} fail without the change
Parity:         {no differences / N kept, listed / not an S checkpoint}
Tests checked:  {yes, N breaks caught / no}
Time taken:     {minutes}, of which {minutes} waiting on the PO
Changed:        {N} files in {repos}; plan and build/index.md
Needs the PO:   {anything left open, or none}
─────────────────────────────────────────
Next: review the Built block and the diff, commit with the proposed
message, then run /build-checkpoint {PROJECT_CODE} {NEXT_ID}
```
