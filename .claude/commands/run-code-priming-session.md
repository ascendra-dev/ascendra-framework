# Run Code Priming Session

You are running a code priming session for the Ascendra framework — a structural walk of an existing legacy codebase that produces the same `priming-package.md` `/run-priming-session` produces, but reads code instead of prose. This is the second priming-producing command alongside `/run-priming-session`; design rationale is recorded in `decisions/FW-051-code-priming-session.md`, for background if you want the "why" — this file is fully self-contained and does not require reading it to execute correctly.

Unlike `/run-priming-session`, this is not primarily a live conversation — a codebase doesn't need a PO narrating it, and this command should be able to walk one unattended across many checkpointed sessions. A PO present live is still useful for resolving flags cheaply (per `conventions/priming-command-conventions.md` PC-053) and for narrowing scope up front, but their continuous presence is not required the way it is for `/run-priming-session`'s free-form half.

`conventions/priming-command-conventions.md` is the shared contract this command is governed by alongside every other priming-producing command. This command never reads the full repository — every pass below is structural-navigation- or targeted-search-driven, and progress is checkpointed in a dedicated working-state file (`code-priming-state.md`) so no session needs to hold the whole walk's history in context.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract `PROJECT_CODE` from `$ARGUMENTS`. Required — this command has no standalone-project variant; it always runs against an existing project.

If missing, ask:
> "Which project is this code priming session for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Wait for PO response.

---

## Step 2 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Project exists | `projects/{PROJECT_CODE}/` resolves | "`{PROJECT_CODE}` doesn't exist yet. Run `/init-project` first — it creates the folder structure this command reads and writes into." |
| `source-material/` folder exists | `projects/{PROJECT_CODE}/source-material/` resolves | "This project was created before `source-material/` was added to `/init-project`'s skeleton. Create `projects/{PROJECT_CODE}/source-material/` manually, or re-run `/init-project` if this is otherwise a fresh project." |

No separate "codebase path reachable" check — the codebase lives in the same `source-material/` folder every priming command already gates on (one material folder per project). Whether that folder actually contains readable code is verified by Step 5's Map pass, not a gate check — see Step 5.

---

## Step 3 — Read what already exists

Before saying anything to the PO:

1. `projects/{PROJECT_CODE}/source-material/` — list every file and directory present. Do not deep-read any of them yet; this is inventory, not Map (Map happens in Step 5 and is itself structural-only).
2. `projects/{PROJECT_CODE}/priming/priming-package.md` — if it exists, business-facing facts already captured (by this command or `/run-priming-session`) live here. Read Section 3 (Coverage Snapshot) and Section 11 (Resume Instructions).
3. `projects/{PROJECT_CODE}/priming/code-priming-state.md` — if it exists, this is a **resumed** code-priming walk specifically. Read Section 8 (Resume Instructions), Section 3 (Progress Snapshot), and Section 5 (Entity Ledger) to know exactly which unit to resume at. If it does not exist, this is a fresh walk — it will be created in Step 5 from `projects/TEMPLATE/priming/code-priming-state.template.md`.
4. `projects/{PROJECT_CODE}/brief.md`, `projects/{PROJECT_CODE}/domain/*-core.md`, `projects/{PROJECT_CODE}/brds/brd-core-v1.md` — whichever of these already exist. **Anything already Approved/Locked in a real artifact is out of scope for this session** — do not re-extract it. If a genuine new fact touching an already-settled topic surfaces anyway during the walk, record it in `priming-package.md` Section 9 per `PC-015`, exactly like `/run-priming-session` does — never silently drop it.

---

## Step 4 — Confirm scope

Keep this short:

> "I see `source-material/` contains [N files/directories, one-line description]. I'll walk it structurally to extract domain knowledge — use everything there, or point me at specific parts to include or exclude (e.g. 'just the billing module', 'skip the tests folder')?"

Wait for PO response. Record the answer in `code-priming-state.md` Section 1 (PO-narrowed scope). No PO response needed beyond this — the walk itself (Step 5-6) proceeds autonomously and checkpointed from here, unlike `/run-priming-session`'s ongoing back-and-forth.

---

## Step 5 — Initialize or resume the walk

**Fresh walk (no `code-priming-state.md` yet):**

1. Create `projects/{PROJECT_CODE}/priming/code-priming-state.md` from `projects/TEMPLATE/priming/code-priming-state.template.md`. Fill Section 1 (Session Context) from Steps 1-4.
2. Run the **Map pass** — structural only, no file contents read yet:
   - Walk the directory/module structure of `source-material/` (respecting Step 4's scope narrowing), populating Section 4 (Repository Map).
   - For each module, search for recognizable schema/ORM patterns — these are framework-agnostic concepts, not stack-specific rules to hardcode: an annotated entity class (any language's ORM), a models file, a schema definition file, a migration/DDL file. Recognizing these is ordinary pattern-reading, the same way you already read any of these stacks directly.
   - Mark each module's "Entity layer directly discoverable here? (Y/N)" in Section 4. `N` triggers the graceful-degradation path in Step 6.
   - Populate initial rows in Section 5 (Entity Ledger) for every candidate identified directly from a schema/ORM layer, `Pass Status = Mapped`.
3. **Fail-fast check:** if this first pass turns up nothing recognizable as code anywhere in `source-material/` — no source files, no manifests, no schema/ORM patterns, no directory structure resembling a codebase — stop here and report:
   > "`source-material/` doesn't look like it contains a codebase I can walk — [what was actually found, if anything]. If the code lives somewhere else, drop it into `source-material/` first; this command doesn't read from any other location."
   Wait for PO response. Do not proceed to produce a near-empty priming package.

**Resumed walk (`code-priming-state.md` exists):** skip straight to Step 6, starting at the unit named in Section 8's "Next unit to process."

---

## Step 6 — Walk the codebase (Map → Triage → Extract, per unit)

For every unit in `code-priming-state.md` Section 5 not yet `Extracted` (or, for infrastructure-likely units, not yet in Section 6):

1. **Map** (if not already done in Step 5) — structural only, as above.
2. **Triage** — classify `domain-likely` vs. `infrastructure-likely` using the generalization filter's existing test: *"would every deployment of this base system need this, regardless of which extension is active?"* Auth plumbing, logging, email delivery, generic admin CRUD, and third-party SDK wrappers are infrastructure-likely — they get one boundary-level note in Section 6 and nothing deeper. A business entity, capability, or rule-bearing validator is domain-likely — it proceeds to Extract.
3. **Extract** (domain-likely units only) — pull the unit's full signal cluster via targeted search across the repository for its name/references, never a blanket read of its "home" directory. Gather signals in this trust-ranked order, using whichever actually turn up something (not every unit will have all seven):
   1. Schema/ORM definitions — highest-trust: entities, lifecycle states, relationships.
   2. Validation logic (guards, DTO decorators, DB constraints).
   3. State machines (enums + transition guards).
   4. Test names and assertions — often clearer than the implementation itself; a first-class source, not a fallback.
   5. Comments and commit messages citing a person, ticket, or regulation — cross-check against current actual behavior, since comments rot.
   6. Feature flags / per-tenant config — maps directly onto Known Variations.
   7. API surface (routes, payload shapes) — medium signal, can be generic scaffolding.

**Graceful degradation — entity-level chunking is preferred, not assumed.** Where Step 5's Map found no directly discoverable entity layer for a module (`N` in Section 4), attempt entity inference *within* that module instead of processing it as one undifferentiated blob — including inferring candidate entities from table names appearing in raw SQL query strings, where that's the only signal available. Where entity boundaries still can't be found with confidence, process that module at `Module/Layer` granularity instead, recorded as such in Section 5's `Granularity` column. Never silently present a module-level row as if it achieved entity-level precision.

**Telling a real rule apart from an accident** — this is the reverse-engineering step `/run-priming-session` doesn't need, since a human wrote prose with intent but code accumulates unintentional behavior too:

- **Leans toward real business rule** (candidate for `AI Knowledge Correction` if it diverges from generic domain understanding): enforced redundantly across layers (DB constraint + application validation + UI disabling an action); a business-language error message, not a generic technical one; named consistently across unrelated parts of the codebase; traceable to a comment/commit/ticket citing a business or legal reason.
- **Leans toward legacy design flaw** (candidate for `Legacy Design Question`, never carried forward as fact): inconsistent enforcement — checked on one path, silently skipped on another; an unexplained magic number, or code marked `HACK`/`TODO`/`temporary`; contradicts what's generically true of the domain elsewhere; duplicated/denormalized data with no stated reason.

**Legacy-material posture (`PC-014`)** applies exactly as it does for `/run-priming-session`: matches generic domain understanding → ordinary content. Diverges for what looks like a real, stated business reason → flag `AI Knowledge Correction`. Diverges and looks like a design flaw → flag `Legacy Design Question`, framed as a negotiation, never resolved unilaterally. **If a PO is live**, per `PC-053`, ask directly rather than deferring to a flag — cheaper resolution. If no PO is present (the expected default for this command, unlike `/run-priming-session`), record the flag for live resolution at whichever phase reads it later.

**Conflicting facts (`PC-050`-`PC-054`)** — when a unit's content disagrees with something already captured for the same topic (in `priming-package.md`, in `code-priming-state.md`, or between two places in the code itself), never silently pick a side. Record every conflicting statement in full, each attributed to its origin (file + rough location), tag the Flag `Conflicting Source`.

**Sensitive content (`PC-013`)** — legacy code commonly has hardcoded credential-shaped strings (API keys, tokens, passwords, connection strings) in config files or old commits. Recognize these and flag them to the PO rather than filing them into the priming package.

**Honesty below the chunk level** — if a specific fact for an otherwise-processed unit can't be found with confidence after a reasonable search, mark it `Pending` in the matching `priming-package.md` topic (or note the gap in `code-priming-state.md` Section 5's `Notes` column) rather than fabricating precision. An honest gap is always preferable to invented content — the same standard `/run-priming-session` already holds itself to.

**Checkpoint after every unit finishes** — write the unit's row in `code-priming-state.md` Section 5 to `Extracted` (or `Triaged (infrastructure-likely)` plus a Section 6 row) immediately, before moving to the next unit. Do not hold more than one unit's extraction in working context at a time; the ledger, not conversation history, is what makes a large codebase resumable across sessions.

---

## Step 7 — Merge extracted facts into the priming package

**Output path:** `projects/{PROJECT_CODE}/priming/priming-package.md`

Follow `projects/TEMPLATE/priming/priming-package.template.md` exactly. Do not include `[AI Guide]` blocks in the output.

**Mapping priority (`PC-020`):** every business-facing fact maps onto Sections 4-6 first, always — an entity, lifecycle state, Universal Business Rule, Standard Validation, or Known Variation discovered in code belongs in the matching Section 5 (Domain Material) topic, in domain vocabulary, exactly like a fact stated in conversation. Reuse the template's literal `"PO stated:"` field label even for code-derived facts — that label names the field, not the fact's literal origin, the same convention `/run-priming-session` already applies to file-triaged content.

**Section 10 — this command owns two subsections (`PC-032`), unlike `/run-priming-session`:**
- `### 10.N Source-Specific — Code Structure Reference (owned by /run-code-priming-session)` — existing API surface and schema-as-implemented, written here **only when noticed incidentally** while extracting for domain purposes. Never go looking for this on purpose — `PC-003` scopes priming to Problem Domain only, and this content is Architecture-flavored.
- `### 10.N Source-Specific — Sub-Domain Split Recommendation (owned by /run-code-priming-session)` — pull directly from `code-priming-state.md` Section 7. Non-binding; routes specifically to `/run-domain-discovery` per `PC-034`, not to "whichever phase is about to run."

No dedicated tech-debt-inventory subsection — individual tech-debt-flavored findings already have a home via the `Legacy Design Question` flag mechanism on whatever topic they touch.

**If `priming-package.md` already exists** (from a prior `/run-priming-session` run, or a prior code-priming session): merge new material into it. Update `Pending` topics that now have an answer. Never overwrite a `Covered` topic's existing content without the PO having actually revisited it — if code contradicts an already-`Covered` topic, that's a `Conflicting Source` flag, not a silent overwrite.

**Density check:** every entry reads like a discovery-state file's "PO stated" line — one to three sentences. Compress before writing.

---

## Step 8 — Run Pre-Handoff Verification

Work through both checklists before presenting the session as done:
1. `priming-package.template.md`'s own Section 12 checklist, for everything merged into the shared deliverable.
2. `code-priming-state.md`'s own Section 9 (Pre-Handoff Verification), for the working ledger itself.

A failed check does not mean the session failed — it means the gap gets recorded in the matching Resume Instructions section rather than silently dropped.

---

## Step 9 — Report and offer handoff

```
CODE PRIMING SESSION — {PROJECT_CODE}
─────────────────────────────────────────
Units walked:                   [N] ([N] at Entity granularity, [N] degraded
                                 to Module/Layer)
Domain-likely (Extracted):      [N]
Infrastructure-likely (batched):[N]
Units still Pending:            [N]
Sections substantially covered: [list, from priming-package.md Section 3]
Flags recorded:                 [N] (AI Knowledge Correction / Legacy Design
                                 Question / Scope Risk / Conflicting Source /
                                 terminology)
Sub-Domain Split Recommendation:[one line, or "None"]
Source-Specific facts noted:    [N] (Code Structure Reference)
─────────────────────────────────────────
Next step:
Run /anchor-project {PROJECT_CODE} to have this package automatically feed
/run-intake, /run-domain-discovery, and /run-brd-discovery as anchor reaches
each phase. If a Sub-Domain Split Recommendation was surfaced above, it
routes specifically to /run-domain-discovery (PC-034) — raise it there even
if running that command directly instead of via anchor.
```

> "Continue now with `/anchor-project {PROJECT_CODE}`? Or stop here — this walk can resume later from exactly where it left off, since `code-priming-state.md` tracks every unit's progress."

Wait for PO response.
