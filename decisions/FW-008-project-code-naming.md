# FW-008 — Project Code Naming Convention

| Field | Value |
|-------|-------|
| **ID** | FW-008 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | All project codes, `/init-project`, `projects/index.md` |

---

## Decision

Project codes follow the format: `CLIENT-DOMAIN-NNN`

```
HARBORVIEW-INV-001
└─ CLIENT  └─ DOMAIN  └─ sequence
```

| Segment | Description | Rules |
|---------|-------------|-------|
| `CLIENT` | Abbreviated client or organisation name | Uppercase, no spaces, hyphens allowed between words (e.g. `HARBORVIEW`) |
| `DOMAIN` | Abbreviated business domain | Uppercase, short (2–5 chars), describes the problem space (e.g. `INV` for invoicing, `PAY` for payments, `SCH` for school) |
| `NNN` | 3-digit sequence number | Always 3 digits with leading zeros: `001`, `002`, `003` |

### The sequence number

The trailing `NNN` allows a PO to run multiple iterations of the same client+domain scope. `001` is the first iteration. If the project is restarted, restructured, or run again from scratch for the same client and domain, increment to `002`.

This is not a version number — it is an iteration counter. The project code is permanent once assigned. Changing a project code invalidates all artifact paths and is not supported.

### Internal projects

If building for the delivery organisation itself (own product or internal tooling), use the organisation name as the CLIENT segment:

```
ASCENDRA-PAY-001    ← Ascendra building their own payment product
ASCENDRA-ADMIN-001  ← Ascendra building internal admin tooling
```

### Examples

```
HARBORVIEW-INV-001   ← Harborview Consulting, invoice management, first iteration
HARBORVIEW-INV-002   ← Same client and domain, second iteration
HARBORVIEW-SCH-001   ← Same client, school fees domain, first iteration
ASCENDRA-PAY-CORE-001 ← Ascendra, payment core module
ASCENDRA-PAY-PKN-001  ← Ascendra, payment Pakistan market layer
```

---

## Rationale

Year-based codes (e.g. `PRJ-2026-001`) become misleading for long-running projects that span multiple years. Domain-based codes are self-describing — the code tells you who it is for and what domain it covers without opening any file.
