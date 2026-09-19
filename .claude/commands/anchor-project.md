# Anchor Project

You are anchoring and sequencing a project through the full Ascendra Framework lifecycle — Brief through delivery of its Walking Skeleton — while keeping the Product Owner in the loop at every stage, in a lighter form than running each command directly. You do this by invoking the framework's own existing commands and executing their instructions exactly as written; the only thing you add is state continuity across sessions, a compression layer on PO-facing interaction, and cross-artifact drift checking. Document ingestion is not your own logic — it's a standalone companion command, `/run-priming-session`, which you hand off to and later decompose the output of (Step 7). You never reimplement what an existing command already does, and you never edit a command or template file to make this easier — this command is a pure, additive layer on top of a pipeline that must work identically whether or not you exist.

Full design rationale, and the reasoning behind every mechanism below, lives in `ANCHOR-PROJECT-DESIGN.md` at the repo root — read it once if you have not already; this file is its executable form, not a restatement of it.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments and resolve the project

`$ARGUMENTS` may contain a `PROJECT_CODE`, or nothing.

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
| priming/priming-package.md | priming session output | 2 Pending topics, 1 open item |

## Open Drift Flags
[None, or a short list — see Step 4]
```

**Reconciliation procedure** (run every time, whether the file existed or not). **Read lightly, not fully:** every check below names a specific field, heading, or small table to look at — use a targeted search (grep for that heading, or read just the relevant few lines) rather than loading a whole BRD or architecture document into context to answer a one-field question. Reconciliation and the cheap drift checks in Step 4 should never require reading a full artifact; only the heavier checks that explicitly need real content (traceability, Ubiquitous Language consistency, gate legitimacy — see Step 4) justify that cost, and only when they actually run.

1. Read `PROJECT-LIFECYCLE.md`'s **"Lifecycle at a Glance"** table (the 28-step + lettered-sub-step sequence, in true execution order) — not the coarser **Gate Summary** table. Gate Summary only names the 15 formal PO-approval gates, and its "Blocks" column documents what closing one unlocks for a human skimming top to bottom — it was never built to answer "what's the very next command to run," and using it that way silently skips real, required commands that sit ungated between two gates (`/gen-domain-knowledge` has no gate of its own at all — "Gate: None — reference material, not a locked artifact" — and neither do `/gen-stories` after the Architecture gate or `/gen-sprint-plan` after the Stories gate, among others). Walk Lifecycle at a Glance top to bottom in table order (per its own note: table position is authoritative over the numeric label where the two disagree). For each row, through Architecture Lock and through however many Walking Skeleton stories Step 8 has anchor actively driving:
   - A row with no ✅ Manual Gate mark is satisfied once its own output already exists on disk, regardless of that artifact's own Status field — the row's Command column is what produced it.
   - A row with a ✅ Manual Gate mark is satisfied only once the real artifact's own gate-relevant Status field actually reads its approved/locked/complete value — existing in Draft is not satisfied.
   - Optional/conditional rows (2a/2b Domain Discovery, 3a Mock Discovery, 9a Screen Design when the BRD shows no UI in scope) are treated as satisfied automatically when their own stated condition doesn't apply — no need to ask.

   The first unsatisfied row is the project's current stage; its own **Command** column — not a Blocks-column lookup — is what Step 5's "Next" names and Step 9 invokes directly. Everything before it is done; everything after it is not yet reached. This is the same thing `PROJECT-LIFECYCLE.md`'s own "How to Resume a Mid-Project Session" step 3 means by "find the first *step* in this document where the gate has not been closed" — "step" there is Lifecycle at a Glance's own numbering, not Gate Summary's coarser gate count; you are automating that procedure, not inventing a new one.

   **Past the Walking Skeleton** (Step 8's own handoff trigger): sequencing shifts entirely to reading the real tracking artifacts directly — `epics/index.md`, `stories/index.md`, the active sprint plan — the same data `/project-status` already reads, rather than forcing multiple parallel developers' work into one linear row. Anchor isn't driving routine sequencing there anyway per Step 8; this only matters for the narrow cases (release prep, Formal-tier `/assess-change`) Step 8 still hands to Step 9.
2. **Where a phase has its own intermediate state file** (`domain-discovery-state.md`, `brd-discovery-state.md`) and it exists but the phase isn't complete, do not infer a resume point yourself — read that file's own Resume Instructions section (domain) or equivalent and copy it into `Resume At` verbatim. That file already knows exactly where it left off.
3. **Where no intermediate file exists and the phase is mid-flight** (e.g. a `brief.md` that exists but isn't yet Approved), the resume point is simply "continue running the owning command" — it will find the partial document and pick up from whatever's incomplete.
4. **Rebuild the Lineage Ledger** from each artifact's own Document Control / Document History table: for every artifact that has a documented upstream dependency (Domain→BRD, BRD→Epics, BRD/Epics→Screen Design, Screen Design/BRD/Epics→Architecture, Architecture→Stories), record the upstream version the downstream artifact's own content or history implies it was built against, and the upstream's actual current version. Where the downstream artifact gives no explicit indication, assume it was built against the upstream's version at the downstream's own creation date and note this as an assumption rather than a confirmed fact.
5. **Extension projects:** if the brief's Extension Context names a base project, add its relevant artifact(s) to the Lineage Ledger too, with a path into that other project's folder (`projects/{BASE_CODE}/...`) rather than this one. Read the base artifact's Status/Document History directly — it does not need to be anchor-managed itself for this to work.
6. **Reference Material Index:** if `projects/{PROJECT_CODE}/priming/priming-package.md` exists, list it here with a count of `Pending` topics and open items (Section 9's row count). Get the `Pending` count by scanning Sections 4-6's own per-topic Status lines directly — a short, targeted scan of one-line fields, not a full-document read. Section 3 (Coverage Snapshot) cannot be used for this: it only gives a coarse per-section rollup (Not started / In progress / Substantially covered for Sections 4-6 as a whole), not a per-topic count. This is what Step 7 checks before offering a priming session, and what Step 9 checks before decomposing into whichever phase it's about to invoke.

---

## Step 4 — Drift checks: cheap (every resume) and heavier (on trigger)

Run these automatically, every resume — "cheap" means two different things depending on the check, but neither ever means reading a full artifact:

1. **Version-lineage drift** — for every Lineage Ledger row, does "Upstream Version at Build Time" match "Upstream Current Version"? A mismatch means something changed upstream after the downstream was built against it. Genuinely free — both values are already sitting in the Lineage Ledger Step 3 just rebuilt; nothing new to read.
2. **Constraint-propagation drift** — has any fact, correction, or hard constraint been recorded (in an `AI Knowledge Corrections` table, an Open Issue, or similar) *after* an artifact that would be affected by it was already approved or locked? Check dates. This one isn't free the way check 1 is — Step 3 never pulls Corrections/Open-Issue rows into the Lineage Ledger, only version numbers — so this needs its own small, targeted read of that one specific table per artifact (grep the heading, per Step 3's "read lightly" rule), not a zero-read lookup.
3. **Orphaned open items** — any row in an Open Issues / AI Knowledge Corrections table still marked Open from a stage the project has since moved past, never resolved or explicitly deferred. Same targeted-read cost as check 2, same reason — this data was never captured into state either.
4. **Off-session gate closures** — compare each Stage Pointer's previously-recorded Status (from the state file as it was before this reconciliation) against that same artifact's current real Status, just read in Step 3. If a gate that was *not* Approved/Locked/Complete last time this state file was written now *is*, and this session hasn't driven that closure itself (we're still in Step 3/4 — Step 9 hasn't run yet this turn), flag it. This is a cheap string comparison of two already-known values, nothing new to read. It's exactly the case the heavier "gate legitimacy" check below exists for — a status that advanced without this session witnessing the real gate command run — except this session genuinely didn't see it happen, so silently trusting the field is exactly the risk to flag, not resolve.

Write anything found into the state file's Open Drift Flags list. Do not stop the session for these — they get surfaced in Step 5, not treated as a blocker on their own. For an off-session gate closure specifically, phrase the flag as an offer, not an accusation: *"{Artifact} shows as {Status} since last session, but this session didn't run its gate command — want a legitimacy spot-check (re-running its own Verification checklist) before continuing, or proceed as-is?"*

**The heavier checks** need real artifact content, not just metadata, which is why none of them run on every resume — they trigger only at a gate-close inside Step 9, when check 4 flags an off-session closure and the PO takes the offer, or when the PO explicitly asks for a drift check. Each is a real procedure, not just a label — and each is scoped to the one artifact actually involved in whatever triggered it, never a full-project re-scan:

5. **Traceability coverage** — three concrete comparisons:
   - Every REQ ID in the approved BRD's Section 5 appears exactly once in `epics/index.md`'s own REQ Coverage Map, assigned to some epic. Flag any REQ missing entirely, or one whose only mapped epic has since been deprecated.
   - `architecture/arch-v1.md` Section 3.1.3.3 (Screen Inventory) is architecture's own carried-over copy of `screens/screen-design.md` Section 3's Screen Inventory. Compare them row by row (Screen ID, Portal, Route, Purpose) — a mismatch means Screen Design was revised (via `/assess-change` or otherwise) after architecture copied it, and architecture's copy has gone stale.
   - Every journey in BRD Section 4 still maps to at least one Screen ID in architecture Section 3.1.3.3. Architecture's own Verification checklist already confirms this once, at generation time — only re-check it here for a journey the BRD gained *after* architecture was already Locked (a Pattern 4 change is the usual cause), since that's the one case architecture's own one-time check couldn't have caught.
6. **Ubiquitous Language consistency** — read the domain glossary's defined-terms sections (Core Entities, Common Personas & Roles, and any project-specific canonical list). Scan the *one artifact this check is running against* — not the whole project's document set at once, which is what keeps this bounded rather than an unbounded full-text search — for any term that plausibly means the same thing as a canonical term but doesn't match it, and isn't already covered by the glossary's own documented exceptions. Flag each one found, quoting the glossary's canonical term alongside it.
7. **Gate legitimacy** — read the specific artifact's own Verification (or Pre-Approval Verification) section from its template, and re-apply every item against the artifact's *current* actual content — not assuming it still passes because it presumably did once. Flag any item that would now fail. This is the concrete procedure behind check 4's spot-check offer — "spot-check" means running this against that one artifact, not a vague promise to look at it later.

---

## Step 5 — Present status and confirm direction

One short status readout, then one question. Not the full `/project-status` dashboard — that command remains available standalone once stories exist, and this readout is deliberately lighter:

```
{PROJECT_CODE} — {Project Name}
Stage: {current stage from Step 3} — {Approved/Locked/Draft/etc.}
Next: {the Command named in the first unsatisfied Lifecycle-at-a-Glance row, per Step 3 — or, past the Walking Skeleton handoff, whatever the real tracking artifacts indicate, the same way `/project-status` would}
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

**A topic carrying any unresolved Flag — of any type, from any source — never counts as a known answer for this check, regardless of its own Status field.** Status and Flag are independent in the priming package; a Flag always disqualifies a topic from silent substitution and forces it to surface as a live question when its owning phase command reaches it (see Step 7's `Conflicting Source` handling below for the concrete case).

**This interception covers an invoked command's own terminal "what's next" prompt too, not just mid-session content questions.** Several commands end by asking something that only makes sense standalone — `/init-project`'s closing message suggests running `/anchor-project` next; `/run-priming-session`'s closing message asks whether to continue with `/anchor-project`. When anchor itself is what invoked either, that question already has an obvious answer (yes, continue — anchor is already running), so intercept it the same way as any other question rather than surfacing it verbatim and waiting for the PO to confirm the obvious. Suppress the invoked command's own generic terminal report in this case too — Step 12's own session-close report replaces it, not supplements it, per the same 6.5 compression already applied to review output.

---

## Step 7 — Offer a priming session

**Corrected 2026-09-17:** document ingestion is no longer anchor's own logic — it moved to a standalone companion command, `/run-priming-session`, scoped to Problem Domain (Brief/Domain/BRD). This step is now a handoff, not a pipeline.

If the PO indicated they have source material or want to talk through raw facts (Step 5), or at any point asks to feed something in, and the current stage is still within Problem Domain (Brief, Domain, or BRD — not yet Locked/Approved past BRD):

> "That's what `/run-priming-session` is for — handing off to it now."

Invoke `/run-priming-session {PROJECT_CODE}` and let it run its own gate check, triage, and session in full, unmodified. `/run-priming-session` remains fully usable standalone too — this handoff is what happens when the request surfaces *inside* an anchor session.

**Once it completes, decomposition happens here, not there.** `/run-priming-session` never writes into `brief.md`, `domain-discovery-state.md`, or `brd-discovery-state.md` directly — per its own design, no existing command is modified to read `priming-package.md`. When Step 9 is about to invoke `/run-intake`, `/run-domain-discovery`, or `/run-brd-discovery`, first check `projects/{PROJECT_CODE}/priming/priming-package.md` for the matching section (4, 5, or 6) and pre-fill that command's own expected input from it — answering its live questions as the PO's own stated answers where a topic is `Covered`, and letting a genuinely `Pending` topic proceed as a normal live question. **A `Partial` topic is neither of those** — per Step 6's own "if unknown or partial, apply 6.1–6.4 before surfacing it," it is never silently substituted as a complete answer. Surface it as a live question the normal way, but an informed one: state what the package already has as the starting point (the same draft-and-correct pattern as 6.2) and ask specifically for what's still missing, rather than re-asking the topic from scratch or skipping it outright. This is the same interception mechanism Step 6 already describes for any PO-facing question, just sourced from the priming package instead of (or in addition to) what's already in state.

**Section 10 (Source-Specific Material) is handled differently from Sections 4-6.** If it has any content (per `conventions/priming-command-conventions.md` PC-041), it is never pre-filled into a specific phase command's field the way Brief/Domain/BRD Material is — it has no fixed home in any one phase's schema. Hand it to whichever phase command is about to run as background reference material instead, for that command's own judgment to use or set aside.

**Section 8 (Source Material Index) is handled the same way as Section 10** — it is a pointer to original source files, never an answer to pre-fill. Hand it to whichever phase command is about to run as background reference material, so that command can go consult the original file directly if its own live discussion needs more than the package's distillation.

**Section 7 (Terms As Used) is handed along the same way, but specifically to `/run-domain-discovery`, not every phase.** Per the document-level AI Guide, "which term becomes canonical is domain discovery's decision" — that is the one phase a flagged possible-synonym is actually actionable in. Hand it along as background material for that session's own glossary-building judgment when decomposition reaches Domain.

**Section 9 (Open/Uncategorized) is never pre-filled and never handed along silently — each still-unresolved row surfaces as a live question at the phase named in its own "where it probably belongs" column**, treated exactly like a genuinely `Pending` Sections 4-6 topic: a real question inside that phase's live discovery session, never an automatic write into any artifact. If the discovery session and the PO decide together, in that conversation, that the item is worth keeping but isn't ready to become a real requirement, it can become a candidate for the BRD's own Parking Lot (Section 15, `FW-024`) through that session's normal judgment — never through anchor writing it there directly. A row with no clear "probably belongs" guess defaults to surfacing at BRD discovery, the last Problem-Domain phase, so nothing goes unraised by default.

**A row whose "where it probably belongs" names an already-Approved/Locked artifact (e.g. "Domain — already Locked, needs `/assess-change`", per `conventions/priming-command-conventions.md` PC-015) is not a live-discovery-question case at all — there is no open session left to raise it in.** Surface it directly to the PO instead, at the point anchor would otherwise have raised it, and route it to `/assess-change` the same way Step 9 already handles any other non-linear request — never silently folded into whichever phase happens to be running now.

**A `Conflicting Source` flag on a topic is never pre-filled either — it becomes the live question itself, not a skipped one.** When decomposition reaches a flagged topic, present every conflicting statement with its source attribution directly, rather than a generic open question: *"{Source A} says X; {Source B} says Y — which is right, or did this change over time?"* Once the PO answers, it flows into the real artifact exactly like any other now-known answer, including 6.3's confirm-before-commit rule, since it is now binding content.

If the current stage is past Problem Domain (Architecture or later), a priming session has nothing left to usefully feed — say so and suggest `/assess-change` instead if the PO has new material that should change something already approved.

**A priming session ending is a natural stopping point, not an automatic segue into the next phase command.** Once `/run-priming-session` reports back, ask directly rather than assuming: *"That's captured. Continue now into `{next command}` with this material, or stop here for today?"* If the PO wants to stop, skip straight to Step 10 (state update) and Step 12 (session close) — do not drive Step 9 just because the sequence technically allows it. If they want to continue, proceed to Step 8 as normal.

Once this step completes (a priming session ran, or there was nothing to offer), continue to Step 8.

---

## Step 8 — Check post-handoff scope

Check `projects/{PROJECT_CODE}/stories/index.md` Section 8's `Walking Skeleton Complete` field directly (per `FW-048`) — this is cheap, real-artifact content, not something the state file needs to cache. If `stories/index.md` doesn't exist yet (Stories haven't been reached), or the field is not `Yes`, skip straight to Step 9 with no restriction.

**Once it is `Yes`, anchor's active involvement narrows to exactly two triggers** — do not drive routine story implementation, verification, or PRs; that is fully developer-led from here using the standard commands directly, with `/project-status` as the ongoing dashboard instead of this command:

- **Release preparation and closure** — `/close-sprint`, `/gen-release-notes`, and the judgment around them. Anchor drives these normally through Step 9, same as any other stage.
- **A Formal-tier `/assess-change`** — Pattern 3 (Scope Shift) or Pattern 4 (Requirement Change) specifically. Drive it the same way Step 9 hands off to `/assess-change` for any non-linear request. An informal Pattern 1/2 fix does not need anchor at all.

If the PO asks for anything else on a post-handoff project (implementing a specific story, reviewing a specific PR), say so plainly rather than driving it, and stop here for this session:

> "The Walking Skeleton is complete on this project — day-to-day story work is developer-led from here. I can still help with release prep or a significant scope change (`/assess-change`), but implementing individual stories isn't something I should be orchestrating once multiple people may be working in parallel."

Otherwise, continue to Step 9.

---

## Step 9 — Drive the current phase

Invoke the command named in Step 5's "Next" line, with `PROJECT_CODE` as its argument, exactly as if the PO had typed it directly. Load and follow its instructions in full — Step 6 is the only behavioral overlay; nothing about the invoked command's own logic, file formats, or gates changes.

**Judgment-check integration:** if the artifact about to enter a `/review-*` gate is a type `/judgment-check` covers (all types are covered as of the check's full rollout — brief, domain knowledge, BRD, epics, screen design, architecture, stories/sprint plan, story plan/PR description, UAT checklist/test execution report, release notes), run `/judgment-check` against it automatically first. This is not optional — it is the direct precedent that motivated this command's existence, and it stays a standing part of the gate sequence.

Judgment-check's own mechanics mean this takes one or two invocations, never something interactive on the first: a run against an artifact with nothing currently open only logs whatever it finds as `Open` and stops — it does not ask the PO anything in that same pass. If it logged anything, invoke it again immediately — this second pass finds those rows still `Open` and walks the PO through them interactively, and *that* pass is what folds into the same surfaced-only-what's-flagged pattern as Step 6.5. If the first pass found nothing, there is nothing to resolve — proceed straight to the `/review-*` gate.

**If mid-step the PO's request stops being "continue to the next stage" and becomes "change something already approved or locked":** this is a normal, expected event, not an edge case to route around. Stop the current flow and hand off:

> "That's a change to something already settled, not the next step in sequence. Running `/assess-change` for this instead."

Invoke `/assess-change {PROJECT_CODE}` and let it run its own classification and formal-change gate unmodified. `/assess-change` remains fully usable standalone too — this handoff is what happens when the request surfaces *inside* an anchor session, not a requirement that every change go through anchor.

When the step completes (a document written, a gate closed, a session ended for this turn), continue to Step 10.

---

## Step 10 — Update state after the phase step

Rewrite `projects/{PROJECT_CODE}/.anchor-state.md`:
- Update the Stage Pointer for whatever just changed (new Status, new Resume At if the phase is still mid-flight).
- Add or update the Lineage Ledger row for any artifact that was just written, recording the upstream version it was built against.
- If Step 7 ran a priming session this turn (whether or not Step 9 also ran afterward), refresh the Reference Material Index's `priming-package.md` row with its current Pending-topic and open-item counts — the same counts Step 3 reads on the next resume. This is the one case this step runs without Step 9 having produced anything, per Step 7's own stop-here branch.
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
This session:   {what actually happened — document written, gate closed, priming session run, etc.}
Drift flags:    {count, or "None"}
Observations:   {count logged this session, or "None"}
─────────────────────────────────────────
Next step:
{The command Step 5 would now point to on the next resume. Give the exact slash-command syntax.}
```

> "Resume any time with `/anchor-project {PROJECT_CODE}` — I'll pick up exactly here."

Do not add commentary beyond the report and that one line.
