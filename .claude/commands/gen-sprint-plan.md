# Generate Sprint Plan Document

You are generating a focused sprint plan document for a single sprint of an Ascendra project. This document serves as the sprint kickoff reference, daily tracking aid, and completion record. It is generated once per sprint, before the sprint goes Active.

The sprint plan document captures: sprint goal, pre-sprint readiness checklist, story board with live statuses, delivery sequence, capacity breakdown, Should Have decisions, and a retrospective section to complete at sprint end.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

`$ARGUMENTS` must contain a project path and a sprint number. Examples:

```
/gen-sprint-plan projects/ASCENDRA-PAY 01
/gen-sprint-plan projects/ASCENDRA-PAY 07
```

If `$ARGUMENTS` is missing or ambiguous, ask:
> "What project and sprint? Provide: project path (e.g. `projects/ASCENDRA-PAY`) and sprint number (e.g. `01`)."

From the arguments, derive:
- `PROJECT_CODE` — the folder name (e.g. `ASCENDRA-PAY`)
- `SPRINT_NUMBER` — strip whitespace, then zero-pad to two digits (e.g. `1` → `01`, `7` → `07`, `01` stays `01`). State the normalised value: "Sprint number: {NN}"
- Output path: `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md`
- Create the `sprints/` directory if it does not exist.

---

## Step 2 — Read the index files and template

Read all three files in full before doing anything else:

1. `projects/{PROJECT_CODE}/stories/index.md` — Sprint Planning table (Section 8) for the sprint row, and Story Registry (Section 1) for story statuses and sizes.
2. `projects/{PROJECT_CODE}/epics/index.md` — Epic statuses and dependency layers.
3. `projects/TEMPLATE/sprints/sprint-plan.template.md` — the document template. Read this in full now. In Step 6 you will populate a sprint plan following this template exactly — do not use any other document structure.

Do not read individual story files or epic files — all needed data is in the two indexes.

---

## Step 3 — Extract sprint data from the indexes

From Section 8 Sprint Planning table, find the row for Sprint {NN}. Extract:
- **Status** (Planning / Active / Complete)
- **Epic(s)** — the epic or epics covered by this sprint
- **Stories** — expand ranges (e.g. US-01-001 → US-01-009 means stories 001 through 009 for that epic) and comma-separated lists into a full ordered list of story IDs
- **Count** — total story count
- **Sizes** — the size breakdown (e.g. 1L + 2M + 6S)
- **Delivery notes** — the delivery sequence embedded in the Notes column

From Section 1 Story Registry, for each story ID in this sprint's list:
- Title, Size, Priority (Must Have / Should Have), current Status, Flag

From Section 6 Should Have register: identify any Should Have stories in this sprint.
From Section 5 Flags Register: identify any flags (L-size split decisions, capacity flags) affecting this sprint.

From the epics index:
- For each epic in this sprint: current Status, Dependencies
- For each dependency epic: current Status (to populate the pre-sprint checklist)

---

## Step 4 — Gather missing inputs

Ask the following **all at once** before generating the document. Ask only what is missing from `$ARGUMENTS`.

| Input | Question if missing |
|-------|---------------------|
| **Sprint goal** | "What is the sprint goal — one sentence describing the outcome delivered by the end of this sprint? (Or type 'auto' to have it drafted from the epic scope.)" |
| **Sprint dates** | "What are the sprint start and end dates? (e.g. 2026-07-07 to 2026-07-18, or type 'TBD')" |

If the user types `auto` for sprint goal, draft it from the epic(s) scope:
- For a single epic: "Complete all {Epic Title} stories, establishing the {domain capability} layer of the platform."
- For multiple epics: "Complete {Epic A} and begin {Epic B}, establishing {capability A} and the foundation for {capability B}."

Present the drafted goal and ask for confirmation before proceeding.

---

## Step 5 — Run the pre-sprint readiness check

Before generating the document body, evaluate each gate item. Record Pass / Fail / N/A for each.

| Gate | Check | How to evaluate |
|------|-------|----------------|
| Gate 5 | Architecture document exists and is not Draft | Check if `projects/{PROJECT_CODE}/architecture/arch-v1.md` exists. If it does not exist: Fail. If it exists: read its Status field. Locked = Pass, Draft = Fail. |
| Epics Approved | All epics for this sprint are Approved or beyond | Check epic statuses in the epics index. Draft = Fail. |
| Stories Reviewed | All stories for this sprint are Reviewed or beyond | Check story statuses in Section 1 registry. Draft = Fail. |
| Dependencies cleared | All dependency epics for this sprint's epics are Done | Check dependency epic statuses. Not Done and not In Progress = Fail. In Progress = Warn (not hard block). |
| L story decision | L-sized stories in this sprint have a recorded PO split decision | Check Flags Register in stories index. If L flag exists for any sprint story and no resolution is recorded: Fail. If no L stories: N/A. |
| Capacity decision | Over-capacity sprints have a PO-confirmed split | Check Flags Register for capacity flags on this sprint's epics. If flag exists and no resolution noted: Fail. If not flagged: N/A. |

---

## Step 6 — Generate the sprint plan document

Use `projects/TEMPLATE/sprints/sprint-plan.template.md` as the document structure (you read this in Step 2). Populate every section in order following that template exactly. Do not skip any section — write "None" or "N/A" where applicable.

**Output format:** the generated sprint plan does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Remove the `## Part 1 — Template` label — the header fields (`**Project:**`, `**Sprint:**`, etc.) start directly after `## Document Control`. Unlike stories and epics, the `## Part 2 — Verification` checklist is **not** carried into the output at all, renamed or otherwise — it is an internal pre-write self-check only (run it before writing the file; do not persist it). This matches `projects/TEMPLATE/sprints/sprint-plan.example.md`, which has neither a `Part 1` label nor any Verification section.

Fill in the following data from Steps 3–5:
- **Header fields:** sprint number, status, epic(s), story count with size breakdown, sprint goal, start/end dates, generation date
- **Pre-Sprint Readiness Checklist:** results from Step 5 gate checks (Pass / Fail / N/A per item). If any item is Fail, add a resolution note below the checklist.
- **Story Board:** all stories in delivery sequence order (not registry order), with current status
- **Delivery Sequence:** reformatted from the Notes column in stories/index.md Section 8 — make it readable (Step 1 → Step 2 → etc.)
- **Capacity Analysis:** size breakdown table, total count, Must Have vs Should Have breakdown
- **Should Have Decision:** list each Should Have story with its inclusion decision
- **Definition of Done:** use the standard 5 items from the template exactly
- **Sprint Notes:** leave as blank placeholder
- **Sprint Retrospective:** leave as blank placeholder — this is filled by `/close-sprint` at sprint end

---

## Step 7 — Write the output file

**Output path:** `projects/{PROJECT_CODE}/sprints/sprint-{NN}.md`

Create the `sprints/` directory if it does not exist.

After writing the file, report:

```
SPRINT PLAN GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Sprint:        {NN}
Output:        projects/{PROJECT_CODE}/sprints/sprint-{NN}.md
Readiness:     {N}/6 gates Pass
Gates Failed:  [list of failed gate names, or "None"]
─────────────────────────────────────────
Next step:
[If all gates Pass: "Run /update-status projects/{PROJECT_CODE} sprint {NN} Active to begin the sprint."]
[If any gate Fails: "Resolve the following before the sprint can go Active: [list]. Re-run /gen-sprint-plan once resolved."]
Update story statuses during the sprint with /update-status. Run /project-status
at any time for the live dashboard.
```
