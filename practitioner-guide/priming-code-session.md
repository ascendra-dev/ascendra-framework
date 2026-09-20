# Priming Code Session

Covers [`/run-priming-code-session`](../.claude/commands/run-priming-code-session.md), governed by the same [`conventions/priming-command-conventions.md`](../conventions/priming-command-conventions.md) contract as [Priming Session](priming-session.html). Read that chapter first if you haven't — this one only covers what's genuinely different about reading code instead of prose. No stage of its own; scoped to Problem Domain only, same as its sibling command.

---

## Purpose

`/run-priming-session` and this command produce the same output — `priming-package.md` — but solve a different problem. A PO narrating their business in conversation already knows what matters and why; a legacy codebase has no author in the room to ask, and it accumulated unintentional behavior right alongside its real business rules. This command exists to reverse-engineer business capability from code structure, and to do it without ever reading the whole repository — a codebase is often hundreds of files, and holding all of them in context at once isn't the constraint this command is built to work around.

Two problems had to be solved before this command could exist at all, and both are worth understanding because they shape everything else in this chapter: how do you tell a real business rule apart from an accident, with nobody around to ask? And how do you chunk a codebase too large to read in full, without silently presenting a guess as if it were precision?

---

## Entity-centric chunking, with honest degradation

The Map pass is structural only — no file contents read yet. It walks the directory and module structure, and for each module it looks for recognizable schema or ORM patterns: an annotated entity class, a models file, a schema definition, a migration file. These are framework-agnostic concepts, not stack-specific rules — recognizing them is the same ordinary pattern-reading you'd do reading any of these stacks directly, in any language.

Where a module's entity layer is directly discoverable, chunking proceeds entity by entity — the finest, most useful granularity. Where it isn't — raw-SQL-heavy code, no formal ORM, or any codebase with no discoverable entity boundary — the command doesn't give up and process the whole module as one undifferentiated blob. It attempts entity inference *within* the module first, including inferring candidate entities from table names appearing in raw SQL query strings when that's the only signal available. Only when entity boundaries genuinely can't be found with confidence does it degrade to Module/Layer granularity — and when it does, that's recorded explicitly in the Entity Ledger's own Granularity column, never silently presented as if it achieved entity-level precision.

<div class="rule-callout">A module-level row is never dressed up as an entity-level one. If the walk couldn't find the boundary, the record says so.</div>

---

## The reverse-engineering heuristic: telling a rule from an accident

This is the problem prose priming never had to solve, because a human wrote prose with intent. Code accumulates unintentional behavior too, and the walk has to guess which is which from the evidence in front of it.

**Leans toward a real business rule** — a candidate for `AI Knowledge Correction` if it diverges from generic domain understanding:
- Enforced redundantly across layers (a DB constraint, plus application validation, plus the UI disabling an action)
- A business-language error message, not a generic technical one
- Named consistently across unrelated parts of the codebase
- Traceable to a comment or commit citing a business or legal reason

**Leans toward a legacy design flaw** — a candidate for `Legacy Design Question`, never carried forward as fact:
- Inconsistent enforcement — checked on one path, silently skipped on another
- An unexplained magic number, or code marked `HACK`/`TODO`/`temporary`
- Contradicts what's generically true of the domain elsewhere
- Duplicated or denormalized data with no stated reason

The same legacy-material posture from the prose-priming chapter applies here — a flaw candidate is never resolved unilaterally, only ever framed as a negotiation for a human to settle. The one real difference: if a PO happens to be live during the walk, ask directly, since that's cheaper resolution than deferring to a flag. If no PO is present — the expected default for this command, unlike its prose sibling — the flag waits for whichever later session actually reads it.

---

## Signal sources, ranked by trust

Extract pulls a unit's full signal cluster via targeted search across the repository for its name and references — never a blanket read of its "home" directory. Not every unit has all seven signals; use whichever actually turn up something, in this order:

1. **Schema/ORM definitions** — highest trust: entities, lifecycle states, relationships.
2. **Validation logic** — guards, DTO decorators, DB constraints.
3. **State machines** — enums plus transition guards.
4. **Test names and assertions** — a first-class source, not a fallback. Often clearer than the implementation itself.
5. **Comments and commit messages** citing a person, ticket, or regulation — cross-checked against current actual behavior, since comments rot.
6. **Feature flags and per-tenant config** — maps directly onto Known Variations.
7. **API surface** — routes, payload shapes. Medium signal; can be generic scaffolding.

The ranking matters because it tells you which signal to trust when two disagree — a schema constraint outranks a stale comment, a test assertion outranks an API route that might just be boilerplate.

---

## The two subsections this command actually owns

Its prose sibling owns no Section 10 (Source-Specific Material) subsection at all — everything it extracts maps onto Sections 4–6. This command is the exception, and owns two:

- **Code Structure Reference** — existing API surface and schema-as-implemented. Captured only incidentally, while extracting for domain purposes — never pursued on purpose. Priming is scoped to Problem Domain only; this content is Architecture-flavored, and going looking for it on purpose would mean this command quietly doing architecture's job early.
- **Sub-Domain Split Recommendation** — a non-binding recommendation, derived from the codebase's own module boundaries, that the eventual domain document(s) may need splitting (the same test Chapter 2 taught you to apply from the brief, applied here from the code's own structure instead). This one routes specifically to `/run-domain-discovery`, not to whichever phase happens to run next — a routing exception worth remembering, because it's the one case in this whole framework where a priming artifact names its own destination phase rather than leaving that to whoever reads it.

---

## Checkpointing: per unit, not per session

Progress is tracked in a dedicated working-state file, `code-priming-state.md`, kept distinct from `priming-package.md` itself — the same relationship a discovery-state file already has to its own finished artifact. Every unit's row is written to Extracted (or Triaged, for an infrastructure-likely one) immediately after it finishes, before moving to the next unit. The ledger, not conversation history, is what makes a large codebase resumable across sessions — a walk can checkpoint and resume any number of times, and a genuinely large file taking more than one session to fully triage is entirely normal, not a sign something went wrong.

<div class="caution">Do not hold more than one unit's extraction in working context at a time. The discipline that makes this command scale to a real legacy codebase is exactly the discipline that makes it feel slower unit by unit — resist the urge to batch several units' extraction together to save a round trip.</div>

---

## Terminology recap

- **Domain-likely vs. infrastructure-likely** — the triage test applied to every unit: would every deployment of this base system need this, regardless of which extension is active? Auth plumbing, logging, and generic admin CRUD are infrastructure-likely and get one boundary note. A business entity or rule-bearing validator is domain-likely and proceeds to full Extract.
- **Entity vs. Module/Layer granularity** — recorded explicitly in the Entity Ledger, never silently upgraded. A Module/Layer row is an honest admission the entity boundary couldn't be found with confidence, not a lesser-effort shortcut.
- **`code-priming-state.md` vs. `priming-package.md`** — a working ledger vs. the actual PO-facing deliverable. The ledger is operational bookkeeping; only the priming package is read by later commands.

---

## Common mistakes

- Reading a module's whole "home" directory instead of a targeted search for the unit's own name and references.
- Presenting a Module/Layer-granularity result as if it achieved entity-level precision.
- Carrying forward an inconsistently-enforced rule as fact instead of flagging it as a `Legacy Design Question`.
- Trusting a comment over a schema constraint when the two disagree, instead of following the signal trust order.
- Going looking for architecture-flavored facts on purpose instead of only noting them when they surface incidentally.
- Batching several units' extraction together in one pass instead of checkpointing after each one.
- Treating the Sub-Domain Split Recommendation as a decision this command makes, rather than a non-binding note for `/run-domain-discovery` to actually weigh.

---

## Harborview in practice

Be honest about what's actually here: Harborview's own domain was judged well-known enough to skip even the prose priming sub-flow's deepest path, and no legacy codebase was ever part of its worked material — there's no `code-priming-state.example.md` in this repository, and this command hasn't yet been validated against a real legacy walk. That's a real, stated gap in the framework itself, not something this guide is glossing over.

A brief, clearly-labeled hypothetical, not drawn from any real file: imagine Harborview's old spreadsheet-and-email process had instead been a small, ten-year-old PHP invoicing tool nobody remembers the reasoning behind. A `getInvoiceTotal()` function computes VAT with a hardcoded `0.20` multiplier — no config, no per-client override anywhere in the codebase. That's a candidate for `AI Knowledge Correction`, not a flaw: it's enforced the same way everywhere the function is called, and matches what James actually confirmed live in the real priming session (VAT is always 20%). Now imagine a second function, `applyLegacyDiscount()`, invoked from exactly one billing path and nowhere else, silently zeroing out VAT for a hardcoded list of three client IDs with a comment reading `// TEMP - remove after Q3`. That's the other case — inconsistent enforcement, an unexplained exception, a `TEMP` marker years old. It gets flagged `Legacy Design Question` and put to whoever's live for a real answer, not carried forward as if Harborview's invoicing genuinely has a VAT exemption rule.
