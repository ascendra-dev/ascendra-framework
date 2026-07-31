# Generate UI Mocks

You are generating persistent, portal-scoped HTML mocks for an Ascendra client project, visualising the screens defined in Screen Design (and, later, patched with the gating and cross-cutting content architecture adds). This is not its own approval gate — the base pass is reviewed inside `/review-screen-design`, and the patch pass is reviewed inside `/review-architecture` Concern A.

This command runs twice, in two different modes, invoked by two different upstream commands — it is not normally run standalone by the PO.

- **Base mode** — invoked by `/gen-screen-design` right after it writes `screen-design.md`. Generates the full mock: layout, navigation, and every screen's shape, with real `ascendra-ui` component-source fidelity. No gating captions or cross-cutting banners yet — Section 6 of architecture (the security model) doesn't exist at this point.
- **Patch mode** — invoked by `/gen-architecture` once Section 6 is written. Adds Action Gating captions and Cross-Cutting UI Rules banners into the existing mock files. Does not touch layout or content already generated and reviewed in base mode.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments and determine mode

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required. E.g. `HARBORVIEW-INV-001`

If PROJECT_CODE is missing, ask:
> "Which project are these UI mocks for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

**Determine mode automatically:**

| Condition | Mode |
|-----------|------|
| No mock files exist yet in `projects/{PROJECT_CODE}/mocks/` | **Base mode** |
| Mock files exist, and architecture Section 6 (Security & Access Control) is written | **Patch mode** |
| Mock files exist, and architecture Section 6 is not yet written | Nothing to do — report: "Base mocks already exist and architecture hasn't reached Section 6 yet. Nothing to patch." Stop. |

---

## Step 1.5 — Gate check

**Base mode:**

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Screen Design exists | `projects/{PROJECT_CODE}/screens/screen-design.md` exists | "No screen design found. Run `/gen-screen-design {PROJECT_CODE}` first." |
| Design system source is reachable | `../ascendra-ui/app/globals.css` and `../ascendra-ui/ascendra-ui/components/` exist (paths relative to the framework repo root — a sibling directory of this repo, not inside `projects/`) | "Expected the design system source at `../ascendra-ui` (sibling directory of this repo) — not found. This path is currently hardcoded in this command; if your design system lives elsewhere, tell me the correct path before I continue." |

> **[Note — intentionally fixed path]** `../ascendra-ui` is the shared design-system source — a single, framework-level constant, the same for every project. This is a different concern from a project's own repo paths (see `/implement-story` Step 3, which reads those from architecture Section 3.1 once it exists): this command runs at Step 9a, before any project-specific architecture document exists, and `../ascendra-ui` is never project-specific in the first place. It is not blocked on any open design question — hardcoding it here is the correct, permanent answer, not a temporary limitation. (Patch mode has one narrow exception to this — see Step 2 Patch mode item 3 — for the case where this project's web repo has already been scaffolded by the time a later `/assess-change` gap-fill re-invokes patch mode.)

**Patch mode:**

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Architecture exists and Section 6 is written | Read the architecture document | "Architecture Section 6 (Security & Access Control) is not yet written — patch mode has nothing to read yet." |
| Base mocks exist | `projects/{PROJECT_CODE}/mocks/` has at least one file | "No base mocks found. Run `/gen-screen-design {PROJECT_CODE}` first — this generates the base mocks patch mode builds on." |

---

## Step 2 — Read required files

**Base mode:**
1. `projects/TEMPLATE/mocks/ui-mock.template.md` — the full template, including the Document Level AI Guide and Part 2 Verification checklist. Do not read `ui-mock.example.md` — examples are for PO reference only, never consumed by a generation command.
2. `projects/{PROJECT_CODE}/screens/screen-design.md` — Portals, Navigation Map, Screen Inventory (with Surface, UI Pattern, and Matched Reference per screen), and Data Requirements.
3. The approved BRD referenced by screen design — Section 4 (User Journeys) and Section 5 (Functional Requirements), for the specific fields, labels, and data points each screen should plausibly show. Never fabricate content that does not trace to a requirement or journey step here.
4. `ascendra-ui/app/globals.css` — base color tokens.
5. `ascendra-ui/docs/ui-reference.md` and `ascendra-ui/docs/showcase-reference.md` — to resolve each screen's Matched Reference to its real composition.
6. For every component named in a screen's Matched Reference, its real source file under `ascendra-ui/ascendra-ui/components/{category}/` — resolved by searching for the component name, not a hardcoded path table (a fixed list would go stale, as already seen elsewhere in this framework). Read only the components actually matched across this project's screens — not the whole library.

**Patch mode:**
1. Every existing file in `projects/{PROJECT_CODE}/mocks/`.
2. The architecture document — Section 3.1.3.4 (Action Gating) and Section 3.1.3.5 (Cross-Cutting UI Rules).
3. **Component/reference source, only when Step 3a needs it (gating banner styling):** read the confirmed web repo name from architecture Section 3.1, then check whether `../{web-repo-name}` exists on disk.
   - **Exists** (this project's web repo has already been scaffolded — e.g. a later `/assess-change` gap-fill, run after implementation has started) → read `{web-repo-name}/ascendra-ui/docs/ui-reference.md` and the matched component's source under `{web-repo-name}/ascendra-ui/components/{category}/`. This is the copy actually vendored in that repo — the one `/implement-story` builds against — so patching against anything else risks citing a component that isn't really there yet.
   - **Does not exist** (the normal case — patch mode invoked from `/gen-architecture`, before any repo has been scaffolded) → fall back to `../ascendra-ui/docs/ui-reference.md` and `../ascendra-ui/ascendra-ui/components/{category}/`, the same shared source Step 2 (base mode) uses.
   - This is a one-line existence check, not a structural validation — do not run `/implement-story`'s full bootstrap check here.

---

## Step 3 — Base mode: generate one mock file per portal

For each row in `screen-design.md` Section 1 (Portals):

1. Create `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html` (slug derived from the portal name, lowercase-hyphenated).
2. **Navigation:** one entry per row in Section 2 whose Portal matches this file, with its visibility condition shown as a visible caption. Use grouped `SideBarMenuSet`-style sections when the row's Grouping column says "grouped"; a flat list otherwise.
3. **Screens — container shape by Surface:**
   - **Page** → a persistent, full-width `<section class="screen">`, exactly as before: heading (Screen ID + route), purpose, and content. Back-navigation via a `Header`-style breadcrumb — never invent a back-button.
   - **Dialog / Sheet / Drawer** → shown **open**, as a labeled overlay-styled block positioned directly under the Page section that triggers it. Always visible for illustration, never live-toggled (same "no live toggling" principle already applied to cross-cutting notes). Label it clearly, e.g. `<div class="mock-overlay mock-overlay--sheet">Overlay: Add Payment (Sheet, right)</div>`. Close affordance is the pattern's real `showCloseButton` treatment (×), never a back arrow.
4. **Screen content shape by UI Pattern**, following the Matched Reference's real composition (from `showcase-reference.md`'s code template for that entry, or the primitive list for Report/Table-List):
   - **Form** — field groups matching the matched pattern's section count and layout (Simple/Medium/Complex, per the catalog).
   - **Dashboard** — KPI tiles + chart placeholders + a table, per the matched dashboard's KPI/chart-type list.
   - **Report** — compose from Report Header + Report Document + Report Content primitives.
   - **Table-List** — compose from Data Table + Page Bar; include an Empty State variant note in a comment if the screen's Notes column flagged one.
   - **Detail** — compose from Item / Card primitives per the matched Sheet/Dialog entry.
   - **Tabs**, where the Screen Inventory row proposes them — use the real `Tabs`/`TabList`/`TabTrigger`/`TabContent` structure.
   - **Expandable sections** — use native `<details>/<summary>` (zero-JS, real browser expand/collapse), styled from Card's real CSS for the expanded/collapsed states. Card's actual `collapseable` behavior is React state and can't be literally reproduced in a static file — this is the one legitimate substitution, not an invented shape.
   - Every field or data point shown must trace to the BRD requirement or journey step cited as that screen's Source in `screen-design.md`.
5. **Real component fidelity:** for every component named in a screen's Matched Reference, use the literal `box-shadow`, gradient, hover/focus, and radius values read from its real source file in Step 2 — not approximations, not `globals.css` tokens alone. `globals.css` supplies base color values; component source supplies the actual visual treatment.
6. **CSS size-control rules (enforced, not just guidance):**
   - Declare each component's CSS rule exactly once per file; reference it by class everywhere it's used on this portal.
   - Only include CSS for components and variants actually rendered on this portal — never the whole catalog "just in case."
   - Light mode only, unless the project has a confirmed dark-mode toggle in scope.
7. **Self-containment:** inline `<style>` only. No external stylesheet, script, font, or image request. `<details>/<summary>` satisfies the expand/collapse need with zero script.

If a delegated-access portal note exists in Section 1 (e.g. "not a separate portal — enters X under delegated access"), do not generate a separate file for it — it is already covered by the portal it enters.

---

## Step 3a — Patch mode: layer in gating and cross-cutting content

For each existing mock file:

1. **Action gating:** for any action listed in architecture Section 3.1.3.4 belonging to a screen in this file, add a visible caption naming the required role/capability next to the action — do not alter the action's existing markup otherwise.
2. **Cross-cutting rules:** if Section 3.1.3.5 defines a blanket read-only mode or a delegated-access indicator, add one visible banner per rule at the top of the mock, using `ascendra-ui`'s Simple Alert (or Unsaved Changes Bar, whichever fits) real styling — read its source per Step 2 Patch mode item 3 (the project's scaffolded web repo if it exists, `../ascendra-ui` otherwise), if not already read this session. Do not implement live toggling.
3. Do not regenerate layout, navigation, or screen content already present — this is an additive patch, not a redraw.

---

## Step 4 — Run the Verification checklist

Run every check in `ui-mock.template.md` Part 2 against every generated or patched file. Fix any failure before proceeding to Step 5 — do not report completion with a known failing check.

---

## Step 5 — Report to the user

**Base mode:**

```
UI MOCKS GENERATED — base mode
─────────────────────────────────────────
Project:       {PROJECT_CODE}
Screen Design: projects/{PROJECT_CODE}/screens/screen-design.md
─────────────────────────────────────────

FILES CREATED:
- projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html — [portal name], [N] screens

VERIFICATION: [All checks passed / N failed — list and fix before continuing]
─────────────────────────────────────────
These are reviewed inside /review-screen-design, alongside the Screen
Inventory rows they visualise. A second, additive pass runs automatically
from /gen-architecture once Section 6 is written, to add gating captions
and cross-cutting banners.
```

**Patch mode:**

```
UI MOCKS PATCHED — gating + cross-cutting content added
─────────────────────────────────────────
Project:       {PROJECT_CODE}
Architecture:  {arch-path}, Section 3.1.3.4 / 3.1.3.5
─────────────────────────────────────────

FILES PATCHED:
- projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html — [N] gating captions, [N] cross-cutting banners added

VERIFICATION: [All checks passed / N failed — list and fix before continuing]
─────────────────────────────────────────
These are reviewed inside /review-architecture Concern A, alongside the
Screen Inventory rows they visualise — not as a separate approval step.
```
