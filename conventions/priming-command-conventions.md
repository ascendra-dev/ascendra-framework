# Priming Command Conventions

Verifiable rules for any command that writes into `projects/{PROJECT_CODE}/priming/priming-package.md` — today `/run-priming-session`, and any future command handling a different source mechanic (legacy code, an OpenAPI spec, a live DB schema — see `ANCHOR-PROJECT-DESIGN.md` §5.11). Apply these when writing a new priming-producing command and when modifying an existing one. Each rule is independently checkable with a yes/no.

**Governing principle:** These conventions exist for consistency only. `conventions/command-conventions.md` and `conventions/template-conventions.md` still apply in full to every priming command and to `priming-package.template.md` itself — this file is a narrower, additional layer specific to how multiple mechanic-specific commands share one output artifact without colliding or drifting apart. Where following a convention here would compromise a priming command's own extraction quality, add the exception to this document under the relevant rule — then apply it consistently across every priming command that needs it. Do not document exceptions inside individual command files.

---

## Section 1 — Scope

**PC-001** A "priming-producing command" is any command whose output is all or part of `projects/{PROJECT_CODE}/priming/priming-package.md`. This file governs every such command uniformly, present and future — a command does not get to redefine these rules for itself.

**PC-002** Every priming-producing command merges into the **same one** project-level `priming-package.md` — never a second file, never a per-mechanic package. This preserves the one-document, decomposed-later model (`ANCHOR-PROJECT-DESIGN.md` §5.9): the PO reviews one coherent artifact, and decomposition into `brief.md` / `domain-discovery-state.md` / `brd-discovery-state.md` still happens later, at the moment `/anchor-project` actually reaches each phase.

**PC-003** A priming-producing command is scoped to Problem Domain only (Brief, Domain, BRD) — the same boundary `/run-priming-session` already observes. A source mechanic that could inform Architecture or later has nothing to contribute here; see `ANCHOR-PROJECT-DESIGN.md` §5's own note that Epics/Screen Design/Architecture are AI synthesis from an approved BRD, not raw PO knowledge.

---

## Section 2 — Shared Spine Contract

**PC-010** Every priming-producing command populates the package's shared spine (Session Context, Session History, Coverage Snapshot, Terms As Used, Source Material Index, Open/Uncategorized, Resume Instructions, Pre-Handoff Verification) using the mechanics `priming-package.template.md` already defines — it does not invent a parallel tracking scheme for its own contributions.

**PC-011** Every priming-producing command uses the same status/flag vocabulary already established: `Covered` / `Partial` / `Pending` per topic, and typed Flags (`AI Knowledge Correction`, `Legacy Design Question`, `Scope Risk`, `Conflicting Source`, or a synonym/terminology note) — never an untyped flag, never a command-specific status label.

**PC-012** Volume handling is checkpointed, not held in full context across turns, for any source large enough to need it (`ANCHOR-PROJECT-DESIGN.md` §5.8) — this applies regardless of source kind (a large document, a large codebase, a large schema dump).

**PC-013** Sensitive-content triage applies regardless of source kind: a command recognizes obviously credential-shaped strings (keys, tokens, passwords, connection strings) in whatever it reads and flags them to the PO rather than filing them into the package silently (`ANCHOR-PROJECT-DESIGN.md` §5.7).

**PC-014** The legacy-material posture — matches generic domain understanding → ordinary content; diverges for a stated business reason → `AI Knowledge Correction`; diverges and looks like a design flaw rather than a real constraint → `Legacy Design Question`, framed as a negotiation, never resolved unilaterally — applies to every priming-producing command reading anything that describes or embodies an existing system, not only to `/run-priming-session` reading legacy prose (`ANCHOR-PROJECT-DESIGN.md` §5.6).

---

## Section 3 — Mapping Priority

**PC-020** A priming-producing command maps every extracted fact onto Brief/Domain/BRD Material (Sections 4-6) first, always — regardless of source mechanic. A business rule discovered by reading validation logic in code belongs in Domain Material's "Universal Business Rules" topic, in domain vocabulary, exactly like a business rule stated in conversation. The source mechanic changes how a fact is found, never where a business-facing fact is recorded.

**PC-021** Section 9 (Open/Uncategorized) and Section 10 (Source-Specific Material) serve different purposes and are not interchangeable:
- **Section 9** is for a genuine one-off — a fact this specific session surfaced that has nowhere to go, regardless of source mechanic.
- **Section 10** is for a *class* of fact a given mechanic routinely produces and that never maps onto Brief/Domain/BRD vocabulary (e.g. a code-reading command's existing API surface, schema-as-implemented, technical-debt inventory).

A command should very rarely need both for the same kind of content in the same session — if something keeps landing in Section 9 across multiple sessions, that is a signal it should have its own Source-Specific Section instead (see Section 4 below), not a reason to keep routing it to Section 9.

---

## Section 4 — The Source-Specific Section Mechanism

**PC-030** A priming-producing command that routinely surfaces facts with no home in Sections 4-6 contributes its own subsection under `## 10. Source-Specific Material`, using the heading pattern:

```
### 10.N Source-Specific — [Descriptive Name] (owned by /command-name)
```

**PC-031** Every Source-Specific subsection names its owning command inline in its own heading (per PC-030's pattern) — a reader must never have to infer which command is responsible for a given subsection's content or upkeep.

**PC-032** A priming-producing command that owns a Source-Specific subsection documents, at the point that command is authored, what section name(s) it adds and what each one captures — this is what lets a second, later command check it isn't colliding with a name a prior command already claimed. Maintain the registry below as new commands are added:

| Owning Command | Section Name | What It Captures |
|-----------------|--------------|-------------------|
| *(none yet — `/run-priming-session` maps everything onto Sections 4-6 and owns no Source-Specific subsection; see PC-033)* | | |

**PC-033** A priming-producing command is not required to own a Source-Specific subsection. `/run-priming-session` currently owns none — prose, an SRS, raw notes, and live conversation always have a home somewhere in Brief/Domain/BRD Material; a genuine one-off from that command still goes to Section 9, never Section 10.

---

## Section 5 — Verification and Handoff

**PC-040** Pre-Handoff Verification (the package's own Section 12) checks that Section 10, if it contains any subsection, has every subsection properly attributed per PC-031 — an unattributed or unlabeled Source-Specific entry is a failed check, not a silent pass.

**PC-041** When `/anchor-project` decomposes the priming package into a phase command's expected input, Section 10 is handled differently from Sections 4-6: its content is **not** pre-filled into a specific template field the way Brief/Domain/BRD Material is, because it has no fixed home in any one phase's schema. It is instead handed to whichever phase command is about to run as background reference material, for that command's own judgment to use or set aside.

**PC-042** Sections 7 (Terms As Used) and 8 (Source Material Index) are handed along the same way as Section 10 — background reference material, never pre-filled into a phase's field, since neither carries a substitutable answer. Section 8 goes to whichever phase command is about to run, same as Section 10. Section 7 goes specifically to `/run-domain-discovery` and no other phase, since the package's own document-level guide already establishes that "which term becomes canonical is domain discovery's decision" — that is the one phase a flagged possible-synonym is actually actionable in.

**PC-043** Section 9 (Open/Uncategorized) is handled differently from Sections 7/8/10 — it is not reference material, it is unresolved content that still needs a real answer. Each unresolved row surfaces as a live question at the phase named in its own "where it probably belongs" column, treated exactly like a genuinely `Pending` Sections 4-6 topic — never pre-filled, never handed along silently as background material, and never written directly into any downstream artifact (including the BRD's own Parking Lot, `FW-024` Section 15) by `/anchor-project` itself. If that live discovery session and the PO decide together the item belongs in the Parking Lot, the session's own normal judgment puts it there — the same way any other Parking Lot entry gets added, never as an automatic decomposition step. A row with no clear "probably belongs" guess defaults to surfacing at BRD discovery, the last Problem-Domain phase, so nothing goes unraised by default.

---

## Section 6 — Conflict Detection and Resolution

**PC-050** The recognized flag types include `Conflicting Source` — two or more captured statements about the same topic disagreeing with each other. The typing requirement (PC-011) applies uniformly across Sections 4-7 of the priming package, not only Sections 5-7.

**PC-051** Detecting a conflict is a designed comparison step, not incidental noticing: every priming-producing command compares new content for a topic against whatever that topic already holds, checking for disagreement in addition to the redundancy check it already performs — regardless of whether the compared content came from the same source, a different source, or a live PO statement.

**PC-052** A `Conflicting Source` flag always records every conflicting statement in full, each attributed to its origin (file + rough location, or "PO stated live") — never silently drops one side, never silently prefers one over the other.

**PC-053** When a live PO is present at the moment a conflict is noticed, ask about it directly rather than deferring to a flag — this is the cheaper resolution and should be preferred. A flag is for when no live resolution is possible in the moment: pure file triage, or two sources disagreeing before the PO has been asked about either.

**PC-054** A topic carrying any unresolved Flag — of any type — is never treated as a known/complete answer by downstream consumption (e.g. `/anchor-project`'s pre-fill decomposition), regardless of its own Status field. Status and Flag are independent fields in the priming package; a Flag always disqualifies silent substitution, and the owning phase command must surface it as a live resolution question instead of pre-filling past it.

---

## Checklist for Verifying a Priming Command

Use this before marking any new or modified priming-producing command ready:

- [ ] PC-001  Command's output is all or part of `priming-package.md`, and it is treated as governed by this file
- [ ] PC-002  Command merges into the one project-level package — never writes a second file
- [ ] PC-003  Command stays scoped to Problem Domain (Brief, Domain, BRD) only
- [ ] PC-010  Command uses the package's existing spine mechanics — no parallel tracking scheme
- [ ] PC-011  Command uses Covered/Partial/Pending and typed Flags only — no untyped flags, no invented status labels
- [ ] PC-012  Volume handling is checkpointed for large sources
- [ ] PC-013  Sensitive-content triage applied to whatever the command reads
- [ ] PC-014  Legacy-material posture (AI Knowledge Correction / Legacy Design Question) applied where relevant
- [ ] PC-020  Facts mapped onto Sections 4-6 first, always, before any Source-Specific routing is considered
- [ ] PC-021  Section 9 vs. Section 10 usage matches their distinct purposes (one-off vs. routine class of fact)
- [ ] PC-030  Any Source-Specific subsection uses the `### 10.N Source-Specific — [Name] (owned by /command-name)` heading pattern
- [ ] PC-031  Every Source-Specific subsection names its owning command inline
- [ ] PC-032  Any new Source-Specific subsection is added to this file's registry table (Section 4)
- [ ] PC-040  Pre-Handoff Verification checks Section 10 attribution
- [ ] PC-041  Command's own handling (if it invokes or is invoked by `/anchor-project`) treats Section 10 as reference material, never a pre-filled field
- [ ] PC-042  Sections 7 and 8 are treated as reference material like Section 10 — Section 7 routed specifically to `/run-domain-discovery`, Section 8 to whichever phase is running
- [ ] PC-043  Section 9 rows surface as live questions at their probable phase, never pre-filled, never handed along silently, never auto-written into a downstream artifact
- [ ] PC-050  `Conflicting Source` recognized as a flag type; typing required uniformly across Sections 4-7
- [ ] PC-051  Conflict detection is a designed comparison step (disagreement, not just redundancy), regardless of origin
- [ ] PC-052  A `Conflicting Source` flag records every statement in full, each attributed — no silent preference
- [ ] PC-053  A live PO present at detection time is asked directly, not deferred to a flag
- [ ] PC-054  Any unresolved Flag disqualifies a topic from pre-fill/silent substitution, regardless of Status
