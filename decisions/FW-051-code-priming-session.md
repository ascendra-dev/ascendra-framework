# FW-051 — Code Priming Session

| Field | Value |
|-------|-------|
| **ID** | FW-051 |
| **Date** | 2026-09-19 |
| **Status** | Decided — implemented, pending validation against a real legacy codebase |
| **Area** | `.claude/commands/run-code-priming-session.md` (new), `projects/TEMPLATE/priming/code-priming-state.template.md` (new), `conventions/priming-command-conventions.md` (`PC-016`, `PC-032`, `PC-034` added), `projects/TEMPLATE/priming/priming-package.template.md` (audience block + Section 10 note), `CLAUDE.md` |
| **Depends on** | `FW-049` (Anchor Project and Priming Session) — this is the deferred second priming-producing command its own §5.11 named and left for later |

---

## Decision

`/run-code-priming-session {PROJECT_CODE}` is the second priming-producing command, alongside `/run-priming-session` — same output (`priming-package.md`), same governing contract (`conventions/priming-command-conventions.md`), different extraction mechanic: it reads an existing legacy codebase (dropped into the same `source-material/` folder every priming command already gates on) instead of prose, and reverse-engineers business capability from code structure rather than reading linearly.

Chunking is entity-centric where a schema/ORM layer is directly discoverable (any language or framework — this is stack-agnostic by construction, since recognizing an annotated class, a models file, or a schema definition is ordinary pattern-reading, not something requiring per-stack rules), gracefully degrading to module/layer granularity via a Repository Map where it isn't (raw-SQL-heavy code, no formal ORM, or any codebase with no discoverable entity boundary). The walk never reads the full repository — every pass is structural-navigation- or targeted-search-driven — and progress is checkpointed per unit in a dedicated working-state file, `code-priming-state.md`, kept distinct from `priming-package.md` itself (`PC-016` — a working-state file is operational bookkeeping, not a second PO-facing deliverable, the same relationship `domain-discovery-state.md`/`brd-discovery-state.md` already have to their own artifacts).

The command owns two Section 10 (Source-Specific Material) subsections, registered in `priming-command-conventions.md`'s `PC-032` table: **Code Structure Reference** (existing API surface, schema-as-implemented — captured only incidentally while extracting for domain purposes, never pursued on purpose, since it's Architecture-flavored content and `PC-003` scopes priming to Problem Domain only) and **Sub-Domain Split Recommendation** (a non-binding recommendation, derived from the codebase's own module boundaries, that the eventual domain document(s) may need splitting per `FW-014`) — the latter needed a new routing exception, `PC-034`, since it's only actionable at `/run-domain-discovery`, not "whichever phase is about to run" (`PC-041`'s generic default).

No new gate check: reuses `/init-project`'s existing `source-material/` folder (the same one `/run-priming-session` already gates on). The "does this actually look like a codebase" sanity check is folded into the Map pass's own first result rather than a standalone check — a Map pass that turns up nothing recognizable fails fast rather than producing a near-empty priming package.

---

## Why This Was Needed

`FW-049` named legacy code as "a fundamentally different extraction mechanic" that would need its own command, deferred at the time because two things hadn't been designed: the reverse-engineering step (translating "what the code does" into "what business capability it reflects," with no human author's intent to lean on the way prose priming can) and a chunking strategy for a codebase too large to read in full. Both were resolved in a discussion-first design pass, matching the same discuss-then-build discipline the rest of this framework's design work follows, before any command file was written.

**The reverse-engineering heuristic** — telling a real business rule apart from an accident, with no PO in the room to ask, the one problem prose priming never had to solve:
- Leans toward real rule (→ `AI Knowledge Correction` if it diverges from generic domain understanding): enforced redundantly across layers, a business-language error message, named consistently across unrelated parts of the codebase, traceable to a comment/commit/ticket citing a business or legal reason.
- Leans toward legacy design flaw (→ `Legacy Design Question`, never carried forward as fact): inconsistent enforcement across paths, an unexplained magic number or `HACK`/`TODO` marker, contradicts what's generically true of the domain elsewhere, duplicated/denormalized data with no stated reason.

**Signal sources, ranked by trustworthiness** (used in this order during Extract): schema/ORM definitions → validation logic → state machines → test names and assertions (a first-class source, not a fallback — often clearer than the implementation itself) → comments/commit messages citing a person, ticket, or regulation (cross-checked against current behavior, since comments rot) → feature flags/per-tenant config (maps directly onto Known Variations) → API surface (medium signal, can be generic scaffolding).

**"Bounded context" was considered and rejected as new vocabulary.** Whether a codebase's own module boundaries matter turned out to be two independent axes the framework already has separate answers for, not one new concept: separability (reuse `FW-014` sub-domain decomposition — surfaced as a non-binding recommendation to `/run-domain-discovery`, never a decision this command makes itself) and depth/priority (reuse the generalization filter, applied via a concrete code-triage signal: domain-likely vs. infrastructure-likely). A bounded context can *be* the core of a system, so "it's separable" was never a safe proxy for "give it less depth" — keeping the axes independent avoids that trap without importing DDD vocabulary the rest of the framework doesn't use.

**"Standard/extended scope" turned out to be Core/Extension restated, not a third concept** — confirmed directly rather than assumed, since the phrase could plausibly have meant the Universal-Business-Rule/Known-Variation axis instead (a different layer: rule truth-value, not deployment scope). No new mechanism needed once confirmed.

**Honesty over false precision, at two levels.** Neither the chunk boundary nor an individual fact is ever presented with more confidence than it earned: a unit that can't be resolved to entity granularity is recorded as `Module/Layer` granularity, not silently upgraded; a fact that can't be found after a reasonable search is marked `Pending`, never fabricated — the same standard `/run-priming-session` already holds itself to, extended to cover the chunk-boundary case prose priming never had.

---

## Relationship to Other Decisions

- **`FW-049`** — this is the deferred second priming-producing command its own §5.11 named; governed by the same `conventions/priming-command-conventions.md` contract, contributing to the same `priming-package.md`.
- **`FW-014`** (Sub-Domain Decomposition) — the separability axis above reuses this decision directly rather than introducing "bounded context" as new vocabulary.

---

## A Note on This Decision's Own Documentation

No companion root-level `*-DESIGN.md` scratch file is kept for this decision. The working file used during design (`CODE-PRIMING-DESIGN.md`) was deleted once its content was distributed here and into the executable command, template, and conventions files themselves — this ADR is the sole permanent record of the design reasoning, and nothing in the framework depends on a file outside `decisions/` or the command files to be complete.

---

## Implementation Status

Built and self-checked, 2026-09-19:

- [x] `.claude/commands/run-code-priming-session.md` — new command, self-checked against `conventions/command-conventions.md`
- [x] `projects/TEMPLATE/priming/code-priming-state.template.md` — new template, the per-unit Map/Triage/Extract checkpoint ledger
- [x] `conventions/priming-command-conventions.md` — `PC-016` (working-state file clarification), `PC-032` registry entries, `PC-034` (routing exception) added
- [x] `projects/TEMPLATE/priming/priming-package.template.md` — `/run-code-priming-session` audience block added (`template-conventions.md` T-005a), Section 10 guide updated to name the real command
- [x] `CLAUDE.md` — command table entry added

Not yet done:

- [ ] Validation against a real legacy codebase — the entity/module-degradation split, the reverse-engineering heuristic, and the checkpoint mechanism have not yet been exercised by an actual walk
