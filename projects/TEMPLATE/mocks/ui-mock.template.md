# UI Mock Template

> **[AI Guide — Document Level]**
> A UI mock is a persistent, self-contained HTML file that visualises one portal's screens. It is generated in two passes by `/gen-ui-mocks`: a **base pass**, invoked by `/gen-screen-design` right after `screen-design.md` is written — full layout, navigation, and every screen's shape, before architecture exists — and a **patch pass**, invoked by `/gen-architecture` once Section 6 (security model) is written, which adds Action Gating captions and Cross-Cutting UI Rules banners without touching the base layout. Never generated per epic — generating per epic would mean re-patching the same shared screen every time a later epic adds a field or action to it; the base pass draws from the complete, all-journeys-resolved Screen Inventory instead.
>
> **Four sources feed this document:**
> - **`screen-design.md`** (base pass) **or architecture Section 3.1.3** (patch pass, once carried over) — which portals exist, which screens belong to each, each screen's Surface (Page/Dialog/Sheet/Drawer) and UI Pattern (Form/Dashboard/Report/Table-List/Detail) with its Matched Reference, the navigation condition for each nav item, and any action-level gating. This is the structural source of truth; a mock must never show a screen, nav item, or action this data does not define, and must never render a shape other than the one its Surface/Pattern citation specifies.
> - **The approved BRD** — Section 4 (Journeys) and Section 5 (FRs) — for the specific fields, labels, and data points each screen plausibly shows. Every field or data point must trace to a requirement or journey step; never fabricated to "fill the layout."
> - **`ascendra-ui`'s real component source** (`ascendra-ui/ascendra-ui/components/{category}/`, not just `globals.css`) — for every component named in a screen's Matched Reference, its literal `box-shadow`, gradient, hover/focus, and radius treatment. This is what makes a mock's page formation actually resemble `ascendra-ui`, not just its colour palette.
> - **The design system's base design tokens** (`../ascendra-ui/app/globals.css` — a hardcoded path relative to the framework repo root, not project-configured; project-level repo path configuration is a separate, still-open design question) — colours, radii, and type scale. Read the actual token values; do not approximate or invent close-enough ones.
>
> **What this document is not:**
> - Not literally `ascendra-ui`'s component code — it does not import the real component library, and it carries no client-side JavaScript. But its CSS is not invented either: shadow, gradient, hover, and radius values are transcribed verbatim from real component source, so the *visual* result should closely match the real thing even though the *implementation* (plain HTML/CSS) differs.
> - Not a new approval gate. A mock carries no independent Status or Document Control — its base pass is reviewed inside `/review-screen-design`, alongside the Screen Inventory rows it visualises; its patch pass is reviewed inside `/review-architecture` Concern A. It Locks when the architecture document Locks.
> - Not a place to invent screen content. Every field, label, or data point shown must trace back to a BRD requirement or journey step — never fabricated to "fill the layout."
> - Not modeling Toast notifications — a static file cannot meaningfully represent a transient notification. This is a deliberate non-goal, not an oversight.
>
> **Exception — no Document Control table, no Status field (per T-011a/T-014a in `conventions/template-conventions.md`):** unlike every other generated artifact in this framework, a UI mock does not carry its own version history or lifecycle status. Its lifecycle is inherited in two stages: from `screen-design.md` (Draft while Screen Design is Draft/Under Review, stable once Approved) and then from the architecture document once the patch pass runs (frozen when architecture Locks). Recording a separate version history here would duplicate — and could drift from — the ones that already exist on those two documents.
>
> **File location:** `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html`
> e.g. `projects/ASCENDRA-PAY-001/mocks/issuer-app-mock.html`

---

## Part 1 — Template

---

> **[AI Guide]** Generate one file per Portal row in `screen-design.md` Section 1. Do not combine two portals into one file, and do not split one portal across multiple files. The skeleton below shows the fixed outer shell (sidebar nav + main content area) — it is not the shape of every screen; each screen's actual content shape comes from its Surface and UI Pattern, per the guidance below the skeleton.

**Portal:** [Portal name — from `screen-design.md` Section 1]
**Source:** `projects/{PROJECT_CODE}/screens/screen-design.md` (base pass); `projects/{PROJECT_CODE}/architecture/arch-v{N}.md` Section 3.1.3.4/3.1.3.5 (patch pass)

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<title>[Portal name] — UI Mock</title>
<style>
  /* [AI Guide] Every value below must be read from ../ascendra-ui/app/globals.css — never invented
     or approximated, and never left as the same value across projects without checking. All six are
     placeholders on purpose: baking in "usually-correct" literals here would invite copying them
     without actually reading the source file. */
  :root {
    --background: [token value];
    --foreground: [token value];
    --card: [token value];
    --border: [token value];
    --primary: [token value];
    --radius: [token value — drives --radius-sm/md/lg/xl via calc(), see globals.css];
  }
  /* [AI Guide] Declare additional component-specific CSS classes below this line — one rule per
     class, declared exactly once, only for components/variants actually rendered on this portal.
     Each rule's shadow/gradient/hover/radius values are transcribed from that component's real
     source file (e.g. ascendra-ui/ascendra-ui/components/ui/button.tsx), not invented. Light mode
     only unless this project has a confirmed dark-mode toggle in scope. */
  body { margin: 0; background: var(--background); color: var(--foreground); font-family: system-ui, sans-serif; }
  .shell { display: flex; min-height: 100vh; }
  .sidebar { width: 240px; border-right: 1px solid var(--border); padding: 1rem; }
  .main { flex: 1; padding: 1.5rem; }
  .screen { border: 1px solid var(--border); border-radius: var(--radius); padding: 1.5rem; margin-bottom: 1.5rem; }
  .mock-overlay { border: 1px solid var(--border); border-radius: var(--radius); padding: 1.5rem; margin: 0 0 1.5rem 2rem; }
  .mock-overlay::before { content: attr(data-label); display: block; font-size: 0.75rem; color: var(--foreground); opacity: 0.6; margin-bottom: 0.5rem; }
</style>
</head>
<body>
  <div class="shell">
    <nav class="sidebar">
      <!-- [AI Guide] One entry per row in screen-design.md Section 2 whose Portal matches this file.
           Render the entry's visibility condition as a visible caption (e.g. "requires: Configuration"),
           not as hidden/interactive logic — this is a static illustration, not a permission engine.
           Use grouped sections (mirroring ascendra-ui's Sidebar Menu SideBarMenuSet) when the row's
           Grouping column says "grouped"; a flat list of entries otherwise. -->
      <!-- [Nav item] — [visibility condition] -->
    </nav>
    <main class="main">
      <!-- [AI Guide] One illustration block per Screen ID in Section 3 whose Portal matches this file.
           The container shape depends on the row's Surface:

             Surface: Page   → a persistent <section class="screen">, exactly as below. Back-navigation
                               is a Header-style breadcrumb — there is no back-button primitive in
                               ascendra-ui, so never invent one.
             Surface: Dialog/Sheet/Drawer → a <div class="mock-overlay" data-label="Overlay: ..."> shown
                               OPEN, positioned directly after the Page section that triggers it — always
                               visible for illustration, never live-toggled. Close affordance is the
                               pattern's real showCloseButton (×) treatment, not a back arrow.

           Content shape inside the container depends on the row's UI Pattern and Matched Reference —
           follow that specific catalog entry's real section/field/component layout (from
           showcase-reference.md), not a generic box:

             Form         → field groups matching the matched pattern's section count and layout
             Dashboard    → KPI tiles + chart placeholders + a table, per the matched dashboard's KPIs
             Report       → Report Header + Report Document + Report Content primitives
             Table-List   → Data Table + Page Bar; note an Empty State variant if the row flagged one
             Detail       → Item / Card primitives, per the matched Sheet/Dialog entry
             Tabs (if proposed on the row) → real Tabs/TabList/TabTrigger/TabContent structure
             Expandable sections → native <details>/<summary> (zero-JS), styled from Card's real
                               expanded/collapsed CSS — Card's real collapse is React state and can't be
                               literally reproduced in a static file; this is the one legitimate
                               substitution, not an invented shape.

           Every field/data point must trace to the BRD requirement or journey step cited as that row's
           Source. If Section 3.1.3.4 (Action Gating, patch pass only) lists an action for this screen,
           render it with a visible caption naming which role/capability it requires. -->
      <section class="screen">
        <h2>[Screen ID] — [route]</h2>
        <p class="purpose">[Purpose, from screen-design.md Section 3]</p>
        <!-- [Illustrative layout content, shaped per Surface/UI Pattern above, traced to a specific BRD REQ-ID or journey step] -->
      </section>
    </main>
  </div>
</body>
</html>
```

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before reporting `/gen-ui-mocks` complete. A mock set that fails any check must be corrected before the PO reviews it (in `/review-screen-design` for the base pass, `/review-architecture` Concern A for the patch pass).

- [ ] One mock file exists for every Portal row in `screen-design.md` Section 1 — no portal missing, no extra file with no matching portal
- [ ] Every Screen ID in Section 3 whose Portal matches this file appears as an illustration block in the mock — no screen silently dropped
- [ ] Every screen's container shape matches its Surface (Page = persistent section; Dialog/Sheet/Drawer = shown-open overlay block) — no screen rendered in the wrong container
- [ ] Every screen's content shape matches its UI Pattern and Matched Reference's real composition — not a generic box regardless of pattern
- [ ] Every nav entry rendered traces to a row in Section 2 with a matching Portal — no invented nav item
- [ ] Every action rendered from architecture Section 3.1.3.4 (patch pass) shows its required role/capability as a visible caption
- [ ] Every field or data point shown traces to a specific BRD requirement or journey step — nothing fabricated to fill space
- [ ] Colour, radius, and spacing base values match the actual tokens in `../ascendra-ui/app/globals.css` — not approximated
- [ ] Shadow, gradient, hover/focus, and radius treatment for every named component is transcribed from that component's real source file — not `globals.css` alone, not invented
- [ ] No CSS rule is declared more than once in the file
- [ ] Every declared CSS rule corresponds to a component actually rendered on this portal — no unused component styles included
- [ ] The file is fully self-contained (inline `<style>`, no external stylesheet, script, font, or image requests; `<details>/<summary>` used for any expand/collapse, not a script)
- [ ] No Document Control table or Status field is present — this artifact inherits its lifecycle from `screen-design.md` and the architecture document (documented exception, see Document Level guide above)
