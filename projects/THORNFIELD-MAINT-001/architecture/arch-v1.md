Thornfield Property Management Ltd — Maintenance Ticketing System, Sprint 1, BRD v1.0.

**Status:** Draft
**BRD Version:** v1.0

---

## Part 1 — Template

### 1. System Overview

A NestJS/Drizzle/Postgres backend and a Next.js/Ascendra UI frontend serving the Staff App (Property Manager, Operations Lead), plus two link-based, no-auth surfaces (Tenant report/track, Contractor per-job) that share the same backend. `domain-events`, `audit-logging`, `notification`, `authorization` (Model 1 only), and `optimistic-locking` are adopted from `ascendra-commons` — see Section 3.2 note and `standards/*-pattern.md`.

### 2. Tech Stack

See `standards/system-architecture.md` — the full table and the Actor Identity Shape decision (multiple actor identity spaces; varchar actor-label columns, never FK).

### 3.1 Project Structure

**3.1.1 Backend module layout:** `src/modules/tickets/`, `src/modules/notifications/`, `src/modules/audit/`, `src/modules/crm-sync/`, `src/ascendra-commons/{domain-events,audit-logging,notification,authorization,optimistic-locking}/`, `src/common/`, `src/db/`.

**3.1.2 Frontend route groups:** `(staff)` — Dashboard, Tickets list/detail; standalone `/report`, `/track/[token]`, `/job/[token]` outside any route group (no nav shell, per Screen Design's Portal boundary finding).

**3.1.3 Screen & Navigation Map** (carried over verbatim from `screens/screen-design.md`):

**3.1.3.1 Portals** — Staff App (Property Manager, Operations Lead), standing session. Tenant/Contractor surfaces are not Portals — correctly excluded, see Screen Design Section 1.

**3.1.3.2 Navigation Map** — Dashboard, Tickets (flat, Staff App only).

**3.1.3.3 Screen Inventory** — SCR-001 through SCR-005, per `screen-design.md` Section 3.

**3.1.3.4 Action Gating** — SCR-002 (ticket detail): Set Severity and Assign actions visible only to Property Manager and Operations Lead (both can act); Operations Lead's view is read-heavy in practice per the BRD but not technically restricted from the same actions — no BRD requirement asked for that restriction, so none is added speculatively.

**3.1.3.5 Cross-Cutting UI Rules** — None beyond standard auth gating; no delegated-access or view-only mode was identified in discovery.

### 3.2 Extension Points

None — this is a standalone, single-organization deployment (BRD Section 1.6). No extension dimension named or anticipated.

**Note on `ascendra-commons`:** not the same thing as this section — `ascendra-commons` modules (below) are vendored, copy-distributed backend patterns; Section 3.2 is this project's own architecture extension points for a *future project* to build on. Different, unrelated concepts that happen to share the word "Core," per `practitioner-guide/00-orientation.md`'s terminology map — do not conflate the two.

### 4. Data Model

**`tickets`** (Master)
**Purpose:** The Maintenance Ticket record and its current state.

```typescript
export const tickets = pgTable('tickets', {
  id: uuid('id').primaryKey().defaultRandom(),
  tenancyId: uuid('tenancy_id').notNull(), // synced from CRM, not FK'd to a locally-owned table
  severity: varchar('severity', { length: 20 }).notNull(), // Emergency | Urgent | Routine | Planned
  status: varchar('status', { length: 20 }).notNull(),
  description: text('description').notNull(),
  assignedContractorId: uuid('assigned_contractor_id'),
  linkedTicketId: uuid('linked_ticket_id'), // rare multi-trade case, REQ-006
  version: integer('version').notNull().default(1), // FW-045 — PM and Contractor both mutate this row
  createdAt: timestamp('created_at').notNull().defaultNow(),
  createdBy: varchar('created_by', { length: 200 }).notNull(), // actor-label — Tenant has no FK-able identity
  updatedAt: timestamp('updated_at'),
  updatedBy: varchar('updated_by', { length: 200 }),
  deletedAt: timestamp('deleted_at'),
  deletedBy: varchar('deleted_by', { length: 200 }),
});
```

**`ticket_status_history`** (Detail, immutable — feeds `audit-logging`'s consumption, but is this project's own record, not `audit-logging`'s table)
**Purpose:** One row per state transition, the source REQ-009's timeline is built from.

```typescript
export const ticketStatusHistory = pgTable('ticket_status_history', {
  id: uuid('id').primaryKey().defaultRandom(),
  ticketId: uuid('ticket_id').notNull().references(() => tickets.id),
  fromStatus: varchar('from_status', { length: 20 }),
  toStatus: varchar('to_status', { length: 20 }).notNull(),
  occurredAt: timestamp('occurred_at').notNull().defaultNow(),
  actorLabel: varchar('actor_label', { length: 200 }).notNull(), // same actor-label shape, same reasoning
});
```

**Actor column shape applied consistently:** every Master/Detail table above uses `varchar` actor-label columns, never a FK — matching the confirmed Actor Identity Shape (Section 2 / `system-architecture.md`). A Tenant's `createdBy` on a ticket they reported resolves to a fixed label pattern (e.g. `Tenant (tracking token {short-id})`); a Contractor's `updatedBy` resolves similarly; Staff resolves from their session's name claim; the nightly CRM sync (EPIC-004) writes with a fixed literal (`System (CRM sync)`).

### 5. API Contracts

`POST /tickets` (public, no auth — REQ-001), `PATCH /tickets/:id/triage`, `POST /tickets/:id/assign`, `POST /tickets/:id/accept` (Contractor, token-scoped), `PATCH /tickets/:id/status` (`If-Match` required — `version`-tracked, `FW-045`), `GET /tickets/:id/timeline` (`audit-logging.api`'s pattern, gated by `authorization.core.static-map`).

### 6. Security Design

**6.1 Roles:** Property Manager, Operations Lead (both Staff, authenticated via Google Workspace SSO per brief Section 6). Tenant and Contractor: token-scoped, not role-based in the RBAC sense — a token grants access to exactly one ticket (Tenant) or one assignment (Contractor), never a role.

**6.2 Permitted actions per role:** Property Manager — triage, assign, view own managed properties. Operations Lead — view all, no triage/assignment duties of their own (matches BRD Section 3).

**6.3 Delegated Access:** Not applicable.

**6.4 Sensitive fields:** Tenant contact details (email/phone) — never logged in plaintext; redaction path configured per `standards/logging-standards.md`.

**6.5 Logging & Exception Handling:** Per `standards/logging-standards.md` — Pino/Axiom, `AllExceptionsFilter`, `BusinessRuleViolationException` and `StaleVersionException` (409, `code: 'STALE_VERSION'`, since `tickets.version` uses optimistic locking).

### 7. Infrastructure

Vercel (web), Render (API), Supabase (Postgres). Standard four-environment table (Local/QA/UAT/Production) per `standards/environment-strategy.md`.

### 8. Open Decisions

None.

### 9. Deviation Declarations

None — every choice in Section 2 matches the framework's common default.

### 10. Approval

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | (Simulation) | Approved | 2026-09-16 |

---

## Standards Files Generated

| File | Tier | Why |
|------|------|-----|
| `api-standards.md` | Cross-cutting | Baseline |
| `database-standards.md` | Cross-cutting | Baseline |
| `security-standards.md` | Cross-cutting | Baseline |
| `logging-standards.md` | Cross-cutting | Baseline |
| `test-strategy.md` | Cross-cutting | Baseline |
| `git-standards.md` | Cross-cutting | Baseline |
| `ci-cd-pipeline.md` | Cross-cutting | Baseline |
| `environment-strategy.md` | Cross-cutting | Baseline |
| `nestjs-standards.md` | Per-technology | Confirmed backend |
| `nextjs-standards.md` | Per-technology | Confirmed frontend |
| `domain-events-pattern.md` | Additional (ascendra-commons) | Ticket state changes as events, feeding notification + audit-logging |
| `audit-logging-pattern.md` | Additional (ascendra-commons) | REQ-009/010 compliance timeline |
| `notification-pattern.md` | Additional (ascendra-commons) | REQ-007/008 |
| `authorization-pattern.md` | Additional (ascendra-commons) | Model 1 only — adopted primarily as `audit-logging.api`'s dependency, and sufficient for this project's genuinely simple 2-role RBAC |
| `optimistic-locking-pattern.md` | Additional (ascendra-commons) | `tickets.version` — PM and Contractor both mutate the same row |

**Declined `ascendra-commons` modules, with reasons (per the adoption guide's "declining is normal" principle):** `multi-tenancy` (single-org deployment, no real signal), `background-jobs` (the nightly CRM sync is a single-instance scheduled task at this scale — no multi-instance double-run risk to guard against yet; revisit if deployment scales), `authorization.core.permission-table` (Model 2 — no composable-capability need, 2 static roles is genuinely simple enough for Model 1), and the remaining eight modules (`file-storage`, `rate-limiting`, `payment-link-security`, `payment-gateway-adapter`, `localization-currency`, `soft-delete`, `idempotency-key`) — no BRD requirement names a need for any of them.

*(Only `audit-logging-pattern.md` and `system-architecture.md` were written out in full for this simulation — the remaining standards files are named and their adoption reasoning is recorded above, but their full content was not generated, to keep the simulation's scope tractable. This is noted explicitly rather than silently implied as complete.)*

---

## Part 2 — Verification (abbreviated for simulation)

- [x] Actor Identity Shape stated explicitly and applied consistently across every table in Section 4
- [x] `version` column present on `tickets`, with a matching `If-Match` requirement in Section 5 and a typed exception in Section 6.5
- [x] Section 3.2 states "None" and is explicitly distinguished from the unrelated `ascendra-commons` adoption
- [x] Every adopted `ascendra-commons` module has a real, named BRD signal — none adopted speculatively
- [x] Every declined module has a stated reason, not a silent omission

---

## Section 11 — Density & Judgment Findings

*Generated by `/judgment-check` against `practitioner-guide/06-architecture.md` on 2026-09-16.*

| # | Test (chapter source) | Location | Finding |
|---|---|---|---|
| 1 | "Slow down specifically on Actor Identity Shape even though it's bundled" | Section 2 / `system-architecture.md` | The three-actor-identity-space conclusion is stated but the chapter's own advice is to ask the AI to name every actor kind it counted and why, explicitly — is "Staff / Tenant / Contractor" genuinely exhaustive, or could a fourth kind (e.g. a scheduled job actor for the nightly CRM sync itself) also need its own identity-space treatment? |
| 2 | Section 9's structural blind spot — "the only real guardrail on the Step 5 stack confirmation is attention in the moment" | Section 2, Cache row ("None this phase") | Declining a cache isn't wrong, but was it genuinely considered against a real signal, or defaulted past without scrutiny — same as any other Step 5 row deserves the "why this and not X" question |
| 3 | Core/Extension methodology vs. `ascendra-commons` — "don't conflate the two" | Section 3.2 note | The note correctly distinguishes them, but does it actually help a PO who skims rather than reads carefully — worth a plainer one-line summary right at the top of the note, not just at the end |

## Section 12 — Density & Judgment Resolution

| Date | Finding | PO Response | Detail |
|------|---------|-------------|--------|
| 2026-09-16 | Actor identity space completeness | Confirmed as-is | The nightly CRM sync writes with a fixed literal actor label (`System (CRM sync)`) — it's a *value* within the existing "System/webhook-triggered" pattern the actor-label shape already handles, per `database-standards.md`'s own population-mechanism rule, not a fourth structurally distinct identity space needing its own treatment. Three is correct. |
| 2026-09-16 | Cache decision scrutiny | Confirmed as-is | No hot-path read pattern exists in this domain at this scale (a few hundred tenancies, a handful of tickets a day) — genuinely considered and correctly declined, not defaulted past. |
| 2026-09-16 | Section 3.2 note clarity | Revised | Reordered the note so the one-line distinction comes first, the elaboration after — applied directly above. |
