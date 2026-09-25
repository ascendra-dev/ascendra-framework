# Code Style Conventions

General code-style preferences that apply across any project's implementation, independent of
which UI component library or framework a project uses — distinct from
`reference/ascendra-ui/hard-instructions.md`, which is scoped specifically to `ascendra-ui`
component usage. Referenced by `/implement-story`'s implementation rules.

**Unlike `hard-instructions.md`'s entries, these are preferences to default to, not hard rules**
— each one says explicitly when it's fine to deviate.

---

## Rule 1 — Hoist a `.map()`'s per-item computation, or a JSX child's conditional string, above the `return` — but only when that's actually simpler

When a `.map()` embedded directly in JSX needs a block body (extra `const`s before `return (...)`),
or a JSX child mixes static text with a multi-line ternary, prefer computing the result above the
component's `return` instead of inline.

**Don't** (a `.map()` that needs a block body purely to compute two per-item values):
```tsx
{KPI_DEFS.map((def) => {
  const kpi = overview.data?.kpis[def.key];
  const up = kpi ? kpi.deltaPct >= 0 : true;
  return (
    <Card key={def.key}>...</Card>
  );
})}
```

**Do** (shape the data first, so the JSX-embedded `.map()` is a single-expression arrow):
```tsx
const kpiRows = KPI_DEFS.map((def) => {
  const kpi = overview.data?.kpis[def.key];
  return { ...def, kpi, up: kpi ? kpi.deltaPct >= 0 : true };
});

// ...

{kpiRows.map((row) => (
  <Card key={row.key}>...</Card>
))}
```

**Don't** (a JSX child mixing static text with an inline multi-line ternary):
```tsx
<CardHeaderSubtitle>
  Past 30 days
  {overview.data
    ? ` · ${formatDateRange(overview.data.perDay[0].day, overview.data.perDay.at(-1)!.day)}`
    : ""}
</CardHeaderSubtitle>
```

**Do** (fold the whole thing into one string computed above the return):
```tsx
const volumeSubtitle = overview.data
  ? `Past 30 days · ${formatDateRange(overview.data.perDay[0].day, overview.data.perDay.at(-1)!.day)}`
  : "Past 30 days";

// ...

<CardHeaderSubtitle>{volumeSubtitle}</CardHeaderSubtitle>
```

**This is a preference, not a hard rule.** If hoisting would make a given case *more* complex
than leaving it inline — the inline ternary is already a single short expression, or hoisting
would require awkwardly threading extra variables into a separate array just to save one line of
indentation — keep the inline version.

**Do not extract a separate function or component purely to avoid a block-bodied `.map()`.** That
fixes the indentation but costs "read top-to-bottom in one place" — you now have to jump
elsewhere to see what actually renders. The point of this rule is to remove the indentation
without that cost, not to force extraction; hoisting the data-shaping step to a plain local
variable in the same function achieves that, an extracted function/component does not.
