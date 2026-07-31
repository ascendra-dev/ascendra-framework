# FW-003 — Template and Example File Conventions

| Field | Value |
|-------|-------|
| **ID** | FW-003 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | `projects/TEMPLATE/`, all template and example files |

---

## Decision

### File naming

| File type | Suffix | Example |
|-----------|--------|---------|
| Template | `.template.md` | `brief.template.md`, `brd.template.md` |
| Worked example | `.example.md` | `brief.example.md`, `brd.example.md` |

All template files use the `.template.md` suffix — including README files in template folders (e.g. `standards/README.template.md`). No exceptions.

### Co-location

Each `.example.md` file lives beside its `.template.md` in the same folder. Examples are not embedded inside templates.

### Separation of template and example

Templates contain Part 1 (structure) and Part 2 (verification checklist). Examples are extracted into their own `.example.md` files. Commands read templates only — never examples. Examples are for PO reference when filling a document manually.

### The TEMPLATE project

`projects/TEMPLATE/` is a **hard framework dependency**. It must never be deleted, renamed, or moved. It is committed to the repository and tracked. All client project folders (`projects/{PROJECT_CODE}/`) are gitignored. TEMPLATE is not a client project — it is the framework's artifact schema.

---

## Rationale

Separating templates from examples keeps templates lean. Commands that load a template do not accidentally consume example content. Example content can be updated independently of the template structure. Co-location keeps related files discoverable without a separate examples directory.
