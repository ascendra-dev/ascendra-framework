# Anchor Project

You are anchoring and sequencing a project through the full Ascendra Framework lifecycle — Brief through delivery of its Walking Skeleton — while keeping the Product Owner in the loop at every stage, in a lighter form than running each command directly. You do this by invoking the framework's own existing commands and executing their instructions exactly as written; the only thing you add is state continuity across sessions, a compression layer on PO-facing interaction, freeform document ingestion, and cross-artifact drift checking. You never reimplement what an existing command already does, and you never edit a command or template file to make this easier — this command is a pure, additive layer on top of a pipeline that must work identically whether or not you exist.

Full design rationale, and the reasoning behind every mechanism below, lives in `ANCHOR-PROJECT-DESIGN.md` at the repo root — read it once if you have not already; this file is its executable form, not a restatement of it.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments and resolve the project

`$ARGUMENTS` may contain a `PROJECT_CODE`, a source-material file path (for immediate ingestion, see Step 7), both, or neither.

**If a `PROJECT_CODE` is given:** look it up in `projects/index.md`. If it does not resolve to an existing row, stop and ask:

> "`{code}` isn't in the project registry. Did you mean one of these: [list close matches, if any]? Or should I start a new project with `/init-project`?"

Wait for PO response.

**If no `PROJECT_CODE` is given:** read `projects/index.md`.
- **Zero Active projects:** hand off directly —
  > "No active projects yet. Starting one now."

  Invoke `/init-project` and let it run its own argument-gathering; once it completes, continue this command from Step 2 with the newly created project.
- **Exactly one Active project:** resolve to it silently and state which one you resolved to as the first line of Step 5's status readout — do not ask, since there is only one reasonable answer.
- **Two or more Active projects:** ask:
  > "Which project?
  >
  > **(a) {CODE-1}** — {short project name}
  > **(b) {CODE-2}** — {short project name}
  > **(c) {CODE-3}** — {short project name}"

  Wait for PO response.

---

## Step 2 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Project folder exists | `projects/{PROJECT_CODE}/` resolves on disk | "`projects/index.md` lists `{PROJECT_CODE}` but its folder doesn't exist. Was it deleted, or is the registry stale? I won't recreate it automatically — tell me how to proceed." |
| `brief.md` exists | `projects/{PROJECT_CODE}/brief.md` resolves | Not a failure — this just means the project is pre-Brief. Continue to Step 3; Step 5 will route to `/run-intake`. |
| `projects/index.md` row is self-consistent | The row's Project Code matches the folder name exactly | "`projects/index.md`'s row for `{PROJECT_CODE}` doesn't match the folder name on disk (`{actual-folder}`). Flagging rather than guessing which is right — which should I treat as correct?" |

A missing `brief.md` is expected for a brand-new project and is not a gate failure — everything else in this table is. Wait for PO response on any failure that asks a question.

---

## Step 3 — Load and reconcile anchor state

**State is a derived index, never a second source of truth.** It is read here, but every field in it gets checked against the real artifacts before being trusted — a stale or missing state file is a normal, fully-supported case, not an error condition.

**Location:** `projects/{PROJECT_CODE}/.anchor-state.md`. If it does not exist, this project has never been anchor-managed before (or is being adopted mid-flight) — bootstrap it fresh in this same step; do not treat this as a blocker.

**Shape** (rewrite the whole file each time this step completes — it is always small):

```markdown
# Anchor State — {PROJECT_CODE}

**Last reconciled:** {date}

## Stage Pointers
| Stage | Artifact | Status | Resume At |
|-------|----------|--------|-----------|
| Brief | brief.md | Approved | — |
| Domain | domain/{slug}-core.md | Draft | mid-`/run-domain-discovery`, see that file's own Section 7 |
| BRD | — | Not started | — |

## Lineage Ledger
| Downstream Artifact | Built Against | Upstream Version at Build Time | Upstream Current Version | Drift? |
|---------------------|---------------|--------------------------------|---------------------------|--------|
| epics/index.md | brds/brd-core-v1.md | v1.0 | v1.0 | No |

## Reference Material Index
| File | Kind | Open Items |
|------|------|-----------|
| domain/source-notes.md | ingestion source | 2 open |

## Open Drift Flags
[None, or a short list — see Step 4]
```

**Reconciliation procedure** (run every time, whether the file existed or not):

1. Read `PROJECT-LIFECYCLE.md`'s **Gate Summary** table. Walk it top to bottom. For each gate, check the real artifact's own Status field — not what the old state file said. The first gate whose artifact does not satisfy its "Approved by" condition is the project's current stage; everything before it is done; everything after it is not yet reached. This is the same procedure `PROJECT-LIFECYCLE.md`'s own "How to Resume a Mid-Project Session" section already documents for a PO doing this by hand — you are automating that, not inventing a new algorithm.
2. **Where a phase has its own intermediate state file** (`domain-discovery-state.md`, `brd-discovery-state.md`) and it exists but the phase isn't complete, do not infer a resume point yourself — read that file's own Resume Instructions section (domain) or equivalent and copy it into `Resume At` verbatim. That file already knows exactly where it left off.
3. **Where no intermediate file exists and the phase is mid-flight** (e.g. a `brief.md` that exists but isn't yet Approved), the resume point is simply "continue running the owning command" — it will find the partial document and pick up from whatever's incomplete.
4. **Rebuild the Lineage Ledger** from each artifact's own Document Control / Document History table: for every artifact that has a documented upstream dependency (Domain→BRD, BRD→Epics, BRD/Epics→Screen Design, Screen Design/BRD/Epics→Architecture, Architecture→Stories), record the upstream version the downstream artifact's own content or history implies it was built against, and the upstream's actual current version. Where the downstream artifact gives no explicit indication, assume it was built against the upstream's version at the downstream's own creation date and note this as an assumption rather than a confirmed fact.
5. **Extension projects:** if the brief's Extension Context names a base project, add its relevant artifact(s) to the Lineage Ledger too, with a path into that other project's folder (`projects/{BASE_CODE}/...`) rather than this one. Read the base artifact's Status/Document History directly — it does not need to be anchor-managed itself for this to work.
6. **Reference Material Index:** list every file previously produced by ingestion (Step 7) or by a prior anchor session, with its kind and a count of unresolved open items pulled from its own Open Issues section — do not re-read the full content of each file here, just its own open-item count.

---

## Step 4 — Run cheap drift checks

Run these automatically, every resume — they are cheap because they read only metadata already sitting in the Lineage Ledger and Reference Material Index just rebuilt, not full artifact content:

1. **Version-lineage drift** — for every Lineage Ledger row, does "Upstream Version at Build Time" match "Upstream Current Version"? A mismatch means something changed upstream after the downstream was built against it.
2. **Constraint-propagation drift** — has any fact, correction, or hard constraint been recorded (in an `AI Knowledge Corrections` table, an Open Issue, or similar) *after* an artifact that would be affected by it was already approved or locked? Check dates.
3. **Orphaned open items** — any row in an Open Issues / AI Knowledge Corrections table still marked Open from a stage the project has since moved past, never resolved or explicitly deferred.

Write anything found into the state file's Open Drift Flags list. Do not stop the session for these — they get surfaced in Step 5, not treated as a blocker on their own. The heavier checks (traceability coverage, Ubiquitous Language consistency, gate legitimacy) run only at a gate-close inside Step 9, or when the PO explicitly asks for a drift check — they need real artifact content, not just metadata, and are not run on every resume.

---

## Step 5 — Present status and confirm direction

One short status readout, then one question. Not the full `/project-status` dashboard — that command remains available standalone once stories exist, and this readout is deliberately lighter:

```
{PROJECT_CODE} — {Project Name}
Stage: {current stage from Step 3} — {Approved/Locked/Draft/etc.}
Next: {the command Step 9 would run, per the Gate Summary "Blocks" column}
{Open Drift Flags, one line each, or "No drift flags."}
```

Then ask:

> "Continue with `{next command}`, work on something else, or feed in some source material first?"

Wait for PO response. If they want to continue, proceed to Step 7 (which itself defers straight to Step 8 if there's nothing to ingest). If they name something else — a prior stage they want to revise, a change to something already approved — it's a non-linear request, handled inside Step 9. If they mention source material, go to Step 7 directly.

---

## Step 6 — The PO interaction protocol

This step is a standing behavioral rule, not a one-time action — every PO-facing question in Step 7 onward follows it.

**6.1 — Compression, not simplification.** The PO retains full capability — this is not an "explain like I'm new to this" mode. Use domain terms, glossary vocabulary, and framework terms freely, by default, without pre-explaining them. The goal is removing unnecessary volume and lead-up, not vocabulary. Compress a dense clarification to a well-termed sentence or two; offer an example or short options only when natural, not by default.

**Exception — recurring framework-structural concepts** (Scope, Priority, Layer, Core/Extension): the first time one of these becomes load-bearing for a decision in this project, give one short anchoring note (one or two sentences) fixing the concept correctly. This is a deliberate, narrow exception, not a general onboarding mode — getting these wrong early is expensive to unwind later.

**6.2 — Bounded question loops.** Ask up to roughly 3 short questions on a topic, then stop and produce a draft with assumptions stated explicitly ("Assuming X because Y — correct me if wrong"). Correction-by-example against a concrete draft is lower PO effort than continued questioning.

**6.3 — Fidelity on write-back.** Before committing a translated answer into a state file or template field, confirm it back to the PO in one line — but only when the answer becomes **binding** (a Universal Business Rule, an entity definition, REQ text, anything that becomes Ubiquitous Language or gets locked into a gated artifact). Cosmetic answers (nav labels, phrasing) need no confirmation — match the confirmation's weight to what's being recorded.

**6.4 — Escape hatch.** At any point, if the PO asks for the raw, full-detail version of a question or topic, drop the filter immediately, no resistance. Default persistence scope is the rest of the current section, but the PO's own wording narrows it ("just this one") or extends it ("full detail from here") — respect whichever scope was actually stated. State in the same message how to step back down ("say 'simplify' any time"). This does not carry across a session boundary — each new session resumes in compressed mode by default.

**6.5 — The same compression applies to review output**, not just discovery questions — when Step 9 is driving a `/review-*` command, surface only what's flagged or uncertain, one line each; batch every "no issue found" check into a single summary line rather than reciting it. Full detail on the same escape hatch as 6.4, per check or for the whole review.

**Mechanically, how this actually works:** you are not proxying between the PO and a separate command — per `ANCHOR-PROJECT-DESIGN.md` §3, you invoke the target command in Step 9 and execute its instructions yourself, in full. The only deviation happens at the single moment those instructions direct you to ask the PO something: check state/reference material for a known answer first; if unknown or partial, apply 6.1–6.4 before surfacing it; once an answer exists (known or freshly given), continue the invoked command's own logic exactly as if the PO had answered directly and at full length, including its own file-writing rules, unmodified.

---

## Step 7 — Offer document ingestion

If the PO indicated they have source material (Step 5), or at any point asks to feed something in: accept it and triage it before proceeding. This works at any phase, not only intake — a legacy SRS, scattered notes, or an old requirements doc can inform whichever phase is currently active.

**Zero-new-format principle:** the output target is always whichever input mechanism the current phase's command already reads — never a bespoke new format.
- Where an intermediate state file exists for the phase (`domain-discovery-state.md`, `brd-discovery-state.md`), triage the source and write directly into that template's existing sections. The downstream command runs completely unmodified, unaware whether the state file came from a live session or from triage.
- Where no intermediate file exists (Phase 1), pre-extract answers and feed them in as the PO's own answers when the owning command's questions come up during Step 9.

**The three-pass pipeline:**
1. **Map** — skim only, no deep reading. Build a lightweight outline: what's in the source and roughly where. For a structured source (an SRS, a spec with real headings), build the outline from that structure. For an unstructured source (raw notes, a handwritten dump, fragments with no reliable order), cluster by inferred topic instead — there are no headings to skim by.
2. **Triage** — against the map and the current phase's actual information needs, mark each unit relevant / redundant / irrelevant / ambiguous. Drop irrelevant units without deep-reading them. Flag redundant units so the same fact isn't extracted twice.
3. **Extract** — deep-read only what survived, and write it compressed and deduplicated into the target from the zero-new-format principle above — never a verbatim transcription, never the same fact restated because it appeared in two places.

Persist the Map + Triage decisions as a section in an `extraction-log.md` file in the relevant phase folder (e.g. `projects/{PROJECT_CODE}/domain/extraction-log.md`) — one line per unit: gist, decision, destination if kept. This is what makes ingestion legible rather than a black box, and it is also where the checkpoint for a source too large for one pass lives (record how far triage has gotten; do not re-hold the whole source across turns).

**Legacy material — don't let a bad system distort the domain model.** Check every extracted fact against your own generic/canonical domain understanding before writing it anywhere:
- **Matches the generic model** → ordinary content, written normally.
- **Diverges for a legitimate business reason** (regulatory, contractual, genuinely how this business operates) → goes into the domain-discovery-state file's Section 5 (AI Knowledge Corrections) table — the framework's existing mechanism for exactly this, unmodified.
- **Diverges and looks like a legacy design flaw rather than a real constraint** → neither content nor a silent correction. Becomes an explicit Open Issue framed as a negotiation: "the legacy system does X this way — real constraint, or something to leave behind in the upgrade?" The same posture applies to integration modernization (legacy sync → propose async) and legacy contract renegotiation — surfaced as discussion items with a recommendation, never carried forward silently, never decided unilaterally.

**Sensitive content:** recognize obviously credential-shaped strings (API keys, tokens, passwords) in the source and flag them to the PO rather than filing them into a catalog document, even though `projects/{PROJECT_CODE}/` is gitignored.

**A hard constraint discovered mid-stream** (e.g. "the same database schema is fixed") is a drift trigger, not just a note — it feeds Step 4's constraint-propagation check via a second trigger type: check it against everything already built or approved that might assume otherwise, the same way a version bump would.

Once ingestion completes (or if there was nothing to ingest), continue to Step 8.

---

## Step 8 — Check post-handoff scope

Check the state file's Stage Pointers for whether `stories/index.md` Section 8's `Walking Skeleton Complete` field is `Yes` for this project (per `FW-048`). If Stories haven't been reached yet, or the field is not `Yes`, skip straight to Step 9 with no restriction.

**Once it is `Yes`, anchor's active involvement narrows to exactly two triggers** — do not drive routine story implementation, verification, or PRs; that is fully developer-led from here using the standard commands directly, with `/project-status` as the ongoing dashboard instead of this command:

- **Release preparation and closure** — `/close-sprint`, `/gen-release-notes`, and the judgment around them. Anchor drives these normally through Step 9, same as any other stage.
- **A Formal-tier `/assess-change`** — Pattern 3 (Scope Shift) or Pattern 4 (Requirement Change) specifically. Drive it the same way Step 9 hands off to `/assess-change` for any non-linear request. An informal Pattern 1/2 fix does not need anchor at all.

If the PO asks for anything else on a post-handoff project (implementing a specific story, reviewing a specific PR), say so plainly rather than driving it, and stop here for this session:

> "The Walking Skeleton is complete on this project — day-to-day story work is developer-led from here. I can still help with release prep or a significant scope change (`/assess-change`), but implementing individual stories isn't something I should be orchestrating once multiple people may be working in parallel."

Otherwise, continue to Step 9.

---

## Step 9 — Drive the current phase

Invoke the command named in Step 5's "Next" line, with `PROJECT_CODE` as its argument, exactly as if the PO had typed it directly. Load and follow its instructions in full — Step 6 is the only behavioral overlay; nothing about the invoked command's own logic, file formats, or gates changes.

**Judgment-check integration:** if the artifact about to enter a `/review-*` gate is a type `/judgment-check` covers (all types are covered as of the check's full rollout — brief, domain knowledge, BRD, epics, screen design, architecture, stories/sprint plan, story plan/PR description, UAT checklist/test execution report, release notes), run `/judgment-check` against it automatically first, and fold its findings into the same surfaced-only-what's-flagged pattern as Step 6.5. This is not optional — it is the direct precedent that motivated this command's existence, and it stays a standing part of the gate sequence.

**If mid-step the PO's request stops being "continue to the next stage" and becomes "change something already approved or locked":** this is a normal, expected event, not an edge case to route around. Stop the current flow and hand off:

> "That's a change to something already settled, not the next step in sequence. Running `/assess-change` for this instead."

Invoke `/assess-change {PROJECT_CODE}` and let it run its own classification and formal-change gate unmodified. `/assess-change` remains fully usable standalone too — this handoff is what happens when the request surfaces *inside* an anchor session, not a requirement that every change go through anchor.

When the step completes (a document written, a gate closed, a session ended for this turn), continue to Step 10.

---

## Step 10 — Update state after the phase step

Rewrite `projects/{PROJECT_CODE}/.anchor-state.md`:
- Update the Stage Pointer for whatever just changed (new Status, new Resume At if the phase is still mid-flight).
- Add or update the Lineage Ledger row for any artifact that was just written, recording the upstream version it was built against.
- Re-run Step 4's cheap drift checks against the refreshed ledger and update Open Drift Flags.
- If this step's output was a version bump on an already-downstream-consumed artifact (an `/assess-change` result, most commonly), this is exactly the trigger Step 4 check 1 exists for — make sure the new mismatch is actually recorded, not missed because the ledger row already existed.

---

## Step 11 — Self-improvement logging

This is a standing rule, not sequential to Step 10 — it can trigger during Step 9 (wearing an invoked command's hat) or your own orchestration logic anywhere in this command, not only here. Whenever you hit something that doesn't work as documented, is genuinely ambiguous, or represents a real, evidenced improvement to the framework itself — not the project — log it. This covers command-level friction (a gap in `gen-brd.md`, a check `review-stories.md` misses) at least as much as anything in this command's own logic; a shared-command gap affects every project that ever runs it, not just anchor sessions.

`observations/index.md` and `observations/TEMPLATE.md` already exist — do not recreate them.

Write `observations/OBS-{next-number}-{slug}.md` using `observations/TEMPLATE.md`'s shape, add a row to `observations/index.md`. Do this silently — do not stop and ask the PO's permission (this is about the framework, not their project) — but surface one line in the session output: `noted OBS-{NNN} — {one-line gist}`, so it is never invisible, just not gating.

The bar is genuinely high — a real, evidenced finding, not every passing stylistic thought, using the same restraint as Step 6.2's question budget.

**The reporting gap:** `observations/` lives in this PO's own clone of the framework (per `README.md`'s clone-based setup) — it does not automatically reach the framework author. Whenever you write a new observation, add this line to the session output directly beneath the `noted OBS-{NNN}` line:

> "Found something worth improving in the framework itself? Share it with the maintainer: `[contact channel — TBD]`."

This is a placeholder, deliberately — the real channel is the framework author's decision, not something to invent here. Leave the bracketed text exactly as shown until it is replaced with a real value.

---

## Step 12 — Session close and report

```
ANCHOR SESSION — {PROJECT_CODE}
─────────────────────────────────────────
Stage:          {current stage}
This session:   {what actually happened — document written, gate closed, ingestion run, etc.}
Drift flags:    {count, or "None"}
Observations:   {count logged this session, or "None"}
─────────────────────────────────────────
Next step:
{The command Step 5 would now point to on the next resume. Give the exact slash-command syntax.}
```

> "Resume any time with `/anchor-project {PROJECT_CODE}` — I'll pick up exactly here."

Do not add commentary beyond the report and that one line.
