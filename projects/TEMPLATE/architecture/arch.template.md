# Architecture Document Template

> **[AI Guide — Document Level]**
> The architecture document is produced via `/gen-architecture` after all epics are approved and before development begins. It is the complete blueprint for implementation — every module, table, endpoint, role, and environment variable is specified here. `/implement-story` implements exactly what is defined in this document — nothing more, nothing less.
>
> **Gate artifact:** this document must be approved by the Product Owner (Gate 5) before `/implement-story` runs for Sprint 1. Once approved, the document is Locked. Any change after locking requires a formal change request — `/implement-story` does not implement scope that is not in the Locked document.
>
> **What this document is not:** a product requirements document (that is the BRD), a sprint plan (that is the sprint plan), or a design mockup. If a sentence describes what the user can do rather than how the system is built, it belongs in the BRD, not here.
>
> **Source of truth order:** BRD → Architecture Document → Sprint Plan. If there is a conflict, the BRD wins. If the architecture document cannot satisfy a BRD requirement, that is an Open Decision (Section 8) — do not make a silent architectural choice that violates the BRD.
>
> **File location:** `projects/{PROJECT_CODE}/architecture/arch-v{N}.md`
> e.g. `projects/ASCENDRA-PAY/architecture/arch-v1.md`

---

## Part 1 — Template

---

**Project:** [Project name — e.g. ASCENDRA PAY Phase 1]
**Version:** [e.g. 1.0]
**Status:** Draft / Under Review / Approved / Locked
**Produced by:** Ascendra AI
**BRD Version:** [e.g. v1.1 — the approved BRD this architecture is derived from]
**Sprint Plan:** [e.g. Sprint 1 Plan — approved YYYY-MM-DD]

> **[AI Guide]** Status transitions: Draft (being written) → Under Review (submitted to Product Owner) → Approved (Product Owner approved) → Locked (set immediately on approval — no changes after this point without a change request). Record the Locked date in the Locked on field. `/implement-story` reads this field to confirm the document is in a state it may act on.

**Locked on:** [date — set when Product Owner approves]

---

### Change History

> **[AI Guide]** This table lives inside the architecture document itself — not in the template's Document Control above. Record every material change: a table column added, an endpoint removed, a role renamed. Do not record formatting-only edits. If the document is Locked, any change must be preceded by a change request approval — record that decision here before making the change.

| Version | Date | Sections Changed | Reason |
|---------|------|-----------------|--------|
| 1.0 | [date] | — | Initial submission |

---

### 1. System Overview

> **[AI Guide]** 2–4 sentences. Written for a non-technical stakeholder — a Product Owner or a client, not a developer. Describe what the system does, who uses it, and the primary business problem it solves.
>
> **Test:** could a stakeholder read this section and explain the system to a colleague without asking a follow-up question? If not, it is too technical or too vague.
>
> **Do not write:** technology names, API routes, database tables, or any implementation detail. If the word "endpoint", "schema", "module", or "service" appears here, rewrite it.
>
> **Domain, Delivery model, Complexity:** pick from the defined options. Complexity drives the implementer's capacity planning — do not inflate to Large to signal importance.

[2–4 sentences describing what the system does, who uses it, and the primary business problem it solves]

**Domain:** [Finance / Education / Healthcare / Logistics / HR / E-commerce / Internal tooling]
**Delivery model:** SaaS product / Custom project
**Estimated complexity:** Small / Medium / Large

---

### 2. Tech Stack

> **[AI Guide]** The tech stack is recommended on the fly per project, from constraints discovery (deployment target, team capability, budget posture, compliance signals) — see `FW-025`. There is no fixed "Ascendra standard stack" every project must use. The table below shows the common default choices as a starting point, not a mandate; `gen-architecture`'s constraints-discovery and tech-stack-recommendation steps determine what's actually right for this project and write the confirmed choice to `projects/{PROJECT_CODE}/standards/system-architecture.md`. Populate this section from that file. Rows that are genuinely not in scope for this project (e.g. payments integration not needed) should be removed entirely — do not leave N/A rows in the final document.
>
> **Deviation rule:** any row in this table that differs from what's recorded in `system-architecture.md` (e.g. a later change made without updating that file) must be marked ⚠ Deviation and have a corresponding entry in Section 9. A deviation cannot be self-approved — it must be flagged to the Product Owner before the document is submitted for review. Do not assume a deviation is obvious or acceptable without documentation.
>
> Remove any rows not in scope for this project before submitting. The table below shows illustrative defaults — remove, do not mark N/A.

| Layer | Technology | Version | Deviation? |
|-------|-----------|---------|-----------|
| Frontend | Next.js (App Router) | 14+ | — |
| UI Library | Ascendra UI | Latest | — |
| Backend API | NestJS | 10+ | — |
| Language | TypeScript (strict mode) | 5+ | — |
| Database access | Drizzle ORM | Latest stable | — |
| Database | PostgreSQL | 16 | — |
| Cache / Queues | Redis | 7 | — |
| Email | Resend | — | — |
| Payments | [PSP name — e.g. Stripe / Safepay] | — | — |
| File storage | AWS S3 | — | — |
| Logging | [e.g. Pino, shipped to Axiom] | Latest | — |
| Error Tracking | [e.g. Sentry] | Latest | — |
| Containerisation | Docker + Docker Compose | — | — |
| CI/CD | GitHub Actions | — | — |

---

### 3. Module Breakdown

> **[AI Guide]** List every backend module the implementer will create — the default illustration below uses NestJS's module terminology (see `FW-025`'s default), since that's this framework's most common stack; a project confirmed on a different backend framework uses that framework's own unit of organisation (e.g. a Spring `@Service`/`@RestController` package, a Django app) but follows the same principle. One module per bounded domain — do not group unrelated responsibilities into one module. Every external service integration (PSP, email, file storage) must have its own dedicated module — do not embed external service calls in a domain module.
>
> **Module naming:** one bounded-domain unit per row, named per the confirmed framework's own convention — e.g. `{Domain}Module` for NestJS (`ClientModule`, `InvoiceModule`, `PaymentModule`, `EmailModule`), a `{domain}` package for Spring or Django. Never a catch-all unit (`UtilsModule`/`HelperModule`/`CommonModule` or equivalent) — if you find yourself creating one, it means responsibilities are not correctly separated.
>
> **Internal layer pattern:** the default is Controller → Service — no repository class, with database queries written directly in the service using the confirmed ORM/query layer (Drizzle by default per `FW-025`). This is a starting assumption, not a mandate: `gen-architecture`'s Architecture Pattern Discovery step evaluates domain complexity per project and recommends a fuller layered pattern (adding a Repository or Domain layer) when warranted. The pattern actually confirmed for this project is recorded in `projects/{PROJECT_CODE}/standards/system-architecture.md` and must be stated explicitly at the top of this section — follow it exactly, whichever it is. No module imports another module's service directly — cross-module calls go via the confirmed framework's own inter-module mechanism (e.g. NestJS module exports/imports, Spring dependency injection).
>
> **Completeness check:** every table in Section 4 must be owned by exactly one module listed here. Every external service in Section 7.2 must have a matching module here. If any table or service is unowned, the module breakdown is incomplete.

| Module | Responsibility | External services used |
|--------|---------------|----------------------|
| AuthModule | JWT issuance, refresh token rotation, login, logout, current-user endpoint | — |
| [DomainModule] | [Single, bounded responsibility — what domain records this module owns and what operations it provides] | [e.g. Resend / Safepay / AWS S3, or "—"] |

---

### 3.1 Project Structure

> **[AI Guide]** Define the physical repository layout and folder structure for every repo in this project's confirmed topology. The implementer uses this section to know exactly where to create files — it does not make structural decisions on its own.
>
> **Repository layout:** every Ascendra client project keeps documents separate from code:
> - `projects/{PROJECT_CODE}/` — all product documents: BRD, epics, stories, architecture, within this framework workspace. No code lives here.
>
> The default code topology is two repos — `{project-code}-api/` (NestJS backend, scaffolded per Section 3.1.1) and `{project-code}-web/` (Next.js frontend, built on `ascendra-ui`'s component library per Section 3.1.2). This is a starting assumption, not a mandate: `gen-architecture`'s Architecture Pattern Discovery step evaluates platform needs per project (a distinct mobile persona, background/scheduled processing) and recommends additional repos — `{project-code}-mobile/`, `{project-code}-worker/`, etc. — when warranted, see `FW-025`. The topology actually confirmed for this project is recorded in `projects/{PROJECT_CODE}/standards/system-architecture.md`; for any repo beyond api/web, add an unnumbered `####` subsection immediately after 3.1.2 and before 3.1.3 (e.g. `#### Mobile — {project-code}-mobile/`), following the same principle as 3.1.1/3.1.2 — physical folder layout, one entry per domain module. Do not reuse 3.1.3 or later numbers — those are reserved for Screen & Navigation Map and Extension Points.
>
> **Naming convention:** `{project-code}` is the lowercase hyphenated project code. Examples: `ascendra-pay-api`, `ascendra-pay-web`, `leavetrack-api`, `leavetrack-web`.

---

#### 3.1.1 Backend — `{project-code}-api/`

> **[AI Guide]** The layout below is the framework's default illustration, for a NestJS backend — the most common confirmed choice per `FW-025`. **If the confirmed backend (from `system-architecture.md`) is NestJS:** the implementer creates this by running `nest new {project-code}-api` and restructuring `src/` to match the layout below exactly.
>
> **If a different backend framework is confirmed:** define the equivalent structure for that framework, following the same principles the NestJS layout below demonstrates — one folder/package per domain module (from Section 3), the confirmed internal layering pattern (from Step 6 of `gen-architecture`) reflected in real file/class names, schema/migration files organised per the confirmed ORM or data-access approach, and auth/cross-cutting concerns (guards, interceptors, exception handling) using that framework's own idiomatic mechanism — not the NestJS names shown below. Declare this as the confirmed stack in Section 2, not as a Section 9 deviation; a deviation is only a departure from what `system-architecture.md` itself records.

```
{project-code}-api/
├── src/
│   ├── modules/              # One folder per module (Section 3)
│   │   ├── auth/             # AuthModule (always present)
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.module.ts
│   │   │   └── dto/
│   │   └── {domain}/         # One folder per domain module (Section 3)
│   │       ├── {domain}.controller.ts
│   │       ├── {domain}.service.ts
│   │       ├── {domain}.module.ts
│   │       └── dto/
│   ├── db/
│   │   ├── schema/           # ORM table definitions — one file per module
│   │   ├── migrations/       # ORM migration files
│   │   └── index.ts          # DB client export
│   ├── common/
│   │   ├── guards/           # Auth guards (e.g. JwtAuthGuard, RolesGuard for NestJS)
│   │   ├── decorators/       # Cross-cutting param/route decorators (e.g. @CurrentUser(), @Roles())
│   │   ├── interceptors/     # Response serialisation setup
│   │   └── filters/          # Global exception filter
│   └── main.ts               # Bootstrap — global pipes, interceptors, API docs
├── .env.example               # Generated from Section 7.3 of this document
├── docker-compose.yml         # Local dev services (database, cache, etc.)
└── package.json                # or the confirmed language's equivalent manifest (pom.xml, requirements.txt, ...)
```

*(Shown for a NestJS + Drizzle default. For a different confirmed backend, replace the file names and folder purposes above with that framework's real equivalents — the shape of the structure, not the literal filenames, is what must be preserved.)*

---

#### 3.1.2 Frontend — `{project-code}-web/`

> **[AI Guide]** The frontend is **not scaffolded from scratch**. It is built on the verified `ascendra-ui` repository's component library, copied locally into this project — but it is **not** a wholesale clone of the `ascendra-ui` repo, and it does not use `ascendra-ui`'s own `create-project.js`/`npm run upgrade` scripts. Those exist for standalone `ascendra-ui` consumers with no project-management system of their own; this is an Ascendra Framework project, which already has one (sprints, releases, PR descriptions, its own `.claude/commands`) — shipping `ascendra-ui`'s generic project bookkeeping would duplicate or conflict with it.
>
> **Setup steps the implementer follows** (full procedure: `implement-story.md`'s scaffolding steps):
> 1. Copy `ascendra-ui`'s component library folder (`ascendra-ui/ascendra-ui/`, excluding `template/`) into `{project-code}-web/ascendra-ui/` — managed, never hand-edited
> 2. Copy the root app shell only (`layout.tsx`, `globals.css`, `favicon.ico`) into `app/` — not the showcase demo pages or the generic getting-started/sandbox shell
> 3. Copy `ascendra-ui/docs/` into `{project-code}-web/ascendra-ui/docs/` — nested under the library folder, since it's library reference documentation, not this project's own docs
> 4. Copy the config files needed to build (`components.json`, `next.config.ts`, `tsconfig.json`, `postcss.config.mjs`, `eslint.config.mjs`, `global.d.ts`) from `ascendra-ui`'s repo root
> 5. `package.json` name field is `{project-code}-web`
> 6. Create `.env.example` from Section 7.3 of this document (frontend env vars only — those prefixed `NEXT_PUBLIC_`)
> 7. The `app/` directory now contains only the project's application screens — all implemented story by story
>
> **Do not re-configure Tailwind, the component imports, or the design system.** These are already correct as copied. Only application-level screens and their routes are added.

```
{project-code}-web/
├── app/                      # Next.js App Router
│   ├── (auth)/               # Unauthenticated routes: login, register
│   ├── (dashboard)/          # Authenticated routes — one folder per domain
│   │   └── {domain}/         # e.g. invoices/, payers/, settings/
│   ├── layout.tsx
│   └── globals.css
├── ascendra-ui/               # Managed component library — copied at scaffold time, never hand-edited
│   └── docs/                  # Library reference docs (ui-reference.md, showcase-reference.md)
├── components/                # Project-specific components (beyond the ascendra-ui library)
├── hooks/                     # Project-specific hooks — not nested under lib/
├── lib/
│   ├── api.ts                 # Typed API client (fetch wrapper for the backend)
│   └── auth.ts                # Session/auth utilities
├── .env.example                # NEXT_PUBLIC_ vars only — generated from Section 7.3
└── package.json
```

---

### 3.1.3 Screen & Navigation Map

> **[AI Guide]** Include this section whenever the confirmed tech stack (Section 2) includes a UI module (web/mobile/any) — this is generic to every project with a UI, not conditional on portal count or permission model. The only case where this entire section is replaced is a project with **no UI module at all** (pure API/backend/integration project): replace it with "None — no UI module in scope for this project."
>
> Portal count and permission model (composable vs. fixed-role) do not gate whether this section exists. They only shape how much content each subsection carries — a single-portal, fixed-role project still gets a real Portals table (one row), a real Navigation Map (flat, "baseline — always visible" entries), and, always, a Screen Inventory: `/implement-story` needs a screen list to build from regardless of how simple the project's permission model is. 3.1.3.4 (Action Gating) and 3.1.3.5 (Cross-Cutting UI Rules) already scale correctly on their own — each collapses to its own "None" statement only when nothing in the project triggers it, not gated by a parent condition; that per-item pattern is the model the rest of this section follows too.
>
> **What this section is not:** a visual mockup or wireframe. It is a structural inventory — which screens exist, who can reach them, and how navigation is composed — so that information architecture is decided once, before implementation, instead of invented story by story. Pixel-level layout and composition are the implementer's job, using the component library confirmed in 3.1.2.

#### 3.1.3.1 Portals

One row per genuinely distinct navigation shell — not one row per persona. Two personas share a portal if they log into the same app and their navigation differs only by which items are visible (governed by 3.1.3.2), not by a materially different shell.

| Portal | Personas | Access model | Base route |
|---|---|---|---|
| [e.g. Staff App] | [roles/capabilities that use it] | [standing session / other] | [route prefix] |

If Section 6.3 (Delegated Access) applies, name the delegated session here explicitly as entering an existing portal, not as its own row — e.g. "Not a separate portal — enters [Portal] under delegated access; see Section 6.3."

#### 3.1.3.2 Navigation Map

One row per nav item, with the condition that makes it visible — not one full nav tree per persona. A given user's rendered navigation is the union of every item whose condition they satisfy.

| Nav item | Portal | Visibility condition |
|---|---|---|
| [e.g. Settings] | [Portal] | [e.g. "Configuration" capability, or "baseline — always visible"] |

#### 3.1.3.3 Screen Inventory

One row per screen. Derive from BRD Section 4 (User Journeys) and Section 5 (Functional Requirements) — every journey step or requirement with a "persona can see/do X" shape gets a row here. A screen missing from this table is a gap to close before this document is submitted for review, not something to leave for `/implement-story` to invent.

| Screen ID | Portal | Route | Reachable by | Purpose | Source |
|---|---|---|---|---|---|
| SCR-001 | [Portal] | [route] | [role/capability] | [one line] | [REQ-ID / Journey ref] |

#### 3.1.3.4 Action Gating

Only for screens shared across multiple roles/capabilities where a specific action on the screen needs restriction finer than page-level access (e.g. everyone who can open an invoice can view it, but only a Write-off Approval capability holder sees the approve action on that same screen). If no screen needs this: "None — every gated screen restricts at the page level only; see 3.1.3.3."

| Screen ID | Action | Requires |
|---|---|---|
| SCR-XXX | [button/action] | [capability/role] |

#### 3.1.3.5 Cross-Cutting UI Rules

Rules that apply globally, across every screen, rather than being attached to one screen's row above. State each once here rather than repeating it per screen.

- **Blanket read-only mode** (if `viewOnly` is declared in Section 6.1): every mutating action, on every screen, is hidden or disabled for the duration of the session — this overrides whatever 3.1.3.4 would otherwise allow.
- **Delegated access indicator** (if Section 6.3 applies): a persistent, session-long UI element identifying the delegated identity being acted on and offering an explicit exit — not a one-time notice.

If neither applies: "None — no cross-cutting UI rules beyond standard page-level and action-level gating."

---

### 3.2 Extension Points

> **[AI Guide]** Fill this section if the BRD's Section 1.6 (Designed for Extension) named future variation dimensions, or if this project is known to have planned extension projects.
>
> An extension point is a technical seam this base system deliberately exposes so that an extension project can inject logic without modifying the base code. Naming these seams here:
> - Tells extension project architects exactly where to integrate
> - Prevents coupling between base and extension code
> - Makes the extension relationship explicit, reviewable, and testable
>
> Apply `reference/architecture/layered-domain-architecture.md` — the framework's universal Core/Extension methodology — as the governing standard for this section. It defines the One Rule (extensions depend on core, never the reverse), the Membership Test for deciding what belongs where, and the concrete patterns (extension tables, the extension registry, the strategy pattern for behavioral variation) that this section's Extension Points table should apply. Its paired `.example.md` is a fully worked illustration from one real project — read it for illustration only, never copy its specific entities or table names into a different project.
>
> For each extension point, define the mechanism the base system exposes (abstract class, event, strategy, config hook) and which module owns it. The extension project implements the interface or registers its handler — the base system never imports from the extension.
>
> If this is a Standalone project with no planned extensions, write: "None — no extension dimensions identified in BRD Section 1.6."

| Extension Point | Mechanism | Owning Module | Extension Projects |
|----------------|-----------|--------------|-------------------|
| [e.g. `TaxCalculationStrategy`] | [Abstract class — extension registers a country-specific implementation] | [InvoiceModule] | [PROJECT_CODE of extension project, or "Planned — TBD"] |

---

### 4. Data Model

#### 4.1 Entity Relationship Summary

> **[AI Guide]** 3–6 sentences of prose describing the main entity relationships. Written at the conceptual level — not SQL, not Drizzle. Someone reading this section must be able to sketch the ER diagram without seeing Section 4.2.
>
> Structure: start with the top-level entity (Organisation or the primary domain entity), then describe what hangs off it and how entities relate. Use the words "has many", "belongs to", "has one" — not "foreign key", "JOIN", or "reference".
>
> **Do not skip this section.** The implementer uses 4.1 to sanity-check the 4.2 schema before writing it.

[Prose describing main entity relationships at a conceptual level]

---

#### 4.2 Table Definitions

> **[AI Guide]** One table block per table, grouped by module. Every table in the system must appear here — the implementer only creates tables listed in this section. A table not listed here requires a change request.
>
> **Stack note:** the column definitions and code blocks below are shown in the framework's default illustration — Drizzle ORM against PostgreSQL, per `FW-025`. If a different database/ORM is confirmed in `system-architecture.md` (e.g. Spring Data JPA against MySQL, TypeORM, Prisma), translate the same column set, types, and rules below into that stack's actual syntax — the rule (which columns exist, their semantics, indexing, soft delete) is what's mandatory; the Drizzle/TypeScript syntax is one illustration of it, not a requirement to use Drizzle.
>
> **Mandatory columns for Master tables** (primary domain records: users, invoices, clients, payers, etc.):
> ```typescript
> id:        uuid('id').primaryKey().defaultRandom(),
> orgId:     uuid('org_id').notNull().references(() => orgs.id),
> version:   integer('version').notNull().default(1),   // optimistic lock counter
> createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
> updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
> createdBy: uuid('created_by').references(() => users.id),  // null = system action
> ```
>
> **Mandatory columns for Detail tables** (child records tied to a master: line items, approval actions, etc.):
> ```typescript
> id:        uuid('id').primaryKey().defaultRandom(),
> orgId:     uuid('org_id').notNull().references(() => orgs.id),  // redundant by design
> createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
> updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
> ```
>
> **Every table with `updated_at` requires a trigger** in the same migration file:
> ```sql
> CREATE TRIGGER set_updated_at
> BEFORE UPDATE ON [table_name]
> FOR EACH ROW EXECUTE FUNCTION set_updated_at();
> ```
> The `set_updated_at()` function is created once in the initial migration and reused by all tables.
>
> **Money columns:** always `integer` in the smallest currency unit (pence for GBP, paisa for PKR, cents for USD). Never `decimal` or `float`. £100.00 is stored as `10000`.
>
> **Soft delete:** use a `deletedAt` timestamp column, not a boolean `isDeleted`. `deletedAt = null` means active. Filter `WHERE deleted_at IS NULL` on all list queries.
>
> **Index rule:** every `orgId` column gets an index. Every foreign key that will be used in a WHERE clause gets an index. Do not index columns that are never filtered on.

---

##### [table_name]

> **[AI Guide]**
> - **Master** — primary domain records with full lifecycle (users, invoices, payers, clients). Always include all mandatory Master columns. Always soft-delete with `deletedAt`.
> - **Detail** — child records that exist only in the context of a Master (line items, approval actions, audit log entries). Mandatory Detail columns apply. No soft delete — Detail records are immutable once created.
> - **Lookup** — application-managed reference rows with additional attributes beyond just a code and label (e.g. `dunning_policies` with configurable step counts, `notification_templates` with body text). Managed via the application UI or admin tools. Include `orgId` if lookup values are per-org; omit if they are system-wide.
> - **Reference** — static data seeded once and not modified at runtime (e.g. `currencies`, `countries`). No `orgId`. No `updatedAt` trigger needed. These rows are defined in Section 4.4 (Seed Data).

**Type:** Master / Detail / Lookup / Reference
**Module:** [Which module owns this table]
**Purpose:** [One sentence — what this table stores]

*(Default illustration — Drizzle/PostgreSQL. Translate into the confirmed ORM/database's own syntax if different; keep the same columns, types, and index.)*

```typescript
export const [tableName] = pgTable(
  '[table_name]',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    version: integer('version').notNull().default(1),
    // domain columns
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
    createdBy: uuid('created_by').references(() => users.id),
  },
  (table) => ({
    orgIdx: index('[table_name]_org_idx').on(table.orgId),
  }),
);
```

[Repeat for each table, grouped by module]

---

#### 4.3 Enums

> **[AI Guide]** List every enum used in this project (default illustration: PostgreSQL enum; use the confirmed database's own enum/constrained-value mechanism if different — e.g. a MySQL `ENUM` column or a lookup table with a check constraint). An enum must be declared here before it can be used in a table definition in Section 4.2. If a column has a fixed set of possible values, it should be an enum — do not use a plain string column with validation-only enforcement.
>
> Name the enum with the pattern `{domain}_{field}` — e.g. `invoice_status`, `payment_method`, `user_role`.
>
> If this project has no custom enums, write: "No project-specific enums — the standard `user_role` enum from the base schema is sufficient."

```typescript
export const [name]Enum = pgEnum('[name]', ['Value1', 'Value2', 'Value3']);
```

---

#### 4.4 Seed Data

> **[AI Guide]** Seed data is data that must exist in the database before the application is usable. It is distinct from enums: enums define valid values in the schema; seed data is actual rows inserted at deploy time.
>
> **What belongs here:**
> - Lookup/reference table rows the application queries at runtime (e.g. notification channel types, system roles, default policy templates)
> - Configuration records required before any user action can succeed (e.g. a default organisation config row, a system user for automated actions)
> - Any row a story's acceptance criteria implicitly assumes exists at test time
>
> **What does NOT belong here:**
> - Test data or demo data — those belong in a separate seed script not deployed to production
> - Data that users create through the UI — that is application data, not seed data
> - Enum values — those are already in Section 4.3
>
> **Implementation:** The implementer creates a single `seed.ts` script in `src/db/seed.ts`. It runs after migrations (`npm run db:seed`) and is idempotent — running it twice produces no duplicates. The implementer also creates an EPIC-001 story for the seed script.
>
> **If this project has no seed data** (all lookups are enums, no runtime-required rows), write: "No seed data required — all lookup values are enums declared in Section 4.3."

| Table | Row description | Key fields |
|-------|----------------|-----------|
| [table_name] | [What this row represents and why it must exist at startup] | [field: value, field: value] |

---

### 5. API Contracts

> **[AI Guide]** Every endpoint the implementer will implement must appear in this section, organised by module. The implementer does not create endpoints not listed here.
>
> **Base path:** `/api/v1/`. Every endpoint requires `Authorization: Bearer <token>` unless explicitly marked [public].
>
> **`orgId` rule:** `orgId` is always extracted from the JWT payload on the server. It must never appear in a URL path, query parameter, or request body. A DTO that includes `orgId` as a field is a security defect.
>
> **DTOs:** required for every POST and PATCH endpoint. GET and DELETE endpoints have no request body. Every field in a DTO must have at least one validation rule, using the confirmed backend's own validation mechanism (default illustration: NestJS `class-validator` decorators — `@IsString()`, `@IsUUID()`, `@IsInt()`, etc.; a Spring backend uses Bean Validation annotations like `@NotNull`/`@Size` instead — same rule, different syntax).
>
> **Money fields in DTOs:** always validated as an integer type in the confirmed framework's validation mechanism (`@IsInt()` for NestJS) — never a floating-point type. The client sends the integer amount in the smallest currency unit.
>
> **Response format for list endpoints:** always `{ data: [...], meta: { total, page, perPage } }`. Single-resource endpoints return the resource object directly.
>
> **State transitions** (e.g. `/invoices/:id/send`): use POST with an action path segment. Do not use PATCH to change status — PATCH is for field edits, POST is for state machine transitions.

#### [ModuleName] Endpoints

| Method | Path | Auth | Request Body | Response | Notes |
|--------|------|------|-------------|----------|-------|
| GET | `/api/v1/[resources]` | JWT | — | `{ data: [...], meta: { total, page, perPage } }` | Paginated list |
| POST | `/api/v1/[resources]` | JWT | `Create[Resource]Dto` | Resource object | 201 on success |
| GET | `/api/v1/[resources]/:id` | JWT | — | Resource object | |
| PATCH | `/api/v1/[resources]/:id` | JWT | `Update[Resource]Dto` | Resource object | |
| DELETE | `/api/v1/[resources]/:id` | JWT | — | 204 No Content | Soft delete only |
| POST | `/api/v1/[resources]/:id/[action]` | JWT | Action payload if needed | Updated resource | State transition |

**DTO definitions:**

*(Default illustration — NestJS `class-validator`. Translate into the confirmed backend framework's own validation mechanism if different, keeping the same fields and rules.)*

```typescript
export class Create[Resource]Dto {
  @IsUUID()
  relatedEntityId: string;

  @IsString()
  @MaxLength(100)
  name: string;

  // Money fields: always @IsInt() — never @IsNumber() or float
  @IsInt()
  @Min(1)
  amountSmallestUnit: number;

  @IsISO8601()
  dueDate: string;
}

export class Update[Resource]Dto extends PartialType(Create[Resource]Dto) {}
```

[Repeat per module]

---

### 6. Security Design

#### 6.1 Authentication

> **[AI Guide]** The self-rolled AuthModule contract below (register/login/refresh, JWT issued and managed by this codebase) is the default illustration — the common case per `FW-025`. **If the confirmed stack (from `system-architecture.md`) uses this default:** the contract below is mandatory as written — do not remove or modify it. **If a different auth provider is confirmed instead** (e.g. Supabase Auth, Auth0, Clerk — a managed service that issues and manages tokens itself): this codebase verifies tokens rather than issuing them, and the endpoint contract below does not apply — document the actual auth flow (how the frontend obtains a session, how the backend verifies it) in its place. This is not a Section 9 deviation — it's whatever `system-architecture.md` confirms; Section 9 only tracks departures from that file, not from this default.
>
> Two items are project-specific regardless of auth provider:
> - The list of public endpoints (endpoints that do not require auth) — always list these explicitly. An endpoint missing from this list is implicitly protected; a protected endpoint that should be public creates a runtime bug
> - Webhook endpoints — these use request signature verification instead of standard auth; list them here

- **Auth method:** JWT Bearer token (`Authorization: Bearer <token>`)
- **Access token TTL:** 15 minutes
- **Refresh token TTL:** 7 days, rotated on every use, stored in `refresh_tokens` table
- **Public endpoints (no JWT required):** [list explicitly — e.g. POST /auth/login, POST /auth/register, POST /auth/refresh]
- **Webhook endpoints (signature verification, not JWT):** [list paths — e.g. POST /webhooks/stripe]

**AuthModule endpoints (default illustration — self-rolled JWT; see AI Guide above for a managed-provider project):**

| Method | Path | Auth | Request Body | Response | Description |
|--------|------|------|-------------|----------|-------------|
| POST | `/auth/register` | Public | `{ email, password, name, orgName? }` | `{ accessToken, refreshToken, user }` | Creates user and org; returns JWT pair |
| POST | `/auth/login` | Public | `{ email, password }` | `{ accessToken, refreshToken, user }` | Validates credentials; returns JWT pair |
| POST | `/auth/refresh` | Public | `Authorization: Bearer <refreshToken>` | `{ accessToken, refreshToken }` | Rotates refresh token; invalidates the old one |
| POST | `/auth/logout` | JWT | — | 204 No Content | Revokes refresh token |
| GET | `/auth/me` | JWT | — | `{ id, email, name, role, orgId }` | Returns the authenticated user |

> **[AI Guide]** Most projects use a single `role: string` claim — one fixed role per user. Some projects define a composable permission model instead: the BRD explicitly describes permissions as independently assignable rather than a fixed role list (e.g. "any user may hold multiple capabilities at once," "capabilities can be assigned in any combination"). For those projects, replace `role` with `capabilities: string[]`, and add `viewOnly: boolean` if the project also has a blanket read-only role — one that mirrors another role's visibility but can take no action anywhere in the system. Do not add either field speculatively; only use the composable shape if the BRD's permission model actually requires it. See Section 6.2 for the corresponding authorisation table.

**JWT payload structure:**
```
{ sub: string, orgId: string, role: string, iat: number, exp: number }
```
or, for a composable capability model:
```
{ sub: string, orgId: string, capabilities: string[], viewOnly: boolean, iat: number, exp: number }
```

**User roles / capabilities defined for this project:** [list all roles from BRD Section 3 — e.g. `org_admin`, `finance_manager`, `finance_officer`, `viewer` — or, for a composable model, list the independently assignable capabilities, e.g. `configuration`, `write_off_approval`, `staff_management`]

---

#### 6.2 Authorisation (RBAC)

> **[AI Guide]** Every role from BRD Section 3 (User Roles) must appear in both tables below. Do not invent roles not in the BRD; do not omit a BRD role from this section.
>
> **Role permissions table:** be specific about what each role can and cannot do. "Admin can do everything" is not sufficient — list the operations. If a role cannot perform a sensitive action (e.g. delete records, approve invoices), state that explicitly.
>
> **State machine transitions table:** list every status-to-status transition in the system and which roles can trigger it. If the system has no state machine (no entities with status fields), write "No state machine transitions — this project has no status-bearing entities."
>
> **Access failure response codes:**
> - Cross-tenant request (record belongs to a different org): always `404` — never reveal the record exists to another tenant
> - Same-org, insufficient role: `403 FORBIDDEN`

| Role | Description | Permitted Actions |
|------|-------------|------------------|
| [role_name] | [Who holds this role and in what context] | [Specific operations permitted — be explicit] |

> **[AI Guide]** If Section 6.1 declares a composable capability model, add the Capability Permissions table below alongside (or instead of, if no fixed roles remain) the Role Permissions table above. One row per capability — do not collapse two capabilities into one row even if their permitted actions happen to overlap today.

**Capability Permissions** *(only if Section 6.1 declares a composable model — otherwise omit this table)*

| Capability | Held by (account type) | Grants access to |
|-----------|------------------------|-------------------|
| [capability_name] | [which account type this capability applies to] | [Specific operations permitted — be explicit] |

**State machine transitions:**

| Transition | From Status | To Status | Permitted Roles |
|-----------|------------|----------|----------------|
| [e.g. Send invoice] | Draft | Sent | [role_name, role_name] |

---

#### 6.3 Delegated Access

> **[AI Guide]** Include this subsection only if the BRD names a delegated or impersonated access pattern — e.g. internal support staff temporarily acting on a customer's behalf under an explicit, revocable grant. If no such pattern exists in the BRD, write: "Not applicable — no delegated access pattern in this project."
>
> When applicable, define:
> - **How the session represents delegation** — e.g. the JWT or session carries the delegated identity's own permissions (role or capabilities) for the duration of the grant, not the actor's own permissions. Do not merge or union the two.
> - **Audit requirement** — every action taken under delegated access must be logged, unconditionally, independent of whatever the base system's normal audit logging covers.
> - **Persistent UI indicator** — the session must be visibly distinguishable as delegated for its entire duration (e.g. a banner naming who is being acted on behalf of, with an explicit exit control). Cross-reference Section 3.1.3's cross-cutting UI rules for where this is specified on the frontend.
> - **Revocation** — how and when the grant can be ended, and what happens to any request already in flight when it is.

[Describe the delegated access pattern, or: "Not applicable — no delegated access pattern in this project."]

---

#### 6.4 Sensitive Fields

> **[AI Guide]** List every field in this project that contains data the system must protect beyond standard authentication — PII, financial data, credentials, regulated data (GDPR personal data, PDPA, HIPAA, PCI DSS). For each field, specify the protection mechanism.
>
> **Common protection mechanisms:**
> - A response-serialisation exclusion, using the confirmed backend's own mechanism — e.g. NestJS `@Exclude()` via `ClassSerializerInterceptor`, or Jackson's `@JsonIgnore` for a Spring backend — field is omitted from serialised API responses
> - Encryption at rest — field value is encrypted before storage; decrypted only when needed
> - Audit log — all reads or writes to this field are written to an audit log
> - Redaction in logs — the field must not appear in application logs even in a truncated form
>
> If this project has no sensitive fields beyond standard user authentication data (email, hashed password), write: "No project-specific sensitive fields beyond standard authentication data — standard response-serialisation exclusion on password hash applies."

| Field | Table | Sensitivity | Protection Mechanism |
|-------|-------|-------------|---------------------|
| [e.g. bank_account_number] | [table_name] | Financial | Excluded from API responses (confirmed framework's serialisation-exclusion mechanism); not logged |

---

#### 6.5 Logging & Exception Handling

> **[AI Guide] (`FW-041`)** Never omit this section or leave it aspirational — state what is actually built, not what a standards file merely describes. A real project shipped with `nestjs-standards.md` describing a global exception filter and `ClassSerializerInterceptor` as if built, for months, when neither existed in code at all; this section exists so that gap is caught here, at generation/review time, not discovered later by a PO reading source code directly. Full detail lives in `standards/logging-standards.md` (Tier 1, always generated per `gen-architecture.md` Step 7) — this section is a summary with pointers, not a duplicate.
>
> Cover, briefly: the correlation-id mechanism (how one request/interaction is traced across every layer that touches it); the exception contract (the global filter/handler mechanism, the typed business-rule-violation exception and its distinct status code, what the caller is and is not shown for an unexpected failure); the redaction policy (cross-reference Section 6.4's sensitive-field table — every field there must also appear in the logger's redact list, in the same change, in perpetuity); the confirmed logging/error-tracking tools (from Section 2's Tech Stack).

Summary: [correlation mechanism] / [exception contract, both the expected and unexpected path] / [redaction — reference Section 6.4] / [confirmed tools, reference Section 2]. Full contract: `standards/logging-standards.md`.

---

### 7. Infrastructure

#### 7.1 Environments

> **[AI Guide]** This table is standard across all Ascendra projects — do not modify the environment names, deployment triggers, or approval requirements. The only exception is the Production approval role — for client projects this may be the client's sign-off authority rather than the CEO; note the specific name.

| Environment | Purpose | Deployment Trigger | Approval Required |
|------------|---------|-------------------|------------------|
| Local | Development | Manual (`docker compose up`) | — |
| QA | Automated test execution | Merge to `develop` branch | — |
| UAT | Customer acceptance testing | Manual | Product Owner (Gate 7) |
| Production | Live system | Manual | Product Owner (Gate 8) |

**Deployment architecture:** each repo in the confirmed topology (Section 3.1) is deployed as its own container on the same internal network — default illustration: Next.js (port 3000) and NestJS (port 3001), same principle for a different confirmed stack. The frontend never accesses the database directly — it always goes through the backend API.

---

#### 7.2 External Services

> **[AI Guide]** List only services that are in scope for this project — services the implementer needs to integrate with. For each service, there must be a corresponding module in Section 3. If a service is listed here with no module in Section 3, the module breakdown is incomplete.
>
> If no external services are in scope (e.g. a purely internal tool), write: "No external services in scope for this project."

| Service | Purpose | Owning Module | Credentials Source |
|---------|---------|--------------|-------------------|
| [e.g. Resend] | [Transactional email] | [EmailModule] | [`RESEND_API_KEY` env var] |

---

#### 7.3 Environment Variables

> **[AI Guide]** List every environment variable the implementer must configure to run the project. This is the authoritative list — the implementer creates the `.env.example` file from this table. An env var not listed here does not get a `.env.example` entry.
>
> The first four rows are standard across all Ascendra projects. Add project-specific variables below them. For any variable marked "If in scope", remove the row if that feature is not in scope for this project — do not leave conditional rows in the final document.

| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_URL` | Database connection string (confirmed database) | Always |
| `REDIS_URL` | Redis connection string | Always |
| `JWT_SECRET` | JWT signing secret — minimum 256-bit random string | Always |
| `ALLOWED_ORIGINS` | CORS allowlist, comma-separated | Always |
| `RESEND_API_KEY` | Resend email service API key | If email in scope |
| `[PSP]_SECRET_KEY` | Payment service provider API key | If payments in scope |
| `[PSP]_WEBHOOK_SECRET` | Webhook signature verification secret | If payments in scope |
| [project-specific variable] | [Purpose] | [Always / If in scope] |

---

### 8. Open Decisions

> **[AI Guide]** Use this section during architecture drafting to park decisions that need Product Owner input before the document can be finalised. Every item here blocks approval — this section must be empty before the document is submitted for approval.
>
> **An architecture document with open decisions cannot be approved.** If you submit a document with items in this section, the Product Owner will return it. Resolve every item — either make a documented decision, or escalate to the Product Owner and record their decision — before submitting.
>
> If all decisions are resolved: replace this entire section with "None — all design decisions resolved before submission."

| # | Decision Required | Options | Recommendation | Blocked By |
|---|-----------------|---------|---------------|-----------|
| 1 | [What must be decided] | [Option A / Option B] | [Recommended option and why] | [What is needed to resolve this] |

---

### 9. Deviation Declarations

> **[AI Guide]** A deviation is any choice in this document that differs from what is recorded in `projects/{PROJECT_CODE}/standards/system-architecture.md` — the tech stack, layering pattern, and repo topology confirmed for this project during `gen-architecture`. This is not about matching some universal framework default; it's about the document staying consistent with what was actually confirmed for this project. Deviations require explicit Product Owner approval — a deviation cannot be self-approved by the AI.
>
> **How to handle a potential deviation:**
> 1. Identify the confirmed component (from `system-architecture.md`) and the proposed alternative
> 2. Write a clear reason why the confirmed choice does not meet this project's needs — "the client prefers X" is not sufficient; "the confirmed choice lacks Y capability required by BRD REQ-xxx" is
> 3. Flag the deviation to the Product Owner before submitting the document for review
> 4. Record the Product Owner's approval decision here
>
> If no deviations: replace this entire section with "None — this project matches the stack, layering, and topology confirmed in `standards/system-architecture.md` in full."

| # | Standard Component | Deviation | Reason the Standard Does Not Fit | Approved By |
|---|------------------|----------|----------------------------------|------------|
| 1 | [e.g. Redis queues] | [e.g. PostgreSQL-based queue] | [Specific reason] | [Product Owner — date] |

---

### 10. Approval

> **[AI Guide]** Leave Decision and Date blank — the Product Owner completes this section. Do not pre-fill or fabricate. Set the Locked on date in the document header only after the Product Owner approves. Before locking, confirm Section 8 is empty and Section 9 is either empty or has all deviations approved.

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | [Name] | Approved / Request Changes / Rejected | |

---

## Part 2 — Verification

> **[AI Guide — Verification]** Run every check below before submitting this document for Product Owner review. A document that fails any check will be returned. The implementer must not begin development until all checks pass and the document is in Approved status.

- [ ] System Overview contains no technical implementation detail — no module names, API routes, table names, or technology choices
- [ ] Tech Stack table covers every layer confirmed in `system-architecture.md`; rows not in scope have been removed (not marked N/A)
- [ ] Every deviation from what's confirmed in `system-architecture.md` (Section 2) has a corresponding entry in Section 9 with a named Product Owner approval
- [ ] Every module in Section 3 has a single, bounded responsibility — no module covers two unrelated domains
- [ ] Section 3.2 is present — either lists extension points derived from BRD Section 1.6, or explicitly states "None"
- [ ] Every table in Section 4.2 is owned by exactly one module listed in Section 3
- [ ] Every Master table in Section 4.2 includes all mandatory Master columns (`id`, `orgId`, `version`, `createdAt`, `updatedAt`, `createdBy`)
- [ ] Every Detail table in Section 4.2 includes all mandatory Detail columns (`id`, `orgId`, `createdAt`, `updatedAt`)
- [ ] Every table with an `updatedAt` column has a trigger declaration in the same migration block
- [ ] No money column in any table uses `decimal` or `float` — all are `integer`
- [ ] Every endpoint in Section 5 references a module that exists in Section 3
- [ ] DTOs are provided for every POST and PATCH endpoint in Section 5
- [ ] No DTO includes `orgId` as a field — `orgId` is always extracted from the JWT on the server
- [ ] Every role from BRD Section 3 appears in the RBAC table in Section 6.2
- [ ] Public endpoints are explicitly listed in Section 6.1 — no unlisted public endpoints
- [ ] Every external service in Section 7.2 has a corresponding module in Section 3
- [ ] Every environment variable in Section 7.3 is referenced somewhere in the document (Section 7.2, Section 4, or Section 5)
- [ ] Section 3.1 specifies both the backend folder structure and the frontend setup approach (ascendra-ui component library copied into ascendra-ui/, no showcase pages)
- [ ] Section 4.4 (Seed Data) is present — either listing seed rows or explicitly stating none are required
- [ ] Every Lookup and Reference table in Section 4.2 has corresponding seed data entries in Section 4.4 (or a stated reason why none are needed at startup)
- [ ] Section 8 (Open Decisions) is empty — replaced with "None" statement
- [ ] Section 9 (Deviation Declarations) explicitly states either no deviations or lists all deviations with named approval
- [ ] If Section 6.1 declares a composable capability model, every capability listed there appears as a row in Section 6.2's Capability Permissions table
- [ ] Section 3.1.3 (Screen & Navigation Map) is present whenever a UI module is in scope — "None — no UI module in scope" is only valid when the project has no UI module at all
- [ ] If Section 3.1.3 is populated, every journey in BRD Section 4 maps to at least one Screen ID in 3.1.3.3
- [ ] Section 6.3 (Delegated Access) is present — either describing the delegated access pattern or explicitly stating "Not applicable"
- [ ] Section 6.5 (Logging & Exception Handling) is present, names actual confirmed tools/mechanisms (not left aspirational), and every field in Section 6.4's sensitive-field table is covered by the redaction policy it references (`FW-041`)
- [ ] Tech Stack table (Section 2) includes a Logging row and an Error Tracking row — never omitted, never "TBD" (`FW-041`)
- [ ] `standards/logging-standards.md` exists and matches what Section 6.5 summarizes

---

## Section 11 — Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/gen-architecture` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/06-architecture.md`'s own tests (the Actor Identity Shape derivation, the deviation-detection blind spot, the composable-capability silent fork, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 12 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this architecture document, this section does not appear at all; do not pre-create it empty.

## Section 12 — Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Change History. Each row records one finding from Section 11's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this architecture document, this section does not appear at all; do not pre-create it empty.

> **[AI Guide — reserved numbering]** Sections 11 and 12 are reserved for `/judgment-check` as shown here. `/review-architecture` creates its own **Section 13 (Review Record)** at the end of the document on first run — never Section 11, to avoid colliding with these two. If `/judgment-check` has not run yet when `/review-architecture` creates Section 13, Sections 11–12 simply don't exist yet; Section 13 still follows immediately after Part 2 (Verification) in that case.
