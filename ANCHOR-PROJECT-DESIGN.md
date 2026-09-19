# Anchor Project — Working Design Document

**Status:** Implemented and internally consistent as of 2026-09-19. `.claude/commands/anchor-project.md` (Step 7 rewritten to hand off to `/run-priming-session`, all cross-references re-verified after the reorder; further extended same day for Partial-status handling, Section 7/8/9 consumption, and the already-locked-topic case; Step 11 rewritten in full same day for §8's journal/extraction redesign — see below), `.claude/commands/run-priming-session.md`, `.claude/commands/init-project.md` (also gained `journal.md` scaffolding), `projects/TEMPLATE/priming/priming-package.template.md` + `.example.md`, `projects/TEMPLATE/journal.template.md` + `.example.md` (new), `conventions/priming-command-conventions.md` (§5.15 through §5.18, all added 2026-09-19), `conventions/template-conventions.md` (one new exception, `T-042`), and `observations/index.md` + `TEMPLATE.md` (repointed at journal-sourced extraction) all exist and match this document. Converted into `decisions/FW-049-anchor-project-and-priming-session.md` (amended 2026-09-19 for §5.15 through §5.18) and `decisions/FW-050-project-journal-and-framework-learning.md` (new, for §8). Still pending: validation against a real project (§12's remaining item) — now covering both `FW-049` and `FW-050` together, since neither has been exercised by a real anchor session yet.
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

## 5. The Priming Session — `/run-priming-session`

**Corrected 2026-09-17:** an earlier version of this section described ingestion as a mode of `/anchor-project` itself, usable at any phase. Both halves turned out to be wrong once the composability principle was actually applied to this design: embedding a source-agnostic, document-processing mechanism inside the one command that isn't source-agnostic (anchor is a sequencer) was the same kind of internal contradiction §9 in the earlier draft warned against elsewhere. It is now a **standalone companion command**, `/run-priming-session`, fully usable on its own — not a mode, not phase-1-only in the old sense, but scoped specifically to **Problem Domain** (Brief, Domain, BRD) and nothing past it. Epics, Screen Design, and Architecture are AI synthesis performed *from* an approved BRD, not raw PO knowledge — there is nothing for a front-loaded session to capture there. (This is the same category of mistake §10, Rejected Approaches, already warned against for two other shapes this design considered and discarded — a third instance, caught later than the first two.)

### 5.1 What it produces, and where it comes from

Output: `priming-package.template.md` (new location: `projects/TEMPLATE/priming/`). It is a new *file*, not a new *schema* — its sections are composed from `brief.template.md`, `domain-discovery-state.template.md`, and `brd-discovery-state.template.md`'s own existing structure, not hand-invented. A future field added to any of those (the way `Walking Skeleton` just got added to two of them by `FW-048`) is inherited automatically rather than drifting out of sync — this is the concrete answer to "does this limit anchor's capability": nothing here can fall behind the templates it mirrors, because it isn't a separate thing to keep in sync, it's built from them.

**Density is calibrated to match, not exceed.** The package sits at discovery-state-file density — short "PO stated" distillations per topic — never final-artifact density. It is explicitly not trying to *be* the BRD or the domain knowledge document; it materializes facts as a head start for the sessions that actually produce those, nothing more. Nothing downstream is skipped, shortened, or has its rigor reduced by this command existing.

### 5.2 The session itself — free-form, no fixed order

No script, no fixed question order, no template governing *how* the conversation goes — genuinely informal, topics can come up in whatever order the PO raises them. What the session owes the PO before closing is not completeness, it's clarity: a clear sense of what got captured and what didn't, using the same Covered/Pending convention the mirrored templates already carry. **100% coverage was never the bar.** Gaps are expected and fine — `/run-domain-discovery` and `/run-brd-discovery` still run afterward and fill them, exactly as they do today when no priming session happened at all. This command's existence changes where those sessions start from, not whether they still run in full.

### 5.3 Reproducibility — structural completeness, not identical output

A real question: since this is an adaptive, AI-conducted conversation, running it twice on the same source material won't produce identical output — different wording, different order, possibly different things noticed. The framework already has an answer to exactly this problem, because `/run-domain-discovery` and `/run-brd-discovery` are *also* free-form adaptive sessions and face the same variance. Neither is made deterministic to solve it — instead, the *output* is made structurally checkable regardless of wording: a Confidence Score, a Covered/Pending flag per section, a Pre-Generation Verification checklist that gates whether the file is usable yet at all. Two different priming sessions might disagree on phrasing; both get held to the same structural bar before anything downstream trusts them. This command reuses that mechanism rather than inventing a new one.

### 5.4 No Ubiquitous Language yet

The domain glossary this framework enforces conformance against (`CLAUDE.md`'s own rule) doesn't exist until domain discovery actually establishes it — so at this session, there is nothing to enforce conformance *to*. Capture the PO's own terms exactly as used, without normalizing. Where the conversation seems to use two different names for the same concept, flag it — *"PO said both 'Client' and 'Customer' for what looks like the same thing"* — rather than silently picking one. Which term becomes canonical is domain discovery's decision, not this session's; resolving it here would be presumptuous and would risk losing a distinction that turns out to matter.

### 5.5 The three-pass pipeline: map, triage, extract

The mechanism behind "top-down triage" is three distinct passes, not one undifferentiated read-and-write:

1. **Map** — skim only, no deep reading. Build a lightweight outline of the source: what's in it and roughly where, one short gist per unit. Cheap even for a large document, since nothing is deep-read yet.
2. **Triage** — against that map, and the Problem Domain sections the package needs to fill, mark each unit relevant / redundant (covered by another unit already) / irrelevant / ambiguous. Irrelevant units are dropped **without** being deep-read at all — this is what keeps a genuinely large or noisy source cheap to process. Redundant units are flagged so the same fact doesn't get extracted twice from two places. Each relevant unit is also compared against whatever its target topic already holds — not only for redundancy but for disagreement — regardless of whether the existing content came from an earlier unit in the same source, a different source, or something the PO already stated this session; see §5.16.
3. **Extract** — deep-read only what survived triage, and write it into the priming package (per §5.9) compressed and deduplicated — never a verbatim transcription, never the same fact restated because it appeared in two source units.

**Structured vs. unstructured sources use the same three passes, but the Map step adapts to what's actually there:**
- A source with real structure (an SRS, a spec, anything with real headings) — Map builds its outline directly from that structure, and Triage works section by section.
- A source with no reliable structure (raw notes, a handwritten dump, stream-of-consciousness fragments) — Map has no headings to skim by, so it clusters fragments by inferred topic instead, before Triage can ask "relevant or not" of each cluster. Same pipeline, same discipline, different technique for the one step that depends on the source actually being organized.

**Transparency:** persist the Map + Triage decisions themselves (not just the final extraction) — **simplified during implementation** from a separate `extraction-log.md` file to the priming package's own Section 8 (Source Material Index), which already records file, purpose, and when to reference it back; a second file duplicating the same information wasn't worth the extra artifact. One line per source unit: gist, decision, destination if kept. This is what makes ingestion legible rather than a black box that silently dropped content, and it's also where §5.8's checkpoint pointer lives for a source too large to process in one pass.

### 5.6 Legacy material — don't let a bad system distort the domain model

Every extracted fact gets checked against the AI's own generic/canonical domain understanding before being written anywhere:

- **Matches the generic model** → ordinary domain content, written normally.
- **Diverges, and looks like a legitimate business reason** (regulatory, contractual, genuinely how this business operates) → goes into the priming package's Domain section under the same AI Knowledge Corrections shape `domain-discovery-state.template.md` already has (`Topic | AI Assumption | PO Correction | Confidence`) — mirrored, not reinvented, per §5.1. `/gen-domain-knowledge`'s existing rule ("PO's answer takes precedence, note the divergence") does the rest, unmodified, once this content is decomposed into the real discovery-state file.
- **Diverges, and looks like a legacy design flaw rather than a real constraint** (bad normalization, an accidental workaround that calcified) → written as neither content nor a silent correction. It becomes an explicit Open Issue framed as a negotiation: *"the legacy system does X this way — real constraint, or something to leave behind in the upgrade?"* This is the PO negotiation point, not a default.
- Same posture applies to integration modernization (e.g. legacy sync integration → propose async) and legacy contract renegotiation with a third party — these are surfaced as discussion items with a recommendation, never carried forward silently just because "that's what the old system did," and never decided unilaterally.

Note: this is about a document *describing* a legacy system (an old SRS, migration notes) — prose, handled by this command. Extracting directly from a legacy *codebase* is a different mechanic entirely; see §5.11.

### 5.7 Sensitive content

Legacy material can carry credential-shaped strings (API keys, tokens, passwords) copy-pasted into old notes or an old SRS. `projects/{CODE}/` being gitignored is not the same as safe to persist verbatim — triage should recognize obviously sensitive strings and flag them to the PO rather than silently filing them into the package.

### 5.8 Volume handling

"It can be massive" was named explicitly as a real scenario. Triage of a genuinely large source document should proceed as an iterative, checkpointed pass rather than assume everything fits in one context window — the checkpoint (how far into the source triage has gotten) is itself lightweight state, not a reason to re-hold the whole source in memory across turns. This is the same checkpoint §5.5 already places in the priming package's Section 8 — one mechanism, not two.

### 5.9 The staging model — one document, decomposed later

The session produces **one consolidated package** the PO reviews as a single, coherent thing — not three scattered files that each feel unfinished on their own. Decomposition into `brief.md`, `domain-discovery-state.md`, and `brd-discovery-state.md` happens later, when anchor actually reaches each of those phases — not speculatively ahead of time. This keeps state-is-derived-not-authoritative (§4.3) honest: the decomposition is reconciled against what's actually true at the moment it happens, not pre-committed early and left to go stale. No existing command ever reads the priming package directly — `/run-intake`, `/gen-domain-knowledge`, `/run-brd-discovery`, and `/gen-brd` all keep reading exactly the file they already expect; anchor is what pre-fills it from the package, at the right moment, before invoking them.

### 5.10 Constraints discovered mid-stream are a drift trigger, not just a note

If a hard fact surfaces later than expected (example used in discussion: learning at BRD stage that the same database schema is a fixed constraint), this is structurally the same problem as an artifact version bump — something material is now true that earlier, already-approved artifacts didn't know about. It feeds the same lineage/drift mechanism (§7) via a second trigger type: not just "upstream artifact version changed" but "a new hard constraint was recorded," checked against everything already built or approved that might assume otherwise.

### 5.11 One command per mechanic, not per format

`/run-priming-session` handles prose/document material — an SRS, freeform notes, any reference material, regardless of format — because all of it is read the same way (§5.5's pipeline, adapting technique for structured vs. unstructured, never the command itself). A **fundamentally different extraction mechanic** gets its own command instead of bloating this one. Legacy **code** is the concrete case: navigated structurally (modules, entry points, schema/ORM files), not read linearly, and it needs a reverse-engineering step this command doesn't — translating "what the code does" into "what business capability it reflects." That's a planned second command, deferred — chunking strategy for large codebases still needs its own design pass. A third genuinely-different mechanic later (an existing OpenAPI spec, a live DB schema) gets a third command when it actually arrives, not a rewrite of the first two.

Each mechanic-specific command may also contribute its own **Source-Specific Section** to the shared package (governed by `conventions/priming-command-conventions.md`, added 2026-09-19) — see §5.15 for the concrete mechanism. This is the resolution to a gap this principle implied but hadn't yet solved: a different mechanic doesn't just mean a different way of reading, it can mean a different *kind* of fact, one that never had a home in the package's original Brief/Domain/BRD-mirrored taxonomy.

### 5.12 Where source material actually lives

`/init-project` gains one small, additive change: a `source-material/` folder created as part of its normal skeleton, so there is somewhere to drop files before priming even runs. **Sequencing:** `/run-priming-session` runs *after* `/init-project`, not before — it needs that folder to exist. It reads from `source-material/` and also accepts material pasted directly into the conversation. Running it more than once on the same project is expected and supported — new material merges into the existing package rather than starting over or erroring.

### 5.13 Section skeleton (as built, current as of the 2026-09-19 Source-Specific Sections amendment)

```
priming-package.template.md
├── Section 1: Session Context (Project Code, Client, source-material folder)
├── Section 2: Session History (one row per session — date, who, sections touched)
├── Section 3: Coverage Snapshot (Sections 4-6 at a glance — Not started /
│              In progress / Substantially covered, no numeric score, see §5.3)
├── Section 4: Brief material (mirrors brief.template.md's sections)
├── Section 5: Domain material (mirrors domain-discovery-state.template.md
│              Section 4's playbook-progress structure — Covered/Pending per topic)
├── Section 6: BRD material (mirrors brd-discovery-state.template.md similarly)
├── Section 7: Terms as used (the UL flag bucket from §5.4 — no normalization)
├── Section 8: Source Material Index (which files were fed in, one-line purpose
│              each, when to cross-reference later — not a full read)
├── Section 9: Open / Uncategorized (Parking-Lot-style overflow — a genuine
│              one-off with nowhere to go, nothing dropped just because it
│              doesn't fit above; see §5.15 for how this differs from Section 10)
├── Section 10: Source-Specific Material (§5.15 — a mechanic-specific command's
│               own attributed subsection(s) for an expected category of fact
│               that never maps onto Sections 4-6; "None" by default)
├── Section 11: Resume Instructions (penultimate — what to load, where the next
│               session starts, what's still Pending, open flags)
└── Section 12: Pre-Handoff Verification (final — blocking checks before handoff)
```

### 5.14 Relationship to anchor

Fully standalone-runnable, like every other command (§2's principle extends here) — a PO can run `/run-priming-session` directly without ever touching anchor. When anchor is driving (its own ingestion step, now much lighter than the earlier draft): if the PO wants to feed material in and no priming package exists yet, anchor invokes `/run-priming-session` and resumes once it completes. Anchor no longer contains any triage logic of its own — that entire mechanism moved here.

### 5.15 Source-Specific Sections — added 2026-09-19

**The gap:** §5.1's "not a new schema, mirrors the templates" reasoning is sound for the command that existed when it was written — prose only, one mechanic. §5.11 already planned for further mechanic-specific commands (legacy code first, deferred; an OpenAPI spec or live DB schema later) but didn't yet address a consequence of that plan: a different mechanic doesn't only mean a different way of *reading*, it can mean a different *kind* of fact surfacing — an existing API surface, schema-as-implemented, business rules embedded in validation logic, a tech-debt inventory. None of that compresses into Brief/Domain/BRD vocabulary. Before this amendment, the only place for it was Section 9 (Open/Uncategorized) — built for a genuine one-off with nowhere to go, not for a whole expected output category from a class of source material.

**The resolution:** two additions, kept deliberately separate from each other.

1. **`conventions/priming-command-conventions.md`** (new file, rule prefix `PC-0NN`) — a shared contract binding every priming-producing command uniformly, present and future, mirroring the shape `conventions/command-conventions.md` and `conventions/template-conventions.md` already use for their own domains. It states: every such command still merges into the one project-level package (§5.9 unchanged); every command populates the shared spine (Covered/Pending, typed Flags, checkpointed volume, sensitive-content triage, the legacy-material posture of §5.6) the same way; every command maps onto Brief/Domain/BRD Material *first*, always; Section 9 and the new Section 10 are not interchangeable — Section 9 for a one-off, Section 10 for an expected class of fact a given mechanic routinely produces.
2. **Section 10 — Source-Specific Material**, new in `priming-package.template.md` (old Sections 10-11 renumbered to 11-12). A mechanic-specific command that routinely surfaces non-mappable facts contributes its own subsection here, using the heading pattern `### 10.N Source-Specific — [Descriptive Name] (owned by /command-name)`, always naming its owning command inline so responsibility is never ambiguous. `/run-priming-session` owns no subsection today — everything it extracts has a home in Sections 4-6.

**Naming decision:** deliberately *not* called "extension" anything. That word already governs a distinct, unrelated concept in this framework — Core/Extension methodology's architecture-level technical seams (`arch.template.md` §3.2 "Extension Points," reused by `conventions/template-conventions.md` T-023's "extension point table"). Reusing it here for a document-section concept would be exactly the kind of unrecorded synonym drift `CLAUDE.md`'s Ubiquitous Language rule exists to catch — so the framework's own design process is held to the same discipline it enforces on every project it produces. "Source-Specific Section" was chosen instead, reusing vocabulary this template already carries ("Source Material Index," the `source-material/` folder, §5.11's "source-agnostic"/"source mechanic" language) rather than importing a new term.

**Forward note:** when the legacy-code-priming command (§5.11) is actually built, it documents its own Source-Specific Section(s) here — what it adds, and what each one captures — both in this design doc and in `priming-command-conventions.md`'s own registry table (Section 4), so a later third command can check it isn't colliding with a name already claimed.

### 5.16 Conflicting Source facts — added 2026-09-19

**The gap, found by tracing what happens when source material disagrees with itself** (not just with the PO): three concrete holes existed. First, Section 4's own AI Guide already invited "contradiction" as a Flag reason, but Sections 5-6 never repeated it, and Pre-Handoff Verification only required Flags to be typed in Sections 5-7 — Section 4 was silently exempt. Second, there was no recognized Flag type that actually fit "two sources disagree" — `AI Knowledge Correction` and `Legacy Design Question` both trigger on divergence from generic domain understanding, not on two PO-provided or source-provided facts disagreeing with each other. Third, and most consequential: the package's Status field and Flag field are independent — a topic can read `Covered` while carrying an unresolved Flag — and anchor's pre-fill decision (§3, §6) checks for a known answer in a way that in practice reads Status, not Flags. A flagged topic could be silently substituted into a downstream draft without the PO ever seeing the conflict.

**The resolution, kept deliberately general over origin:** whether the disagreement is between two places in the same source file, two different source files, or the priming package and something the PO says live, it is the same underlying situation — a topic already has content, and new candidate content for it disagrees. One mechanism handles all three:

- A new recognized Flag type, **`Conflicting Source`**, added to the closed list and now defined once, at the document level, rather than partially redefined per section.
- Detection becomes a designed comparison step in Triage/Extract (§5.5) — compare new content against what a topic already holds, checking disagreement, not only redundancy.
- When flagged, both (or all) conflicting statements are recorded in full, each attributed to its origin — never silently dropped or preferred.
- If the PO is live in conversation at the moment a conflict is noticed, ask directly right then — cheapest resolution. A flag is for when that isn't possible in the moment (pure file triage, or two files disagreeing before the PO has weighed in on either).
- Resolution reuses the existing decomposition routing, no new gate: a flagged topic in Sections 4-6 resolves when its owning phase command reaches it, exactly like a `Pending` topic already does; a flag in Section 9 routes via that section's own existing "where it probably belongs" column.
- **The closing guarantee:** a topic carrying any unresolved Flag is never treated as a known answer for pre-fill purposes, regardless of its own Status field. This is a narrow, explicit addition to the "check state/reference material for a known answer first" logic §3 already describes — a Flag simply disqualifies a topic from that check succeeding.

Full rule set: `conventions/priming-command-conventions.md` Section 6 (`PC-050`–`PC-054`).

### 5.17 Sections 7, 8, and 9 had no defined consumer — added 2026-09-19

**The gap:** §5.16 above already asserted, in passing, that "a flag in Section 9 routes via that section's own existing 'where it probably belongs' column" — but no mechanism actually existed to make that true. Auditing `anchor-project.md` directly showed it reads exactly two things out of the priming package: Sections 4-6 (pre-filled into a phase command's fields) and Section 10 (handed along as background material, per PC-041). Sections 7 (Terms As Used), 8 (Source Material Index), and 9 (Open/Uncategorized) are all written by `/run-priming-session` but never read by anything downstream — not by anchor's decomposition logic, and not by the package's own Resume Instructions (Section 11), which lists unresolved Pending topics and unresolved Flags but never unresolved Section 9 rows. A fact captured in any of these three sections could be recorded once and then never surface again.

The first instinct — since Section 9 is explicitly modeled on the BRD's Parking Lot (`FW-024`) — was to have anchor auto-write leftover Section 9 rows into the BRD's own Section 15 at `/gen-brd` time. Rejected on inspection: `FW-024` itself states Parking Lot entries are "typically empty at initial generation... not during discovery-driven authoring" — the exact moment that fix would have written to it. The deeper reason it's the wrong shape: a BRD Parking Lot entry is a specific, deliberate idea a human raised (`FW-024`'s own example: a PO's pointed question during epic review); a Section 9 row is raw, un-triaged material from an informal pre-discovery conversation that no one has vetted yet. Auto-writing it into an approved artifact's real section would smuggle unvetted content past the PO, breaking the same review discipline `CLAUDE.md` and §6.3 (fidelity on write-back) both exist to protect.

**The resolution — three sections, two different treatments, both reusing mechanisms this design already has:**

- **Sections 7 and 8 are reference material, like Section 10** — neither carries a substitutable answer, so neither is ever pre-filled into a phase's field. Both are handed along as background material the same way Section 10 already is. Section 8 (a pointer to source files) goes to whichever phase command is about to run, same as Section 10. Section 7 (flagged possible-synonyms) goes specifically to `/run-domain-discovery` and no other phase — the package's own document-level guide already states "which term becomes canonical is domain discovery's decision," so that is the one place a flagged term is actually actionable.
- **Section 9 is unresolved content, not reference material — it is handled like a `Pending` topic.** Each unresolved row surfaces as a live question at the phase named in its own "where it probably belongs" column, exactly the treatment a genuinely `Pending` Sections 4-6 topic already gets: a real question inside that phase's live discovery session, never pre-filled, never handed along silently, and never auto-written into any artifact by anchor itself. If that live session and the PO decide together the item belongs in the BRD's Parking Lot, it goes there through that session's own ordinary judgment — the same way any other Parking Lot entry is added, not through a decomposition shortcut. A row with no clear "probably belongs" guess defaults to surfacing at BRD discovery, the last Problem-Domain phase, so nothing is silently skipped by having an ambiguous destination.
- Section 11 (Resume Instructions) gains a third list — unresolved Section 9 rows — alongside its existing Pending-topics and Flags lists, completing a pattern that was previously incomplete rather than introducing a new one.

Full rule set: `conventions/priming-command-conventions.md` PC-042/PC-043.

### 5.18 "Out of scope" was silently swallowing new facts about locked artifacts — added 2026-09-19

**The gap, found while walking through a real mid-project scenario** (Brief and Domain already Approved/Locked, BRD still Draft, new source material dropped in): `/run-priming-session`'s own Step 3 says an already-Approved/Locked topic is "out of scope for this session — do not re-ask about it." That instruction is about not wasting the PO's time re-confirming settled things. It says nothing about what happens if new material *volunteers* a fact touching one of those topics on its own — a genuinely new fact, not a re-ask. Read literally, "out of scope" reads as "skip it," which means such a fact could be silently dropped: never written anywhere, never reaching the constraint-propagation drift check (§7.1 item 2 / §5.10) that exists specifically to catch a fact recorded after an artifact was already approved — because that check only fires on facts that got recorded somewhere first.

**The resolution:** "out of scope" is now explicit that it governs *asking*, not *noticing*. A fact touching an already-Approved/Locked topic still gets recorded — into Section 9 (Open/Uncategorized), same section as any other unresolved item, but with "Where it probably belongs" naming the affected artifact and its status (e.g. "Domain — already Locked, needs `/assess-change`") rather than a phase to raise a live question in. Anchor's own Section 9 handling (§5.17) is extended to match: a row phrased this way is never treated as a live-discovery-question case — there's no open session left to raise it in — it surfaces directly to the PO and routes to `/assess-change`, the same way any other non-linear request already does at Step 9.

Full rule set: `conventions/priming-command-conventions.md` PC-015.

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
2. **Constraint-propagation drift** — any fact/correction/constraint recorded after a given artifact was approved, checked against that artifact's scope (§5.10's trigger).
3. **Orphaned open items** — Open Issues / AI Knowledge Corrections rows still marked Open from a stage the project has since moved past, never resolved or explicitly deferred.

### 7.2 Heavier checks — content-level, run at gate-close or on explicit request

4. **Traceability coverage** — every REQ resolves to an epic (cross-check the REQ Coverage Map that already exists in `epics/index.md`); every epic's referenced screens exist in Screen Design; every "carried over from screen-design.md" claim in Architecture still matches the current screen-design.md.
5. **Ubiquitous Language consistency** — terms used downstream (stories, architecture) checked against the domain glossary for unrecorded synonym drift. This is CLAUDE.md's existing rule; nothing today checks it proactively across a whole project.
6. **Gate legitimacy** *(lower priority)* — an artifact marked Approved/Locked whose own template verification checklist wouldn't actually pass if re-run now; catches a status advanced via `/update-status` without the real review gate having run.
7. **Judgment-check coverage** — for artifact types `/judgment-check` already covers (BRD, Epics, Architecture today), anchor runs it automatically before presenting the matching `/review-X` gate to the PO, and folds its findings into the same surfaced-only-what's-flagged pattern as §6.5 — this is the direct precedent that motivated this whole command, and it should be a standing part of anchor's gate sequence, not an optional extra.

### 7.3 Non-linear requests — mid-project changes

Everything above assumes forward sequencing (stage N → N+1). A PO wanting to revise something already approved or locked is a normal, expected event, not an edge case anchor can ignore. Anchor recognizes when a PO's request isn't "move to the next stage" but "change something already settled," and routes it to `/assess-change` rather than plowing forward as if the project were still purely linear. The resulting version bump feeds the lineage ledger and the drift triggers in §7.1 exactly like any other change.

### 7.4 Extension projects — lineage crosses project boundaries

The lineage ledger as described in §4.1 tracks upstream→downstream pairs within one project. An extension project's domain knowledge and/or BRD depends on a **base** document that lives in a different project's folder (per the framework's Core/Extension model). Anchor's version-lineage check (§7.1 item 1) needs to know to look there too — an extension project can drift not only against its own prior artifacts but against a base document it doesn't own.

**Resolution: a cross-folder read, not a new mechanism.** The base-project relationship already exists as data — the brief's Extension Context (captured during `/run-intake`, per `CLAUDE.md`) already records which base project an extension depends on. So this isn't new tracking, it's reading what already exists: the lineage ledger gains one additional row type for an extension project — same shape as any other row (artifact, version, consumed-by, when), except the artifact path points into the **base** project's folder instead of the current one, sourced from the brief's Extension Context. The version-lineage check (§7.1 item 1) simply includes these cross-project rows in its normal comparison — no special-casing beyond "this path happens to live in another `projects/{CODE}/` folder." It doesn't require the base project to be anchor-managed either — the same §4.3 bootstrap logic reads the base artifact's own Status/Document History regardless of whether that project itself has ever been anchor-driven.

## 8. Project Journal and Framework Learning — redesigned 2026-09-19

**Implemented 2026-09-19**, converted into `decisions/FW-050-project-journal-and-framework-learning.md`. This section replaces the original `observations/`-only design below (kept in §8.5 for the record) with a two-layer model: one continuously-authored per-project artifact, and periodic analysis passes derived from it.

### 8.1 The problem with the original design

The original mechanism (§8.5) had one layer only: `observations/`, written ad hoc, in the moment, at a deliberately high bar ("a real, evidenced finding, not every passing stylistic thought"). Two things this couldn't do: capture the PO's own real-time reactions and judgment calls as they actually happened (only Claude's own after-the-fact interpretation of friction), and capture a project's *strengths* and ordinary texture, not just candidate framework bugs — the raw material for real analysis was never actually being kept, only a thin, pre-filtered slice of it decided in the moment.

### 8.2 The Project Journal — one continuously-authored artifact per project

**File:** `projects/{PROJECT_CODE}/journal.md`. Created empty by `/init-project` as part of its normal skeleton (the same additive pattern `source-material/` used for `FW-049`).

**Written by anchor only** — this is anchor's own distinct contribution (same category as drift-checking, §7's intro), not something standalone commands are modified to write. A PO who never touches anchor gets no journal, same as today's `observations/` behavior.

**Low bar, not high.** Unlike the old `observations/` bar, the journal takes anything real — a PO's offhand comment, a phase that went unusually smoothly, a workaround anchor had to apply. The filtering happens later, at extraction (§8.3), with the benefit of hindsight and pattern-matching across multiple entries — not in the moment, where a single incident's significance often isn't clear yet.

**Shape:** a lightweight index at the top (Project Code, entry count, "extracted through" pointer — the last entry any extraction pass has already processed, so re-runs don't reprocess old ground), followed by chronological entries, oldest first. One file, growing for the life of the project — no chunking, per your call above.

**Entry format:**
```
### {date} — {Phase/Stage} — {Category}
{Body — a verbatim PO quote, a short description of the friction/strength/finding, whatever fits}
```

**Category — a closed vocabulary, same discipline as the Flag types elsewhere in this framework, not free text:**
- `PO Feedback` — a reaction volunteered by the PO, quoted verbatim wherever possible, not paraphrased. Positive or negative both count.
- `Strength` — something that worked cleanly and is worth reinforcing, not just fixing what's broken.
- `Friction` — something that didn't work smoothly, whether in anchor's own orchestration or whichever command it was wearing the hat of.
- `Drift Resolution` — a drift flag (§7) surfaced and how it actually got resolved.
- `Judgment-Check Finding` — something `/judgment-check` caught.
- `Documentation Gap` — a practitioner-guide or command-instruction gap noticed in passing.
- `Limitation` — a known capability boundary actually hit ("command X doesn't handle Y" surfacing for real, not speculatively).
- `Workaround` — anchor or the PO had to route around something to keep moving.

**When entries get written** — at the natural points anchor already passes through, not a separate bookkeeping pass: Step 4 (a drift flag found or resolved), Step 6 (the PO reacts to something, praise or complaint, prompted or not), Step 9 (a judgment-check pass returns a finding; friction or a documentation gap surfaces while wearing an invoked command's hat), anywhere §11's old trigger already fired. This is the same "write in the moment, during execution" timing the original design already got right (§8.5) — only the *bar* for writing changes, not the timing discipline.

### 8.3 Extraction — where real analysis happens, on a deliberate schedule

**Triggered automatically at milestones anchor already recognizes** — a gate closing, the Walking Skeleton handoff (§9), project close (`/close-sprint`/`/gen-release-notes`) — never continuously, and never per-entry. This is the direct answer to "don't patch reactively": the gap between writing an entry and analyzing it is the whole point, not an inefficiency to remove.

**What it reads:** every journal entry since the index's "extracted through" pointer — never a full re-read of the whole file.

**What it produces, at two different scales:**
- **Observations** (every extraction pass) — the same `observations/` mechanism and promotion discipline as §8.5 already established (`New` → `Under Review` → `Promoted → FW-XXX` / `Rejected` / `Won't Fix`, `observations/index.md`, `observations/TEMPLATE.md`, `observations/OBS-NNN-{slug}.md`) is retained unchanged in shape — only its *input* and *timing* change. A candidate observation now has to show a real pattern across the accumulated entries (the same friction recurring, a limitation hit more than once) rather than being decided from a single in-the-moment incident, which is a stronger evidentiary bar than the original design could actually apply in practice.
- **A project testimonial** (only at the larger milestones — Walking Skeleton handoff, project close) — a polished narrative synthesis of the journal so far: what the project was, what worked, what friction showed up, in prose. This is a distinct artifact from an observation — it's for a human reader (the PO, or whoever the PO chooses to share it with), not a framework-decision candidate.

### 8.4 Sharing — the reporting gap, revisited

The original placeholder problem (§8.5.2 in the old numbering) still applies and still isn't solved here: `projects/{CODE}/` is gitignored, private to each PO's clone, so nothing here reaches the framework author or the community automatically. What changes is *what's safe to recommend sharing*: never the raw journal by default — it will carry real client-specific content (verbatim PO quotes, real project facts) that has no business leaving a PO's own clone. The **observations extraction** is the thing actually worth sharing — by construction it's already been generalized away from any specific client into a framework-level statement ("gap in `/gen-brd`'s Section 5.21 guidance"), which is both safer and more directly useful to the framework author than raw journal content would be. A testimonial is shareable at the PO's own discretion, same as any case study. The actual channel (email, a GitHub issue template, something else) is still the framework author's decision to make, same placeholder as before.

### 8.5 Original design (superseded, kept for the record)

Kept fully separate from `decisions/` — a raw waiting room, not a decision log:

- `observations/index.md` — registry: ID, one-line title, Status, Date, Area
- `observations/TEMPLATE.md` — its own lighter template (this is a raw finding, not a ratified decision): what was observed, where/when, why it matters, a proposed direction
- `observations/OBS-001-{slug}.md` per finding
- Status field distinct from decisions': `New` → `Under Review` → `Promoted → FW-XXX` / `Rejected` / `Won't Fix`

**Trigger timing:** the write happens in the moment, during execution — whether anchor is running its own orchestration logic or wearing an invoked command's hat mid-`/gen-brd`, mid-`/review-epics`, or anywhere else — whenever it hits something that doesn't work as documented, is genuinely ambiguous, or represents a real, evidenced improvement to the framework itself (not the project). Not a scheduled batch pass, not deferred to session-end — the same way `FRAMEWORK-AUDIT.md`'s defects were actually found, as real friction hit while actually using the thing. It writes silently, without stopping to ask the PO's permission each time (this is about the framework, not their project — asking would undercut the point of the whole design), but it still surfaces briefly in the session output ("noted OBS-004 — a gap in X") so it's never invisible, just not gating. The bar should be genuinely high, the same restraint as the question-budget in §6.2 — a real, evidenced finding, not every passing stylistic thought, or `observations/` turns into noise nobody reviews.

### 8.5.1 Scope: command-level friction, not just anchor's own orchestration (resolved 2026-09-17)

`observations/` is not scoped narrowly to things anchor itself gets wrong. It also captures friction in whatever underlying command anchor happens to be wearing the hat of at the time (§3) — an ambiguous instruction in `gen-brd.md`, a check in `review-stories.md` that doesn't catch something it should, a template section that doesn't match what its own command actually does. This is, if anything, the more valuable half: a gap in a shared command affects every project that ever runs it, anchored or not, while a gap in anchor's own orchestration only affects anchor sessions. Anchor is simply well-positioned to notice both, since it's the one thing that runs the full pipeline repeatedly and end to end.

### 8.5.2 Reaching the framework author — the reporting gap (resolved 2026-09-17)

`observations/` living in the project's own working tree only solves half the problem. The framework's real usage model is **one PO running their own clone of this repo** (`README.md`'s own setup instructions: `git clone` the framework, work inside it) — not one shared instance the framework author has direct visibility into. An observation anchor writes during a PO's session sits in that PO's own local `observations/` folder; it never reaches the framework author unless the PO deliberately does something to send it there. The promotion path described above (framework author reviews, decides, writes a real `FW-XXX`) silently assumed access that doesn't actually exist for anyone except the author's own clone.

**The fix, for now:** whenever anchor writes a new observation, it also surfaces a one-line prompt encouraging the PO to share it with the framework author — a placeholder contact channel for the moment (e.g. "Found something worth improving in the framework itself? Share it: `[contact channel — TBD]`"), not yet wired to anything real. The actual channel (an email address, a GitHub issue template, something else) is a decision for the framework author to make later, not something to invent speculatively here. Until then, `observations/` still does its local job — a PO benefits from their own session's findings being tracked and visible even before any of it reaches anyone else — the reporting prompt is additive, not a blocker on the rest of the mechanism working.

Promotion path (unchanged): the framework author reviews an observation — their own, or one a PO reported to them through whatever channel eventually replaces the placeholder above — and, if worth acting on, writes it up as a real `FW-XXX` in `decisions/` using the existing decision template. Observations never write into `decisions/` directly, and are never auto-applied.

## 9. Solution Domain Boundary — The Walking Skeleton Handoff

This resolves what was previously the biggest open item in §11 (Phase 6–9 scope), using a real mechanism, not a placeholder.

**The problem it resolves:** anchor's entire design assumes single-session, single-actor state — one lightweight pointer file, one lineage ledger, reconciled by whoever's driving (§4.3). That assumption holds all the way through Problem Domain and Architecture, and holds through a first thin end-to-end slice, because that's still one thread of work. It stops holding the moment multiple developers are implementing and verifying separate stories in parallel — now there are concurrent writers to state that was designed for one.

**The boundary:** anchor drives architecture through to Locked, generates the sprint plan, then leads execution of exactly the stories tagged `Walking Skeleton: Yes` (`FW-048`) through `implement-story` → `verify-story` → PR — the smallest real, thin, end-to-end slice that proves the architecture's layers actually connect. The moment `stories/index.md` Section 8's `Walking Skeleton Complete` field flips from `No` to `Yes` — every Walking Skeleton story reaches `Done`, QA-verified, written by `/update-status` per FW-048 — anchor's active involvement ends there. Execution hands off to normal developer-led sprints using the standard commands directly, with `/project-status` as the ongoing dashboard instead of anchor.

This is a mechanical, checkable trigger, not a judgment call anchor has to make each time — `FW-048` was written specifically to make this boundary real rather than something anchor would otherwise have to eyeball. Day-to-day story implementation after handoff stays fully developer-led — that's the concurrent-writer zone anchor's state model was never built for, and it stays out of it.

### 9.1 Post-Handoff Role (resolved 2026-09-17)

Anchor's involvement after handoff isn't all-or-nothing — it narrows to exactly two triggers, both of which are single-actor decision points, not the parallel-development zone §9 excludes it from:

- **Release preparation and closure** (`/close-sprint`, `/gen-release-notes`, and the judgment around them) — this is naturally single-actor work (one person compiles a release regardless of how many developers built the stories in it), and it's squarely the kind of cross-cutting, whole-project judgment anchor exists to apply. Anchor comes back for this.
- **A Formal-tier `/assess-change`** — specifically Pattern 3 (Scope Shift) or Pattern 4 (Requirement Change), the two patterns `assess-change.md` Step 5 already gates behind its own formal change confirmation. A big scope or architecture change is exactly the anti-drift scenario §7 exists for; an informal Pattern 1/2 fix stays fully developer-led and never needs anchor.

This isn't a hard technical gate — `/assess-change` and `/gen-release-notes` both remain fully standalone-runnable per §2, and nothing forces a developer through anchor for either. It's a recommendation baked into how anchor itself behaves: when anchor resumes a project (§4.3's reconciliation) and finds a Formal-tier change already happened outside it, that's exactly the situation its drift-checking (§7) is built to catch and reconcile — so the practical effect is the same either way, whether the PO consciously brings anchor back in beforehand or anchor picks the change up on its next resume.

Everything else post-handoff — implementing individual stories, day-to-day verification, routine PRs — stays with whichever developer owns that story, using the standard commands directly, no anchor involvement expected or needed.

---

## 10. Rejected Approaches (kept for the record)

- **Editing ~20 existing commands to adopt a shared "lighter interaction" convention file.** Rejected because it violates the frozen-commands constraint (§2) and creates a second place PO-interaction logic could drift from itself over time (the version baked into the shared convention vs. each command's own copy). Superseded by §3 (interception at the invocation seam, zero file edits).
- **A single fused command that reimplements discovery/review interaction itself instead of invoking the real commands.** Rejected for the same reason — would duplicate logic that already exists correctly in each phase's command, and a PO running that command standalone would get a second, divergent experience instead of today's.
- **Document ingestion embedded as a mode inside `/anchor-project` itself.** The original §5 design — caught during a later composability pass, not at authoring time. Anchor is a sequencer; ingestion is a source-agnostic document-processing mechanism; welding the second into the first meant anchor's own file wasn't source-agnostic even though the mechanism it contained was, and every future input mechanic (starting with legacy code) would have meant editing `anchor-project.md` again instead of adding a new command. Superseded by §5's current form — `/run-priming-session`, a standalone companion command.

## 11. Invocation

`/anchor-project` takes no required arguments. Bare invocation:
- Looks at `projects/index.md`. Exactly one active project → resume it. Several → ask (one short question) which. None → route to `/init-project`.
- Naming a project (e.g. `/anchor-project THORNFIELD-MAINT-001`) still works directly — useful once several projects are running in parallel, which is the framework's stated target usage pattern.
- For project identity specifically (the four-part code, name, client, domain a PO often struggles to form) — anchor proposes the identity fields itself as a draft and asks for confirmation, rather than asking the PO to originate them.

A PO who wants the standalone, unanchored experience simply runs any command directly, as today — anchor is optional at every level, not a required front door.

## 12. Open Items — Resolved 2026-09-17

Everything previously listed here is now settled. Kept as a record of what was decided and why, rather than deleted outright — a design doc this heavily revised benefits from showing its work.

- **Post-handoff role** → resolved as §9.1: release prep and Formal-tier `/assess-change` only; everything else stays developer-led.
- **Extension/base-document lineage** → resolved as §7.4: a cross-folder read off the brief's Extension Context, one extra lineage-ledger row type — no new mechanism.
- **Validation plan** → both candidate approaches accepted, not mutually exclusive: adopt THORNFIELD-MAINT-001 mid-flight (exercises §4.3's bootstrap path, though it predates `FW-048` so has no Walking Skeleton tags — treat it as a pre-FW-048 case rather than backfilling tags onto a project that was never discovered with them in mind) and/or a fresh second simulation end to end.
- **Drift-check automation depth** → confirmed: both tiers, as already written in §7.1/§7.2.
- **Escape-hatch persistence across sessions** → confirmed: resets by default each session, as already written in §6.4.
- **`observations/` scope** → resolved as §8.1/§8.2: command-level friction is in scope, and arguably the more important half, alongside anchor's own findings; a placeholder reporting-channel prompt added to close the gap between a PO's local clone and the framework author actually seeing it.
- **Exact state file / lineage ledger file name and location** — still genuinely deferred to implementation time (candidate: `projects/{CODE}/.anchor-state.md` or similar); not load-bearing for the design itself, not worth deciding speculatively.

**Command file structure — done.** `.claude/commands/anchor-project.md` is written, self-checked against every rule in `conventions/command-conventions.md`'s verification checklist, and two exceptions it genuinely needed were proposed centrally rather than improvised locally, exactly per that document's own governing principle — both added under their relevant rules: `C-011` (gate-check placement for a cross-cutting orchestration command, treated as session-command-like) and `C-013` (distributed rather than single-step file reading, since anchor's inputs are inherently sequence-dependent). Two supporting scaffold files were also created since the command depends on them existing: `observations/TEMPLATE.md` and `observations/index.md` (§8).

This design document's own status can now move from "in discussion" — the mechanism it describes is real and implemented. Validation (the remaining resolved item above) is the next actual step, not further design.

---

## Document History

| Date | Change |
|------|--------|
| 2026-09-16 | Initial consolidation of design discussion (mechanics, interaction protocol, ingestion, anti-drift axes, self-improvement logging, invocation semantics) |
| 2026-09-16 | Full review pass: added review-output compression (§6.5), state reconciliation/bootstrap (§4.3), no-gate-fast-forwarding constraint (§2), non-linear request routing (§7.3), extension/base lineage gap (§7.4), judgment-check integration (§7.2 item 7), sensitive-content and volume handling in ingestion (§5.6, §5.7); flagged Phase 6-9 scope as the one significant unresolved fork (§11) |
| 2026-09-17 | Resolved the Phase 6-9 scope fork using the Walking Skeleton concept (real precedent found in a sibling project, `ascendra-pay-002`, not previously part of this framework): added §9 (Solution Domain Boundary — The Walking Skeleton Handoff), sections renumbered accordingly (Rejected Approaches → §10, Invocation → §11, Open Items → §12). Formalized the underlying concept into the framework itself first, ahead of anchor, as `decisions/FW-048-walking-skeleton-milestone.md` — implemented across the BRD/Epics/Stories/Sprint Planning pipeline and the practitioner guide — so §9's handoff trigger (`stories/index.md`'s `Walking Skeleton Complete` field) is a real mechanism, not a placeholder |
| 2026-09-17 | Two clarifications filled in that were previously implicit: §5.2 (new) spells out the three-pass ingestion pipeline (map, triage, extract) agreed in discussion but never actually written down, including how the Map step adapts to structured vs. unstructured sources — §5.3-5.8 renumbered accordingly; §8 gained a Trigger Timing paragraph specifying that observations are written in the moment during execution, silently but with a brief session-output mention, at a high evidentiary bar |
| 2026-09-17 | Resolved every remaining §12 open item: added §9.1 (Post-Handoff Role — release prep and Formal-tier assess-change only), rewrote §7.4 with the actual cross-folder-lineage mechanism (had been answered in discussion but never written in), added §8.1 (command-level friction is in scope) and §8.2 (the reporting-gap problem and a placeholder contact channel), confirmed validation plan/drift-check depth/escape-hatch persistence as already-written defaults, and recorded the FW-020 exception-handling discipline that will govern `anchor-project.md`'s eventual authoring. §12 kept as a resolution record rather than deleted |
| 2026-09-17 | `.claude/commands/anchor-project.md` written in full and self-checked against every rule in `conventions/command-conventions.md`'s verification checklist. One real structural bug caught and fixed during self-check: an early draft's Step 9 (post-handoff scope) told the reader to consult it "before Step 8," breaking sequential step order — fixed by swapping step content so post-handoff scope now correctly runs before driving the phase. Two genuinely-needed conventions exceptions proposed centrally in `command-conventions.md` itself (under `C-011` and `C-013`) rather than improvised locally, per that document's own governing principle. `observations/TEMPLATE.md` and `observations/index.md` created as supporting scaffolding the command depends on. Status updated: implemented, pending validation and formal `decisions/FW-XXX` writeup |
| 2026-09-17 | §5 rewritten in full: document ingestion is no longer a mode of `/anchor-project` — applying the composability principle to the finished design surfaced a real contradiction (a source-agnostic mechanism embedded in the one command that isn't source-agnostic), caught later than it should have been, now recorded as a third entry in §10 (Rejected Approaches). Ingestion becomes `/run-priming-session`, a standalone companion command scoped to Problem Domain only (Brief/Domain/BRD — Epics/Screen Design/Architecture are AI synthesis, nothing to front-load), producing `priming-package.template.md` (composed from existing templates' own sections, not a new schema). New content added, not previously specified: reproducibility via structural completeness rather than identical output (§5.3, reusing the same mechanism `/run-domain-discovery` already relies on), explicit handling for the absence of Ubiquitous Language at this stage (§5.4 — capture terms as used, flag apparent synonyms, never normalize), the staging/decomposition model (§5.9 — one consolidated package the PO reviews, split into each phase's real file only when anchor reaches it), the one-command-per-mechanic principle with legacy code named as the deferred second command (§5.11), and `/init-project`'s own small required addition — a `source-material/` folder (§5.12). `anchor-project.md` itself still needs a matching Step 7 rewrite (now a simple handoff) — not yet done, tracked as a next step, not re-added to §12 |
| 2026-09-19 | Added §5.15 (Source-Specific Sections): a new `conventions/priming-command-conventions.md` (`PC-0NN` rules) governs every priming-producing command uniformly, and `priming-package.template.md` gained a new Section 10 (Source-Specific Material, "None" by default) for facts a mechanic-specific command routinely produces that never map onto the Brief/Domain/BRD-mirrored Sections 4-6 — distinct from Section 9 (Open/Uncategorized), which stays reserved for genuine one-off leftovers. Deliberately not named "extension" anything, to avoid colliding with the unrelated Core/Extension "Extension Points" vocabulary already used in `arch.template.md` §3.2. Old Sections 10-11 renumbered to 11-12; §5.11 and §5.13 updated to match; §5.13's skeleton — previously stale, showing 6 sections against the template's real (then-)11 — corrected to the current 12-section shape in the same pass. `decisions/FW-049-anchor-project-and-priming-session.md` amended to match |
| 2026-09-19 | Added §5.16 (Conflicting Source facts): closed three gaps found by tracing source-internal contradiction handling — Section 4's Flag guidance mentioned "contradiction" but Sections 5-6 and Pre-Handoff Verification's typing rule (scoped to Sections 5-7 only) didn't; no recognized Flag type actually fit "two sources disagree"; and the package's Status/Flag field independence meant a flagged topic could be silently pre-filled by anchor despite an unresolved conflict. Resolved with one origin-agnostic mechanism: a new `Conflicting Source` flag type (defined once at the document level, not per-section), a designed disagreement-check in Triage/Extract (§5.5), and an explicit guarantee that any unresolved Flag disqualifies a topic from anchor's pre-fill regardless of Status. `conventions/priming-command-conventions.md` Section 6 (`PC-050`-`PC-054`), `priming-package.template.md`/`.example.md`, `run-priming-session.md`, and `anchor-project.md` all updated to match; `decisions/FW-049-anchor-project-and-priming-session.md` amended |
| 2026-09-19 | Follow-up review pass against `anchor-project.md`'s own decomposition logic (not tied to a new priming-package feature) surfaced three further items, all fixed same day: (1) Step 3's Reference Material Index cited "Section 3" as the cheap source for a per-topic Pending count, but Section 3 (Coverage Snapshot) only holds a coarse per-section rollup with no such count — corrected to describe scanning Sections 4-6's own Status lines directly. (2) Step 7's pre-fill logic described only a binary Covered/Pending split, silently omitting the `Partial` status PC-011 already makes canonical — Step 6's own "if unknown or partial, apply 6.1–6.4" already implied the right behavior, so Step 7 was brought in line with it: a `Partial` topic is surfaced as a live but *informed* question (state what's known, ask what's missing), never silently substituted as complete. (3) Added §5.17 (Sections 7, 8, and 9 had no defined consumer) — the larger of the three, covering `anchor-project.md`'s failure to read Sections 7 (Terms As Used), 8 (Source Material Index), or 9 (Open/Uncategorized) at all; resolved by treating 7/8 as background reference material like Section 10 (PC-042) and Section 9 as unresolved content surfaced live at its probable phase, never auto-written into any artifact including the BRD Parking Lot (PC-043). `conventions/priming-command-conventions.md` (PC-042/PC-043 added), `priming-package.template.md`/`.example.md` (Section 11 gained a third unresolved-items list), `anchor-project.md`, and `decisions/FW-049-anchor-project-and-priming-session.md` all updated to match |
| 2026-09-19 | Walking through a real mid-project scenario (Brief/Domain already Locked, BRD still Draft, new source material arriving) surfaced a fourth item: added §5.18 — `/run-priming-session`'s "already Approved/Locked = out of scope" instruction only meant "don't ask," but read literally could be mistaken for "ignore," letting a genuinely new fact about a locked topic get silently dropped before it ever reached the constraint-propagation drift check that exists to catch exactly this. Resolved with `PC-015`: such a fact is recorded in Section 9 with "Where it probably belongs" naming the affected artifact and its status (e.g. "Domain — already Locked, needs `/assess-change`") rather than a phase to question. `anchor-project.md`'s §5.17 Section 9 handling extended to recognize this phrasing and route straight to `/assess-change` instead of treating it as a live-discovery-question case. `run-priming-session.md` (Step 3, Step 5a, Step 5b), `priming-package.template.md`, `conventions/priming-command-conventions.md` (PC-015 added), `anchor-project.md`, and `decisions/FW-049-anchor-project-and-priming-session.md` all updated to match |
| 2026-09-19 | §8 (Self-Improvement Logging) rewritten in full as §8.1–§8.4, following a design discussion prompted by wanting the framework's own learning loop to be based on real, accumulated evidence rather than in-the-moment, ad hoc calls. The original `observations/`-only design had one layer at one bar; this splits it into a continuously-authored, low-bar per-project Project Journal (`journal.template.md`, new — eight closed entry categories) and a periodic, milestone-triggered extraction pass that promotes real cross-entry patterns into `observations/` (unchanged in shape, now better-evidenced) and, at the two largest milestones only, a project testimonial. A tempting simplification — collapsing `observations/` into the journal entirely — was considered and rejected: the two artifacts sit on opposite sides of the private/shareable line (journal: per-project, gitignored, real client content; observations: cross-project, git-tracked, already-generalized), and collapsing them would sacrifice one of those properties. Original design kept as §8.5/§8.5.1/§8.5.2 for the record (renumbered from a heading collision this rewrite briefly introduced and then fixed). New exception `T-042` added to `conventions/template-conventions.md` for this template's append-only shape. `.claude/commands/anchor-project.md` (Step 3, Step 11, Step 12), `.claude/commands/init-project.md`, `projects/TEMPLATE/journal.template.md` + `.example.md` (new), `observations/index.md` + `TEMPLATE.md`, and `CLAUDE.md` (which also gained a pre-existing missing `priming/` entry, caught in passing) all updated to match. Converted into `decisions/FW-050-project-journal-and-framework-learning.md` |
