## Part 3 — Worked Example

US-01-001 — Invoice database schema (Harborview Consulting Ltd, Sprint 1).

---

**PR Title:** `feat(invoice): add invoice schema and migration`

---

### 1. Summary

Adds the `invoices` and `invoice_line_items` database tables via a Drizzle migration, including the `invoice_status` enum and the `set_updated_at` trigger for both tables. This is the foundational schema that all invoice feature stories in Sprint 1 depend on.

Closes HBV-001

---

### 2. Type of Change

- [x] `feat` — new feature
- [ ] `fix` — bug fix
- [ ] `refactor` — no behaviour change
- [ ] `test` — tests only
- [ ] `chore` — tooling, deps, config
- [ ] `perf` — performance improvement

---

### 3. What Changed

- `src/db/schema/invoices.ts` — added `invoices` table definition with all mandatory Master columns, `invoice_status` enum FK, `reference_number` unique varchar, subtotal/VAT/total integer columns (pence), and `due_date`
- `src/db/schema/invoice-line-items.ts` — added `invoice_line_items` Detail table with `invoice_id` FK, description, quantity, unit price, and line total columns
- `src/db/schema/index.ts` — exported both new tables
- `drizzle/migrations/0003_invoice_schema.sql` — generated migration; includes `CREATE TYPE invoice_status`, both table definitions, `set_updated_at` trigger for each table, and org/client/reference indexes
- `src/db/schema/enums.ts` — added `invoiceStatusEnum` pgEnum definition (`draft`, `sent`, `paid`, `overdue`, `cancelled`)

---

### 4. How to Test

1. Starting state: Docker services running (`docker compose up -d`); database has no `invoices` or `invoice_line_items` tables
2. Run `npm run db:migrate` — migration should apply without errors; output shows `0003_invoice_schema.sql` applied
3. Connect to the database (`psql $DATABASE_URL`) and run `\d invoices` — confirm all columns are present including `reference_number`, `subtotal_pence`, `vat_pence`, `total_pence`, `due_date`, `deleted_at`, and all mandatory Master columns (`id`, `org_id`, `version`, `created_at`, `updated_at`, `created_by`)
4. Run `SELECT typname, enumlabel FROM pg_enum JOIN pg_type ON pg_type.oid = pg_enum.enumtypid WHERE typname = 'invoice_status'` — confirm all five enum values are present: `draft`, `sent`, `paid`, `overdue`, `cancelled`
5. Run `\d invoice_line_items` — confirm Detail columns are present (`invoice_id` FK, `description`, `quantity`, `unit_price_pence`, `line_total_pence`) and mandatory Detail columns (`id`, `org_id`, `created_at`, `updated_at`)
6. Run `SELECT tgname FROM pg_trigger WHERE tgrelid = 'invoices'::regclass` — confirm `set_updated_at` trigger is present; repeat for `invoice_line_items`
7. Run `npm run db:migrate` a second time — should be idempotent; no errors, migration shows as already applied

---

### 5. Screenshots

*(Section deleted — backend-only PR, no UI changes)*

---

### 6. Pre-Merge Checklist

- [x] PR title follows Conventional Commits format — `feat(invoice): add invoice schema and migration`
- [x] Type of Change has exactly one box ticked and it matches the PR title type
- [x] All CI checks pass — no failing tests, no lint errors, no build errors
- [x] No `.env` files, secrets, or credentials committed
- [x] New database migrations include the `set_updated_at` trigger for any table with an `updated_at` column — ✓ applied to both `invoices` and `invoice_line_items`
- [x] `orgId` is sourced from the JWT payload — N/A (migration only, no API endpoint)
- [x] React Query cache is invalidated in the relevant mutation hook — N/A (backend-only PR)
- [x] Screenshots are included for any visible UI change — N/A (backend-only; section deleted)
