# Stage 1 — Intake

## Purpose

Everything downstream — domain knowledge, BRD discovery, architecture, every story `/implement-story` ever writes — gets loaded as context that traces back to one file: `brief.md`. Intake is where that file gets written. It is not a form-filling exercise; it is the one point in the lifecycle where you decide what "this project" even *means* (its boundary, its code, whether it stands alone or extends something else) and hand the AI enough real client context that it stops asking you to re-explain the client in every later session. Get intake thin or wrong and the damage doesn't show up here — it shows up three phases later as a BRD that's missing a constraint nobody wrote down, or an architecture decision made without knowing a regulation applied.

Two commands run this stage: [`/init-project`](../.claude/commands/init-project.md) creates the project and its identity; [`/run-intake`](../.claude/commands/run-intake.md) fills `brief.md` section by section. `CLAUDE.md` and `PROJECT-LIFECYCLE.md` tell you these two commands exist and what order they run in. This chapter tells you what to actually say to them.

---

## `/init-project` → `/run-intake`: the decision flow

### Project boundary — one product, one project

*(A process decision made at project creation, before `brief.md`'s content exists — not something checkable by re-reading the finished document.)*

`/init-project` Step 2 asks a single question before it creates anything: *"In one sentence, what is this project — what product or capability does it deliver, and who uses it?"* If your answer names more than one distinct product or user group, the command stops you before a folder is even created. The built-in test is already good: **a web app and its API are one project (technical components); separate portals for different user groups are separate projects.** Three edge cases you'll actually hit that the one-line rule doesn't spell out on its own:

- **Teacher portal + parent portal, same school system.** Two projects. Even though they share a backend, a domain, and probably a client, a teacher (recording attendance and grades) and a parent (viewing them and paying fees) are different user groups solving different problems. Each gets its own brief, its own BRD, its own lifecycle — even if the architecture later decides they share one API.
- **Admin dashboard bolted onto an existing product, for the *same* users.** If the Finance Manager who already uses Harborview's invoicing system also gets a screen to manage invoice categories or reminder templates, that's not a new project — it's new scope inside the existing one. Same user, same purpose, no independent go-live date. This belongs in the existing project's BRD as new requirements, not a new brief.
- **Admin/ops console for a *different* internal team.** If instead the capability is a cross-tenant console for Ascendra's own support staff to manage every client's invoicing account — a different user group, a different purpose (platform operations, not invoicing), and a capability that could ship on its own schedule regardless of what the Finance Manager's roadmap looks like — that's a separate project. It will very likely be an **Extension** of the original (see below), because it can't be understood without the parent's domain knowledge, but it still gets its own brief and code.

The test that generalizes across all three: *would this capability have its own go-live date, its own primary user group, and its own BRD sign-off, even if the "host" project never shipped it?* If yes, separate project. If it only exists as an addition to what an existing user group already does inside an existing project, it's scope, not a boundary.

### Project code format — `CLIENT-DOMAIN-NNN`, and when to add `EXT`

The baseline format, per [`FW-008`](../decisions/FW-008-project-code-naming.md), is `CLIENT-DOMAIN-NNN` — `HARBORVIEW-INV-001`. `/init-project` also accepts a fourth segment, `CLIENT-DOMAIN-EXT-NNN`, when the project is an extension of another project and you want that relationship visible in the code itself (this validator was one of the 14 defects the framework audit fixed — earlier it rejected 4-segment codes outright, contradicting FW-008's own canonical examples).

The critical thing to understand about the `EXT` segment: **it is cosmetic.** It exists so a human scanning `projects/index.md` or a folder listing can guess the relationship without opening any file. It is never read by any command to determine behavior — the real extension relationship is always the brief's `Project Type` / `Extends` / `Extension Adds` header fields, filled during `/run-intake`, per [`FW-016`](../decisions/FW-016-brief-driven-extension-model.md). A project code with an `EXT` segment whose brief says `Project Type: Standalone` is legal (if confusing) — the brief wins every time.

Three worked examples:

| Code | Reading |
|---|---|
| `HARBORVIEW-INV-001` | Harborview, invoice domain, first iteration. Standard standalone or root-of-chain project. |
| `ASCENDRA-PAY-CORE-001` | Ascendra's own payment product, core layer. The `CORE` segment signals "this is the base of an extension chain" — still cosmetic; the brief's `Project Type: Standalone` (a chain root has no parent) is what actually says so. |
| `ASCENDRA-PAY-PKN-001` | The Pakistan market layer on top of `ASCENDRA-PAY-CORE-001`. The `PKN` segment is a human hint. The brief's `Extends: ASCENDRA-PAY-CORE-001` is the actual dependency. |

Don't spend time engineering a clever code. Get `CLIENT-DOMAIN-NNN` (or `-EXT-NNN`) structurally right, then let the brief carry the real relationship.

### Standalone vs. Extension — what actually changes

This choice isn't cosmetic — it changes what gets loaded into context for nearly every generating command downstream. FW-016's loading table, condensed:

| Command | If Extension: also loads from the parent chain |
|---|---|
| `/gen-domain-knowledge` | Parent domain knowledge, base-first — this project's document becomes a **delta only**, not a full restatement |
| `/gen-brd-playbook` / `/gen-brd` | Parent BRD as baseline context — this project's BRD captures only what's new or different |
| `/gen-architecture` | Parent architecture's Extension Points section specifically |
| `/implement-story` | Parent architecture, to know which codebase/service a story targets |
| `/gen-epics`, `/gen-stories` | Nothing — these are always project-specific regardless of type |

Practical effect: if you mark a project `Extension` and it really isn't, `/gen-domain-knowledge` writes a thin delta document assuming context that was never established, and your domain knowledge base ends up incomplete. If you mark it `Standalone` when it really depends on another project's domain, you get full duplicate discovery — wasted effort at best, contradictory definitions at worst.

**The decision test:** *would someone who has never seen the other project be able to fully understand, discover, and build this one from its own brief and BRD alone?* If yes — even if it shares a client, a similar domain name, or reuses some of the same code — it's Standalone. If no — if a reviewer would have to open the other project's domain knowledge or BRD to make sense of a requirement here — it's an Extension, and you name the direct parent only; the AI resolves the rest of the chain (with cycle detection) by reading each brief transitively.

### Interactive vs. hand-filled — sizing the intake session

*(A process decision about how the session is run, not something visible in the finished document's own content.)*

`/run-intake` and manually editing `brief.md` against its `[AI Guide]` notes are both fully supported paths — `/init-project`'s own Step 6 note says so. The value `/run-intake` adds over hand-filling isn't the writing, it's two mechanisms hand-filling doesn't replicate: the **per-section follow-up loop** (Step 5C asks one targeted question at a time when your answer is missing something the guide requires), and the **automatic Verification checklist** that gates "Brief complete" and that you'd otherwise have to run against yourself, honestly, under time pressure. Size the decision on how much those two things are worth to you:

- **Small internal project** (you are effectively both PO and client — an internal tool for your own team, low stakeholder count, no external discovery call happened or is needed): hand-filling is fine and usually faster. You already know the Problem Statement, the stakeholders, the constraints — there's no vague client answer for a follow-up question to catch, because there's no client in the room.
- **Medium client project** (a real external client, a discovery call happened or will happen, more than one stakeholder, some real constraints): run `/run-intake` interactively. Its single-question-at-a-time cadence mirrors an actual client interview, and Step 5C's follow-up prompts are specifically good at catching the vague answers a client tends to give in the room ("we want it to be efficient") before they get written down as if they were specific.
- **Large or complex project** (regulated domain, extension chain, multiple stakeholders with different Decision Authority, high budget/deadline stakes): always interactive, and run it as close to the actual discovery conversation as you can. The per-section approval gate plus the mandatory Verification checklist is the only mechanism in the framework that forces a check like "does every named regulation cite the specific statute" or "does every stakeholder have a named Decision Authority" before that gap becomes load-bearing context for a BRD discovery session, and eventually architecture, that you can't easily unwind.

If you do hand-fill, still run the Part 2 Verification checklist in [`brief.template.md`](../projects/TEMPLATE/brief.template.md) against your own draft before you approve it — nothing else will.

### Resuming, revising, amending

`/run-intake` reads the **Content Status** field to decide which of three modes it's in (see the state machine below for the full field):

- **Intake mode** (`Draft`) — the normal first pass. Sections are worked through 1→9 in order; anything already ✅ from a prior interrupted session is skipped, with an offer to review it first.
- **Revision mode** (`Brief complete`) — intake is done but not yet approved. Name a section, it re-runs the draft/approve loop for just that section, no verification re-run unless you ask.
- **Amendment mode** (`Approved`) — the brief is already gating downstream work. Any change here is a **tracked amendment**: after each write, the PO gives a one-line summary of what changed and why, and it's appended as a new row to a Document History table at the bottom of `brief.md` with an incremented minor version (1.1, 1.2, ...). Use this when something material changes after domain knowledge or BRD discovery has already started building on the approved brief — a stakeholder change, a moved deadline, a new constraint. Note that amending does **not** revert Content Status back to `Draft` or `Brief complete` — it stays `Approved` throughout, so an amendment is a visible audit entry, not a re-opened gate. If the change is big enough that downstream artifacts might now be wrong, that's a judgment call for `/assess-change`, not something amendment tracking does for you automatically.

---

## Section-by-section: how much is enough

This is the part the framework's other docs don't cover, and it's the part that actually determines whether your BRD discovery session starts productive or starts by re-deriving facts that were already sitting in the brief.

### The Content Status state machine

Every gated artifact in the framework tracks readiness with a single `Status` field. `brief.md` is the one exception — it has two independent fields, and this split exists nowhere else in the framework:

| Field | Tracks | Set by |
|---|---|---|
| **Status** | The whole project's lifecycle (`Active` / `On Hold` / `Complete` / `Cancelled`) | `/init-project` once, rarely touched again; mirrors `projects/index.md` |
| **Content Status** | Whether `brief.md`'s *content* is ready to use | `/init-project`, `/run-intake`, and `update-status` — see below |

Content Status is the field every downstream gate actually reads. Its full lifecycle, from [`update-status.md`](../.claude/commands/update-status.md):

```
Draft ──/run-intake completes all 9 sections + verification──▶ Brief complete
                                                                     │
                                                    PO reviews, runs `update-status ... Approved`
                                                                     ▼
                                                                 Approved ───▶ gates /gen-domain-knowledge
                                                                     │
                                              (optional, manual) formal freeze or replacement
                                                                     ▼
                                                          Locked / Superseded
```

- **`Draft`** — set by `/init-project`; every section is still the awaiting-intake placeholder.
- **`Under Review`** — a manually-set intermediate value (via `update-status`) if you want to flag the brief as out for client sign-off before formally approving it. `/run-intake` never sets this itself; it's yours to use if your process needs a visible "sent for review" state.
- **`Brief complete`** — set automatically by `/run-intake` Step 8, the moment all 9 sections are filled *and* every Part 2 Verification check passes. This is not a PO action — it's the command confirming its own gate.
- **`Approved`** — set manually, by you, running `update-status projects/{CODE}/brief.md Approved`. This is the actual gate: `/gen-domain-knowledge` checks for exactly this value before it will run.
- **`Locked` / `Superseded`** — available values, meaning "frozen, formal change request required" and "replaced by a later version" respectively. Neither command file specifies an exact trigger for these on the brief specifically (unlike BRD/architecture Lock, which has a defined rationale-prompt flow in `update-status.md`) — treat them as available for your own process discipline (e.g., lock the brief once architecture is locked and nothing about "why" should change again) rather than something the framework will prompt you into.

Two things worth remembering here so you don't go looking for this pattern elsewhere: this two-field split is **unique to the brief** — every other artifact (BRD, Architecture, Epic, Story) uses one `Status` field for both purposes, so don't hunt for a separate Content Status on the BRD; it isn't there. And Content Status governs the per-section flow above directly — which mode `/run-intake` opens in is a direct read of this field.

### 1. Problem Statement

Two to four sentences, answering in order: what's happening now that shouldn't be (or vice versa), what it costs, and what would be true if it were solved. Write it from the client's perspective, describing the current-state pain — never the solution. [`brief.example.md`](../projects/TEMPLATE/brief.example.md)'s version names the actual mechanism of failure ("the spreadsheet fails to flag overdue invoices automatically, leaving the Finance Manager to manually chase payments every week") rather than a generic complaint ("invoicing is inefficient"). That specificity is the bar: a bad Problem Statement is either too thin ("the client needs better invoicing" — no mechanism, no cost) or already the solution ("we are building an automated reminder system" — this is explicitly the template's own incorrect example).

**What breaks downstream if this is wrong:** too thin, and BRD discovery opens with the AI asking you to re-explain the same context a client discovery call already surfaced — wasted session time re-deriving what should already be written down. Written as a solution instead of a problem, and it quietly biases BRD requirement discovery toward one implementation before discovery has had a chance to explore alternatives — you've pre-decided the "what" in a document meant only to capture the "why."

### 2. Client Context

Enough that a developer or AI agent joining mid-project understands the client's business without asking follow-up questions: who they are, their scale (team size, transaction volume — whatever affects system design), their current process, and anything that would change a design decision (multi-country operation, a compliance team, regulated customers). Harborview's version names concrete numbers — 25 staff, 40–60 invoices a month, 30 clients, one Finance Manager, three Directors, an existing but unconnected Stripe account — every one of which is a fact that later shapes a real requirement or architecture decision. What it deliberately omits: company history, mission statements, anything that wouldn't change how the system is built.

**What breaks downstream if this is wrong:** too thin (no transaction volume, no team size), and neither domain knowledge nor architecture has a basis for scoping decisions like expected data volume, concurrency, or entity cardinality — you'll find yourself guessing at architecture time what should have been established at intake. Padded with marketing or history, and you've added dead weight to a document that gets loaded as context into nearly every subsequent AI session — every irrelevant sentence here is a sentence competing for attention in every future prompt.

### 3. Strategic Goals

Two to five outcomes, not features. The test in the template is the sharpest tool in this whole chapter: *could the client achieve this goal without the system, in principle?* If yes, it's a goal — the system is one way to get there. If no, it's a feature, and belongs in the BRD, not here. Harborview's "Eliminate manual invoice tracking — full AR visibility in real time" is a goal (they could in principle hire more finance staff to get there); "add automated email reminders" is a feature (there's no non-system way to automate an email). If the client gives you something vague ("be more efficient"), your job is to sharpen it into something observable, not to write down the vague version.

**What breaks downstream if this is wrong:** goals that are secretly features get treated by BRD discovery as if they were already decided — the discovery session skips exploring alternatives for something that should still have been an open design question. Genuinely vague goals ("be more efficient") give the BRD nothing to check its own priority calls against — you lose the ability to answer "why is this Must Have?" by pointing back to a strategic goal.

### 4. Key Stakeholders

Only people whose involvement affects delivery — decision-makers, primary users, anyone who can block or derail the project — not a company directory. Every row needs a named Decision Authority from the four defined options (Final sign-off / Input only / Daily user / End user only) and a real description of their expected involvement. Harborview's table names four rows, each with a distinct authority level, not a generic "stakeholders: Harborview staff."

**What breaks downstream if this is wrong:** missing or vague Decision Authority means the BRD's approval workflow and UAT sign-off are ambiguous — you can build a whole sprint's worth of stories and discover at UAT that the person who actually needed to approve them was never identified. Listing every employee at the client organisation, on the other hand, inflates every later session that loads this table without adding real signal.

### 5. Constraints

Hard, non-negotiable limits only — budget, deadline (with the *reason* it can't move, not just a date), mandated technology, named regulations, anything else genuinely fixed. The template's own discipline: if a constraint can be relaxed with a conversation, it's a preference, and belongs in Section 6, not here. "None" is a valid, expected answer for a category with no real constraint — never leave a row blank. Regulatory citations must be specific: Harborview's brief names "UK GDPR" and "UK Companies Act 2006, Section 386," not "privacy laws" or "data retention rules."

**What breaks downstream if this is wrong:** a vague regulatory constraint ("privacy laws apply") gives architecture nothing concrete to design against — no specific data residency rule, no specific retention period, no specific consent mechanism to implement, and you'll be reopening this conversation at architecture time instead of building from it. Listing a genuine preference as a constraint does the opposite kind of damage: it locks architecture out of options that were actually still open, for no real reason.

### 6. Technology Preferences

The client's actual technology landscape: existing systems that must be integrated with or replaced (factual, not optional), stack preferences (and whether they're a strong preference from prior investment or a mild one), hosting, and existing identity/auth provider. "No preference — client defers to delivery team" is the correct answer when true, and should be written exactly that way, not left blank. Harborview's Section 6 names Stripe and Gmail as existing systems, states no stack preference, and is explicit that authentication is standalone (no SSO) — small facts, each one directly usable by the architecture phase.

**What breaks downstream if this is wrong:** omitting an existing system the new system must integrate with (a payment processor, an email platform) means architecture designs without knowing that integration surface exists, and you're redesigning around it later. Writing a preference as if it were a hard requirement — without noting it's negotiable — causes architecture to over-engineer around a constraint you never actually meant to impose.

### 7. Success Criteria

Three to seven criteria, each one a Product Owner could verify in UAT without asking the client for a subjective opinion. The template's test: could you check this off during UAT unaided, or would you need the client to say "yes, that feels right"? "Finance Manager can create and send an invoice in under 5 minutes from login — verified in UAT" passes; "the system is easy to use" fails — it's an expectation, not a criterion. Fewer than three suggests the client hasn't thought carefully about what success means yet; more than seven usually means the list has been diluted with things that were always obvious.

**What breaks downstream if this is wrong:** vague criteria give `/gen-uat-checklist` nothing concrete to translate into a pass/fail scenario — you end up writing UAT scenarios from scratch at UAT-checklist time instead of deriving them from something already agreed at intake. A bloated list (more than seven, overlapping with individual requirements) turns UAT into a feature checklist instead of an outcomes checklist — you lose the ability to distinguish "did we build the right thing" from "did we build everything."

### 8. Risks and Open Questions

Two distinct item types in one table, never combined in a row: a **Risk** is something that might go wrong regardless of whether anyone acts on it, and needs a mitigation plan; an **Open Question** is something that must be answered before or during BRD discovery, without which a requirement can't be written. Every row needs a named owner — an unowned item is, in practice, an item nobody will ever resolve. Harborview's table is a good model of density: each Open Question has a concrete due date and an explicit resolution status, and each Risk has a stated mitigation, not just a description of the danger.

**What breaks downstream if this is wrong:** an Open Question with no owner (or "TBD" in the owner column) tends to stay open indefinitely — BRD discovery either stalls waiting on it or, worse, proceeds on an unstated assumption that turns out wrong. Misfiling a real Risk as an Open Question (or vice versa) means it never gets the mitigation plan a Risk actually needs — it just sits there as an unanswered question with no plan for what happens if the answer is bad.

### 9. Related Artifacts

Auto-populated by `/run-intake` with the brief itself as the first row; every artifact created afterward (BRD, architecture, sprint plans) should be added as it's produced, so the table works as a navigation index — someone should be able to find every document for this project from this table alone, without searching the filesystem. One thing worth knowing explicitly: not every generating command auto-maintains this table today — `/gen-brd`, `/gen-architecture`, `/gen-sprint-plan`, `/gen-release-notes`, and `update-status` itself don't all keep Section 9 current automatically (a known partial-implementation gap). The safest habit: run `/update-status` after each artifact changes status, and don't assume Section 9 is accurate without checking — treat a stale "Draft" row next to an artifact you know is Approved as a sign to fix the table by hand.

**What breaks downstream if this is wrong:** a stale entry (an old "Draft" status on a document that's actually Approved, or a missing row for a document that exists) sends whoever — you, a teammate, an AI session — to the wrong version of a document as if it were current. Since the brief is loaded as context into later sessions, a wrong Section 9 doesn't just mislead a human reader; it can mislead the AI about which BRD or architecture version is authoritative.

---

## Terminology recap

The Content Status vs. Status split above is covered in full in [`00-orientation.md`](00-orientation.md)'s terminology table — that entry, plus the Extension vs. Core/Extension-architecture distinction (relevant the moment you mark a project `Extension` in this stage), are both worth re-reading before your first real intake session. This chapter doesn't repeat them; go there for the reference version.

---

## Common mistakes

- Writing the Problem Statement as a feature list ("we're building X, Y, Z") instead of the current-state pain — the template's own incorrect example, and the single most common way Section 1 goes wrong.
- Marking a project `Extension` because it shares a client or a similar-sounding domain, not because it actually can't be understood without the parent's domain knowledge or BRD.
- Treating a genuine preference ("they'd like to use Postgres") as a Section 5 Constraint instead of a Section 6 Preference — this silently removes architecture options that were never actually closed.
- Leaving Section 8 owners as "TBD" — the template explicitly calls this out as a Verification failure, but it's an easy field to rush past in a live session.
- Approving the brief (`Content Status: Approved`) and then making a material change without running `/run-intake` in amendment mode — the edit happens, but with no Document History entry and no visible audit trail for anything already built on the earlier version.
- Choosing to hand-fill a brief for a project that actually needed a live client interview — the Verification checklist will still technically pass on vague-but-present answers; only the interactive follow-up loop reliably catches vagueness in real time.

---

## Harborview in practice

[`brief.example.md`](../projects/TEMPLATE/brief.example.md) is the finished, Approved artifact this whole chapter has been pointing at. Worth noticing on a direct read: Section 1's Problem Statement never once describes the system being built — it stays entirely in the language of the spreadsheet, the Finance Manager's weekly manual check, and the Directors' email-forwarding approval process. And Sections 5 and 6 are a clean worked example of the Constraint/Preference split this chapter leans on hardest: Stripe is a Section 5 Constraint (an existing negotiated rate, non-negotiable), while stack and hosting sit in Section 6 as genuine preferences ("No preference — client defers to delivery team") — the same document shows both sides of the distinction side by side, which is the fastest way to internalize it.
