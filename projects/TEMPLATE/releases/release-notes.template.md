# Release Notes — [Project Name] v[Version]

> **[AI Guide — Document Level]**
> Release notes are a client-facing artefact. Write for a non-technical reader.
> Describe every change in terms of what a user can now do or experience differently — not what was built, which endpoint changed, or which database table was added.
> Do not include: story IDs, epic IDs, table names, API routes, library versions, or internal ticket numbers.
> Do not invent anything. Every item listed here must correspond to a merged story in the sprint.
>
> Tone: professional but clear. Active voice. Present tense for features, past tense for fixes.
> Length target: 1–3 sentences per feature. Enough context that the client understands the impact; not so much detail that it reads like a technical summary.
>
> **File location:** `projects/{PROJECT_CODE}/releases/v{version}-release-notes.md` — keyed by version, not sprint number, since a release doesn't always map 1:1 to a sprint.
> e.g. `projects/ASCENDRA-PAY-001/releases/v1.0-release-notes.md`

---

## Document Control

| Field | Value |
|-------|-------|
| Project | |
| Version | |
| Release Date | |
| Sprint | |
| Produced by | Ascendra AI |
| Approved by | |
| Status | Draft |

---

## 1. What's New

> **[AI Guide]** Group by feature area, not by story. If three stories all contribute to invoice management, they appear under one heading "Invoice Management", not as three separate entries.
> Prioritise the most significant changes first within each group.
> Use plain, outcome-focused language. Example: "You can now..." / "The system now..." / "Payments are now..."

### [Feature Area Name]

- **[Feature name]:** [One sentence describing what the user can now do or what changed for them. Focus on the outcome, not the mechanism.]

### [Feature Area Name]

- **[Feature name]:** [Description]

---

## 2. Improvements

> **[AI Guide]** List changes that improve existing behaviour — faster, more reliable, more informative. 
> Only include if the improvement is user-perceptible. Leave this section blank if there are no improvements.
> Write: "X now [does something better]" — not "Improved X."

- **[Improvement name]:** [What was improved and what the user will notice.]

---

## 3. Bug Fixes

> **[AI Guide]** List bugs resolved in this release. Write what was happening before and what happens now.
> Only include bugs that users could have observed. Skip internal/infrastructure-only fixes.
> If no bugs were fixed: write "No bug fixes in this release."

- **[Issue description]:** [What was happening before (the symptom, not the root cause). What happens now.]

---

## 4. Known Limitations

> **[AI Guide]** List any feature gaps, edge cases, or workarounds the client should be aware of in this version. If a feature was partially implemented (Some but not all stories merged), name it and describe what is available now vs what will be complete in a future sprint.
> Write what the user should expect, and what the workaround is if one exists.
> If there are no known limitations: write "None."

- **[Limitation]:** [Description. Workaround if applicable. Planned resolution sprint if known.]

---

## 5. Deployment Notes

> **[AI Guide]** List every step a deployer or administrator must take to apply this release correctly. Be explicit.
> Common items: run database migrations, update environment variables, clear caches, reseed lookup data.
> If deployment is fully automated (no manual steps): write "No manual steps required — deployment pipeline handles all changes."
> If there are environment variable changes: list each variable name, whether it is new or modified, and where to find the value.

### Pre-Deployment

- [ ] [Step that must happen BEFORE deployment — e.g. "Take a database backup"]

### Post-Deployment

- [ ] [Step that must happen AFTER deployment — e.g. "Run database migrations: `npm run db:migrate`"]
- [ ] [Step — e.g. "Verify smoke test endpoint responds with 200"]

### Environment Variable Changes

| Variable | Change | Required Action |
|---------|--------|----------------|
| `VARIABLE_NAME` | New / Modified / Removed | [What to do] |

_If no environment variable changes: None._

---

## 6. Change Summary

> **[AI Guide]** A concise count of what changed. Fill in numbers from the sprint data.

| Category | Count |
|---------|-------|
| New features / capabilities | |
| Improvements | |
| Bug fixes | |
| Stories delivered | |
| Epics completed | |

---

## 7. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled when these release notes are generated, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/10-release-support-hotfix.md`'s own tests (release-notes versioning by release vs. sprint, and the client-facing-language discipline that excludes story IDs, epic IDs, table names, and API routes) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 8 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against these release notes, this section does not appear at all; do not pre-create it empty.

## 8. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control. Each row records one finding from Section 7's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against these release notes, this section does not appear at all; do not pre-create it empty.
>
> **[AI Guide — numbering note]** Sections 7 and 8 are reserved for `/judgment-check` as shown here, immediately after Section 6 (Change Summary) and before Part 2 (Pre-Release Verification) — note this template's Part 2 heading is named "Pre-Release Verification," not "Verification" as in every other template; that name is unchanged by this section. No `/review-release-notes` command exists in this framework, so there is no other command reserving a conflicting section number here.

---

## Part 2 — Pre-Release Verification

> **[AI Guide]** Run these checks before writing the file. Record any failure.

- [ ] Every item in Section 1 corresponds to a Merged story in this sprint (no invented features)
- [ ] No story IDs, table names, API routes, or technical internals appear in Sections 1–3
- [ ] Section 4 (Known Limitations) accurately reflects what was NOT implemented (check sprint carry-forward items)
- [ ] Section 5 Deployment Notes lists every migration, env var change, and seed data change introduced in this sprint
- [ ] Version number in Document Control matches the version passed to the command
- [ ] Release Date is the actual or target go-live date (not today's generation date unless they are the same)
- [ ] Approved by field is blank — Product Owner fills this on approval

**Verification failures:** [List any, or write "None — all checks passed"]
