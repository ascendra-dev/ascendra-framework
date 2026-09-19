# Cross-Cutting Commands

Covers [`/assess-change`](../.claude/commands/assess-change.md), [`/update-status`](../.claude/commands/update-status.md), [`/project-status`](../.claude/commands/project-status.md), and [`/judgment-check`](../.claude/commands/judgment-check.md) — the four commands [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md) names explicitly as belonging to no single phase, because they "operate across all three regions [Problem Domain, Bridge, Solution Domain] and belong to none of them." Read [`00-orientation.md`](00-orientation.md) first if you haven't. `/init-project` is also technically cross-cutting in the sense that it isn't a `/gen-X`/`/review-X` pair, but [`01-intake.md`](01-intake.md) already covers it in full — this chapter's job is the four commands that actually have no stage home at all.

---

## Purpose

Every other chapter in this guide opens the same way: "covers Stage N." This one can't, and that's exactly why these three commands are the easiest ones in the framework to forget exist. A command tied to a stage gets run because you're *in* that stage — you run `/gen-architecture` because you just finished Screen Design and architecture is obviously next. Nothing puts `/assess-change`, `/update-status`, or `/project-status` in front of you that way. You have to remember to reach for them yourself, usually at the exact moment — mid-sprint, client on the phone asking for something new, a story that's clearly stuck — when reaching for the right tool matters most and stopping to think "which command handles this" is the last thing you want to do.

All four earn their place for the same reason: they're the ones that keep the other twenty-nine commands honest once real change, real drift, real time pressure, or quiet quality drift enter the picture. `/assess-change` is what happens when the world doesn't respect the document pipeline you locked in good faith. `/update-status` is the mechanism — largely manual, more often than PO's expect — that keeps every gate in `PROJECT-LIFECYCLE.md` actually true of the files on disk. `/project-status` is the one place that reads the whole project back to you honestly, blockers included. `/judgment-check` is the one that catches an artifact quietly getting vaguer or denser than this guide's own tests would allow, before that artifact locks in through its own gate. This chapter's job is to make each of the four a reflex instead of an afterthought.

---

## `/assess-change`: classifying a change correctly

### Pattern 2 vs. Pattern 3 — the actual test

`/assess-change` sorts every change into one of five patterns (1 Fix, 2 Gap, 3 Scope Shift, 4 Requirement Change, 4a Parking Lot Entry), and the command's own text is genuinely well built for four of the five — concrete trigger examples, a formality rule, and exact Document Control snippets for each. The one pair with no built-in test is the pair a PO hits constantly: **Pattern 2 (Gap)** and **Pattern 3 (Scope Shift)** are both described, almost identically, as "new content lands in an epic, and the BRD doesn't change." The command's own examples for each read like paraphrases of each other — "missing story for an In Scope bullet that already exists" (Pattern 2) versus "new In Scope item added to an epic" (Pattern 3) — with nothing telling you which bucket a given request actually falls into.

Under deadline pressure, the natural drift is toward Pattern 2 every time, because it's cheaper: no version bump, no Document Control table entry beyond a gap-fill row, no formal confirmation gate. That drift is exactly what erodes the discipline the whole pattern system exists to enforce — a Pattern 3 change waved through as Pattern 2 means a real scope addition entered the epic with none of the paper trail a scope addition is supposed to leave.

**The test, mirrored on the same logic [`FW-024`](../decisions/FW-024-parking-lot-deferred-capabilities.md)'s Parking Lot test already taught you in [Chapter 3](03-brd-discovery-and-scope-lock.md#section-521-future-capabilities-vs-section-15-parking-lot-the-actual-test):**

> Can you point to the specific sentence in an approved BRD requirement that already implies this behavior? If yes — Pattern 2. If you're inferring intent, filling a gap you believe was "obviously meant," rather than citing text that's actually there — Pattern 3.

Grounded in a real Harborview epic ([`epic.example.md`](../projects/TEMPLATE/epics/epic.example.md), EPIC-001 Payer Management): REQ-004 reads, in full, "The system must allow the Finance Manager to import Payers from a CSV file. The import must run deduplication checks and produce a per-row import log showing success or failure. Duplicate rows are flagged and not imported — manual resolution required." Suppose, mid-sprint, nobody wrote a story covering what the Finance Manager actually sees when a CSV row is flagged as a duplicate during import. That's Pattern 2 — the behavior ("duplicate rows are flagged and not imported — manual resolution required") is already sitting in REQ-004's own sentence; the story generation pass simply missed it. Now suppose instead the request is "let the Finance Manager select several flagged rows at once and resolve them all in one bulk action." Nothing in REQ-004, REQ-002, or REQ-003 says anything about bulk resolution — that's a genuinely new piece of scope you'd be inferring makes sense, not one you can point to in the requirement text. That's Pattern 3.

**What each classification actually costs you, mechanically:**

| | Pattern 2 (Gap) | Pattern 3 (Scope Shift) |
|---|---|---|
| BRD | Untouched | Untouched |
| Epic | No version bump. Document Control gets a gap-fill row only if a new story was added (`— \| [date] \| Ascendra AI \| Gap fill: added [story ID] — no scope change`) | Version bump. Document Control row stating what changed and why |
| Story | Updated in place per the normal story-state rules | Created/deprecated/updated per the normal story-state rules |
| Confirmation gate | None — applied directly | Step 5's formal gate: an explicit "confirm the scope adjustment is intentional and agreed" before anything is touched |
| Epic `Status` field | Unchanged | Unchanged — Pattern 3 does **not** revert the epic to `Draft` or reopen `/review-epics`; "formal" here means a version bump and an explicit PO confirmation, not an un-approval |

That last row is worth being precise about, because "formal" is easy to over-read as "the epic goes back through review." It doesn't. Formal means the change leaves a version-bumped, dated, reasoned trail in the Document Control table and was applied only after you explicitly said yes to the blast radius — not that the gate reopens.

### The Promotion Gate surprise, and the habit that avoids it

If the change you describe to `/assess-change` turns out to already be sitting in the BRD's Section 15 (Parking Lot), the command stops mid-session — after you've already gone through Step 1's context read and Step 2's description — to tell you it can't author the requirement text itself and redirect you to a discovery pass first (`/run-domain-discovery` and/or `/run-brd-discovery`; see `FW-024`, and [Chapter 3](03-brd-discovery-and-scope-lock.md) for the Section 5.21 vs. Parking Lot test that decides whether something lands there in the first place). That's the right behavior — a Parking Lot entry has no domain modeling behind it, and `/assess-change` shouldn't be inventing requirement text on the spot — but it's a genuinely wasted read-and-classify cycle if it surprises you partway through a session.

**The one-line habit that avoids it:** before you describe a change to `/assess-change`, glance at the project BRD's Section 15 yourself. If the idea is already sitting there — Harborview's worked instance is PL-001 in [`brd.example.md`](../projects/TEMPLATE/brds/brd.example.md), the admin-assistant-submits-invoices idea, parked with no REQ-ID and an explicit Promotion Path note — say so up front: "this is PL-001, and I want to know whether it's ready to promote, or whether it still needs discovery." That turns a mid-session redirect into a five-second check before you start, and it means the session opens already knowing which of the two paths (formal Pattern 4 promotion, or a discovery pass first) it's actually going to take.

---

## `/update-status`: when do you actually need to run this yourself?

This is the most practically useful thing in this chapter, so treat it as the reference to bookmark. `PROJECT-LIFECYCLE.md`'s own Gate Summary table and Status Values Reference are accurate, but the inline-vs-manual distinction is scattered across both, phrased slightly differently for each artifact type — nothing puts the two columns side by side in one place. Here they are.

### Set inline — no `update-status` call needed in the normal path

These transitions happen automatically, as part of running the corresponding `/review-X` command (or, for the brief, `/run-intake` itself). You'll never need to run `/update-status` for these unless you're deliberately working outside the normal flow.

| Artifact | Transition | Set by |
|---|---|---|
| Brief (Content Status) | `Draft` → `Brief complete` | `/run-intake` Step 8, the moment all 9 sections are filled and Part 2 Verification passes |
| BRD | `Draft`/`Under Review` → `Approved` | `/review-brd`, when the PO types `'approved'` |
| Epic | `Draft` → `Approved` | `/review-epics`, during the walkthrough |
| Screen Design | `Draft`/`Under Review` → `Approved` | `/review-screen-design`, when the PO types `'approve'` |
| Architecture | `Draft`/`Under Review` → `Locked` | `/review-architecture`, on the lock confirmation |
| Story | `Draft` → `Reviewed` | `/review-stories`, during the per-story/per-epic walkthrough |

For every row above, `/update-status` still exists as a manual fallback if the approval genuinely needs to happen outside a review session (e.g. resuming a fragmented session) — but it is not the primary path, and reaching for it here is redundant most of the time.

### Always manual — no inline path exists, `update-status` is the only way

None of these transitions is ever set as a side effect of any other command. If you want one of these to happen, you have to run `/update-status` yourself — nothing else in the framework will do it for you, including `/implement-story` and `/verify-story`, both of which explicitly leave story status changes to the PO.

| Artifact | Transition | Notes |
|---|---|---|
| Brief (Content Status) | `Brief complete` → `Approved` | The actual gate `/gen-domain-knowledge` checks — see [Chapter 1](01-intake.md) |
| Brief (Content Status) | `Approved` → `Locked` / `Superseded` | No command defines a trigger for these; they're yours to use for your own process discipline |
| Brief (Status — project lifecycle) | `Active` / `On Hold` / `Complete` / `Cancelled` | Every transition on this field, always — it's never touched by any generating or reviewing command |
| BRD | `Approved` → `Locked` | Formal freeze; `/review-brd` only ever sets `Approved`, never `Locked` |
| Epic | `Approved` → `In Progress` | Warns if a dependency epic isn't `Done` yet |
| Epic | `In Progress` → `Done` | Blocked unless every story in the epic is `Done` or `Deprecated` |
| Epic | any → `Deprecated` | — |
| Story Plan ([`FW-031`](../decisions/FW-031-story-planning-phase-separation.md)) | `Draft` → `Confirmed` | Hard gate, no override — `/implement-story` refuses to run without it |
| Story | `Reviewed` → `In Progress` | Warns if a dependency story isn't `Merged`/`Done` yet |
| Story | `In Progress` → `PR Created` → `Changes Requested` → `Merged` → `Done` | Every one of these steps, including `Merged` — raising the PR is a manual step, and merging it doesn't update the story file on its own |
| Story | any → `Deprecated` | — |
| Sprint | `Planning` → `Active` | `update-status projects/{CODE} sprint {NN} Active` |
| Sprint | `Active` → `Complete` | `update-status projects/{CODE} sprint {NN} Complete` — warns and asks you to confirm anyway if any story in the sprint isn't `Done`/`Deprecated` |

The pattern underneath both tables, if you want the one-sentence version: **a review command sets the status it exists to grant** (`/review-brd` approves the BRD; it doesn't also march the BRD to `Locked`, and it definitely doesn't touch epic or story status). **Everything that isn't the specific thing a review command was built to grant is on you.** The four cases named most often as surprises — Epic → `In Progress`/`Done`, Story → `Merged`/`Deprecated`, Sprint → `Active`/`Complete`, and Brief → `Approved` after `/run-intake` finishes — are all in the second table for exactly that reason: nothing else in the pipeline advances them, ever.

---

## `/project-status`'s ATTENTION section: what to actually do with each flag

`/project-status` renders a dashboard from the epics and stories indexes and closes with an ATTENTION section listing whatever needs a human look. The command's own closing line — "run `/update-status` to update any stale statuses, or `/assess-change` to handle any scope changes" — reads as if it covers every possible flag. It doesn't. Two of the five example triggers the command itself lists don't map to either command at all, and pointing a PO at the wrong tool for those wastes exactly the time this dashboard exists to save. Here's a real action for each of the five, honestly, including the two where the honest answer is "there isn't a command for this."

| ATTENTION trigger | What it actually means | Real action |
|---|---|---|
| L-sized stories in the active sprint not yet split | A story review flagged this as complex and it slipped through without being split | Check the Flags Register entry for why. If it's genuinely still unsplit and not yet started, split it now — this is a story-authoring fix, not a scope change, so it doesn't need `/assess-change` unless the split also changes what's In/Out of Scope |
| Stories marked "Changes Requested" | The PO has open review feedback on a submitted PR | This is a human action, not a command action — go review the PR. Neither `/update-status` nor `/assess-change` moves this forward; only your own PR review does |
| Active sprint > 50% of its duration with < 30% stories closed (slip risk) | The sprint is genuinely behind pace | **Be honest with yourself here: the framework has no built-in remedy command for this.** It's a PO judgment call with three real options — add capacity, descope with the client (which, if it actually removes scope, goes through `/assess-change` as a real change), or extend the sprint. There's no fourth mechanism hiding in the framework; don't go looking for one |
| Epic `In Progress` with no stories `In Progress` (stale) | Ambiguous by design — could be a stale status or a real pause | Tell the two apart before acting: check whether any story in that epic changed status in the last N days of real activity (a few days of nothing on a fast-moving project, longer on a slow one), or just ask directly whether the work is genuinely paused. If every story in the epic actually finished and nobody advanced the epic itself, that's a data-entry fix — run `update-status ... Done`. If it's genuinely paused (blocked dependency, resourcing pulled), that's not a status problem at all — it's a real blocker to escalate, and `/update-status` won't fix it because there's nothing wrong with the data |
| Dependency: a story/epic `In Progress` whose dependency isn't `Done` | This exact case was already flagged once, at the moment someone set the dependent item to `In Progress` — `update-status` warns on this transition and requires confirmation | Ask whether this was a deliberate, accepted override at the time (nothing to do — it was a known, accepted risk) or whether the dependency regressed afterward (e.g. it was later deprecated or reopened). Only the second case needs action, and the action is re-sequencing the work, not a status edit |

---

## `/judgment-check`: the two-invocation habit that's easy to skip

`/judgment-check {artifact-path}` runs a density/judgment-quality pass against one already-generated artifact, sourced from that artifact type's own chapter of this guide — not from the artifact's template, and not a repeat of its `/review-X` walkthrough. It checks something neither of those does: whether the artifact's density, boundaries, and judgment calls actually hold up against the tests this guide derives from reading every command and template closely, not just whether the document is structurally complete or PO-confirmed correct.

**The one thing easy to miss: a single invocation never both finds and resolves.** The command runs as two mutually exclusive modes — Analysis Mode surfaces findings and logs them `Open` in the artifact's Density & Judgment Log; Resolve Mode walks you through open findings and closes them out — and never both in the same run. Run it once against a fresh artifact and you get "N new findings logged as Open," with no interactive follow-up in that same invocation. Stop there and move straight to `/review-{artifact-type}`, and those findings sit `Open` forever — logged, but never actually weighed by anyone. Closing the loop on one artifact always takes at least two invocations: once to surface, once (or more, if a finding drags) to resolve.

**When anchor is driving, both passes happen automatically, back to back** — anchor invokes `/judgment-check` before every `/review-X` gate it drives, and if the first pass logs anything, invokes it again immediately for the interactive resolve pass. Running commands standalone, you're the one who has to remember the second invocation — nothing else in the framework runs it for you.

**It checks one artifact, not the project.** It deliberately reads only the target artifact, its one matching chapter, and `00-orientation.md` — never another artifact in the same project, even when a chapter's own worked example cross-references one. A finding here is never "this epic contradicts an assumption in the BRD" — that's a cross-artifact concern, and it's what anchor's own drift-checking (`decisions/FW-049-anchor-project-and-priming-session.md`) exists for, not this command.

**A finding is a question, never a verdict — and "no findings" isn't a clean bill of health either.** The command is explicit that a model checking its own or a sibling model's output can't certify correctness the way independent human judgment can. Zero findings means nothing checkable against this specific artifact's content stood out against the chapter's tests — it doesn't mean the chapter's entire test suite passed, and it's not something to chase down to zero the way you'd chase a build error.

---

## The Gate Summary table is now a complete reference

[`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md)'s [Gate Summary](../PROJECT-LIFECYCLE.md#gate-summary) table now includes every hard gate in the pipeline, including the Story Plan Confirmed gate (`7a`, per [`FW-031`](../decisions/FW-031-story-planning-phase-separation.md)) that was missing before this guide's audit pass. That's not a big change in itself, but it means you can now treat the table as genuinely complete for the one question it exists to answer — "what status do I need in place before I can run X" — without checking somewhere else for a gate that might be missing from it.

---

## Terminology recap

This chapter leans on two things [`00-orientation.md`](00-orientation.md) already defines in full — go there for the base explanation, not here:

- **Content Status vs. Status** — the brief's two-field split. Every other artifact in the tables above (BRD, Epic, Screen Design, Architecture, Story) uses a single `Status` field for both readiness and gate state; the split is unique to the brief.
- **Section 5.21 vs. Parking Lot** — the test that decides whether a not-yet-built idea has a REQ-ID at all. This chapter's Pattern 2/3 test is a different judgment call, made after something is already in the BRD — don't conflate the two.
- **Analysis Mode vs. Resolve Mode** — `/judgment-check`'s own two mutually exclusive modes (surface vs. resolve). `Open` / `Confirmed as-is` / `Revised` / `Acknowledged tradeoff` are the only Outcome values a finding can carry once resolved.

---

## Common mistakes

- Defaulting to Pattern 2 because it's cheaper, without being able to point to the actual BRD sentence that justifies it.
- Describing a change to `/assess-change` without first checking whether it's already a Parking Lot entry — burning a full read-and-classify cycle before the Promotion Gate redirects you.
- Assuming a `/review-X` command advances every status field on an artifact, when it only ever sets the one status it exists to grant.
- Treating a Story `Merged` in GitHub as a Story `Merged` in the framework — the PR merge and the story-file status update are two separate actions, and only you do the second one.
- Pointing every ATTENTION flag at the same "run `/update-status` or `/assess-change`" line, instead of checking which of the two — if either — actually applies.
- Inventing a remedy command for sprint slip risk because the dashboard's generic follow-up sentence implies one exists. It doesn't; that one is a judgment call, every time.
- Reading a stale "Epic In Progress, no stories In Progress" flag as always meaning either "fix the data" or always meaning "real blocker" — it's genuinely either, and the dashboard can't tell you which.
- Running `/judgment-check` once, seeing "N new findings logged as Open," and moving straight to `/review-X` without a second invocation to actually resolve them — nothing blocks this, so nothing stops you, but the findings just sit there unweighed.
- Treating `/judgment-check`'s "no findings" as a quality certification rather than "nothing checkable against this content stood out" — it's not a pass/fail gate and was never meant to be chased to zero.
- Expecting `/judgment-check` to catch a cross-artifact inconsistency — it only ever reads the one artifact in front of it; that's anchor's drift-checking's job, not this command's.

---

## How these four commands work together

None of the four is useful alone for very long. `/project-status` is where a real Tuesday starts — you run it, see the ATTENTION section, and work down the list. Most of what it surfaces resolves into either a status that's gone stale (`/update-status` fixes it directly) or a change that's actually entered scope (`/assess-change` classifies and applies it). The loop closes back through `/project-status` on your next check-in, which is also the only place in the framework that will tell you, without being asked, whether last week's `/assess-change` session actually left every index consistent.

`/judgment-check` sits at a different point in the same overall discipline — not a Tuesday habit like the other three, but a pre-gate one. It runs against whatever artifact is about to go through `/review-X`, gets resolved (the second invocation), and only then does the review walkthrough proceed. It never overlaps with the other three's territory — it doesn't touch a status field, doesn't classify a change, doesn't read the whole project — it's the one pass that asks whether what you're about to lock in actually holds up against this guide's own tests before you lock it in.

Harborview's own worked material only exercises one corner of this directly — PL-001 in `brd.example.md`'s Section 15, sitting there with no REQ-ID and a Promotion Path note, exactly the kind of entry that would trigger the Promotion Gate surprise this chapter opened with if someone described it to `/assess-change` cold. The rest of this chapter's judgment calls — Pattern 2 vs. 3, the manual-status table, the ATTENTION categories, the judgment-check habits above — are the kind of thing that shows up only once a project is actually running, past the point where a single fixed example can capture every case. That's true of this chapter's subject matter generally: these four commands exist precisely for the parts of a project that don't fit a template.

That's also as good a place as any to close this guide's first full pass. Twelve SDLC stages, and every one of them now has a chapter walking through the same question: what do you actually write, and what breaks downstream if you get it wrong. The pass being complete doesn't mean every judgment call in the framework has been reduced to a test — some genuinely haven't, and this guide says so wherever that's true. [`case-studies/harder-cases.md`](case-studies/harder-cases.md) keeps building out the harder scenarios Harborview doesn't cover; the phase chapters themselves will keep getting corrected as real projects run against them.
