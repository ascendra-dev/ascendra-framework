# `FieldHint` — Full Component Reference

Companion to `reference/ascendra-ui/hard-instructions.md` AUI-002 and AUI-010. Written from a
full read of the component's actual source (`../ascendra-ui/ascendra-ui/components/ui/field.tsx`)
and every real usage across `../ascendra-ui/components/forms/*.tsx` (~66 usages surveyed). Read
this before using `FieldHint` in any non-trivial way — its behavior has more moving parts, and
more sharp edges, than `docs/ui-reference.md`'s prop table shows.

---

## What it is

`FieldHint` is the single component that renders **everything that goes below a field's
control**: a validation error, an informational description, and/or a row of small badges
(`Mandatory`/`Optional`, a help tooltip). It replaces `FieldError` + `FieldDescription` used
separately (see AUI-002) — one component, several mutually-composable concerns.

## Props

| Prop | Type | Default | What it does |
|---|---|---|---|
| `error` | `{ message?: string }` | — | Validation error. When truthy, **completely replaces** the description text with `error.message`, styled destructive, prefixed with a warning-triangle icon (`LuTriangleAlert`). Pass a react-hook-form `fieldState.error` (or `errors.fieldName`) object directly — its shape already matches. |
| `description` | `string` | non-breaking space (` `) — see "Default value fix" below | Informational hint text. Only shown when `error` is **not** set — `error` always wins; a field can't show both at once, and `description` is silently ignored while an error is present. |
| `mandatory` | `boolean` | — | Renders a small `Mandatory` badge (`SimpleBadge variant="secondary" size="tiny"`). |
| `optional` | `boolean` | — | Renders a small `Optional` badge, same styling. **Mutually exclusive with `mandatory` in rendering**, not just in meaning — see Gotcha 1 below. |
| `help` | `string` | — | Renders a small `?` badge that shows this text in a tooltip on hover (`Tooltip`/`TooltipContent`, `side="bottom"`). Additive — can appear alongside `mandatory`/`optional` at the same time. |
| `className` | `string` | — | Passed to the outer wrapping `<div>`. |
| ...`props` | `React.ComponentProps<"div">` | — | Spread onto the outer `<div>`. |

There is **no `children` prop that does anything** — see Gotcha 2 / AUI-010. Passing children is
silently discarded; the component always renders its own fixed internal structure.

---

## Rule: `FieldHint` is mandatory under every field's control, even when nothing applies

**Never omit `FieldHint` from a field just because it has no error, description, or badge.**
Every `Field`'s control (`Input`, `Select`, `RadioGroup`, etc.) must be followed by a `FieldHint`
— call it bare (`<FieldHint />`) when none of its props apply. This is the dominant real pattern
across the whole library (**45 of ~66 surveyed usages are bare**) — its purpose is purely to
reserve that row's height so every field's total vertical footprint stays consistent, whether or
not it actually has something to show. Skipping `FieldHint` entirely on a field (rather than
calling it bare) is what causes uneven field heights and "space fluctuation" across a form.

A bare call reliably reserves that space today — see "Default value fix" below for why this
didn't used to work and what changed.

---

## Rendering logic, in order

1. **Early exit:** `if (!error && !description && !mandatory && !optional && !help) return null;`
   — with `description` now defaulting to a non-breaking space (see below), this only actually
   triggers `null` if a caller explicitly overrides `description` to `undefined` or `""` *and*
   passes nothing else — not a case that occurs anywhere in real usage.
2. **Text row:** if `error` is set, show `error.message` (destructive, with the warning icon) —
   `description` is never even read in this branch. Otherwise show `description ||  ` (a
   fallback that only matters if you explicitly pass `description=""`, since the empty string is
   falsy but was still explicitly provided).
3. **Badge row** (only rendered at all if `mandatory || optional || help`):
   - `mandatory`/`optional`: `{mandatory ? "Mandatory" : "Optional"}` — a single badge, and
     `mandatory` wins if you somehow pass both. Never pass both true on the same field; they're
     designed as one boolean choice, not two independent flags.
   - `help`: a separate `?` badge with its own tooltip, shown in addition to the mandatory/
     optional badge, not instead of it.

---

## Default value fix (2026-07-18): `description` now defaults to a non-breaking space

**The problem this fixed:** across the ten real form pages, `FieldHint` is called ~66 times —
45 of those bare, with zero props. That overwhelming majority pattern clearly means "no hint
needed here, but keep this field's height consistent with its neighbors." But the early-exit
guard above used to defeat that: a fully bare `<FieldHint />` returned `null` — no wrapping
`<div>` at all, zero height — contradicting what its own dominant real usage assumed was
happening. Whether this was visible depended on layout (a `FieldGrid` row's CSS Grid track still
stretches to match a taller sibling in the same row regardless of a shorter cell's contents; a
`FieldGroup`'s stacked flex children do not, and would show a visibly smaller gap under a field
with no rendered hint than one with a hint or badge).

**The fix**, applied directly to `../ascendra-ui/ascendra-ui/components/ui/field.tsx`:

```tsx
function FieldHint({
  error,
  // Defaults to a non-breaking space — a plain " " collapses under normal
  // HTML/CSS whitespace handling and does not reliably reserve line height;
  // " " (the same character &nbsp; represents) does not collapse.
  description = " ",
  mandatory,
  optional,
  help,
  className,
  ...props
}: ...) {
```

**A plain ASCII space (`" "`, U+0020) does not work here** — it's collapsible whitespace under
normal CSS text handling and does not reliably reserve height on its own as the sole content of
an element. It must be a **non-breaking space** (` `, the character `&nbsp;` represents).
This isn't a novel choice — `FieldInfo`, defined a few dozen lines above `FieldHint` in the same
file, already solves its own equivalent empty-state case with literal `&nbsp;` JSX content.
The pre-existing internal fallback deeper in `FieldHint`'s render body (`description || " "`)
was, on inspection, **already** using a real non-breaking space — the original author had that
part right. The actual defect was purely the early-exit guard above it, which blocked every bare
`<FieldHint />` call from ever reaching that already-correct fallback.

With the default in place, the early-exit's `!description` term is always `false` unless a caller
explicitly overrides it — so the component always renders, and a bare call always reserves its
row's height, with no manual workaround needed.

**Status:** Applied to `../ascendra-ui/ascendra-ui/components/ui/field.tsx`, verified via lint and
a full `next build` (all 119 showcase pages, including every form/dialog that calls `FieldHint`).
**Not yet committed** — left as a working-tree change in `ascendra-ui` (a shared library repo with
its own release process — `CHANGELOG.md`/`BACKLOG.md`/`release.js`) pending the maintainer's own
review and commit.

---

## `FieldInfo`'s matching gap (2026-09-27): the default branch has the same bug — diagnosed, fix reverted, mitigate via `description` instead

**Where this was found:** `audit-logging.ui`'s Advanced Filter (`QueryFieldRenderer`, not the
regular `Field`/`FieldHint` forms this doc otherwise focuses on) — fields with `field.info` set sat
visibly lower than sibling fields without it, in the same `columns`-grid row.

`FieldInfo` (defined a few dozen lines above `FieldHint` in the same `field.tsx`) renders the label
subtitle — `info` in `FieldDef` terms, "shown below the label, above the input." It has **two**
render branches, gated on whether a `breakPoint` prop was passed:

```tsx
function FieldInfo({ className, children, breakPoint, ...props }: ...) {
  if (!children && breakPoint) {
    return <p aria-hidden ...>&nbsp;</p>;   // ← already correct
  }
  return <p ...>{children}</p>;              // ← the gap: no fallback at all
}
```

The `breakPoint`-set branch already has exactly the fix `FieldHint` needed (the literal `&nbsp;`
fallback described above) — so calls that pass a `breakPoint` are already safe. But the *default*
branch — no `breakPoint`, which is what `QueryFieldRenderer` always calls
(`<FieldInfo>{field.info}</FieldInfo>`, never a `breakPoint`) — has no such fallback. `<p>{undefined}</p>`
renders a genuinely empty paragraph, which does not reliably reserve the same line height as a
sibling `<p>` with real text — the identical underlying problem `FieldHint` was fixed for, just
never found in `FieldInfo`'s other branch. **This diagnosis is still accurate** — what follows is
what was tried, then undone.

**The fix that was proposed and briefly applied**, then reverted the same day (maintainer's call —
see AUI-037 in `hard-instructions.md` for the reasoning):

```tsx
function FieldInfo({ className, children, breakPoint, ...props }: ...) {
  if (!children) {
    return (
      <p
        aria-hidden
        data-slot="field-info"
        className={cn("mt-0.5 text-xs", breakPoint && fieldInfoBreakpointClasses[breakPoint], className)}
        {...props}
      >
        &nbsp;
      </p>
    );
  }
  return <p data-slot="field-info" className={cn("mt-0.5 text-xs", className)} {...props}>{children}</p>;
}
```

The condition would have been `!children` alone — any call with no content gets the placeholder,
whether or not a `breakPoint` was given. Reverted rather than change shared `FieldInfo` behavior for
a gap that has a workable mitigation at the content-authoring level instead (below).

**The actual mitigation in place — use `description` for isolated fields, `info` only when uniform:**
`FieldHint`'s own bare-call fallback (the "Default value fix" above) *was* kept, so `description`
already reserves height safely even when only some fields in a row have one set. So:
- Need a hint on just one or a few "odd" fields in a grid row? Put it in `description`
  (`FieldHint`), not `info` (`FieldInfo`) — safe today.
- Want to give every field in the row a subtitle? `info` is fine there — a row where `info`
  presence is uniform across every field never hits this gap, since there's no "some empty, some
  not" case to misalign.

**Rule:** Don't reach for `info` on a subset of fields in a `columns`-grid row — check whether
every sibling in that row will also have `info` set; if not, use `description` on the ones that
need it instead.

**Status:** Diagnosis confirmed via direct source read; fix written, verified (lint, typecheck,
live render), committed and pushed (`ascendra-ui` commit `9402593`) — then reverted the same day
(`91e8b3f`). The gap itself is unresolved in `FieldInfo`; the mitigation above is a content-authoring
workaround, not a library fix.

---

## Gotcha 1 — `mandatory` and `optional` are a single choice, not two flags

`{mandatory ? "Mandatory" : "Optional"}` means passing both `mandatory` and `optional` as `true`
on the same `FieldHint` silently shows only `Mandatory` — `optional` is checked but never wins.
Treat these as one enum-like choice per field (`mandatory`, `optional`, or neither), never both.

## Gotcha 2 — `description` prop, not children (AUI-010, restated here for completeness)

`FieldHint` never reads `children`. Pass hint text via `description="..."`. Passing text as JSX
children between the tags silently renders nothing (see `hard-instructions.md` AUI-010 for the
original finding and the real bug it caused in `app/signup/page.tsx`).

---

## Form-authoring guidance — when to use `mandatory` vs `optional`

Don't reach for the `mandatory`/`optional` badge reflexively on every field. The real forms use
it selectively:

- **If most fields on a form/section are required and only a few are optional** — mark the
  *few optional ones* with `optional`, and leave the required majority with no badge at all
  (their required-ness is already implied by being the norm, and the visual noise of ten
  `Mandatory` badges in a row adds nothing).
- **If most fields are optional and only a few are truly required** — mark the *few required
  ones* with `mandatory` instead, leaving the optional majority unbadged.
- **Never badge every field in both directions at once** — a form where every single field
  carries either `Mandatory` or `Optional` is visually noisy and defeats the purpose of the
  badge (it stops drawing the eye to the exception, since there is no exception left). The
  badge exists to flag the *minority* case in a given form/section, whichever direction that is.
- This is independent of the Rule above: every field still gets a `FieldHint` (bare or not) for
  height consistency; the mandatory/optional badge is a separate, selective decision layered on
  top of that baseline.
