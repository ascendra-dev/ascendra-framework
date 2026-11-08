# Build Story

Builds one story from a slice spec into code, runs its tests, and writes its build summary. It replaces `/gen-story-plan`, `/implement-story`, `/verify-story` and `/gen-pr-description` for projects delivered slice by slice, where stories live in a slice spec and the architecture is a short note, not a Locked architecture document.

It runs once per story, in the order the slice spec gives. It produces code on a new branch in the code repo, a build summary file next to the slice spec, and a **Built:** line under the story. It never commits, pushes or opens a PR. The PO does that.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code and a story ID. Optional: `--check-tests`.

- **Project code** — e.g. `ASCENDRA-PAY-004`
- **Story ID** — e.g. `US-00-02`
- **`--check-tests`** — after the tests pass, break the code on purpose to prove the tests can fail (Step 6). Off unless given.

If both required fields are missing:

> "To build a story I need:
> 1. **Project code** — the project folder under `projects/` (e.g. `ASCENDRA-PAY-004`)
> 2. **Story ID** — the story in a slice spec (e.g. `US-00-02`)"

Wait for PO response.

If only the story ID is missing:

> "Which story should I build? Provide the story ID (e.g. `US-00-02`)."

Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Story exists | Search `projects/{CODE}/slices/*/spec.md` for a `### {STORY_ID}:` heading | > "I can't find {STORY_ID} in any slice spec under `projects/{CODE}/slices/`. Check the ID." |
| Not already built | The story has no **Built:** line | > "{STORY_ID} already has a Built: line. Rebuilding it would overwrite its summary. Do you want to rebuild it?" |
| Earlier stories built | Every story above it in the same spec has a **Built:** line | > "{STORY_ID} comes after {EARLIER_IDS}, which aren't built. The spec says order matters. Build them first, or tell me to continue anyway." |
| Architecture note exists | `projects/{CODE}/architecture/architecture-note.md` exists | > "There is no architecture note at `projects/{CODE}/architecture/architecture-note.md`. `/build-story` needs it." |

On any failure, show the message and wait for PO response.

---

## Step 2 — Read files

Record the start time now. It goes in the build summary, because the cost of each story is measured.

Read, in this order:

1. The slice spec's story guide, the section just above the first story ("How to read a story"). It sets which headings are binding.
2. The story itself, in full. **Acceptance, Decisions, Tests and Out of scope are binding. Needs first is binding. Notes are context. Artifacts names are indicative.**
3. The slice spec's sections that the story refers to by number, such as rules (`SL00-R2`), records and contracts, and screens.
4. `projects/{CODE}/architecture/architecture-note.md` — the decisions every story must stay within. Read in full.
5. `projects/{CODE}/architecture/commons-changes.md` — which commons copies we've changed, so none of them is re-copied.
6. `projects/{CODE}/delivery-approach.md`, the section on using commons — only if the story copies or changes a commons module.
7. Any document the story links to in Needs first, Acceptance or Decisions — such as an API contract. These are binding where the story says so.
8. The build summaries of earlier stories in the same slice (`us-*-build-summary.md` next to the spec) — only their "Differences from the story" sections, to stay consistent with choices already made.
9. **Only for a story with screens:** in `reference/ascendra-ui/`, read `hard-instructions.md`, `ui-reference.md`, `code-style.md` and `field-hint-guide.md`. Read `showcase-reference.md` only to find the pattern for a specific screen.

Do not read the domain knowledge, the brief, the BRD documents or the source material unless the story links to them. The story and the architecture note already carry what they decided.

---

## Step 3 — Check before building

Work out three things, then show them in one message:

1. **Needs first:** each item, and whether it's met. Check it yourself where you can, such as a repo or a container that exists. Mark it "PO to confirm" where you can't, such as an account.
2. **Needed from the PO:** secrets, connection strings, accounts, or a choice between providers or modes. For each one, say exactly where it goes and what it looks like.
3. **Bigger than an implementation detail:** anything the story doesn't state that would change scope, add a commons module, add a dependency on a new outside service, or differ from the architecture note. For each, give your recommendation.

> "Before building {STORY_ID}:
>
> **Needs first:** [each item — met / not met / PO to confirm]
> **Needed from you:** [each item and where it goes, or 'nothing']
> **Not stated by the story:** [each item with my recommendation, or 'nothing']
>
> Shall I start? If anything above is wrong, tell me what to change."

Wait for PO response. Do not write any code until the PO says go.

If all three are empty, say so in one line and still wait.

**Questions in this step:** ask in plain chat, never through a selection widget. If a choice needs a separate answer, ask one question at a time, each with your recommendation.

**Secrets:** the PO puts them in a gitignored `.env` in the code repo, never in chat if avoidable, and never in a committed file. Give the PO an `.env.example` with every key blank. When the PO says the `.env` is ready, check it yourself without printing any secret: each key is set, nothing is left as a placeholder, and the user names, hosts and ports are as intended.

---

## Step 4 — Build

**Branch:** in the code repo, create and switch to `story/{story-id-lowercase}`, e.g. `story/us-00-02`. If the repo doesn't exist yet and the story creates it, run `git init` first. Make no commits.

**Code repo:** the one the story or architecture note names. If neither names one, ask.

Build what the story's Acceptance and Artifacts describe, and nothing in Out of scope.

**Rules while building:**
- **Stay within the architecture note.** If the story can't be built that way, stop and ask. Don't change the design silently.
- **Commons:** copy a module only with the commons tool, and only the parts the story names. If a copied module needs a change, make the smallest change in our copy, and add a row to `commons-changes.md`. Never re-copy a module listed there.
- **Small choices the story leaves open** are yours to make. Record each one for "Differences from the story". Don't stop for them.
- **Stop and ask** before anything that Step 3 would have listed, if it only turns up now.
- **Use the domain's terms** in names, as the slice spec uses them.
- **No provider calls inside a database transaction.** The architecture note owns the transaction rules.

---

## Step 5 — Test

Tests are always written and run. They are not optional.

- Write every test in the story's Tests heading. If the story has none, write the fewest tests that prove each acceptance point, and list them in the summary.
- Run them against the real thing the story names, such as a real database or a provider's pooler, not a mock, unless the story says otherwise. Where the story says to run locally first and then on staging, do both.
- Run the type check and any linting the repo has.
- If a test fails, fix the code, not the test. Change a test only when the test itself is wrong, and say so in the summary.

If a test can't pass without breaking a binding part of the story, stop:

> "{TEST} can't pass without [what would break]. My recommendation: [option]. How do you want to handle it?"

Wait for PO response.

---

## Step 6 — Check the tests can fail (only with `--check-tests`)

Skip this step unless `--check-tests` was given. If skipped, the summary says "Tests not checked by breaking the code."

For each rule the story most depends on (at most three):

1. Back up the file, then break the rule in the code. For example, let a write escape the transaction, or let a setting outlive it.
2. Run the tests with a short timeout, so a hang can't stall the run. Record which tests failed.
3. Restore the file from the backup, and confirm it matches the original byte for byte before the next one.

Every break must make at least one test fail. If one doesn't, a test is missing: add it, and say so in the summary. After the last break, run the full suite again and confirm everything passes.

---

## Step 7 — Write the build summary

Record the end time. Write `us-{NN}-{NN}-build-summary.md` next to the slice spec. It must fit on one screen, apart from its results table, and contain no code.

**Header table:** Story (linked to its heading in the spec), Built (date and time taken), Where (code repo and branch), Result (one sentence).

**1. What was built, in review order:** data (tables, migrations), rules (where each business rule is enforced), contracts, endpoints, screens, tests. Each line gives what it is, where it lives, and the acceptance point or rule it serves. Then a table: each test, and where it passed.

**2. Differences from the story:** everything that differs from Artifacts, every choice the story didn't state, every change to a commons copy, every PO decision taken during the build, and anything found that a later story must act on. This is the part the PO reviews most carefully, so leave nothing out.

**3. Check it by hand:** up to about 8 numbered steps, each with its expected result, taken from the acceptance criteria. Use dev login locally, or staging. A story with no screen gets commands and their expected output.

---

## Step 8 — Record

1. Under the story in the spec, after Size, add: `**Built:** [build summary](us-{NN}-{NN}-build-summary.md), {date}.`
2. If the story proved or disproved something the architecture note lists as unproven, record the result there, with the fallback if it failed.
3. If the build changed anything the architecture note decides, don't edit the note. Add it to the slice spec's "Architecture changes from this slice" section, for the slice review.
4. Run the project's link check, if it has one.

Commit nothing in either repo. List every changed file, so the PO can review and commit.

---

## Step 9 — Report

```
STORY BUILT — {PROJECT_CODE} {STORY_ID}
─────────────────────────────────────────
Branch:          {code repo} story/{story-id} (not committed)
Tests:           {N} passed, on {where}
Tests checked:   {yes, N breaks caught / no}
Time taken:      {minutes}, of which {minutes} waiting on the PO
Summary:         {path to build summary}
Also changed:    {spec Built line, architecture note, commons-changes.md}
Needs the PO:    {anything left open, or none}
─────────────────────────────────────────
Next: review "Differences from the story", then commit both repos.
```
