# FW-038 — Audit Column Shape Discovery in `/gen-architecture`

| Field | Value |
|-------|-------|
| **ID** | FW-038 |
| **Date** | 2026-07-17 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/gen-architecture.md` (Step 4, `database-standards.md` generation), `.claude/commands/implement-story.md` |
| **Amends** | FW-035 — Audit Column Symmetry; FW-036 — Audit Column Population Mechanism |
| **Related** | FW-037 — Audit Column Actor-Label Design (project-level fix for `ASCENDRA-PAY-001` that this decision generalizes into the framework) |

---

## Decision

`gen-architecture.md`'s instructions for generating `database-standards.md` no longer hardcode the actor columns (`created_by`/`updated_by`/`deleted_by`) as a `uuid` foreign key into a single actor table. Step 4 (Constraints Discovery) now derives an **Actor Identity Shape** from BRD Section 3 (User Roles) — whether the project has one authenticated, in-tenant actor identity space, or more than one (an unauthenticated/no-login actor, a cross-tenant internal role, background/scheduled processes) — the same "derive from the documents, present to the PO, ask only what can't be inferred" pattern already used for multi-tenancy shape and the rest of Step 4. The derived shape then determines which of two documented column shapes `database-standards.md` states: FK-based (single actor table) or a self-contained actor-label (multiple actor identity spaces, no FK, no type column). `implement-story.md`'s corresponding instruction now reads whichever shape a project's own `database-standards.md` states, instead of assuming the FK variant.

---

## The Problem That Triggered This Decision

`FW-035`/`FW-036` fixed audit-column existence and population discipline at the framework level, but the accompanying `database-standards.md`-generation instructions in `gen-architecture.md` (and the parallel implementation instruction in `implement-story.md`) went further than either of those decisions actually intended: they stated the actor columns are a foreign key "into the project's own actor table" — singular, assumed, never derived. `ASCENDRA-PAY-001` hit the limits of that assumption directly: it has at least three actor identity spaces (`issuer_users`, `ascendra_staff`, and unauthenticated Payers reached only via a tokenized link), which a single-table FK cannot represent — see `FW-037` for the full resolution on that project. Left uncorrected here, every future project with a similar actor shape (common for any product with an external/unauthenticated user role, a cross-tenant admin role, or background jobs) would rediscover the identical gap independently, the same way `US-01-003`'s planning session did for this one.

---

## Why a Discovery Step, Not a Different Hardcoded Default

Simply flipping the framework's default from "always FK" to "always actor-label" would repeat the same mistake in the other direction — plenty of projects genuinely have one uniform actor table, where the FK shape is simpler and gets referential integrity for free at no real cost. The right fix is to make the *shape* a derived, PO-confirmable decision, the same way `gen-architecture.md` Step 4 already treats multi-tenancy shape, scale profile, and compliance signals: derivable from the BRD, presented to the PO, corrected only where the documents don't already say enough. This keeps `/gen-architecture` consistent with its own established Step 4 pattern instead of introducing a one-off interview just for this column, and keeps the framework's on-the-fly, project-specific architecture philosophy (`FW-025`) intact rather than substituting one silent assumption for another.

Whether audit columns exist *at all* was deliberately left as-is (default include, omit only with a stated reason) — not turned into a per-project yes/no question. Nearly every production system needs some write-history trail, and the existing opt-out escape hatch already covers the genuine exception case without demanding an explicit question every time.

---

## What Changed

- **`gen-architecture.md` Step 4** — added an "Actor identity shape" row to the constraints-derivation table (derived from BRD Section 3), and a corresponding bullet in the "Present what you derived" block, presented for confirmation alongside the existing derivations.
- **`gen-architecture.md`'s `database-standards.md` generation instructions** — the actor-column guidance now states two documented shapes (FK-based vs. actor-label) keyed off the Step 4 derivation, instead of asserting the FK shape unconditionally. The population-mechanism paragraph now covers both shapes and states, once, generally: which of the three columns gets written is decided by the operation being performed, never by a type/flag parameter passed alongside the actor value — true for both shapes, stated at the source so it flows into whichever a project generates, rather than being restated (and risking drift) in every consuming command.
- **`implement-story.md`'s database-schema implementation rule** — generalized from "thread the actor id... foreign key" to "thread the acting value; the exact shape is whatever this project's own `database-standards.md` Rule 4 states" — read the project artifact, don't assume.

---

## Implementation Status

Applied immediately during this session — `gen-architecture.md` and `implement-story.md` edited directly. No existing project's already-generated `database-standards.md` is affected by this framework-level change on its own; `ASCENDRA-PAY-001`'s was already corrected directly via `FW-037`/`/assess-change`. `review-architecture.md` was checked and needs no change — none of its 34 structural checks inspect actor-column shape.

---

## Relationship to Other Decisions

Generalizes `FW-037` (a project-level fix for `ASCENDRA-PAY-001`) into the framework's own generation and implementation commands, so future projects derive the right shape instead of rediscovering the same gap. Amends `FW-035`/`FW-036` in the same sense `FW-037` did — refining shape and guidance, not existence or the core "population is always manual" rule, both of which stand unchanged.
