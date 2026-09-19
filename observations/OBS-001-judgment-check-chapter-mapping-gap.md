# OBS-001 — `/judgment-check`'s chapter mapping can't reach `/close-sprint`'s own content

| Field | Value |
|-------|-------|
| **ID** | OBS-001 |
| **Date** | 2026-09-19 |
| **Status** | New |
| **Area** | `.claude/commands/judgment-check.md` (Step 1 mapping table), `practitioner-guide/07-sprint-planning.md`, `practitioner-guide/10-release-support-hotfix.md` |
| **Found while** | Direct practitioner-guide review (not extraction — no real anchor session has run yet; found while auditing whether the guide serves `/judgment-check` well) |

> Status values: `New` (default) · `Under Review` · `Promoted → FW-XXX` · `Rejected` · `Won't Fix`

---

## What was observed

`/judgment-check`'s Step 1 mapping table routes `sprints/sprint-*.md` exclusively to `practitioner-guide/07-sprint-planning.md`. But `practitioner-guide/10-release-support-hotfix.md` is where `/close-sprint`'s own behavior is actually taught — its Step 6 retrospective and, more substantively, its Step 8 "forward-looking learnings" check (a genuine, checkable test distinguishing a correction that changes what a not-yet-built epic needs to know from ordinary sprint-specific noise). Since `/close-sprint` writes into the same `sprints/sprint-{NN}.md` file chapter 07 owns, chapter 10's content is never reached when checking that file — and chapter 10 is only reached at all via `releases/*.md`, where most of its own content (the close-sprint check, the Hotfix Cycle's git mechanics, Support's closure/retainer decision) isn't about release-notes content either.

## Why it matters

A real, well-built checkable test (chapter 10's forward-looking-learning table) is structurally unreachable by `/judgment-check` under the current one-artifact-type-to-one-chapter mapping, no matter which artifact it's run against. This weakens exactly the class of check the framework cares most about catching — a correction from one sprint silently failing to propagate to a not-yet-built epic — for every project that relies on `/judgment-check` rather than manually re-reading chapter 10 at every sprint close.

## Proposed direction

Not clear which fix is best — needs the framework author's judgment. Candidates: (a) route `sprints/sprint-*.md` to both chapters 07 and 10 (breaks judgment-check's "read exactly one chapter" economy); (b) move the close-sprint-specific content from chapter 10 into chapter 07, since that's the chapter actually mapped to the artifact it modifies; (c) accept the gap for `/judgment-check` specifically and rely on the chapter being read directly by a PO at sprint close instead.
