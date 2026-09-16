# Generate User Stories

You are generating user stories for the Ascendra platform. Stories are generated one epic at a time — after epics are approved and before sprint planning. Each story covers one testable, independently deliverable slice of an epic's scope. Stories must be implementation-ready: a developer must be able to start coding without asking a single follow-up question.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **Epic file path** — required. E.g. `projects/HARBORVIEW-INV-001/epics/EPIC-001-invoice-management.md`

Derive from the epic path:
- **PROJECT_CODE** — second segment of the path (e.g. `HARBORVIEW-INV-001`)
- **Architecture path** — `projects/{PROJECT_CODE}/architecture/arch-v1.md`

If the epic file path is missing, ask:
> "Which epic should stories be generated for? Provide the path (e.g. `projects/HARBORVIEW-INV-001/epics/EPIC-001-invoice-management.md`)."

Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Epic is Approved | Read the epic file. Check `**Status:**` in the epic header. Must be `Approved`, `In Progress`, or `Done`. | "Epic at `{path}` has status `Draft`. Stories cannot be generated until the epic is Approved. Run `/review-epics` and then `/update-status {epic-path} Approved`." |
| Architecture exists | `projects/{PROJECT_CODE}/architecture/arch-v1.md` must exist | "No architecture document found. Run `/gen-architecture {PROJECT_CODE}` to generate it first." |
| Architecture is Locked | The architecture document's `**Status:**` must be `Locked` | "Architecture document is not yet Locked. Run `/review-architecture {PROJECT_CODE}` and get Product Owner approval before generating stories." |

---

## Step 1.6 — Estimation standard

Check if `projects/{PROJECT_CODE}/standards/estimation.md` exists.

**If it exists:** Read it. Use it in Step 5 for all sizing decisions.

**If it does not exist:** Ask the PO to choose one now. This sets the sizing model for all stories in this project.

> "No estimation standard found for {PROJECT_CODE}. Choose one before generating stories:
>
> **(a) T-Shirt Sizing (Recommended)** — scope-based sizing using 4 factors: AC count, pattern novelty, context footprint, and approval cycle risk. Sizes: XS / S / M / L / XL. Sprint capacity: 8–10 stories per 2-week sprint.
>
> **(b) Fibonacci Story Points** — points-based sizing using the 1 / 2 / 3 / 5 / 8 / 13 scale. Sprint capacity planned by velocity. Best for teams with existing Scrum baselines.
>
> **(c) Time-Based** — hours per story. Sprint capacity in total hours. Best for regulatory projects or client billing by time.
>
> Type a, b, or c."

Wait for PO response.

Copy the chosen reference to `projects/{PROJECT_CODE}/standards/estimation.md`:
- (a) → copy `reference/estimation/tshirt-sizing.md`
- (b) → copy `reference/estimation/fibonacci-points.md`
- (c) → copy `reference/estimation/time-based.md`

> "Estimation standard set to [chosen name]. Saved to `projects/{PROJECT_CODE}/standards/estimation.md`. This will be used for all story sizing in this project."

---

## Step 2 — Read these files before doing anything else

Read every file listed below in full. Confirm you have read all of them before continuing.

1. `projects/TEMPLATE/stories/story.template.md` — the template every story must follow. Read Part 1 (Template) and Part 2 (Verification) in full.
2. `projects/{PROJECT_CODE}/standards/estimation.md` — the project sizing standard. Apply all sizing rules from this file in Step 5. Already loaded in Step 1.6 — re-read only if not already in context.

The epic file was already read in Step 1. Also read:

3. `projects/{PROJECT_CODE}/brds/brd-core-v1.md` — the BRD referenced by this epic (look for the BRD path in the epic file if this default does not match)
4. `[every domain knowledge file path referenced in the BRD]`
5. `projects/{PROJECT_CODE}/stories/index.md` — if it exists, to know the current story count and last sequence number
6. `projects/{PROJECT_CODE}/epics/index.md` Section 3 (Dependency Graph) — check whether this epic carries a documented story-level interleaving note with another epic (e.g. two epics whose true build order only resolves at the story level, not the epic level). This determines the Step 8 next-step message.

---

## Step 3 — Gather required inputs

Check `$ARGUMENTS`. If any of the following are missing, ask for them **all at once before doing any work**.

PROJECT_CODE and epic file are already resolved from Step 1. The Story ID prefix is derived automatically from the epic file path — do not ask for it. `EPIC-004-...md` → prefix `04`; `EPIC-013-...md` → prefix `13`. This is stable and known regardless of which sprint these stories eventually land in (sprint assignment happens later, in `/gen-sprint-plan`, and is tracked in the stories index, not in the Story ID).

Check `$ARGUMENTS` for this additional input. If missing, ask:

| Input | What to ask if missing |
|-------|----------------------|
| **Starting sequence** | "What sequence number should the first story start at for this epic? Type 'auto' to start at 001, or continue from this epic's existing story count in the stories index if this epic already has some stories from a prior partial run." |

---

## Step 4 — Plan the story list before generating files

Before writing any story file, produce a **planning table** and wait for confirmation:

Read the epic's **Section 3.1 (In Scope)** carefully. Each bullet is a candidate story. Also read **Section 8 (Notes)** — it may explicitly collapse multiple bullets into one story's ACs, or decompose one bullet into multiple stories.

Draft the story list:

| Story ID | Title | Maps to In Scope bullet | Size (initial estimate) | Notes |
|----------|-------|------------------------|------------------------|-------|

Apply these rules:
- One story per In Scope bullet (unless Section 8 Notes say otherwise)
- If a bullet describes two independent capabilities (e.g. "upload CSV and generate import log"), consider splitting
- Infrastructure stories with no user-visible surface use "As a developer" persona
- Scheduled/automated system jobs use "As a platform" persona

> "Does this story breakdown look right? Any adjustments before I generate the files?"

Wait for PO response. Do not write any files until confirmed.

---

## Step 5 — Generate each story file

After confirmation, generate one file per story following `projects/TEMPLATE/stories/story.template.md` exactly.

**Output format:** the generated story file does not include the `[AI Guide — Document Level]` block or any `[AI Guide]` notes from the template. Remove the `## Part 1 — Template` label — the story header fields (`**Story ID:**`, `**Epic:**`, etc.) start directly after `## Document Control`. The verification checklist is included in the output but renamed from `## Part 2 — Verification` to `## Verification`; the `[AI Guide — Verification]` instruction block within it is removed.

For each story:

**Header:** Story ID (US-{epic-number}-{seq}), Epic (EPIC-{NNN} — Title), Title (verb-noun format only), Layer (matches parent epic's Layer), Target (which repo(s) the story's code lands in — derive from the Locked architecture: Section 3.1 names the repo roles, and the story's ACs determine which apply; values `API` / `Web` / `Worker` / `+`-joined combinations in dependency order, e.g. `API+Web`; never name a repo role Section 3.1 doesn't declare), Walking Skeleton (FW-048 — `Yes` if the REQ this story references, per Section 2 below, is tagged `Yes` in the parent epic's Section 2; `—` otherwise; never re-derived)

**Section 1 — Story Statement:**
- Exactly one persona from BRD Section 3 (User Roles)
- Specific action — not a feature area description
- User value in the benefit — not a technical outcome
- Exception: "As a developer" for infrastructure stories, "As a platform" for automated jobs

**Section 2 — FR References:**
Comma-separated BRD REQ-IDs this story satisfies. At least one. Only REQs from the approved BRD. Only REQs assigned to this epic in the index.

**Section 3 — Acceptance Criteria:**
Numbered list of independently testable conditions. Cover:
- (a) Happy path — primary success scenario
- (b) Boundary/rejection case — invalid input or unmet precondition
- (c) Enforcement/idempotency — where domain knowledge or BRD specifies once-only, permanent, or cannot-be-undone operations
- (d) Audit/observability — if the story involves a state change to a financial entity or a privileged action, an audit event AC must be present

**Count:** minimum 3, maximum 10. More than 10 = split the story.
**Format:** Present-tense statement starting with a subject ("The system...", "The form...", "The import log..."). Not "Should" or "Must".
**No implementation detail:** no field names, table names, API routes, library names, or framework choices.

**Section 4 — Out of Scope:**
Never empty. List capabilities a developer would naturally implement next — but that are explicitly excluded from this story. For each item: name the story or concept that covers it. Never "to be handled later" without naming what handles it. Use Pass 1 format now (concept descriptions, not story IDs — story IDs don't exist yet for future stories).

**Section 5 — Size:**
Apply the sizing rules from `projects/{PROJECT_CODE}/standards/estimation.md` (loaded in Step 1.6). When factors disagree, take the highest. XL means split before sprint lock — if you assign XL, flag it immediately.

**Section 6 — Dependencies:**
Use Pass 1 format (plain English concepts). For infrastructure dependencies (env vars, external API credentials): always plain text, never replaced with story IDs. Only write "None" if this story can truly be the first story merged.

**Section 7 — Notes:**
Include only if there is a non-obvious constraint the developer must know: idempotency requirement, specific library or approach required by architecture, known edge case in the data, or context needed to interpret an AC correctly. Omit entirely if nothing non-obvious applies.

**Verification:**
Work through all 16 checklist items. Tick each that passes [x]. Record any failure before writing the file.

---

## Step 6 — Update the epic file and stories index

After all stories for this epic are written:

**Update the epic file (Section 7 — Stories):**
Fill in the Stories table with: Story ID, Title, Size, Status (Draft) for every story just generated.

**Update or create the stories index** at `projects/{PROJECT_CODE}/stories/index.md`:
If the index exists, add the new stories to it. If it does not exist, create it with:
- Story registry table (Story ID, Epic, Title, Layer, Target, Walking Skeleton, Size, Status, Sprint) — the Sprint column starts as "TBD" for every story; it is filled in later by `/gen-sprint-plan`, never at generation time. Walking Skeleton (FW-048) is carried from each story's own header field — never re-derived here.
- Sprint plan section — left as "Not yet planned" until `/gen-sprint-plan` runs; do not invent a sprint grouping here
- Should Have register (list of Should Have story IDs)
- Flags register (L-sized stories requiring PO split review before sprint lock, plus any cross-epic sequencing constraints noted during generation)

---

## Step 7 — Output location

**Story files:** `projects/{PROJECT_CODE}/stories/EPIC-{NNN}/US-{epic-number}-{seq}-{slug}.md`

Slug naming: lowercase, hyphen-separated from the story title. Examples:
- "Register Payer manually" → `register-payer-manually`
- "Generate invoice PDF" → `generate-invoice-pdf`

Create the `EPIC-{NNN}/` subdirectory if it does not exist.

---

## Step 8 — Report to the user

```
STORIES GENERATED — {PROJECT_CODE}
─────────────────────────────────────────
Epic:         {EPIC-ID}: {Epic Title}
Stories:      [N] generated
IDs:          US-{epic-number}-001 through US-{epic-number}-00N
Sizes:        XS:[N]  S:[N]  M:[N]  L:[N]  (XL:[N] — requires split)
Verification: [N] failures / None
Coverage:     [N] In Scope bullets with no story / None
─────────────────────────────────────────
Next step:
[If this epic carries a documented story-level interleaving note with another epic per epics/index.md Section 3 (checked in Step 2): "Generate stories for its paired epic now: /gen-stories {paired epic file path}. Their cross-epic dependencies mean they must be reviewed together — run /review-stories projects/{PROJECT_CODE} only once both epics' stories exist."]
[Otherwise: "Run /review-stories projects/{PROJECT_CODE} to review and sprint-assign this epic's stories now."]
Do not generate stories for every remaining epic before reviewing this
one — review (and sprint-assign) each epic or interleaved pair before
moving to the next, so delivery learnings carry forward instead of
being baked into stories written too far ahead.
```
