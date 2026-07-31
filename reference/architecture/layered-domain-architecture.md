# Layered Domain Architecture — Core vs. Extension Model

This is the authoritative reference for how an Ascendra project separates core domain logic from
project-specific extensions — market/country, sector, or any other dimension a BRD's "Designed for
Extension" section (Section 1.6) names. It applies to every layer of the system: data model,
services, reporting, and frontend. Any engineering decision about where a piece of code, a table
column, or a UI component belongs starts here.

This document is universal — it does not assume any particular domain, tech stack, or extension
dimension. For a fully worked example applying this methodology to a real project (Ascendra Pay's
Market and Sector extensions), see `layered-domain-architecture.example.md` in this same folder.
Read the example for illustration only — do not copy its specific entities or table names into a
different project; derive your own from this methodology instead.

---

## The One Rule

> **Extensions depend on Core. Core never depends on extensions. Extensions never depend on each other.**

This is not a guideline — it is a hard constraint enforced at the code level (import rules, lint).
If any code in `core/` imports anything from `extensions/`, it is a bug.

---

## The Membership Test

When you are unsure whether something belongs in core or an extension, apply this test:

> **"Would every deployment of this base system need this, regardless of which extension is active?"**

- If yes → Core
- If no → Extension (whichever dimension it belongs to)

Apply this consistently to every table, column, service method, event, page, and component — not
just to entities. If a piece of logic only makes sense for one particular value of one extension
dimension (one country, one sector, one integration), it belongs in that extension, not in Core.

---

## The Layers

```
┌─────────────────────────────────────────────┐
│                CORE DOMAIN                   │
│   The entities, services, and behavior       │
│   every deployment needs, regardless of      │
│   which extensions are active.               │
└─────────────────┬─────────────────────────────┘
                  │ ← extensions depend on core
       ┌──────────┴──────────┐
       │                     │
┌──────▼──────┐      ┌───────▼──────┐
│ EXTENSION A │      │  EXTENSION B │
│ (e.g. one   │      │  (e.g. one   │
│ dimension's │      │  dimension's │
│ specific    │      │  specific    │
│ value)      │      │  value)      │
└─────────────┘      └──────────────┘
       │                     │
       └──────────┬──────────┘
                  │ ← never cross-depend
```

A single deployment can activate more than one extension independently. Neither extension knows
about the other.

---

## Layer 1 — Core Domain

### What belongs here

Only what is universal to every deployment of the base system, regardless of which extensions are
active.

- Entities every deployment needs, with only the attributes every deployment needs
- Services implementing behavior every deployment needs
- Domain events that fire regardless of which extensions are active

### What does NOT belong here

- Any field, table, or attribute that only some deployments need
- Any service method that references a specific extension dimension's value
- Any integration-specific reconciliation or processing logic
- Any extension-specific lifecycle event

### Core must be independently testable

The core domain must be fully implementable and testable with no extension tables present in the
database. If a core test requires an extension table to pass, the boundary has been violated.

---

## Layer 2 — Extension

### What belongs here

Anything that applies only because one particular dimension's value is active for this deployment —
not because it is universal.

- Dimension-specific validations, formats, or rules
- Dimension-specific integration logic (a specific provider's API, a specific regulatory
  requirement)
- Dimension-specific entities that have no core counterpart

### Implementation pattern

**Extension table — 1:1 FK to the core entity it extends:**

```sql
-- {Extension} extends {CoreEntity}
CREATE TABLE {core_entity}_{extension}_details (
  id                uuid PRIMARY KEY,
  {core_entity}_id  uuid UNIQUE REFERENCES {core_entities}(id),  -- 1:1, not nullable
  -- extension-specific columns
);
```

**Extension-only entities** (no core counterpart) are new tables scoped entirely within the
extension, referencing core tables via FK where needed but never referenced back by core.

**Extension service — handles dimension-specific logic, calls core, never the reverse:**

```
{Extension}Service
  ├── on{CoreEvent}(event) → handles the dimension-specific reaction
  └── {extensionSpecificOperation}() → eventually calls the corresponding core service method
```

`{Extension}Service` calls the core service. The core service never calls `{Extension}Service`.

---

## The Extension Registry

A single registry table is the source of truth for which extensions are active for a given
deployment. Every runtime decision — which event handlers fire, which strategies are injected,
which frontend components load — starts with a lookup on this table.

```sql
CREATE TABLE {core_entity}_extensions (
  id             uuid PRIMARY KEY,
  org_id         uuid NOT NULL REFERENCES orgs(id),
  {core_entity}_id uuid NOT NULL REFERENCES {core_entities}(id),
  extension_type varchar NOT NULL,  -- one value per active extension
  activated_at   timestamptz NOT NULL,
  created_at     timestamptz NOT NULL DEFAULT now(),
  updated_at     timestamptz NOT NULL DEFAULT now()
);
```

> **Note on extension table schemas throughout this document:** all schemas above show
> domain-specific columns only. Every table in the system also requires the mandatory columns
> defined in the project's `system-architecture.md` (typically `id`, `org_id`, `created_at`,
> `updated_at`, plus `version` and `created_by` for master tables).

When processing any operation, the runtime:
1. Looks up the registry for the relevant core entity
2. Activates the corresponding extension handlers and strategies
3. Proceeds — core is always active; extensions are conditional

---

## Service and Event Pattern

### Core services are the only external entry point

API controllers, agents, and external callers invoke core services. They never invoke extension
services directly. Core fires domain events; extensions react to them.

```
API request → CoreService.coreOperation() → fires CoreEvent
                                              ↓
                                  ExtensionAHandler.onCoreEvent()
                                  ExtensionBHandler.onCoreEvent()
```

### Strategy pattern for behavioural variation

When core needs to produce output that varies by extension, it holds an interface. Extensions
provide implementations registered at startup — core never imports the concrete implementation.

```typescript
// Core defines the interface
interface {Behavior}Strategy {
  execute(context: CoreContext): Result
}

// An extension provides an implementation
class {Extension}{Behavior}Strategy implements {Behavior}Strategy {
  execute(context: CoreContext): Result {
    // extension-specific behavior
  }
}

// Core holds a reference — never knows which implementation runs
class CoreService {
  constructor(private strategy: {Behavior}Strategy) {}
  performBehavior(context: CoreContext) { return this.strategy.execute(context) }
}
```

The strategy is resolved from the extension registry at request time. Core holds the interface.
Extension holds the implementation. Core never imports the extension class. If a core file imports
a class from `extensions/`, it is an architectural defect.

---

## Reporting Pattern

### Core reports — work for every deployment

Built from core tables only. Runnable regardless of which extensions are active.

### Extension reports — require extension tables

Built from core tables plus extension JOINs. Only meaningful when the relevant extension is
active. Report services follow the same dependency rule:

```
ExtensionReportService imports CoreService    ✓
CoreService imports ExtensionReportService    ✗ — violation
```

### Report availability is driven by the extension registry

The reporting layer checks which extensions are active before presenting report options. A
deployment without a given extension active never sees that extension's reports.

---

## Frontend Pattern

### AppShell — core

The shell provides: navigation container, routing, authentication, layout, and core pages. It
never branches on which extension is active by name — it asks one question: what extensions are
registered?

### Extension manifests

Each extension declares what it contributes to the shell — nav items, pages, dashboard widgets,
form slots — loaded at login based on active extensions.

### Form slots

Core forms declare slots where extensions can inject fields. The core form never references an
extension-specific field by name — it receives whatever the active extension injects into the slot
and passes it back to the API as extension metadata.

### Branding vs. functionality

These are independent concerns — tenant branding (logo, colour, font) is driven by the core
tenant's own profile config, not by which extensions are active. Every tenant gets their own
branding regardless of which extensions they have.

---

## Sequencing Rule

```
Phase 1: Core domain — all core entities, services, events, core pages
Phase 2+: Extensions — independently buildable in parallel, once core is complete and tested
```

No extension work begins before the core entity it extends is complete and tested. Extension tests
run against a database that includes core tables. Core tests run against a database with no
extension tables.
