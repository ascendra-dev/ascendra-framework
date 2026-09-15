Harborview Consulting Ltd — Invoice Management System, Sprint 1, BRD v1.1. This example demonstrates: a correct `invoice_status` enum covering all lifecycle states defined in the domain knowledge document (including approval states); the `invoice_approvals` table for the Director approval workflow; the `payers` table with `phoneNumber` and `dedupKeyType` fields required by BRD REQ-001 and REQ-002; the RBAC state machine covering every transition including `pending_approval` → `approved` → `sent`; the extension point for accounting integration identified in BRD Section 1.6; Section 3.1.3's Screen & Navigation Map carried over verbatim from `screen-design.md`; `varchar` actor-label audit columns on every Master table (`FW-037`/`FW-039`, not a `uuid` foreign key); `If-Match` optimistic-concurrency headers on every mutating endpoint against a `version`-tracked table (`FW-045`); Section 6.5's Logging & Exception Handling summary (`FW-041`); and a completed Verification section.

---

**Project:** Invoice Management System
**Version:** 1.0
**Status:** Locked
**Produced by:** Ascendra AI
**BRD Version:** v1.1
**Sprint Plan:** Sprint 1 Plan — approved 2026-07-01
**Locked on:** 2026-07-01

---

### Change History

| Version | Date | Sections Changed | Reason |
|---------|------|-----------------|--------|
| 1.0 | 2026-07-01 | — | Initial submission |

---

### 1. System Overview

Harborview Consulting Ltd manages approximately 40–60 invoices per month through a shared Excel spreadsheet. The Invoice Management System replaces that spreadsheet with a web-based platform where the Finance Manager can create invoices, track payment status in real time, and send automated overdue reminders without manual intervention. Three Directors can approve high-value invoices from any device through a formal approval workflow, replacing the current email-forwarding process.

**Domain:** Finance
**Delivery model:** Custom project
**Estimated complexity:** Medium

---

### 2. Tech Stack

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
| Payments | Stripe | — | — |
| Logging | Pino (structured JSON, shipped to Axiom) | Latest | — |
| Error Tracking | Sentry | Latest | — |
| Containerisation | Docker + Docker Compose | — | — |
| CI/CD | GitHub Actions | — | — |

---

### 3. Module Breakdown

| Module | Responsibility | External Services Used |
|--------|---------------|----------------------|
| AuthModule | JWT issuance, refresh token rotation, login, logout, current-user endpoint | — |
| PayerModule | Payer record management — create, list, retrieve, update, soft delete; deduplication checks on registration; CSV import with per-row import log | — |
| InvoiceModule | Invoice creation, reference number generation, status tracking, line item management, manual Paid marking for bank transfers | — |
| ApprovalModule | Director approval workflow — submit invoice for approval, approve, reject; Director notification trigger; approval record management | — |
| PaymentModule | Stripe checkout session creation, webhook processing, payment status updates | Stripe |
| DunningModule | Overdue detection (scheduled job), overdue reminder scheduling and delivery, escalation to Overdue (Escalated) status, Finance Manager alert on dunning completion | Resend |
| EmailModule | Transactional email delivery via Resend — used by InvoiceModule (invoice sent email), ApprovalModule (approval request and decision notifications), and DunningModule (overdue reminders) | Resend |

---

### 3.1 Project Structure

#### 3.1.1 Backend — `harborview-inv-001-api/`

```
harborview-inv-001-api/
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.module.ts
│   │   │   └── dto/
│   │   ├── payer/
│   │   │   ├── payer.controller.ts
│   │   │   ├── payer.service.ts
│   │   │   ├── payer.module.ts
│   │   │   └── dto/
│   │   ├── invoice/
│   │   │   ├── invoice.controller.ts
│   │   │   ├── invoice.service.ts
│   │   │   ├── invoice.module.ts
│   │   │   └── dto/
│   │   ├── approval/
│   │   │   ├── approval.controller.ts
│   │   │   ├── approval.service.ts
│   │   │   ├── approval.module.ts
│   │   │   └── dto/
│   │   ├── payment/
│   │   │   ├── payment.controller.ts
│   │   │   ├── payment.service.ts
│   │   │   ├── payment.module.ts
│   │   │   └── dto/
│   │   ├── dunning/
│   │   │   ├── dunning.service.ts
│   │   │   ├── dunning.module.ts
│   │   │   └── dunning.scheduler.ts
│   │   └── email/
│   │       ├── email.service.ts
│   │       └── email.module.ts
│   ├── db/
│   │   ├── schema/
│   │   │   ├── auth.schema.ts
│   │   │   ├── payer.schema.ts
│   │   │   ├── invoice.schema.ts
│   │   │   └── approval.schema.ts
│   │   ├── migrations/
│   │   └── index.ts
│   ├── common/
│   │   ├── guards/
│   │   ├── decorators/
│   │   ├── interceptors/
│   │   └── filters/
│   └── main.ts
├── .env.example
├── docker-compose.yml
└── package.json
```

#### 3.1.2 Frontend — `harborview-inv-001-web/`

Copy `ascendra-ui`'s component library (`ascendra-ui/ascendra-ui/`) into `ascendra-ui/`, and its `docs/` into `ascendra-ui/docs/`. Root app shell only (`layout.tsx`, `globals.css`) — no showcase pages. Update `package.json` name to `harborview-inv-001-web`. Create `.env.example` from Section 7.3 (frontend vars only — those prefixed `NEXT_PUBLIC_`).

```
harborview-inv-001-web/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── payers/
│   │   ├── invoices/
│   │   ├── approvals/
│   │   └── settings/
│   ├── layout.tsx
│   └── globals.css
├── ascendra-ui/
│   └── docs/
├── components/
├── hooks/
├── lib/
│   ├── api.ts
│   └── auth.ts
├── .env.example
└── package.json
```

---

### 3.1.3 Screen & Navigation Map

#### 3.1.3.1 Portals

| Portal | Personas | Access model | Base route |
|---|---|---|---|
| Staff App | Finance Manager, Account Owner | Standing session | `/app` |

#### 3.1.3.2 Navigation Map

**Staff App** — flat (no distinct feature-area grouping yet at this scale):

| Nav item | Portal | Visibility condition | Grouping |
|---|---|---|---|
| Dashboard | Staff App | baseline — always visible | flat |
| Payers | Staff App | baseline — always visible | flat |
| Invoices | Staff App | baseline — always visible | flat |

#### 3.1.3.3 Screen Inventory

| Screen ID | Portal | Route | Reachable by | Purpose | Source | Surface | UI Pattern | Matched Reference | Notes |
|---|---|---|---|---|---|---|---|---|---|
| SCR-001 | Staff App | `/app/payers` | baseline | Browse all Payer records | Journey 4.1, step 1, REQ-010 | Page | Table-List | Data Table + Empty State | Empty State candidate — new account |
| SCR-002 | Staff App | overlay on SCR-001 | baseline | Create or edit a Payer record | Journey 4.1, step 1 (Payer must exist before selection), REQ-011 | Sheet | Form | closest: Customer Profile | |
| SCR-003 | Staff App | `/app/invoices` | baseline | Browse all invoices | Journey 4.1, step 4 (post-save landing), REQ-020 | Page | Table-List | Data Table + Page Bar | Empty State candidate — new account |
| SCR-004 | Staff App | `/app/invoices/new` | baseline | Draft and send a new invoice against an existing Payer | Journey 4.1, steps 1–5, REQ-021/022 | Page | Form (Complex) | closest: Create Product Listing (multi-section, mixed grid) | Line-item totals computed inline |
| SCR-005 | Staff App | `/app/invoices/{id}` | baseline | Invoice detail — line items, status, totals | Journey 4.1, step 7; Journey 4.2, steps 6–9 (approval status visible here too), REQ-023 | Page | Detail | Item + Card | |

*(Sections 3.1.3.1–3.1.3.3 carried over verbatim from `projects/HARBORVIEW-INV-001/screens/screen-design.md`, Approved — not re-derived here, per `/gen-architecture`'s Section 3.1.3 guidance.)*

#### 3.1.3.4 Action Gating

Every screen above is reachable by all three roles (`admin`, `finance`, `director` — baseline), but Section 6.2 restricts which of them may act on an invoice. The `director` role opens SCR-004/SCR-005 read-only; it holds no capability that grants any action on either screen.

| Screen ID | Action | Requires |
|---|---|---|
| SCR-004 | Save draft / Send | admin, finance |
| SCR-005 | Edit (Draft only), Send, Void, Mark Paid | admin, finance |

#### 3.1.3.5 Cross-Cutting UI Rules

None — no cross-cutting UI rules beyond standard page-level and action-level gating. Section 6.1 declares no `viewOnly` role, and Section 6.3 (Delegated Access) does not apply to this project.

---

### 3.2 Extension Points

| Extension Point | Mechanism | Owning Module | Extension Projects |
|----------------|-----------|--------------|-------------------|
| `AccountingExportStrategy` | Abstract class — extension registers a provider-specific implementation (Xero, QuickBooks, Sage). Base system exposes stable invoice and payment identifiers; extension registers its own export schedule and format. | InvoiceModule | Planned — not yet scoped (identified in BRD Section 1.6) |

---

### 4. Data Model

#### 4.1 Entity Relationship Summary

An Org has many Users and many Payers. A Payer belongs to one Org and has many Invoices. An Invoice belongs to one Payer and one Org and has many LineItems — each LineItem belongs to one Invoice only. Invoices above the Director approval threshold have one ApprovalRecord before they can be sent. A sent Invoice may have many DunningLogs, one per automated reminder delivered. Payments are confirmed via Stripe webhook and recorded directly on the Invoice (no separate Payment table — the `paid_at` timestamp and Stripe event ID are stored on the invoice row). Processed Stripe event IDs are stored in a `stripe_events` table to enforce idempotency.

---

#### 4.2 Table Definitions

---

##### payers

**Type:** Master
**Module:** PayerModule
**Purpose:** Stores Harborview's Payers — the companies that receive invoices.

```typescript
export const payers = pgTable(
  'payers',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    version: integer('version').notNull().default(1),
    name: varchar('name', { length: 200 }).notNull(),
    email: varchar('email', { length: 254 }).notNull(),
    phoneNumber: varchar('phone_number', { length: 30 }).notNull(),
    dedupKeyType: dedupKeyTypeEnum('dedup_key_type').notNull().default('phone'),
    deletedAt: timestamp('deleted_at', { withTimezone: true }),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
    createdBy: varchar('created_by', { length: 200 }),  // actor label, e.g. "James Okafor"; null = system action
    updatedBy: varchar('updated_by', { length: 200 }),
    deletedBy: varchar('deleted_by', { length: 200 }),
  },
  (table) => ({
    orgIdx: index('payers_org_idx').on(table.orgId),
    emailOrgIdx: index('payers_email_org_idx').on(table.email, table.orgId),
    phoneOrgIdx: index('payers_phone_org_idx').on(table.phoneNumber, table.orgId),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON payers
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

##### invoices

**Type:** Master
**Module:** InvoiceModule
**Purpose:** Stores invoice records — one row per invoice issued by Harborview to a Payer.

```typescript
export const invoices = pgTable(
  'invoices',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    version: integer('version').notNull().default(1),
    referenceNumber: varchar('reference_number', { length: 20 }).notNull().unique(),
    payerId: uuid('payer_id').notNull().references(() => payers.id),
    status: invoiceStatusEnum('status').notNull().default('draft'),
    subtotalPence: integer('subtotal_pence').notNull(),
    vatPence: integer('vat_pence').notNull(),
    totalPence: integer('total_pence').notNull(),
    dueDate: date('due_date').notNull(),
    sentAt: timestamp('sent_at', { withTimezone: true }),
    paidAt: timestamp('paid_at', { withTimezone: true }),
    stripePaymentIntentId: varchar('stripe_payment_intent_id', { length: 100 }),
    voidedAt: timestamp('voided_at', { withTimezone: true }),
    voidReason: text('void_reason'),
    deletedAt: timestamp('deleted_at', { withTimezone: true }),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
    createdBy: varchar('created_by', { length: 200 }),  // actor label, e.g. "James Okafor"; null = system action
    updatedBy: varchar('updated_by', { length: 200 }),
    deletedBy: varchar('deleted_by', { length: 200 }),
  },
  (table) => ({
    orgIdx: index('invoices_org_idx').on(table.orgId),
    payerIdx: index('invoices_payer_idx').on(table.payerId),
    statusIdx: index('invoices_status_idx').on(table.status),
    refIdx: index('invoices_ref_idx').on(table.referenceNumber),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON invoices
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

##### invoice_line_items

**Type:** Detail
**Module:** InvoiceModule
**Purpose:** Stores individual line items within an invoice — immutable once the invoice is sent.

```typescript
export const invoiceLineItems = pgTable(
  'invoice_line_items',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    invoiceId: uuid('invoice_id').notNull().references(() => invoices.id),
    description: varchar('description', { length: 500 }).notNull(),
    quantity: integer('quantity').notNull(),
    unitPricePence: integer('unit_price_pence').notNull(),
    lineTotalPence: integer('line_total_pence').notNull(),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => ({
    orgIdx: index('invoice_line_items_org_idx').on(table.orgId),
    invoiceIdx: index('invoice_line_items_invoice_idx').on(table.invoiceId),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON invoice_line_items
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

##### invoice_approvals

**Type:** Detail
**Module:** ApprovalModule
**Purpose:** Stores the Director approval decision for a high-value invoice. One record per invoice approval lifecycle — created when the first Director acts (approve or reject).

```typescript
export const invoiceApprovals = pgTable(
  'invoice_approvals',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    invoiceId: uuid('invoice_id').notNull().references(() => invoices.id),
    directorId: uuid('director_id').notNull().references(() => users.id),
    decision: approvalDecisionEnum('decision').notNull(),
    rejectionReason: text('rejection_reason'),
    decidedAt: timestamp('decided_at', { withTimezone: true }).notNull().defaultNow(),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => ({
    orgIdx: index('invoice_approvals_org_idx').on(table.orgId),
    invoiceIdx: index('invoice_approvals_invoice_idx').on(table.invoiceId),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON invoice_approvals
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

##### dunning_logs

**Type:** Detail
**Module:** DunningModule
**Purpose:** Records each automated overdue reminder email sent for an invoice.

```typescript
export const dunningLogs = pgTable(
  'dunning_logs',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    invoiceId: uuid('invoice_id').notNull().references(() => invoices.id),
    reminderNumber: integer('reminder_number').notNull(),
    sentAt: timestamp('sent_at', { withTimezone: true }).notNull().defaultNow(),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => ({
    orgIdx: index('dunning_logs_org_idx').on(table.orgId),
    invoiceIdx: index('dunning_logs_invoice_idx').on(table.invoiceId),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON dunning_logs
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

##### stripe_events

**Type:** Detail
**Module:** PaymentModule
**Purpose:** Records processed Stripe webhook event IDs to enforce idempotency — prevents double-processing of the same payment event.

```typescript
export const stripeEvents = pgTable(
  'stripe_events',
  {
    id: uuid('id').primaryKey().defaultRandom(),
    orgId: uuid('org_id').notNull().references(() => orgs.id),
    stripeEventId: varchar('stripe_event_id', { length: 100 }).notNull().unique(),
    processedAt: timestamp('processed_at', { withTimezone: true }).notNull().defaultNow(),
    createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
    updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => ({
    orgIdx: index('stripe_events_org_idx').on(table.orgId),
    eventIdIdx: index('stripe_events_event_id_idx').on(table.stripeEventId),
  }),
);
```

```sql
CREATE TRIGGER set_updated_at
BEFORE UPDATE ON stripe_events
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

#### 4.3 Enums

```typescript
export const invoiceStatusEnum = pgEnum('invoice_status', [
  'draft',
  'pending_approval',
  'approved',
  'sent',
  'overdue',
  'overdue_escalated',
  'paid',
  'void',
  'cancelled',
]);

export const approvalDecisionEnum = pgEnum('approval_decision', [
  'approved',
  'rejected',
]);

export const dedupKeyTypeEnum = pgEnum('dedup_key_type', [
  'phone',
  'email',
  'either',
]);

export const userRoleEnum = pgEnum('user_role', [
  'admin',
  'finance',
  'director',
]);
```

---

#### 4.4 Seed Data

No seed data required — all lookup values are PostgreSQL enums declared in Section 4.3. User roles (`admin`, `finance`, `director`) are defined as enum values in `user_role`. No runtime-required reference rows exist at startup; all application data is created through user actions.

---

### 5. API Contracts

**Optimistic concurrency (`FW-045`):** `payers` and `invoices` both carry a `version` column (Section 4.2). Every mutating request against either resource — `PATCH`, `DELETE`, or a state-transition `POST` action — requires an `If-Match` request header set to the `version` last read for that row (from the single-resource `GET`'s `ETag` response header, or a list row's `version` field); a stale value is rejected with `409 Conflict`, `code: "STALE_VERSION"`. Wire shape is fixed by `reference/api-contract/contract.md` Rule 8 — not restated per endpoint below.

#### PayerModule Endpoints

| Method | Path | Auth | Request Body | Response | Notes |
|--------|------|------|-------------|----------|-------|
| GET | `/api/v1/payers` | JWT | — | `{ data: [...], meta: { total, page, perPage } }` | Filtered to org from JWT |
| POST | `/api/v1/payers` | JWT | `CreatePayerDto` | Payer object | 201 on success; runs deduplication before saving |
| GET | `/api/v1/payers/:id` | JWT | — | Payer object | 404 if cross-tenant; returns current `version` in `ETag` |
| PATCH | `/api/v1/payers/:id` | JWT | `UpdatePayerDto` | Payer object | Requires `If-Match` header (`FW-045`) |
| DELETE | `/api/v1/payers/:id` | JWT | — | 204 No Content | Soft delete — sets `deleted_at`. Requires `If-Match` header (`FW-045`) |
| POST | `/api/v1/payers/import` | JWT | `multipart/form-data` (CSV file) | Import result: `{ imported, failed, log: [...] }` | Per-row import log; creates new Payer rows only, no `If-Match` needed |

```typescript
export class CreatePayerDto {
  @IsString()
  @MaxLength(200)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MaxLength(30)
  phoneNumber: string;
}

export class UpdatePayerDto extends PartialType(CreatePayerDto) {}
```

---

#### InvoiceModule Endpoints

| Method | Path | Auth | Request Body | Response | Notes |
|--------|------|------|-------------|----------|-------|
| GET | `/api/v1/invoices` | JWT | — | `{ data: [...], meta: { total, page, perPage } }` | Supports `?status=` and `?payerId=` filters |
| POST | `/api/v1/invoices` | JWT | `CreateInvoiceDto` | Invoice object | Assigns reference number on creation |
| GET | `/api/v1/invoices/:id` | JWT | — | Invoice object with line items | 404 if cross-tenant; returns current `version` in `ETag` |
| PATCH | `/api/v1/invoices/:id` | JWT | `UpdateInvoiceDto` | Invoice object | Draft only — Sent invoices cannot be edited. Requires `If-Match` header (`FW-045`) |
| POST | `/api/v1/invoices/:id/send` | JWT | — | Invoice object | Transitions Draft → Sent (below threshold) or Draft → Pending Approval (at/above threshold). Requires `If-Match` header (`FW-045`) |
| POST | `/api/v1/invoices/:id/void` | JWT | `VoidInvoiceDto` | Invoice object | Sent or Approved → Void; requires void reason. Requires `If-Match` header (`FW-045`) |
| POST | `/api/v1/invoices/:id/mark-paid` | JWT | — | Invoice object | Sent/Overdue → Paid for bank transfer payments. Requires `If-Match` header (`FW-045`) |
| DELETE | `/api/v1/invoices/:id` | JWT | — | 204 No Content | Draft only. Requires `If-Match` header (`FW-045`) |

```typescript
export class CreateInvoiceLineItemDto {
  @IsString()
  @MaxLength(500)
  description: string;

  @IsInt()
  @Min(1)
  quantity: number;

  @IsInt()
  @Min(1)
  unitPricePence: number;
}

export class CreateInvoiceDto {
  @IsUUID()
  payerId: string;

  @IsISO8601()
  dueDate: string;

  @IsArray()
  @ArrayMinSize(1)
  @ValidateNested({ each: true })
  @Type(() => CreateInvoiceLineItemDto)
  lineItems: CreateInvoiceLineItemDto[];
}

export class UpdateInvoiceDto extends PartialType(CreateInvoiceDto) {}

export class VoidInvoiceDto {
  @IsString()
  @MaxLength(500)
  voidReason: string;
}
```

---

#### ApprovalModule Endpoints

| Method | Path | Auth | Request Body | Response | Notes |
|--------|------|------|-------------|----------|-------|
| GET | `/api/v1/approvals` | JWT (director, admin) | — | `{ data: [...], meta: { total, page, perPage } }` | Pending approvals queue — invoices in `pending_approval` status for this org |
| POST | `/api/v1/invoices/:id/approve` | JWT (director, admin) | — | Invoice object | Pending Approval → Approved; triggers Finance Manager notification. Requires `If-Match` header (`FW-045`) |
| POST | `/api/v1/invoices/:id/reject` | JWT (director, admin) | `RejectInvoiceDto` | Invoice object | Pending Approval → Draft (editable); records rejection reason. Requires `If-Match` header (`FW-045`) |

```typescript
export class RejectInvoiceDto {
  @IsString()
  @MinLength(10)
  @MaxLength(1000)
  rejectionReason: string;
}
```

---

#### PaymentModule Endpoints

| Method | Path | Auth | Request Body | Response | Notes |
|--------|------|------|-------------|----------|-------|
| POST | `/api/v1/invoices/:id/checkout` | JWT | — | `{ checkoutUrl: string }` | Creates Stripe checkout session; returns Stripe-hosted checkout URL |
| POST | `/webhooks/stripe` | Stripe signature | Stripe webhook event | 200 No Content | No JWT; Stripe signature verified on every request |

---

### 6. Security Design

#### 6.1 Authentication

- **Auth method:** JWT Bearer token (`Authorization: Bearer <token>`)
- **Access token TTL:** 15 minutes
- **Refresh token TTL:** 7 days, rotated on every use, stored in `refresh_tokens` table
- **Public endpoints (no JWT required):** POST `/auth/register`, POST `/auth/login`, POST `/auth/refresh`
- **Webhook endpoints (Stripe signature verification, not JWT):** POST `/webhooks/stripe`

**AuthModule endpoints:**

| Method | Path | Auth | Request Body | Response | Description |
|--------|------|------|-------------|----------|-------------|
| POST | `/auth/register` | Public | `{ email, password, name, orgName? }` | `{ accessToken, refreshToken, user }` | Creates user and org; returns JWT pair |
| POST | `/auth/login` | Public | `{ email, password }` | `{ accessToken, refreshToken, user }` | Validates credentials; returns JWT pair |
| POST | `/auth/refresh` | Public | `Authorization: Bearer <refreshToken>` | `{ accessToken, refreshToken }` | Rotates refresh token; invalidates old |
| POST | `/auth/logout` | JWT | — | 204 No Content | Revokes refresh token |
| GET | `/auth/me` | JWT | — | `{ id, email, name, role, orgId }` | Returns authenticated user |

**JWT payload:**
```
{ sub: string, orgId: string, role: string, iat: number, exp: number }
```

**User roles for this project:** `admin`, `finance`, `director`

---

#### 6.2 Authorisation (RBAC)

| Role | Description | Permitted Actions |
|------|-------------|-----------------|
| admin | CFO / Finance team lead (Sarah Chen) | All operations on all resources |
| finance | Finance Manager (James Okafor) | Create/edit/void invoices; manage Payers; import Payers; submit invoices for approval; send approved invoices; manually mark bank transfer invoices as paid; view dashboard |
| director | Invoice approver (Directors) | View invoices (read-only); view approval queue; approve or reject high-value invoices; cannot create, edit, send, or void invoices |

**State machine transitions:**

| Transition | From Status | To Status | Permitted Roles |
|-----------|------------|----------|----------------|
| Send invoice (below threshold) | draft | sent | admin, finance |
| Submit for approval (at/above £10,000) | draft | pending_approval | admin, finance |
| Approve invoice | pending_approval | approved | admin, director |
| Reject invoice | pending_approval | draft | admin, director |
| Send approved invoice | approved | sent | admin, finance |
| Mark overdue (scheduled job) | sent | overdue | System |
| Escalate overdue (dunning complete) | overdue | overdue_escalated | System |
| Mark paid via Stripe webhook | sent, overdue, overdue_escalated | paid | System (webhook) |
| Mark paid manually (bank transfer) | sent, overdue, overdue_escalated | paid | admin, finance |
| Void invoice | sent, approved | void | admin, finance |
| Cancel draft | draft, pending_approval | cancelled | admin |

**Access failure response codes:**
- Cross-tenant request (record belongs to a different org): `404` — never reveal the record exists
- Same-org, insufficient role: `403 FORBIDDEN`

---

#### 6.3 Delegated Access

Not applicable — no delegated access pattern in this project. No BRD role temporarily assumes another user's or Payer's permissions; the Finance Manager, Director, and admin roles each act only under their own standing session.

---

#### 6.4 Sensitive Fields

| Field | Table | Sensitivity | Protection Mechanism |
|-------|-------|-------------|---------------------|
| `email` | `payers` | PII (UK GDPR Art. 6) | `@Exclude()` from list responses; included only in single-Payer GET; not visible to `director` role |
| `phone_number` | `payers` | PII (UK GDPR Art. 6) | `@Exclude()` from list responses; included only in single-Payer GET; not visible to `director` role |
| `password_hash` | `users` | Credential | `@Exclude()` from all responses; never serialised |
| `stripe_payment_intent_id` | `invoices` | Financial identifier | `@Exclude()` from list responses; included only in single-invoice GET for admin and finance roles |

---

#### 6.5 Logging & Exception Handling

Every request carries an `X-Request-Id` — reused if the caller supplies one on the way in, generated otherwise — attached to every log line produced while handling that request and echoed back on the response, so a reported failure is traceable to its exact log lines without asking when it happened. A global `AllExceptionsFilter` catches every unhandled error: an expected business-rule rejection (e.g. sending a Draft invoice below the threshold that has no line items, approving an invoice not currently `pending_approval`) throws a typed `BusinessRuleViolationException` mapped to `422`; a stale `If-Match` (Section 5, `FW-045`) is its own typed exception mapped to `409`, `code: "STALE_VERSION"`; anything unexpected returns a generic `500` with no internal message or stack trace exposed to the caller. Redaction: every field in Section 6.4's sensitive-field table (`email`, `phone_number`, `password_hash`, `stripe_payment_intent_id`) is excluded from log output via a fixed redact-path list configured once at the logger level — a field added to Section 6.4 is added to that list in the same change. Confirmed tools (Section 2): Pino, structured JSON to stdout, shipped to Axiom; Sentry for error tracking, wired into both `harborview-inv-001-api` and `harborview-inv-001-web`. Full contract: `standards/logging-standards.md`.

---

### 7. Infrastructure

#### 7.1 Environments

| Environment | Purpose | Deployment Trigger | Approval Required |
|------------|---------|-------------------|------------------|
| Local | Development | Manual (`docker compose up`) | — |
| QA | Automated test execution | Merge to `develop` branch | — |
| UAT | Customer acceptance testing | Manual | Zaka Shah (Gate 7) |
| Production | Live system | Manual | Zaka Shah (Gate 8) |

**Deployment architecture:** Next.js (port 3000) and NestJS (port 3001) deployed as separate containers on a shared internal Docker network. The frontend never accesses the database directly.

---

#### 7.2 External Services

| Service | Purpose | NestJS Module | Credentials Source |
|---------|---------|--------------|-------------------|
| Stripe | Card payment checkout sessions and webhook payment confirmation | PaymentModule | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` env vars |
| Resend | Transactional email — invoice sent, approval request, approval decision, overdue reminders | EmailModule | `RESEND_API_KEY` env var |

---

#### 7.3 Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `DATABASE_URL` | PostgreSQL connection string | Always |
| `REDIS_URL` | Redis connection string | Always |
| `JWT_SECRET` | JWT signing secret — minimum 256-bit random string | Always |
| `ALLOWED_ORIGINS` | CORS allowlist, comma-separated | Always |
| `RESEND_API_KEY` | Resend email service API key | Always |
| `RESEND_FROM_ADDRESS` | Sender address for outbound emails (`invoices@harborviewconsulting.co.uk`) | Always |
| `STRIPE_SECRET_KEY` | Stripe server-side API key | Always |
| `STRIPE_WEBHOOK_SECRET` | Webhook signature verification secret | Always |
| `APPROVAL_THRESHOLD_PENCE` | Director approval threshold in pence (1000000 = £10,000) | Always |
| `DIRECTOR_EMAIL_ADDRESSES` | Comma-separated list of all Director email addresses for approval notifications | Always |

---

### 8. Open Decisions

None — all design decisions resolved before submission.

---

### 9. Deviation Declarations

None — this project uses the Ascendra standard stack in full.

---

### 10. Approval

| Role | Name | Decision | Date |
|------|------|---------|------|
| Product Owner | Zaka Shah | Approved | 2026-07-01 |

---

## Verification

> **[AI Guide — Verification]** Run every check below before submitting this document for Product Owner review.

- [x] System Overview contains no technical implementation detail — no module names, API routes, table names, or technology choices
- [x] Tech Stack table covers all standard stack layers; File storage row removed (not in scope for this project — no file uploads beyond CSV import, which is in-process)
- [x] Every deviation from the standard stack in Section 2 has a corresponding entry in Section 9 — none present; standard stack used in full
- [x] Every module in Section 3 has a single, bounded responsibility — PayerModule owns payer data; InvoiceModule owns invoice data; ApprovalModule owns approval workflow; no two modules cover the same domain
- [x] Section 3.2 is present — lists one extension point (AccountingExportStrategy) derived from BRD Section 1.6
- [x] Every table in Section 4.2 is owned by exactly one module listed in Section 3: payers → PayerModule; invoices and invoice_line_items → InvoiceModule; invoice_approvals → ApprovalModule; dunning_logs → DunningModule; stripe_events → PaymentModule
- [x] Every Master table in Section 4.2 includes all mandatory Master columns — payers and invoices both have id, orgId, version, createdAt, updatedAt, createdBy/updatedBy/deletedBy as `varchar` actor-label columns, not a `uuid` foreign key (`FW-037`/`FW-039`)
- [x] Every Detail table in Section 4.2 includes all mandatory Detail columns — invoice_line_items, invoice_approvals, dunning_logs, stripe_events all have id, orgId, createdAt, updatedAt
- [x] Every table with an updatedAt column has a trigger declaration — all five tables have a CREATE TRIGGER block
- [x] No money column uses decimal or float — subtotalPence, vatPence, totalPence, unitPricePence, lineTotalPence are all integer
- [x] Every endpoint in Section 5 references a module that exists in Section 3
- [x] DTOs are provided for every POST and PATCH endpoint in Section 5
- [x] No DTO includes orgId as a field — orgId is extracted from the JWT on the server in all modules
- [x] Every role from BRD Section 3 (Finance Manager → finance, Director → director, admin covering CFO/admin) appears in the RBAC table in Section 6.2
- [x] Public endpoints are explicitly listed in Section 6.1 — POST /auth/register, /auth/login, /auth/refresh; POST /webhooks/stripe (signature-verified, not JWT)
- [x] Every external service in Section 7.2 has a corresponding module in Section 3 — Stripe → PaymentModule; Resend → EmailModule
- [x] Every environment variable in Section 7.3 is referenced in the document — DATABASE_URL/REDIS_URL/JWT_SECRET used throughout; RESEND_API_KEY → EmailModule; STRIPE_* → PaymentModule; APPROVAL_THRESHOLD_PENCE → ApprovalModule; DIRECTOR_EMAIL_ADDRESSES → ApprovalModule/EmailModule
- [x] Section 3.1 specifies both the backend folder structure (harborview-inv-001-api/) and the frontend setup approach (ascendra-ui component library copied into ascendra-ui/, no showcase pages)
- [x] Section 4.4 (Seed Data) is present — explicitly states no seed data required with reason
- [x] Every Lookup and Reference table in Section 4.2: none present — all lookup values are enums in Section 4.3
- [x] Section 8 (Open Decisions) is empty — replaced with "None" statement
- [x] Section 9 (Deviation Declarations) explicitly states no deviations
- [x] Section 6.1 declares a standard single `role` claim, not a composable capability model — the Capability Permissions table is correctly omitted from Section 6.2; the flat Role Permissions table alone applies
- [x] Section 3.1.3 (Screen & Navigation Map) is present — Staff App portal, flat navigation map, and Screen Inventory (SCR-001–SCR-005) carried over verbatim from `screen-design.md` (Approved), per `/gen-architecture`'s Section 3.1.3 guidance
- [x] Every journey in BRD Section 4 maps to at least one Screen ID in 3.1.3.3 — 4.1 and 4.2 both covered; 4.3 (Payer payment via email link) is Payer-facing and has no Staff App screen
- [x] Section 6.3 (Delegated Access) is present — states "Not applicable", no delegated or impersonated access pattern exists in the BRD
- [x] Section 6.5 (Logging & Exception Handling) is present, names actual confirmed tools (Pino/Axiom, Sentry — Section 2) and mechanisms (`X-Request-Id` correlation, `AllExceptionsFilter`, `BusinessRuleViolationException`), and covers every field in Section 6.4's sensitive-field table under its redaction policy (`FW-041`)
- [x] Tech Stack table (Section 2) includes a Logging row (Pino) and an Error Tracking row (Sentry) — neither omitted nor left "TBD" (`FW-041`)
- [x] `standards/logging-standards.md` exists and matches what Section 6.5 summarizes
