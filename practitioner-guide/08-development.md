# Development

Covers Stage 8 in [`SDLC.md`](../SDLC.md): [`/gen-story-plan`](../.claude/commands/gen-story-plan.md), [`/implement-story`](../.claude/commands/implement-story.md), [`/verify-story`](../.claude/commands/verify-story.md), and [`/gen-pr-description`](../.claude/commands/gen-pr-description.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't. This chapter assumes stories are already `Reviewed` and sprint-assigned — how they got that way is the Sprint Planning chapter's territory, not this one.

---

## Purpose

Four commands run in sequence, once per story, for the life of the project: plan, implement, verify, describe. It's tempting to read that as three commands' worth of real work plus one bookkeeping step bolted on the front — it isn't. `/gen-story-plan` exists because `/implement-story` used to go straight from gate-checks to writing code, with nothing the PO could review before code existed — the one phase in the framework that didn't honor "every document produced by AI is reviewed before the next step starts." [`FW-031`](../decisions/FW-031-story-planning-phase-separation.md) split that single command's two jobs apart: **plan** states *what* gets built — which endpoints, tables, screens, or jobs, in what order, tested how — by extracting it from the already-Locked architecture and the story's own acceptance criteria; **implement** states *how* to build it, in this project's confirmed stack idiom. A plan never invents architecture — an artifact the architecture doesn't already define is a gap flagged in the plan's own Risks section, not a decision made on the spot.

The gate this buys you is real and unusually strict for this framework: `/implement-story` hard-gates on the plan being `Confirmed`, with **no PO override** — unlike the softer dependency-cleared and Open-Decision gates in the same command, which do allow one. FW-031's own rationale for that strictness is worth holding onto through this whole chapter: *"The PO explicitly requested visibility into what a story would produce before implementation started — pages, endpoints, queries, DTOs, guards."* A plan is cheap enough to produce and review that there's no legitimate "not worth it for this story" case. What follows is what to actually do with that visibility once you have it, because the framework hands you the plan and largely leaves the judgment calls around it to you.

---

## The story-plan confirmation checklist

`/gen-story-plan` Step 9 presents a one-line summary table and asks: *"Does this plan structure look right? Any adjustments before I write the file?"* That's a structure check, not a content review — it confirms every AC has a row, not that the row is *right*. And Step 9 isn't even the real gate: the plan is written with `Plan Status: Draft` regardless of how you answer, and the actual PO decision happens afterward, against the written file, via `update-status ... Confirmed`. That's the moment this checklist is for — after you've opened `projects/{CODE}/story-plans/US-{epic}-{seq}-plan.md` and can read the real API/Web/Worker blocks, not the one-line summary.

FW-031 names exactly what wrong interpretations this step exists to catch — a wrong auth-guard name, a wrong tenant-scoping clause, a wrong screen route. Turn that into questions you can actually answer against the written file:

| Check against | Ask yourself |
|---|---|
| Each `API-N` block's **Auth** line | Does the guard/role named match who you'd actually expect to perform this action — not a plausible-looking placeholder? |
| Each `API-N` block's **Queries** line | Is the tenant-scoping clause (or soft-delete exclusion) actually present when this project has a tenant boundary — or correctly absent when it doesn't? |
| Each `WEB-N` block's **Route** and **Surface** | Does this match the actual screen you approved in the mock — same folder path, same Page/Dialog/Sheet/Drawer, not just a plausible-sounding one? |
| Each `WEB-N` block's **Data layer** | Does the hook name (or hookless mechanism) and the `API-N` it calls actually line up with what you'd expect this screen to do? |
| Section 1 (AC → Artifact Traceability) | Does every AC map to an artifact that genuinely satisfies it — not a loose association that happens to share a keyword? |
| Section 6 (Risks / Open Questions) | Is there anything here you'd actually resolve differently than what's proposed, rather than nod through because it's already written down? |

If any answer is "no" or "I'm not sure," that's a Change through the normal review conversation with the AI, applied before you type `Confirmed` — not a note to fix during implementation. A plan you correct after `/implement-story` has already started building against it costs you the same re-implementation chain a wrong architecture decision does, just scoped to one story instead of the whole system.

---

## What to expect from repo scaffolding on first touch

`/implement-story` Step 3 bootstraps a repository the first time any story touches it — `nest new`, cloning `ascendra-ui`'s component library, wiring the global exception filter and structured logging (`FW-041`), generating the frontend auth session module and typed API client (`FW-043`/`FW-042`), setting up test tooling, writing `README.md`/`CLAUDE.md`. Step 3 is thorough about describing *what* happens. Nothing upstream of it — not the gate-check messages, not the story plan — tells you *in advance* that this is coming.

Say it plainly before you hit it the first time: **if this is the first story for a given repo, expect the diff to include an entire scaffolded project, not just this story's own endpoint or screen.** A one-endpoint story can produce a diff touching dozens of files once you count the module structure, the exception filter, the logging setup, the auth module, and the `.env.example`. That's expected bootstrap work — the first story is quietly paying down a setup cost every later story in the same repo won't have to — not scope creep, and not a sign the AI over-built the story. `gen-pr-description.md`'s own diff-fidelity discipline (Step 3: *"Do not infer 'What Changed' from the story's ACs alone — the PR description must reflect the actual diff"*) means this doesn't get hidden from you either: a first-touch PR's "What Changed" section will honestly list the scaffold alongside the story's own work, not pretend the diff is smaller than it is. Read that as the safeguard it is — if a *later* story on an already-scaffolded repo produces a similarly oversized diff, that's the actual signal worth chasing down.

---

## The asymmetric recovery loop — and what to actually do when something fails

`/implement-story` and `/verify-story` fail in genuinely different ways, and the framework only gives you a clean procedure for one of them.

**Implementation failure has a clear, bounded rule.** Step 5: *"If you cannot fix a failure after two attempts, report it explicitly — state exactly what failed and why — rather than suppressing or skipping it."* Two tries, then stop and tell the PO. You always know where you stand.

**Verification failure doesn't.** `/verify-story` Step 8's failing-AC message says: *"Resolve these before raising the PR. Re-run `/verify-story` after fixing."* It doesn't say *how* to resolve them. The instinct to re-run `/implement-story` is the wrong one and the command itself will stop you: Step 1's gate check requires the story be `Reviewed`, and a story that's already been implemented is `In Progress` — re-running `/implement-story` on it hits that gate and refuses to proceed.

**The actual recovery path:** fix directly on the same story branch (`story/{STORY-ID}-{short-slug}`, already checked out from `/implement-story` Step 3) — edit the code yourself or direct the AI to make the specific correction, re-run lint/build/test, commit — then re-run `/verify-story` against the fix. You're not re-planning or re-scaffolding; you're correcting code that already exists on a branch that already exists.

**A second-order signal worth knowing:** if the *same* acceptance criterion fails verification twice in a row, treat that as a signal the story **plan** is wrong, not just the implementation. A misread requirement, a wrong endpoint assumption, a guard name that doesn't actually exist — these are plan-level errors that no amount of re-fixing the code will resolve, because the code is faithfully building the wrong thing. At that point, stop attempting a third blind implementation fix. Go back to `/gen-story-plan`, regenerate the plan (Step 10 handles this: a new Document Control row, `Plan Status` reset to `Draft`), have the AI walk you through what changed and why against the checklist above, confirm it again, and only then resume implementation.

---

## Two real template mismatches to watch for

The framework's generating commands and their templates are supposed to agree. Two places in this stage they currently don't — worth knowing about since neither file flags the other.

**Story-plan template's Data Layer field is hardcoded to React Query language.** `story-plan.template.md` Section 3 (Web Plan) states the field as `**Data layer:** [React Query hook name], [query / mutation], calls [API-N above]` — literally naming React Query in the placeholder. But `gen-story-plan.md` Step 5 gives the actual rule: *"if it uses hooks (e.g. React Query, SWR), name the hook... if the confirmed approach is hookless (e.g. Server Components only), describe the actual fetch mechanism instead. Never assume React Query if a different (or no) approach was confirmed."* The command's instruction is the correct, conditional one; the template's literal wording is stale. If this project's confirmed frontend approach is hookless, don't take the template field label as gospel — follow the command's conditional instruction and describe the real fetch mechanism instead. There's no worked hookless example in `story-plan.example.md` to pattern-match against either (its one full worked example, US-01-002, is a React Query hook case) — expect to improvise the format the first time you hit a hookless project, and don't read the absence of a hookless example as evidence the hookless path is unsupported.

**PR description template's Pre-Merge Checklist hardcodes two stack-specific items as unconditional.** `pr-description.template.md` Section 6 lists, as plain checklist items with no conditioning: *"`orgId` is sourced from the JWT payload — not from the request body, URL parameter, or query string (backend PRs)"* and *"React Query cache is invalidated in the relevant mutation hook for any endpoint that mutates data (frontend PRs)."* Both assume this project's actual tenancy mechanism and data-fetching choice — exactly the kind of unconditional framework-level assumption [`FW-029`](../decisions/FW-029-stack-agnostic-implementation-commands.md) and [`FW-030`](../decisions/FW-030-project-agnostic-implementation-commands.md) exist to ban everywhere else in this stage. Treat these two items as illustrative, not universal: on a project with a different tenancy scheme (no `orgId`, or tenancy carried some other way) or no React Query, tick "— N/A" honestly rather than forcing a match that doesn't actually describe what the diff does. The template's own AI Guide for this section already gives you the cover to do this — *"If an item is not applicable... tick it and add '— N/A' after the label rather than leaving it unchecked"* — you're applying an instruction the template already states, just correcting which items it applies to.

---

## What "stack-agnostic" actually means when you're reading `/implement-story`'s output

`/implement-story` determines this project's confirmed stack from the architecture document's Section 2 before doing anything else — it never assumes NestJS/Next.js universally, even though that's the framework's common default and the stack Harborview itself confirms. Per FW-029, the NestJS/Next.js syntax you see fully worked out in Steps 3 and 4 is a **reference path for the common case**, not a hardcoded universal rule; every stack-specific instruction is explicitly conditioned ("if the confirmed backend is NestJS...") with a stated fallback to "apply the same underlying principle using whatever framework this project's own `arch-v1.md` documents." Per FW-030, the same agnosticism applies to project-specific domain content — guard names, role names, tenancy rules are read from the architecture document, never assumed as framework law.

What this means practically: if you're running this command against a project whose Section 2 confirms something other than NestJS/Next.js — a Java/Spring backend, an Angular frontend, a different ORM — don't be alarmed when the command's own internal reasoning names that framework's idiom instead of NestJS's. Seeing `spring init` and a `@RestController` annotation named in the command's reasoning on a Spring project isn't the command drifting from a "real" standard hardcoded somewhere — NestJS was never the real standard to begin with, just the worked example this framework's own home project happens to use. The command working correctly on a non-default stack looks exactly like it *not* following the NestJS/Next.js content you've read in this chapter and in `implement-story.md` itself — that's the intended behavior, not a bug to flag.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s terminology map. Terms specific to this phase:

- **Plan Status (`Draft`/`Confirmed`)** — the story plan's own gate field, set via `update-status`. Distinct from the story's own `**Status:**` field (`Reviewed`/`In Progress`/etc.) — a story can be `Reviewed` while its plan is still `Draft`. `/implement-story` checks the plan's Plan Status, not the story's Status, for this particular gate.
- **Target** (`API`/`Web`/`Worker`/`+`-joined) — set once, at `/gen-stories` time, from the Locked architecture's repo topology (`FW-030`). `/gen-story-plan`, `/implement-story`, and `/verify-story` all read it as authoritative, cross-checking rather than re-deriving it, except as a fallback for stories generated before this field existed.
- **Plan Conformance** — the section `/verify-story` adds to its report, comparing what the plan declared (Section 2/3/4) against what was actually delivered. A documented deviation (see next) is a Match-with-rationale, not a Defect; an undeclared one is.
- **Deviations from Plan (plan Section 7)** — where `/implement-story` records any point it had to diverge from what the Confirmed plan stated, and why. Blank at generation; filled in only if execution genuinely needs to depart from the plan.

---

## Common mistakes

- Treating Step 9's "does this look right?" as the real review and typing `Confirmed` off the summary table instead of opening the actual written plan file and running the checklist above against it.
- Panicking at an oversized first diff on a repo's first story instead of recognizing scaffolding-on-first-touch for what it is — expected, and honestly named in the resulting PR description.
- Re-running `/implement-story` to fix a verification failure — it hits the `In Progress` status gate. Fix on the existing story branch directly, then re-run `/verify-story`.
- Attempting a third blind implementation fix when the same AC has already failed verification twice — that's a plan-correctness signal, not an implementation-effort signal. Regenerate the plan instead.
- Copying `story-plan.template.md`'s Data Layer field wording literally on a hookless frontend project instead of following `gen-story-plan.md` Step 5's conditional instruction.
- Forcing a match on `pr-description.template.md`'s `orgId`/React Query Pre-Merge Checklist items on a project that uses neither — tick "— N/A" honestly instead of leaving them unchecked or rewriting the diff's story to fit the checklist.
- Reading a non-NestJS/Next.js framework idiom in `/implement-story`'s own reasoning as a defect, on a project that genuinely confirmed a different stack — that's the stack-agnostic design working as intended.
- Confirming a plan without checking its Auth, Queries, Route, and Data-layer lines against what you actually remember agreeing to during architecture review — the plan extracts from Locked architecture, but extraction can still misread the source.

---

## Harborview in practice

`story-plan.example.md`'s Example 1 — **US-01-002, Create invoice form** — is the worked API+Web plan to read end to end. Running the checklist above against it directly: the **Auth** line on `API-1` (`JwtAuthGuard`, `@Roles('finance_manager')`) names the exact role you'd expect for invoice creation; the **Queries** line (*"insert scoped to caller's `orgId`"*) shows tenant-scoping actually present; `WEB-1`'s **Route** (`/invoices/new`) and **Surface** (Page, SCR-014) match a real screen ID rather than an invented path; and `WEB-1`'s **Data layer** line (`useCreateInvoice` mutation hook, calling `API-1`) is the React Query case this template's field wording was written for — contrast this against the hookless case described above, which you won't find worked out anywhere in this file. Section 5 (Risks / Open Questions) is honestly "None." here — a real instance of a plan with nothing left to flag, not every plan you write will be this clean.

`pr-description.example.md` — **US-01-001, Invoice database schema** — is the PR description to read alongside it, specifically for how its Pre-Merge Checklist handles the two hardcoded items flagged above: *"`orgId` is sourced from the JWT payload — N/A (migration only, no API endpoint)"* and *"React Query cache is invalidated in the relevant mutation hook — N/A (backend-only PR)."* This is the template's own AI Guide instruction — tick and annotate rather than leave unchecked — already in honest practice in the one worked example the framework ships, and it's the exact move to make on your own project whenever either item genuinely doesn't apply to what the diff does.
