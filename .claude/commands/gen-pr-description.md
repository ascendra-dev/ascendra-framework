# Generate PR Description

You are generating a pull request description for a single, already-verified user story. This runs after `/verify-story` reports all acceptance criteria passing, and before the Engineering Agent raises the PR. It produces the exact text the PO will read at Step 20 (PO reviews PR) — the PO should be able to understand what changed, why, and how to verify it without opening a single file.

One story = one PR. If the change spans more than one story, stop and tell the PO — this command does not merge or split scope on its own.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a story file path. Example:

```
/gen-pr-description projects/ASCENDRA-PAY-001/stories/EPIC-004/US-04-001-register-payer-manually.md
```

If missing, ask:
> "Which story is this PR for? Provide the story file path (e.g. `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/US-{epic}-{seq}-{slug}.md`)."

Wait for PO response.

Derive: **PROJECT_CODE**, **Story ID**, **EPIC-{NNN}** from the path.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Story status is `In Progress` or `PR Created` | Read the story's `**Status:**` field | "Story `{ID}` has status `{status}`. A PR description is generated after implementation and verification, not before. Run `/implement-story` and `/verify-story` first." |
| Verification report exists and passed | `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md` exists with **Overall result: PASS** | "No passing verification report found for `{ID}` at `projects/{PROJECT_CODE}/test-reports/US-{epic}-{seq}-verification.md`. Run `/verify-story` first — a PR description is not generated against unverified or failing work." |

---

## Step 2 — Read required files

1. `projects/TEMPLATE/pr/pr-description.template.md` — the template every PR description follows. Read Part 1 and Part 2 (Verification) in full.
2. The story file — Section 1 (Story Statement), Section 2 (FR References), Section 3 (Acceptance Criteria), Section 4 (Out of Scope)
3. The verification report from Step 1.5 — AC Results table and Test Evidence section feed Section 4 (How to Test) below
4. `projects/{PROJECT_CODE}/architecture/arch-v1.md` Section 3.1 (Project Structure) — to resolve the target repo(s), same convention as `/implement-story` Step 3 (`../{repo-name}`)

---

## Step 3 — Read the actual code change

Run `git -C ../{repo-name} diff main --stat` (and for the Web repo too, if this story touched both) to get the real list of changed files. Do not infer "What Changed" from the story's ACs alone — the PR description must reflect the actual diff, not the intended scope.

If the repo has no commits yet on a feature branch (nothing to diff against `main`), ask:
> "No diff found against `main` in `../{repo-name}`. Has this story's work been committed to a branch yet? If not, commit first — a PR description needs a real diff to describe."

Wait for PO response.

---

## Step 4 — Gather missing inputs

Ask **all at once**, only what's missing:

| Input | What to ask if missing |
|-------|------------------------|
| **Ticket tracker** | "Does this project use an external ticket tracker (Jira/Linear/GitHub Issues) alongside these story files? If yes, give the ticket ID for this story (e.g. `ASCE-142`). If no, type 'none' — the PR will reference the story file directly instead of a `Closes`/`Refs` line." |

Wait for PO response.

---

## Step 5 — Determine PR title and Type of Change

Infer **Type of Change** from the story: a story adding a capability that didn't exist before is `feat`; a story correcting a defect found during verification is `fix`; a pure refactor with no behaviour change is `refactor`. State your inference before proceeding — if genuinely ambiguous, ask.

Construct the PR title: `type(scope): subject` — `scope` is the module/feature area from the story's parent epic; `subject` is a short imperative phrase derived from the story title.

---

## Step 6 — Populate the PR description

Follow `projects/TEMPLATE/pr/pr-description.template.md` exactly. Populate every section:

- **Summary:** 1–2 sentences — what changed and why, from the story statement and FR references
- **Ticket link:** `Closes {ticket-id}` / `Refs {ticket-id}` if Step 4 gave one; otherwise `Story: {story file path}` in its place
- **Type of Change:** the single box from Step 5
- **What Changed:** 3–7 bullets built from the real diff in Step 3 — `{file or module} — {what changed}`, not a restatement of the ACs
- **How to Test:** numbered steps built from the verification report's AC Results and Test Evidence (Step 2) — the reviewer follows exactly what was already proven to pass, not a fresh set of steps
- **Screenshots:** include placeholder markers for any Web story; delete the section entirely for API-only stories
- **Pre-Merge Checklist:** tick each item genuinely confirmed from the diff and verification report; use `— N/A` for inapplicable items, never leave unchecked without stating why

**Output format:** the generated PR description does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Remove the `## Part 1 — Template` label — the content starts directly after `## Document Control`. The `## Part 2 — Verification` checklist is not carried into the output at all, renamed or otherwise — it is an internal pre-write self-check only. This matches `projects/TEMPLATE/pr/pr-description.example.md`, which has neither a `Part 1` label nor any Verification section.

---

## Step 7 — Real-world framing and confirmation

Before writing the file, present:

> "**PR for {Story ID}: {Story Title}**
> Title: `{PR title}`
> This PR: {1–2 sentence plain description of what a reviewer will see/get if they merge this}
>
> Does this look right? Tell me what to correct if not."

Wait for PO response. Do not write any files until confirmed.

---

## Step 8 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/pr/US-{epic}-{seq}-pr-description.md`

Create the `pr/` directory if it does not exist.

---

## Step 9 — Report to the user

```
PR DESCRIPTION GENERATED — {Story ID}
─────────────────────────────────────────
Title:        {PR title}
Type:         {feat/fix/refactor/test/chore/perf}
File:         projects/{PROJECT_CODE}/pr/US-{epic}-{seq}-pr-description.md
Checklist:    [N]/[N] items confirmed
─────────────────────────────────────────
Next step:
Raise the PR using this file as the body, e.g.:
gh pr create --title "{PR title}" --body-file projects/{PROJECT_CODE}/pr/US-{epic}-{seq}-pr-description.md
Then: PO reviews the PR (Step 20 — manual gate).
```
