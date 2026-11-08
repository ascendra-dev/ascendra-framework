# Ascendra UI — UI Reference

> Auto-generated on 2026-11-08.
> Run `npm run gen:ui-docs` after any registry or config change.

---

## Overview

**Primitive components:** 72  
**Composite forms:** 10  
**Dialogs:** 12  
**Sheets:** 10  
**Drawers:** 8  
**Dashboards:** 10  
**Reports:** 10

### Primitive components by category

| Category | Components |
|---|---|
| Reports | Report Header, Report Document, Report Content |
| Charts | Chart Legend, Chart Target Legend |
| Feedback & Status | Rating, Color Tile, Simple Badge, Bubble Badge, Status Dot, KPI Tile, Simple Alert, Pro Badge, Unsaved Changes Bar, Progress & Stepper, Skeleton, With State, With Skeleton, With Loading, With Error, Toast |
| Forms & Inputs | Button, Input, Input Group, Checkbox, Radio Group, Switch, Select, Field, Table Lookup, Combobox, Sort Field, File Upload, Rich Text Editor, Color Picker |
| Navigation | Anchor, Nav Link, Header, Nav |
| Overlays | Dialog, Sheet, Dropdown Menu, Tooltip, Command Palette |
| Tables & Data | Simple Table, Table, Empty State, Data Table |
| Layout | Card, Page Header, Page Bar, App Shell & Page Layout, Aside Content, Item |
| Tabs | Segment Control, Tabs |
| Sidebar | Sidebar Menu |
| Utilities | With Copy Text, Name Avatar, Theme Toggle, Pagination Button, Row Action Button, Scroll To Top, Export CSV Button, Date Time, With Empty Value, With Sheet Icon, With Tooltip, Id Text |
| Date & Time | Calendar, Date Picker, Date Range Picker |

---

## Part 1 — Primitive Components

> These are the individual building blocks. Import from `@/ascendra-ui` (or `@/ascendra-ui/shadcn` where noted).

### Feedback & Status

#### Color Tile

Solid-color classification tile with a label and optional sublabel. Ideal for category grids, SDG alignments, priority indicators, and status tiles.

- **Import:** `import { ColorTile, ColorTileTitle, ColorTileSubTitle } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/color-tile](/showcase/feedback/color-tile)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant (ColorTile)` | `'primary' \| 'gray' \| 'green' \| 'emerald' \| 'blue' \| 'sky' \| 'orange' \| 'amber' \| 'yellow' \| 'red' \| 'rose' \| 'violet' \| 'pink' \| 'indigo' \| 'teal' \| 'cyan'` | `'primary'` | Solid background color of the tile. |
| `className (ColorTile)` | `string` | — | Additional CSS classes — use to control width, height, or padding. |
| `className (ColorTileTitle)` | `string` | — | Override the default text-xs font-bold opacity-80 styles. |
| `className (ColorTileSubTitle)` | `string` | — | Override the default text-[0.6rem] font-semibold leading-tight styles. |

---

#### Simple Badge

Small inline status badge for labeling content with semantic color variants.

- **Import:** `import { SimpleBadge } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/simple-badge](/showcase/feedback/simple-badge)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'secondary' \| 'orange' \| 'green' \| 'blue' \| 'red' \| 'amber' \| 'violet' \| 'warning' \| 'positive' \| 'info'` | `'default'` | Color variant of the badge. |
| `asChild` | `boolean` | `false` | Render as a child element using Radix Slot. |
| `className` | `string` | — | Additional CSS classes. |

---

#### Bubble Badge

Gradient pill badges with inset highlight, available in multiple sizes and colors.

- **Import:** `import { BubbleBadge } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/bubble-badge](/showcase/feedback/bubble-badge)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `size` | `'sm' \| 'md' \| 'lg'` | `'sm'` | Size of the badge. |
| `color` | `'gray' \| 'blue' \| 'green' \| 'red' \| 'amber' \| 'orange' \| 'violet' \| 'warning' \| 'positive' \| 'info'` | `'gray'` | Color variant. |

---

#### Status Dot

Tiny status indicator dot with a halo shadow ring, available in semantic colors.

- **Import:** `import { StatusDot } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/status-dot](/showcase/feedback/status-dot)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'orange' \| 'emerald' \| 'sky' \| 'violet' \| 'rose' \| 'amber' \| 'red' \| 'primary' \| 'gray' \| 'warning' \| 'positive' \| 'info'` | `'gray'` | Color variant. |

---

#### KPI Tile

Composable single-metric card for dashboard/report KPI rows — a label, a value, and an optional trend indicator or caption. Not a full Card replacement: wrap in Card/CardPanel for the bordered look, or use bare (no wrapper) for a hero-style oversized metric.

- **Import:** `import { KpiTile, KpiLabel, KpiValue, KpiTrend, KpiCaption } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/kpi-tile](/showcase/feedback/kpi-tile)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `size (KpiValue)` | `'xl' \| '2xl' \| '3xl' \| '4xl'` | `'2xl'` | Value text size. |
| `variant (KpiValue)` | `'default' \| 'warning'` | `'default'` | Set to warning to color the value amber when a metric breaches a threshold. |
| `direction (KpiTrend)` | `'up' \| 'down'` | — | Drives both the trend icon and its color (green/emerald for up, red/rose for down). |
| `variant (KpiTrend)` | `'badge' \| 'text'` | `'badge'` | badge renders a SimpleBadge pill (dashboard style); text renders an inline colored label with no pill background (report-hero style). |
| `className (KpiTile)` | `string` | — | Additional CSS classes on the padded flex-col shell — apply mt-auto to a child to pin it (and everything after it) to the bottom. |

---

#### Simple Alert

Compact alert notification box with semantic severity variants and an optional icon.

- **Import:** `import { SimpleAlert, AlertIcon } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/simple-alert](/showcase/feedback/simple-alert)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'secondary' \| 'destructive' \| 'success' \| 'warning' \| 'amber' \| 'primary'` | `'default'` | Alert severity variant. |
| `icon` | `React.ComponentType<{ className?: string; strokeWidth?: number }>` | — | Icon component to display (defaults to InfoIcon). |

---

#### Toast

Lightweight, accessible toast notifications powered by Sonner. Mount <Toaster /> once in the root layout and call toast() from anywhere.

- **Import:** `import { toast, Toaster } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/toasts](/showcase/feedback/toasts)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `description` | `string` | — | Optional supporting text shown below the title. |
| `action` | `{ label: string; onClick: () => void }` | — | Inline action button shown inside the toast. |
| `duration` | `number` | `4000` | Auto-dismiss delay in milliseconds. Pass Infinity to persist. |
| `id` | `string \| number` | — | Unique ID for updating or dismissing a specific toast. |

---

#### Pro Badge

Premium feature indicator with a blue-purple gradient treatment.

- **Import:** `import { ProBadge } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/pro-badge](/showcase/feedback/pro-badge)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### Unsaved Changes Bar

Fixed bottom bar that surfaces when a form has unsaved changes, with save/reset actions and status states.

- **Import:** `import { UnsavedChangesBar } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/unsaved-changes-bar](/showcase/feedback/unsaved-changes-bar)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `isDirty` | `boolean` | — | Shows the bar when true. |
| `isSaving` | `boolean` | — | Shows a loading spinner and hides action buttons. |
| `isValid` | `boolean` | — | When false on Save click, triggers nudge animation and validation message. |
| `onSave` | `() => boolean \| Promise<boolean> \| void` | — | Called on Save; return true for success, false for error. |
| `onReset` | `() => void` | — | Called on Reset click. |
| `onInvalid` | `() => void` | — | Called when Save is clicked on an invalid form — use to trigger field-level errors. |
| `message` | `string` | `'Unsaved changes'` | Idle state message. |
| `saveLabel` | `string` | `'Save'` | Save button label. |
| `resetLabel` | `string` | `'Reset'` | Reset button label. |
| `className` | `string` | — | Extra classes on the outer wrapper — use to offset centering when a sidebar is present. |

---

#### Progress & Stepper

ProgressBar wraps Radix Progress with size and color variants plus an indeterminate animation. Stepper is a fully custom horizontal step indicator with four statuses. For a multi-step form, StepperProvider holds the steps and the current step, and StepperNavigator renders Previous and Next (Submit on the last step) inside it.

- **Import:** `import { ProgressBar, Stepper, StepperProvider, StepperNavigator, useStepperContext } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/progress](/showcase/feedback/progress)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `number` | — | ProgressBar — progress value 0–100. Omit for indeterminate. |
| `size` | `'xs' \| 'sm' \| 'md' \| 'lg'` | `'md'` | ProgressBar — bar height. |
| `color` | `'default' \| 'success' \| 'warning' \| 'destructive' \| 'info'` | `'default'` | ProgressBar — fill color. |
| `indeterminate` | `boolean` | `false` | ProgressBar — looping animation when progress is unknown. |
| `steps` | `StepperItem[]` | — | Stepper — array of step objects with label, optional description, and optional status. |
| `currentStep` | `number` | — | Stepper — active step index. Drives completed/active/pending status automatically. |
| `steps (StepperProvider)` | `StepperStep[]` | — | Each step: label, optional description, and an optional onNext() => Promise<boolean> that validates before moving on. Returning false stays on the step. |
| `onSubmit (StepperProvider)` | `() => Promise<boolean>` | — | Runs when Next is pressed on the last step. StepperNavigator shows "Submitting…" meanwhile. |
| `StepperNavigator` | `—` | — | No props. Previous (disabled on the first step) and Next or Submit, read from the StepperProvider context. |

---

#### Skeleton

Animated loading placeholder. Includes preset compositions for text blocks, user rows, stat tiles, cards, and table rows — built on shadcn skeleton.

- **Import:** `import { Skeleton, SkeletonText, SkeletonUser, SkeletonCard, SkeletonTableRow, SkeletonTable, SkeletonStat } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/skeleton](/showcase/feedback/skeleton)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className` | `string` | — | Skeleton — additional CSS classes to set size and shape. |
| `lines` | `number` | `3` | SkeletonText — number of text lines. |
| `rows` | `number` | `5` | SkeletonTable — number of data rows. |

---

#### Rating

Star rating display and input with half-star precision, multiple sizes, and semantic color variants.

- **Import:** `import { Rating } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/rating](/showcase/feedback/rating)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `rating` | `number` | — | Current rating value (0–max). |
| `max` | `number` | `5` | Total number of stars. |
| `size` | `'xs' \| 'sm' \| 'md' \| 'lg' \| 'xl'` | `'md'` | Star size. |
| `color` | `'default' \| 'amber' \| 'orange' \| 'red' \| 'green' \| 'blue' \| 'violet'` | `'default'` | Fill color of active stars. |
| `precision` | `'full' \| 'half'` | `'half'` | Whether to render half-star fills for fractional ratings. |
| `showValue` | `boolean` | `false` | Show numeric value label next to the stars. |
| `onChange` | `(value: number) => void` | — | When provided, enables click-to-rate interaction. |
| `readOnly` | `boolean` | `false` | Disables interaction even when onChange is present. |

---

#### With State

Self-guarding sibling components for a loading/error/empty/normal branch, without a ternary chain or nested `DataTable*Body` trio. Each state checks its own `if` prop independently — usable standalone anywhere, not just inside WithState, and nothing enforces the four are mutually exclusive, so pass non-overlapping conditions yourself. LoadingMessage/ErrorMessage/EmptyMessage are the gate-free default content each *State delegates to — use them directly when you need that default content nested inside your own wrapper (e.g. a Card) under a separately-gated *State, rather than nesting a *State inside itself.

- **Import:** `import { WithState, LoadingState, ErrorState, EmptyState, NormalState, LoadingMessage, ErrorMessage, EmptyMessage } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/with-state](/showcase/feedback/with-state)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `if` | `boolean` | — | Gates whether this branch renders. Required on every state component. |
| `children (WithState)` | `React.ReactNode` | — | The sibling state components to render together — WithState itself is a decorative wrapper with no logic. |
| `children (LoadingState)` | `React.ReactNode` | — | Optional. When given, fully overrides the default loading UI — title/description/icon/className are then ignored. |
| `title (LoadingState)` | `string` | `'Loading…'` | Heading shown in the default loading UI. |
| `description (LoadingState)` | `string` | `'Please wait while we load your data.'` | Body text shown in the default loading UI. |
| `icon (LoadingState)` | `React.ReactNode` | `<LuLoaderCircle className="animate-spin" />` | Icon shown in the default loading UI. |
| `children (ErrorState)` | `React.ReactNode` | — | Optional. When given, fully overrides the default error UI. |
| `error (ErrorState)` | `Error \| null` | — | Source error. Its message becomes the default description when description is not explicitly set. |
| `title (ErrorState)` | `string` | `'Failed to load data'` | Heading shown in the default error UI. |
| `description (ErrorState)` | `string` | `error?.message ?? 'Something went wrong.'` | Body text shown in the default error UI. |
| `icon (ErrorState)` | `React.ReactNode` | `<LuCircleAlert />` | Icon shown in the default error UI. |
| `onRetry (ErrorState)` | `() => void` | — | When given, renders a Retry button beneath the message. |
| `children (EmptyState)` | `React.ReactNode` | — | Optional. When given, fully overrides the default empty UI. |
| `title (EmptyState)` | `string` | `'No results found'` | Heading shown in the default empty UI. |
| `description (EmptyState)` | `string` | `'There are no items to display right now.'` | Body text shown in the default empty UI. |
| `icon (EmptyState)` | `React.ReactNode` | `<LuTextSearch />` | Icon shown in the default empty UI. |
| `className (LoadingState / ErrorState / EmptyState)` | `string` | — | Additional classes on the outer Empty container. Only applies to the default UI — dropped along with the rest of the default when children override it. |
| `children (NormalState)` | `React.ReactNode` | — | Required — there is no generic "success" UI, so real content must always be supplied. |
| `LoadingMessage / ErrorMessage / EmptyMessage` | `component` | — | No `if` and no `children` prop — always render. Take the same title/description/icon/className props as their *State counterpart (plus error/onRetry on ErrorMessage), and are exactly what each *State renders internally when given no children. Use one directly when you need the default content wrapped in something (e.g. Card/CardPanel) while a separate *State (or any other condition) controls whether it shows at all. |

---

#### With Skeleton

Minimal pair for "real content vs. skeleton" — pair SkeletonState with NormalState (from With State) to swap in a loading skeleton without a ternary. Deliberately has no built-in skeleton UI: a skeleton's shape (KPI tile vs. bar chart vs. table row) is always specific to what it's replacing, so both branches' content is supplied by the caller.

- **Import:** `import { WithSkeleton, SkeletonState } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/with-skeleton](/showcase/feedback/with-skeleton)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `if` | `boolean` | — | Gates whether this branch renders. |
| `children (WithSkeleton)` | `React.ReactNode` | — | The SkeletonState/NormalState pair to render together — WithSkeleton itself is a decorative wrapper with no logic. |
| `children (SkeletonState)` | `React.ReactNode` | — | Required — the skeleton markup to show while loading. Pair with NormalState (see With State) for the real-content branch. |

---

#### With Loading

Narrower entry point into With State for a section where loading/normal are the only two states possible — no error, no empty. Pairs LoadingState with NormalState. Same components as With State, reused here so the wrapper's name itself tells a reader not to look for error/empty handling in this block.

- **Import:** `import { WithLoading, LoadingState, NormalState } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/with-loading](/showcase/feedback/with-loading)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `if` | `boolean` | — | Gates whether this branch renders. |
| `children (WithLoading)` | `React.ReactNode` | — | The LoadingState/NormalState pair to render together — WithLoading itself is a decorative wrapper with no logic, identical to WithState/WithSkeleton/WithError. |
| `children (LoadingState)` | `React.ReactNode` | — | Optional — see With State for LoadingState's full prop reference (title/description/icon/className defaults). |
| `children (NormalState)` | `React.ReactNode` | — | Required — there is no generic "success" UI, so real content must always be supplied. |

---

#### With Error

Narrower entry point into With State for a section where error/normal are the only two states possible — no loading, no empty (e.g. a value that's always already available except when a background action fails). Pairs ErrorState with NormalState. Same components as With State, reused here so the wrapper's name itself tells a reader not to look for loading/empty handling in this block.

- **Import:** `import { WithError, ErrorState, NormalState } from "@/ascendra-ui"`
- **Showcase:** [/showcase/feedback/with-error](/showcase/feedback/with-error)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `if` | `boolean` | — | Gates whether this branch renders. |
| `children (WithError)` | `React.ReactNode` | — | The ErrorState/NormalState pair to render together — WithError itself is a decorative wrapper with no logic, identical to WithState/WithSkeleton/WithLoading. |
| `children (ErrorState)` | `React.ReactNode` | — | Optional — see With State for ErrorState's full prop reference (error/title/description/icon/onRetry/className defaults). |
| `children (NormalState)` | `React.ReactNode` | — | Required — there is no generic "success" UI, so real content must always be supplied. |

---

### Forms & Inputs

#### Button

Primary CTA button with a 4-layer shadow and gloss system, available in multiple variants and sizes.

- **Import:** `import { Button } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/button](/showcase/inputs/button)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'primary' \| 'secondary' \| 'destructive' \| 'ghost' \| 'link'` | `'primary'` | Visual style of the button. |
| `size` | `'xs' \| 'sm' \| 'default' \| 'lg' \| 'icon'` | `'default'` | Size of the button. |
| `asChild` | `boolean` | `false` | Render as a child element using Radix Slot. |
| `disabled` | `boolean` | `false` | Disables the button. |

---

#### Input

Text input with smart focus: shadow ring for pointer interactions, outline for keyboard navigation.

- **Import:** `import { Input } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/input](/showcase/inputs/input)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `full` | `boolean` | `false` | Expands input to full container width. |
| `disabled` | `boolean` | `false` | Disables the input. |
| `placeholder` | `string` | — | Placeholder text. |

---

#### Input Group

Composite input system with prefix/suffix text, icon addons, and button addons.

- **Import:** `import { InputGroup, InputGroupAddon, InputGroupButton, InputGroupText, InputGroupInput, InputGroupTextarea } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/input-group](/showcase/inputs/input-group)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `align` | `'inline-start' \| 'inline-end' \| 'block-start' \| 'block-end'` | — | Position of the addon relative to the input (on InputGroupAddon). |

---

#### Checkbox

Custom checkbox with gradient overlay and shadow ring on the checked state.

- **Import:** `import { Checkbox } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/checkbox](/showcase/inputs/checkbox)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `checked` | `boolean \| "indeterminate"` | — | Controlled checked state. |
| `defaultChecked` | `boolean` | — | Uncontrolled default checked state. |
| `disabled` | `boolean` | `false` | Disables the checkbox. |
| `onCheckedChange` | `(checked: boolean) => void` | — | Callback fired when the state changes. |

---

#### Radio Group

Custom radio group with glow and gloss effects on the selected item.

- **Import:** `import { RadioGroup, RadioGroupItem } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/radio-group](/showcase/inputs/radio-group)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string` | — | Controlled selected value. |
| `defaultValue` | `string` | — | Uncontrolled default selected value. |
| `disabled` | `boolean` | `false` | Disables the entire group. |
| `onValueChange` | `(value: string) => void` | — | Callback fired when selection changes. |
| `wrapperClassName` | `string` | — | Additional classes for the item wrapper (on RadioGroupItem). |

---

#### Switch

Toggle switch with a gradient overlay on the active track.

- **Import:** `import { Switch } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/switch](/showcase/inputs/switch)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `checked` | `boolean` | — | Controlled checked state. |
| `defaultChecked` | `boolean` | — | Uncontrolled default checked state. |
| `disabled` | `boolean` | `false` | Disables the switch. |
| `onCheckedChange` | `(checked: boolean) => void` | — | Callback fired when toggled. |

---

#### Select

Custom select dropdown with scroll buttons and size variants.

- **Import:** `import { Select, SelectTrigger, SelectContent, SelectItem, SelectGroup, SelectLabel, SelectValue, SelectSeparator } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/select](/showcase/inputs/select)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `size` | `'sm' \| 'default'` | `'default'` | Size of the trigger (on SelectTrigger). |
| `showClear` | `boolean` | `false` | Show the × clear button in place of the chevron while a value is selected (on SelectTrigger). Backspace/Delete on the focused trigger also clears. |
| `position` | `'popper' \| 'item-aligned'` | `'popper'` | Positioning strategy of the dropdown (on SelectContent). |

---

#### Field

Compound form field wrapper with label, description, error display, and orientation variants.

- **Import:** `import { Field, FieldSet, FieldLegend, FieldGroup, FieldContent, FieldLabel, FieldTitle, FieldDescription, FieldSeparator, FieldError } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/field](/showcase/inputs/field)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `orientation` | `'vertical' \| 'horizontal' \| 'responsive'` | `'vertical'` | Layout direction of the label and control (on Field). |
| `variant` | `'legend' \| 'label'` | `'legend'` | Typography style for FieldLegend. |
| `errors` | `Array<{ message?: string }>` | — | Array of error objects — deduplicates and displays messages (on FieldError). |

---

#### Combobox

Searchable dropdown with single and multi-select (chips) modes, built on Base UI.

- **Import:** `import { Combobox, ComboboxInput, ComboboxContent, ComboboxList, ComboboxItem, ComboboxGroup, ComboboxLabel, ComboboxEmpty, ComboboxSeparator, ComboboxChips, ComboboxChip, ComboboxChipsInput } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/combobox](/showcase/inputs/combobox)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `showTrigger` | `boolean` | `true` | Show the chevron toggle button (on ComboboxInput). |
| `showClear` | `boolean` | `false` | Show the × clear button (on ComboboxInput). |
| `showRemove` | `boolean` | `true` | Show the × remove button on each chip (on ComboboxChip). |
| `disabled` | `boolean` | `false` | Disables the input (on ComboboxInput). |

---

#### Sort Field

Picks the columns to sort by as chips in priority order, each toggling ascending / descending. The value is a list of sort tokens (`amount`, `issuedAt:desc`) — toOrderBy turns it into the API contract's `orderBy` param. Also a QueryDef param type: `type: 'sort'`.

- **Import:** `import { SortField, toOrderBy, parseSortToken, formatSortToken } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/sort-field](/showcase/inputs/sort-field)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string[]` | — | Required. Sort tokens in priority order — 'amount' (ascending) or 'issuedAt:desc'. |
| `onChange` | `(value: string[]) => void` | — | Required. Called with the new tokens when a column is added, removed or has its direction switched. |
| `options` | `{ value: string; label: string }[]` | — | Required. Sortable columns — value is the column name sent to the backend. |
| `maxItems` | `number` | — | Most columns that can be sorted on at once. |
| `placeholder` | `string` | `'Sort by…'` | Shown while no column is chosen. |
| `invalid` | `boolean` | `false` | Shows the invalid outline. |
| `disabled` | `boolean` | `false` | Disables the field. |

---

#### Table Lookup

Table-based lookup field for large datasets — supports single and multi-select with chips, async search, and configurable multi-column display.

- **Import:** `import { TableLookup } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/table-lookup](/showcase/inputs/table-lookup)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `columns` | `TableLookupColumn<T>[]` | — | Column definitions — drives table headers and searchable field list. |
| `valueKey` | `keyof T & string` | — | Key used as the unique identifier for each record. |
| `labelKey` | `keyof T & string` | — | Key whose value is shown in chips and the single-mode trigger. |
| `mode` | `'single' \| 'multiple'` | `'single'` | Single replaces the selection on each pick; multiple accumulates chips. |
| `onSearch` | `(query: string, field?: string) => Promise<T[]> \| T[]` | — | Called to load or filter data. Receives the search string and optional field key. |
| `value` | `T \| T[] \| null` | — | Controlled selected value(s). |
| `onChange` | `(value: T \| T[] \| null) => void` | — | Called when the selection changes. |
| `placeholder` | `string` | — | Placeholder text shown when nothing is selected. |
| `disabled` | `boolean` | `false` | Disables the trigger and prevents the popup from opening. |
| `invalid` | `boolean` | `false` | Applies a destructive outline to the trigger to signal a validation error. |

---

#### File Upload

Fully custom file upload with three variants (dropzone, button, inline) and five controlled states (idle, dragover, uploading, success, error). Includes drag-and-drop, type validation, size limits, and a progress bar.

- **Import:** `import { FileUpload } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/file-upload](/showcase/inputs/file-upload)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'dropzone' \| 'button' \| 'inline'` | `'dropzone'` | Layout variant. |
| `accept` | `string` | — | Comma-separated accepted extensions or MIME types. |
| `maxSize` | `number` | — | Max file size in bytes — validated client-side. |
| `multiple` | `boolean` | `false` | Allow selecting multiple files. |
| `state` | `'idle' \| 'dragover' \| 'uploading' \| 'success' \| 'error'` | `'idle'` | Controlled state override. |
| `uploadProgress` | `number` | — | Progress 0–100 shown during uploading state. |
| `errorMessage` | `string` | — | Error text in error state. |
| `onFiles` | `(files: File[]) => void` | — | Called with validated files when selected. |
| `disabled` | `boolean` | `false` | Disables all interactions. |

---

#### Rich Text Editor

Tiptap-based rich text editor with a 6-action toolbar (bold, italic, strike, bullet list, numbered list, link). Controlled value/onChange API compatible with react-hook-form. Matches input ring and shadow tokens.

- **Import:** `import { RichTextEditor } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/rich-text-editor](/showcase/inputs/rich-text-editor)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string` | — | Controlled HTML content. |
| `onChange` | `(html: string) => void` | — | Called with full HTML on every change. |
| `placeholder` | `string` | `'Write something…'` | Placeholder shown in the empty editor. |
| `readOnly` | `boolean` | `false` | Hides toolbar and disables editing. |
| `minHeight` | `number` | `120` | Min height of the editor area in px. |
| `className` | `string` | — | Additional CSS classes on the container. |

---

#### Color Picker

Fully custom color picker using Radix DropdownMenu as the floating shell — the same token-aligned popover as all other dropdowns. Supports HSL sliders, hex input, and a configurable preset grid.

- **Import:** `import { ColorPicker } from "@/ascendra-ui"`
- **Showcase:** [/showcase/inputs/color-picker](/showcase/inputs/color-picker)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string` | `'#2563eb'` | Controlled hex color value (6-digit). |
| `onChange` | `(hex: string) => void` | — | Called with the new hex when the color changes. |
| `presets` | `string[]` | — | Array of hex preset swatches. Defaults to a 30-color system palette. |
| `presetsOnly` | `boolean` | `false` | Hide sliders and hex input — show only the preset grid. |
| `disabled` | `boolean` | `false` | Prevents opening the picker. |
| `className` | `string` | — | Additional CSS classes on the trigger button. |

---

### Date & Time

#### Calendar

Full-featured calendar built on react-day-picker. Supports single, multiple, and range selection with optional month/year dropdowns.

- **Import:** `import { Calendar } from "@/ascendra-ui"`
- **Showcase:** [/showcase/date/calendar](/showcase/date/calendar)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `fixedWeeks` | `boolean` | `true` | Always draw 6 week rows (padding with next-month days), so the height never changes while paging months — a popover opened above its field keeps its arrows under the cursor. Also on in DatePicker and DateRangePicker. |
| `mode` | `'single' \| 'multiple' \| 'range'` | — | Selection mode. |
| `selected` | `Date \| Date[] \| DateRange \| undefined` | — | Controlled selected value. |
| `onSelect` | `function` | — | Called when the selection changes. |
| `showOutsideDays` | `boolean` | `true` | Show days from adjacent months. |
| `captionLayout` | `'label' \| 'dropdown' \| 'dropdown-months' \| 'dropdown-years'` | — | Caption style. Defaults to 'dropdown' when startMonth/endMonth are set. |
| `startMonth` | `Date` | — | Earliest month available — also enables the dropdown caption. |
| `endMonth` | `Date` | — | Latest month available. |
| `numberOfMonths` | `number` | `1` | Number of months to display side-by-side. |

---

#### Date Picker

Popover-based single date picker with a trigger button and an embedded Calendar.

- **Import:** `import { DatePicker } from "@/ascendra-ui"`
- **Showcase:** [/showcase/date/date-picker](/showcase/date/date-picker)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `Date \| undefined` | — | Controlled selected date. |
| `onChange` | `(date: Date \| undefined) => void` | — | Called when the date changes. |
| `placeholder` | `string` | `'Pick a date'` | Trigger button placeholder text. |
| `disabled` | `boolean` | `false` | Disables the trigger button. |
| `invalid` | `boolean` | `false` | Applies destructive outline to the trigger. |
| `fromYear` | `number` | — | Enables year/month dropdown starting from this year. |
| `toYear` | `number` | — | Enables year/month dropdown ending at this year. |
| `showClear` | `boolean` | `false` | Show a × while a value is set; clicking it (or Backspace/Delete on the trigger) clears to undefined without opening the calendar. The query panel turns it on for optional fields. |
| `minDate / maxDate` | `Date` | — | Earliest / latest selectable day — days outside are disabled. |
| `editable` | `boolean` | `false` | Adds a pencil button: the field looks and opens the calendar as usual, and the pencil (or typing a digit on the focused field) switches to a dd/mm/yyyy text box, read on blur/Enter (unreadable text reverts, empty clears, Escape cancels). A range is typed in the same single field. |
| `dateFormat` | `string` | `'dd/MM/yyyy'` | date-fns pattern of the editable text box (display mode keeps the friendly format); /, - and . separators are all accepted. |
| `captionLayout` | `CalendarProps['captionLayout']` | — | Caption style passed to the inner Calendar. |

---

#### Date Range Picker

Popover-based date range picker with configurable month count and an embedded range Calendar.

- **Import:** `import { DateRangePicker } from "@/ascendra-ui"`
- **Showcase:** [/showcase/date/date-range-picker](/showcase/date/date-range-picker)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `DateRange \| undefined` | — | Controlled selected range ({ from, to }). |
| `onChange` | `(range: DateRange \| undefined) => void` | — | Called when the range changes. |
| `placeholder` | `string` | `'Pick a date range'` | Trigger button placeholder text. |
| `numberOfMonths` | `number` | `2` | Number of calendar months shown in the popover. |
| `disabled` | `boolean` | `false` | Disables the trigger button. |
| `invalid` | `boolean` | `false` | Applies destructive outline to the trigger. |
| `fromYear` | `number` | — | Enables year/month dropdown starting from this year. |
| `toYear` | `number` | — | Enables year/month dropdown ending at this year. |
| `showClear` | `boolean` | `false` | Show a × while a value is set; clicking it (or Backspace/Delete on the trigger) clears to undefined without opening the calendar. The query panel turns it on for optional fields. |
| `minDate / maxDate` | `Date` | — | Earliest / latest selectable day — days outside are disabled. |
| `editable` | `boolean` | `false` | Adds a pencil button: the field looks and opens the calendar as usual, and the pencil (or typing a digit on the focused field) switches to a dd/mm/yyyy text box, read on blur/Enter (unreadable text reverts, empty clears, Escape cancels). A range is typed in the same single field. |
| `dateFormat` | `string` | `'dd/MM/yyyy'` | date-fns pattern of the editable text box (display mode keeps the friendly format); /, - and . separators are all accepted. |
| `maxRangeDays` | `number` | — | Longest range, counting both ends — once a start is picked, days beyond it are disabled. |

---

### Navigation

#### Anchor

Styled link component with semantic color variants and keyboard focus states.

- **Import:** `import { Anchor } from "@/ascendra-ui"`
- **Showcase:** [/showcase/nav/anchor](/showcase/nav/anchor)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'primary' \| 'blue' \| 'muted'` | `'primary'` | Color variant. |
| `href` | `string` | — | Link destination URL. |

---

#### Nav

Sticky horizontal navigation bar with muted background and horizontally scrollable content area.

- **Import:** `import { Nav } from "@/ascendra-ui"`
- **Showcase:** [/showcase/nav/nav](/showcase/nav/nav)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### Nav Link

Navigation link with an active underline indicator driven by the current pathname.

- **Import:** `import { NavLink, NavLinkBadge } from "@/ascendra-ui"`
- **Showcase:** [/showcase/nav/nav-link](/showcase/nav/nav-link)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `href` | `string` | — | Link destination URL. |
| `exact` | `boolean` | `false` | Only active when the path matches exactly. |

---

#### Header

Top navigation header with breadcrumb links, slash separators, chevrons, and action slots.

- **Import:** `import { Header, HeaderLink, HeaderSlash, HeaderChevron, HeaderActions, HeaderLinks } from "@/ascendra-ui"`
- **Showcase:** [/showcase/nav/header](/showcase/nav/header)

**Props**

_No custom props — accepts standard HTML attributes._

---

### Overlays

#### Dialog

Modal dialog with header, body, and footer slots. Centered overlay with max-w-sm default.

- **Import:** `import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogBody, DialogFooter, DialogClose } from "@/ascendra-ui"`
- **Showcase:** [/showcase/overlay/dialog](/showcase/overlay/dialog)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `showCloseButton` | `boolean` | `false` | Show the × close button in the top-right corner (on DialogContent). |
| `footer` | `ReactNode` | — | Footer content slot (on DialogContent). |

---

#### Sheet

Slide-out drawer panel that opens from any edge of the screen.

- **Import:** `import { Sheet, SheetTrigger, SheetContent, SheetHeader, SheetTitle, SheetDescription, SheetBody, SheetFooter, SheetClose, SheetNavigator } from "@/ascendra-ui"`
- **Showcase:** [/showcase/overlay/sheet](/showcase/overlay/sheet)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `index (SheetNavigator)` | `number` | — | Zero-based position of the item the sheet shows. Renders "‹ 3 of 10 ›" beside the close button. |
| `count (SheetNavigator)` | `number` | — | Number of navigable items. Previous/next disable at either end. |
| `onIndexChange (SheetNavigator)` | `(index: number) => void` | — | Called with index ± 1. |
| `keyboard (SheetNavigator)` | `boolean` | `true` | ArrowUp/ArrowDown step while mounted (ignored inside inputs). For a DataTable, use DataTableSheetNavigator (selectedId, onSelect) — it steps through the rows the table is showing. |
| `side` | `'top' \| 'right' \| 'bottom' \| 'left'` | `'right'` | Edge the sheet slides in from (on SheetContent). |
| `showCloseButton` | `boolean` | `true` | Show the × close button (on SheetContent). |
| `lockScroll` | `boolean` | `true` | false keeps the page behind scrollable (on Sheet). An outside click still only closes the sheet — a transparent layer takes it, so the row/button underneath never fires. Use for inspector-style detail sheets; keep the default for forms. |

---

#### Dropdown Menu

Full-featured dropdown with items, separators, checkboxes, radio groups, and sub-menus.

- **Import:** `import { DropdownMenu, DropdownMenuTrigger, DropdownMenuContent, DropdownMenuItem, DropdownMenuSeparator, DropdownMenuLabel, DropdownMenuCheckboxItem, DropdownMenuRadioItem, DropdownMenuSub, DropdownMenuSubTrigger, DropdownMenuSubContent, DropdownMenuShortcut } from "@/ascendra-ui"`
- **Showcase:** [/showcase/overlay/dropdown-menu](/showcase/overlay/dropdown-menu)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'destructive'` | `'default'` | Color variant for DropdownMenuItem. |
| `inset` | `boolean` | — | Add left padding to align items without icons (on DropdownMenuItem). |
| `modal` | `boolean` | `false` | true locks page scroll and blocks the page behind while open (on DropdownMenu). Off by default so the page keeps scrolling; an outside click then closes the menu and also reaches the element under it. |

---

#### Tooltip

Floating label that appears on hover or focus. Wraps Radix UI Tooltip with TooltipProvider, all four placements, and rich content support.

- **Import:** `import { Tooltip, TooltipTrigger, TooltipContent, TooltipProvider } from "@/ascendra-ui"`
- **Showcase:** [/showcase/overlay/tooltips](/showcase/overlay/tooltips)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `side` | `'top' \| 'right' \| 'bottom' \| 'left'` | `'top'` | Which side the tooltip appears on (on TooltipContent). |
| `sideOffset` | `number` | `4` | Distance in px between trigger and tooltip (on TooltipContent). |
| `align` | `'start' \| 'center' \| 'end'` | `'center'` | Alignment relative to the trigger (on TooltipContent). |
| `delayDuration` | `number` | `700` | Hover delay in ms before showing (on TooltipProvider). |

---

#### Command Palette

Keyboard-first command palette dialog built on shadcn Command + Dialog. Groups commands with icons and shortcuts. Registers ⌘K/Ctrl+K via useCommandPalette hook.

- **Import:** `import { CommandPalette, useCommandPalette } from "@/ascendra-ui"`
- **Showcase:** [/showcase/overlay/command-palette](/showcase/overlay/command-palette)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `open` | `boolean` | — | Controlled open state. |
| `onOpenChange` | `(open: boolean) => void` | — | Called when open state changes. |
| `groups` | `CommandPaletteGroup[]` | — | Array of grouped command items. |
| `placeholder` | `string` | `'Search commands…'` | Search input placeholder. |
| `emptyMessage` | `string` | `'No results found.'` | Message shown when no items match. |

---

### Reports

#### Report Header

Composable report title zone. Combines an eyebrow label, bold heading, subtitle, and a metadata footer that renders as an inline row or a multi-column stacked grid for dense field sets.

- **Import:** `import { ReportHeaderContent, ReportHeaderBody, ReportHeaderBodyWrap, ReportTitle, ReportTitleHeader, ReportSubTitle, ReportHeaderFooter, ReportHeaderField } from "@/ascendra-ui"`
- **Showcase:** [/showcase/report-ui/report-header](/showcase/report-ui/report-header)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className (ReportHeaderContent)` | `string` | — | Override the default p-8 pb-6 padding on the header container. |
| `className (ReportHeaderBody)` | `string` | — | Override the flex row that positions the title group and any right-side content (logo, badge). |
| `className (ReportHeaderBodyWrap)` | `string` | — | Override the wrapper around ReportTitle, ReportTitleHeader, and ReportSubTitle. |
| `className (ReportTitle)` | `string` | — | Override the uppercase tracking-widest eyebrow label styles. |
| `className (ReportTitleHeader)` | `string` | — | Override the text-3xl font-bold h1 heading styles. |
| `className (ReportSubTitle)` | `string` | — | Override the mt-1 text-sm text-muted-foreground subtitle styles. |
| `stacked (ReportHeaderFooter)` | `boolean` | `false` | When true, renders fields as a responsive 2–3 column grid instead of an inline flex row. Use when there are 5 or more metadata fields. |
| `className (ReportHeaderFooter)` | `string` | — | Override the metadata bar container. Default is mt-5 border-t pt-4 text-xs with flex or grid layout. |
| `label (ReportHeaderField)` | `string` | — | Field label. Displayed inline as "Label: value" in row mode, or above the value in stacked mode. |
| `className (ReportHeaderField)` | `string` | — | Override individual field styles. |

---

#### Report Document

Document shell and structural dividers. ReportDocumentWrapper is the root element targeted by the PDF export (id="report-content"). ReportSectionHeader separates content zones. ReportDocumentFooter anchors legal notices and attribution.

- **Import:** `import { ReportDocumentWrapper, ReportSectionHeader, ReportDocumentFooter, ReportDocumentFooterNote, ReportDocumentFooterLine, ReportDocumentFooterLineLeft, ReportDocumentFooterLineRight } from "@/ascendra-ui"`
- **Showcase:** [/showcase/report-ui/report-document](/showcase/report-ui/report-document)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className (ReportDocumentWrapper)` | `string` | — | Override the flex flex-col gap-10 root container. Note: id="report-content" is hardcoded — the PDF export button targets this id. |
| `className (ReportSectionHeader)` | `string` | — | Override the pb-3 border-b mb-4 section divider. Children are rendered inside — typically a heading or heading + badge. |
| `className (ReportDocumentFooter)` | `string` | — | Override the border-t pt-6 text-xs text-muted-foreground footer container. |
| `className (ReportDocumentFooterNote)` | `string` | — | Override individual disclaimer or note paragraph styles. |
| `className (ReportDocumentFooterLine)` | `string` | — | Override the flex row that holds attribution text and metadata chips. |
| `className (ReportDocumentFooterLineLeft)` | `string` | — | Override the left-side organization/author label styles. |
| `className (ReportDocumentFooterLineRight)` | `string` | — | Override the right-side metadata group (version, classification, date). |

---

#### Report Content

In-body text blocks for executive summaries, analyst notes, and methodology disclosures. ReportPdfExportButton captures the report document and downloads it as a paginated A4 PDF.

- **Import:** `import { ReportSummary, ReportNotes, ReportNote, ReportNoteHeader, ReportNoteText, ReportPdfExportButton } from "@/ascendra-ui"`
- **Showcase:** [/showcase/report-ui/report-content](/showcase/report-ui/report-content)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className (ReportSummary)` | `string` | — | Override the flex-col gap-3 text-sm leading-relaxed text-muted-foreground summary wrapper. |
| `className (ReportNotes)` | `string` | — | Override the flex-col gap-5 notes container. |
| `className (ReportNote)` | `string` | — | Override an individual note block wrapper. |
| `className (ReportNoteHeader)` | `string` | — | Override the mb-1 font-medium text-foreground note title paragraph. |
| `className (ReportNoteText)` | `string` | — | Override the note body paragraph. |
| `fileName (ReportPdfExportButton)` | `string` | — | PDF filename without the .pdf extension. Required. |
| `title (ReportPdfExportButton)` | `string` | `'Export PDF'` | Button label shown when idle. |

---

### Tables & Data

#### Table

Data table with header, body, footer, scrollable container, and optional accent border and gradient background.

- **Import:** `import { Table, TableWrapper, TableHeader, TableHeaderRow, TableBody, TableRow, TableHead, TableCell, TableFooter, TableFoot, EmptyBody } from "@/ascendra-ui"`
- **Showcase:** [/showcase/data-table/table](/showcase/data-table/table)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `scrollable` | `boolean` | `false` | Enables scroll on the table container. |
| `vertical` | `boolean` | `false` | Enables vertical (y-axis) scroll. |
| `horizontal` | `boolean` | `true` | Enables horizontal (x-axis) scroll. |
| `height` | `number` | — | Max height in px when vertical scroll is enabled. |
| `minHeight` | `number` | — | Minimum height in px for the table container. |
| `visibleRows` | `number` | — | Shows this many body rows whole, then scrolls vertically — measured, so rows of any height fit. Needs no scrollable/vertical and takes over from height. No cap while there are this many rows or fewer. Also on DataTable. |
| `size (TableHeaderRow)` | `'default' \| 'lg'` | `'default'` | default: fixed 40px row of column labels. lg: 16px vertical padding, height from content — for a title + subtitle header. The body card always starts under the measured header. |
| `border (TableWrapper)` | `BorderConfig` | — | Optional accent border on the outer wrapper. |
| `border (TableBody)` | `BorderConfig` | — | Optional accent border on the ::before panel element. |
| `bg (TableBody)` | `BgConfig` | — | Optional gradient background on the ::before panel. Removes solid bg when set. |
| `border.side` | `'t' \| 'l' \| 'r' \| 'b'` | `'t'` | Side the border appears on. |
| `border.stroke` | `1 \| 2 \| 3` | `3` | Border thickness in px. |
| `border.color` | `'blue' \| 'amber' \| 'purple' \| 'red' \| 'teal' \| 'orange' \| 'indigo' \| 'slate'` | `'orange'` | Accent color at 60% opacity. |
| `bg.style` | `'linear' \| 'radial' \| 'conic'` | `'linear'` | CSS gradient type. |
| `bg.side` | `'t' \| 'l' \| 'r' \| 'b'` | `'b'` | Gradient direction (linear only). |
| `bg.color` | `'blue' \| 'amber' \| 'purple' \| 'red' \| 'teal' \| 'orange' \| 'indigo' \| 'slate'` | `'orange'` | Start color at 7% opacity. |
| `bg.to` | `'transparent' \| 'white' \| 'black' \| AccentColor` | `'transparent'` | End stop color. Accent colors resolve to 500/60. |

---

#### Simple Table

Light read-only table for short lists in a sheet, card or panel — one border, a muted header strip, divider lines between rows. No provider.

- **Import:** `import { SimpleTable, SimpleTableHeader, SimpleTableHead, SimpleTableBody, SimpleTableRow, SimpleTableCell } from "@/ascendra-ui"`
- **Showcase:** [/showcase/data-table/simple-table](/showcase/data-table/simple-table)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className` | `string` | — | On every part — merged over the built-in styles (e.g. "text-right" on a numeric head and cell). SimpleTable passes it to the <table>. |
| `...props` | `native element props` | — | Each part takes its element's own props: <table>, <thead>, <th>, <tbody>, <tr>, <td> (e.g. colSpan). |

---

#### Empty State

Placeholder for empty content areas with optional icon media and call-to-action.

- **Import:** `import { Empty, EmptyHeader, EmptyTitle, EmptyDescription, EmptyContent, EmptyMedia } from "@/ascendra-ui"`
- **Showcase:** [/showcase/data-table/empty](/showcase/data-table/empty)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'icon'` | `'default'` | Media display style (on EmptyMedia). |

---

#### Data Table

Feature-rich data table with built-in search, filtering, sorting, column management, and pagination — composed via DataTableProvider and QueryProvider.

- **Import:** `import { DataTable, DataTableHeader, DataTableHeaderRow, DataTableHead, DataTableBody, DataTableRow, DataTableCell, DataTableHighlight, DataTableWrapper, DataTableFoot, DataTableLoadingBody, DataTableEmptyBody, DataTableSearchInput, DataTableColumnManager, DataTableSortDropdown, DataTableFilterDropdown, DataTableFilterBar, DataTableFilterItem, DataTableSortIcon, DataTableCheckboxCell, DataTableCheckboxHead, DataTableErrorBody, DataTableBar, DataTableBarAction, DataTableBarContent, DataTableExportButton, DataTableSheetNavigator, DataTableHeadAction, DataTableHeadActionItem, DataTableExportHeadAction, DataTableRefreshHeadAction, DataTableManageColumnsHeadAction, DataTableBulkExportHeadAction, DataTableBulkDeleteHeadAction, DataTableRowAction, DataTableRowActionItem, DataTableEditRowAction, DataTableDuplicateRowAction, DataTableDeleteRowAction, DataTableViewRowAction, DataTableCopyValueAction, DataTableRunQueryAction, BatchNavigator, ManageQueriesDialog, QueryBar, QueryFieldRenderer, QueryParamPanel, SaveQueryDialog } from "@/ascendra-ui"`
- **Showcase:** [/showcase/data-table](/showcase/data-table)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `key` | `keyof T` | — | Data property key for this column. |
| `label` | `string` | — | Column header label. |
| `type` | `'string' \| 'number' \| 'date'` | `'string'` | Data type — drives sort order and search formatting. |
| `sortable` | `boolean` | `true` | Set to false to remove the sort icon and click handler. |
| `freeze` | `boolean` | `false` | Always visible; cannot be hidden via the column manager. |
| `locked` | `boolean` | `false` | Column position cannot be changed by reordering. |
| `active` | `boolean` | `true` | Whether the column is visible by default. |
| `filter` | `boolean` | `false` | Whether the column appears in the filter picker. |
| `align` | `'left' \| 'right'` | `'left'` | Right-aligns the header (label and sort icon) and every cell — for counts and amounts, so the digits line up. |
| `displayValue` | `(raw: string) => string` | — | Converts raw data values to labels in filter chips and options. |

---

### Layout

#### App Shell & Page Layout

The frame every app is built in. Shell: PageLayout > Header, Nav, MainContainer > (SideBarToggle, SideBar, ContentArea > the page). A page inside ContentArea: PageHeader, then PageMain > PageWrapper > MainContent. PageContent sits below a PageBar and lays out main content beside an AsideContent. DashboardContent stacks dashboard rows. See the Layout Guide showcase page for the four page recipes.

- **Import:** `import { PageLayout, MainContainer, ContentArea, PageMain, PageWrapper, MainContent, PageContent, DashboardContent } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout-guide](/showcase/layout-guide)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `PageLayout` | `div props` | — | The root grid (min-h-screen; rows: Header, Nav, content). Applies the device's saved layout mode (centered, full width or menu) before first paint. |
| `MainContainer` | `main props` | — | The centered container under the top bars, holding the sidebar beside the content. Drops its max width in the full-width and menu layout modes. |
| `ContentArea` | `section props` | — | The flexible column beside the sidebar where each page renders (flex-1 min-w-0). |
| `PageWrapper` | `div props` | — | Wraps a page's main content with a separator line above it, hidden when the page holds a Stepper. |
| `MainContent` | `main props` | — | The page's main column: a full-width flex column with gap-6. |
| `PageContent` | `div props` | — | A row below a PageBar (no top margin when it directly follows one) that places main content beside an AsideContent from lg up. |
| `DashboardContent` | `div props` | — | A flex column with gap-4 for a dashboard's stacked rows of panels. |

---

#### Card

Settings-style card with header, collapsible panel, panel items, field, crown badge, and footer. Supports accent border and gradient background on the panel. A card's body is one of three: CardPanel (padded content), CardTable (a Table in place of a panel: the card header above, the table's own column header and rows, a CardFooter below; it folds with the card, and maximized the table scrolls its rows under a sticky column header — put the Table and its loading/empty/error bodies in it), or a CardList inside a CardPanel (a one-list section such as "top sources": rows that look and behave like a table's, instead of a table whose header row carries the title).

- **Import:** `import { Card, CardHeader, CardHeaderTitle, CardHeaderSubtitle, CardPanel, CardFooter, CardFooterIcon, CardPanelItem, CardPanelItemGroup, CardPanelItemCrown, CardPanelField, CardTable, CardList, CardListItem, CardListLink } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout/card](/showcase/layout/card)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `className (CardTable)` | `string` | — | Extra classes on the CardTable body. No other props: it reads collapse and maximize from the enclosing Card. |
| `columns (CardList)` | `string` | — | A CSS grid template, such as "auto auto 1fr". Each row's children line up in these columns across the list, as table cells would. Without it, each row is a flex line. |
| `visibleRows (CardList)` | `number` | — | Shows that many rows whole, then scrolls, measured like Table's visibleRows. Ignored while the card is maximized, when every row shows. |
| `opensSheet (CardListItem)` | `boolean` | — | The row opens a sheet through its own onClick; the hand cursor says so. A row is otherwise a list item with a separator below it and the table's hover tint. |
| `CardListLink` | `TableRowLink props` | — | Makes the whole row a real link (status-bar URL, cmd/middle-click, Tab focus). Same props as TableRowLink. |
| `danger` | `boolean` | `false` | Applies red danger styling (on Card). |
| `collapseable` | `'expanded' \| 'collapsed'` | — | Enables collapse — a click on the CardHeader folds it; a chevron shows the state. |
| `maximizable` | `boolean` | `false` | A maximize toggle in CardHeader: the card fills the screen over a light overlay; Esc or an outside click restores it. Maximized, page links in rows and charts restore the card instead of navigating. |
| `hasError` | `boolean` | `false` | Shows a destructive outline ring when collapsed (on Card). |
| `border` | `BorderConfig` | — | Optional accent border on Card or CardPanel. |
| `bg` | `BgConfig` | — | Optional gradient background on CardPanel. |
| `collapsed` | `boolean` | — | Collapses the item/field to zero height (on CardPanelItem / CardPanelField). |
| `variant` | `'default' \| 'secondary' \| 'destructive' \| 'success' \| 'warning'` | `'default'` | Crown badge color (on CardPanelItemCrown). |
| `nudge` | `boolean` | `false` | Attention-pulse animation (on CardPanelItemCrown). |
| `nudgeVariant` | `'default' \| 'secondary' \| 'destructive' \| 'success' \| 'warning'` | `'warning'` | Nudge color override (on CardPanelItemCrown). |

---

#### Page Header

Page-level header with title, subtitle, and an action slot. Composes PageHeaderGroup, PageTitle, PageSubtitle, and PageHeaderAction. Focus mode: wrap PageHeader + PageMain in PageFocus and put PageFocusToggle last in PageHeaderAction — the page fills the screen (with one DataTable, compact: only its rows scroll, the footer stays at the bottom); links that leave the page restore it first.

- **Import:** `import { PageHeader, PageHeaderGroup, PageTitle, PageSubtitle, PageHeaderAction, PageFocus, PageFocusToggle, BackLink, PageBackButton, useBackTo } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout/page-header](/showcase/layout/page-header)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `href (BackLink)` | `string` | — | The page's parent. BackLink always goes there: it is the page's way up, shown above the page header with an arrow. |
| `children (BackLink)` | `React.ReactNode` | — | What the link says, such as "Back to people". |
| `label (BackLink)` | `string` | — | The parent's name, such as "People". When the page was opened by a cross-link, the title shows its own back icon, and BackLink then shows this label without an arrow, so only one back arrow is on screen. |
| `className (BackLink)` | `string` | — | Extra classes on the link. |
| `PageBackButton` | `—` | — | The back icon beside the page title. Rendered by PageTitle itself, only when the page was opened by a cross-link built with useBackTo, and not while the page is focused. It returns to where the user came from, with that page's filters as they were. |
| `useBackTo(label)` | `(label: string) => backTo(href)` | — | For a link that leaves this page for another section's page: backTo(href) adds this page's URL and the label to href, so that page shows a back icon reading "Back to {label}". |

---

#### Page Bar

Horizontal toolbar for search, filters, and primary actions above page content. TableBar is the same row (stacked on small screens, side by side from sm) for a toolbar directly above a table.

- **Import:** `import { PageBar, PageBarContent, PageBarAction, TableBar } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout/page-bar](/showcase/layout/page-bar)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### Aside Content

Aside panel with optional dimmed state and gradient mask for overflow content on mobile.

- **Import:** `import { AsideContent } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout/aside-content](/showcase/layout/aside-content)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `dimmed` | `boolean` | `false` | Applies a dim overlay on the aside. |

---

#### Item

Flexible list item with media, title, description, actions, header, and footer slots. Supports outline and muted variants.

- **Import:** `import { Item, ItemMedia, ItemContent, ItemTitle, ItemDescription, ItemActions, ItemGroup, ItemSeparator, ItemHeader, ItemFooter } from "@/ascendra-ui"`
- **Showcase:** [/showcase/layout/item](/showcase/layout/item)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'outline' \| 'muted'` | `'default'` | Border and background style of the item (on Item). |
| `size` | `'default' \| 'sm' \| 'xs'` | `'default'` | Padding and gap size (on Item). |
| `asChild` | `boolean` | `false` | Render as a child element using Radix Slot (on Item). |
| `variant` | `'default' \| 'icon' \| 'image'` | `'default'` | Media display type (on ItemMedia). |

---

### Tabs

#### Tabs

Tabbed navigation with dirty-state dot indicator and disabled tab support.

- **Import:** `import { Tabs, TabList, TabTrigger, TabContent } from "@/ascendra-ui"`
- **Showcase:** [/showcase/tabs](/showcase/tabs)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `defaultValue` | `string` | — | Initially active tab value (uncontrolled). |
| `value` | `string` | — | Controlled active tab value. |
| `onValueChange` | `(value: string) => void` | — | Callback when the active tab changes. |
| `disabled` | `boolean` | `false` | Disable a tab trigger (on TabTrigger). |
| `dirty` | `boolean` | `false` | Show unsaved-changes dot on a tab trigger (on TabTrigger). |

---

#### Segment Control

One choice out of a few, all visible at once — a period (Past hour / 24 hours / 7 days), a view mode. A radio group: the chosen segment is raised on a muted track; arrow keys move between segments. For switching whole panels of content, use Tabs.

- **Import:** `import { SegmentControl, SegmentControlItem } from "@/ascendra-ui"`
- **Showcase:** [/showcase/segment-control](/showcase/segment-control)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `segments` | `{ value: string; label: ReactNode; disabled?: boolean }[]` | — | The segments, in order — each rendered as a SegmentControlItem. Omit to compose SegmentControlItem children instead (icons, counts). |
| `value` | `string` | — | The chosen segment (controlled). |
| `defaultValue` | `string` | — | The initially chosen segment (uncontrolled). Defaults to the first of segments. |
| `onValueChange` | `(value: string) => void` | — | Called with the newly chosen segment — not when the chosen one is clicked again. |
| `size` | `'sm' \| 'default'` | `'default'` | Segment height: 24px or 28px. |
| `disabled` | `boolean` | `false` | Disables every segment (on SegmentControl), or one (on SegmentControlItem). |
| `value` | `string` | — | This segment’s value (on SegmentControlItem, required). |
| `aria-label` | `string` | `'Options'` | Names the group for screen readers — say what the choice is about, e.g. "Period" (on SegmentControl). |

---

### Sidebar

#### Sidebar Menu

Expandable sidebar navigation with grouped menu sets, icon headers, active link detection, and an optional SideBarSearch wrapper that filters menu items by name as you type. SideBarLayoutToggle (first inside SideBar) lets the user cycle the large-screen layout — centered → full width → menu (the slide-in menu) — remembered per device.

- **Import:** `import { SideBar, useSideBar, SideBarFooter, SideBarHeader, SideBarMain, SideBarOverlay, SideBarSearch, SideBarToggle, SideBarLayoutToggle, SideBarMenu, SideBarMenuHeader, SideBarMenuContent, SideBarMenuItem, SideBarMenuItemGroup, SideBarMenuSet, SideBarMenuSetTitle } from "@/ascendra-ui"`
- **Showcase:** [/showcase/sidebar-menu](/showcase/sidebar-menu)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `basePath` | `string` | — | Base path used for active link detection (on SideBarMenu). |
| `icon` | `IconType` | — | Icon for the menu header (on SideBarMenuHeader). |
| `path` | `string` | — | Link path for the menu item (on SideBarMenuItem). |
| `alternate` | `'default' \| 'stand-alone'` | `'default'` | Layout variant for the menu item. |
| `children (SideBarSearch)` | `ReactNode` | — | The menu tree to search — SideBarSearch walks it to extract searchable SideBarMenuItem entries; renders a filtered flat result list in place of the tree while a query is active. |
| `children (SideBarToggle)` | `ReactNode` | `'Menu'` | Label beside the menu button. |
| `className (SideBarLayoutToggle)` | `string` | — | Applied to the toggle row. The toggle takes no other props — the mode lives on <html data-layout> and in localStorage (setLayoutMode / readLayoutMode for app code). |

---

### Utilities

#### With Copy Text

Inline copy-to-clipboard trigger with icon feedback and an optional tooltip confirmation.

- **Import:** `import { WithCopyText } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/with-copy-text](/showcase/util/with-copy-text)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string` | — | Text to copy to the clipboard. |
| `opaque` | `boolean` | `false` | Show the copy icon without hover (always visible). |
| `timeout` | `number` | `1200` | Duration in ms before resetting the copied state. |
| `showTooltip` | `boolean` | `false` | Show a "Copied" tooltip confirmation on click. |
| `onClick` | `(e: MouseEvent) => void \| Promise<string>` | — | Custom click handler; return a string to override what gets copied. |

---

#### Date Time

Renders formatDateTime(value, options) as text, with an optional hover tooltip revealing the original timestamp as a full ISO string.

- **Import:** `import { DateTime } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/date-time](/showcase/util/date-time)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string \| Date` | — | The timestamp to format and display. |
| `style` | `'short' \| 'medium' \| 'long' \| 'month' \| 'relative'` | `'medium'` | Same as formatDateTime's style option (`'month'` = "Sep 2026"). |
| `time` | `boolean` | `false` | Append time-of-day. Ignored when style is 'relative'. |
| `relativeFallbackDays` | `number` | `7` | Beyond this age, 'relative' falls back to an absolute 'medium' date. |
| `showTooltip` | `boolean` | `false` | Show a hover tooltip with the original timestamp as a full ISO string. |
| `className` | `string` | — | Applied to the rendered span. |

---

#### Export CSV Button

Button that downloads an array of objects as a CSV file on click, via the downloadCsv utility (@/ascendra-ui/utils/common.util). For a fully custom trigger, call downloadCsv (or the pure objectsToCsv, which just returns CSV text) directly instead of using this component.

- **Import:** `import { ExportCsvButton } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/export-csv-button](/showcase/util/export-csv-button)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `data` | `T[]` | — | Rows to export. Columns are the first row's own keys. |
| `exclude` | `(keyof T)[]` | — | Keys to omit from the exported columns. |
| `filename` | `string` | — | Base name — defaults to "export.csv". Either way, a timestamp is always inserted before the extension, e.g. "audit-events-2026-09-27T05-42-10.csv". |
| `title` | `string` | `'Export CSV'` | Button label. Ignored (used as the accessible name instead) when iconOnly is true. |
| `iconOnly` | `boolean` | `false` | Show only the icon, no visible label. |
| `icon` | `React.ReactNode` | — | Overrides the default download icon. |
| `variant` | `'primary' \| 'secondary' \| 'destructive' \| 'ghost' \| 'link'` | `'secondary'` | Same variant prop as Button. |

---

#### Name Avatar

Auto-generated avatar from name initials with a deterministic background color derived from the name.

- **Import:** `import { NameAvatar } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/name-avatar](/showcase/util/name-avatar)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `name` | `string` | — | Full name to generate initials from. |
| `size` | `number` | `28` | Avatar size in pixels. |
| `href` | `string` | — | Optional URL — wraps avatar in a link. |

---

#### Theme Toggle

Dark/light mode toggle button powered by next-themes.

- **Import:** `import { ThemeToggle } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/theme-toggle](/showcase/util/theme-toggle)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### Pagination Button

Small bordered navigation button for pagination controls.

- **Import:** `import { PaginationButton } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/pagination-button](/showcase/util/pagination-button)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### Row Action Button

Icon button for table row actions — hidden by default, visible on row hover.

- **Import:** `import { RowActionButton } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/row-action-button](/showcase/util/row-action-button)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `opaque` | `boolean` | `false` | Show the button without row hover (always visible). |

---

#### Scroll To Top

Behavior-only component that scrolls the window to the top on every route change and disables browser scroll restoration.

- **Import:** `import { ScrollToTop } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/scroll-to-top](/showcase/util/scroll-to-top)

**Props**

_No custom props — accepts standard HTML attributes._

---

#### With Empty Value

Renders children when value is non-empty, or a fallback (default an em dash) when value is null, undefined, or an empty string. Use for a table cell or detail field whose data may be missing — not for loading/error states, see WithState.

- **Import:** `import { WithEmptyValue } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/with-empty-value](/showcase/util/with-empty-value)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `unknown` | — | Tested for emptiness — null, undefined, and an empty string all count as empty. |
| `fallback` | `React.ReactNode` | `'—'` | Rendered when value is empty. |
| `children` | `React.ReactNode` | — | Rendered when value is non-empty. |

---

#### With Sheet Icon

Display only: shows a panel icon after children on hover (on a touch screen, while pressed; or when their clickable parent is keyboard-focused), signalling that a click opens a sheet — the sheet counterpart of an external link's arrow. It adds no click, label or tooltip; wrap the content inside your own button. Space for the icon is always reserved, so nothing shifts.

- **Import:** `import { WithSheetIcon } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/with-sheet-icon](/showcase/util/with-sheet-icon)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `children` | `React.ReactNode` | — | The content the icon follows. |
| `className` | `string` | — | Applied to the wrapping span. |

---

#### With Tooltip

Wraps any children — plain text, multiple elements, a custom component — in a hoverable tooltip, without callers needing to worry about asChild's single-element requirement.

- **Import:** `import { WithTooltip } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/with-tooltip](/showcase/util/with-tooltip)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `tooltip` | `React.ReactNode` | — | Content shown inside the tooltip on hover/focus. |
| `children` | `React.ReactNode` | — | What gets wrapped — any shape, any number of nodes. |
| `className` | `string` | — | Applied to the wrapping span. |

---

#### Id Text

Fixed-monospace, truncated display for a uuid/id-shaped value ("head…tail"), with copy-to-clipboard and a full-value hover tooltip on by default. Composes WithCopyText and WithTooltip rather than reimplementing either.

- **Import:** `import { IdText } from "@/ascendra-ui"`
- **Showcase:** [/showcase/util/id-text](/showcase/util/id-text)

**Props**

| Prop | Type | Default | Description |
|---|---|---|---|
| `value` | `string` | — | The full, untruncated id/uuid value — copied and shown in the tooltip in full, regardless of size. |
| `size` | `'full' \| 'sm' \| 'md' \| 'lg'` | `'full'` | 'full' shows the value as-is; 'sm'/'md'/'lg' truncate to "head…tail" with progressively more characters, kept on one line. |
| `copyable` | `boolean` | `true` | Copy-to-clipboard on click, with icon feedback. |
| `showTooltip` | `boolean` | `true` | Hover tooltip showing the full value. |
| `className` | `string` | — | Applied to the text span — the monospace font itself is fixed. |
| `style` | `React.CSSProperties` | — | Applied to the text span, e.g. to override font-size. |

---

## Part 2 — Composite Patterns

> **These are showcase-only reference patterns — they are not importable components.** Do not attempt to `import { ContactInquiryForm }` or any pattern name from `@/ascendra-ui`. These entries exist to show which primitives compose well together and at what scale. Build your own pages using those primitives directly, following the code templates in `docs/showcase-reference.md`.

> Full-page patterns built from the primitives above. Each showcases a realistic domain scenario.
> Use the **Components used** lists to understand which primitives to reach for.

### Forms

10 form patterns covering SaaS, HR, e-commerce, finance, and general scenarios.

| Name | Domain | Complexity | Layout |
|---|---|---|---|
| [Contact & Inquiry](/showcase/forms/contact-inquiry) | General / Marketing | Simple | Single column, vertical |
| [User Profile Settings](/showcase/forms/user-profile) | SaaS / Product | Medium | 2 sections, mixed vertical & horizontal fields |
| [Support Ticket](/showcase/forms/support-ticket) | IT / SaaS | Medium | Single column with conditional section reveal |
| [Appointment Booking](/showcase/forms/appointment-booking) | Healthcare / Services | Medium | 3 sections, step-like visual flow |
| [Job Application](/showcase/forms/job-application) | HR / Recruitment | Medium | 3 sections, single column with 2-column grid rows |
| [Financial Transaction](/showcase/forms/financial-transaction) | Finance / Banking | Medium | 2 sections, horizontal-label compact layout |
| [Create Product Listing](/showcase/forms/create-product) | E-Commerce | Complex | 3 sections, mixed 1-column and 2-column grid |
| [Project Kickoff](/showcase/forms/project-kickoff) | Project Management | Complex | 3 sections, mixed 1-column and 2-column grids |
| [Search & Filter Panel](/showcase/forms/search-filter) | Universal Utility | Simple | Compact sidebar panel, single column |
| [Employee Onboarding Stepper](/showcase/forms/employee-onboarding) | HR / Enterprise | Complex | 4-step stepper with step indicator and per-step sections |

### Dialogs

12 dialog patterns grouped by intent type. All are rendered on the shared [Dialog Gallery](/showcase/dialogs) page.

| Name | Type | Components |
|---|---|---|
| Archive Project | Confirmation | Dialog, Button |
| Transfer Ownership | Confirmation | Dialog, Checkbox, Button |
| Delete Record | Destructive | Dialog, Button |
| Delete Account | Destructive | Dialog, Input, Field, Button |
| Rename Item | Input | Dialog, Input, Field, Button |
| Add Note | Input | Dialog, InputGroup, Field, Button |
| Invite Member | Input | Dialog, Input, Select, Field, Button |
| Change Password | Input | Dialog, Input, Field, Button |
| Session Expired | Alert | Dialog, Button |
| Payment Failed | Alert | Dialog, SimpleAlert, Button |
| Feature Announcement | Feature | Dialog, Button |
| Upgrade Required | Feature | Dialog, Checkbox, Button |

---

### Sheets

10 slide-out panel patterns for Detail, Preview, Activity, and Settings use-cases. All rendered on the [Sheet Gallery](/showcase/sheets) page.

| Name | Type | Domain | Components |
|---|---|---|---|
| Employee Profile | Detail | HR | Sheet, SimpleBadge, NameAvatar, Switch, Item |
| Order Details | Detail | E-commerce | Sheet, Item, SimpleBadge, Button |
| Support Ticket | Activity | SaaS | Sheet, Item, NameAvatar, StatusDot, SimpleBadge, Button |
| Notification Preferences | Settings | SaaS | Sheet, Switch, Button |
| Customer Profile | Preview | CRM | Sheet, NameAvatar, SimpleBadge, StatusDot, Item, Button |
| Invoice Preview | Preview | Finance | Sheet, Item, SimpleBadge, SimpleAlert, Button |
| Project Overview | Detail | Project Management | Sheet, NameAvatar, SimpleBadge, StatusDot, Item, Button |
| Audit Log Entry | Detail | System / Admin | Sheet, SimpleBadge, SimpleAlert, Button |
| Product Details | Preview | Catalog | Sheet, SimpleBadge, StatusDot, Button |
| Account Settings | Settings | Platform | Sheet, Switch, SimpleBadge, Button |

---

### Drawers

8 bottom-drawer patterns for mobile-first Action, Panel, Preview, and Input flows. All rendered on the [Drawer Gallery](/showcase/drawers) page.

| Name | Type | Domain | Components |
|---|---|---|---|
| Quick Actions | Action | General | Drawer, Item, Button |
| Smart Filter | Panel | E-commerce | Drawer, Switch, Item, SimpleBadge, Button |
| Share Sheet | Action | General | Drawer, Button |
| Event Preview | Preview | Calendar | Drawer, NameAvatar, StatusDot, Button |
| Media Attachment | Preview | File Management | Drawer, Button |
| Assign Task | Input | Project Management | Drawer, Input, NameAvatar, Item, Button |
| Notification Center | Panel | SaaS | Drawer, BubbleBadge, StatusDot, Button |
| Danger Zone | Action | Settings | Drawer, Input, SimpleAlert, Button |

---

### Dashboards

10 full analytics dashboards, each with KPI tiles, multiple chart types, and a data table. Each has its own showcase page.

| Name | Domain | Chart types | KPIs |
|---|---|---|---|
| [SaaS Revenue & Growth](/showcase/dashboards/saas-revenue) | SaaS / Startup | composed, pie, area, radial, bar | Monthly Recurring Revenue, Annual Run Rate, Churn Rate, Net Revenue Retention |
| [E-commerce Operations](/showcase/dashboards/ecommerce-ops) | Retail / E-commerce | line, bar, treemap, histogram | Gross Merchandise Value, Orders Placed, Avg. Order Value, Return Rate |
| [Marketing Performance](/showcase/dashboards/marketing) | Marketing / Growth | bar, radar, area, pie, composed | Attributed Revenue, Blended ROAS, Customer Acq. Cost, Avg. CTR |
| [Financial P&L](/showcase/dashboards/financial-pnl) | Finance / CFO | composed, radial, area, bar | Total Revenue, EBITDA, Monthly Burn Rate, Cash Runway |
| [Trading & Portfolio](/showcase/dashboards/trading-portfolio) | Finance / Trading | candlestick, histogram, scatter, composed | Portfolio Value, Day P&L, Beta, Sharpe Ratio |
| [Healthcare Analytics](/showcase/dashboards/healthcare) | Healthcare / Clinical | line, radial, bar, radar, histogram | Active Patients, Avg. Wait Time, Bed Occupancy, Recovery Rate |
| [HR & People Analytics](/showcase/dashboards/hr-people) | People Operations | line, pie, bar, scatter | Total Headcount, Attrition Rate, Avg. Time to Hire, eNPS Score |
| [DevOps Monitoring](/showcase/dashboards/devops) | Engineering / SRE | area, line, bar, composed, radial | Uptime (30d), P99 Latency, Error Rate, Deploy Frequency |
| [Supply Chain](/showcase/dashboards/supply-chain) | Operations / Logistics | composed, bar, treemap, radar | On-time Delivery, Inventory Fill Rate, Inventory Turns, Avg. Lead Time |
| [Real Estate Portfolio](/showcase/dashboards/real-estate) | Property Investment | treemap, pie, composed, bar, scatter | Portfolio Value, Avg. Gross Yield, Occupancy Rate, Monthly Income |

#### SaaS Revenue & Growth

Monthly recurring revenue, plan-tier breakdown, churn, and net revenue retention — the core metrics for a SaaS growth review.

- **Slug:** `saas-revenue`
- **Showcase:** [/showcase/dashboards/saas-revenue](/showcase/dashboards/saas-revenue)
- **Domain:** SaaS / Startup
- **KPIs:** Monthly Recurring Revenue, Annual Run Rate, Churn Rate, Net Revenue Retention
- **Chart types:** composed, pie, area, radial, bar

**Layout** (12-column grid)

- Row 1: `chart[8]` MRR & Growth Rate  ·  `chart[4]` Plan Mix
- Row 2: `chart[8]` Revenue by Tier  ·  `chart[4]` NRR Gauge
- Row 3: `table[7]` Top Accounts  ·  `chart[5]` Churn by Cohort

---

#### E-commerce Operations

Order volume, GMV, category performance, and return rates — the ops view for a mid-size online retailer.

- **Slug:** `ecommerce-ops`
- **Showcase:** [/showcase/dashboards/ecommerce-ops](/showcase/dashboards/ecommerce-ops)
- **Domain:** Retail / E-commerce
- **KPIs:** Gross Merchandise Value, Orders Placed, Avg. Order Value, Return Rate
- **Chart types:** line, bar, treemap, histogram

**Layout** (12-column grid)

- Row 1: `chart[12]` Daily Orders
- Row 2: `chart[6]` Orders by Category  ·  `chart[6]` GMV by Category
- Row 3: `chart[5]` Order Value Distribution  ·  `table[7]` Top Products

---

#### Marketing Performance

Channel ROAS, traffic mix, and spend efficiency — the CMO view for a performance-marketing team running multi-channel campaigns.

- **Slug:** `marketing`
- **Showcase:** [/showcase/dashboards/marketing](/showcase/dashboards/marketing)
- **Domain:** Marketing / Growth
- **KPIs:** Attributed Revenue, Blended ROAS, Customer Acq. Cost, Avg. CTR
- **Chart types:** bar, radar, area, pie, composed

**Layout** (12-column grid)

- Row 1: `chart[5]` Channel ROAS  ·  `chart[7]` Channel Scorecard
- Row 2: `chart[8]` Traffic by Source  ·  `chart[4]` Spend Allocation
- Row 3: `chart[8]` Spend vs Revenue  ·  `chart[4]` Status Summary
- Row 4: `table[12]` Active Campaigns

---

#### Financial P&L

Revenue, cost breakdown, EBITDA margin trajectory, and budget attainment — the board-ready financial view for a growth-stage company.

- **Slug:** `financial-pnl`
- **Showcase:** [/showcase/dashboards/financial-pnl](/showcase/dashboards/financial-pnl)
- **Domain:** Finance / CFO
- **KPIs:** Total Revenue, EBITDA, Monthly Burn Rate, Cash Runway
- **Chart types:** composed, radial, area, bar

**Layout** (12-column grid)

- Row 1: `chart[7]` Costs & EBITDA Margin  ·  `chart[5]` Budget Attainment
- Row 2: `chart[12]` Revenue vs Expenses
- Row 3: `chart[6]` Cash Flow Waterfall  ·  `table[6]` P&L Summary

---

#### Trading & Portfolio

Price action, volume, return distribution, and risk/return positioning — a complete view for an equity portfolio manager.

- **Slug:** `trading-portfolio`
- **Showcase:** [/showcase/dashboards/trading-portfolio](/showcase/dashboards/trading-portfolio)
- **Domain:** Finance / Trading
- **KPIs:** Portfolio Value, Day P&L, Beta, Sharpe Ratio
- **Chart types:** candlestick, histogram, scatter, composed

**Layout** (12-column grid)

- Row 1: `chart[12]` Price Action _(xl)_
- Row 2: `chart[4]` Return Distribution  ·  `chart[4]` Risk vs Return  ·  `chart[4]` Price + Volume
- Row 3: `table[12]` Positions

---

#### Healthcare Analytics

Patient flow, bed occupancy, department performance, and outcome rates — the clinical director's operational view of a hospital.

- **Slug:** `healthcare`
- **Showcase:** [/showcase/dashboards/healthcare](/showcase/dashboards/healthcare)
- **Domain:** Healthcare / Clinical
- **KPIs:** Active Patients, Avg. Wait Time, Bed Occupancy, Recovery Rate
- **Chart types:** line, radial, bar, radar, histogram

**Layout** (12-column grid)

- Row 1: `chart[8]` Admissions Trend  ·  `chart[4]` Bed Occupancy
- Row 2: `chart[6]` Conditions by Department  ·  `chart[6]` Department Scorecard
- Row 3: `chart[5]` Patient Age Distribution  ·  `table[7]` Recent Cases

---

#### HR & People Analytics

Headcount movement, attrition drivers, salary distribution, and performance spread — the CHRO's workforce view.

- **Slug:** `hr-people`
- **Showcase:** [/showcase/dashboards/hr-people](/showcase/dashboards/hr-people)
- **Domain:** People Operations
- **KPIs:** Total Headcount, Attrition Rate, Avg. Time to Hire, eNPS Score
- **Chart types:** line, pie, bar, scatter

**Layout** (12-column grid)

- Row 1: `chart[7]` Hiring vs Attrition  ·  `chart[5]` Seniority Mix
- Row 2: `chart[12]` Headcount by Department
- Row 3: `chart[6]` Tenure vs Performance  ·  `table[6]` Recent Hires

---

#### DevOps Monitoring

Latency, throughput, error rates by service, and deployment frequency — the reliability dashboard for an engineering team.

- **Slug:** `devops`
- **Showcase:** [/showcase/dashboards/devops](/showcase/dashboards/devops)
- **Domain:** Engineering / SRE
- **KPIs:** Uptime (30d), P99 Latency, Error Rate, Deploy Frequency
- **Chart types:** area, line, bar, composed, radial

**Layout** (12-column grid)

- Row 1: `chart[12]` Request Volume
- Row 2: `chart[6]` P99 Latency  ·  `chart[6]` Errors by Service
- Row 3: `chart[8]` Error Rate & Throughput  ·  `chart[4]` SLA Uptime

---

#### Supply Chain

Inventory flow, supplier concentration, regional shipments, and delivery performance — the operations director's supply view.

- **Slug:** `supply-chain`
- **Showcase:** [/showcase/dashboards/supply-chain](/showcase/dashboards/supply-chain)
- **Domain:** Operations / Logistics
- **KPIs:** On-time Delivery, Inventory Fill Rate, Inventory Turns, Avg. Lead Time
- **Chart types:** composed, bar, treemap, radar

**Layout** (12-column grid)

- Row 1: `chart[7]` Inventory Flow  ·  `chart[5]` Shipments by Region
- Row 2: `chart[12]` Supplier Spend Share
- Row 3: `chart[5]` Supplier Scorecard  ·  `table[7]` Open Orders

---

#### Real Estate Portfolio

Portfolio valuation, yield analysis, occupancy, and individual property performance — the asset manager's property view.

- **Slug:** `real-estate`
- **Showcase:** [/showcase/dashboards/real-estate](/showcase/dashboards/real-estate)
- **Domain:** Property Investment
- **KPIs:** Portfolio Value, Avg. Gross Yield, Occupancy Rate, Monthly Income
- **Chart types:** treemap, pie, composed, bar, scatter

**Layout** (12-column grid)

- Row 1: `chart[7]` Portfolio by Type  ·  `chart[5]` Geographic Allocation
- Row 2: `chart[8]` Rental Income & Yield  ·  `chart[4]` Properties by Value
- Row 3: `chart[5]` Price vs Area  ·  `table[7]` Properties

---

### Reports

10 document-style reports covering finance, HR, clinical, operational, performance, and compliance domains. Each has its own showcase page.

| Name | Domain | Type | Complexity | Layout |
|---|---|---|---|---|
| [Annual Financial Statement](/showcase/reports/annual-financial-statement) | Finance / Accounting | Financial | Complex | Document |
| [Executive Business Review](/showcase/reports/executive-business-review) | Corporate / C-Suite | Executive | Simple | Wide |
| [Patient Health Summary](/showcase/reports/patient-health-summary) | Healthcare / Clinical | Clinical | Medium | Document |
| [Project Status Report](/showcase/reports/project-status-report) | Project Management | Operational | Simple | Mixed |
| [Sales Pipeline Report](/showcase/reports/sales-pipeline-report) | Sales / CRM | Performance | Medium | Wide |
| [Marketing Campaign Analysis](/showcase/reports/marketing-campaign-analysis) | Marketing / Growth | Performance | Medium | Wide |
| [Supply Chain Operations Report](/showcase/reports/supply-chain-ops-report) | Operations / Logistics | Operational | Complex | Wide |
| [Employee Performance Review](/showcase/reports/employee-performance-review) | Human Resources | Performance | Simple | Document |
| [Cybersecurity Incident Report](/showcase/reports/security-incident-report) | IT / Security | Compliance | Medium | Document |
| [ESG Sustainability Report](/showcase/reports/esg-sustainability-report) | Corporate / ESG | Compliance | Complex | Mixed |

#### Annual Financial Statement

Year-end financial statement for a mid-size company — income statement, balance sheet, and cash flow summary in a formal document layout with section dividers, accounting-style tables, and prior-year comparisons.

- **Slug:** `annual-financial-statement`
- **Showcase:** [/showcase/reports/annual-financial-statement](/showcase/reports/annual-financial-statement)
- **Domain:** Finance / Accounting
- **Report type:** Financial
- **Complexity:** Complex
- **Layout:** Document
- **Key metrics:** Total Revenue, Net Income, Total Assets, Operating Cash Flow
- **Elements:** Accounting tables, Variance columns, Branded header, Document footer, Bar chart, Section dividers

---

#### Executive Business Review

CEO-level quarterly business review with large headline KPIs, a revenue trend chart, division performance summary table, and a written executive highlights section — designed for board presentation.

- **Slug:** `executive-business-review`
- **Showcase:** [/showcase/reports/executive-business-review](/showcase/reports/executive-business-review)
- **Domain:** Corporate / C-Suite
- **Report type:** Executive
- **Complexity:** Simple
- **Layout:** Wide
- **Key metrics:** Total Revenue, Gross Profit, EBITDA, Cash Position
- **Elements:** Large KPI tiles, Bar chart, Summary table, Highlights text, Branded header, Board footer

---

#### Patient Health Summary

Clinical visit summary for a patient portal or physician handoff. Shows vital signs, active medications, recent lab results with reference ranges, and structured visit notes — no charts, entirely tables and text.

- **Slug:** `patient-health-summary`
- **Showcase:** [/showcase/reports/patient-health-summary](/showcase/reports/patient-health-summary)
- **Domain:** Healthcare / Clinical
- **Report type:** Clinical
- **Complexity:** Medium
- **Layout:** Document
- **Key metrics:** Age, BMI, Blood Pressure, Last Visit Date
- **Elements:** Patient info strip, Vital signs grid, Medications table, Lab results w/ status, Clinical notes, No charts

---

#### Project Status Report

Weekly project health report with a RAG traffic-light banner, milestone timeline, budget vs. actual chart, and a risk register — giving a PM and stakeholders a single-page view of delivery status.

- **Slug:** `project-status-report`
- **Showcase:** [/showcase/reports/project-status-report](/showcase/reports/project-status-report)
- **Domain:** Project Management
- **Report type:** Operational
- **Complexity:** Simple
- **Layout:** Mixed
- **Key metrics:** % Complete, Days to Deadline, Budget Consumed, Open Risks
- **Elements:** RAG status banner, Milestone timeline, Paired bar chart, Risk register table, Progress indicators

---

#### Sales Pipeline Report

Monthly sales performance report with pipeline stage funnel, rep quota attainment rankings with inline progress bars, bookings vs. target chart, and deal size distribution.

- **Slug:** `sales-pipeline-report`
- **Showcase:** [/showcase/reports/sales-pipeline-report](/showcase/reports/sales-pipeline-report)
- **Domain:** Sales / CRM
- **Report type:** Performance
- **Complexity:** Medium
- **Layout:** Wide
- **Key metrics:** Pipeline Value, Deals Won, Avg. Deal Size, Win Rate
- **Elements:** Funnel visualization, Rep ranking table, Inline progress bars, Composed chart, Bar chart

---

#### Marketing Campaign Analysis

Post-campaign report for a multi-channel digital marketing campaign — channel ROAS comparison, conversion funnel with drop-off annotations, audience breakdown donut, and campaign ROI summary table.

- **Slug:** `marketing-campaign-analysis`
- **Showcase:** [/showcase/reports/marketing-campaign-analysis](/showcase/reports/marketing-campaign-analysis)
- **Domain:** Marketing / Growth
- **Report type:** Performance
- **Complexity:** Medium
- **Layout:** Wide
- **Key metrics:** Total Impressions, Conversions, Blended ROAS, Cost per Conversion
- **Elements:** Grouped bar chart, Conversion funnel, Donut chart, ROI summary table, Drop-off annotations

---

#### Supply Chain Operations Report

Monthly supply chain report with inventory level trends by category, supplier performance scorecard (color-coded by threshold), regional shipment breakdown, and a critical open orders table.

- **Slug:** `supply-chain-ops-report`
- **Showcase:** [/showcase/reports/supply-chain-ops-report](/showcase/reports/supply-chain-ops-report)
- **Domain:** Operations / Logistics
- **Report type:** Operational
- **Complexity:** Complex
- **Layout:** Wide
- **Key metrics:** On-time Delivery, Inventory Value, Stockout Events, Avg. Lead Time
- **Elements:** Multi-series area chart, Supplier scorecard, Colored table cells, Horizontal bar chart, Critical orders table

---

#### Employee Performance Review

Annual employee review document with a prominent overall rating, competency radar chart, goal achievement table with status badges, peer feedback text blocks, a development plan, and a sign-off section.

- **Slug:** `employee-performance-review`
- **Showcase:** [/showcase/reports/employee-performance-review](/showcase/reports/employee-performance-review)
- **Domain:** Human Resources
- **Report type:** Performance
- **Complexity:** Simple
- **Layout:** Document
- **Key metrics:** Overall Rating, Goals Achieved, 360° Score, Tenure
- **Elements:** Radar chart, Rating display, Goal table, Feedback text blocks, Development plan, Sign-off area

---

#### Cybersecurity Incident Report

Formal post-incident report documenting a security event — TLP classification banner, severity badge, incident timeline, affected systems table, impact assessment, and remediation action tracker.

- **Slug:** `security-incident-report`
- **Showcase:** [/showcase/reports/security-incident-report](/showcase/reports/security-incident-report)
- **Domain:** IT / Security
- **Report type:** Compliance
- **Complexity:** Medium
- **Layout:** Document
- **Key metrics:** Severity Level, Systems Affected, Records at Risk, Time to Containment
- **Elements:** TLP classification, Severity badge, Vertical timeline, Affected systems table, Remediation tracker, Bar chart

---

#### ESG Sustainability Report

Annual ESG progress report with three color-themed sections (Environmental, Social, Governance), emissions trend, energy mix donut, diversity charts, board composition, ESG score gauge, and an industry benchmark radar.

- **Slug:** `esg-sustainability-report`
- **Showcase:** [/showcase/reports/esg-sustainability-report](/showcase/reports/esg-sustainability-report)
- **Domain:** Corporate / ESG
- **Report type:** Compliance
- **Complexity:** Complex
- **Layout:** Mixed
- **Key metrics:** Carbon Emissions, Renewable Energy %, Workforce Diversity, Board Independence
- **Elements:** 3-section color themes, Progress bars to targets, Area chart, Donut chart, Radar chart, SDG alignment grid

---
