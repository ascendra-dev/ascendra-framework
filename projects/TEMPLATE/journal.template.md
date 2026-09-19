# Project Journal — [Project Name]

> **[AI Guide — Document Level]**
> This is a continuous, chronological record of this project's own journey through the framework — PO reactions, friction, strengths, drift resolutions, judgment-check findings, documentation gaps, limitations, and workarounds. It is produced by `/anchor-project` only, appended to at the natural points anchor already passes through across every stage of the lifecycle — it is never generated in one pass and never gates anything.
>
> **What this document is not:**
> - It is not a deliverable artifact. It never goes through an approval gate, carries no Status field, and has no Document Control table — per `conventions/template-conventions.md` T-042, the exception this file's own type is governed by.
> - It is not `observations/`. This file is per-project and lives in `projects/{PROJECT_CODE}/`, gitignored, private to this project. `observations/` (framework root, git-tracked) is the cross-project, framework-facing distillation this file feeds, periodically, through extraction — see `decisions/FW-050-project-journal-and-framework-learning.md`. An entry here is raw material; an observation is a reviewed, promotable candidate built from a real pattern across entries.
> - It is not a testimonial. A testimonial is a polished narrative generated *from* this file at a real milestone (Walking Skeleton handoff, project close) — it is never written directly into this file.
>
> **File location:** `projects/{PROJECT_CODE}/journal.md`. Created empty by `/init-project` as part of its normal project skeleton.
>
> **When writing this file (`/anchor-project`):** append a new entry at the natural points anchor already passes through — a drift flag found or resolved, the PO reacting to something (prompted or not, positive or negative), a `/judgment-check` finding, friction or a documentation gap surfacing while wearing an invoked command's hat, a workaround applied to keep moving. The bar for writing is deliberately low — capture anything real. Filtering happens later, at extraction, with the benefit of hindsight and patterns across multiple entries — not here, in the moment, where a single incident's significance often isn't clear yet.
>
> **When reading this file (`/anchor-project`, at extraction only):** read only entries after the Index's "Extracted Through" pointer below — never the whole file from the top. After an extraction pass completes, advance that pointer to the last entry it processed, so the next pass doesn't reprocess old ground.
>
> **On the Index's placement:** per `conventions/template-conventions.md` T-042, this file's resume-relevant pointer sits at the *top*, not as a penultimate Resume Instructions section like a discovery-state file (T-040/T-041) — this file is append-only and read top-to-bottom chronologically rather than resumed-and-regenerated each session, so the pointer needs to be visible before scrolling past however many entries already exist.

---

## Index

> **[AI Guide]** Keep this table current — update it every time an entry is appended or an extraction pass completes.

| Field | Value |
|-------|-------|
| Project Code | [PROJECT_CODE] |
| Entries recorded | [count] |
| Extracted through | [entry number or date — "None yet" if no extraction has run] |
| Last extraction | [date — "Never" if none yet] |

---

## Entries

> **[AI Guide]** One block per event, appended in chronological order — oldest first, numbered sequentially starting at 1. The number, not the date, is what the Index's "Extracted through" pointer references — dates alone aren't unique enough once more than one entry lands on the same day. Never edit or remove an entry after it's written; a correction is a new entry, not a silent rewrite of an old one. One entry, one event — do not bundle several unrelated things into a single block.
>
> **Category — the only recognized set, used consistently:**
> - `PO Feedback` — a reaction volunteered by the PO, quoted verbatim wherever possible, not paraphrased. Positive or negative both count.
> - `Strength` — something that worked cleanly and is worth reinforcing, not just fixing what's broken.
> - `Friction` — something that didn't work smoothly, whether in anchor's own orchestration or whichever command it was wearing the hat of at the time.
> - `Drift Resolution` — a drift flag (anchor's own cross-artifact coherence checks) surfaced and how it actually got resolved.
> - `Judgment-Check Finding` — something `/judgment-check` caught.
> - `Documentation Gap` — a practitioner-guide or command-instruction gap noticed in passing.
> - `Limitation` — a known capability boundary actually hit, not speculated about.
> - `Workaround` — anchor or the PO had to route around something to keep moving.

### Entry [N] — [Date] — [Phase/Stage] — [Category]
[Body — a verbatim PO quote, or a short, concrete description of what happened. Specific enough that someone reading this months later, out of context, still understands what occurred.]

---

## Pre-Extraction Verification

> **[AI Guide — Verification]** Run before treating this file as ready for an extraction pass to read.

- [ ] Every entry has a Date, a Phase/Stage, and a Category drawn from the closed list above — no untyped or invented categories
- [ ] No entry has been edited or removed after being written — a correction appears as its own new entry
- [ ] The Index's "Entries recorded" count matches the actual number of entries below it
- [ ] The Index's "Extracted through" pointer reflects the last completed extraction pass, if any
