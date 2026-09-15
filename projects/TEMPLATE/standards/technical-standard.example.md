Harborview Consulting Ltd — Invoice Management System (BRD v1.1), `api-standards.md` as generated
by `/gen-architecture` Step 7 for HARBORVIEW-INV-001's confirmed stack (NestJS 10+ backend, Next.js
14+ frontend — the framework default per `FW-025`). This example demonstrates: citing the fixed
`reference/api-contract/contract.md` wire contract rather than re-deriving it; the project-specific
additions the template's AI Guide calls for (base path/versioning, the concrete business-rule and
stale-version exception names, which endpoints enforce `Idempotency-Key`, the CORS wiring point);
a Correct/Incorrect pair for the state-transition action convention; and a complete NestJS
controller/service/DTO example applying every rule together on one real endpoint
(`POST /api/v1/invoices/:id/void`).

---

# API Standards

## Purpose

Governs the REST API contract for HARBORVIEW-INV-001's backend (`harborview-inv-001-api`, NestJS
10+, TypeScript strict mode) and its consumption by the frontend (`harborview-inv-001-web`, Next.js
14+). Applies `reference/api-contract/contract.md` (`FW-042`/`FW-045`) as its fixed base — this file
states only what genuinely varies for this project. Deviations from either this file or the fixed
contract require a documented decision in Section 9 (Deviation Declarations) of `arch-v1.md`.

---

## Rules / Standards

### 1. Fixed contract — cite, don't restate

Error response shape, list-response shape, HTTP method semantics (including the
state-transition `POST /:id/{action}` pattern), pagination query params and response fields, and
header conventions (`X-Request-Id`, `Idempotency-Key`, `If-Match`/`ETag`) are pinned in
`reference/api-contract/contract.md`. This project makes no changes to any of them. Any endpoint in
`arch-v1.md` Section 5 that appears to deviate from that file is a defect in Section 5, not a
project-specific variant — fix the section, don't add a local exception here.

### 2. Base URL and versioning

Every route is prefixed `/api/v1/`, set once via `app.setGlobalPrefix('api/v1')` in `main.ts` — never
repeated per-controller. No per-route version segment. The whole API moves to `/api/v2/` only if a
breaking change becomes unavoidable; not needed for this project as of `arch-v1.md`.

### 3. Resource and path naming

Resource segments are plural, lowercase (`/payers`, `/invoices`, `/approvals`). A state-transition
action segment is a lowercase, hyphenated verb, appended to the singular resource path
(`/invoices/:id/mark-paid`, not `/invoices/:id/markPaid` or `/invoices/:id/mark_paid`). Every
status-bearing resource changes state only through one of these action endpoints — contract Rule 5,
restated here because it is the single most common generation mistake this document guards against.

> **Correct:**
> ```
> POST /api/v1/invoices/:id/void
> Body: { "voidReason": "Client requested cancellation" }
> ```
> **Incorrect:**
> ```
> PATCH /api/v1/invoices/:id
> Body: { "status": "void" }
> ```
> — `status` is never a directly writable field on any `PATCH`. A generic `PATCH` on a status-bearing
> resource that silently accepts a `status` field bypasses the state machine in `arch-v1.md` Section
> 6.2 (permitted transitions, permitted roles) entirely.

### 4. Business-rule exception

Any `422` (contract Rule 2) is thrown as `BusinessRuleViolationException`, a typed exception distinct
from validation-pipe failures — the only exception type that populates `code`. Defined in
`nestjs-standards.md`; the global `AllExceptionsFilter` (also `nestjs-standards.md`) gives it its own
branch so `code` is never dropped to `null`. Concrete codes used in this project so far:

| `code` | Thrown when | REQ / Rule |
|---|---|---|
| `PAYER_DUPLICATE_PRIMARY_KEY` | Payer registration matches an existing Payer on the org's configured primary dedup key | REQ-002 |
| `INVOICE_INVALID_VOID_STATE` | `POST /invoices/:id/void` called on an invoice not in `sent` or `approved` status | Section 6.2 state machine |
| `APPROVAL_ALREADY_DECIDED` | A second Director attempts to approve/reject an invoice already out of `pending_approval` | Section 6.2 state machine |

New business rules add a row here, not a new exception type.

### 5. Optimistic concurrency — stale-version exception

A stale `If-Match` (contract Rule 8) is thrown as `StaleVersionException`, `409`,
`code: "STALE_VERSION"` — a distinct exception type from Rule 4's, never a second `code` value on the
same class. Tables carrying `version` (`database-standards.md` Section 4.2, `arch-v1.md` §4.2):
`payers`, `invoices`. Every `PATCH` or `POST /:id/{action}` on these two resources requires `If-Match`.

**Exempted:** the Stripe webhook's write to `invoices` (`POST /webhooks/stripe`) and the dunning
scheduler's overdue/escalation transitions. Both are system-triggered — there is no client-held
version to compare against, so the `If-Match` contract does not apply to them at all, not even as an
optional header. See `idempotent-stripe-webhook-handling.md` (Section 2, "Concurrency") for the full
reasoning on why that path is safe without it.

### 6. `Idempotency-Key` enforcement

Contract Rule 7 makes server-side enforcement opt-in per endpoint. This project enforces it on:

- `POST /invoices/:id/checkout` — creates a Stripe Checkout Session. A retried request without
  enforcement would create a second session and could prompt the Payer to pay twice.

Not enforced elsewhere. Most other mutating endpoints are already naturally idempotent in effect: a
retried Payer registration is caught by Rule 4's dedup check; a retried `void` on an
already-voided invoice is rejected by `INVOICE_INVALID_VOID_STATE`. Enforcing the header there would
add no protection the business rules don't already provide.

### 7. CORS / cross-origin policy

`ALLOWED_ORIGINS` (comma-separated, `arch-v1.md` Section 7.3) is read at bootstrap and wired via
`app.enableCors({ origin: allowedOrigins, credentials: true })` in `main.ts`, before any other
middleware registration. Required because `harborview-inv-001-api` and `harborview-inv-001-web` are
separate repos on separate origins (Section 3.1) — this framework's standard multi-repo topology.

### 8. NestJS controller conventions

- `orgId` is extracted only from `@CurrentUser()` (resolved from the JWT by `JwtAuthGuard`), never
  from a DTO field, a route param, or a query param. No DTO in this project ever declares an `orgId`
  field — enforced by code review, not by tooling.
- Global `ValidationPipe` runs with `whitelist: true, forbidNonWhitelisted: true` — an unrecognised
  body field is a `400`, not silently dropped.
- A mutating handler's parameter order is fixed: `@Param`, `@Headers`, `@Body`, `@CurrentUser` — kept
  consistent across every controller so a reviewer can scan a method signature without re-deriving
  the convention each time.

---

## Examples

### Void invoice — every rule above applied together

```typescript
// src/modules/invoice/dto/void-invoice.dto.ts
export class VoidInvoiceDto {
  @IsString()
  @MinLength(1)
  @MaxLength(500)
  voidReason: string;
}
```

```typescript
// src/modules/invoice/invoice.controller.ts
@Controller('api/v1/invoices')
@UseGuards(JwtAuthGuard, RolesGuard)
export class InvoiceController {
  constructor(private readonly invoiceService: InvoiceService) {}

  @Post(':id/void')
  @Roles('admin', 'finance')
  async void(
    @Param('id', ParseUUIDPipe) id: string,
    @Headers('if-match') ifMatch: string,
    @Body() dto: VoidInvoiceDto,
    @CurrentUser() user: AuthenticatedUser,
  ): Promise<InvoiceResponseDto> {
    if (!ifMatch) {
      throw new BadRequestException('If-Match header is required to void an invoice');
    }
    return this.invoiceService.voidInvoice(id, dto, Number(ifMatch), user);
  }
}
```

```typescript
// src/modules/invoice/invoice.service.ts
async voidInvoice(
  id: string,
  dto: VoidInvoiceDto,
  expectedVersion: number,
  user: AuthenticatedUser,
): Promise<InvoiceResponseDto> {
  const invoice = await this.invoiceRepository.findByIdForOrg(id, user.orgId);
  if (!invoice) {
    throw new NotFoundException(); // cross-tenant or missing — Section 6.2 access failure rule
  }

  if (invoice.version !== expectedVersion) {
    throw new StaleVersionException(
      'This record was modified by someone else since you last loaded it',
    );
  }

  if (!['sent', 'approved'].includes(invoice.status)) {
    throw new BusinessRuleViolationException(
      `Invoice ${invoice.referenceNumber} cannot be voided from status '${invoice.status}'`,
      'INVOICE_INVALID_VOID_STATE',
    );
  }

  const voided = await this.invoiceRepository.voidInvoice(
    id,
    { voidReason: dto.voidReason, voidedAt: new Date() },
    { updatedBy: user.id }, // actor column — database-standards.md
  );

  return InvoiceResponseDto.fromEntity(voided);
}
```

Demonstrates, in one path: Rule 3 (`POST /:id/void`, not a `PATCH` on `status`), Rule 4
(`BusinessRuleViolationException` with a project-specific `code`), Rule 5 (`If-Match` required and
checked before the business-rule check — a stale read fails fast, before the server evaluates a rule
against data the client hasn't seen yet), Rule 8 (`orgId` sourced only from `@CurrentUser()`), and the
`updated_by` actor column populated explicitly by the service, never left to an ORM hook
(`database-standards.md`).
