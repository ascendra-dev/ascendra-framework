# FW-025 — On-the-Fly Architecture and Standards Generation

| Field | Value |
|-------|-------|
| **ID** | FW-025 |
| **Date** | 2026-07-11 |
| **Status** | Decided |
| **Area** | Architecture generation — `gen-architecture`, `arch.template.md` |

---

## Decision

`gen-architecture` determines the best-fit architecture for a project entirely on the fly, from
BRD, domain knowledge, and brief signals plus PO-confirmed constraints — not by selecting from a
predefined, curated library. This applies to every layer of the decision:

- **Tech stack** — recommended from constraints, no fixed "Ascendra standard stack."
- **Standards files** (`api-standards.md`, `database-standards.md`, `security-standards.md`,
  `test-strategy.md`, and any additional project-specific standard) — a baseline four are always
  evaluated; anything beyond that is proposed by the AI based on what the project actually needs
  (compliance domain, integration complexity, module count), not limited to a fixed list.
- **Internal layering pattern** (Controller→Service vs. a fuller layered/DDD structure) and
  **repo/scaffold topology** (api+web vs. additional mobile/worker/etc. repos) — both derived per
  project from BRD signals (platform needs, domain complexity).

A default/fallback exists for each of these (see below) so the common, simple case has a fast
path — but every default is a starting assumption the AI can and should deviate from when project
signals justify it, with deviations declared in Section 9 of the architecture document. Nothing
here is a rigid library the AI selects from; nothing here is a blank slate designed from zero
either.

**Defaults (fallback, not mandate):**

| Dimension | Default | Deviate when |
|-----------|---------|--------------|
| Repo topology | `{code}-api` + `{code}-web` | BRD signals a real additional platform surface (distinct mobile persona, background/scheduled processing) |
| Internal layering | Controller → Service | Domain complexity crosses a threshold (many bounded contexts, rich business-rule/state-machine logic benefiting from isolation) |
| Standards files | 4 baseline (api/database/security/test-strategy) | Project signals warrant an additional standard (e.g. a compliance-specific standard for a regulated domain) |

---

## The Problem That Triggered This Decision

`FW-018` (2026-06-29) proposed a `references/architecture/` pattern-library model: a curated set
of structural patterns the AI would select and adapt from, with community contribution via PR.
That model was judged, in practice, too large in scope — effectively a separate project in its
own right (a library, a contribution process, a scoring mechanism, ongoing curation) rather than
a feature of `gen-architecture`. The Product Owner abandoned the approach informally, but the
decision record was never updated to reflect that. `gen-architecture.md` continued unmodified —
which, as it turned out, meant it was already doing approximately the right thing (free-form,
on-the-fly recommendation), while `FW-018` sat in the decision log reading as current, contradicted
policy. This created confusion during a later architecture-command review: the command and the
decision log disagreed, and there was no record of why.

This decision closes that gap: it retroactively documents the direction that was already being
followed in practice, extends it explicitly to layering and repo topology (which had never been
addressed by either `FW-018` or the original command), and formally supersedes `FW-018`.

---

## Relationship to Other Decisions

- **Supersedes `FW-018`** (Architecture References Model) — the pattern-library approach is not
  pursued.
- Standards files continue to live in `projects/{CODE}/standards/`, never in project identity —
  consistent with `FW-006` at the time this was written, and unaffected by `FW-006`'s later
  supersession by `FW-027` (project.json retired entirely; identity now lives only in
  `projects/index.md`).
