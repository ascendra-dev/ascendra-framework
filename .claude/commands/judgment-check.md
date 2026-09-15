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

Resolve the artifact's type from its path and filename, then its matching practitioner-guide chapter, from this table — the only two things this command ever reads:

| Artifact | Chapter | Pilot status |
|---|---|---|
| `brief.md` | `practitioner-guide/01-intake.md` | Not yet piloted |
| `domain/*.md` (domain knowledge) | `practitioner-guide/02-domain-discovery.md` | Not yet piloted |
| `brds/brd-*.md` | `practitioner-guide/03-brd-discovery-and-scope-lock.md` | **Piloted** |
| `epics/EPIC-*.md` | `practitioner-guide/04-product-structuring-epics.md` | **Piloted** |
| `screens/screen-design.md` | `practitioner-guide/05-screen-design.md` | Not yet piloted |
| `architecture/arch-v1.md` | `practitioner-guide/06-architecture.md` | **Piloted** |
| `stories/US-*.md`, `sprints/sprint-*.md` | `practitioner-guide/07-sprint-planning.md` | Not yet piloted |
| `story-plans/*.md`, `pr/*.md` | `practitioner-guide/08-development.md` | Not yet piloted |
| `sprints/*-uat-checklist.md`, `test-reports/*.md` | `practitioner-guide/09-qa-uat.md` | Not yet piloted |
| `releases/*.md` | `practitioner-guide/10-release-support-hotfix.md` | Not yet piloted |

If the path doesn't match any row, stop:

> "`{path}` doesn't match a known artifact type. Judgment Check currently covers: brief, domain knowledge, BRD, epics, screen design, architecture, stories/sprint plan, story plan/PR description, UAT checklist/test execution report, release notes."

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Artifact file exists | Read the path from Step 1 | "`{path}` not found. Check the path and try again." |
| Artifact type is piloted | Cross-check the Step 1 table's Pilot status column | "Judgment Check is not yet extended to {artifact type} — currently piloted on BRD, Epics, and Architecture only, while the mechanism is validated in real use. Ask the Product Owner if this should be extended next." |

Do not proceed past a failed check.

---

## Step 2 — Read exactly two files, nothing more

**Context is a finite resource — this command reads only what this one invocation needs, never the whole guide, never the whole project.** Read:

1. The one matching practitioner-guide chapter resolved in Step 1 — in full.
2. The one target artifact from `$ARGUMENTS` — in full.

Do not read any other practitioner-guide chapter. Do not read `00-orientation.md` unless the target chapter's own text explicitly directs you to a specific part of it for a specific test (most chapters link to it for background only — that is not a directive to read it here). Do not follow every cross-reference the chapter makes to a decision record, another `.example.md` file, or another chapter — pull in a secondary file only when a specific test you are about to apply explicitly depends on it (e.g. the BRD chapter's Section 5.21-vs-15 test genuinely requires the exact wording in `FW-024`, which is short; reading `FW-024` for that one test is fine — reading every ADR the chapter happens to cite is not). Do not read other artifacts in the same project (other epics, the BRD an epic traces to, etc.) even if the chapter's own "Harborview in practice" section references them — this command checks the one artifact in front of it, not the whole pipeline.

---

## Step 3 — Apply the chapter's tests

Work through the chapter's decision heuristics, density-guidance sections, and "Common mistakes" list, in the order they appear in the chapter. For each one that is genuinely checkable against this specific artifact's actual content (not every heuristic in a chapter applies to every artifact — a chapter's "when to run X live vs. skip it" guidance, for instance, is a process decision made before this artifact existed, not something checkable against the finished document):

- Quote or closely paraphrase the specific test from the chapter.
- Quote the specific spot in the artifact the test applies to.
- State plainly what the test would flag if applied strictly, and why.
- Frame it as a question for the PO, not a finding of fact — e.g. "REQ-014 reads as one sentence but covers three separately-shippable behaviors (per the REQ-writing density test in Chapter 3) — is this intentional, or should it split?" never "REQ-014 violates the density rule."

Do not invent a test the chapter doesn't state. Do not apply a test from memory of a different chapter. If a chapter section doesn't map to anything checkable in this artifact, skip it silently — do not manufacture a finding to have something to report.

---

## Step 4 — Walk the PO through findings, one at a time

If Step 3 produced no findings:

> "No findings — this {artifact type} doesn't raise anything against {chapter name}'s tests. This isn't the same as a clean bill of health on everything the chapter covers; it means nothing checkable against this artifact's actual content stood out. Continue with `/review-{artifact-type}` as normal."

Stop here if there are no findings — do not write empty sections to the artifact.

If Step 3 produced findings, present each one:

> "Finding {N} of {total} — {one-line summary}:
>
> {the quoted test from the chapter}
>
> {the specific spot in the artifact this applies to}
>
> {the question, per Step 3}
>
> **(a) Confirmed as-is** — this is intentional, no change needed
> **(b) Revise** — tell me what should change and I'll update it now
> **(c) Acknowledged tradeoff** — I see the concern, keeping it anyway, here's why"

Wait for PO response. On **(b)**, make the described change to the artifact directly (the same way `/review-brd`'s walkthrough applies agreed changes immediately), then continue to the next finding. On **(a)** or **(c)**, record the response verbatim and continue.

---

## Step 5 — Write the two sections

**Density & Judgment Findings** (regenerated in full every run — this is this run's output, not a durable record):

```markdown
## {N}. Density & Judgment Findings

*Generated by `/judgment-check` against `{chapter path}` on {date}. Findings are questions raised by the guide's own tests, not verdicts — see the Resolution section below for how each was actually handled.*

| # | Test (chapter source) | Location | Finding |
|---|---|---|---|
| 1 | {short test name} | {artifact location} | {one-line finding} |
```

If this section already exists from a prior run, replace it wholesale with this run's output — do not append to a stale list.

**Density & Judgment Resolution** (append-only, every run adds to this, never replaces it — mirrors `Document Control`'s and `Change History`'s append-only discipline elsewhere in this framework):

```markdown
## {N+1}. Density & Judgment Resolution

| Date | Finding | PO Response | Detail |
|------|---------|-------------|--------|
| {date} | {short test name} | Confirmed as-is / Revised / Acknowledged tradeoff | {the PO's own words from Step 4} |
```

Append one row per finding from this run beneath any existing rows. Never delete or overwrite a prior run's rows, even if a later run's finding covers the same spot in the artifact — a changed circumstance is a new row, not an edit to the old one.

Write both sections to the artifact file at the section numbers reserved for this artifact type (see each artifact's own template for the exact reserved numbers — this command does not renumber a template on the fly).

---

## Step 6 — Report to the PO

```
JUDGMENT CHECK — {artifact type}
─────────────────────────────────────────
Artifact:      {path}
Chapter:       {chapter path}
Findings:      {N} raised, {N} resolved this session
─────────────────────────────────────────
```

> "This reflects {chapter name}'s own tests, applied by the same kind of model that wrote the guide — treat these as things you've now weighed, not as a certification. Continue with `/review-{artifact-type}` for the structural walkthrough."
