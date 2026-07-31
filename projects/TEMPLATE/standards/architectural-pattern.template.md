# Architectural Pattern Standard Template

> **[AI Guide — Document Level]**
> Use this template when a project has a domain-specific integration or architectural strategy
> substantial enough to warrant its own standing document — e.g. a payment adapter model, a
> multi-provider integration strategy, a phased rollout of an integration approach. This is
> different from the universal, framework-shipped Core/Extension methodology at
> `reference/architecture/layered-domain-architecture.md` — that one is fixed and applies to
> every project; this template is for a **project-specific** pattern `gen-architecture` proposes
> and generates fresh for one project, as part of the "AI-determined additional standards" step.
> Only generate a document from this template when there's a concrete signal warranting it — an
> empty proposal (no additional pattern document needed) is a valid, common outcome.
>
> **File location:** `projects/{PROJECT_CODE}/standards/{pattern-name}.md`

---

# {Pattern Name} — {PROJECT_CODE}

## Document Control

| Version | Date | Reason |
|---------|------|--------|
| 1.0 | {date} | Initial generation |

---

## 1. Overview

{2–4 sentences: what problem this pattern solves for this project, and why it's substantial
enough to warrant its own document rather than living inline in the architecture document.}

---

## 2. The Pattern

> **[AI Guide]** Name the core abstraction (an interface, an adapter contract, a strategy) and show
> how the base/core code depends only on the abstraction, never on a concrete implementation — same
> principle as the Core/Extension methodology, applied to one specific integration or strategy
> rather than the whole system.

### {Core abstraction} — defined once, never the implementation

```{language}
{The interface/contract core code depends on}
```

### Implementations — never imported by core code directly

```{language}
{One or more concrete implementations of the abstraction}
```

### Resolution

{How the correct implementation is selected at runtime — a registry lookup, a config value, a
per-tenant setting.}

---

## 3. Model 1 — {Name} ({status, e.g. "Current" / "Phase 1"})

> **[AI Guide]** Only include multiple numbered models if this pattern genuinely has more than one
> phase or variant (e.g. an integration starting with an aggregator and later moving to a direct
> connection). If there's only one model, drop the "Model N" numbering and describe it directly
> under Section 2.

### What it is

{...}

### Flow

{...}

### Characteristics

{...}

---

## 4. Model 2 — {Name} ({status, e.g. "Future / Phase 2 target"})

{Repeat the same structure as Model 1, only if a second model genuinely exists or is planned.}
