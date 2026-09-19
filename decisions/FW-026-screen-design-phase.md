# FW-026 — Screen Design Phase

| Field | Value |
|-------|-------|
| **ID** | FW-026 |
| **Date** | 2026-07-11 |
| **Status** | Decided — implementation in progress |
| **Area** | New phase between Epic Approval and Architecture; `gen-architecture.md`, `gen-ui-mocks.md`, `implement-story.md`, `review-architecture.md`, `init-project.md`, [`CLAUDE.md`](../CLAUDE.md), [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md), [`SDLC.md`](../SDLC.md) |

---

## Decision

A new Screen Design phase is inserted between Epic Approval and `/gen-architecture`. It produces its own reviewed, approved artifact (`screens/screen-design.md`) covering Portals, Navigation Map, and Screen Inventory — the three subsections of the former architecture Section 3.1.3 that only ever depended on the BRD, never on the confirmed tech stack. Architecture Section 3.1.3 no longer originates this content; it carries it over from the approved Screen Design artifact and adds only the two subsections that genuinely need architecture (3.1.3.4 Action Gating, 3.1.3.5 Cross-Cutting UI Rules — both depend on Section 6's security model).

Screen Design also introduces explicit pattern-matching against `ascendra-ui`'s real component catalog: every screen is assigned a Surface (Page/Dialog/Sheet/Drawer) and UI Pattern (Form/Dashboard/Report/Table-List/Detail), each cited to a specific named example in `ascendra-ui/docs/ui-reference.md` Part 2 (or, for Report/Table-List, to a primitive composition — no Part 2 composite exists for those two) — never an invented shape. `/gen-ui-mocks` generates real mocks from this immediately (base mode), then patches in gating/cross-cutting content once architecture reaches Section 6 (patch mode). `/implement-story` reads the same Surface/Pattern decision before building any Web UI, so the PO-approved mock is what actually gets built, not an independent second guess.

Unlike Domain Discovery (`FW-013`), Screen Design is **not optional**. Domain Discovery's optionality rests on well-known domains being sufficiently covered by AI training knowledge. Screens have no equivalent shortcut — they are project-specific by definition, derived from this project's BRD journeys and FRs, so there is no "well-known enough to skip" case.

---

## Why This Was Needed

Three compounding problems, found while auditing `/gen-ui-mocks` for a PO-requested quality review:

1. **`/implement-story` never read architecture Section 3.1.3 at all** (confirmed by grep — zero references). Mocks and real implementation each independently guessed screen shape, with no shared source of truth. A PO-approved mock carried no weight over what actually got built.
2. **Section 3.1.3.3 (Screen Inventory) had no field for screen shape.** Columns were `Screen ID | Portal | Route | Reachable by | Purpose | Source` — nothing recorded whether a screen was a routed page, a modal dialog, a slide-out sheet, or a bottom drawer, nor what composite UI pattern (form, dashboard, report, table, detail view) it should follow.
3. **API and data-model correctness depend on real screen data needs**, and those needs were inferred generically from BRD FRs a second time inside `/gen-architecture`, disconnected from what a screen actually renders — risking endpoints that don't match what the UI needs.

Investigating why Screen Inventory sat inside architecture at all found the real dependency was narrower than assumed: only 3.1.3.4 and 3.1.3.5 need Section 6 (security model). Portals, Navigation Map, and Screen Inventory only need the BRD, which is already final at Epic Approval — they were inside architecture only because that's where the framework happened to put them, not because of a structural dependency.

---

## Updated Lifecycle

```
Epic Approval
     ↓
/gen-screen-design       [NEW]
     ↓
/review-screen-design    [NEW]
     ↓
Screen Design Approval   [NEW gate]
     ↓
/gen-architecture         (changed — carries over 3.1.3.1–3.1.3.3, generates 3.1.3.4–3.1.3.5 fresh)
     ↓
/review-architecture      (changed — Concern A checks 11–13 verify carry-over + patch, not fresh derivation)
     ↓
Architecture Lock         (mock patch pass completes here)
     ↓
/gen-stories ...
```

---

## New and Changed Commands

### `/gen-screen-design` (new)
- **Purpose:** Derive Portals, Navigation Map, and Screen Inventory from the BRD alone, with every screen pattern-matched against `ascendra-ui`'s real component catalog and PO-confirmed before architecture starts.
- **Gate:** Epics Approved. UI-in-scope check derived from BRD Personas/brief, not confirmed tech stack.
- **Inputs:** BRD Sections 2–5, domain knowledge, `ascendra-ui/docs/ui-reference.md`, `ascendra-ui/docs/showcase-reference.md`.
- **Output:** `projects/{PROJECT_CODE}/screens/screen-design.md`.
- **Also invokes:** `/gen-ui-mocks` in base mode for the first HTML generation pass.

### `/review-screen-design` (new)
- **Purpose:** Structured PO review of Screen Design, mirroring `review-epics.md`'s sequential-walkthrough shape.
- **Checks:** every BRD journey has a covering screen; every Surface/Pattern citation resolves to a real catalog entry or primitive, never invented; Portals are genuinely distinct navigation shells; base mocks match the table.

### `/gen-architecture` (changed)
- New gate: Screen Design Approved (when UI in scope).
- Section 3.1.3.1–3.1.3.3 carried over from `screen-design.md`, not re-derived — same treatment already given to `standards/system-architecture.md`.
- Section 3.1.3.4–3.1.3.5 generated fresh once Section 6 is written.
- Section 4 (Data Model) and Section 5 (API Contracts) cross-reference `screen-design.md`'s per-screen data requirements.
- Invokes `/gen-ui-mocks` in patch mode once Section 6 is written.

### `/gen-ui-mocks` (changed)
- **Base mode:** invoked from `/gen-screen-design` — generates full mocks from `screen-design.md`'s Surface/Pattern data, reading real `ascendra-ui` component source (not just design tokens) for shadow/gradient/hover fidelity.
- **Patch mode:** invoked from `/gen-architecture` — adds gating captions and cross-cutting banners to the existing mock files without touching layout already reviewed and approved.

### `/implement-story` (changed)
- Reads `screen-design.md` (or architecture 3.1.3 post-carry-over) for the story's screen(s) before building any Web UI — the Surface/Pattern/matched-component decision the PO already approved is what gets built.

### `/review-architecture` (changed)
- Concern A checks 11–13 change meaning: from "did the AI derive this correctly" to "was it carried over correctly from the approved `screen-design.md`, and were 3.1.3.4/3.1.3.5 added consistently."

---

## New Templates and Folders

| Item | Location |
|---|---|
| `screen-design.template.md` | `projects/TEMPLATE/screens/` |
| `screens/` folder | `projects/TEMPLATE/` and `projects/{CODE}/` |

`screen-design.md` carries its own Document Control + Status table (unlike `ui-mock.template.md`, which inherits its lifecycle from architecture) — it is now an independently gated artifact with real downstream consequences.

---

## CLAUDE.md Command Table Changes

Add:
- `/gen-screen-design` — Generate the Screen & Navigation Design from the approved BRD, pattern-matched against `ascendra-ui`
- `/review-screen-design` — Review and approve Screen Design before architecture generation begins

---

## Design Rationale

Splitting Screen & Navigation Map on its real dependency boundary (BRD-only vs. architecture-only subsections) lets screens be designed, mocked, and reviewed while architecture hasn't started — catching navigation and layout gaps at the point where they're cheapest to fix, and giving `/gen-architecture` real per-screen data requirements to design endpoints against instead of inferring them a second time from FRs alone.

Grounding Surface/UI Pattern decisions in `ascendra-ui`'s actual catalog (named example citations, real component source for shadow/hover/gradient treatment, not `globals.css` tokens alone) makes every AI-proposed screen shape checkable against a real, inspectable showcase page — consistent with how this framework already handles tech stack and architecture pattern recommendations (AI proposes with reasoning, PO confirms or redirects), rather than introducing free-form design judgment with nothing to verify it against.

Wiring `/implement-story` to read the same Surface/Pattern decision closes the gap that motivated this whole phase: without it, a PO-approved mock carries no weight over what actually gets built.

---

## Full implementation scope

See the plan recorded for this session (Screen Design phase implementation) for the complete file-by-file task list.
