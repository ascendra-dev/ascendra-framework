# Generate Screen Design

You are generating the Screen Design document for an Ascendra client project — the Portals, Navigation Map, and Screen Inventory that define every screen the project needs, pattern-matched against `ascendra-ui`'s real component catalog. This runs after Epic Approval and before `/gen-architecture`, because none of this content depends on a confirmed tech stack — it comes entirely from the approved BRD.

Every screen is assigned a Surface (Page/Dialog/Sheet/Drawer) and a UI Pattern (Form/Dashboard/Report/Table-List/Detail), each cited to a specific named example in `ascendra-ui/docs/ui-reference.md` or, for Report/Table-List, a primitive composition — never an invented shape. `/gen-architecture` later carries Portals, Navigation Map, and Screen Inventory forward as-is; `/implement-story` reads the same Surface/Pattern decision before building any Web UI, so the mock the PO reviews here is what actually gets built.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required. E.g. `HARBORVIEW-INV-001`
- **BRD path** — optional. If not provided, use the most recent `brd-core-v*.md` in `projects/{PROJECT_CODE}/brds/`.

If PROJECT_CODE is missing, ask:
> "Which project is this screen design for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Derive the following paths automatically:
- Brief: `projects/{PROJECT_CODE}/brief.md`
- Epics index: `projects/{PROJECT_CODE}/epics/index.md`
- Domain knowledge: `projects/{PROJECT_CODE}/domain/`

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| BRD exists and is Approved | Read BRD status header — must be `Approved` or beyond | "The BRD is not yet Approved. The Product Owner must approve the BRD before screen design can be generated." |
| Epics index exists | `projects/{PROJECT_CODE}/epics/index.md` must exist | "No epics index found. Generate epics first using `/gen-epics {brd-path}`." |
| All epics Approved | Every epic in the index must have status `Approved` or beyond | "The following epics are not yet Approved: [list]. Run `/review-epics` and then `/update-status` for each epic." |

**UI-in-scope check:** read the BRD's Section 2 (Personas) and the brief. If every persona interacts with the system only through an external channel with no login (e.g. an emailed payment link, an SMS notification) and no persona uses a browser/app session, this is a pure API/backend project. Stop and report:

> "This project has no UI in scope (BRD Section 2 shows no persona with a logged-in session). Screen Design is not applicable — proceed directly to `/gen-architecture`."

Otherwise, proceed.

---

## Step 2 — Read required files

1. `projects/TEMPLATE/screens/screen-design.template.md` — the full template, including the Document Level AI Guide and Part 2 Verification checklist.
2. The approved BRD (path from Step 1) — Section 2 (Personas), Section 3 (User Roles), Section 4 (User Journeys), Section 5 (Functional Requirements).
3. Domain knowledge files referenced by the BRD, for entity/field vocabulary.
4. `ascendra-ui/docs/ui-reference.md` — Part 1 (primitives) and Part 2 (composite patterns: Forms, Dialogs, Sheets, Drawers, Dashboards) in full.
5. `ascendra-ui/docs/showcase-reference.md` — for the specific code templates behind whichever Part 2 patterns get matched in Step 4.

Do not read architecture standards or any file under `projects/{PROJECT_CODE}/standards/` — none of this document depends on the confirmed tech stack.

---

## Step 3 — Derive Portals, Navigation Map, and Screen Inventory

Following `screen-design.template.md`'s AI Guide exactly:

1. **Portals (Section 1):** group BRD personas into portals by shared navigation shell, not one row per persona. If BRD Section 6.3 (Delegated Access) is populated, note the delegated session as entering an existing portal.
2. **Navigation Map (Section 2):** one row per nav item with its visibility condition, derived from BRD Section 3 (roles/capabilities) and Section 5 (feature areas). Note whether each portal's item set needs grouping (multiple distinct feature areas) or stays flat.
3. **Screen Inventory (Section 3):** one row per journey step or FR with a "persona can see/do X" shape, derived from BRD Section 4 and Section 5. A journey step with no covering screen is a gap to close now.

---

## Step 4 — Pattern-match every screen

For every Screen Inventory row, in order:

1. **Extract signals** from the row's source journey step/FR: the action verb (create/edit/view/approve/browse), whether the language says "without leaving the page" / "quickly" / "inline" (signals an overlay), how many related sub-views are shown together, and roughly how much data the screen holds.
2. **Assign Surface** (Page / Dialog / Sheet / Drawer): "without leaving the page" language → Dialog (compact, single action) or Sheet (more content, slide-out) depending on data volume; a genuinely separate destination → Page. Back-navigation: Page surfaces use `ascendra-ui`'s `Header` breadcrumbs (there is no dedicated back-button primitive in the catalog); Dialog/Sheet/Drawer use their documented `showCloseButton` (×) instead.
3. **Assign UI Pattern and Matched Reference**, citing a specific real catalog entry — never a bare category label:
   - **Form** → match `ui-reference.md`'s Forms table (`Domain | Complexity | Layout`) to the nearest named pattern.
   - **Dialog/Sheet/Drawer content** → match the corresponding table's `Type | Components` (Sheets/Drawers also have `Domain`).
   - **Dashboard** → match `Domain | Chart types | KPIs`.
   - **Report or Table-List** → **no Part 2 composite example exists for these two.** Cite a primitive composition instead — e.g. "Report Header + Report Document + Report Content" or "Data Table + Empty State + Page Bar."
4. **Tabs:** propose only when the source journey/FR describes 2+ independent sub-views reachable from one entry point with no route change. Layer onto whatever Surface/Pattern was already chosen — never a default.
5. **Empty State:** note in the row's Notes column when a List/Table-List/Dashboard screen plausibly starts with zero records (new tenant, first-run) — this is a state, not a separate screen type.
6. **Command Palette and Toast:** never matched per screen. If the BRD signals a power-user/admin role that would benefit from a command palette, note it as a candidate for architecture's future Cross-Cutting UI Rules (3.1.3.5) — do not add it here. Toast is not modeled at all (a static mock cannot represent a transient notification).

A row with a UI Pattern but no Matched Reference, or a Matched Reference that is not a real, verifiable catalog entry or primitive set, is a defect — correct it before proceeding to Step 5.

---

## Step 5 — Record per-screen data requirements

For every screen, list the fields it shows or collects and the actions it exposes (Section 4 of the template). This becomes `/gen-architecture`'s direct input for Data Model and API Contracts generation — every field here should trace to a BRD entity or business rule; every screen action here should later correspond to an endpoint.

---

## Step 6 — PO confirmation

Present the full Screen Inventory table — Screen ID, Portal, Route, Surface, UI Pattern, Matched Reference, and the one-line reason for each — plus the Portals and Navigation Map tables.

> "Here is the proposed screen design: [N] portals, [N] screens. Does this breakdown, and each screen's Surface/UI Pattern match, look right? Flag anything that should be a different shape, or any screen that's missing."

Wait for PO response. Do not write any files until confirmed. Apply any requested changes and re-present before proceeding.

---

## Step 7 — Write the document

Write `projects/{PROJECT_CODE}/screens/screen-design.md` following `screen-design.template.md` exactly. Set Status to `Draft`.

Run the template's Part 2 Verification checklist. Fix any failure before proceeding — do not write a document with a known failing check.

---

## Step 8 — Generate base mocks

Invoke `/gen-ui-mocks {PROJECT_CODE}` in base mode (reading the `screen-design.md` just written). Base-mode mocks show layout, navigation, and screen shape using real `ascendra-ui` component-source fidelity — they do not yet include Action Gating captions or Cross-Cutting UI Rules banners, since those depend on architecture Section 6, which does not exist yet.

---

## Step 9 — Report to the user

```
SCREEN DESIGN GENERATED
─────────────────────────────────────────
Project:      {PROJECT_CODE}
BRD source:   {brd-path}
─────────────────────────────────────────

PORTALS:      [N]
SCREENS:      [N] ([N] Page, [N] Dialog, [N] Sheet, [N] Drawer)

FILES CREATED:
- projects/{PROJECT_CODE}/screens/screen-design.md
- projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html — [portal name], [N] screens

VERIFICATION: [All checks passed / N failed — list and fix before continuing]
─────────────────────────────────────────
Status: Draft. Run /review-screen-design {PROJECT_CODE} next — Screen Design
must reach Approved before /gen-architecture can run.
```
