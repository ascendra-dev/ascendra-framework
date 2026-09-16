# Anchor Project — Working Design Document

**Status:** In discussion — not decided, not implemented. This is a living document; update it in place as the design evolves across sessions.
**Working command name:** `/anchor-project`
**Started:** 2026-09-16
**Owner:** Zaka Shah (framework author)

**What this becomes once settled:** the working command name graduates to a real `.claude/commands/anchor-project.md`; the settled design decisions here get written up as one or more `decisions/FW-XXX-*.md` ADRs (this document is not itself an ADR — it's the scratch space that produces them); this file is then marked historical, the same fate `FRAMEWORK-AUDIT.md` names for itself.

---

## 1. The Problem

Running a project through the Ascendra Framework end to end means the PO carries the entire lifecycle in their head: which of the ~30 commands to run next, what state each artifact is in, when a gate is actually satisfied, and how to answer each command's discovery/review questions — all while the framework's own commands (correctly) go deep and dense at each phase, because that density is what makes their output good.

Two distinct frictions were identified, confirmed against a real end-to-end run (the THORNFIELD-MAINT-001 simulation):

1. **Sequencing and continuity burden** — remembering what's done, what's next, and picking the whole thing back up correctly after a gap, possibly days or weeks later, without the state itself becoming a second thing to maintain.
2. **Interaction load** — each phase's discovery/review commands ask real, necessary questions, but doing that consistently across 12 stages is tiring even for an expert PO, and prohibitive for a less experienced one. Separately: raw, messy source material (a legacy SRS, a pile of notes) has no home anywhere in the pipeline today — every command expects either a live conversation or an already-structured state file.

## 2. Non-Negotiable Constraints

These came out of discussion and bound every design choice below:

- **The existing framework must not regress.** A PO running any command directly, standalone, without the anchor, must get exactly today's behavior — same density, same rigor, same file formats. This ruled out an earlier draft of this design that proposed editing ~20 command files to adopt a shared "lighter interaction" convention. That draft is rejected — see §9 (Rejected Approaches).
- **Commands and templates are frozen.** Anchor is a pure additive layer: new files only. It never edits `.claude/commands/*.md` or `projects/TEMPLATE/**`.
- **Anchor is not full automation.** It keeps the PO in the loop at every stage and applies judgment to prevent drift — the goal is offloading burden, not removing oversight.
- **State must stay lightweight** — safe to reread every session without degrading context. Substantive content always lives in real project artifact files, never in "state."
- **A PO who wants the direct, undiluted, full-detail experience can always get it** — either by running commands standalone (bypassing anchor entirely) or via the in-session escape hatch (§6.4) while anchored.
- **Anchor never fast-forwards a gate.** A Status field only ever advances by actually running the real gate command (`/review-X`, or that command's inline lock/approve step) — never via `/update-status`'s manual fallback as a shortcut to keep the sequence moving. That fallback exists for the PO's own deliberate use; anchor reaching for it on the PO's behalf would itself create the "gate legitimacy drift" §7.2 item 6 exists to catch.

## 3. Core Mechanism — One Agent, Two Hats

This is the load-bearing design fact everything else depends on, and it was worth clarifying precisely (see discussion 2026-09-17):

There is no separate "anchor" process and "domain-discovery" process exchanging messages. It is the same Claude, in the same conversation, sequentially following different files' instructions — the same mechanism the framework already uses today when `/gen-architecture` reaches its Section 6 and starts following `/gen-ui-mocks`'s instructions in the same breath.

So: **anchor invokes `/run-domain-discovery` (or whichever command), loads its instructions in full, and executes them itself** — including all of that command's own logic for writing its state file, scoring confidence, maintaining its Open Issues table. None of that is reimplemented by anchor. The *only* deviation happens at one narrow seam: **wherever the invoked command's instructions direct it to ask the PO something, anchor intercepts that single moment** — checks whether the answer is already known from state or reference material, and if not, applies the interaction protocol (§6) before surfacing it. The moment an answer exists, control returns to the invoked command's own logic exactly as if the PO had answered directly and at full length. The invoked command writes its own file, in its own format, unaware whether the answer came via anchor's simplification or a raw session.

This is why the design can be commands-frozen and still deliver a lighter PO experience: the change lives entirely in *how a question gets composed and how an answer gets translated back*, never in what gets written down or how.

## 4. State Model

Two categories, kept strictly separate:

### 4.1 State (tiny, pointer-only, rewritten every step, safe to reread every session)

- **Stage pointers** — per phase, current status (matches each artifact's own Status field: Draft / Under Review / Approved / Locked), and, when a phase is mid-flight, where exactly to resume (e.g. "BRD discovery, session 2 of ~3, next: Section 6").
- **Lineage ledger** — for every artifact, which version of which upstream artifact(s) it was built against, and when. E.g.: Epics v1 built against BRD v1.0 on 2026-09-16. This is the data source the anti-drift check (§7) runs against — without it, "check for drift" has nothing to compare.
- **Reference-material index** — pointers to what extraction/reference artifacts exist (§5) and a count of unresolved open items in each, not their content.

Nothing else lives in state. If it's substantive, it's a real file, not a state entry.

### 4.2 Reference material (real project files, not state)

Everything produced by document ingestion (§5) or accumulated PO answers that isn't already captured by an existing template — written as durable, properly organized files under the relevant `projects/{CODE}/` phase folder. Read on demand when relevant, never held in "memory."

### 4.3 Reconciliation and bootstrap — state is derived, not authoritative

State is a cache of what the artifacts already say, never a second source of truth competing with them. Every time anchor resumes, it reconciles its stage pointers and lineage ledger against the actual current state of the artifact files (their own Status fields, Document Control / Document History tables) rather than trusting what it last wrote — this is what keeps anchor correct if a file was edited outside of anchor, or if two people alternate running sessions on the same project.

This also makes **adopting an already-in-progress, non-anchor-driven project a first-class case, not a special one** — a real, immediate example being THORNFIELD-MAINT-001, currently mid-lifecycle (Architecture: Draft) with no anchor state at all. If state is missing, stale, or was never created, anchor bootstraps it by scanning the existing artifacts: infer each phase's status from its own Status field, rebuild the lineage ledger from the version numbers already recorded in each artifact's own Document History section, and initialize from that — no separate "import" step needed.

## 5. Document Ingestion Mode

A mode of `/anchor-project`, not a change to any existing command, and not phase-1-only. Usable whenever the PO has raw material for whatever phase is currently active — a legacy SRS, scattered notes, an old requirements doc, anything, in any order, possibly self-contradictory, possibly large.

### 5.1 Zero-new-format principle

Ingestion's output target is **whichever input mechanism the current phase's command already reads** — never a bespoke new format:

- Where an intermediate state file already exists for the phase (`domain-discovery-state.template.md`, `brd-discovery-state.template.md`), ingestion triages the source material and writes directly into that template's existing sections — the downstream command (`/gen-domain-knowledge`, `/gen-brd`) runs completely unmodified, unaware whether the state file came from a live discovery session or from triage.
- Where no such intermediate file exists (Phase 1 — `/run-intake` writes straight to `brief.md`), ingestion pre-extracts answers and feeds them in as the PO's own answers when `run-intake`'s questions come up.

### 5.2 Multi-artifact triage, not one flat dump

Real source material carries genuinely distinct kinds of knowledge — domain entities/rules, integrations (count, sync/async, technology, third-party contract shape), personas, historical constraints. Triage classifies by kind and routes each into its own properly organized file (e.g. an integration catalog is a real, reusable artifact useful to `/gen-architecture` later — not a footnote in a scratch file), written to be read by a human stakeholder, not just by the AI.

### 5.3 Legacy material — don't let a bad system distort the domain model

Every extracted fact gets checked against the AI's own generic/canonical domain understanding before being written anywhere:

- **Matches the generic model** → ordinary domain content, written normally.
- **Diverges, and looks like a legitimate business reason** (regulatory, contractual, genuinely how this business operates) → goes into the domain-discovery-state file's existing **Section 5, AI Knowledge Corrections** table (`Topic | AI Assumption | PO Correction | Confidence`) — a mechanism the framework already has, not a new one. `/gen-domain-knowledge`'s existing rule ("PO's answer takes precedence, note the divergence") does the rest, unmodified.
- **Diverges, and looks like a legacy design flaw rather than a real constraint** (bad normalization, an accidental workaround that calcified) → written as neither content nor a silent correction. It becomes an explicit Open Issue framed as a negotiation: *"the legacy system does X this way — real constraint, or something to leave behind in the upgrade?"* This is the PO negotiation point, not a default.
- Same posture applies to integration modernization (e.g. legacy sync integration → propose async) and legacy contract renegotiation with a third party — these are surfaced as discussion items with a recommendation, never carried forward silently just because "that's what the old system did," and never decided by anchor unilaterally.

### 5.4 Constraints discovered mid-stream are a drift trigger, not just a note

If a hard fact surfaces later than expected (example used in discussion: learning at BRD stage that the same database schema is a fixed constraint), this is structurally the same problem as an artifact version bump — something material is now true that earlier, already-approved artifacts didn't know about. It feeds the same lineage/drift mechanism (§7) via a second trigger type: not just "upstream artifact version changed" but "a new hard constraint was recorded," checked against everything already built or approved that might assume otherwise.

### 5.5 Composability

Ingestion's input contract is source-agnostic: a document plus which phase it's informing. It doesn't matter whether the document was pasted by the PO, uploaded, or produced by some other tool/command entirely (e.g. a hypothetical future "extract from legacy code" command). Anchor only needs a document and a target phase — this keeps the door open to composing anchor with other tooling later without redesigning the ingestion step.

### 5.6 Sensitive content

Legacy material can carry credential-shaped strings (API keys, tokens, passwords) copy-pasted into old notes or an old SRS. `projects/{CODE}/` being gitignored is not the same as safe to persist verbatim — triage should recognize obviously sensitive strings and flag them to the PO rather than silently filing them into a catalog document.

### 5.7 Volume handling

"It can be massive" was named explicitly as a real scenario. Triage of a genuinely large source document should proceed as an iterative, checkpointed pass rather than assume everything fits in one context window — the checkpoint (how far into the source triage has gotten) is itself lightweight state (a pointer, per §4.1), not a reason to re-hold the whole source in memory across turns.

## 6. PO Interaction Protocol

### 6.1 Compression, not simplification-for-a-layman

Correction made during discussion, important: the PO retains full capability in anchored mode — this is not a "explain like I'm new to this" mode. The goal is removing unnecessary volume and lead-up, not vocabulary or sophistication. Default behavior uses domain terms, glossary vocabulary, and framework terms freely, by default, without pre-explaining them. Plain-language/example mode is offered only reactively, when the PO signals they don't follow something — never assumed proactively.

**Exception:** recurring *framework-structural* concepts that are consistently hard to internalize and expensive to get wrong later — Scope, Priority, Layer, Core/Extension — get one short anchoring note (one or two sentences, not a lecture) the first time each becomes load-bearing for a decision in a given project. This is a deliberate, narrow exception to "terse by default," not a general onboarding mode.

### 6.2 Bounded question loops — draft-and-correct

Not "ask until satisfied" (unpredictable, fatiguing) and not a fixed count (too rigid for variable topic complexity). Pattern: ask up to ~3 short questions on a topic, then stop asking and produce a draft with assumptions stated explicitly ("Assuming X because Y — correct me if wrong"). Correction-by-example against a concrete draft is lower PO effort than continued interrogation, and it's already how the rest of the framework works (every `/gen-X` drafts, every `/review-X` corrects) — this just applies the same pattern one level earlier, at the question level as well as the document level.

### 6.3 Fidelity on write-back

Compression happens twice in this design — once simplifying the question, once translating the PO's answer back into precise, template-correct language. For anything that becomes **binding** (a Universal Business Rule, an entity definition, REQ text — anything that becomes Ubiquitous Language or gets locked into a gated artifact), anchor confirms the translated answer back to the PO in one line before committing it to the underlying file — draft-and-correct applied at the single-answer level, not just the whole-document level. For anything **cosmetic** (nav labels, phrasing, non-binding detail), no confirmation step is needed — the weight of the confirmation should match the weight of what's being recorded.

### 6.4 Escape hatch

At any point, the PO can ask for the raw, full-detail version of a question or topic ("show me the full version" / "let me answer this directly"), and anchor drops the filter immediately, no resistance. Default persistence scope is the rest of the current section — but the PO's own wording can narrow it ("just this one") or extend it ("full detail from here"), and anchor respects whichever scope was actually stated. Whenever anchor is in detail mode, it states in the same message how to step back down (e.g. "say 'simplify' any time to go back to the short version") — the PO is never left to guess how to return to the lighter mode. Detail-mode does not carry across a session boundary by default — each new session resumes in compressed mode, since state stays lightweight and doesn't record mid-topic UI preference (§11 notes this as worth revisiting if it proves annoying in practice).

### 6.5 The same compression applies to review output, not just discovery questions

`/review-X` commands walk a dense list of checks (e.g. `/review-brd`'s full section-by-section pass) — the same density problem exists on the way *out* of a command as on the way in. Anchor applies the identical principle here: surface only what's flagged or uncertain, one line each; batch every "no issue found" check into a single summary line rather than reciting it. Full detail is available on the same escape hatch as §6.4, per check or for the whole review.

## 7. Anti-Drift — Cross-Artifact Coherence

This is the orchestrator's actual unique contribution — not sequencing convenience, but a function nothing today performs: watching *across* phases as they complete, since within a phase, drift is already guarded (`/review-X` gates, `/judgment-check`).

### 7.1 Cheap checks — metadata-only, run automatically on every resume

Driven entirely by the lineage ledger and open-item counts already in state, so "run constantly" isn't a burden:

1. **Version-lineage drift** — for every recorded upstream→downstream pair (Domain→BRD, BRD→Epics, BRD/Epics→Screen Design, Screen Design/BRD/Epics→Architecture, Architecture→Stories), compare the version the downstream was built against to the upstream's current version. Mismatch → flag.
2. **Constraint-propagation drift** — any fact/correction/constraint recorded after a given artifact was approved, checked against that artifact's scope (§5.4's trigger).
3. **Orphaned open items** — Open Issues / AI Knowledge Corrections rows still marked Open from a stage the project has since moved past, never resolved or explicitly deferred.

### 7.2 Heavier checks — content-level, run at gate-close or on explicit request

4. **Traceability coverage** — every REQ resolves to an epic (cross-check the REQ Coverage Map that already exists in `epics/index.md`); every epic's referenced screens exist in Screen Design; every "carried over from screen-design.md" claim in Architecture still matches the current screen-design.md.
5. **Ubiquitous Language consistency** — terms used downstream (stories, architecture) checked against the domain glossary for unrecorded synonym drift. This is CLAUDE.md's existing rule; nothing today checks it proactively across a whole project.
6. **Gate legitimacy** *(lower priority)* — an artifact marked Approved/Locked whose own template verification checklist wouldn't actually pass if re-run now; catches a status advanced via `/update-status` without the real review gate having run.
7. **Judgment-check coverage** — for artifact types `/judgment-check` already covers (BRD, Epics, Architecture today), anchor runs it automatically before presenting the matching `/review-X` gate to the PO, and folds its findings into the same surfaced-only-what's-flagged pattern as §6.5 — this is the direct precedent that motivated this whole command, and it should be a standing part of anchor's gate sequence, not an optional extra.

### 7.3 Non-linear requests — mid-project changes

Everything above assumes forward sequencing (stage N → N+1). A PO wanting to revise something already approved or locked is a normal, expected event, not an edge case anchor can ignore. Anchor recognizes when a PO's request isn't "move to the next stage" but "change something already settled," and routes it to `/assess-change` rather than plowing forward as if the project were still purely linear. The resulting version bump feeds the lineage ledger and the drift triggers in §7.1 exactly like any other change.

### 7.4 Extension projects — lineage crosses project boundaries

The lineage ledger as described in §4.1 tracks upstream→downstream pairs within one project. An extension project's domain knowledge and/or BRD depends on a **base** document that lives in a different project's folder (per the framework's Core/Extension model). Anchor's version-lineage check (§7.1 item 1) needs to know to look there too — an extension project can drift not only against its own prior artifacts but against a base document it doesn't own. Flagged here; full handling isn't settled — see Open Items.

## 8. Self-Improvement Logging

Kept fully separate from `decisions/` — a raw waiting room, not a decision log:

- `observations/index.md` — registry: ID, one-line title, Status, Date, Area
- `observations/TEMPLATE.md` — its own lighter template (this is a raw finding, not a ratified decision): what was observed, where/when, why it matters, a proposed direction
- `observations/OBS-001-{slug}.md` per finding
- Status field distinct from decisions': `New` → `Under Review` → `Promoted → FW-XXX` / `Rejected` / `Won't Fix`

Promotion path: the framework author reviews an observation and, if worth acting on, writes it up as a real `FW-XXX` in `decisions/` using the existing decision template. Observations never write into `decisions/` directly, and are never auto-applied.

## 9. Solution Domain Boundary — The Walking Skeleton Handoff

This resolves what was previously the biggest open item in §11 (Phase 6–9 scope), using a real mechanism, not a placeholder.

**The problem it resolves:** anchor's entire design assumes single-session, single-actor state — one lightweight pointer file, one lineage ledger, reconciled by whoever's driving (§4.3). That assumption holds all the way through Problem Domain and Architecture, and holds through a first thin end-to-end slice, because that's still one thread of work. It stops holding the moment multiple developers are implementing and verifying separate stories in parallel — now there are concurrent writers to state that was designed for one.

**The boundary:** anchor drives architecture through to Locked, generates the sprint plan, then leads execution of exactly the stories tagged `Walking Skeleton: Yes` (`FW-048`) through `implement-story` → `verify-story` → PR — the smallest real, thin, end-to-end slice that proves the architecture's layers actually connect. The moment `stories/index.md` Section 8's `Walking Skeleton Complete` field flips from `No` to `Yes` — every Walking Skeleton story reaches `Done`, QA-verified, written by `/update-status` per FW-048 — anchor's active involvement ends there. Execution hands off to normal developer-led sprints using the standard commands directly, with `/project-status` as the ongoing dashboard instead of anchor.

This is a mechanical, checkable trigger, not a judgment call anchor has to make each time — `FW-048` was written specifically to make this boundary real rather than something anchor would otherwise have to eyeball. Release prep, `/close-sprint`, and similar post-handoff work stay with a lead (not necessarily the PO) rather than reverting to anchor — that's single-actor work again, just not anchor's actor.

**Still open:** whether anchor is fully done with the project after handoff, or returns for the next project-level gate (a final release, or a later `/assess-change` big enough to warrant re-involving it) — not yet decided, carried into §12 (Open Items).

---

## 10. Rejected Approaches (kept for the record)

- **Editing ~20 existing commands to adopt a shared "lighter interaction" convention file.** Rejected because it violates the frozen-commands constraint (§2) and creates a second place PO-interaction logic could drift from itself over time (the version baked into the shared convention vs. each command's own copy). Superseded by §3 (interception at the invocation seam, zero file edits).
- **A single fused command that reimplements discovery/review interaction itself instead of invoking the real commands.** Rejected for the same reason — would duplicate logic that already exists correctly in each phase's command, and a PO running that command standalone would get a second, divergent experience instead of today's.

## 11. Invocation

`/anchor-project` takes no required arguments. Bare invocation:
- Looks at `projects/index.md`. Exactly one active project → resume it. Several → ask (one short question) which. None → route to `/init-project`.
- Naming a project (e.g. `/anchor-project THORNFIELD-MAINT-001`) still works directly — useful once several projects are running in parallel, which is the framework's stated target usage pattern.
- For project identity specifically (the four-part code, name, client, domain a PO often struggles to form) — anchor proposes the identity fields itself as a draft and asks for confirmation, rather than asking the PO to originate them.

A PO who wants the standalone, unanchored experience simply runs any command directly, as today — anchor is optional at every level, not a required front door.

## 12. Open Items — Not Yet Settled

- **Post-handoff role.** Per §9, does anchor return to a project after the Walking Skeleton handoff — for a final release, or a later `/assess-change` big enough to warrant it — or is its involvement fully done once handoff happens? Not yet decided.
- **`FW-048` implementation must land before anchor does.** §9's handoff mechanism depends on `stories/index.md`'s `Walking Skeleton Complete` field existing, which depends on `FW-048` (Walking Skeleton Milestone) being applied across the BRD/Epics/Stories/Sprint Planning pipeline — done as of 2026-09-17, ahead of `/anchor-project` itself, exactly as sequenced.
- **Validation plan for anchor itself before treating it as ready.** Two candidate approaches, not mutually exclusive: (a) adopt THORNFIELD-MAINT-001 mid-flight via anchor, which directly exercises the bootstrap/reconciliation case in §4.3 since that project has no anchor state at all today (and predates `FW-048`, so it also has no Walking Skeleton tagging — worth deciding whether to backfill it or treat THORNFIELD as a pre-FW-048 case); (b) run a fresh second simulation entirely through anchor end-to-end, the same way THORNFIELD itself validated the base framework before `/judgment-check` was layered on.
- **Drift-check automation depth** — §7.1's "cheap checks run automatically every resume" vs. only at gate-close — leaning toward both (cheap on every resume, since lineage makes it near-free; full pass at gate-close) but not confirmed.
- **Escape-hatch persistence across sessions** (§6.4) — currently defaults to resetting each session; revisit if that proves annoying for a PO who consistently wants full detail on a given project.
- **Extension/base-document lineage** (§7.4) — flagged as a real gap, not yet resolved how the lineage ledger represents or checks a dependency that crosses project boundaries.
- **Exact state file / lineage ledger file name and location** — naming TBD at implementation time (candidate: `projects/{CODE}/.anchor-state.md` or similar); not load-bearing for the design itself.
- **Whether `observations/` should also capture command-level friction the way `FRAMEWORK-AUDIT.md` did as a one-off**, or stays scoped to anchor's own findings — not yet discussed.
- **Command file structure** — once this design is settled, `.claude/commands/anchor-project.md` still needs to be authored against `conventions/command-conventions.md` (FW-020) like every other command; not started.

---

## Document History

| Date | Change |
|------|--------|
| 2026-09-16 | Initial consolidation of design discussion (mechanics, interaction protocol, ingestion, anti-drift axes, self-improvement logging, invocation semantics) |
| 2026-09-16 | Full review pass: added review-output compression (§6.5), state reconciliation/bootstrap (§4.3), no-gate-fast-forwarding constraint (§2), non-linear request routing (§7.3), extension/base lineage gap (§7.4), judgment-check integration (§7.2 item 7), sensitive-content and volume handling in ingestion (§5.6, §5.7); flagged Phase 6-9 scope as the one significant unresolved fork (§11) |
| 2026-09-17 | Resolved the Phase 6-9 scope fork using the Walking Skeleton concept (real precedent found in a sibling project, `ascendra-pay-002`, not previously part of this framework): added §9 (Solution Domain Boundary — The Walking Skeleton Handoff), sections renumbered accordingly (Rejected Approaches → §10, Invocation → §11, Open Items → §12). Formalized the underlying concept into the framework itself first, ahead of anchor, as `decisions/FW-048-walking-skeleton-milestone.md` — implemented across the BRD/Epics/Stories/Sprint Planning pipeline and the practitioner guide — so §9's handoff trigger (`stories/index.md`'s `Walking Skeleton Complete` field) is a real mechanism, not a placeholder |
