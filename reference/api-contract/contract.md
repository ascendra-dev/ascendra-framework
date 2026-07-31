# Client↔Server API Contract

Applies to every project's backend and frontend regardless of tech stack — this is a wire-level
(HTTP + JSON) convention, not tied to NestJS, Next.js, or any specific framework. Referenced by
`/gen-architecture` when generating `api-standards.md`, and by `/implement-story` when scaffolding
a backend's exception handling and a frontend's API client. A project's `api-standards.md` may add
project-specific detail (which endpoints enforce `Idempotency-Key`, which fields are sensitive) but
never changes the shapes below — they are fixed, the same across every project this framework
generates, so a client-side implementation written against this contract never has to be
re-derived per project.

Close cousin of [RFC 7807 (Problem Details for HTTP APIs)](https://www.rfc-editor.org/rfc/rfc7807)
for the error shape specifically — this isn't an arbitrary invention, it's a simplified, renamed
version of an established IETF pattern for exactly this problem.

---

## Rule 1 — Error responses are always flat, never nested

```json
{
  "statusCode": 422,
  "error": "Unprocessable Entity",
  "message": "A staff member with email 'x@y.com' already exists",
  "code": "STAFF_EMAIL_ALREADY_EXISTS",
  "path": "/api/v1/staff"
}
```

| Field | Rule |
|---|---|
| `statusCode` | Always mirrors the real HTTP status code. |
| `error` | The standard HTTP reason phrase for that status (`"Bad Request"`, `"Unprocessable Entity"`, `"Internal Server Error"`) — never custom text. |
| `message` | Human-readable. `string` normally; `string[]` only when more than one field fails validation at once. |
| `code` | Stable, machine-readable identifier. `null` unless the error is a typed business-rule violation (see Rule 2). The frontend branches on `code`, never on `message` — `message` stays free text, safe to reword without breaking a client. |
| `path` | The request path that failed. |

**Never** wrap this in a `success: false` envelope, and **never** nest it under an `error: {...}`
object — flat only, so it's trivially parseable without a nested type.

## Rule 2 — `400` vs `422`: malformed input vs. a business-rule rejection

A `400` means the input's *shape* was wrong — a missing field, a malformed email — and is thrown
automatically by request validation before business logic ever runs. A `422` means the input was
well-formed and passed validation, but the operation still isn't allowed given the current state
of the system (a duplicate email, a cap already reached). Every `422` is thrown via one typed
exception distinct from a generic validation failure (concretely, `code` is populated only for
this exception type) — name it explicitly in the project's Tier 2 backend standard (e.g.
`BusinessRuleViolationException` for NestJS).

## Rule 3 — Success responses have no forced envelope

Return the resource's natural shape directly, at the correct 2xx status. Do not wrap it in
`{ success: true, data: T }` — the HTTP status code already carries that signal, and a redundant
boolean flag adds boilerplate on every backend framework for zero benefit. Every endpoint's success
shape is typed by its own DTO/interface — there is nothing generic to standardize here.

## Rule 4 — Lists get one fixed shape

```json
{
  "data": [ /* ... */ ],
  "meta": { "page": 1, "perPage": 20, "total": 143, "totalPages": 8 }
}
```

Any endpoint returning a collection uses this shape, with these exact field names (`perPage`, not
`limit`; `totalPages` included, not left for the client to compute).

## Rule 5 — Request method semantics

- **`GET`** — read-only, no body. All filtering/sorting/pagination via query params (Rule 6).
- **`POST`** — create a resource, **or** a state-transition action on an existing one
  (`POST /:id/void`, `POST /:id/remove`). Every status-bearing entity changes state only via a
  dedicated `POST /:id/{action}` endpoint — never `PATCH` on a status field directly.
- **`PATCH`** — partial update of non-status fields.
- **No raw `DELETE`** for anything business-meaningful — destructive or state-changing removal is
  a `POST` action (previous bullet), never a bare `DELETE`. A project may still use `DELETE` for a
  genuinely trivial, non-business-critical resource if its own `api-standards.md` says so
  explicitly.

## Rule 6 — Query parameter conventions (`GET`)

Pagination: `page` (1-indexed, default `1`), `perPage` (default `20`, max `100`). Sorting:
`sortBy` (column name), `sortDir` (`asc` | `desc`). Filtering: one query param per filterable
column, exact-match only unless a route documents range/partial support.

## Rule 7 — Headers, both directions

| Header | Direction | Rule |
|---|---|---|
| `Content-Type: application/json` | Request | Always. |
| `Authorization: Bearer <token>` | Request | Wire shape is fixed; how the token is issued/retrieved is a per-project choice (`system-architecture.md` Section 2) — this contract does not standardize auth provider. |
| `X-Request-Id` | Both | A caller **may** send one inbound (e.g. forwarding an existing ID across a hop); the server reuses it if present, generates one otherwise. The server **always** echoes it on the response, success or error — as a header, never a body field, so a `500` with a malformed body still lets the caller report a traceable reference. |
| `Idempotency-Key` | Request | The client **always** sends one on `POST`/`PATCH`/`PUT`. Server-side enforcement (rejecting/deduplicating a replayed key) is **opt-in per endpoint** — a low-stakes endpoint may simply ignore it; a money-moving or non-repeat-safe endpoint should honor it. |

## Rule 8 — Optimistic concurrency (`version`-tracked resources)

Any resource whose table carries a `version` column (per `database-standards.md`'s optimistic-locking
rule) is concurrency-controlled at the wire level:

- **`GET` of a single resource** returns its current version in an `ETag` response header, quoted
  (e.g. `ETag: "3"`).
- **`GET` of a collection** (Rule 4's list shape) cannot carry one `ETag` for many rows — each
  object in `data[]` instead includes its own `version` field in the body. A client reads the
  version for a specific row from whichever response last returned it (a single-resource `GET`'s
  `ETag`, or a list row's `version` field) — the wire location differs by shape, the value means the
  same thing either way.
- Any mutating request against that resource — a `PATCH`, or a `POST` state-transition/removal
  action (Rule 5) — **must** carry an `If-Match` request header set to the version the client last
  read, regardless of which of the two forms above it came from.
- The server compares `If-Match` against the row's current `version` at write time. A match applies
  the change and increments `version`. A mismatch means the row was modified since the client last
  read it: reject with `409 Conflict`, `code: "STALE_VERSION"`, no partial effect.

```json
{
  "statusCode": 409,
  "error": "Conflict",
  "message": "This record was modified by someone else since you last loaded it",
  "code": "STALE_VERSION",
  "path": "/api/v1/staff/abc123/remove"
}
```

`409` here is reserved specifically for this stale-version case — distinct from Rule 2's `422`,
which rejects a business rule regardless of timing. The request side (`If-Match`) is always a
header, never a request-body field: this keeps the mechanism identical across `PATCH` (which has a
body) and bodyless `POST` action endpoints alike, and reuses the same wire pattern already
established for `Idempotency-Key` (Rule 7) instead of inventing a body-shape variant per endpoint.
The response side varies by shape (single-resource `ETag` header vs. per-row `version` body field on
a list) only because HTTP has no mechanism for a response to carry many `ETag`s at once — this is a
constraint of the response shape, not an inconsistency in the pattern. Close cousin of
[RFC 7232](https://www.rfc-editor.org/rfc/rfc7232) (`If-Match`/`ETag` for conditional requests) — same
rationale as Rule 1's RFC 7807 kinship: an established pattern, not a bespoke invention.

## Explicitly not standardized

- **Auth provider / token issuance** — stays a `system-architecture.md` stack choice.
- **The literal shape of any individual success resource** — Rule 3.
- **Which specific endpoints enforce `Idempotency-Key`** — a per-project decision, recorded in
  that project's own `api-standards.md`.
- **Whether a given table uses optimistic locking at all** — a per-table `database-standards.md`/
  Section 4.2 decision. Rule 8 only fixes the wire shape once that decision has already been made.
