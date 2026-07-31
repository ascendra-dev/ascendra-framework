# FW-001 — Document Versioning Rule

| Field | Value |
|-------|-------|
| **ID** | FW-001 |
| **Date** | 2026-06-27 |
| **Status** | Decided |
| **Area** | All artifact types |

---

## Decision

File versioning (`-v1`, `-v2` suffix in the filename) is used **only** when another artifact holds a stable cross-reference to a specific version that must remain readable after the original evolves.

All other artifacts use a **single file** with an internal Document History table to track changes over time.

---

## Rationale

The purpose of retaining an older version file is not general-purpose change history — that is handled by the Document History table inside the file and by Git. The purpose is to keep a specific historical state readable when another artifact has hard-referenced it by version.

The latest version of a versioned document is always the complete, current document. It is not a diff. It contains all content from previous versions plus any changes. Older version files are retained in the project solely because they are referenced by other artifacts that were built against them.

---

## Artifacts that use file versioning

| Artifact | Example | Why |
|----------|---------|-----|
| BRD | `brds/brd-core-v1.md` | Architecture document records `BRD Version: v1.x` — the BRD version the architecture was derived from must stay readable even after a v2 BRD is produced |
| Architecture | `architecture/arch-v1.md` + `arch-v1-ref.md` | `/implement-story` reads `arch-v1-ref.md` on every story; sprint plans are locked against a specific architecture version — that version must remain intact if architecture is revised |

---

## Artifacts that do NOT use file versioning

All other artifact types use a single file with a Document History table:

| Artifact | Naming scheme | Reason versioning is not needed |
|----------|--------------|----------------------------------|
| Brief | `brief.md` | No other artifact holds a hard reference to a specific brief version |
| Discovery State | `discovery-state.md` | Working document — not cross-referenced by version |
| Domain Knowledge | `{slug}-core.md`, `{slug}-{country}.md` | Named by type, not version; no version cross-references |
| Playbook | `{slug}-discovery.md` | Not cross-referenced by version |
| Epics | `EPIC-{NNN}-{slug}.md` | Numerically named; referenced by ID not version |
| Stories | `US-{epic}-{seq}-{slug}.md` | Numerically named; referenced by ID not version — epic-keyed (`FW-028`), stable at generation regardless of sprint assignment |
| Sprint Plan | `sprint-{NN}.md` | One per sprint; numbered not versioned |
| UAT Checklist | `sprint-{NN}-uat-checklist.md` | Tied to sprint number |
| Test Execution Report | `US-{epic}-{seq}-verification.md` | Tied to story ID |
| Release Notes | `v{version}-release-notes.md` | Versioned by software release, not document version |
| PR Description | ephemeral per story | Disposable; not cross-referenced |
| Standards | named files | Living documents; no version cross-references |

---

## File naming convention for versioned artifacts

Version number is always the **last segment before the file extension**:

```
{artifact}-{descriptor}-v{N}.md
```

| Correct | Incorrect | Notes |
|---------|-----------|-------|
| `brd-core-v1.md` | `brd-v1-core.md` | Version must come after descriptor |
| `arch-v1.md` | — | No descriptor — version is already last ✅ |

> **Note (corrected 2026-07-13):** the companion architecture ref file is an established exception to the "version last" rule — every command (`gen-architecture.md`, `review-architecture.md`, `implement-story.md`, `verify-story.md`, `assess-change.md`) and the actual generated artifact use `arch-v1-ref.md`, not `arch-ref-v1.md`. This decision originally stated the reverse form as correct; that was never implemented and the actual convention has been consistent as `arch-v1-ref.md` since the file type was introduced. Treat `{artifact}-v{N}-ref.md` as the correct form for companion ref files specifically — the general `{artifact}-{descriptor}-v{N}.md` rule still governs everything else.

When a new version is produced, increment the number: `brd-core-v1.md` → `brd-core-v2.md`. The descriptor (`core`, `ref`) does not change between versions — it describes the artifact type, not the version.

---

## Rule

> **File versioning is used only when another artifact holds a stable cross-reference to a specific version that must remain readable after the original evolves. All other artifacts use a single file with a Document History table.**
>
> **Version number is always the last segment before the file extension: `{artifact}-{descriptor}-v{N}.md`.**
