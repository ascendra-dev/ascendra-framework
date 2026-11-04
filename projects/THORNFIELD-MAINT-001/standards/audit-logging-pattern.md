# Architectural Pattern — Audit Logging (`ascendra-commons`)

## Document Control

| Version | Date | Reason |
|---------|------|--------|
| 1.0 | 2026-09-16 | Adopted via `/gen-architecture` Step 7, Tier 3 — `ascendra-commons` module evaluation |

**Source frozen at:** `ascendra-commons` commit as of 2026-09-12 (per that repo's own root `README.md` last-updated date — no formal release tooling exists yet, so this is the freezing discipline `reference/ascendra-commons/adoption-guide.md` requires).

---

## 1. Overview

Adopted because REQ-009 and REQ-010 (EPIC-003, Compliance & Audit Timeline) require exactly what this pattern exists to provide: an immutable, timestamped "who did what, to what, when" record, independently retrievable by the Operations Lead for any ticket — the thing Thornfield needs to answer a tenant or Housing Ombudsman dispute. This is not a speculative fit; the epic exists because of this need.

## 2. The Pattern

Cited from `../ascendra-commons/audit-logging/README.md` — not restated as an invention. Feature code raises an `AuditRecordRaised` domain event; the shipped `AuditListener` persists it through an `AuditEventRepository` port. No Model 1/Model 2 config split (unlike `notification`/`authorization`) — one model, split only along delivery seams.

**Packages adopted:**
- `audit-logging.core` — `AuditRecordRaised`, `AUDIT_EVENT_NAME`, `AuditListener`, the repository/reader ports, correlation-id mechanism. Depends on `domain-events.core` (also adopted — see `domain-events-pattern.md`).
- `audit-logging.core.postgres-drizzle` — the real Postgres implementation, matching this project's confirmed database.
- `audit-logging.api` — the read endpoints (`GET /audit/entities/:type/:id/history`) that REQ-010 ("retrieve a ticket's timeline in under a minute") maps directly onto. This package itself depends on `authorization.core` (per the module's own README) — which is why `authorization.core` + `authorization.core.static-map` are adopted alongside this pattern even though this project's own RBAC need is otherwise simple (see `authorization-pattern.md`).

## 3. Model 1 (Adopted)

The single shipped model — `AuditListener` → `AuditEventRepository` (Postgres) → `audit-logging.api`'s read endpoints. No further decision needed; this module has no Model 2 to consider.

## 4. Model 2

Not applicable — this module ships one model only (see Section 2 above).

## 5. The `.core`-only rule, stated explicitly

Every story implementing against this module imports `audit-logging.core` (and, for the admin read endpoints, `audit-logging.api`) only — never reaches into `audit-logging.core.postgres-drizzle` directly from feature code. The Postgres adapter is wired once, at module setup, via `forRoot()`; nothing in `EPIC-003`'s own story code should import a Drizzle table or repository class from that adapter package directly.

## 6. Consuming this module

Per the real README's "Consuming this module" section: copy `audit-logging/audit-logging.core/`, `audit-logging/audit-logging.core.postgres-drizzle/`, and `audit-logging/audit-logging.api/` into `src/ascendra-commons/audit-logging/` on first touch (the first story implementing a EPIC-003 endpoint) — plus `domain-events/domain-events.core/` and `domain-events/domain-events.core.in-process/` (a dependency), and `authorization/authorization.core/` + `authorization/authorization.core.static-map/` (a dependency of `.api`).
