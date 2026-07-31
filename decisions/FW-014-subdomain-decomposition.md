# FW-014 — Sub-Domain Decomposition

| Field | Value |
|-------|-------|
| **ID** | FW-014 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | Domain knowledge documents, discovery phase |

---

## Decision

A domain knowledge document should cover one cohesive sub-domain. Large domains that span multiple distinct capability areas are split into separate domain documents — one per sub-domain. This is a best practice recommendation, not a hard constraint. The framework guides but does not enforce.

---

## The Problem

A domain knowledge document for a large system (e.g. a school management platform covering admin, teacher, parent, and student portals) becomes unmanageably large if treated as a single document. Consequences:

- **Context window degradation** — every command that loads the domain document pays the full context cost. A 2000-line document leaves little room for the artifacts being generated.
- **Discovery session focus loss** — a discovery session covering all portals at once produces shallow coverage of each. A session focused on one sub-domain produces deep, accurate requirements.
- **Mixed concerns** — entities, rules, and lifecycles for different user types become entangled. Cross-sub-domain assumptions creep in.

---

## The Solution: Sub-Domain Decomposition

Split large domains into focused sub-domain documents:

**Instead of:**
```
domain/school-management-core.md       ← covers everything
```

**Do this:**
```
domain/school-admin-core.md            ← admin portal sub-domain
domain/school-teacher-core.md          ← teacher portal sub-domain
domain/school-parent-core.md           ← parent portal sub-domain
domain/school-student-core.md          ← student portal sub-domain
```

Each document is:
- Independently loadable — commands load only what they need
- Independently discoverable — one discovery session per sub-domain
- Independently verifiable — Section 12 checks run per document
- Independently reusable — a school-parent sub-domain document is reusable across different school systems

---

## When to Split

Split when the domain covers more than one of:
- A distinct user-facing portal with its own personas and journeys
- An independent capability area with its own entities and lifecycle (e.g. fee management vs timetabling)
- A separately deliverable module that could ship independently

Do not split purely by technical layer (API vs frontend). Sub-domains are business capability boundaries, not technical boundaries.

---

## Phases vs Sub-Domains

If Phase 1 covers the admin portal and Phase 2 covers the parent portal, these are **different sub-domains**, not phases of the same domain. Each sub-domain gets its own domain document. The brief-driven extension model (`FW-016` — Extension Context fields in `brief.md`, not a `composes` field in `project.json`; see `FW-027`) handles the cross-phase dependency — Phase 2 project extends Phase 1. No separate domain-level phase mechanism is needed.

---

## Enforcement

This is a **best practice guidance**, not a hard constraint. The framework makes the right path obvious through:
- The AI Guide note in `domain.template.md` Section 1.1 — surfaces the recommendation at document creation time
- This decision document — available to POs who want to understand the principle

The PO decides. The framework does not block generation of a large single document if that is the PO's choice.
