# FW-024 — Parking Lot for Undiscovered Deferred Capabilities

| Field | Value |
|-------|-------|
| **ID** | FW-024 |
| **Date** | 2026-07-10 |
| **Status** | Decided |
| **Area** | BRD template (`projects/TEMPLATE/brds/brd.template.md`, `brd.example.md`), `/gen-brd`, `/review-brd`, `/assess-change` |

---

## Decision

The BRD template gains a new **Section 15 — Parking Lot**, placed after Section 14 (Approval). It holds ideas that surface outside the discovery process — during epic review, story writing, or ongoing delivery — that are not yet requirements: no domain knowledge model backs them, and no discovery session has explored them. This is distinct from Section 5.21 (Future Capabilities), which holds deferred capabilities that already have full domain/BRD backing (e.g. Estimates/Quotes, Customer Statements) and only need a REQ-ID to be traceable.

A Parking Lot entry can never be promoted directly into a real REQ-ID by `/assess-change`. Promotion requires a domain and/or BRD discovery pass first (`/run-domain-discovery`, `/run-brd-discovery`, or an equivalent live client/PO conversation) — `/assess-change` is gated to refuse authoring REQ text for an item that only exists as a Parking Lot row.

`/review-brd`'s existing Section 15 (Review Record) shifts to Section 16 to make room.

---

## The Problem That Triggered This Decision

During epic review for ASCENDRA-PAY-001 / EPIC-002 (Issuer Onboarding & Verification), the PO asked whether Ascendra staff should be able to onboard an Issuer on the Issuer's behalf, rather than only self-service signup. Checking the BRD, domain knowledge doc, and epic confirmed this capability appears nowhere — not as an entity, not as a rule, not as an Out of Scope bullet.

The instinct was to treat this like the project's existing precedent — REQ-067 (Estimates/Quotes) and REQ-068 (Customer Statements), both added to Section 5.21 (Future Capabilities) during epic review with no cascade beyond the BRD. But that precedent only works because the domain model already fully backs those two capabilities (UBR-018, UBR-019, UBR-021 pre-existed in the domain doc); the REQ text is a one-line label pointing at real modeling that already happened. Staff-assisted onboarding has no such backing — writing a REQ for it now would mean inventing requirement text (personas, consent model, audit trail, account-creation flow) on the spot, mid epic-review, with no discovery behind it. That is exactly the failure mode `/assess-change`'s formality tiers exist to prevent for in-scope changes — but nothing stopped it for a deferred one, because Pattern 4 assumes the requirement text is already ready to write.

Section 5.21 itself was also never in the BRD template — it was improvised mid-project the first time this need arose (see BRD Document Control v1.6). Left unaddressed, every future project would re-invent its own ad hoc holding section, with no consistent promotion discipline.

---

## What Distinguishes Section 15 (Parking Lot) from Section 5.21 (Future Capabilities)

| | Section 5.21 (Future Capabilities) | Section 15 (Parking Lot) |
|---|---|---|
| Domain modeling | Already exists (entity/lifecycle/rule in the domain knowledge doc) | Does not exist yet |
| Discovery | Already happened (client stated it, or it's a known domain variation) | Has not happened |
| Gets a REQ-ID | Yes, immediately | No — only after promotion |
| Priority / Layer / Source tags | Yes | No — these fields don't apply to an undiscovered idea |
| `/assess-change` can write it directly | Yes (Pattern 4, BRD-only blast radius, no epic/story cascade) | No — blocked until discovery happens |

If in doubt which section an item belongs in: can you write one factual sentence describing exactly how it will behave, backed by something already in the domain doc? If yes, Section 5.21. If the honest answer requires guessing, Section 15.

---

## How Commands Use This

**`/gen-brd`** — Section 15 is typically empty at initial generation ("None identified"). Parking Lot entries accumulate later, during epic review, story writing, or delivery — not during discovery-driven authoring.

**`/review-brd`** — Section 15 (Parking Lot) is informational, not walked through line-by-line like Sections 5/6/8/9 — there is nothing to confirm against the client yet. `/review-brd`'s own Review Record moves to **Section 16** to make room.

**`/assess-change`** — gains two behaviors:
1. Adding a *new* Parking Lot entry is BRD-only (like Section 5.21 additions): version bump, Document Control row, no epic/story cascade, since nothing is entering scope.
2. Promoting an *existing* Parking Lot entry into a real REQ is gated: `/assess-change` must check whether the requested change only exists today as a Parking Lot row. If so, it stops and directs the PO to run discovery first — it does not author the REQ text itself. Once discovery produces real requirement text (via a domain/BRD discovery session or an equivalent documented conversation), `/assess-change` Pattern 4 applies normally, and the Parking Lot row is marked `Promoted → REQ-XXX`.

**Epics** — an epic's Out of Scope section may cite a Parking Lot ID directly (e.g. `PL-001`) instead of a REQ-ID, explicitly marked as "not yet a formal requirement, pending discovery" — this is a permitted exception to the usual "every Out of Scope item needs a REQ-ID" rule, since a Parking Lot item is by definition not yet at REQ maturity.

---

## Implementation Status

Applied immediately during this session:
- `projects/TEMPLATE/brds/brd.template.md` — added Section 15 (Parking Lot); added one check to 13.1 (Content Integrity)
- `projects/TEMPLATE/brds/brd.example.md` — added matching worked example
- `.claude/commands/review-brd.md` — renumbered Review Record Section 15 → 16
- `.claude/commands/gen-brd.md` — added Section 15 generation guidance
- `.claude/commands/assess-change.md` — added Parking Lot entry handling and the discovery-gated promotion rule
- Applied retroactively to ASCENDRA-PAY-001's BRD (v1.9): added Section 15 with PL-001 (Ascendra-staff-assisted Issuer onboarding), cited from EPIC-002 Section 3.2

---

## Relationship to Other Decisions

- **FW-021** (Template Conventions) — Section 15 follows the same `[AI Guide]` annotation convention as every other template section.
- **FW-022** (Problem Domain / Solution Domain Boundary) — a Parking Lot entry is pre-problem-domain: it hasn't even entered discovery yet, let alone been scoped. Promotion moves it into the problem domain proper (BRD → Epics) only once discovery has run.
