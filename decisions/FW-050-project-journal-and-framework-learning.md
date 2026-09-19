# FW-050 — Project Journal and Framework Learning

| Field | Value |
|-------|-------|
| **ID** | FW-050 |
| **Date** | 2026-09-19 |
| **Status** | Decided — implemented, pending validation against a real project |
| **Area** | `.claude/commands/anchor-project.md`, `.claude/commands/init-project.md`, `projects/TEMPLATE/journal.template.md` + `.example.md`, `conventions/template-conventions.md` (one exception: `T-042`), `observations/index.md` + `TEMPLATE.md`, `CLAUDE.md` |
| **Depends on** | `FW-049` (Anchor Project and Priming Session) — this redesigns the self-improvement logging mechanism `FW-049` originally shipped, using the same one-agent orchestration model |

---

## Decision

Anchor's original self-improvement mechanism (`observations/`, from `FW-049`) had one layer: a high-bar, ad-hoc log written in the moment. Two things it structurally couldn't do: capture the PO's own real-time reactions and judgment calls as they actually happened, and capture a project's ordinary texture and strengths, not just candidate framework bugs decided worth recording on the spot.

This decision replaces it with a two-layer model:

**The Project Journal** (`projects/{PROJECT_CODE}/journal.md`) — a continuous, chronological, per-project record written only by `/anchor-project`, at a deliberately *low* bar. It captures eight typed categories of entry (`PO Feedback`, `Strength`, `Friction`, `Drift Resolution`, `Judgment-Check Finding`, `Documentation Gap`, `Limitation`, `Workaround`), appended at the natural points anchor already passes through — never batched, never filtered in the moment. It is per-project, gitignored, and private, same as every other artifact in `projects/{PROJECT_CODE}/`.

**Extraction** — a distinct, periodic analysis pass, triggered automatically at milestones anchor already recognizes (a `/review-X` gate closing, the Walking Skeleton handoff, project close), never per-entry and never continuously. It reads only journal entries since the last extraction, looks for a real pattern across more than one entry, and produces two different artifacts at two different scales: an **observation** (`observations/OBS-NNN`, every pass — same file shape and promotion discipline `FW-049` already established, now fed by patterns instead of single incidents) and, only at the two largest milestones (Walking Skeleton handoff, project close), a **project testimonial** (`projects/{PROJECT_CODE}/testimonial.md`) — a polished narrative synthesis for a human reader, not a framework-decision candidate.

`observations/` itself is **not removed** — it stays exactly where it already lives (framework root, git-tracked, cross-project) and remains the thing actually worth sharing with the framework author or community, since by construction it has already been generalized away from any specific client's content. The raw journal is never recommended for sharing by default.

---

## Why This Was Needed

Working through a real design conversation about what the framework should learn from its own use surfaced a distinction the original `observations/`-only design conflated: **raw capture** and **deliberate analysis** are different activities with different correct bars. A single mechanism trying to do both at once either captures too little (a high bar applied in the moment misses real signal whose significance isn't clear yet) or, if the bar were lowered to capture more, would turn into noise nobody reviews. Separating them — a low-bar, continuous journal feeding a high-bar, periodic extraction — lets each side do its actual job.

A related problem: the original design had no place for the PO's own real-time reactions, verbatim, as they happened — only Claude's own after-the-fact interpretation of friction. Since anchor is the one thing that sits across a project's entire lifecycle in one continuous, orchestrated session, it's uniquely positioned to capture that directly, without asking the PO to maintain a separate feedback channel themselves.

**One design temptation, considered and rejected:** collapsing `observations/` entirely into the journal, on the reasoning that "everything comes from the journal anyway, so why maintain two things." Rejected because the two artifacts sit on opposite sides of every axis that matters here — the journal is per-project, gitignored, and full of real client-specific content (verbatim PO quotes, real business facts); `observations/` is cross-project, git-tracked, and (by construction, since it's generalized during extraction) safe to share externally. Collapsing them would mean either exposing raw client content by default, or losing the durable, cross-project, shareable artifact the framework author actually needs. The journal is *raw material for* observations now, not a replacement for them — the same "single source of truth, not a redundant parallel file" instinct `FW-027` already established for project identity, applied here to keep the two artifacts from drifting apart, while still keeping each on the correct side of the private/shareable line.

---

## Key Mechanisms

- **Two layers, two different bars.** The journal takes anything real, generously; extraction applies the high bar the original design could only apply in the moment, now with the benefit of hindsight and multiple data points.
- **A closed category vocabulary, not free text** — the same discipline the framework already applies to Flag types elsewhere (`Conflicting Source` and friends in `priming-command-conventions.md`), extended here to journal entries.
- **Extraction is milestone-triggered, never reactive** — the deliberate gap between writing an entry and analyzing it is the actual point, not an inefficiency. This is the direct answer to "don't patch the framework reactively, only from real analysis."
- **The testimonial is a distinct artifact from an observation** — one is a narrative for a human, the other a framework-decision candidate for the maintainer. Generating both from the same extraction pass, at different scales, avoids needing a third mechanism.
- **The reporting-gap placeholder from `FW-049` is unchanged in kind, refined in scope** — still an unsolved real channel to the framework author/community, but now explicitly the *observation* that gets recommended for sharing, never the raw journal.
- **Extraction re-verifies before promoting, never trusts a journal entry on its own** — added same day, after a direct question surfaced that a terse, low-bar entry written gates earlier could describe a gap that's since been fixed. Extraction re-reads the implicated file and confirms the pattern is still current before writing an observation; a stale pattern is not evidence.

---

## Conventions Exception (proposed centrally, per `template-conventions.md`'s own governing principle)

- **`T-042`** — a continuous, append-only session-record template (`journal.template.md` is the first of this kind) keeps its resume-relevant pointer in a top-of-file Index instead of T-040's penultimate Resume Instructions placement, and carries no Document Control table or Status field, for the same reason discovery-state files already don't (T-011, T-014). Added to `template-conventions.md` itself, not improvised inside the journal template.

---

## Relationship to Other Decisions

- **`FW-049`** (Anchor Project and Priming Session) — this decision redesigns the self-improvement logging mechanism `FW-049` shipped, using the same "anchor as the one continuous orchestrator" premise the whole command is built on.
- **`FW-027`** (single source of truth for project identity, no separate `project.json`) — the direct precedent against maintaining a redundant parallel tracking file; applied here to justify why the journal and `observations/` stay separate artifacts rather than collapsing into one, each serving a role the other structurally can't.
- **`FW-024`** (Parking Lot) — a related but distinct precedent for "record it, don't silently drop it, without immediately deciding its significance" — the journal's low-bar capture follows the same spirit, one level earlier in the pipeline.

---

## Implementation Status

Built and self-checked, 2026-09-19:

- [x] `projects/TEMPLATE/journal.template.md` — new file, following `template-conventions.md`'s discovery-state-file pattern with the `T-042` exception
- [x] `projects/TEMPLATE/journal.example.md` — worked example, Harborview-consistent, all eight categories illustrated across a Brief → Domain → BRD arc, entry-numbering scheme validated against a same-day-multiple-entries case
- [x] `conventions/template-conventions.md` — `T-042` exception added, checklist updated
- [x] `.claude/commands/anchor-project.md` — Step 11 rewritten in full (11a journal writing, 11b extraction), Step 12's session-close report extended (Journal/Observations/Testimonial lines), Step 3's Reference Material Index and reconciliation procedure extended with a Journal row/check
- [x] `.claude/commands/init-project.md` — Step 7 extended to create and pre-fill `journal.md`, Step 9's report updated
- [x] `observations/TEMPLATE.md` — "Found while" field repointed at journal entries
- [x] `observations/index.md` — description updated to reflect the extraction-sourced model
- [x] `CLAUDE.md` — folder structure diagram gained `journal.md`, `testimonial.md`, and `journal.template.md`; a pre-existing gap (`priming/` was never added to this diagram by `FW-049`) caught and fixed in passing
- [x] `ANCHOR-PROJECT-DESIGN.md` — §8 rewritten in full (§8.1–§8.4 new, original design kept as §8.5/§8.5.1/§8.5.2 for the record)

Amended 2026-09-19 (same day) — re-verification safeguard added to extraction:

- [x] `.claude/commands/anchor-project.md` — Step 11b extended: extraction now re-reads the implicated file and confirms a pattern is still current before writing an observation, never promoting a journal entry's claim on trust alone
- [x] `ANCHOR-PROJECT-DESIGN.md` — §8.3 extended to match, Document History updated

Not yet done:

- [ ] Validation against a real project — the journal/extraction mechanism has not yet been exercised by an actual anchor session
- [ ] The reporting-gap placeholder (`[contact channel — TBD]`) remains unresolved, same as `FW-049` left it
