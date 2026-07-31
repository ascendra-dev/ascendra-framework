# Fibonacci Story Points

**Best suited for:** Projects where the team has an existing Scrum velocity baseline, or where the client requires story point estimates for reporting. Velocity-based capacity planning is more accurate when at least one sprint of historical data exists.

**Not suited for:** First sprints with no velocity history (capacity planning degrades to guesswork), or projects where the PO has no prior Scrum experience.

---

## Point Scale

| Points | Complexity | Equivalent scope |
|--------|-----------|-----------------|
| **1** | Trivial — single isolated change, no dependencies | Equivalent to XS |
| **2** | Simple — one layer, well-understood pattern | Equivalent to XS–S |
| **3** | Moderate — one layer, some novelty or edge cases | Equivalent to S |
| **5** | Substantial — full feature slice or first instance of a pattern | Equivalent to M |
| **8** | Complex — multiple layers, external integration, significant schema change | Equivalent to L |
| **13** | Very complex — consider splitting | Equivalent to L–XL boundary |
| **21+** | Must be split before sprint lock | XL — not acceptable in a locked sprint |

---

## Sizing Guidance

Assign points by comparing the story to others already estimated in this project. Relative sizing is more reliable than absolute.

| Approach | When to use |
|----------|------------|
| **Anchor comparison** — pick a known S story as your 3, score everything relative to it | When 1+ sprints of stories already exist |
| **AC count proxy** — use AC count as the primary signal (1–2 ACs = 2pts, 3–4 = 3pts, 5–7 = 5pts, 8–9 = 8pts, 10+ = 13pts) | First sprint, no anchors yet |

**When factors conflict, take the higher points value — never average.**

---

## Sprint Capacity

Capacity is planned by **velocity** — the total story points completed in a sprint.

**For the first sprint (no velocity history):**
Start conservatively. Assume a velocity of 20–25 points for a 2-week sprint with daily PO review availability. Adjust after the sprint closes based on actual points completed.

**For subsequent sprints:**
Use the average velocity of the last 2 completed sprints as the capacity ceiling. Do not plan above it.

| Sprint composition example | Points | Fits at velocity 20? |
|---------------------------|--------|----------------------|
| 4× 3pt + 2× 5pt | 22 | Yes — at limit |
| 3× 5pt + 1× 8pt | 23 | Yes — at limit |
| 2× 8pt + 2× 5pt | 26 | No — over capacity |
| Any 13pt story + others | 13+ | Only if velocity ≥ 25 |

Reserve ~20% of velocity for change request rework cycles on PRs that come back with changes.

---

## Split Triggers

A story must be split when any of the following apply:

- Points reach 13 or more
- The story spans two independent user goals that could ship separately
- Estimating the story triggers significant disagreement (signals hidden complexity — split to expose it)
- Implementation requires an architectural decision not yet made → write a Spike story instead

---

## Spike Stories

A spike is used when the implementation approach is unknown. It is a time-boxed investigation that produces a decision or recommendation — not production code.

| Field | Value |
|-------|-------|
| Points | Always 1 or 2 |
| Output | Written finding: what was learned, what approach is recommended |
| Next step | A new, properly estimated implementation story is written from the spike output |

---

## Rules

1. Points measure complexity relative to other stories — not hours of effort
2. 21+ points is never valid in a locked sprint plan
3. First sprint: plan to 20–25 points conservatively; calibrate after close
4. Subsequent sprints: plan to average of last 2 sprint velocities
5. Reserve ~20% of velocity for PR change request cycles
6. A spike always precedes an implementation story when the approach is unknown
