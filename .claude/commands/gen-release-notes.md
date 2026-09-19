# Generate Release Notes

You are generating client-facing release notes for a completed sprint. Release notes describe every user-visible change in plain language — what users can now do, what was fixed, and what they need to know before or after deployment. They are reviewed by the Product Owner before the production deployment is approved.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a project path, sprint number, and version string. Examples:

```
/gen-release-notes projects/ASCENDRA-PAY 01 1.0.0
/gen-release-notes projects/ASCENDRA-PAY 07 1.6.0
```

Extract:
- **Project path** — e.g. `projects/ASCENDRA-PAY`
- **PROJECT_CODE** — from project path
- **SPRINT_NUMBER** — strip whitespace, then zero-pad to two digits (e.g. `1` → `01`, `7` → `07`, `01` stays `01`). State the normalised value: "Sprint number: {NN}"
- **Version** — semantic version string (e.g. `1.0.0`, `1.6.0`)

If any are missing, ask for them all at once:
> "Provide: (1) project path, (2) sprint number, (3) version string (e.g. `1.0.0`)."

---

## Step 2 — Gate check: sprint must be Complete

Read `projects/{PROJECT_CODE}/stories/index.md`. Find Sprint {NN} in Section 8 (Sprint Planning table). Check the Status column.

- If Status is `Complete` → proceed
- If Status is not `Complete` → stop:
  > "Sprint {NN} status is `{status}`. Release notes can only be generated after the sprint is Complete. Run `/close-sprint` and then `/update-status projects/{PROJECT_CODE} sprint {NN} Complete` first."

---

## Step 3 — Read required files

Read these files in full:

1. `projects/TEMPLATE/releases/release-notes.template.md` — the template you must follow. Read all sections including the AI Guide notes.
2. `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md` — the sprint plan: sprint goal, story list, retrospective (carry-forward items)
3. `projects/{PROJECT_CODE}/brief.md` — project name and client name for the document header
4. `projects/{PROJECT_CODE}/architecture/arch-v1.md` — Section 7.3 (Environment Variables) and any migration notes, to populate Deployment Notes

---

## Step 4 — Read all sprint story files

From the sprint plan document, extract the full list of story IDs in this sprint (all Must Have + included Should Have stories that are Done or Merged).

For each story ID, read the story file at `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/{story-filename}.md`.

Extract:
- **Story title** — to know what was built
- **Story statement** (Section 1) — for user outcome context
- **Acceptance criteria** (Section 3) — to understand what the system now does
- **Epic** — to group by feature area

---

## Step 5 — Group stories by feature area

Group the stories by their parent epic title. These become the feature area headings in Section 1 (What's New) of the release notes.

Examples:
- EPIC-001 Platform Foundation → skip (infrastructure stories; no user-visible changes)
- EPIC-002 Payer Management → "Payer Management"
- EPIC-003 Invoice Management → "Invoice Management"

Skip infrastructure stories (platform/developer persona) in Section 1. If they involve a deployment step (migration, seed), include only their deployment note in Section 5 (Deployment Notes).

---

## Step 6 — Write the release notes

Follow `projects/TEMPLATE/releases/release-notes.template.md` exactly. Populate every section:

### Section 1 — What's New

For each feature area group: one heading, then a bullet per user-visible capability delivered. Write in outcome language.

**Do:**
- "You can now create and send invoices directly from the dashboard."
- "Payment links can be generated for any open invoice and shared with the payer via email."
- "Payer accounts can be imported from a CSV file."

**Don't:**
- "The `POST /api/v1/invoices` endpoint now accepts a `payerId` field." (technical)
- "EPIC-003 story US-01-012 delivers invoice creation." (internal)
- "The InvoiceModule was extended with a create service." (implementation)

### Section 2 — Improvements

Only if there are user-perceptible improvements to existing behaviour. For a first release (v1.x.0) this is often empty.

### Section 3 — Bug Fixes

Only if defects from a previous sprint were fixed in this sprint. For a first sprint this is always empty.

### Section 4 — Known Limitations

Check the sprint retrospective (carry-forward items) and any Should Have stories that were deferred. List them here as known limitations with their planned resolution sprint.

### Section 5 — Deployment Notes

From the architecture document Section 7.3: list all environment variables that are new or changed in this sprint. From the sprint stories: identify any that required a migration (`npm run db:migrate`), seed data (`npm run db:seed`), or cache flush. Describe each deployment step explicitly.

### Section 6 — Change Summary

Fill in the counts.

### Part 2 — Verification

Work through all verification checks. Record any failure before writing the file.

---

## Step 7 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/releases/v{version}-release-notes.md`

Create the `releases/` directory if it does not exist.

After writing the file, update `projects/{PROJECT_CODE}/brief.md` Section 9 (Related Artifacts) if the file exists — add a row:

| Artifact | Location | Status |
|----------|----------|--------|
| Release v{version} | [`releases/v{version}-release-notes.md`](releases/v{version}-release-notes.md) | Published |

Per `conventions/command-conventions.md` C-043, the Location column is a real markdown link relative to `brief.md`'s own location, never plain backtick text. Each release gets its own row — Section 9 accumulates one row per release, same as Sprint Plans. `Published` is fixed at creation, never updated afterward (`FW-002`: release notes carry no lifecycle status). If `brief.md` does not exist, skip this step silently.

---

## Step 8 — Report to the user

After writing the file:

```
RELEASE NOTES GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Sprint:            {NN}
Version:           v{version}
Output:            projects/{PROJECT_CODE}/releases/v{version}-release-notes.md
Feature areas:     {N}
Deployment steps:  [the most important ones, or "None — automated deployment"]
─────────────────────────────────────────
Next steps:
1. Product Owner reviews and approves the release notes.
2. On approval: production deployment can proceed.
3. After deployment: update project status if this was the final sprint.
```
