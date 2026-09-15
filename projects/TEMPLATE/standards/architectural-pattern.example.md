Harborview Consulting Ltd — Invoice Management System (BRD v1.1), `idempotent-stripe-webhook-handling.md`
as an AI-proposed Tier 3 standard from `/gen-architecture` Step 7 — triggered by a concrete signal
(Payment Collection is money-moving and this is the only integration point where an external party,
not this system's own client, initiates the request). Ties directly to Example 3 in
`story.example.md` (US-01-012, Stripe webhook payment confirmation) and to the `stripe_events` table
and `PaymentModule` already defined in `arch.example.md`. This example demonstrates: a single-model
pattern (the "drop the Model N numbering" branch of the template's AI Guide), a per-event-type
handler abstraction resolved by a registry rather than a conditional chain, and an explicit
cross-reference back to `api-standards.md` explaining why this one write path is deliberately exempt
from that file's `If-Match` rule.

---

# Idempotent Stripe Webhook Event Handling — HARBORVIEW-INV-001

## Document Control

| Version | Date | Reason |
|---------|------|--------|
| 1.0 | 2026-07-01 | Initial generation |

---

## 1. Overview

`PaymentModule` receives Stripe webhook events at `POST /webhooks/stripe` to confirm card payments
without manual Finance Manager intervention (BRD REQ-014, REQ-015). Stripe delivers events
at-least-once — the same event can arrive twice (a retry after a slow acknowledgement, a genuine
duplicate delivery) — and delivery order across event types is not guaranteed. Getting this wrong
either double-processes a payment or silently drops one, both of which are money-correctness bugs,
not cosmetic ones. That combination — an externally-triggered, retry-prone, money-moving write path
— is substantial enough to warrant its own document rather than a paragraph inside `arch-v1.md`
Section 3.

---

## 2. The Pattern

### Core abstraction — defined once, never the implementation

```typescript
// src/modules/payment/handlers/stripe-event-handler.interface.ts
export interface WebhookContext {
  orgId: string;
  actor: 'stripe-webhook'; // fixed, centralized literal — database-standards.md actor-column rule
}

export interface StripeEventHandler {
  readonly eventType: string; // e.g. 'checkout.session.completed'
  handle(event: Stripe.Event, ctx: WebhookContext): Promise<void>;
}
```

`StripeWebhookController` and `StripeEventDispatcher` (below) depend only on `StripeEventHandler` —
neither imports a concrete handler class. Adding support for a new Stripe event type never touches
either of them.

### Implementations — never imported by core code directly

```typescript
// src/modules/payment/handlers/checkout-session-completed.handler.ts
@Injectable()
export class CheckoutSessionCompletedHandler implements StripeEventHandler {
  readonly eventType = 'checkout.session.completed';

  constructor(private readonly invoiceService: InvoiceService) {}

  async handle(event: Stripe.Event, ctx: WebhookContext): Promise<void> {
    const session = event.data.object as Stripe.Checkout.Session;
    const invoiceId = session.metadata?.invoiceId;
    if (!invoiceId) {
      return; // not one of ours — ignore rather than throw (Section "Failure Handling")
    }

    const invoice = await this.invoiceService.findByIdForOrg(invoiceId, ctx.orgId);
    if (!invoice || invoice.status !== 'sent') {
      return; // no matching Sent invoice to apply this payment to
    }

    const receivedPence = session.amount_total ?? 0;
    if (receivedPence < invoice.totalPence) {
      await this.invoiceService.flagPaymentDiscrepancy(invoice.id, receivedPence, ctx.actor);
      return; // AC-4 — never partially apply a payment
    }

    await this.invoiceService.markPaid(invoice.id, {
      stripeEventId: event.id,
      paidAt: new Date(),
      actor: ctx.actor,
    });
  }
}
```

```typescript
// src/modules/payment/handlers/payment-intent-succeeded.handler.ts
@Injectable()
export class PaymentIntentSucceededHandler implements StripeEventHandler {
  readonly eventType = 'payment_intent.succeeded';

  constructor(private readonly invoiceService: InvoiceService) {}

  async handle(event: Stripe.Event, ctx: WebhookContext): Promise<void> {
    const intent = event.data.object as Stripe.PaymentIntent;
    const invoiceId = intent.metadata?.invoiceId;
    if (!invoiceId) return;

    const invoice = await this.invoiceService.findByIdForOrg(invoiceId, ctx.orgId);
    if (!invoice || invoice.status !== 'sent') return;

    if (intent.amount_received < invoice.totalPence) {
      await this.invoiceService.flagPaymentDiscrepancy(invoice.id, intent.amount_received, ctx.actor);
      return;
    }

    await this.invoiceService.markPaid(invoice.id, {
      stripeEventId: event.id,
      paidAt: new Date(),
      actor: ctx.actor,
    });
  }
}
```

Both handlers read `metadata.invoiceId` — set by `PaymentModule` on session/intent creation
(`POST /invoices/:id/checkout`) — rather than reverse-searching by amount or timestamp. This is the
one piece of state the two sides of the integration share deliberately, so matching never depends on
fuzzy heuristics.

### Resolution

```typescript
// src/modules/payment/handlers/stripe-event.dispatcher.ts
export const STRIPE_EVENT_HANDLERS = Symbol('STRIPE_EVENT_HANDLERS');

@Injectable()
export class StripeEventDispatcher {
  private readonly handlersByType = new Map<string, StripeEventHandler>();

  constructor(@Inject(STRIPE_EVENT_HANDLERS) handlers: StripeEventHandler[]) {
    for (const handler of handlers) {
      this.handlersByType.set(handler.eventType, handler);
    }
  }

  async dispatch(event: Stripe.Event, ctx: WebhookContext): Promise<void> {
    const handler = this.handlersByType.get(event.type);
    if (!handler) {
      this.logger.log(`Ignoring unhandled Stripe event type: ${event.type}`);
      return; // acknowledged (200), not an error — Stripe must not retry an event we never act on
    }
    await handler.handle(event, ctx);
  }
}
```

The registry is built once, from `PaymentModule`'s provider array (`useValue`/multi-provider
injection of `STRIPE_EVENT_HANDLERS`), not from a `switch` on `event.type` inside the controller. A
future event type (e.g. `charge.refunded`, not yet built) is a new class added to that provider
array — `StripeEventDispatcher` and `StripeWebhookController` are both unchanged.

### Signature verification

```typescript
// src/modules/payment/payment.controller.ts
@Controller('webhooks/stripe')
export class StripeWebhookController {
  constructor(
    private readonly stripe: Stripe,
    private readonly dispatcher: StripeEventDispatcher,
    private readonly stripeEventsRepository: StripeEventsRepository,
  ) {}

  @Post()
  async handleWebhook(
    @Req() req: RawBodyRequest<Request>,
    @Headers('stripe-signature') signature: string,
  ): Promise<{ received: true }> {
    let event: Stripe.Event;
    try {
      event = this.stripe.webhooks.constructEvent(
        req.rawBody,
        signature,
        process.env.STRIPE_WEBHOOK_SECRET,
      );
    } catch {
      throw new BadRequestException('Invalid Stripe signature'); // AC-2 — 400, nothing processed
    }

    const orgId = await this.stripeEventsRepository.resolveOrgForEvent(event);
    const alreadyProcessed = await this.stripeEventsRepository.exists(event.id, orgId);
    if (alreadyProcessed) {
      return { received: true }; // AC-6 — idempotent no-op, no dispatch
    }

    await this.stripeEventsRepository.transaction(async (tx) => {
      await this.stripeEventsRepository.insert({ stripeEventId: event.id, orgId }, tx);
      await this.dispatcher.dispatch(event, { orgId, actor: 'stripe-webhook' });
    });

    return { received: true };
  }
}
```

`main.ts` registers raw-body parsing for this one route only (`rawBody: true` on the Nest
application factory, or an `express.raw()` middleware scoped to `webhooks/stripe`) — JSON
body-parsing anywhere ahead of signature verification corrupts the byte-for-byte payload
`constructEvent()` needs and every signature check fails.

### Idempotency enforcement

This is a distinct mechanism from `api-standards.md` Rule 6's `Idempotency-Key` header, which is
client-initiated (the client generates the key). Here, Stripe is the caller, and Stripe's own
`event.id` is the natural de-duplication key — there is no separate header to invent.

`stripe_events.stripe_event_id` carries a `UNIQUE` constraint (`arch-v1.md` Section 4.2), so the
insert-then-dispatch order above is defence in depth, not the only guard: the pre-check
(`exists()`) makes the common case (a benign retry after Harborview already responded 200) cheap,
and the unique constraint is what actually closes the race if two deliveries for the same event
arrive close enough together to both pass the pre-check. The insert and the handler's invoice
mutation share one transaction, so a crash between them leaves neither committed — a retried
delivery after a crash replays cleanly rather than being mistaken for a duplicate of a
half-applied one.

### Flow

**Phase 1 — Delivery and signature verification.** Stripe POSTs the event. The controller verifies
the signature against the raw body before touching any application state. An invalid signature is
rejected `400` and nothing downstream ever runs.

**Phase 2 — Idempotency check and dispatch.** The controller checks `stripe_events` for
`event.id`. Already present → return `200` immediately, no dispatch (AC-6). Not present → open a
transaction, insert the `stripe_events` row, and hand the event to `StripeEventDispatcher`, which
resolves the matching `StripeEventHandler` by `event.type` (Section 2, Resolution) or logs and
returns if no handler is registered for that type.

**Phase 3 — Handler execution and state change.** The handler resolves the target invoice via
`metadata.invoiceId`, compares the amount received against `invoices.total_pence`, and either calls
`InvoiceService.markPaid()` (status → `paid`, `paid_at` set, audit entry with `actor = 'stripe-webhook'`
per AC-5) or `flagPaymentDiscrepancy()` if the amount is short (AC-4) — never both, and never a
partial status change. The whole transaction commits or rolls back together; the controller responds
`200` to Stripe well within the 5-second budget (AC-7), since no step in this path makes an outbound
network call after the initial signature check.

### Concurrency — why this path does not use `If-Match`

`invoices.version` still increments on `markPaid()`, the same as any other mutation
(`database-standards.md`'s optimistic-locking rule) — this path is not exempt from versioning itself,
only from the `If-Match` wire contract. `api-standards.md` Rule 5 requires `If-Match` on every
client-initiated mutation of a version-tracked resource, because that contract exists to catch a
*client* acting on a version it read earlier that has since gone stale. This path has no such client
read to compare against — Stripe never fetched the invoice's `version` and has no way to supply one.

The concurrency risk this path actually has to guard against is different: two near-simultaneous
Stripe deliveries for the *same* event racing each other, not a client racing a server. That risk is
closed by the `stripe_events` unique constraint and the shared transaction above, not by `If-Match`.
Applying `If-Match` here regardless would only add a header Stripe has no value to fill in — worth
stating explicitly, once, here, so a future reviewer doesn't read `api-standards.md` Rule 5 in
isolation and flag this endpoint as non-compliant.

### Failure handling

- **No matching invoice, or invoice not `sent`** (event references an invoice already paid, voided,
  or belonging to a different org): handler returns without effect, no exception. Not every Stripe
  event is actionable — that is a normal outcome, not an error.
- **Amount received below `invoices.total_pence`:** `flagPaymentDiscrepancy()` records the shortfall
  and alerts the Finance Manager (AC-4); the invoice status is not changed. Applying a partial
  payment as `paid` would silently understate what the Payer still owes.
- **Handler throws an unexpected error:** allowed to surface as `500` via the standard
  `AllExceptionsFilter` (`nestjs-standards.md`) — this is the one endpoint in the system where a
  `500` is the *correct* outcome to let through undisguised, because Stripe's own retry-with-backoff
  is the recovery mechanism, and the shared transaction above guarantees a failed attempt commits
  neither the `stripe_events` row nor the invoice mutation, so the retried delivery is processed as a
  first attempt, not misread as a duplicate.
