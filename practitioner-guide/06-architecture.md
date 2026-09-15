# Architecture

Covers Stage 6 in [`SDLC.md`](../SDLC.md): [`/gen-architecture`](../.claude/commands/gen-architecture.md) and [`/review-architecture`](../.claude/commands/review-architecture.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't. This is the densest phase in the framework — the architecture document is the last artifact with no code behind it and the first one every subsequent command treats as literal, binding fact. Get a judgment call wrong here and it doesn't cost you a re-write of a document; it costs you a schema migration, a re-implemented endpoint, or a security defect nobody notices until a client does.

---

## Purpose

`/gen-architecture` turns an approved BRD, approved epics, and (if there's UI) an approved Screen Design into the complete technical blueprint: every module, every table, every endpoint, the security model, seed data, environment variables. `/implement-story` implements exactly what this document says — nothing more, nothing less. `/review-architecture`'s own framing states the stakes plainly, and it's worth taking at face value: *"Every table name, column, endpoint, auth rule, and enum value becomes law for every story and every line of code."* A requirement misunderstood here costs BRD update → architecture revision → story updates → re-implementation → re-verify → re-PR → re-UAT — the most expensive correction chain in the whole framework.

Two commands, two jobs, same as Epics: `/gen-architecture` derives and proposes; `/review-architecture` runs a 35-check structural audit across five concerns before the Product Owner locks the document. This chapter is about the judgment calls embedded in both — several of which are genuinely dense, bundled into compound confirmations, or resolved by heuristics the command never surfaces as a decision point at all.

---

## What to prepare beforehand — and what the command figures out on its own

State this plainly, because [`CLAUDE.md`](../CLAUDE.md)'s Workspace Setup step 5 reads as a prerequisite and isn't one: **pre-authoring `standards/` is optional, not required.** `/gen-architecture` works from a completely empty `standards/` folder by default — Steps 4 through 7 derive the tech stack, the architecture pattern, and every standards file conversationally, presenting what it derived and asking only what it genuinely can't infer. A first-time PO who reads CLAUDE.md literally and tries to hand-write `api-standards.md`, `database-standards.md`, and `security-standards.md` from the bare template before running the command is doing work the command will do for you, against a template with no worked instance to calibrate against.

Pre-authoring earns its keep in exactly one situation: you have a **hard, non-negotiable mandate** — an existing cloud vendor contract, a database the client has already paid for, a compliance requirement that rules out an option the AI might otherwise recommend — and you want it locked in *before* the AI proposes something else and you have to walk back a recommendation mid-conversation. If that's your situation, write only the file carrying that mandate (usually `system-architecture.md`, sometimes one Tier 1 standard); `/gen-architecture` reads whatever exists in Step 2 and Step 5 explicitly asks "should I proceed with this stack, or change anything?" rather than silently overwriting it.

If you do want to hand-author something, or want to sanity-check a generated file against a real bar, two worked examples now exist for exactly this — a defect the framework's own audit found and closed:

- [`technical-standard.example.md`](../projects/TEMPLATE/standards/technical-standard.example.md) — Harborview's `api-standards.md`. Read it for the shape: cite the fixed wire contract rather than restate it, a Correct/Incorrect pair for the one rule generation gets wrong most often, a table of concrete business-rule exception codes tied to REQ IDs, and one complete controller/service/DTO example (`POST /invoices/:id/void`) applying every rule together.
- [`architectural-pattern.example.md`](../projects/TEMPLATE/standards/architectural-pattern.example.md) — Harborview's `idempotent-stripe-webhook-handling.md`, a genuinely warranted Tier 3 document (money-moving, externally-triggered, retry-prone — the concrete signal the template's own AI Guide asks for, not a generic "might be useful" addition). Read it to calibrate *when* a project-specific pattern document earns its own file instead of a paragraph inside the architecture document itself.

Both are dense in the same way REQ-002 was dense in Chapter 3 — every sentence is doing real work, not padding a template out to look thorough. That's the bar.

---

## The dense judgment calls inside `/gen-architecture` itself

Three decisions in this command are consequential, hard to reverse, and easy to skim past because none of them is asked as its own clearly-flagged question. Slow down for all three.

### Actor Identity Shape — a schema decision disguised as one bullet in a derivation summary

Step 4 (Constraints Discovery) derives five things from the BRD and presents them together in one block — scale, integrations, multi-tenancy shape, compliance signals, and **Actor Identity Shape** — then asks four separate, explicit questions (deployment target, team capability, budget posture, compliance confirmation) and invites you to "confirm or correct my derivations above" in the same breath. Actor Identity Shape is the fifth derivation, not one of the four questions — which means it's easy to read past it as background context rather than a decision you're being asked to make.

**What's actually being decided, in plain language:** every Master table in the database (invoices, payers, staff records — every primary domain record) carries three audit columns: who created this row, who last updated it, who deleted it. The command has to pick one of two shapes for those columns:

- **A clickable reference** (a `uuid` foreign key into one user table) — works only if every actor in the system is the same kind of authenticated, in-tenant user.
- **A plain name string** (`varchar`, no foreign key) — required the moment there's more than one kind of actor: an authenticated Staff member *and* an unauthenticated customer reached only via a tokenized link, *and*/or a scheduled job or webhook acting on its own. A single foreign key can't point at three different kinds of thing — for any action taken by the second or third kind of actor, the column would just silently be `NULL`.

Say it to yourself the way you'd want it said to a client: **every record will show a name string for who changed it, not a clickable reference to a user account you could click through to see their profile — and this is genuinely hard to change later, because it's not a config flag, it's a migration touching every Master table in the schema, project-wide, at once.** The full reasoning — including why a third option (a type-discriminator column alongside an opaque id) was considered and rejected on cost grounds — lives in [`FW-037`](../decisions/FW-037-audit-column-actor-label-design.md), but nothing in the command surfaces that reasoning to you in the moment; you get one derived sentence in a bundled summary.

**What to actually do:** when Step 4's summary reaches the Actor Identity Shape line, stop and read it on its own, separately from the four questions below it. If the derivation says "multiple actor identity spaces," ask the AI to name every actor kind it counted and why — a webhook, a scheduled job, and a tokenized-link customer are each their own identity space even though none of them "logs in" the way a Staff member does. Getting this wrong doesn't fail a check later; check 18 in `/review-architecture` only confirms the mandatory columns exist, not that their *shape* is right for this project's real actor population.

### Deviation detection's blind spot — Section 9 can't catch what it's built from

Section 9 (Deviation Declarations) is precisely defined: it flags anything in the architecture document that differs from what's recorded in `standards/system-architecture.md`. That's a genuinely useful mechanism for catching *later* drift — someone hand-edits the architecture document after generation, or a standards file gets updated and the document doesn't follow. `/gen-architecture`'s own Step 3 (Standards Drift Check) exists specifically for that case.

What it structurally cannot catch: a bad choice made in the *same* confirmation that produces the baseline it checks against. Step 5's tech-stack table — Frontend, State Management, Backend, Language, Database, ORM, Cache, Logging, Error Tracking, Containerisation, CI/CD, Hosting, twelve rows presented and confirmed together — *is* `system-architecture.md` the moment you confirm it. If a row is wrong and you approve it anyway, Section 9 has nothing to compare it against; the "wrong" choice and the baseline are the same document.

**Say it plainly:** the only real guardrail on that twelve-row stack confirmation is your own attention in the moment it's presented. Nothing downstream re-litigates it. If you're not sure why a row was recommended — why Redis over an in-database queue, why NestJS over a lighter framework, why this specific hosting target — **ask the AI "why this and not X" before you confirm, not after.** Don't wait for `/review-architecture` or a later Section 9 flag to catch it, because by design it can't.

### Composable capability detection — a silent fork off BRD wording

Section 6.1 scans the BRD's User Roles section for composable-permission language — phrases like "independently assignable" or "any user may hold multiple capabilities at once" — and if it finds a match, switches the entire JWT payload shape from a single `role: string` to `capabilities: string[]` (plus an optional `viewOnly: boolean`). This is not a small stylistic choice: it changes the RBAC table structure in Section 6.2, every guard downstream, and how `/implement-story` writes every permission check in the codebase. It's decided once, silently, by pattern-matching, and only surfaces to you as already-settled fact when `/review-architecture` frames Concern D — never as its own "I detected X — confirm?" checkpoint the way the tech stack and repo topology are.

**What to do:** when Concern D's framing states which model was detected — composable capabilities or a fixed role list — don't read past it as background. Check it against your own memory of how the client actually described permissions during discovery. If the BRD says "any Staff member can be granted Configuration access and Write-off Approval independently" and the framing says composable, that's correct. If the framing says composable off a looser phrase that doesn't really mean independent assignment, say so before Concern D closes — reversing this after stories are written means rewriting every guard, not editing one sentence.

---

## Using the 35-check review without redoing the AI's work

`/review-architecture` is genuinely well designed: five concerns (Module Structure, Data Model, API Contracts, Security & Access Control, Infrastructure & Compliance), each opening with a plain-language "Frame this concern" narration before any check runs, each closing with a targeted PO question aimed at exactly the kind of gap a structural check can't find. The honest limitation the framework's own audit surfaced: **the 35 checks themselves run off-screen.** You see "FLAGS — Concern B: Data Model / No flags — all Concern B checks passed" — a pass/fail summary, not the work. You cannot independently verify "no flags" without re-deriving every REQ-to-table mapping and every column list yourself, which is exactly the work the review exists to do *for* you.

### What's genuinely reviewable vs. what you're meant to trust

Nothing in the framework states this outright, so state it to yourself before you start reading a real `arch-v1.md`: **the plain-language framing and the FLAGS are the reviewable surface. The raw Drizzle/TypeScript schema blocks and DTO decorator code underneath are generation artifacts, checked structurally by the 35 checks — you are not expected to read them line by line.** Give yourself permission to stop trying to parse `@IsUUID()` decorators and Drizzle table definitions you're not equipped to audit, and redirect that effort to the parts actually written for you.

A compact map, keyed to section numbers, to use while reading a real document:

| Section | Genuinely reviewable | Why |
|---|---|---|
| 1 — System Overview | Yes | Plain prose, written for a non-technical stakeholder by design |
| 3.1.3 — Screen & Nav Map | Yes (carry-over check) | Already approved in `/review-screen-design`; here you're confirming nothing was lost in the carry, not re-deriving it |
| 4.2 — Table Definitions | Trust | Drizzle/TypeScript code — structurally checked, not written for PO reading |
| 5 — API Contracts / DTOs | Trust | DTO decorator blocks — same reason |
| 6.1 — role/capability list | Yes | A plain list of names — check it against who you know uses the system |
| 6.2 — Permitted Actions | Yes | Written as plain-English sentences per role, exactly for this purpose |
| 8 — Open Decisions | Yes | Must be empty before lock — read every row, there should be none |
| 9 — Deviation Declarations | Yes | Plain statements of what differs from the confirmed stack and why |
| 10 — Approval | Yes | Just confirm it's blank until you fill it |

### How to engage with each Concern

Read the "Frame this concern" paragraph in full every time — it's written to be understood without engineering background, and it's the thing most worth your attention. Answer the confirmation question honestly rather than reflexively. When FLAGS come back empty, don't take that as proof nothing's wrong — take it as "nothing the structural checks are built to catch is wrong." The place to actually push is the **Targeted PO question at the end of each concern** — these are the plain-language checkpoints the review actually built for a non-architect, and they ask things a structural check structurally cannot: "is there a capability you discussed with the client that doesn't have an obvious home," "does the full state list match what you'll need to report on," "is there any action a role should NOT be able to perform that isn't restricted here." Spend your real attention there, not on trying to eyeball the DTO blocks.

If something about a concern feels off and "no flags" doesn't sit right, the practical mitigation the audit surfaced is simple: **ask the AI directly which specific check numbers ran for that concern and what each one found.** The checks are numbered (1–34, plus 20a) and documented in `review-architecture.md` — nothing stops you from asking for the receipts on any one of them. Use that when a summary feels too clean, not as a routine step on every concern.

### The lock confirmation — use the rationale

Locking is the final, hardest-to-reverse gate in this phase. `/review-architecture`'s lock confirmation now captures an optional one-line rationale: reply `lock: {what you specifically verified}` (e.g. `lock: confirmed RBAC matches every BRD role and the schema covers every screen's data requirements`), or a bare `lock` with no rationale — both are accepted, and the command asks once if you decline to give one, then proceeds either way. It costs you one sentence. Given what locking commits you to — *"every table name, column, endpoint, auth rule, and enum value becomes law"* — that sentence is worth writing. It's not ceremony; it's the thing you point back to later if a decision gets questioned mid-project and nobody remembers exactly what was checked before the document became binding.

---

## Core/Extension methodology — genuinely PO-readable, most POs never find out

If BRD Section 1.6 named any extension dimension, Section 3.2 (Extension Points) applies [`reference/architecture/layered-domain-architecture.md`](../reference/architecture/layered-domain-architecture.md) — the framework's universal Core/Extension methodology — to generate the technical seams a future extension project will plug into. This document is unusually accessible for something this technical: its Membership Test is two sentences, and a non-architect can apply it directly:

> *"Would every deployment of this base system need this, regardless of which extension is active?"* If yes → Core. If no → Extension.

The framing you'll see in `/review-architecture`'s Concern A is a one-sentence summary per extension point — enough to confirm the table exists, not enough to actually judge whether the seam is drawn in the right place. **Don't stop at the summary.** Open the reference document itself and apply the Membership Test to at least the extension points that feel least obvious — the ones where you're not immediately sure why something landed on the Core side of the line rather than the Extension side. It reads in minutes and the test is genuinely yours to run, not something you need an architect to run for you.

For a worked instance of applying it to a real, non-Harborview project, see [`case-studies/harder-cases.md`](case-studies/harder-cases.md)'s Case 2 — the Ascendra Pay Pakistan extension — which walks the boundary test against a real Core/Extension split rather than Harborview's single, unexercised extension point (Accounting Export, `AccountingExportStrategy` — see Harborview in practice, below).

---

## The honest limit: ongoing conformance is manual

Say this plainly, because nothing in the framework says it outright: **"no undocumented deviations" is enforced by your own review, not by a standing structural gate.** Locking the architecture document makes it binding in principle. It does not create any command that continuously checks real running code against it.

The evidence is a repeated pattern across three separate ADRs, each describing the same failure shape:

- [`FW-033`](../decisions/FW-033-implement-story-standards-compliance.md) — `security-standards.md` and `arch-v1.md` both correctly specified JWKS token verification; the implemented code used a shared-secret check instead. Caught only when a real Supabase login failed in practice, not by any structural check.
- [`FW-040`](../decisions/FW-040-ascendra-ui-hard-instructions.md) — five separate UI-implementation gaps (wrong component used, a prop that doesn't exist, a documented pattern nobody actually follows in real pages) found only by a PO manually opening the implemented screens and comparing them against real `ascendra-ui` source.
- [`FW-041`](../decisions/FW-041-logging-and-exception-handling.md) — `nestjs-standards.md` described a global exception filter and structured logging in detail; neither existed anywhere in the actual codebase, for months, on a live project. Found by manual PO review, not by any test, lint, or gate.

In every case, the standard said X, the real code did something else, and the gap was closed by a person reading source code directly or by a real failure forcing the issue — never by a command whose job was "confirm the running code still matches the Locked document." `/implement-story`'s own self-check runs once, per story, at implementation time, invisibly — it is not a periodic audit and doesn't claim to be one.

**What this means for you in practice:** don't treat "the architecture is Locked" as a guarantee that stays true on its own. Periodically — at sprint boundaries is reasonable — spot-check real code in the repos against the Locked document the same way the PO did in each of these three cases: pick a standards rule that matters (an auth mechanism, an exception pattern, a UI convention) and confirm it's actually there, not just documented as intended. There is no command that does this for you continuously. That's a real gap, not a hidden safety net you haven't found yet.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s terminology map. New terms specific to this phase:

- **Actor Identity Shape** — the derived decision on whether audit columns (`createdBy`/`updatedBy`/`deletedBy`) are a foreign key into one user table or a self-contained actor-label string. Single actor identity space → FK is fine. More than one kind of actor (authenticated user, tokenized-link customer, scheduled job/webhook) → the label shape is required, because one FK cannot represent more than one target table. See the worked explanation above.
- **Deviation (Section 9)** — a difference between the architecture document and `standards/system-architecture.md`, the file the document is checked against. Only catches drift *after* the baseline exists — it structurally cannot flag a bad choice made in the same confirmation that produces the baseline.
- **Composable capability model** — the `capabilities: string[]` JWT shape, used instead of a single `role: string` when the BRD describes permissions as independently assignable rather than one fixed role per user. Detected automatically from BRD wording; confirm it against your own memory of the actual permission conversation.
- **Fix tiers (Tier 1–4)** — the same pattern as `/review-epics`, reused here: Tier 1 applies silently, Tier 2 confirms before applying, Tier 3 is a PO decision, Tier 4 halts the review (structurally incomplete coverage, multiple unresolved Open Decisions).

---

## Common mistakes

- Treating [`CLAUDE.md`](../CLAUDE.md)'s "add standards before running `/gen-architecture`" as a hard prerequisite and stalling on a blank template instead of just running the command.
- Reading Step 4's Actor Identity Shape line as background context because it's bundled with four other questions, instead of stopping on it deliberately.
- Assuming Section 9 will catch a bad tech-stack choice later — it can't, because the choice and the baseline it's checked against are confirmed in the same breath.
- Letting Concern D's composable-vs-fixed-role detection pass unchallenged because it's presented as already-settled fact rather than a decision point.
- Trying to review Section 4.2's Drizzle code or Section 5's DTO blocks line by line instead of trusting the structural checks and spending that attention on the framing narration and targeted questions.
- Accepting "no flags — all checks passed" as proof of correctness rather than as "nothing the structural checks are built to catch was found."
- Typing a bare `lock` out of habit when a one-line rationale costs nothing and gives you something to point back to later.
- Reading the Extension Points table's one-sentence summary as sufficient, instead of opening `layered-domain-architecture.md` and applying the Membership Test yourself to the least-obvious seams.
- Assuming the Locked architecture document stays true of the real codebase on its own, with no periodic manual check — three separate ADRs (`FW-033`, `FW-040`, `FW-041`) show this gap is real, not hypothetical.

---

## Harborview in practice

[`arch.example.md`](../projects/TEMPLATE/architecture/arch.example.md) is now a complete, current reference — regenerated against the live template, all 776 lines — and worth reading end to end once, not just consulting section by section:

- **`varchar` actor-label columns**, not a `uuid` foreign key, on every Master table (`payers`, `invoices`) — the concrete outcome of the Actor Identity Shape derivation walked through above, because Harborview has more than one actor kind: authenticated Staff, and `System` (the dunning scheduler, the Stripe webhook) acting on its own.
- **Section 3.1.3**, carried over verbatim from the approved `screen-design.md` — the Portals, Navigation Map, and Screen Inventory tables (SCR-001–SCR-005) are untouched from Screen Design; only 3.1.3.4 (Action Gating) and 3.1.3.5 (Cross-Cutting UI Rules) are generated fresh here, once Section 6's security model exists to generate them from.
- **Section 3.2**, one real Extension Point — `AccountingExportStrategy`, the seam for the Xero/QuickBooks export REQ-022 already named as a grounded-but-deferred capability back in Chapter 3 — a small, single-point instance of the Core/Extension methodology; see Case 2 in [`case-studies/harder-cases.md`](case-studies/harder-cases.md) for a fuller worked application.
- **Section 5's `If-Match` headers**, present on every mutating endpoint against the `version`-tracked `payers` and `invoices` tables — the concrete wire-level result of [`reference/api-contract/contract.md`](../reference/api-contract/contract.md)'s Rule 8, cited rather than restated, exactly the pattern `api-standards.md` itself demonstrates.
- **Section 6.5**, Logging & Exception Handling — a real, concrete summary (`X-Request-Id` correlation, `AllExceptionsFilter`, `BusinessRuleViolationException`, Pino/Axiom, Sentry) rather than the aspirational gap `FW-041` found on a real project; this section exists specifically so that gap gets caught here, at generation and review time.
- The completed **Verification** section at the bottom — every checkbox annotated with the specific table, section, or column it verified against, the same standard of specificity `/review-epics`' checks held Chapter 4's epic example to.

Alongside it, [`technical-standard.example.md`](../projects/TEMPLATE/standards/technical-standard.example.md) (`api-standards.md`) and [`architectural-pattern.example.md`](../projects/TEMPLATE/standards/architectural-pattern.example.md) (`idempotent-stripe-webhook-handling.md`) are the two files to read for what a hand-authored or generated standards file should actually look like — dense, cited against the fixed contract rather than restating it, and grounded in one real, complete worked path each.
