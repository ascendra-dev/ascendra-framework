# Release, Support & Hotfix

Covers Stage 11 (Sprint Closure & Release) and Stage 12 (Support) in [`SDLC.md`](../SDLC.md), plus the Hotfix Cycle: [`/close-sprint`](../.claude/commands/close-sprint.md), [`/gen-release-notes`](../.claude/commands/gen-release-notes.md), and the manual git steps that carry a production fix safely forward. Read [`00-orientation.md`](00-orientation.md) first if you haven't. This chapter assumes a sprint has already cleared UAT sign-off — how it got there is the QA/UAT chapter's territory, not this one. Defect severity levels (Critical/High/Medium/Low) and the deferral rules that govern them during UAT are also that chapter's job; this chapter only uses severity to decide what happens to a defect found *after* go-live, which is a different question.

---

## Purpose

Release, Support, and Hotfix look like they could be one "ship it and watch it" step, and the framework deliberately keeps them apart. Each does a job the other two can't:

- **Sprint Closure & Release (Stage 11)** is where the sprint's own learnings get checked against not-yet-built scope, and where a client-facing version boundary is decided — a decision that plainly does not always line up with a sprint boundary.
- **Support (Stage 12)** is a standing posture, not an event: it runs once per project, for a monitoring window, and has to end in one of two structurally different ways (closure or retainer) that change what the framework's own records say about the project going forward.
- **The Hotfix Cycle** is an emergency deviation from the normal pipeline — no sprint planning, no UAT — precisely because it exists for the moment when waiting for the normal pipeline is the wrong call. Collapsing it into ordinary release work would either slow down a Critical fix that needs to land in hours, or quietly skip the one git-safety step that keeps a fast, narrow fix from being undone by unrelated work already in flight.

Get the git logic in the third one wrong and the fix you deployed can vanish from `main` without anyone touching it on purpose — the failure mode that made this exact step worth rewriting in the framework's own files (see the Hotfix Cycle section below). That's the single most consequential thing in this chapter to get right.

---

## `/close-sprint`'s forward-looking learnings check

*(This is about `sprints/sprint-{NN}.md` content, not `releases/*.md` — the only artifact type that actually routes to this chapter. Not checkable when this chapter is reached via a release notes file.)*

`/close-sprint` does two structurally different things, and it's easy to read them as one. Step 6 fills in the retrospective — what went well, what to improve, carry-forward stories. That's a **backward-looking, process-only record**. Nothing in Step 6 touches a requirement, an epic, or a not-yet-generated story. Step 8 is the separate, forward-looking check, and it exists because nothing else in the pipeline re-examines earlier epics in light of what a later sprint just learned:

> "Before starting the next wave: did anything from this sprint's delivery, verification, or UAT reveal a gap, defect, or scope issue relevant to an epic or requirement that hasn't had stories written yet?"

**Why this check has to exist at all:** `/gen-stories` generates a wave's stories from the Locked architecture and the Approved BRD as they stand at that moment — it does not re-read every prior sprint's retrospective before it runs. If Sprint 1's UAT surfaces a real correction (a client clarifies a business rule mid-session that the BRD got wrong, or an edge case the domain model never modeled), and nobody acts on it, that correction sits in Sprint 1's retrospective forever. Sprint 2's stories get generated against the BRD and epics exactly as they were before the correction — the stale version, not the corrected one — because nothing forces a re-check. Step 8 is the one deliberate place in the pipeline where that silent staleness gets caught, and it hands you the fix mechanism directly: `/assess-change`. Step 8 itself does not correct anything — it only decides whether `/assess-change` needs to run before the next wave starts.

**What counts as a forward-looking learning worth flagging, vs. sprint-specific noise:**

| Signal | Forward-looking (flag it) | Sprint-specific noise (don't) |
|---|---|---|
| A client clarifies a business rule mid-UAT that touches a capability in a **later** epic (e.g. "actually, VAT rounding should always round up, not to nearest" — and a later epic's interest calculation on overdue invoices depends on the same rounding rule) | Yes — the not-yet-built epic will inherit the wrong assumption if nobody tells it | — |
| A defect found in **this** sprint's own stories, already fixed on the story branch before merge | — | No — already resolved inside the sprint that found it, nothing to propagate |
| A UI preference the client expressed about **this** sprint's screens (button copy, a column order) | — | No — sprint-local, handled through the normal PR review, not a scope issue |
| UAT reveals a domain concept the discovery process never modeled at all (a new actor type, a relationship pattern nobody described) that a future epic will need | Yes — this is the same "structurally new axis of variation" test from BRD discovery (Chapter 3): if it needs new domain modeling, not just a BRD edit, flag it here too, before the next wave builds on the gap | — |
| A carry-forward story moving to next sprint because it ran out of time, with no new information behind it | — | No — that's ordinary capacity management, already handled by the carry-forward table in Step 7 |

The test that separates the two columns: **does this new information change what a not-yet-built epic should have been told, or does it only affect work that's already been built or already has its own fix in hand?** If the former, it's a Step 8 item. If the latter, it belongs in the retrospective's "what to improve" note at most.

`/close-sprint` will not run `/assess-change` for you — Step 8 explicitly waits for the PO to either run it or say to proceed without it. Don't let "I'll deal with it later" become the answer by default; the whole point of asking before the next wave, not after, is that `/gen-stories` for that wave hasn't run yet, so the correction is still cheap.

---

## Release notes are versioned by release, not by sprint

The output path is `projects/{PROJECT_CODE}/releases/v{version}-release-notes.md` — keyed by version string, not by sprint number. The template says why directly: "since a release doesn't always map 1:1 to a sprint." That's not a filing detail; it's telling you there's a real decision to make that "one release note per sprint" would let you skip past without noticing.

**The decision you actually need to make, twice, every time a sprint closes:**

1. **Is this the moment a release goes out, or is this sprint's work going out bundled with a later one?** Nothing forces a production deployment at the end of every sprint. A sprint can close (`Sprint {NN}` status → `Complete`, per `/update-status`) without a release following it immediately, if the plan is to batch it with the next sprint's work into one client-facing deploy.
2. **If a release is going out now, what version number does it get** — and does that number reflect one sprint's work or several?

**Two concrete scenarios to ground this:**

- **A sprint ships as its own release.** Harborview's Sprint 01 closes, UAT passes, and the client wants Invoice Creation and Client Management live immediately — there's no reason to hold it back for Sprint 02's Payment Collection work. `/gen-release-notes projects/HARBORVIEW-INV-001 01 1.0.0` runs right after `/close-sprint`, one sprint's stories, one version, one deploy. This is the common case and the one the command's Step 2 gate check (`Sprint {NN}` must be `Complete`) is built around directly.
- **Two sprints batch into one release.** Suppose Sprint 01 (Client Management, Invoice Creation) and Sprint 02 (Payment Collection) both close, but the client has a hard policy of one production change window per month, and both sprints land inside the same window. The PO's call here is to treat the deploy — not the sprint boundary — as the thing that triggers `/gen-release-notes`, and to pick one version (`1.0.0`, covering both sprints) rather than generating `1.0.0` and `1.1.0` back to back for two deploys that never actually happened separately.

**The practical catch with batching:** `/gen-release-notes` as written reads exactly one sprint plan file (Step 3, Step 4 — "from the sprint plan document, extract the full list of story IDs in **this** sprint"). It has no built-in mode for pulling stories from two sprint plans into one document. If you're batching, the command alone won't do it for you: point it at the later sprint, then manually fold the earlier sprint's merged stories into Section 1 (What's New) and Section 6 (Change Summary) yourself — or ask the AI to read both sprint files in the same session before drafting. Don't assume running the command once against the later sprint number silently picks up the earlier one; it doesn't.

**What decides the version number itself, in practice:** this framework doesn't prescribe a versioning scheme, so treat it as an ordinary semver call tied to what actually happened in that deploy — a first production release is `1.0.0`; a deploy that only adds capability (no breaking change, nothing removed) bumps the minor number; a hotfix-only deploy bumps the patch number (see below). The number belongs to the deploy event, not to whichever sprint number happens to be freshest.

---

## The Hotfix Cycle, explained clearly

*(Git process and branch mechanics — not content that lives in a release notes document to check.)*

This is the part of the chapter worth reading twice. `PROJECT-LIFECYCLE.md`'s Hotfix Cycle step 8 was rewritten specifically because the old wording described a git object — "the current sprint's working branch" — that doesn't exist anywhere in this framework's actual git model. The model, per `git-standards.md` and `implement-story.md`, is trunk-based: one branch per story, cut from `main`, merged directly back into `main`. There is no persistent sprint-integration branch sitting between a story branch and `main` at any point. Once you hold that fact fixed, the corrected hotfix logic is genuinely simple — but only if you walk it through step by step, because the risk it's protecting against is easy to state wrong.

### The walkthrough

1. **A Critical or High defect is confirmed in production.** The PO classifies it and confirms a hotfix should proceed.
2. **A hotfix branch is cut from `main`** — `hotfix/{PROJECT_CODE}-{issue-description}` — the same starting point every story branch uses, not a special branch.
3. **The fix is implemented and verified** on that branch, scoped strictly to the defect's acceptance criteria. `/verify-story` confirms the defect scenario is fixed and nothing in the affected module regressed.
4. **A PR is raised against `main`.** The PO reviews and approves it — the same review gate as any other PR.
5. **The PR merges into `main`.** This is the moment the fix becomes part of the trunk.
6. **The PO approves production deployment** (rollback plan in hand) and it goes live; smoke tests confirm it's healthy.
7. **Here's the part that used to be described wrong.** Because every story branch in this framework is cut from `main` and merges back into `main` — there is no separate branch to "merge the hotfix into" — the hotfix landing on `main` in step 5 already means every story branch cut from `main` *after* that point automatically contains the fix. You don't have to do anything for those branches; they inherited it the moment they were created.

### The one thing that actually needs a manual check

The exposure is narrower than "did the fix propagate everywhere" — it's specifically about timing. Picture it as two branches racing against the same file:

- A story branch for the **current sprint** was cut from `main` **before** the hotfix PR merged. That branch's copy of `main` is frozen at the moment it was created — it has no knowledge the hotfix ever happened.
- If that story branch happens to touch the same lines the hotfix touched (entirely by coincidence — two people fixing unrelated things near the same code), and its PR merges into `main` **without first pulling the hotfix in**, git has no way to know the hotfix's version of those lines should win. From git's point of view, the story branch's edit is simply newer than what it started from, and its version of the lines is what lands — silently overwriting the hotfix the moment that story's PR merges.

**The check, stated as something you can actually do:** for every story branch currently open for the sprint in progress, ask whether it was created before or after the hotfix PR merged into `main`. If it was cut *before*, merge or rebase `main` into that branch before that story's own PR is allowed to merge — resolve any conflicts at that point, not later. If it was cut *after* the hotfix merged, it already started from a `main` that includes the fix, and there's nothing to do.

A simple rule of thumb that avoids checking every branch individually: **the instant the hotfix PR merges, treat "pull `main`" as a required step before merging any story branch that was already open at that moment** — don't wait to discover which ones happen to overlap the hotfix's files. It costs one `git merge main` (or rebase) per open branch and removes the guesswork about which files actually collided.

### A worked timeline

Imagine Harborview is mid-Sprint 02 when a Critical defect surfaces in production: an invoice's VAT total is calculated wrong for a specific rounding edge case. Two story branches for Sprint 02 are already open — `story/US-01-011` (cut from `main` on Monday) and `story/US-01-014` (not yet started). The hotfix branch is cut from `main` Tuesday morning, fixed, reviewed, and merged into `main` Tuesday afternoon.

- `story/US-01-014` gets cut Wednesday, after the merge — it already includes the VAT fix. Nothing to do.
- `story/US-01-011` was cut Monday, before the merge. Before its PR is allowed to merge, `main` (now carrying the VAT fix) needs to be merged or rebased into it — even if `US-01-011`'s own work has nothing to do with VAT calculation, skipping this step means whatever `US-01-011`'s branch currently has for that VAT logic (the pre-fix version, frozen from Monday) is what ends up on `main` once its PR lands, silently undoing Tuesday's hotfix.

That's the entire mechanism. It isn't about the hotfix "not reaching" the rest of the codebase — `main` already has it the moment the PR merges. It's about one specific class of branch (already open, forked before the fix landed) being able to overwrite it on its own way in.

---

## What triggers a hotfix vs. what gets batched

*(A triage decision, tracked in the Support Log — not release notes content to check.)*

[`reference/defect-severity.md`](../reference/defect-severity.md)'s SLA table gives you the urgency numbers directly — worth having in view here even though the deferral mechanics themselves belong to the QA/UAT chapter:

| Severity | SLA | In production |
|---|---|---|
| Critical | Fix within 4 hours; hotfix deployed same day | Hotfix Cycle triggered immediately, no exceptions |
| High | Fix within 1 business day | Hotfix Cycle triggered immediately |
| Medium | Fix within the current sprint | Batched into a future sprint |
| Low | Fix in the next sprint or backlog | Batched into a future sprint |

One rule from `defect-severity.md` matters specifically for Stage 12: **a defect found in production is automatically escalated one severity level** (Medium → High, High → Critical). A bug that would have been Medium if caught in UAT can be High once it's live — check the escalated severity, not the severity you'd have assigned it pre-release, before deciding whether it's a hotfix.

**What "batched" actually means, concretely — where it goes so it doesn't disappear:** a Medium/Low production defect isn't quietly dropped just because it isn't urgent enough for a hotfix. Log it using the Defect Log Format (`defect-severity.md`'s ID/Title/Severity/Steps/Status fields) as part of the Support Log — the artifact Stage 12 produces continuously through the monitoring period. Every entry needs a target sprint recorded against it, same as the Deferred Defect Process requires during UAT. When that target sprint is planned (`/gen-sprint-plan`), the PO is responsible for pulling the logged defect into that sprint's scope explicitly — a production defect sitting in the support log doesn't automatically show up in a future sprint's story list the way a UAT carry-forward story does, because it was never a story in the first place. If it needs real implementation work, it typically becomes a story through `/assess-change` before that sprint is planned, the same path Step 8 above routes forward-looking learnings through.

---

## Support's two end states

*(About `brief.md`'s Status field, a different artifact — not checkable from a release notes document.)*

Stage 12 is entered once per project — not once per sprint — after the final sprint's production deployment, and it runs for a monitoring window (default 30 days, overridable by BRD Section 10 or explicit client agreement). It ends one of two ways, and the practical trigger for choosing between them is simple:

**Is there a firm end date and scope, with no further paid engagement after the monitoring window — or is the relationship ongoing?**

| | Closure | Retainer |
|---|---|---|
| **Trigger** | The contracted scope is delivered, the monitoring window elapses cleanly (zero open Critical/High defects), and there's no agreement for further work | The client wants ongoing development or support beyond what was contracted, and a new agreement is in place |
| **What changes in the framework's own records** | Brief `Status` field is set to `Complete` via `update-status` — the project's overall lifecycle state, not the brief's Content Status field (see [`00-orientation.md`](00-orientation.md)'s Content Status vs. Status distinction) | A new brief, or a formal addendum to the existing one, is created — the project directory stays `Active`, not `Complete` |
| **What gets produced** | A formal closure document — what was delivered, any open known issues, handover notes | No closure document; work continues under the new/extended brief, and the framework's normal stage cycle (Discovery → BRD → ... → Sprint Delivery) applies to whatever the retainer covers |

The decision isn't really a framework mechanic — it's a business-relationship fact you already know by the end of the monitoring window. What the framework asks you to do with that fact is make it explicit in the project's own records rather than letting the project drift in an ambiguous `Active` state with no monitoring actually happening. A project that's functionally done but never gets its brief status set to `Complete` looks, to anyone reading `projects/index.md` later, like a project still in delivery.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s terminology map — in particular the Content Status vs. Status distinction, used directly above in Support's Closure path. New terms specific to this phase:

- **Forward-looking learning (`/close-sprint` Step 8)** — a piece of information from this sprint's delivery, verification, or UAT that would change what a not-yet-built epic should have been told. Distinct from the retrospective itself (Step 6), which is a backward-looking process record and corrects nothing on its own.
- **Release version vs. sprint number** — a release note is keyed by `v{version}`, decided at the moment a production deploy actually happens; a sprint closing (`Sprint {NN}` → `Complete`) doesn't by itself trigger one.
- **Hotfix branch** — cut from `main` like any story branch, merged directly back into `main`. There is no persistent sprint-integration branch in this framework's git model for it to merge into instead.
- **Support Log** — the running defect log Stage 12 produces for the monitoring period, using `defect-severity.md`'s Defect Log Format; where a batched (Medium/Low, or deferred High) production defect is tracked so it doesn't disappear before its target sprint.
- **Closure vs. Retainer** — the two end states of Stage 12, distinguished by whether paid engagement continues past the monitoring window, not by any technical condition.

---

## Common mistakes

- Treating `/close-sprint`'s retrospective (Step 6) as if it already corrects stale scope — it doesn't; Step 8 and `/assess-change` are the actual correction path.
- Letting a real forward-looking learning slide because "we'll catch it when we get to that epic" — nothing re-checks earlier epics against later learnings except Step 8, and only if you answer it honestly.
- Flagging ordinary sprint-specific feedback (a UI tweak, a defect already fixed on its own branch) as a Step 8 item — it just adds noise to a check meant for genuine scope corrections.
- Assuming one release note per sprint is the default — check whether this sprint's work is actually going out on its own or batching with a later one before running `/gen-release-notes`.
- Running `/gen-release-notes` once against a later sprint and assuming it picked up an earlier batched sprint's stories too — it only reads the one sprint plan file you point it at.
- Reading the old "merge the hotfix into the sprint branch" framing anywhere and looking for a sprint-integration branch that doesn't exist in this framework's git model — the corrected logic is: the hotfix lands on `main` directly, and the only manual step is pulling `main` into any story branch that predates the merge.
- Assuming a hotfix "reaching" the rest of the codebase is the risk — it already has, the moment its PR merges into `main`. The actual risk is an *older, already-open* branch overwriting it on its own way in.
- Not escalating a defect's severity when it's found in production — `defect-severity.md`'s automatic one-level escalation (Medium → High, High → Critical) can be the difference between "batch it" and "hotfix now."
- Logging a batched production defect nowhere durable — a Support Log entry with a target sprint is what keeps it from silently vanishing before that sprint is planned.
- Leaving a project's brief `Status` at `Active` after the support window closes cleanly, instead of explicitly setting it to `Complete` — the framework's own records should reflect that the project is actually done.

---

## Harborview in practice

[`release-notes.example.md`](../projects/TEMPLATE/releases/release-notes.example.md) is Harborview's real Sprint 01 release, v1.0 — read it for what a genuinely clean, single-sprint release looks like end to end:

- **Section 1 (What's New)** groups by feature area (Client Management, Invoice Creation) exactly as the template requires — no story IDs, no table names, no API routes anywhere in the client-facing sections, the discipline this chapter's release-notes guidance is built around.
- **Section 4 (Known Limitations)** names what didn't ship — dunning reminders, payment collection — as real limitations with a planned epic and, implicitly, a future sprint, rather than silently omitting them. This is the honest version of what a batched-release Known Limitations section needs to do too: say plainly what's not in yet, not just what is.
- **Section 5 (Deployment Notes)** shows a real, populated environment variable change (`INVOICE_NUMBER_PREFIX`) with an explicit required action — the level of specificity to hold every Deployment Notes section to, hotfix or ordinary release alike.
- **Section 6 (Change Summary)** honestly records "Epics completed: 0 (EPIC-002 and EPIC-003 partially delivered)" rather than rounding up — a small but real instance of the same "don't invent, don't round up" discipline that matters even more once a release starts batching multiple sprints' worth of partial epic delivery together.

Harborview's own worked examples stop at Sprint 01, so there's no real Hotfix Cycle or Stage 12 closure example to point to in the repo — the walkthrough and worked timeline above are this chapter's own construction, built to the same rigor, for exactly the gap Harborview's single-sprint example can't fill.
