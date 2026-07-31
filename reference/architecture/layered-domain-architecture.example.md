# Layered Domain Architecture — Worked Example (Ascendra Pay)

This is a fully worked example applying the methodology in `layered-domain-architecture.md` to one
real project: Ascendra Pay, whose extension dimensions are Market/Country and Sector (see its BRD
Section 1.6). **Read this for illustration only.** Do not copy these entity names, table names, or
extension dimensions into a different project — derive your own from the methodology document,
using this only to see the pattern applied end to end.

---

## Applying the Membership Test

| Thing | Test result | Belongs in |
|-------|------------|-----------|
| `invoices.amount` | Yes — every issuer has invoice amounts | Core |
| `invoices.student_id` | No — not every issuer has students | School extension table |
| `issuer.ntn` | No — NTN is Pakistan-specific | Pakistan extension table |
| `InvoiceService.createInvoice()` | Yes — every issuer creates invoices | Core service |
| `SchoolFeeService.rolloverAcademicYear()` | No — rollover is school-specific | School extension service |
| `InvoicePage` | Yes — every issuer views invoices | Core shell |
| Invoice aging report | Yes — every issuer wants aging | Core report |
| Fee collection by grade | No — grades are school-specific | School extension report |
| Safepay settlement reconciliation | No — Safepay is Pakistan-specific | Pakistan extension report |

---

## Core Domain (Ascendra Pay)

**Core entities:** `Issuer`, `Payer`, `Invoice`, `Payment`, `PaymentMethod`, `PSPTransaction`

**Core services:** `InvoiceService`, `PaymentService`, `IssuerService`, `PayerService`

**Core domain events:** `InvoiceCreated`, `InvoiceStatusChanged`, `PaymentInitiated`,
`PaymentAuthorised`, `PaymentSettled`, `PaymentFailed`, `PaymentRefunded`

## Extensions (Ascendra Pay)

**Pakistan (Market/Country extension):** CNIC/NTN validations, Safepay-specific reconciliation,
FBR e-invoicing compliance.

**School (Sector extension):** `Student`, `AcademicYear`, `FamilyAccount` entities; academic-year
rollover; fee-blocking rules.

```sql
-- Pakistan extends Issuer
CREATE TABLE issuer_pakistan_compliance (
  id          uuid PRIMARY KEY,
  issuer_id   uuid UNIQUE REFERENCES issuers(id),
  ntn         varchar(7)  NOT NULL,
  strn        varchar(13),
  psp_merchant_id varchar
);

-- School extends Issuer
CREATE TABLE school_profiles (
  id                uuid PRIMARY KEY,
  issuer_id         uuid UNIQUE REFERENCES issuers(id),
  school_name       varchar NOT NULL,
  registration_no   varchar,
  school_type       varchar
);

-- School-only entity (no core counterpart)
CREATE TABLE students (
  id                uuid PRIMARY KEY,
  family_account_id uuid REFERENCES family_accounts(id),
  student_name      varchar NOT NULL,
  grade             varchar NOT NULL
);
```

---

## The Payment Adapter Pattern

The most critical application of the strategy pattern in Ascendra Pay. It abstracts the payment
rail so that Core never knows which PSP, network, or bank is processing the payment — enabling the
platform to progress through multiple integration models without touching Core.

```typescript
// core/payment/payment-adapter.interface.ts
interface PaymentAdapter {
  initiate(intent: PaymentIntent): Promise<PaymentInitiationResult>
  handleCallback(payload: unknown, signature: string): Promise<PaymentEvent>
  reconcile(from: Date, to: Date): Promise<ReconciliationResult[]>
}

// extensions/pakistan/adapters/safepay.adapter.ts
class SafepayAdapter implements PaymentAdapter {
  async initiate(intent: PaymentIntent) { /* calls Safepay API, returns redirect URL */ }
  async handleCallback(payload: unknown, signature: string) { /* verifies HMAC, maps to PaymentEvent */ }
  async reconcile(from: Date, to: Date) { /* downloads settlement CSV, maps to ReconciliationResult[] */ }
}

// core/payment/payment.service.ts — holds the interface, never the implementation
class PaymentService {
  constructor(private readonly adapter: PaymentAdapter) {}
  async initiatePayment(intent: PaymentIntent) {
    const result = await this.adapter.initiate(intent)
    // record result, fire PaymentInitiated — no PSP-specific code here
  }
}
```

The active adapter for a given issuer resolves at request time from the extension registry. When
the platform upgrades an issuer's integration model, only the adapter registration changes —
`PaymentService` and all Core code are untouched.

---

## Worked Example: School in Pakistan — Complete Table List

```
Core tables (always present):
  issuers, payers, invoices, payments, payment_methods, psp_transactions, issuer_extensions

Pakistan extension tables:
  issuer_pakistan_compliance   ← extends issuers
  payer_pakistan_identity      ← extends payers
  pakistan_payment_psp_details ← extends payments
  suspense_items                ← Pakistan-layer artefact

School extension tables:
  school_profiles          ← extends issuers
  family_accounts          ← extends payers
  students                 ← school-only entity
  academic_years           ← school-only entity
  school_invoice_details   ← extends invoices

No table in core references any Pakistan or school table.
No Pakistan table references any school table. No school table references any Pakistan table.
```

## Worked Example: Onboarding INSERT Sequence

Traces the creation order for The City School (Pakistan) — the sequence enforces that core records
exist before extension records that reference them.

```sql
-- 1. Core Issuer first — no Pakistan or school columns
INSERT INTO issuers (id, name, org_id) VALUES ('iss-001', 'The City School', 'org-123');

-- 2. Core Payer first
INSERT INTO payers (id, name, email, org_id) VALUES ('pay-001', 'Ahmed Ali', 'ahmed@gmail.com', 'org-123');

-- 3. Register active extensions (drives all runtime decisions)
INSERT INTO issuer_extensions (issuer_id, extension_type) VALUES ('iss-001', 'school'), ('iss-001', 'pakistan');

-- 4 & 5. Extensions extending Issuer — independent of each other, both depend on step 1
INSERT INTO issuer_pakistan_compliance (issuer_id, ntn, strn) VALUES ('iss-001', '2847391', '1234567890123');
INSERT INTO school_profiles (issuer_id, registration_no, school_type) VALUES ('iss-001', 'PEF-REG-4521', 'private');

-- 6 & 7. Extensions extending Payer — independent of each other, both depend on step 2
INSERT INTO payer_pakistan_identity (payer_id, cnic) VALUES ('pay-001', '3520112345671');
INSERT INTO family_accounts (id, payer_id, family_code) VALUES ('fam-001', 'pay-001', 'FAM-2024-0081');

-- 8. School-only entity — depends on step 7
INSERT INTO students (id, family_account_id, student_name, grade) VALUES ('stu-001', 'fam-001', 'Sara Ali', 'Grade 5');
```

## Worked Example: Safepay Payment Flow — Three Phases

**Phase 1 — Initiation:** Core creates the payment record (`status=initiated`) and fires
`PaymentInitiated`. The Pakistan extension handler reacts: generates a merchant order reference,
stores it, calls the Safepay API, returns the redirect URL. Core's `invoices` table is unchanged.

**Phase 2 — Authorisation:** Safepay's webhook arrives at the Pakistan extension, which updates its
PSP details table and calls `PaymentService.authorisePayment()`. Core updates `payments.status` and
`invoices.status`, and fires `PaymentAuthorised`.

**Phase 3 — Settlement:** The Pakistan extension's daily reconciliation job downloads the Safepay
settlement CSV and matches rows by merchant order reference. A match calls
`PaymentService.settlePayment()` (core updates `payments.status='settled'`). No match writes to a
Pakistan-layer `suspense_items` table — core is never called for an unmatched row.

The invoiced amount and the settled amount differ by the Safepay fee. Core's `payments` table holds
the business truth — what was owed and paid. The Pakistan extension table holds the PSP truth —
what was actually received in the bank. Neither contaminates the other.

---

## Worked Reporting Example — Contrasting Queries, Same Core, Different Extensions

**School: Fee Collection by Grade** (core + school extension — cannot run for a non-school issuer,
since `school_invoice_details` and `students` don't exist for them):

```sql
SELECT s.grade, COUNT(DISTINCT s.id) AS total_students, SUM(i.amount) AS total_due
FROM invoices i                                          -- core
JOIN school_invoice_details sid ON sid.invoice_id = i.id -- school extension
JOIN students s ON s.id = sid.student_id                 -- school extension
WHERE i.issuer_id = :issuer_id
GROUP BY s.grade;
```

**Any issuer: Invoice Aging by Client** (core only — runs identically for a school, a hospital, or
a retailer; touches no extension table):

```sql
SELECT p.name AS client, COUNT(i.id) AS invoices, SUM(i.amount) AS total_outstanding
FROM invoices i                     -- core
JOIN payers p ON p.id = i.payer_id  -- core
WHERE i.status IN ('unpaid', 'overdue') AND i.issuer_id = :issuer_id
GROUP BY p.name;
```
