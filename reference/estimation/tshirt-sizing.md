# T-Shirt Sizing *(Recommended)*

**Best suited for:** AI-assisted delivery where implementation scope drives sizing more than human effort hours. The binding constraint is PO review throughput — how many PRs and change request cycles can happen in a sprint — not how fast the implementer writes code.

---

## Size Scale

| Size | Scope | AC count | Implementer time (PR ready) |
|------|-------|----------|-----------------------------|
| **XS** | Single isolated change — one function, one component, no DB migration, no external call | 1–2 | < 30 mins |
| **S** | Single endpoint or UI component; one layer touched (API or UI, not both) | 3–4 | 1–2 hrs |
| **M** | Full feature slice (API + DB + UI), external service integration, or complex business logic | 5–7 | 3–5 hrs |
| **L** | Multiple related endpoints, complex multi-state UI, or significant DB schema changes | 8–9 | 6–8 hrs |
| **XL** | Anything larger — **must be split before sprint lock** | 10+ | > 1 day |

---

## Sizing Factors

Four factors inform size. **When factors disagree, take the highest resulting size — never average.**

### Factor 1 — Acceptance Criteria Count *(primary)*

Count every criterion on the story before assigning a size.

| AC count | Size |
|----------|------|
| 1–2 | XS |
| 3–4 | S |
| 5–7 | M |
| 8–9 | L |
| 10+ | XL — split required |

### Factor 2 — Pattern Novelty

Has this type of thing been built before in this codebase?

| Situation | Adjustment |
|-----------|-----------|
| Third or later instance of an established pattern (e.g. third CRUD module following existing pattern) | Reduce one size level |
| First instance of a standard pattern (e.g. first CRUD module in this codebase) | No adjustment |
| New pattern type for this codebase (e.g. first background job, first file upload) | Increase one size level |
| External service not yet integrated | Always M minimum — never lower |

### Factor 3 — Context Footprint

How many existing files does the implementer need to read to understand the codebase before implementing?

| Files to read | Indicative size |
|--------------|----------------|
| 1–3 | XS–S |
| 4–7 | S–M |
| 8–12 | M–L |
| 13+ | L — consider splitting |

### Factor 4 — Approval Cycle Risk

Does not change the size classification. Flags stories likely to consume more calendar time. Record the risk level in the story's Notes field.

| Signal | Risk level |
|--------|-----------|
| ACs are specific and unambiguous | Low |
| Business rule has multiple edge cases | Medium |
| UI layout or visual design decisions involved | High |
| Complex external service test setup required | High |

**High-risk stories must be placed early in the sprint** so a change request cycle does not collide with the sprint boundary.

---

## Sprint Capacity

**Standard sprint:** 2 weeks (10 working days)

The binding constraint is PO review throughput — how many PR reviews and change request cycles can happen in 10 working days.

Working assumptions:
- PO reviews 1–2 PRs per working day
- ~30% of PRs receive at least one "Request Changes" response
- Each change request cycle consumes ~1 working day (implementer iterates + PO re-reviews)

**Target: 8–10 stories per sprint**

| Sprint composition | Fits? |
|-------------------|-------|
| 8× S stories | Yes |
| 4× M stories | Yes |
| 2× L + 4× M stories | Yes — tight, no additions |
| 3× L + 4× M stories | No — descope one story |
| Any XL story present | No — split before submission |

Reserve 2 working days per sprint for change request cycles. A sprint planned to 10 full days has no buffer.

---

## Split Triggers

A story must be split when any of the following apply:

- AC count reaches 10 or more
- Two or more sizing factors independently land at L
- The story crosses a module boundary (logic from two unrelated modules in one story)
- Approval Cycle Risk is High on more than one dimension simultaneously
- Implementation requires an architectural decision not yet made → write a Spike story instead

**How to split:** Find the seam — usually a layer boundary (API vs. UI) or a distinct user action. Each child story must be independently implementable, independently testable, and carry its own ACs with no overlap.

---

## Spike Stories

A spike is used when the implementation approach is unknown. It is a time-boxed investigation that produces a decision or recommendation — not production code.

| Field | Value |
|-------|-------|
| Size | Always XS or S |
| Output | Written finding: what was learned, what approach is recommended |
| FR Reference | The FR the follow-on implementation story will satisfy |
| Next step | A new, properly sized implementation story is written from the spike output |

A spike must never be sized M or above. If the investigation needs more than half a day, scope it down.

---

## Rules

1. When sizing factors disagree, take the highest resulting size — never average
2. XL is never a valid size in a locked sprint plan
3. Sprint capacity target is 8–10 stories per 2-week sprint
4. High approval cycle risk stories are placed early in the sprint during the ordering pass
5. A spike always precedes an implementation story when the approach is unknown
