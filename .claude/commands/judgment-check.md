# Judgment Check

You are running a supplementary judgment-quality check against one already-generated artifact, using the corresponding chapter of `practitioner-guide/` as the source of every test applied. This is not a replacement for the artifact's own structural Verification checklist or its `/review-X` walkthrough — those check completeness and PO-confirmed correctness. This checks something neither of those does: whether the artifact's density, boundaries, and judgment calls actually match the tests the practitioner guide itself derived from reading every command and template in this framework closely. Run it before the artifact's `/review-X` command, as an extra pass that surfaces things worth raising during that walkthrough — never after Locked/Approved status, and never as a gate that blocks anything on its own.

Every finding this command produces is a question for the Product Owner to weigh, never a verdict. The guide's own material is explicit that a check like this — the same kind of model applying its own test to output it or a sibling model produced — cannot certify correctness the way a human's independent judgment can. This command's job is to make the guide's tests easy to apply, not to replace the PO's reading of them.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the argument and resolve the artifact type

`$ARGUMENTS` should contain a path to one existing artifact file (e.g. `projects/HARBORVIEW-INV-001/brds/brd-core-v1.md`).

If no path is given, ask:

> "Which artifact should I check? Provide the file path (e.g. `projects/{CODE}/brds/brd-core-v1.md`)."

Resolve the artifact's type from its path and filename, then its matching practitioner-guide chapter, from this table. The chapter itself — and `00-orientation.md`, the terminology map every chapter assumes rather than repeats — are only actually read in Analysis Mode (Step 3). Resolve Mode (Step 4) never needs either, since a finding's full text is already sitting in the artifact's own log.

| Artifact | Chapter |
|---|---|
| `brief.md` | `practitioner-guide/01-intake.md` |
| `domain/*.md` (domain knowledge) | `practitioner-guide/02-domain-discovery.md` |
| `brds/brd-*.md` | `practitioner-guide/03-brd-discovery-and-scope-lock.md` |
| `epics/EPIC-*.md` | `practitioner-guide/04-product-structuring-epics.md` |
| `screens/screen-design.md` | `practitioner-guide/05-screen-design.md` |
| `architecture/arch-v1.md` | `practitioner-guide/06-architecture.md` |
| `stories/EPIC-*/US-*.md`, `sprints/sprint-*.md` | `practitioner-guide/07-sprint-planning.md` |
| `story-plans/*.md`, `pr/*.md` | `practitioner-guide/08-development.md` |
| `sprints/*-uat-checklist.md`, `test-reports/*.md` | `practitioner-guide/09-qa-uat.md` |
| `releases/*.md` | `practitioner-guide/10-release-support-hotfix.md` |

Every row above is live — the pilot on BRD/Epics/Architecture validated the mechanism (see the simulation record referenced in `practitioner-guide/README.md`), and it has since been extended to the remaining seven rows, each with the reserved template section and a worked example. Every corresponding template and `.example.md` pair carries the one Density & Judgment Log section this command writes to.

If the path doesn't match any row, stop:

> "`{path}` doesn't match a known artifact type. Judgment Check currently covers: brief, domain knowledge, BRD, epics, screen design, architecture, stories/sprint plan, story plan/PR description, UAT checklist/test execution report, release notes."

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Artifact file exists | Read the path from Step 1 | "`{path}` not found. Check the path and try again." |
| Artifact has its reserved Density & Judgment Log section (or, for release notes only, doesn't yet — expected pre-first-run) | Look at the artifact content already loaded for the check above — no separate template read; per `C-013a` this command doesn't read `projects/TEMPLATE/`. Every covered type except release notes reserves this section from generation onward; release notes only gains it once `/judgment-check` first runs. Missing it outside that one expected case is a framework defect, not something to route around by inventing a section shape on the spot | "`{path}` has no Density & Judgment Log section at all, and this isn't release notes' expected pre-first-run state — this shouldn't happen for a covered artifact type; flag it rather than improvising a section shape." |
| Artifact isn't already past its own approval gate | Read the artifact's own status field directly (no template lookup needed) and compare against the table below. Artifact types with no row have no gate concept — always proceed for these. | "`{path}` is already `{status}`. Judgment Check runs *before* that gate closes, not after — editing an already-approved/locked artifact here would bypass `/assess-change`'s version-bump and Document Control trail. Run `/assess-change` if this content genuinely needs to change; otherwise run Judgment Check while the artifact is still pre-approval." |

**Per-type gate status to check** (field name and the value(s) that mean "already past the gate"):

| Artifact | Status field | Already-gated value(s) |
|---|---|---|
| `brief.md` | Content Status | Approved, Locked, Superseded |
| `brds/brd-*.md` | Status | Approved |
| `epics/EPIC-*.md` | Status | Approved, In Progress, Done |
| `screens/screen-design.md` | Status | Approved |
| `architecture/arch-v1.md` | Status | Approved, Locked |
| `stories/EPIC-*/US-*.md` | Status | Approved, In Progress, Merged, Done |
| `sprints/sprint-*.md` | Status | Active, Complete |
| `story-plans/*.md` | Plan Status | Confirmed |

`domain/*.md` (no gate — PROJECT-LIFECYCLE.md: "reference material, not a locked artifact"), `sprints/*-uat-checklist.md`, `pr/*.md`, `test-reports/*.md`, and `releases/*.md` have no status field to check — always proceed for these.

Do not proceed past a failed check.

---

## Step 2 — Check for open findings before doing anything else

Read the target artifact once. Look at its **Density & Judgment Log** section — reserved by the template at generation time for every artifact type except release notes, so it's normally always present, holding only the placeholder pointer line and no table before this command's first run. Release notes are the one exception (their own template's client-facing rule keeps this section fully dynamic, per that template's own note) — there, the section simply doesn't exist at all until the first run.

- **No table yet** (section holds only the placeholder line, or doesn't exist at all for release notes) **, or every row's Outcome column is filled in** (Confirmed as-is / Revised / Acknowledged tradeoff) — there is nothing open. Continue to **Step 3 (Analysis Mode)**.
- **At least one row's Outcome column still reads `Open`** — something from a prior run was never closed out. Skip Step 3 entirely and go straight to **Step 4 (Resolve Mode)**.

These two modes are mutually exclusive per invocation, always. A run that surfaces new findings never also resolves them in the same breath, and a run that resolves old findings never also looks for new ones in the same breath — that's what keeps this command from ever piling a second batch of findings on top of an unaddressed first batch. To fully close the loop on one artifact, the PO always invokes this command at least twice: once to surface, once (or more, if a finding drags) to resolve.

---

## Step 3 — Analysis Mode: apply the chapter's tests

Reached only when Step 2 found nothing open. Read exactly three files, nothing more — context is a finite resource:

1. `practitioner-guide/00-orientation.md` — in full, every time, regardless of chapter. It's short (barely 60 lines), and every chapter without exception builds on its terminology map (Scope vs. Priority vs. Layer, and the other base pairs) and its "scores and thresholds you can't reproduce" caution rather than restating either — a chapter's own "Terminology recap" section always says some form of "go there for the base definitions, not here." Skipping it isn't cheap-vs-thorough; it's applying a test without the definitions the chapter itself assumes you already have.
2. The one matching practitioner-guide chapter resolved in Step 1 — in full.
3. The one target artifact from `$ARGUMENTS` — already read once in Step 2; no need to re-read it.

Do not read any other practitioner-guide chapter. Do not follow every cross-reference the chapter makes to a decision record, another `.example.md` file, or another chapter — pull in a secondary file only when a specific test you are about to apply explicitly depends on it (e.g. the BRD chapter's Section 5.21-vs-15 test genuinely requires the exact wording in `FW-024`, which is short; reading `FW-024` for that one test is fine — reading every ADR the chapter happens to cite is not). Do not read other artifacts in the same project (other epics, the BRD an epic traces to, etc.) even if the chapter's own "Harborview in practice" section references them — this command checks the one artifact in front of it, not the whole pipeline.

Work through the chapter's decision heuristics, density-guidance sections, and "Common mistakes" list, in the order they appear in the chapter. For each one that is genuinely checkable against this specific artifact's actual content (not every heuristic in a chapter applies to every artifact — a chapter's "when to run X live vs. skip it" guidance, for instance, is a process decision made before this artifact existed, not something checkable against the finished document):

- Quote or closely paraphrase the specific test from the chapter.
- Quote the specific spot in the artifact the test applies to.
- State plainly what the test would flag if applied strictly, and why.
- Frame it as a question for the PO, not a finding of fact — e.g. "REQ-014 reads as one sentence but covers three separately-shippable behaviors (per the REQ-writing density test in Chapter 3) — is this intentional, or should it split?" never "REQ-014 violates the density rule."

Do not invent a test the chapter doesn't state. Do not apply a test from memory of a different chapter. If a chapter section doesn't map to anything checkable in this artifact, skip it silently — do not manufacture a finding to have something to report.

**If this produced no findings:**

> "No findings — this {artifact type} doesn't raise anything against {chapter name}'s tests. This isn't the same as a clean bill of health on everything the chapter covers; it means nothing checkable against this artifact's actual content stood out. Continue with `/review-{artifact-type}` as normal."

Stop here — do not write anything to the artifact; leave the placeholder pointer line exactly as it is.

**If this produced findings:** do not present them to the PO in this invocation — logging and resolving are always two separate invocations, per Step 2's branch. Continue to Step 5 to log them, then Step 6 to report.

---

## Step 4 — Resolve Mode: walk the PO through open findings, one at a time

Reached only when Step 2 found something open. Present only the rows whose Outcome column still reads `Open` — never a row that already carries a resolution outcome, even one from an older block:

> "Finding {N} of {total open} — {one-line summary}:
>
> {the finding, exactly as logged}
>
> **(a) Confirmed as-is** — this is intentional, no change needed
> **(b) Revise** — tell me what should change and I'll update it now
> **(c) Acknowledged tradeoff** — I see the concern, keeping it anyway, here's why"

Wait for PO response. On **(b)**, make the described change to the artifact directly (the same way `/review-brd`'s walkthrough applies agreed changes immediately). On any of **(a)**/**(b)**/**(c)**, immediately write that one finding's Outcome and Detail cells per Step 5's mechanics before moving to the next finding — do not batch writes until the whole walkthrough finishes, the same discipline `/review-brd`'s own Requirement Log already follows ("update the corresponding row immediately. Do not wait until the end of the session"). This is what makes an interrupted session safe: if the PO stops partway — session ends, connection drops, anything — every finding already answered is already saved, and the next invocation's Step 2 check finds exactly the rows still open and resumes Resolve Mode against only those, nothing repeated, nothing lost.

Once every open row from this invocation has an outcome, continue to Step 6 to report. Do not run Step 3's analysis in this same invocation, even once everything currently open is resolved — the next fresh pass happens on the PO's next invocation of this command.

---

## Step 5 — Write the Density & Judgment Log

The section itself is reserved by the artifact's own template at generation time, at the number given there (this command does not renumber a template on the fly), and normally always exists — holding only the template's placeholder pointer line before this command's first run. Release notes are the one exception: the section doesn't exist at all until this step first creates it, per that template's own note.

**Analysis Mode (Step 3 produced findings) — append a new dated block:**

```markdown
*Generated by `/judgment-check` against `{chapter path}` on {date}. Findings are questions raised by the guide's own tests, not verdicts.*

| # | Test (chapter source) | Location | Finding | Outcome | Detail |
|---|---|---|---|---|---|
| 1 | {short test name} | {artifact location} | {the finding, as raised} | Open | — |
```

If the section holds only the placeholder line, replace that line with this one block. If the section doesn't exist yet at all (release notes only), create it fresh with this one block, at the number its template reserves. If earlier blocks already exist (from prior analysis passes, all fully resolved by now per Step 2's gate), append this as a new block below them — its own dated header line, its own `#` numbering starting again at 1. Never touch an earlier block's header or rows.

**Resolve Mode (per finding, immediately after the PO answers it in Step 4 — never batched) — fill in that row's Outcome and Detail columns:**

Update the finding's **Outcome** cell to the Product Owner's choice (`Confirmed as-is` / `Revised` / `Acknowledged tradeoff`) and its **Detail** cell to the Product Owner's own words — do not add a new row, do not touch the Finding cell, do not touch the block's dated header line, do not touch any other row.

A row, once its Outcome is filled in, is permanently closed. A later run that flags a new issue at the same spot in the document gets its own new row in its own new block — never a re-edit of this one.

---

## Step 6 — Report to the PO

**After Analysis Mode:**

```
JUDGMENT CHECK — {artifact type}
─────────────────────────────────────────
Artifact:      {path}
Chapter:       {chapter path}
New findings:  {N} logged as Open
─────────────────────────────────────────
```

> "{N} new findings logged as Open in the Density & Judgment Log. Run `/judgment-check {path}` again to review and close them before `/review-{artifact-type}` — this run only surfaced them, it didn't ask you about them yet."

**After Resolve Mode:**

```
JUDGMENT CHECK — {artifact type}
─────────────────────────────────────────
Artifact:      {path}
Chapter:       {chapter path}
Findings:      {N} resolved this session{, M still Open if the session ended early}
─────────────────────────────────────────
```

If every open row got an outcome:

> "This reflects {chapter name}'s own tests, applied by the same kind of model that wrote the guide — treat these as things you've now weighed, not as a certification. Continue with `/review-{artifact-type}` for the structural walkthrough."

If the session ended with rows still Open (the PO stopped partway):

> "{N} resolved, {M} still Open — nothing lost, they're saved as answered so far. Run `/judgment-check {path}` again any time to pick up exactly where this left off."
