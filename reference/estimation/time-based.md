# Time-Based Sizing

**Best suited for:** Projects with time-tracking requirements (regulatory, client billing by hour, government contracts), very short sprints (1 week or less), or clients who require hour estimates per story for approval.

**Not suited for:** Projects where the implementer is an AI assistant — AI implementation speed does not map linearly to human hours, making hour-based capacity planning unreliable. If this standard is selected for an AI-assisted project, treat hours as a complexity proxy, not a literal time prediction.

---

## Size Bands

| Band | Hours | Scope |
|------|-------|-------|
| **XS** | < 2h | Single isolated change — one function, one component, no migration, no external call |
| **S** | 2–4h | Single endpoint or UI component; one layer touched |
| **M** | 4–8h | Full feature slice (API + DB + UI), external service integration, or complex business logic |
| **L** | 8–16h | Multiple related endpoints, complex multi-state UI, or significant DB schema changes |
| **XL** | 16h+ | Must be split before sprint lock |

*Hours = implementation start to PR ready. Does not include review time or change request cycle time.*

---

## Sizing Guidance

Estimate by reasoning through the implementation steps, not by gut feel.

For each story:
1. List the implementation steps (e.g. schema migration, service method, controller endpoint, UI component, unit tests)
2. Estimate hours per step based on complexity and novelty
3. Sum the steps
4. Round up to the nearest band

**When in doubt, round up — not down.** Under-estimation is the most common planning failure.

**Pattern novelty adjustment:**
- Repeating a well-established pattern (e.g. third CRUD module): reduce estimate by 20–30%
- First instance of a new pattern (e.g. first file upload, first background job): increase estimate by 30–50%
- External service not yet integrated: minimum M (4h), never lower

---

## Sprint Capacity

**Standard sprint:** 2 weeks (10 working days)

Capacity = available implementation hours per sprint, minus time reserved for PR review cycles.

| Sprint type | Total hours | Reserve (review cycles) | Available for stories |
|-------------|-------------|------------------------|----------------------|
| 2-week sprint, full-time implementer | 80h | 16h (~20%) | ~64h |
| 2-week sprint, part-time implementer | 40h | 8h (~20%) | ~32h |
| 1-week sprint, full-time implementer | 40h | 8h (~20%) | ~32h |

Plan to the "Available for stories" figure. Do not plan to total hours — change request cycles will consume the reserve.

---

## Split Triggers

A story must be split when any of the following apply:

- Estimated hours reach 16h or more
- Two or more implementation steps are independently testable and could ship separately
- The story spans two module boundaries
- Implementation requires an architectural decision not yet made → write a Spike story instead

---

## Spike Stories

A spike is used when the implementation approach is unknown. It is a time-boxed investigation that produces a decision or recommendation — not production code.

| Field | Value |
|-------|-------|
| Size | Always XS (< 2h) — time-boxed investigation only |
| Output | Written finding: what was learned, what approach is recommended |
| Next step | A new, properly estimated implementation story is written from the spike output |

If the investigation cannot be completed in under 2h, scope it down. A spike that needs a full day is a sign the unknowns are too large — escalate to the PO before continuing.

---

## Rules

1. Estimate by reasoning through implementation steps, not by feel
2. Round up when uncertain — never down
3. XL (16h+) is never valid in a locked sprint plan
4. Reserve 20% of available hours per sprint for change request rework
5. Apply pattern novelty adjustment to every story where the pattern has or has not been used before
6. A spike always precedes an implementation story when the approach is unknown
