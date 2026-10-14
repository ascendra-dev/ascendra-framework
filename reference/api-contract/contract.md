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

## Rule 4 — Lists get one of two fixed shapes, or a bare array when bounded

**Page-based (the default).** Use this unless one of the exceptions below applies:

```json
{
  "data": [ /* ... */ ],
  "meta": { "page": 1, "perPage": 20, "total": 143, "totalPages": 8 }
}
```

With these exact field names (`perPage`, not `limit`; `totalPages` included, not left for the
client to compute — compute it server-side as `Math.ceil(total / perPage)` alongside the `total`
count, never leave it for the client to derive).

**Cursor-based (the documented exception).** A page-based list requires a `COUNT(*)` (or
equivalent) over the full filtered set on every request. For a table where that count is
expensive to compute (very large or high-write-volume) or where `LIMIT`/`OFFSET` drifts under
concurrent inserts (rows shifting between pages as new ones land), an endpoint may instead use
forward-only keyset pagination:

```json
{
  "data": [ /* ... */ ],
  "meta": { "nextCursor": "eyJvY2N1cnJlZEF0IjoiMjAyNi0xMC0wOVQxMjowMDowMFoiLCJpZCI6IjQyIn0=" }
}
```

`nextCursor` is opaque to the client — pass it back verbatim as a `cursor` query param to fetch
the next page, never decode or construct it. Omit `nextCursor` once there are no more rows. There
is no `total`/`totalPages`/`page` in this shape: a cursor-paginated endpoint cannot answer "how
many total" or "jump to page N" without reintroducing the `COUNT(*)` it exists to avoid — a client
needing either of those should use the page-based shape instead.

This is an explicit **opt-in exception**, not a free choice per endpoint: default to page-based,
and pick cursor-based only for a documented reason (cite it in the project's `api-standards.md` —
e.g. "audit log, append-only, COUNT(*) over the full table is not viable"). Query params for the
cursor-based case: `cursor` (opaque, omit on the first request) and `limit` (same default/max as
`perPage` — default `20`, max `100`), in place of page-based's `page`/`perPage` (Rule 6).

**Bare array (bounded, unpaginated collections).** A collection whose size has a small, fixed
upper bound by nature — enum-like lookups, dropdown options, one entity's own short history —
may return the rows directly as the response body, with no `data`/`meta` wrapper:

```json
[ /* ... */ ]
```

Wrapping such a collection in the page-based shape would only ever report `totalPages: 1`, so the
envelope adds nothing (same reasoning as Rule 3). This does **not** cover a collection that grows
with usage (records, transactions, events) — "we don't need pagination yet" is not a bound, and
retrofitting pagination later is a breaking change for every client. If in doubt, use page-based.
No pagination query params apply; filtering params (Rule 6) still may.

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

Pagination: `page` (1-indexed, default `1`), `perPage` (default `20`, max `100`) — or, for a
cursor-based endpoint (Rule 4's documented exception), `cursor` (opaque, omit on the first
request) and `limit` (same default/max as `perPage`) in place of `page`/`perPage`. Sorting:
`orderBy` — a comma-separated list of sort keys in priority order, each a column name, ascending by
default, with `:desc` appended for descending (`orderBy=issuedAt:desc,amount` — newest first, then
by amount ascending). The server accepts only its documented sortable columns, rejects an unknown
column or a repeated one with `400`, and may cap how many keys it takes. Filtering: one query param
per filterable column, exact-match only unless a route documents range/partial support.

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

## Rule 9 — List exports (CSV download of a full filtered list)

A list that offers an export exposes it as a sibling `GET` route with a `.csv` suffix
(`GET /audit/events` → `GET /audit/events.csv`). It accepts the list's own **filter** params
(Rule 6) unchanged, so a client exports exactly the query it is showing — but **no** pagination
params (`page`/`perPage`/`cursor`/`limit`): the export returns every matching row, not one page.
It is a separate route rather than a flag on the list route so the list keeps exactly one response
shape (Rule 4) and the export can stream.

Every export has a server-side **row cap** (a per-endpoint config value). A larger result is
truncated to the cap, not rejected — the client still receives a file, and is told it's partial.
Exporting more than the cap is an asynchronous export job, out of scope for this route.

The body is the file itself (`Content-Type: text/csv`, header row first). Everything about the
export is carried in response headers, since the body can't hold metadata and the server streams
it:

| Header | Rule |
|---|---|
| `Content-Disposition` | `attachment; filename="<name>.csv"` — the server names the file (e.g. with a timestamp). |
| `X-Export-Row-Count` | Rows in this file (≤ the cap). |
| `X-Export-Truncated` | `true` when more rows matched than the cap allowed, `false` otherwise. |
| `X-Export-Limit` | The cap in effect. |
| `Access-Control-Expose-Headers` | Lists the four headers above — without it a cross-origin browser client can't read them. |

Because headers go out before the body, the server determines the row count and truncation
**before** streaming (a capped count: `COUNT(*)` over the filtered set with `LIMIT cap + 1`), and
pins the result set at request start where it can (e.g. an append-only list fixes its upper time
bound to "now") so rows written mid-export can't make the file disagree with the headers. Errors before streaming starts use Rule 1's shape as usual.

## Explicitly not standardized

- **Auth provider / token issuance** — stays a `system-architecture.md` stack choice.
- **The literal shape of any individual success resource** — Rule 3.
- **Which specific endpoints enforce `Idempotency-Key`** — a per-project decision, recorded in
  that project's own `api-standards.md`.
- **Whether a given table uses optimistic locking at all** — a per-table `database-standards.md`/
  Section 4.2 decision. Rule 8 only fixes the wire shape once that decision has already been made.
