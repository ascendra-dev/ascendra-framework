# Ascendra UI — Hard Instructions

## Purpose

A living list of corrections to how `ascendra-ui` is actually used in practice — found by
PO manual review of implemented screens, or by direct investigation against `../ascendra-ui`'s
real component source and real page implementations (not just its own prose docs).

**This file supplements, never replaces, `ascendra-ui`'s own documentation.** Any command
building UI against `ascendra-ui` (currently `/implement-story`; potentially `/gen-ui-mocks`
in future) reads `ascendra-ui/docs/ui-reference.md` and `ascendra-ui/docs/showcase-reference.md`
(or the project's own `standards/ui-reference.md` if it maintains one) exactly as it always
has. This file is read *in addition to* those, as the last thing checked before writing UI
code — because in practice those docs are sometimes incomplete, or (rarer, but confirmed —
see AUI-003, AUI-004) actually wrong about what the real component library does. The
underlying condition is unchanged: `ascendra-ui` is used because the project's confirmed
tech stack (`system-architecture.md` Section 2) selected Next.js + Ascendra UI — this file
does not change that decision or apply when a different frontend is confirmed.

**This file is not a UI style guide and does not replace exploring `../ascendra-ui` directly.**
When a pattern isn't covered below, the fallback is still: read the docs, then check
`../ascendra-ui/components/{forms,dialogs,sheets,drawers}/` and `../ascendra-ui/app/showcase/`
for the closest real page and copy its actual structure — never invent a structure freehand.

---

## How to add an entry

Append a new entry below with the next `AUI-NNN` ID, today's date, and:

- **Observed gap:** what was actually built (with a file:line or story reference if known)
- **Correct pattern:** what `ascendra-ui` actually provides — name the component(s)
- **Rule:** the instruction to follow going forward, stated so it can be checked against a diff
- **Reference:** the real file(s) in `../ascendra-ui` that prove the correct pattern (a
  showcase page, a real form/dialog implementation, or the component's own source) — never
  cite `docs/*.md` alone as the reference, since these entries exist precisely for cases where
  the docs and the real implementation disagree

Never delete or renumber an entry. If a later entry corrects or narrows an earlier one, mark
the earlier one `Superseded by AUI-NNN` rather than removing it.

---

## Entries

### AUI-001 — Section headers inside a form/dialog use `FieldLegend`, never a raw heading tag

**Date:** 2026-07-18

**Observed gap:** `app/signup/page.tsx` (US-02-001) uses raw `<h3 className="text-muted-foreground mb-3 text-[0.8125rem] font-medium tracking-wide uppercase">` for its two section headers ("Business details", "Your details (Owner)").

**Correct pattern:** `FieldLegend` (from `field.tsx`) — `<FieldLegend>Section Title</FieldLegend>` for a full section header, `<FieldLegend variant="label">Sub-label</FieldLegend>` for a smaller inline label. Confirmed: every real form page in `../ascendra-ui/components/forms/*.tsx` uses `FieldLegend` for this; none use a raw heading element.

**Rule:** Any section header inside a `Card`/`CardPanel`/dialog form body must use `FieldLegend`, styled via its own `variant` prop, never a hand-rolled heading element with manual Tailwind classes.

**Reference:** `../ascendra-ui/components/forms/contact-inquiry-form.tsx` (`<FieldLegend>Additional Information</FieldLegend>`); `../ascendra-ui/ascendra-ui/components/ui/field.tsx` (component definition).

---

### AUI-002 — Field error/hint display uses `FieldHint`, not `FieldError`

**Date:** 2026-07-18

**Observed gap:** `app/signup/page.tsx` and `components/staff/add-staff-dialog.tsx` (US-01-003) both use `<FieldError errors={errors.x ? [errors.x] : []} />` under each input.

**Correct pattern:** `FieldHint` — `<FieldHint error={errors.x as {message?: string}} mandatory />` (or `optional`, or with no props at all for a field with neither state). `FieldError` is exported from `field.tsx` and is a valid component, but it is not the convention actually followed anywhere: `grep -rl FieldError ../ascendra-ui/components/forms/` returns zero files; `grep -rl FieldHint` returns all ten.

**Rule:** Every `Field`'s error/hint row must use `FieldHint`, passing its `error` prop (a single `{message?}` object, not an array) and `mandatory`/`optional`/`help` as applicable — never `FieldError`.

**Reference:** `../ascendra-ui/components/forms/contact-inquiry-form.tsx` lines ~150–153. **See `reference/ascendra-ui/field-hint-guide.md` for the component's full prop reference, its space-reservation behavior, and the `mandatory`/`optional` badge-density authoring guidance** — `FieldHint` has more surface area than this entry alone covers.

---

### AUI-003 — `Field` does not need a `FieldContent` wrapper

**Date:** 2026-07-18

**Observed gap:** `app/signup/page.tsx` and both staff dialogs wrap every field's control and error as `<Field><FieldLabel/><FieldContent><Input/><FieldError/></FieldContent></Field>`.

**Correct pattern:** `Field > FieldLabel + Control + FieldHint`, as direct siblings — no `FieldContent` wrapper. Confirmed: `grep -rl FieldContent ../ascendra-ui/components/` returns exactly one file, `previews/field-preview.tsx` (the isolated component-preview page used only to document the `Field` primitive itself) — it is not used in any real form, dialog, sheet, or drawer page.

**Note — the docs disagree with the real pages here:** `ascendra-ui/docs/showcase-reference.md`'s own "Template 2 — Settings/Form Page" and "Template 5 — Dialog Patterns" code samples both show `FieldContent` wrapping the control. When a docs template and the real page implementations under `../ascendra-ui/components/` disagree, follow the real pages — the docs prose can drift from the library; the component source and real usage cannot.

**Rule:** Never wrap a `Field`'s control in `FieldContent`. Put the control and `FieldHint` directly inside `Field`.

**Reference:** `../ascendra-ui/components/forms/contact-inquiry-form.tsx`; `../ascendra-ui/components/dialogs/invite-member-dialog.tsx`.

---

### AUI-004 — `DialogContent` has no `showCloseButton` prop

**Date:** 2026-07-18

**Observed gap:** `components/staff/add-staff-dialog.tsx` (US-01-003) passed `<DialogContent showCloseButton>` — this is a real TypeScript build error (`Property 'showCloseButton' does not exist`), caught only because `next build` was run before committing.

**Correct pattern:** `DialogContent` takes no such prop — confirmed against `../ascendra-ui/ascendra-ui/components/ui/dialog.tsx`'s actual source. `docs/ui-reference.md`'s own Dialog prop table lists `showCloseButton` — that documented prop does not exist on the real component.

**Rule:** Never pass `showCloseButton` (or any prop not visible in the component's actual source at `../ascendra-ui/ascendra-ui/components/`) — when a docs prop table is the only source for an unfamiliar prop, verify it against the real component source before using it, since this specific prop was already found to be fictional.

**Reference:** `../ascendra-ui/ascendra-ui/components/ui/dialog.tsx`.

---

### AUI-005 — A Data Table page's primary action button goes in `DataTableBarAction`, not `PageHeaderAction`

**Date:** 2026-07-18

**Observed gap:** `app/(issuer-app)/staff/page.tsx` (US-01-003) places the "Add Staff" dialog trigger inside `<PageHeader><PageHeaderAction>`.

**Correct pattern:** `<DataTableBar><DataTableBarContent>...(search/filters)...</DataTableBarContent><DataTableBarAction><Button>+ Add X</Button></DataTableBarAction></DataTableBar>`, placed above the table itself. Confirmed by grepping every real usage of `PageHeaderAction` across the whole showcase (`components/` + `app/showcase/`): it appears only in dashboard pages (date-range pickers, export actions) — never on a page whose main content is a table/list.

**Rule:** On any screen whose primary content is a `DataTable`/`DataTableWithQueryProvider`, the "Add/Create" action lives in that table's own `DataTableBarAction`, not the page header.

**Reference:** `../ascendra-ui/app/showcase/data-table-lab/page.tsx` (`<DataTableBarAction><Button>+ Add Invoice</Button></DataTableBarAction>`).

---

### AUI-006 — Explore the full Data Table feature surface before building a listing screen; use only what the requirements need

**Date:** 2026-07-18

**Observed gap:** N/A — not yet a defect, but a process gap worth stating before it becomes one: `/implement-story` has so far only ever built the plain `DataTableProvider` + search-input shape (the "Static Data Table" template), without checking whether a story's actual requirements call for more of what `ascendra-ui`'s Data Table system offers.

**Correct pattern:** The Data Table system supports — among other things — sorting, select-all/bulk row actions, per-row action menus, search (including fuzzy search via `DataTableHighlight`), column filtering, column reordering and visibility, saved column preferences, a query provider supporting multiple named/preset queries with their own param forms ("My Queries", saved/runnable filters), and server-driven pagination across batches. `../ascendra-ui/app/showcase/data-table-lab/page.tsx` is the single fullest real reference exercising most of this at once; `docs/showcase-reference.md`'s "DataTable System" section documents the underlying API (`ColumnDef`, `QueryFn`, `QueryDef`, `FieldOptionsMap`).

**Rule:** Before building any Table-List screen, read the "DataTable System" section of `docs/showcase-reference.md` and open `data-table-lab/page.tsx` — not to copy every feature, but to make a deliberate choice of which features the story's ACs actually call for. A compact, search-only table is a legitimate choice when that's genuinely all a screen needs (e.g. a short staff roster) — it should never be the *default* arrived at without having checked what else was available.

**Reference:** `../ascendra-ui/app/showcase/data-table-lab/page.tsx`; `ascendra-ui/docs/showcase-reference.md` "DataTable System" section.

---

### AUI-007 — Copy the structural shape of the closest real reference page; don't hand-tune spacing

**Date:** 2026-07-18

**Observed gap:** PO review of the signup page and both staff dialogs described them as visually "rough — no proper spacing" compared to real `ascendra-ui` pages, despite using the right primitive components.

**Correct pattern:** Every real form/dialog/sheet/drawer page's spacing (gaps in `FieldGrid`/`FieldSet`/`FieldGroup`, `CardPanelItem` grouping, dialog `DialogBody`/`DialogFooter` padding) comes from the primitives' own built-in class names, not page-level utility classes layered on top. When a page looks "off," the usual cause is a hand-added spacing class fighting the primitive's own default, or a structural piece (e.g. `FieldSet`/`FieldGrid` around a set of related fields) being skipped rather than nested correctly.

**Rule:** For any new form or dialog, find the closest matching real page under `../ascendra-ui/components/{forms,dialogs,sheets,drawers}/` and copy its structural nesting exactly (which wrapper contains which, in what order) — change only the field content, labels, and validation. Do not add spacing/layout utility classes not already present in the reference page's own JSX.

**Reference:** `../ascendra-ui/components/forms/` and `../ascendra-ui/components/dialogs/` (pick the closest match by field count/complexity, same method `screen-design.md`'s "Matched Reference" column already uses at the design stage).

---

### AUI-008 — The real application shell has two top bars (`Header`, `Nav`) in addition to the sidebar; `Header` is never optional, `Nav` is situational

**Date:** 2026-07-18

**Observed gap:** `app/(issuer-app)/layout.tsx` (US-01-003 — the first story to reach this portal) has no `Header` and no `Nav` at all — just `SideBarOverlay > MainContainer(SideBarToggle + SideBar + ContentArea)`.

**Correct pattern:** The real full shell (`../ascendra-ui/app/showcase/layout.tsx`, the library's own dogfooded layout) is:
```
PageLayout
├── SideBarOverlay
├── Header            ← HeaderLinks (brand/breadcrumb) + HeaderActions (ThemeToggle, NameAvatar, sign-out, etc.)
├── Nav               ← optional: NavLink[] secondary sticky tab-strip, top-level section switcher
└── MainContainer
    ├── SideBarToggle
    ├── SideBar > SideBarMain > SideBarMenuSet[...]
    └── ContentArea
```

**Rule:**
- **`Header` is required on every real app shell, never treated as optional scaffolding** — even a "first screen in this portal" story must include at least a minimal `Header` (brand name via `HeaderLinks`/`HeaderLink`, a `HeaderActions` slot for at least a sign-out affordance). It is not blocked by an unbuilt login screen — a sign-out button can call the auth client's `signOut()` directly regardless of whether a dedicated login page exists yet.
- **`Nav` is situational, not a peer requirement to `Header`.** Only add it when the architecture's Navigation Map or a screen's own design actually calls for top-level section-switching or page-level tabs (see `docs/showcase-reference.md` Template 1's "with secondary nav tabs" variant, itself a different, page-level use of the same `Nav` component). Its absence is a legitimate scope decision, not a gap, when nothing in the architecture calls for it — check the Navigation Map (`arch-v1.md` Section 3.1.3.2) before adding one speculatively.

**Reference:** `../ascendra-ui/app/showcase/layout.tsx`.

---

### AUI-009 — A flat, non-collapsible sidebar group uses `SideBarMenuItemGroup` directly; only a real collapsible group uses `SideBarMenu` + `SideBarMenuHeader` + `SideBarMenuContent`

**Date:** 2026-07-18

**Observed gap:** `app/(issuer-app)/layout.tsx` built the Staff nav group as `SideBarMenu basePath="/app" > SideBarMenuSet > SideBarMenuContent > SideBarMenuItem` — a collapsible-accordion composition — for a group that is a single flat item with no sub-items and no need to collapse.

**Correct pattern:** Two distinct, real compositions exist and are not interchangeable:
- **Collapsible group** (has its own expand/collapse state, toggled by a clickable header): `SideBarMenuSet > SideBarMenu(basePath) > SideBarMenuHeader(icon) + SideBarMenuContent > SideBarMenuItem[]`. `SideBarMenuHeader` is the required clickable toggle (chevron icon, click handler) — `SideBarMenuContent`'s visibility CSS (`group-data-[open=true]`) depends on `SideBarMenu`'s own `data-open` state, which `SideBarMenuHeader`'s click handler (or an active-route match) sets.
- **Flat, always-visible group** (no collapse behavior at all): `SideBarMenuSet > SideBarMenuSetTitle + SideBarMenuItemGroup > SideBarMenuItem[]` — no `SideBarMenu`, no `basePath`, no `SideBarMenuHeader`, no `SideBarMenuContent`. Confirmed real usage: the "Samples" section's Dialog/Sheet/Drawer Gallery group, and the ungrouped Tabs/Sidebar Menu pair, in `../ascendra-ui/app/showcase/layout.tsx`.

Using the collapsible composition without its required `SideBarMenuHeader` (as built here) happens to still render — because a `basePath` matching every route in the portal forces it permanently "open" — but it's the wrong component pair for what's actually displayed, not a cosmetic variant of the right one.

**Rule:** Before adding a sidebar group, check whether it needs real expand/collapse behavior (multiple items under a toggleable header). If it's a single item or a short always-visible list — matching a group in the approved mock/Navigation Map that shows no collapse affordance — use `SideBarMenuItemGroup` directly, never `SideBarMenu`/`SideBarMenuContent` without its required `SideBarMenuHeader`.

**Reference:** `../ascendra-ui/app/showcase/layout.tsx`; `../ascendra-ui/ascendra-ui/components/side-bar/side-bar-menu-header.tsx`, `side-bar-menu-item-group.tsx`.

---

### AUI-010 — `FieldHint` ignores `children`; pass hint text via the `description` prop

**Date:** 2026-07-18

**Observed gap:** `app/signup/page.tsx` used `<FieldHint>Only Pakistan is supported in v1 — any other selection is rejected.</FieldHint>` and `<FieldHint>You'll be seeded with all four capabilities...</FieldHint>` — passing the hint text as JSX children. Reading `field.tsx`'s actual implementation: `FieldHint`'s parameters are `{ error, description, mandatory, optional, help, className, ...props }`, and its body never reads `children` — the component always renders its own fixed internal JSX (an error-or-description row, plus optional mandatory/optional/help badges) and guards with `if (!error && !description && !mandatory && !optional && !help) return null;`. Passing only children (no `description`) means every one of those checks is falsy — **the component returns `null` and the hint text does not render at all.** This was a real, silent rendering bug, not just a convention mismatch — found only by reading the component's source, not by looking at the page.

**Correct pattern:** `<FieldHint description="Only Pakistan is supported in v1 — any other selection is rejected." />`. Confirmed real usage: `../ascendra-ui/components/forms/create-product-form.tsx`, `user-profile-form.tsx` — every non-error hint uses `description=`, never children.

**Rule:** `FieldHint` never takes children. Pure informational hint text uses `description="..."`; a validation error uses `error={fieldState.error}`; `mandatory`/`optional`/`help` are separate boolean/string props layered on top of either. Never pass hint text as JSX children — it will silently not render.

**Reference:** `../ascendra-ui/ascendra-ui/components/ui/field.tsx` (`FieldHint`'s actual body); `../ascendra-ui/components/forms/create-product-form.tsx`. **Full component reference:** `reference/ascendra-ui/field-hint-guide.md`.

---

### AUI-011 — `FieldLegend` is followed immediately by a `FieldDescription` explaining the section

**Date:** 2026-07-18

**Observed gap:** `app/signup/page.tsx` (US-02-001) used `FieldLegend` alone for "Business details" and "Your details (Owner)" — no description sentence explaining what each section is for or why it matters.

**Correct pattern:** A section header is `FieldLegend` immediately followed by a `FieldDescription` — one to two sentences giving the section context, before the fields themselves. Confirmed real usage: `../ascendra-ui/components/forms/contact-inquiry-form.tsx` line ~366 (`<FieldLegend>Additional Information</FieldLegend>` directly followed by `<FieldDescription>Please provide any additional information that may help us better assist you.</FieldDescription>`). `FieldDescription`'s own CSS even has a rule specifically for this adjacency (`[[data-variant=legend]+&]:-mt-1.5` — a tighter top margin when it directly follows a legend).

**Rule:** Every `FieldLegend` used as a section header should be immediately followed by a `FieldDescription` stating what the section covers or why it matters — not left to stand alone. Compose the sentence from the story/BRD's own domain language for that section (what the fields are used for, what happens as a result of filling them in) rather than generic filler text.

**Reference:** `../ascendra-ui/components/forms/contact-inquiry-form.tsx`; `../ascendra-ui/ascendra-ui/components/ui/field.tsx` (`FieldDescription`'s adjacency CSS rule).

---

### AUI-012 — A form's success/error banner uses `SimpleAlert`, not a hand-rolled `<p>`

**Date:** 2026-07-22

**Observed gap:** `app/signup/page.tsx` (US-02-001) renders its mutation-error state as
`<p className="text-destructive text-sm" role="alert">{...}</p>` — a hand-rolled element, not a
real component.

**Correct pattern:** `SimpleAlert` (`common-ui/simple-alert.tsx`) — `variant="destructive"` for an
error, `variant="success"` for a success confirmation. Confirmed real usage:
`components/verification/document-upload-form.tsx` (`<SimpleAlert variant={...}>...</SimpleAlert>`
for a status banner above a form).

**Rule:** Any form-level success or error banner (not a per-field `FieldHint`) uses `SimpleAlert`
with the matching `variant`, never a hand-styled `<p>`/`<div>` with ad hoc Tailwind classes.

**Reference:** `components/verification/document-upload-form.tsx`; `ascendra-ui/components/common-ui/simple-alert.tsx` (component definition).

---

### AUI-013 — Multi-panel dashboard/report rows use a 12-column grid, not an ad hoc `grid-cols-2`

**Date:** 2026-09-23

**Observed gap:** `ascendra-commons-ui`'s Audit Log Overview dashboard laid out its chart/table rows on
whatever column split seemed reasonable per-row, with no consistent grid unit — confirmed by PO
review against `ascendra-ui/app/showcase/dashboards/`, where every one of the 10 real sample
dashboards uses the same underlying grid.

**Correct pattern:** `<div className="grid grid-cols-12 gap-4">` as the row wrapper, with each panel
as `<div className="col-span-12 md:col-span-N">` — `N=8`/`4` for a 2:1 split (e.g. a primary chart
beside a secondary donut/gauge), `N=7`/`5` for a table beside a chart. `col-span-12` alone (full
width, no split) is the correct choice for a row with one panel — not a reason to fall back to a
plain `grid-cols-2`.

**Rule:** Any dashboard or report row with more than one panel uses `grid-cols-12` + `col-span-12
md:col-span-N`, never a plain `grid-cols-2`/`grid-cols-3` wrapper, so column proportions stay
expressible in twelfths and consistent with every other real dashboard.

**Reference:** `ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx` (lines ~332, ~496, ~655
show the 8/4, 8/4, and 7/5 splits respectively).

---

### AUI-014 — A KPI tile showing a period-bound metric pairs its value with a `SimpleBadge` trend/delta, never a bare value

**Date:** 2026-09-23

**Observed gap:** The Audit Log Overview's KPI tiles (`packages/audit-logging.ui`) rendered a label
and a value only — no comparison to the metric's prior equivalent period — despite the screen's own
mock (`audit-logging.api/mocks.html`) specifying a delta on at least one tile.

**Correct pattern:** `Card > CardPanel > <div className="flex flex-1 flex-col p-5">`, with the label
pinned to the top and a `<div className="mt-auto flex ... items-center justify-between">` row at the
bottom holding the formatted value plus `<SimpleBadge variant={up ? "green" : "red"}>` wrapping a
`LuTrendingUp`/`LuTrendingDown` icon and the delta text. The `mt-auto` row is what keeps the value+
badge pinned to a consistent baseline across tiles of slightly different content height.

**Rule:** Any KPI tile bound to a specific time window (today, 7d, 30d, …) shows a `SimpleBadge`
trend/delta against that same window's prior equivalent period — never a bare value with no
comparison. State explicitly, in a small caption or the tile's label, which prior period the delta
is against when a screen's tiles don't all share the same window (e.g. one tile vs. yesterday,
another vs. prior 30 days) — don't leave the comparison basis ambiguous.

**Reference:** `ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx` lines 306-329;
`financial-pnl/page.tsx`'s own KPI row.

---

### AUI-015 — A panel's header is one of three deliberate choices, keyed to whether it needs explanation — never applied uniformly

**Date:** 2026-09-23

**Observed gap:** Every panel on the Audit Log Overview dashboard used the same full `CardHeader`
treatment regardless of content, rather than choosing per-panel the way the real dashboards do.

**Correct pattern:** Three real, coexisting patterns, chosen by what the panel actually needs:
1. **Full `CardHeader`** (`CardHeaderTitle` + `CardHeaderSubtitle`) — for a chart/table whose axes,
   window, or ranking need a sentence of explanation.
2. **Inline title only, no `CardHeader`** — `<p className="text-sm font-medium">Title</p>` directly
   inside the `CardPanel`'s padded wrapper — for a compact, self-explanatory single-metric panel.
3. **Bare `CardHeader` with no enclosing `Card`**, sitting directly above a `TableWrapper` — for a
   full-bleed table section that doesn't need a card's padding/background around it.

**Rule:** Pick the header pattern per panel based on whether it needs an explanatory subtitle, not
by copying whichever pattern the previous panel on the same page used.

**Reference:** `ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx` — all three appear in one
file: "MRR & Growth Rate" (pattern 1), "Plan Mix" (pattern 2), "Top Accounts" (pattern 3). A second
confirmed pattern-3 example: `dashboards/marketing/page.tsx`'s "Active Campaigns" table — full-width
`col-span-12` row, bare `CardHeader` directly above `TableWrapper`, `<Table scrollable horizontal
vertical height={300}>`, and a trailing empty `<CardFooter className="border-t-0 pt-0" />` inside
`TableWrapper` closing it out.

---

### AUI-016 — A screen's table structure must mirror the mock's/API's actual shape — don't split or merge tables without a stated reason

**Date:** 2026-09-23

**Observed gap:** The Audit Log Overview dashboard rendered two separate tables, "Top actions" and
"Top actors" — the approved mock (`audit-logging.api/mocks.html`) has one table, "Top actions",
with an `Actors` column folded in.

**Correct pattern:** Read the mock/API response shape the screen is built from and match its table
boundaries exactly — one combinable table stays one table (see also AUI-006's identical principle
applied to filter design, not table layout). Splitting or merging is a legitimate choice only when
the mock/API genuinely models the data as separate resources, not as a default.

**Rule:** Before rendering more or fewer tables than a screen's source-of-truth mock/API shows,
check whether that's actually what the source models — don't invent a friendlier-seeming split
(or an unwarranted merge) without a stated reason recorded next to the code.

**Reference:** the specific `<module>.api/mocks.html` backing whichever screen is being built.

---

### AUI-017 — Fixture/mock data backing a dashboard must be realistic-scale, not toy-sized — an undersized fixture is a real, visible defect

**Date:** 2026-09-23

**Observed gap:** `audit-logging.ui`'s mock fixture (`mocks/audit-events.mock.ts`) had 12 rows total,
aggregated directly into the Overview dashboard's stats. Per-day counts of 1-3 made Recharts' own
tick algorithm pick decimal Y-axis steps (0, 0.5, 1, 1.5…), and KPI tiles read "Records today: 1" —
both symptoms of fixture size, not a chart or KPI-logic bug (same root-cause class as this file's own
AUI-007 finding about an undersized batch size hiding correct pagination behavior).

**Correct pattern:** A dashboard's stats should come from data generated at a realistic order of
magnitude for the domain (hundreds-to-thousands per day for a busy audit/event log, not single
digits), deterministically (a seeded PRNG, not raw `Math.random()`) so the fixture is stable across
runs. Pair this with `allowDecimals={false}` on any Recharts `YAxis` showing count data, as a
defensive backstop regardless of data scale. Format large values with `toLocaleString()` (thousands
separator) below a project-chosen threshold, and compact `K`/`M` notation above it — via a small
page-local helper, since no shared compact-number formatter exists in `ascendra-ui` today (every
real dashboard hand-rolls its own, e.g. `fmtMrr` in `saas-revenue/page.tsx`).

**Rule:** Treat a fixture that can't produce realistic-looking chart ticks or KPI values as a defect
to fix at the fixture, not something to patch around in the chart/component code. Generate
dashboard-scale fixture data separately from small illustrative row-level fixtures (e.g. the handful
of rows a list/detail screen needs for click-through demos) when the two need different scales —
they don't have to share one dataset.

**Reference:** `ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx`'s `fmtMrr` helper and its
`YAxis tickFormatter` usage.

---

### AUI-018 — User-facing labels and subtitles spell out time windows in prose — never a raw API param abbreviation

**Date:** 2026-09-23

**Observed gap:** The Audit Log Overview dashboard's KPI labels and chart subtitle used raw internal
window params directly as UI copy — "Records (7d)", "Volume by day (30d)" — interpolating the same
string the API query param uses.

**Correct pattern:** Every real dashboard's `CardHeaderSubtitle` spells the window out in full prose
— "12-month trend", "Total revenue by month · in millions USD", "Past 8 accounts ranked by monthly
recurring revenue" — never a bare "12mo" or "30d" lifted from a query param.

**Rule:** Translate a window/param value into plain English before it reaches a label, title, or
subtitle a user reads. Reserve short-form abbreviations (if any) for space-constrained contexts only
— a chart axis tick label, never a heading, subtitle, or KPI tile label.

**Reference:** `ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx` and
`ecommerce-ops/page.tsx` — every `CardHeaderSubtitle` in both files is full prose.

---

### AUI-019 — `DataTableLoadingBody`/`DataTableErrorBody`/`DataTableEmptyBody` work standalone now — don't adopt the full DataTable system just to get their visuals

**Date:** 2026-09-23

**Observed gap:** These three components originally required context (`useDataTableData()`,
`useOptionalQueryContext()`) that only `DataTableProvider`/`DataTableQueryProvider` supply — so a
screen with a simple, manually-managed `useQuery` table (no sorting/filtering/pagination need, per
AUI-006's own "a compact table is a legitimate choice" guidance) had exactly two options: hand-roll
equivalent `Empty`/`EmptyHeader`/`EmptyMedia` markup from scratch (real risk of drifting from the
real components — found happening in `ascendra-commons-ui`'s Audit Log Overview dashboard, which
had hand-rolled a loading block before this fix landed), or adopt the entire DataTable system purely
to unlock three display components it didn't otherwise need.

**Correct pattern:** All three now accept explicit prop overrides that take priority over context
when given — `isLoading` (Loading/Empty), `isEmpty` (Empty), `isError`/`error`/`onRetry` (Error) —
falling back to an optional provider context otherwise, the same pattern `DataTableProvider` itself
already used for its own `data`/`isLoading` via `useOptionalQueryContext`. A new
`useOptionalDataTableData()` hook sits alongside the existing strict `useDataTableData()`. Each
component also now accepts `className`, applied to its outer `EmptyBody` — necessary whenever the
real table has a fixed height cap (`Table height={N}`), since `EmptyBody` sits outside that table's
own scroll wrapper and needs its height set independently to avoid a jump when swapping between a
loading/error/empty state and real rows (net out to the loaded state's total height minus the real
`TableHeader`'s own height, which stays outside the capped area in both states).

**Rule:** A screen with a simple, manually-managed `useQuery`-driven table should use these three
components directly with prop overrides — never hand-roll equivalent `Empty` markup, and never adopt
`DataTableProvider` solely to unlock them. When the table has a fixed height cap, pass a matching
`className` height on all three.

**Reference:** `ascendra-ui/components/data-table/{data-table-loading-body,data-table-error-body,
data-table-empty-body}.tsx` (component source); `ascendra-ui/providers/data-table/data-table.provider.tsx`
(`useOptionalDataTableData`). No real `ascendra-ui` showcase page uses these standalone yet — the
first real consumer is `ascendra-commons-ui`'s Audit Log Overview (`testbed/app/observability/audit/page.tsx`).

---

### AUI-020 — Extract a shared component only once real usage validates the shape; prefer composable sub-components over one component with many variant props

**Date:** 2026-09-23

**Observed gap:** The same single-metric "KPI tile" shape (a label, a value, optionally a trend
indicator, optionally a caption) was hand-rolled independently in all 10 real sample dashboards (40
near-byte-identical tile instances) and in 8 of the 10 sample reports (10 more instances across 6
further variant shapes) — with visible drift already showing between copies: two report files
(`marketing-campaign-analysis`, `sales-pipeline-report`) had byte-identical duplicated JSX in
separate files, and `esg-sustainability-report`'s KPI data carried an unused `positive` field that
should have driven the trend's color but never did, because the color was hardcoded instead of wired
to it.

**Correct pattern:** `KpiTile`/`KpiLabel`/`KpiValue`/`KpiTrend`/`KpiCaption` — composable primitives,
not one monolithic component. `KpiTile` deliberately renders no `Card` wrapper of its own, so it
composes under `Card`/`CardPanel` for the common bordered tile, or bare for a wrapper-less hero row
(the real shape `executive-business-review` uses) — no "opt out of a wrapper" prop needed, because
it never assumed one. `KpiValue` takes `size` (`xl`/`2xl`/`3xl`/`4xl`) and a `warning` variant for a
threshold-breach state (`supply-chain-ops-report`'s amber tiles) instead of each page inventing its
own conditional className. `KpiTrend` takes a `variant` (`badge` = `SimpleBadge` pill, `text` =
inline colored label) and a `direction`, rendering its own up/down icon — never hand-roll that icon
ternary again.

**Rule:** Before hand-rolling a new instance of a UI shape that already recurs elsewhere in
`ascendra-ui`, check whether real usage justifies extracting (or reusing) a shared component. A
shape appearing 2-3 times in one file isn't enough evidence — see `ascendra-commons-ui`'s own
`packages/README.md`, which explicitly defers building `shared/observability/` until validated
against a second real consumer. A shape appearing in 10+ real pages, independently drifting between
copies, is overdue. When extracting, default to composable sub-components — matching every other
family in this library (`Card`/`CardHeader`/`CardPanel`, `Field`/`FieldLabel`/`FieldHint`,
`Empty`/`EmptyHeader`/`EmptyTitle`) — over one component absorbing every independently-varying axis
(size, wrapper, trend style, warning state) as props, which tends toward exactly the prop-soup this
library's composable convention exists to avoid.

**Reference:** `ascendra-ui/components/common-ui/kpi-tile.tsx` (component source); all 10
`app/showcase/dashboards/*/page.tsx` and the 8 `app/showcase/reports/*/page.tsx` files with real KPI
content, all retrofitted onto it — each KPI row is a real, varied adoption, not a toy example.

---

### AUI-021 — A loading/error/empty state replaces only the dynamic content inside a section's shell, never the whole shell

**Date:** 2026-09-23

**Observed gap:** An early version of `ascendra-commons-ui`'s Audit Log Overview dashboard gated
entire sections behind `{isLoading && <Text>}` / `{data && (<...the whole Card/KPI-row/table...>)}`
— so during loading the page showed a bare loading line, then the entire KPI row + chart + table
popped into existence at once when the query resolved. The same defect showed up more subtly on a
fixed-height table: its loading placeholder wasn't dimensioned to match the loaded state's height,
causing a visible jump on swap (see AUI-019's height-matching note for the fix).

**Correct pattern:** Render a section's static shell — `Card`/`CardHeader`/`CardPanel`, any label
text that isn't actually fetched, a real `Table`'s `TableHeader` — unconditionally, every render,
regardless of loading state. Only the innermost genuinely-fetched content (a KPI's value+trend, a
chart's data series, a table's rows) swaps between a matched-dimension placeholder and the real
content. For a chart specifically, the placeholder should be the real chart component fed realistic
placeholder data (flat, muted-colored bars; real axis labels where they're computable independent of
the fetch, e.g. a known date range) rather than a blank box or an unrelated generic shape — share
tick-formatter functions between the placeholder and the real chart so nothing about the axes
changes when data swaps in.

**Rule:** Before writing `{isLoading ? ... : data ? ... : ...}` around an entire section, check
whether any of that section's content doesn't actually depend on the fetch — that content belongs
outside the conditional, rendered unconditionally, with only the truly dynamic remainder swapped.

**Reference:** No real `ascendra-ui` showcase page demonstrates a loading state yet — all 10
dashboards and 10 reports render static synthetic data with no `useQuery` at all, so there's nothing
in the showcase this corrects (yet). `ascendra-commons-ui`'s Audit Log Overview
(`testbed/app/observability/audit/page.tsx`) is the first real implementation; the primitives it's
built from are real: `ascendra-ui/components/ui/skeleton.tsx` (`Skeleton`) and
`ascendra-ui/components/ui/empty.tsx` / `ui/table.tsx` (`Empty`, `EmptyBody`).

---

### AUI-022 — Use `WithState`/`WithSkeleton` for a section's loading/error/empty/normal branching outside a real `DataTableProvider` table

**Date:** 2026-09-25

**Observed gap:** `ascendra-commons-ui`'s Audit Log Overview dashboard
(`testbed/app/observability/audit/page.tsx`) branched loading/error/empty/normal two different ways
in the same file — a raw ternary for each KPI tile, and the `DataTableLoadingBody`/
`DataTableEmptyBody`/`DataTableErrorBody` trio (see AUI-019) for the Top Actions table — leaving no
generalized primitive for a non-table section (KPI tiles, chart panels) needing the same four-way
branch AUI-021 already established the shell/content-swap discipline for.

**Correct pattern:** `ascendra-ui` now ships two composable families in `components/common-ui/`:
`WithState`/`NormalState`/`LoadingState`/`ErrorState`/`EmptyState` (showcase `feedback/with-state`)
and `WithSkeleton`/`SkeletonState` (showcase `feedback/with-skeleton`). Each state is a self-guarding
sibling gated by its own `if` prop — usable standalone or together, inside or outside a table.
`LoadingState`/`ErrorState`/`EmptyState` render a built-in `Empty`-based UI by default and accept
`children` to override it entirely; `NormalState`/`SkeletonState` always require `children`, since
there is no generic "success"/"skeleton" shape — consistent with AUI-021's point that a skeleton
must match what it's replacing.

**Rule:** For any async-state branch outside a real `DataTableProvider`-driven table (KPI tiles,
cards, chart panels, any query-driven section), use this family instead of a hand-rolled ternary or
reusing `DataTable*Body` outside its scope. Nothing enforces the four `WithState` branches are
mutually exclusive — pass non-overlapping `if` conditions yourself. When deriving those from a
`@tanstack/react-query` result, reuse `query.isLoading`/`query.isError` directly rather than
re-deriving them under a helper using those same names — `useQuery` already returns fields called
`isLoading`/`isError`, so a helper re-exporting those names forces every caller to alias one set to
avoid a variable collision when both are destructured together.

**Reference:** `ascendra-ui/components/common-ui/with-state.tsx`;
`ascendra-ui/components/common-ui/with-skeleton.tsx`; showcase pages `feedback/with-state`,
`feedback/with-skeleton`. First real consumer: `ascendra-commons-ui`'s Audit Log Overview
(`testbed/app/observability/audit/page.tsx`), not yet retrofitted as of this entry.

---

### AUI-023 — Every top-level screen's `PageHeader` needs `variant="dashboard"`

**Superseded by AUI-033** (2026-09-26) — narrowed to dashboard-type screens only; see AUI-033 for
the full per-page-type recipe. Left below unedited per this file's append-only convention.

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: a module screen's first pass used a bare
`<PageHeader>` — no bottom border separating the title from the content below.

**Correct pattern:** `PageHeader` takes a `variant?: "dashboard"` prop —
`variant === "dashboard" && "border-b border-border mb-8"` in the component's own source.
Without it, `PageHeader` renders with no border at all by design (meant for callers composing
their own separator).

**Rule:** Every top-level module/dashboard screen's `PageHeader` passes `variant="dashboard"`,
unless there's a specific, stated reason not to.

**Reference:** `ascendra-ui/components/layout/page-header.tsx`.

---

### AUI-024 — Table header labels are sentence case; `DataTableHead`/`TableHead` default to `whitespace-nowrap`

**Date:** 2026-09-14. **Resolved 2026-09-26** — the interim per-usage workaround below is no
longer needed; the component itself now defaults to `whitespace-nowrap` (commit
`50a5d2ccbc1f08d586b08838e769970e8c95e0cb`, squash-merged to `ascendra-ui` `main`). Every real
`DataTableHead`/`TableHead` usage in `ascendra-commons-ui` has had its explicit
`className="whitespace-nowrap"` removed accordingly — a caller wanting wrapped header labels can
still override via `className` (`cn()` uses `tailwind-merge`, so a caller's own `whitespace-*`
class wins over the default).

**Observed gap:** Found building `audit-logging.ui`: column labels were Title Case ("Entity
Type", "Correlation ID") instead of sentence case, and headers wrapped onto two lines at
narrow widths.

**Correct pattern (historical — see resolution above):** No wrap behavior existed on
`DataTableHead`/`TableHead` before the fix — confirmed against
`ascendra-ui/components/data-table/data-table-head.tsx` and
`ascendra-ui/components/ui/table.tsx`; neither set `whitespace-nowrap` by default. This was a
genuine component gap, not a misuse of the API.

**Rule:** Column/field labels are always sentence case ("Occurred at", not "Occurred At") —
capitalize only the first letter and proper nouns. Do **not** re-add
`className="whitespace-nowrap"` to a `DataTableHead`/`TableHead` usage — it's redundant now that
the component defaults to it; only add a `className` when a header specifically needs to wrap.

**Reference:** `ascendra-ui/components/data-table/data-table-head.tsx`;
`ascendra-ui/components/ui/table.tsx`.

---

### AUI-025 — A record reached by clicking a table row is a `Sheet`, not a plain page

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: a record-detail screen's first pass was a
plain page (`PageHeader` + `Card`/`CardPanel` + a hand-rolled `<dl>`) instead of a `Sheet`.

**Correct pattern:** `Sheet`/`SheetContent`/`SheetHeader`/`SheetTitle`/`SheetDescription`/
`SheetBody`/`SheetProperties`/`SheetKey`/`SheetValue`/`SheetSection`/`SheetSectionHeader` — the
"Detail view" pattern `create-sheet.md` documents (key-value rows via `SheetProperties`,
grouped with `SheetSection`). `create-sheet.md`'s own default assumes the sheet's open/close
state lives on the same page as the triggering table (`useState`, no navigation) — a
deep-linkable/permalink record is the one case that stays its own route instead, still styled
as a `Sheet` via a `Sheet open onOpenChange={...}` wrapper that navigates back on close.

**Rule:** Any "click a row to see the record" screen uses the `Sheet` system, never a full
`PageHeader`/`Card` page. If the record needs a stable, shareable URL, keep it a route but
still render `Sheet`/`SheetContent` inside it (open state controlled by the route itself,
`onOpenChange` navigating back) rather than a plain page layout.

**Reference:** `ascendra-ui/components/ui/sheet.tsx`;
`ascendra-ui/.claude/commands/create-sheet.md` ("Detail view" pattern).

---

### AUI-026 — `DataTableHighlight` is required on every searchable cell of a query-driven table

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: a list screen's first pass rendered plain
text in every cell — `DataTableSearchInput` was present and functional, but no cell used
`DataTableHighlight`, so a search match was never visually marked in the results.

**Correct pattern:** `DataTableHighlight` (`text`, `item`, `itemKey` props) — wraps a cell's
text and marks the matched substring via `useDataTableSearch()`'s `getRanges`. Used for every
plain-text cell value in the real reference page (`data-table-lab`).

**Rule:** Any cell showing a searchable text value in a `DataTableWithQueryProvider`/
`DataTableProvider` table wraps that text in `DataTableHighlight`. A composed cell (badge + id,
a link) still wraps its plain-text portion.

**Reference:** `ascendra-ui/components/data-table/data-table-highlight.tsx`;
`ascendra-ui/app/showcase/data-table-lab/page.tsx`.

---

### AUI-027 — `CopyText` is required on any identifier a user would want to copy

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: actor ids, entity ids, correlation ids,
and event/record ids all rendered as plain text with no copy affordance — `CopyText` exists in
the library and was simply never used.

**Correct pattern:** `CopyText` (`value`, `showTooltip`, `className`, `children`) — wraps an
identifier, shows a copy icon, copies `value` to the clipboard on click, optional tooltip
confirmation.

**Rule:** Any cell/property value that is an id a person would plausibly want to copy (event
id, entity id, correlation id, actor id) uses `CopyText`. When the same value is also a
navigation link (e.g. a correlation id linking to its trace), give the link and the copy
affordance separate click targets — `CopyText`'s own root element owns the click-to-copy
handler, so nesting a second `onClick` inside its children fires both at once; use a sibling
element (e.g. a small icon-button) for the navigation action instead.

**Reference:** `ascendra-ui/components/util/copy-text.tsx`.

---

### AUI-028 — Model a combinable filter set as one "Advanced Filter" `QueryDef`, not fragmented single-purpose presets, whenever the API ANDs fields independently

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: a list screen's first pass split one
ANDed query (every field optional, all combinable) into several separate, mutually-exclusive
named `QueryDef`s ("By Entity", "By Actor & Action", …) — a user could never combine, say, an
actor filter with a date range, even though the underlying API supports exactly that.

**Correct pattern:** One `group: 'filter'` `QueryDef` carrying every combinable field as its
own `ParamItem`, laid out with `QueryDef.columns` (`{ sm, md, lg }`, already supported by
`data-table-query.types.ts`, easy to forget to set) for a proper multi-column grid instead of
one field per row.

**Rule:** Before splitting a filterable list screen into multiple named `QueryDef`s, check
whether the backing API's filter fields are independently ANDable. If they are, model it as
one "Advanced Filter" `QueryDef` with all fields together (plus `columns` for layout) rather
than one preset per field — reserve separate named presets for scenarios that are genuinely
different requests (e.g. a dedicated endpoint), not different filters on the same one. See
also AUI-006's related but distinct point on exploring the full DataTable feature surface
before defaulting to the plainest shape.

**Reference:** `ascendra-ui/providers/data-table-query/data-table-query.types.ts`
(`QueryDef.columns`).

---

### AUI-029 — A mock/fixture batch size smaller than every "Results per page" option makes pagination look broken when it isn't

**Date:** 2026-09-14

**Observed gap:** Found building `audit-logging.ui`: PO review reported a "Results per page"
selector "does not reflect changes" on a list screen.

**Correct pattern:** Confirmed against `ascendra-ui/providers/data-table/data-table.provider.tsx`
and `use-pagination.hook.ts`: when a table uses `DataTableWithQueryProvider`,
`DataTableProvider`'s `data` is the current server batch only (from the query context), and
`usePagination` re-slices client-side within that one batch. A fixture batch size smaller than
every "Results per page" option (5/10/15/20/100) means changing the page size has nothing to
visibly change — the control itself can be wired correctly regardless.

**Rule:** Before concluding a `DataTableWithQueryProvider` pagination control is broken, check
whether the mock/fixture batch size is at least as large as the smallest "Results per page"
option being tested — a batch smaller than every page-size choice will always look inert,
independent of whether the control works. Same root-cause class as AUI-017's undersized-fixture
finding.

**Reference:** `ascendra-ui/providers/data-table/data-table.provider.tsx`;
`ascendra-ui/providers/data-table/use-pagination.hook.ts`;
`ascendra-ui/components/ui/table.tsx` (`TableFoot`'s page-size `Select`).

---

### AUI-030 — `Empty`'s `border-dashed` class renders no visible border on its own; pair it with a real container or an explicit `border` utility

**Date:** 2026-09-25

**Observed gap:** Testing `ErrorState`'s default rendering (standalone, no `Card` wrapper) on
`ascendra-commons-ui`'s Audit Overview page showed a completely boundary-less block — no border,
no background, nothing distinguishing it from surrounding page content.

**Correct pattern:** `Empty`'s own base classes include `border-dashed` (`rounded-xl border-dashed
p-10 text-center text-balance`) but never pair it with a `border`/`border-{n}` width utility.
Tailwind's preflight resets `border-width: 0` globally (`*, ::before, ::after { border: 0 solid;
}`), so a border-*style* utility alone has zero visual effect without an accompanying
border-*width* utility — this affects every usage of bare `Empty`/`LoadingState`/`EmptyState`/
`ErrorState`'s default rendering, not just one screen. `Card`'s own default look (`bg-muted
rounded-xl`, no border at all) is a different, actually-working visual language for the same
"boundary" need.

**Rule:** Don't rely on `Empty`'s own default classes for a visible boundary — either wrap it in
`Card`/`CardPanel` (matching the rest of a dashboard's panels), or pass an explicit `className`
that adds real border width (e.g. `border` alongside the existing `border-dashed`) if a dashed
box is specifically wanted standalone. This is a real component gap worth fixing at the source
(add `border` to `Empty`'s own base classes) rather than patching around at every call site
indefinitely — flag for a library-level fix if it recurs.

**Reference:** `ascendra-ui/components/ui/empty.tsx`; `ascendra-ui/components/card/card.tsx`
(`bg-muted rounded-xl`, no border, as the working alternative); Tailwind's own
`node_modules/tailwindcss/preflight.css` (`border: 0 solid` reset).

---

### AUI-031 — Need a `*State`'s default content nested inside your own wrapper under a separate condition? Use its `*Message` sibling, never a nested `*State`

**Date:** 2026-09-25

**Observed gap:** `ascendra-commons-ui`'s Audit Overview page needed `ErrorState`'s default
content (title/description/icon/retry button) wrapped in `Card`/`CardPanel`, with the whole thing
still gated on `overview.isError`. Since `Card`/`CardPanel` have to wrap `Empty` from the outside
but the `if` gate lives on `ErrorState` (which generates that `Empty` internally), a first attempt
nested a second, always-true `ErrorState` inside an outer gating `ErrorState`
(`<ErrorState if={cond}><Card><CardPanel><ErrorState if onRetry={...} /></CardPanel></Card></ErrorState>`)
purely to reach the default content a second time. This works, but a component nested inside
itself to serve two unrelated roles (gate vs. content template) is exactly the kind of thing this
library's naming is supposed to make unnecessary — and it's easy to also drop the specific
`title`/`error` props in the process, silently falling back to generic copy.

**Correct pattern:** `LoadingState`/`ErrorState`/`EmptyState` each delegate their own default
rendering to a gate-free sibling — `LoadingMessage`/`ErrorMessage`/`EmptyMessage` — with no `if`
and no `children` prop, always rendering. Use the `*Message` directly for exactly this
composition: `<ErrorState if={cond}><Card><CardPanel><ErrorMessage error={...} onRetry={...}
/></CardPanel></Card></ErrorState>`. One `*State` in the tree (the real gate), one `*Message` (the
content) — no same-component nesting, no risk of forgetting a prop the outer instance was never
going to pass through anyway.

**Rule:** Never nest a `*State` component inside another `*State` of the same kind just to reach
its default content — reach for the matching `*Message` component instead. If a future addition
to this family (a hypothetical new `*State`) doesn't yet have a `*Message` sibling and this
composition need comes up again, that's the signal to add one, following this same pattern.

**Reference:** `ascendra-ui/components/common-ui/with-state.tsx`
(`LoadingMessage`/`ErrorMessage`/`EmptyMessage`); showcase page `feedback/with-state` ("Default
content nested inside a separately-gated wrapper" example).

---

### AUI-032 — A chart loading skeleton is the real chart fed parameterized placeholder data — there is no single component that infers its shape automatically

**Date:** 2026-09-26

**Observed gap:** AUI-021 already established that a chart's placeholder should be "the real chart
component fed realistic placeholder data" rather than a blank box, and `ascendra-commons-ui`'s Audit
Log Overview volume chart (`testbed/app/observability/audit/page.tsx:104-110`, `:256-293`) is a working
example — but it's the *only* one, and its placeholder generator (`buildPlaceholderPerDay`) is
hardcoded to a `{ day: string, count: number }` shape: 30 calendar days back from today, one series. A
survey of every `<BarChart>`/`<ComposedChart>` in `ascendra-ui`'s showcase (10 dashboards + 6 reports,
19 bar-bearing charts total) found at least six structurally distinct shapes in real use: flat-fill
single-series (4), per-datum conditional `<Cell>` fill where color depends on the value itself (5:
`devops`, `real-estate`, `marketing`, `trading-portfolio`, `financial-pnl`), grouped multi-series (7),
stacked multi-series (2), and stacked-bars-plus-`<Line>`-overlay on a dual Y-axis (2, `financial-pnl`).
13 of 19 use a categorical axis (department/region/channel/histogram-bin/property-name), not a
calendar date; none use a rolling "N days back from today" domain like the one example that exists.
Every chart hardcodes its bar `fill` as a literal `var(--chart-N)` or a conditional expression — zero
use the dynamic `var(--color-{key})` `chartConfig` convention — so there is no shared config value a
generic component could read to know what "muted" means for a given chart. None of these 16 files has
a real query hook yet (all render static mock arrays), so this is forward-looking guidance, not a
retrofit. Conclusion: a single `ChartSkeleton` that inspects a real chart's data/config and invents its
own placeholder is not achievable — it would have to guess axis domain type, series count, stacking
mode, and layout orientation, none of which are inferable from a data array.

**Correct pattern:** Target the same fidelity tier as the existing example — the real
`ChartContainer`/chart component (`BarChart`, `LineChart`, `AreaChart` alike; the recipe doesn't care
which), real `CartesianGrid`/axis/tick-formatter config shared verbatim with the loaded state, no
`ChartTooltip`, series color swapped to `var(--muted)`, wrapped in `animate-pulse` — so the plot area is
pixel-identical between loading and loaded and nothing shifts when real data lands. A flat, non-zero
placeholder value renders a visible shape for any of these chart types (a bar, line, or area at `y=0`
is invisible/degenerate the same way a zero-height bar is). What does **not** carry over automatically
is the placeholder *data itself* — every chart still needs its own domain (a date-range generator, or
an explicit category-label list) and its own series keys supplied as parameters; there is no canned
dataset that fits more than one chart. For the per-datum conditional-`<Cell>`-fill charts, skeleton
mode should force one flat muted fill for every cell and drop the conditional entirely — there's no
real value yet for the condition to evaluate, so faking it isn't meaningful. Once any of these charts
gains a real query, delay showing the skeleton by ~150-300ms so a fast/cached response never flashes
one frame of it (matches the general debounce convention for any `LoadingState`, not chart-specific).

**Rule:** When giving a new dashboard/report chart (bar, line, or area) a loading state, reuse this
recipe — real chart component + shared axis config + muted flat placeholder data, `WithSkeleton`/
`SkeletonState` gating it (AUI-022) — never a generic `<Skeleton>` rectangle or shimmer silhouette for
a chart that already has a real axis config to share. Do not reach for or build a single
auto-inferring `ChartSkeleton` component expecting it to work across chart shapes unmodified; instead,
each chart's own file supplies its own domain generator, series keys, and layout orientation, the same
way each chart today already supplies its own `chartConfig`/tick formatters by hand. If a shared
low-level helper is built (a `makePlaceholderData`-style generator, mirroring the existing
`makeTooltipFormatter`/`ChartSeriesLegend` precedent of small composable per-concern helpers rather
than one monolithic chart component), it takes the domain and series keys as explicit arguments — it
does not infer them.

**Reference:** `ascendra-commons-ui/testbed/app/observability/audit/page.tsx:104-110,256-293`
(working example, AUI-021); `ascendra-ui/components/charts/make-tooltip-formatter.tsx`,
`ascendra-ui/components/charts/chart-series-legend.tsx` (existing composable-helper precedent). No
`ChartSkeleton`/placeholder-generator component exists in `ascendra-ui` yet as of this entry — this is
a design decision to steer the eventual extraction, not a record of a shipped component.

---

### AUI-033 — A page's top-level structure is one of four fixed recipes, keyed to page type; `PageMain` is never skipped except by the two that replace it entirely

**Date:** 2026-09-26

**Observed gap:** Every non-dashboard screen in `ascendra-commons-ui`'s `audit-logging.ui`
(`events/page.tsx`, `retention/page.tsx`, `trace/[correlationId]/page.tsx`,
`entities/[type]/[id]/page.tsx`, `actors/[actor]/page.tsx`, and the `observability` landing page)
skipped `PageMain` entirely — going straight from the header `Fragment` to a bare `MainContent` —
and none used `PageWrapper`/`PageContent` at all, despite `ascendra-ui` shipping a full page-layout
family (`PageMain`, `PageWrapper`, `PageContent`, `MainContent`, `AsideContent`, `DashboardContent`,
`ReportDocumentWrapper`, and the `Tabs`/`TabList`/`TabTrigger`/`TabContent` family) purpose-built for
exactly this. Investigating the real component source and showcase also surfaced that AUI-023's rule
("every top-level screen needs `PageHeader variant="dashboard"`") was an overgeneralization drawn
from having only built one dashboard-type screen at the time — the real pattern is narrower (see
below) — and a second, unrelated finding: `TableBar`'s `data-slot` had a typo (`"able-bar"` instead
of `"page-bar"`) that silently made `PageContent`/`TabContent`'s margin-collapse behavior permanently
dead code. **That typo has since been fixed directly in `ascendra-ui`** (commit
`220732fa16f44a3051fae99c31598f05cb2d50e8`, squash-merged to `main`) — this is not a workaround
entry, the component itself is corrected.

**Correct pattern — four page-type recipes, each keyed by real `ascendra-ui` source/showcase:**

1. **Dashboard page** — `PageHeader variant="dashboard"` (the border/margin comes from the header
   itself: `variant === "dashboard" && "border-b border-border mb-8"` in `page-header.tsx`) +
   `DashboardContent` (`flex flex-col gap-4`, no custom props). `DashboardContent` **replaces**
   `PageMain`/`PageWrapper`/`PageContent` entirely — none of the three appear. Confirmed: every real
   `app/showcase/dashboards/*/page.tsx` imports `DashboardContent` + `PageHeader` and nothing from
   the `PageMain` family.
2. **Report page** — no `PageHeader` at all. `(BackLink) + ReportDocumentWrapper` only, which
   replaces everything including the header. Confirmed: every real `app/showcase/reports/*/page.tsx`.
3. **Tabbed page** — plain `PageHeader` (**no** `variant="dashboard"`) → `PageMain` → `Tabs
   defaultValue="..."> TabList > TabTrigger[]` + `TabContent[]`. Skip `PageWrapper`/`PageContent` —
   `TabList` draws its own bottom hairline as the divider, and `TabContent` already carries
   `PageContent`'s exact spacing className (`mt-8 flex flex-col gap-6`, inner `flex flex-col
   items-start gap-8 lg:flex-row`) internally, so nesting `PageContent` inside `TabContent` would
   double it. Each `TabContent` wraps `MainContent` directly (plus `AsideContent` as a sibling, for a
   two-column tab). Confirmed: `app/showcase/data-table-lab/page.tsx:83-256` (`PageMain > Tabs >
   TabList + TabContent > MainContent`, real page skips `PageWrapper` entirely here, same as this
   entry recommends) and `app/showcase/layout-guide/page.tsx:395-481`.
4. **Standard page** (list/detail/settings — everything not covered by 1-3) — plain `PageHeader` (no
   variant) → `PageMain` (mandatory, never skipped) → `PageWrapper` (renders one unconditional
   `border-border border-b` divider div, nothing else — this is the header-to-content separator) →
   `PageContent` (`mt-8 flex flex-col gap-6`, plus an inner `flex flex-col items-start gap-8
   lg:flex-row` div that is what makes `MainContent` + `AsideContent` sit side-by-side at `lg:` —
   `MainContent` itself is `lg:flex-1`, `AsideContent` is `lg:w-fit`) → `MainContent` (+ `AsideContent`
   as a sibling, only when the page has a genuine secondary panel). Confirmed:
   `app/showcase/layout-guide/page.tsx:313-333` and its own doc text at line 292: *"The standard
   full-page composition tree: (BackLink) → PageHeader → PageMain → PageWrapper → PageContent body →
   MainContent → sections. PageWrapper renders the horizontal rule that separates the header from the
   content area. For tabbed pages, replace PageWrapper with Tabs — the TabList provides the divider."*
   — this is `ascendra-ui`'s own stated rule for recipe 3 vs. 4, not an inference.

`PageContent`'s `mt-8` **always** applies for recipe 4 — its collapse-to-`mt-0` selector
(`[[data-slot=page-bar]+&]:mt-0`, meant to zero the margin when a search/filter toolbar sits directly
above) requires a preceding sibling with `data-slot="page-bar"` exactly; before this entry's fix, no
component in `ascendra-ui` ever set that value (`TableBar`, the component actually showcased as "Page
Bar", had the `able-bar` typo), so don't design a layout assuming this collapse will fire based on
old behavior — it now does, but only when `TableBar` is that direct preceding sibling.

**Superseding AUI-023:** `variant="dashboard"` on `PageHeader` is exclusive to recipe 1 (paired with
`DashboardContent`) — never apply it to a tabbed or standard page. A standard/tabbed page's header
divider comes from `PageWrapper` or `TabList` instead, never from the header itself.

**Rule:**
- Classify every new screen into exactly one of the four recipes above before writing its JSX — never
  default to a bare `MainContent` with no `PageMain` wrapper, and never mix a recipe's divider source
  with another's (e.g. `variant="dashboard"` alongside `PageWrapper` would double the divider).
- `PageMain` is skipped only for recipes 1 and 2, where `DashboardContent`/`ReportDocumentWrapper`
  subsume it. Every recipe-3/4 screen keeps it.
- `PageWrapper` and `PageContent` are a pair for recipe 4 — don't use one without the other; `Tabs`
  replaces both together for recipe 3, not just `PageWrapper`.
- Add `AsideContent` only when a screen has a genuine secondary/contextual panel, as a sibling of
  `MainContent` inside `PageContent` (recipe 4) or `TabContent` (recipe 3) — never as a substitute for
  a second `MainSection`-style block within the same column (`ascendra-ui` has no `MainSection`
  family; use `Card`/`CardHeader`/`CardPanel`/`CardPanelItem` — see AUI-020 — for grouped/divided
  content within one column).

**Reference:** `ascendra-ui/components/layout/{page-main,page-wrapper,page-content,main-content,
aside-content,dashboard-content}.tsx`; `ascendra-ui/components/tabs/{tabs,tab-list,tab-trigger,
tab-content}.tsx`; `ascendra-ui/components/layout/table-bar.tsx` (the `data-slot="page-bar"` fix,
commit `220732fa16f44a3051fae99c31598f05cb2d50e8`); `ascendra-ui/app/showcase/data-table-lab/page.tsx`;
`ascendra-ui/app/showcase/dashboards/saas-revenue/page.tsx`;
`ascendra-ui/app/showcase/reports/annual-financial-statement/page.tsx`;
`ascendra-ui/app/showcase/layout-guide/page.tsx` (lines 292-333 standard shell, 395-481 tabs,
906-985 aside).

---

### AUI-034 — Use `formatDateTime` for any timestamp display; never a raw `.toLocaleString()`/`.toLocaleDateString()`

**Date:** 2026-09-26

**Observed gap:** `ascendra-commons-ui`'s `audit-logging.ui` had 9 real call sites calling
`new Date(x).toLocaleString()`/`.toLocaleDateString()` directly — event timestamps, actor
first/last-seen, retention job's last-run — each locale/OS-dependent and inconsistent with the
`'en-US'`-hardcoded convention every other formatter in the library already follows. No shared
utility supported time-of-day or relative/elapsed time ("3 days ago") at all before this entry —
a real, previously-flagged gap (this same need was noted while building this module's Overview
dashboard, months before this fix).

**Correct pattern:** `formatDateTime(value, options)` in `ascendra-ui/utils/common.util.ts` —
`style?: 'short' | 'medium' | 'long' | 'relative'` (default `'medium'`), `time?: boolean` (append
time-of-day, ignored when `style: 'relative'`), `relativeFallbackDays?: number` (default 7 — past
this age, `'relative'` auto-falls-back to an absolute `'medium'` date, via
`date-fns`'s `formatDistanceToNow`, so an old timestamp never reads as "8 months ago"). The
existing `formatDate`/`formatShortDate`/`formatDateRange` now delegate to it internally and are
`@deprecated` — not removed, since they're already used across real dashboards/reports (same
"can't remove, mark obsolete instead" precedent as any widely-adopted shared function) — new code
should call `formatDateTime` directly rather than the deprecated wrappers.

Choosing `time`/`style` is a real per-field decision, not a default to copy everywhere — this
module split it as: a **raw audit-event timestamp** (the event list's "Occurred at" column, the
record detail Sheet, the entity/trace timeline) uses `{ time: true }` (precise, absolute — an audit
trail's whole point is exact time), while an **activity-status field** (actor "first seen"/"last
seen", a retention job's "last ran", a per-action "last occurred") uses `{ style: 'relative' }`
(a human-friendly "2 hours ago" is more useful there than a precise timestamp, and the
`relativeFallbackDays` auto-fallback keeps a very old "first seen" from reading as nonsense).

**Rule:** Never call `.toLocaleString()`/`.toLocaleDateString()` on a `Date` directly for anything
a user reads — always `formatDateTime`. Before picking `style`/`time` for a new field, ask whether
it's a precise historical record (→ absolute, usually with `time: true`) or an at-a-glance
recency/status indicator (→ `style: 'relative'`) — don't default to one without considering which
the field actually is.

**Reference:** `ascendra-ui/utils/common.util.ts` (`formatDateTime`, commit
`11b8d802d39ba70751514faa53371aa1971b5306`); retrofitted real usage across
`ascendra-commons-ui`'s `audit-logging.ui` — `events/page.tsx`, `events/[id]/page.tsx`,
`_audit-event-timeline.tsx`, `retention/page.tsx`, `actors/[actor]/page.tsx` (both `packages/` and
`testbed/` copies).

---

### AUI-035 — A column with a custom-formatted cell needs its own `ColumnDef.searchValue`, or search/highlight silently disagree with what's shown

**Date:** 2026-09-26

**Observed gap:** Found via `audit-logging.ui`'s `occurredAt` column (`type: 'date'`, displayed via
`formatDateTime(row.occurredAt, { time: true })` — see AUI-034) after switching it to a custom,
time-inclusive display format. `useSearch`'s two search paths each computed a searchable string
completely independent of what a column actually renders: plain (non-fuzzy) search used a fixed
`new Date(val).toLocaleDateString('en-GB', { day: '2-digit', month: 'short', year: 'numeric' })` —
no time-of-day, ever — and fuzzy search (Fuse.js) indexed the **raw field value** directly (e.g.
the raw ISO string). `DataTableHighlight` then applies whatever character ranges either path
produces straight onto the caller's own `text` prop. Confirmed via a direct Fuse.js reproduction:
searching "sep" in fuzzy mode against a raw ISO date returns indices computed against that raw
string, which — sliced out of a differently-formatted display string like `"Sep 04, 2026, 6:05
PM"` — land on the wrong characters. Plain-mode search happened to still work for this specific
column purely by coincidence (both the internal en-GB format and the display format use the same
month abbreviations), but the time-of-day portion was never searchable, and nothing guaranteed the
coincidence would hold for a differently-formatted column (e.g. a `style: 'relative'` cell like
"2 hours ago" — searching "hours" would find nothing, since the internal formatter never produces
that string).

**Correct pattern:** `ColumnDef<T>` now accepts an optional `searchValue?: (value, row) => string`.
When present, both search paths call it instead of the built-in per-`type` formatter — the plain
filter directly, and Fuse's per-key `getFn` (Fuse.js v7's `keys: [{ name, getFn }]` form, not a bare
key-name string) for fuzzy mode, so fuzzy match/highlight ranges are computed against the same
string rather than the raw value. **The contract:** `column.searchValue(value)` must return the
exact same string the cell passes to `DataTableHighlight`'s `text` prop — nothing enforces this at
compile time, it's a convention. `audit-logging.ui`'s `occurredAt` column sets
`searchValue: (value) => formatDateTime(value as string, { time: true })`, mirroring its cell's
`formatDateTime(row.occurredAt, { time: true })` exactly. Columns with no override are unaffected —
confirmed byte-identical fallback behavior to the pre-existing formatter.

**Rule:** Any column whose cell renders a custom-formatted string (a shared date/time formatter, a
relative-time string, a currency format that differs from the library's built-in `type: 'number'`
handling, etc.) sets a matching `searchValue` that calls the *identical* formatter/options the cell
uses — never leave it to the library's generic per-`type` default once the display diverges from
that default. A `type: 'date'`/`'number'` column with no custom cell formatting doesn't need one —
the built-in default already matches what's shown.

**Reference:** `ascendra-ui/providers/data-table/data-table.types.ts` (`ColumnDef.searchValue`);
`ascendra-ui/providers/data-table/use-search.hook.ts` (`resolveSearchValue`, the Fuse `getFn`
wiring); commit `3bab67f19b989540f1a9f4e0f365ee792482b788`. Real usage:
`ascendra-commons-ui/packages/audit-logging.ui/queries/audit-query-defs.ts` (`occurredAt`'s
`searchValue`, both `packages/` and `testbed/` copies).

---

### AUI-036 — Plain (non-fuzzy) `DataTable` search now matches multi-word queries word-by-word, not as one literal phrase

**Date:** 2026-09-26

**Observed gap:** Found immediately after AUI-035, real PO usage of the retrofitted `occurredAt`
column: searching `"sep"` matched and highlighted correctly, but `"sep 2"` returned zero rows.
Root cause, confirmed directly: plain-mode search (`use-search.hook.ts`'s non-fuzzy filter) treated
the whole typed string — spaces included — as one literal contiguous substring to find.
`"Sep 04, 2026, 6:05 PM"` never contains the literal 5-character sequence `"sep 2"` (`"04, "` sits
between `"Sep"` and `"2026"`), even though both words are individually present. `DataTableHighlight`'s
own fallback highlighter (used whenever fuzzy-computed ranges aren't available — i.e. always in
plain mode) had the identical single-phrase-regex limitation. Confirmed fuzzy mode already tolerated
this via Fuse's approximate matching — the gap was plain mode only. Not specific to date columns or
to AUI-035's `searchValue` addition — any column where a user types two words that aren't contiguous
in that exact order would misbehave the same way; a formatted date string just makes it obvious.

**Correct pattern:** Both the plain filter and `DataTableHighlight`'s fallback now split the typed
term on whitespace and treat each word independently — the filter requires every word to appear
somewhere across the row's searchable columns (not necessarily the same column, not necessarily
contiguous — row-wide AND, matching typical global-search-box UX), and the highlighter builds an
alternation regex (`(word1|word2|...)`) so each matched word highlights on its own. Single-word
queries are unaffected — confirmed equivalent to the prior behavior.

**Rule:** Don't design column search-value formatting to defensively avoid multi-word display
strings, or worry that "Month Day, Year" style formatting will break search — plain mode now handles
it word-by-word like a normal search box. This is orthogonal to AUI-035: AUI-035 makes search match
what a column *displays*; this entry makes multi-word phrases within that display actually findable.

**Reference:** `ascendra-ui/providers/data-table/use-search.hook.ts` (the word-split filter);
`ascendra-ui/components/data-table/data-table-highlight.tsx` (the word-alternation highlight
fallback); commit `878af90abbd724d1811a78678d74cda4489cc794`.

---

### AUI-037 — `FieldInfo`'s default (no-`breakPoint`) branch doesn't reserve height when empty, only its `breakPoint` branch does — diagnosed, fix proposed, then reverted; mitigate via `description` instead

**Date:** 2026-09-27 (fix proposed and applied); **reverted 2026-09-27** (same day, maintainer
decision).

**Observed gap:** Found reviewing `audit-logging.ui`'s Advanced Filter: fields with an `info`
subtitle sat visibly lower/misaligned relative to sibling fields without one in the same
`columns`-grid row (a real problem specifically at `lg: 3` — 5 identity-style fields plus a
date range meant not every field in a row had `info` set). Root cause, confirmed directly in
`ascendra-ui/components/ui/field.tsx`: `FieldInfo` has two render branches. The
`breakPoint`-set branch already has the exact fix `FieldHint` needed (see the "Default value fix"
section in `field-hint-guide.md`) — it falls back to a literal `&nbsp;` when `children` is falsy,
so it reliably reserves the row's height either way. The *default* branch (no `breakPoint` —
`QueryFieldRenderer`'s own call, `<FieldInfo>{field.info}</FieldInfo>`, never passes one) has no
such fallback: `<p>{children}</p>` with `children` `undefined` renders a genuinely empty
paragraph, which does not reliably match the height of a sibling `<p>` that has real text —
same underlying class of bug `FieldHint` was already fixed for, just not found in `FieldInfo`'s
other branch. **This diagnosis is still accurate and current** — only the proposed code fix
below was reverted, not the finding itself.

**A fix was proposed and briefly applied** (`FieldInfo` checking `!children` unconditionally
instead of `!children && breakPoint`, so any call with no content falls back to the same `&nbsp;`
placeholder regardless of `breakPoint`) — **but reverted the same day**, maintainer's call, rather
than change shared `FieldInfo` behavior for a gap already worked around at the content-authoring
level (see "Correct pattern" below). See `field-hint-guide.md`'s "FieldInfo's matching gap"
section for the full before/after of the reverted fix, kept for reference in case this is
revisited later.

**Correct pattern (the actual mitigation in place):** Don't rely on `info` (`FieldInfo`, rendered
above the input) for hint text on only *some* fields in a `columns`-grid row — that's exactly the
case this gap affects. Instead:
- If only one or a few "odd" fields in the row need explanatory text, put it in `description`
  (`FieldHint`, rendered below the input) instead of `info` — `FieldHint` already reserves height
  safely for a bare/empty call (that fix was applied and kept, see field-hint-guide.md's "Default
  value fix"), so mixing "some fields have a description, some don't" within one row is safe today,
  unlike the equivalent for `info`.
- Only reach for `info` when giving it to *every* field in the row uniformly (all have it, or
  none do) — a row where `info` presence is consistent across every field never hits this gap in
  the first place, since there is no "some empty, some not" case to misalign.

**Rule:** Before using `info` on a `FieldDef`/regular form field, check whether every sibling in
the same grid row will also have `info` set. If not, use `description` for the one(s) that need a
hint instead. Don't assume `FieldInfo` itself will keep a row aligned regardless of which fields
use `info` — it doesn't, and (per the revert above) that isn't being fixed at the library level for
now.

**Reference:** `ascendra-ui/components/ui/field.tsx` (`FieldInfo`);
`ascendra-ui/components/data-table/query-field-renderer.tsx` (the call site that surfaced it);
`field-hint-guide.md`; commits `9402593` (the fix) and `91e8b3f` (the revert) in `ascendra-ui`.

---

### AUI-038 — `QueryDef.params` (`FieldDef`) shares `field-hint-guide.md`'s `mandatory`/`optional` badge-minority rule — and a bounded range is one `daterange` field, not two independent `date` fields

**Date:** 2026-09-27

**Observed gap:** Found reviewing `audit-logging.ui`'s Advanced Filter against `data-table-lab`'s
own "Advanced Invoice Filter" reference example. Two separate gaps in the same screen:

1. Every one of the filter's 7 fields carried `optional: true` — every field in the form badged,
   which is exactly the crowding `field-hint-guide.md`'s "Form-authoring guidance" section already
   warns against for regular `Field`/`FieldHint` forms ("never badge every field in both
   directions at once ... the badge exists to flag the *minority* case"). That guidance was written
   against `components/forms/*.tsx` usage but applies identically to `QueryDef.params`/`FieldDef`
   — `mandatory`/`optional` on a `FieldDef` renders through the exact same `FieldHint` component
   (see `query-field-renderer.tsx`'s `<FieldHint mandatory={field.mandatory}
   optional={field.optional} .../>`), so the same crowding argument holds. When *every* field in a
   `QueryDef` is uniformly optional (or uniformly mandatory), there's no minority left to flag —
   the badge should be dropped entirely and the uniform fact stated once instead.
2. `occurredAfter`/`occurredBefore` were modeled as two independent `type: 'date'` `FieldDef`s, even
   though they're really one bounded range (a user picks a start and an end for the same
   condition) — `FieldType` already includes `'daterange'` for exactly this, demonstrated in
   `data-table-lab`'s own reference example (`dateRange`, `issueDateRange`, `dueDateRange`).

**Correct pattern:**
1. When every field in a `QueryDef` is uniformly optional/mandatory, remove the per-field
   `optional`/`mandatory` flags entirely and state it once via `QueryDef.info` (rendered in the
   filter panel's `CardFooter`) — e.g. `"Every field is optional — set any combination and results
   must match all of them (AND logic)."` — instead of an identical badge repeated on every field.
   Reserve per-field `mandatory`/`optional` for when a form has a genuine minority to flag, exactly
   as field-hint-guide.md already prescribes for regular forms.
2. Model a single bounded range (a from/to pair meant as one condition, not two independently
   optional ones) as one `type: 'daterange'` `FieldDef`, not two `type: 'date'` fields. The
   `DateRange` value it produces (`{ from?, to? }`) is already handled end-to-end —
   `serializeParamValues`/`deserializeParamValues` in `preferences.storage.ts` already special-case
   `DateRange` for localStorage persistence — so nothing else needs to change to support it; the
   query function/mock that previously read two separate `Date` params just reads
   `params.someRangeField as DateRange` and splits `.from`/`.to` back into whatever two-param shape
   the underlying API actually expects, entirely inside that one function.

**Rule:** Before adding `optional`/`mandatory` to every field in a new `QueryDef`, count how many
fields would actually get the badge — if that's all of them (or none), drop the badges and use
`QueryDef.info` instead. Before modeling two `date` fields for what's really one range, check
whether `type: 'daterange'` fits instead — it almost always does when the two dates are the same
logical condition (a bounded window), not two unrelated ones.

**Reference:** `ascendra-ui/providers/data-table-query/data-table-query.types.ts` (`FieldType`,
`FieldDef.span`/`.mandatory`/`.optional`, `QueryDef.info`); `field-hint-guide.md`'s
"Form-authoring guidance" section (the badge-minority rule itself, written for `Field`/`FieldHint`
but equally applicable here); `ascendra-ui/preferences/preferences.storage.ts`
(`serializeParamValues`/`deserializeParamValues`'s existing `DateRange` handling). Real usage:
`ascendra-commons-ui/packages/audit-logging.ui/queries/audit-query-defs.ts` (both `packages/` and
`testbed/` copies) and its `audit-query-functions.ts`/mock's `occurredRange` split.
