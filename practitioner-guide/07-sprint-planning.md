# Sprint Planning

Covers Stage 7 in [`SDLC.md`](../SDLC.md): [`/gen-stories`](../.claude/commands/gen-stories.md), [`/review-stories`](../.claude/commands/review-stories.md), and [`/gen-sprint-plan`](../.claude/commands/gen-sprint-plan.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't — this chapter owns the full depth of its Wave vs. Sprint vs. Epic row, which the orientation table only summarizes.

This chapter picks up where [`04-product-structuring-epics.md`](04-product-structuring-epics.md) left off. That chapter covered epic sizing, dependency ordering, and the story-level interleaving mechanism `/gen-epics` Step 6 now writes into `epics/index.md` Section 3 — this chapter uses that mechanism as a given, rather than re-explaining it. What happens after a story is Reviewed and sprint-assigned — `/gen-story-plan`, `/implement-story`, `/verify-story` — is the next chapter's territory, not this one's.

---

## Purpose

Stories are where the BRD's scope finally becomes something a developer can pick up without a follow-up question. `/gen-stories` turns one epic's In Scope bullets into a candidate story list; `/review-stories` is the rigorous, per-story audit that has to pass before any story enters a sprint; `/gen-sprint-plan` turns an already-made sprint assignment into a kickoff document. Three commands, three different jobs — and, as with epics, most of the risk in this phase is doing one command's job at the wrong point in the sequence, or trusting a number the framework hands you without a way to check it.

Two decisions in this phase are made once and then live with the project for its duration: which estimation model you use, and how disciplined you are about catching an oversized story before it's written rather than after. Everything else in this chapter is either a real test the framework doesn't spell out, or a piece of terminology worth getting exactly right before it costs you a broken sprint plan.

---

## Choosing an estimation model

`/gen-stories` Step 1.6 presents three options — T-Shirt Sizing, Fibonacci Story Points, Time-Based — as a menu with a one-line description each, and asks the PO to type a, b, or c. Read as a menu, all three look equally legitimate. Read the reference files themselves, and one of them almost always isn't.

### Time-Based is disqualified in the normal case

[`time-based.md`](../reference/estimation/time-based.md) states this about itself, not about the framework: *"Not suited for: Projects where the implementer is an AI assistant — AI implementation speed does not map linearly to human hours, making hour-based capacity planning unreliable."* Every project run through this framework has an AI implementer — that's the entire premise of `/implement-story`. So this isn't a caveat that applies to some projects and not others; it applies to all of them, every time, and it disqualifies the model for the normal case before you even get to comparing it against the other two.

**The one legitimate exception:** a client contract that requires hour-denominated billing or hour-based reporting regardless of whether the hours are accurate — Time-Based's own "Best suited for" line names this directly ("client billing by hour, government contracts... clients who require hour estimates per story for approval"). If you're in that situation, choose Time-Based, but hold the number the way the reference file itself tells you to: *"If this standard is selected for an AI-assisted project, treat hours as a complexity proxy, not a literal time prediction."* You're not measuring how long the AI will take — you're producing a number the client's process requires, sized consistently enough to be internally comparable. Say this to the client if the relationship allows it; don't let an hour estimate silently imply a promise about wall-clock delivery time that AI-assisted delivery doesn't actually make.

Outside that one exception, don't select Time-Based. Choosing it because it "sounds more precise" or because option (c) is habit from prior non-AI projects means every sprint capacity number downstream is planned against a unit the model's own author tells you doesn't hold.

### T-Shirt vs. Fibonacci: the real test

With Time-Based off the table in the normal case, the actual decision is T-Shirt vs. Fibonacci. Both reference files state their own fit conditions plainly — read them side by side rather than picking on vibes:

[`tshirt-sizing.md`](../reference/estimation/tshirt-sizing.md): *"Best suited for: AI-assisted delivery where implementation scope drives sizing more than human effort hours. The binding constraint is PO review throughput — how many PRs and change request cycles can happen in a sprint — not how fast the implementer writes code."*

[`fibonacci-points.md`](../reference/estimation/fibonacci-points.md): *"Best suited for: Projects where the team has an existing Scrum velocity baseline, or where the client requires story point estimates for reporting. Velocity-based capacity planning is more accurate when at least one sprint of historical data exists."* And, on the other side: *"Not suited for: First sprints with no velocity history (capacity planning degrades to guesswork), or projects where the PO has no prior Scrum experience."*

Distilled into a test you can actually apply at Step 1.6:

> **Does this specific engagement already have a real Scrum velocity baseline — from a prior phase with this same client and team, or a client mandate that requires point-based reporting for their own governance?** If yes, Fibonacci is viable. If no — which is the default state of every new Ascendra project, since Fibonacci's own file admits first-sprint velocity is guesswork — choose T-Shirt. It's the framework's actual recommended default for a reason that isn't cosmetic: its capacity model (PO review throughput — PRs reviewed and change-request cycles per sprint) is the constraint that genuinely governs an AI-implemented, single-PO-reviewed project. Fibonacci's capacity model (velocity from completed sprints) measures a quantity this framework's delivery shape doesn't produce until several sprints in.

One more thing worth being honest about: even when Fibonacci is the right call because of an existing baseline, that baseline was almost certainly built on a different team/tool/domain combination than this specific AI-assisted engagement. Expect the first sprint or two of *this* project to be noisy even with "real" velocity history — a baseline transfers imperfectly across a change in implementer.

---

## The wave-vs-epic boundary, using the real mechanism

A **wave** is the unit `/gen-stories` generates against in one run — normally one epic. `/gen-stories` reads exactly one epic file (Step 1), plans stories against exactly that epic's In Scope bullets (Step 4), and reports on exactly that epic's story set (Step 8). One command run, one wave, one epic — that's the default and it holds almost all the time.

It's legitimately more than one epic only when the pair carries a documented **story-level interleaving note**. [`04-product-structuring-epics.md`](04-product-structuring-epics.md#the-story-level-interleaving-note) already covers where this note comes from and the trigger test `/gen-epics` Step 6 applies to write it — this chapter doesn't re-derive that mechanism, it uses it. What matters here is what happens on the sprint-planning side once that note exists in `epics/index.md` Section 3:

`/gen-stories` Step 2 reads that section as part of its required inputs, specifically to decide what to print at Step 8. If the epic you just generated stories for carries an interleaving note with another epic, the report doesn't say "run `/review-stories` now" — it says *"Generate stories for its paired epic now... their cross-epic dependencies mean they must be reviewed together."* If it doesn't, it says review now. Two branches, and until recently a PO had nothing to check the branch against except trusting the AI read the note correctly. Now you do.

**What to actually check at Step 8, before following either instruction:** open `epics/index.md` Section 3 yourself and read the interleaving note for the pair. Confirm it names one *specific* story in the depended-on epic — not the whole epic — that has to land before one *specific* story in the epic you just generated. If the note reads that way, "generate the paired epic first" is doing its job: reviewing the pair together lets `/review-stories`' Dependency Chain Coherence check (epic-level check #4) verify that specific story-to-story link once both files exist. If the note is vague — names the whole epic, or reads like a generic "these are related" gesture rather than a named story pair — that's the same failure mode Chapter 4 taught you to catch at the source: don't let a soft interleaving note push you into generating and reviewing two epics together when the real dependency resolves cleanly at the epic level and one wave, reviewed alone, would have been correct. Push back and ask for the note to be sharpened before you commit to the paired-generation path.

---

## Sprint Assignment and the split-boundary heuristic

`/review-stories` Part F is the single best-designed mechanism in this phase, and it's worth being explicit about why, because the parts worth trusting and the one part that still needs your judgment look similar on the page.

**Why it's well-designed:**
- **Sourced capacity, not a guess.** The 8–10 stories/sprint target isn't invented in `review-stories.md` — it's pulled from `projects/{PROJECT_CODE}/standards/estimation.md`, the same file you set at `/gen-stories` Step 1.6. The number you're planning against is traceable to a decision you already made and can re-check.
- **Honest handling of leftover stories.** The "below target" branch doesn't silently round a light wave up to a full sprint or leave it stranded — it surfaces the choice explicitly (fold in the next unblocked epic, or accept a lighter sprint) and, if you accept the lighter sprint, it separately asks whether any earlier-epic leftover stories from a prior split should top it up. Nothing gets quietly lost or quietly padded.
- **Three clear branches.** At/near target, below target, above target — each has one deterministic action, not a vague "use judgment" instruction.

**The one gap: the "above target" branch gives you the decision but not the tool.** When an epic alone exceeds capacity, Part F's exact prompt is: *"Where should the split fall? Stories before the boundary go to Sprint {NN}; the remainder carries forward as the start of Sprint {NN+1}."* That's a real question with no worked answer anywhere in `review-stories.md` — and the naive answer, "put the first 8–10 in this table into the sprint, carry the rest," is exactly the arbitrary story-count cutoff the estimation references themselves warn against.

The fix is already sitting in the estimation standard you picked at Step 1.6 — [`tshirt-sizing.md`](../reference/estimation/tshirt-sizing.md)'s own Split Triggers section gives the rule for splitting a story that's too large, and the same logic scales up cleanly to splitting a wave that's too large:

> **How to split:** Find the seam — usually a layer boundary (API vs. UI) or a distinct user action. Each child [half] must be independently implementable, independently testable, and carry its own [stories] with no overlap.

Applied to a wave split, not a single story: **the split should fall where the epic's own intra-epic dependency order (Section 6 of each story) already shows a natural seam — a point where everything before it is one coherent, demoable user action and everything after it is a distinct next action — not at whatever story happens to be eighth or tenth in the planning table.**

A worked hypothetical (Invoice Management's real REQs, a story count inflated past what the real epic actually produces, to give the split something to bite on): suppose EPIC-003 Invoice Management's story breakdown, once every AC was written, came to thirteen stories rather than the roughly seven the real epic produces — schema, create-form validation, line-item table behavior, due-date logic, subtotal/VAT calculation, draft save, submit-for-approval trigger, reference number generation, PDF generation, PDF template customization, void action, invoice list view, invoice detail page. That's well above the 8–10 target on its own.

The wrong split: sprint the first ten rows of the planning table, carry the remaining three. That cutoff has no relationship to what's actually buildable and demoable on its own — it just happens to be where the table ran out of room.

The right split, using the seam heuristic: everything needed to **create and persist a valid draft invoice** — schema, create-form validation, line-item behavior, due-date logic, subtotal/VAT calculation, reference number generation, draft save — is one coherent, independently demoable user action: a Finance Manager creates an invoice and it exists with a correct reference number and correct totals. Everything after that — submit-for-approval trigger, PDF generation and template customization, void, list view, detail page — is a distinct next action (or several) that assumes a persisted invoice already exists. Split there. Sprint N gets "create and persist an invoice," Sprint N+1 picks up "act on an existing invoice" as its own coherent scene, not a leftover fragment.

---

## Walking Skeleton Independence: the check that actually enforces FW-048

Every phase before this one has already done something with the Walking Skeleton: the BRD tagged the requirements that compose it (Chapter 3), epics carried the tag forward and got sequenced first (Chapter 4). None of that, on its own, guarantees anything real. A REQ can be correctly tagged, land in the right epic, and still produce a story that quietly depends on a story nobody tagged — and the moment that happens, the "thin, real, end-to-end path" stops being either thin or independently provable, which is the exact failure this whole convention exists to prevent.

`/review-stories` check #7 (Walking Skeleton Independence) is where this actually gets caught, and it's a narrow, mechanical check, not a judgment call: **for every `Walking Skeleton: Yes` story, every dependency listed in its own Section 6 — intra-epic or cross-epic — must itself be `Yes`-tagged.** Not "probably fine," not "practically equivalent" — a single non-tagged dependency on a Walking Skeleton story is a finding, full stop, because it means the path you're about to build first secretly isn't buildable first.

**Where you'll actually see this bite:** a Walking Skeleton story that needs *some* supporting capability — an audit log entry, a notification, a lookup table — where the natural story to depend on wasn't itself part of the tagged chain. The fix is never to quietly accept the dependency; it's to ask whether that supporting capability genuinely belongs in the skeleton (tag it too, if leaving it out would break the chain — Chapter 3's BRD-level test, "does the journey still run end-to-end without it," applies exactly the same way here) or whether the Walking Skeleton story's own scope was drawn too wide and needs to shed the piece that's pulling in an untagged dependency.

**`/review-stories` Part F also carries a lighter, upstream half of this discipline**: Walking Skeleton ordering into sprints isn't a separate decision Part F makes — it's inherited automatically from the epic build order Chapter 4 covers, as long as epics are worked through `/gen-stories`/`/review-stories` in that order. The one thing worth watching for yourself: a PO request to jump ahead and review a later epic out of sequence is exactly the kind of thing that can quietly displace Walking Skeleton stories from the sprint that's supposed to prove them first — Part F now flags this rather than silently following it, but it's still worth noticing when you're the one asking for the jump.

---

## Story sizing: proactive, not just reactive

The story template's sizing rules themselves are not the gap — the AC-count proxy (3–10 ACs, XL must split) is clear and it's checked consistently: at generation (`gen-stories.md` Step 5), at review (`review-stories.md` check #8, Size Validity), and again at the sprint gate (`gen-sprint-plan.md` Step 5, L-story split decision). Where every one of those checks sits is *after* the story breakdown from `/gen-stories` Step 4 is already fixed and files already exist. A story that's obviously going to come out L or XL gets caught at `/review-stories`, once it's a real file with real ACs written against a scope boundary nobody questioned at the point where questioning it was cheapest.

**The fix, applied at Step 4 itself — before any file is generated:** the planning table at that step already gives you Story ID, Title, the In Scope bullet it maps to, an *initial* size estimate, and Notes. You don't have ACs yet to run the formal AC-count proxy, but you don't need them to catch the obvious cases. Read each candidate row the way `tshirt-sizing.md`'s own Factor 2 (Pattern Novelty) and Factor 3 (Context Footprint) ask you to read a story: does this bullet describe a brand-new pattern for this codebase (first file upload, first background job, first multi-step state machine)? Does it plausibly touch more than half a dozen existing files or concepts to implement? Does the bullet itself read as two or three things joined by "and" — the same over-fat tell Chapter 3 taught you to catch in a REQ, one level down?

**The concrete rule to apply at the table:** if more than two stories in the Step 4 planning table already look L or XL on this rough read, propose splitting them right there — before generating a single file. Bring the same seam heuristic from the previous section to the table: a bullet whose obvious split point is a layer boundary or a distinct sub-action should become two candidate rows in the table, not one row you generate and hope stays under ten ACs. This costs you nothing but a slightly longer planning conversation. Catching the same problem at `/review-stories` costs a Tier 3 fix — PO states the split boundary, the AI creates two stories and removes the original, the epic file and stories index both need updating — on a story that already has ACs, an Out of Scope section, and dependencies written against the wrong boundary.

---

## Terminology in full depth: wave vs. sprint vs. epic

`00-orientation.md` gives you the one-paragraph version of this distinction. Here's the full depth, because getting it wrong costs you a sprint plan built on a false assumption.

**Epic** is a fixed unit of scope, decided once, at `/gen-epics` time, and locked at `/review-epics` approval. It doesn't move after that.

**Wave** is a unit of *story generation*, decided at `/gen-stories` time — normally one epic, occasionally an interleaved pair (covered above). A wave is decided epic-by-epic (or interleaved-pair-by-pair), one `/gen-stories` run at a time, as you work down the dependency order.

**Sprint** is a unit of *delivery cadence*, decided separately and later, at `/review-stories` Part F — purely from real story count vs. the capacity target in `standards/estimation.md`. Sprint boundaries are not known, and cannot be known, until stories actually exist with real sizes, because the whole mechanism is "count what you actually have against a capacity number," not "estimate ahead of time how many sprints an epic will take."

**The load-bearing fact: a wave and a sprint are not the same size and are not decided at the same time.** A wave is sized by what one epic's In Scope bullets turn into — that's a product-structuring fact, fixed since epics were approved. A sprint is sized by real capacity — that's a delivery-cadence fact, only knowable once stories exist. Nothing forces these two numbers to align, and in a real project they usually won't.

**A worked timeline showing exactly where they diverge**, continuing the split hypothetical from the previous section:

| Step | What happens | Wave | Sprint |
|---|---|---|---|
| 1 | `/gen-stories` on EPIC-003 (Invoice Management) — produces 6 stories | Wave A = EPIC-003, 6 stories | — |
| 2 | `/review-stories` reviews EPIC-003. Running total (6) is below the 8–10 target. PO folds in EPIC-004 (Director Approval, unblocked, next in dependency order) rather than accept a light sprint | — | — |
| 3 | `/gen-stories` on EPIC-004 — produces 4 stories | Wave B = EPIC-004, 4 stories | — |
| 4 | `/review-stories` resumes at EPIC-004. Running total is now 6 + 4 = 10 — at target | Waves A and B are now both consumed | Sprint 01 = 10 stories, spanning **two waves** (EPIC-003 and EPIC-004) |
| 5 | `/gen-stories` on the hypothetical 13-story EPIC-005 hypothetical from the previous section | Wave C = EPIC-005, 13 stories | — |
| 6 | `/review-stories` reviews it — above target on its own. PO gives the "create and persist" split boundary from above: 7 stories before it, 6 after | Wave C is one wave, but its stories split across two sprints | Sprint 02 = the first 7 stories of Wave C. The remaining 6 are left `TBD` |
| 7 | `/gen-stories` on EPIC-006 (Overdue Management) — produces 4 stories | Wave D = EPIC-006, 4 stories | — |
| 8 | `/review-stories` reviews EPIC-006. Running total for this fresh review pass is 4 — below target. PO is asked about the 6 leftover, already-Reviewed stories from Wave C's split, and agrees to fold them in | — | Sprint 03 = 6 leftover stories from **Wave C** + 4 fresh stories from **Wave D** — one sprint absorbing the tail of an earlier wave plus a whole new one |

Read that table twice and the two divergent cases the framework can genuinely produce both show up in it: Sprint 01 shows **one sprint absorbing two full waves** because neither was near capacity alone; Sprint 02/03 shows **one wave splitting across two sprints** because it was too large alone, and Sprint 03 shows the second half of that pattern — **a sprint made of a wave's leftover plus a completely different fresh wave**. None of this is a special case the framework handles awkwardly — it's Part F's three branches (at/near target, below target, above target) doing exactly what they're designed to do, run repeatedly across a real project's actual epic sizes. The only discipline required of you is not assuming, walking into `/review-stories`, that "this epic" and "this sprint" mean the same thing — they don't, until Part F actually says so.

---

## The tie-break rule, worked: two personas, one capability

`story.template.md`'s Section 1 AI Guide states a real rule that none of the three worked examples in `story.example.md` ever exercises: *"If two personas need the same capability, check whether the permission model actually distinguishes them — if not, pick the lower-privilege role."* All three real examples (reference number generation, the create-invoice form, the Stripe webhook) have exactly one persona each with no ambiguity about who else might plausibly want the same thing — so there's nothing in the framework's own files to pattern-match against when you actually hit this case. Here's a hypothetical grounded in Harborview's real roles, clearly labeled as invented for this purpose rather than drawn from the example files.

Harborview's BRD (Section 3, User Roles) defines two roles relevant here:

- **ROL-001 Finance Manager** — can create, edit, void invoices; view all invoices and payment status; full data visibility including Payer contact details.
- **ROL-002 Director** — can approve or reject invoices, view full invoice details before acting, view all invoices (read-only); cannot create or edit; Payer contact details explicitly **not** visible to this role.

**Hypothetical story: "View an approved invoice's detail" — reference number, line items, totals, approval decision, and approver name, with no Payer contact information anywhere on the screen.** Both roles genuinely need this: a Director needs it to confirm what they approved without asking the Finance Manager to pull it up; a Finance Manager needs it to confirm an invoice's approval status before sending. Apply the rule:

1. **Do both personas need the same capability?** Yes — the same screen, the same fields, for both roles.
2. **Does the permission model actually distinguish them on this specific capability?** Check the Data Visibility lines, don't assume. Director's restriction ("Payer contact details not visible") only bites if the screen shows Payer contact details — and this hypothetical's AC scope deliberately excludes them (reference number, line items, totals, approval decision, approver — no phone, no email). Scoped that way, nothing in ROL-001 or ROL-002's stated permissions differentiates who can see this particular screen. The permission model does not distinguish them here.
3. **Pick the lower-privilege role.** Between the two, Director is the narrower role overall — view and approve/reject only, with no create, edit, void, or Payer-management authority anywhere in its "Can" list. Finance Manager's overall permission set is broader. So the story statement is written from the Director's side: *"As a Director, I want to view an approved invoice's line items, total, and approval decision, so that I can confirm what I approved without asking the Finance Manager to pull up the record."*

**Why this is the correct call, not an arbitrary one:** writing the story for the broader role (Finance Manager) risks a developer implicitly gating the screen to Finance-Manager-only access, since that's the persona named in the ACs — silently locking the Director out of a capability they equally need. Writing it for the narrower role forces the acceptance criteria to be built against the more restrictive permission set from the start; a Finance Manager, whose permissions are a superset for everything this screen touches, is never the one who ends up implicitly excluded. If a later story needs to add something Director genuinely can't see (say, a line-item cost breakdown Finance Manager alone should view), that's a signal the capability has actually diverged and deserves a second story — not a reason to abandon the shared one.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s Wave vs. Sprint vs. Epic row — this chapter is the "full depth" that row points to, not a duplicate of it. Two things specific to this phase, not in the orientation table:

- **Story-level interleaving note** — defined and worked through in [`04-product-structuring-epics.md`](04-product-structuring-epics.md#the-story-level-interleaving-note); this chapter shows what to check with it at `/gen-stories` Step 8, not what it is.
- **Sprint Assignment (`/review-stories` Part F)** — the three-branch mechanism (at/near target, below target, above target) that actually decides sprint boundaries. Nothing upstream of Part F commits to a sprint number — not the epic, not the wave, not `stories/index.md`'s `TBD` placeholder.

---

## Common mistakes

- Selecting Time-Based because it "sounds more rigorous," without checking that its own reference file disqualifies it for every AI-implemented project except the hour-billing exception.
- Choosing Fibonacci on a brand-new engagement with no velocity history, then being surprised when first-sprint capacity planning is guesswork — the reference file says this will happen.
- Treating `/gen-stories` Step 8's "generate the paired epic first" instruction as self-justifying, without opening `epics/index.md` Section 3 to confirm the interleaving note names a specific story pair rather than a vague "these are related" gesture.
- Assuming one epic = one sprint, and being confused when Part F either combines two waves into one sprint or splits one wave across two.
- Waiting until `/review-stories` flags an L or XL story instead of scanning the Step 4 planning table for more than two rows that already look oversized, before any file exists.
- Splitting an over-capacity wave at whatever story happens to land eighth or tenth in the table, instead of finding the layer-boundary or distinct-user-action seam the estimation standard's own Split Triggers guidance describes.
- Writing a shared-capability story for the higher-privilege persona by default, without checking whether the permission model actually distinguishes the two roles on that specific capability.
- Accepting a Walking Skeleton story's dependency on a non-tagged story as "practically fine" instead of treating it as the finding it is (FW-048) — either tag the dependency in too, or narrow the story's scope.

---

## Harborview in practice

[`story.example.md`](../projects/TEMPLATE/stories/story.example.md)'s three worked stories are the model for density and structure in this phase, each demonstrating a different story shape:

- **US-01-005 (Generate invoice reference number)** — a backend logic story, Size S, with a Notes section (Section 7) carrying a real non-obvious constraint (single global database sequence, transaction-wrapped, gaps acceptable but year-boundary resets are not) that has no business being in an acceptance criterion.
- **US-01-002 (Create invoice form)** — a UI story, Size M, showing Dependencies in Pass 2 (resolved story ID) format and an Out of Scope section that correctly names the covering story for every excluded capability rather than gesturing at "handled later."
- **US-01-012 (Stripe webhook payment confirmation)** — an integration story, Size M, whose ACs cover all four required categories from `gen-stories.md` Step 5 in one story: happy path (AC-3), boundary/rejection (AC-4, partial payment), enforcement/idempotency (AC-6, duplicate event), and audit (AC-5, the audit log entry) — worth reading end to end as the reference instance of AC category coverage.

All three of the stories above are, not by coincidence, `Walking Skeleton: Yes` (FW-048) — they satisfy REQ-008, REQ-006, and REQ-014/REQ-015 respectively, exactly the requirements `brd.example.md` Section 5 tags as part of Harborview's Walking Skeleton (Journey 4.4: register a Payer, create and price an invoice, send it, collect real payment). Their own Section 6 (Dependencies) is a real illustration of what check #7 actually looks at: US-01-012 (payment confirmation) depends on US-01-001 (invoice schema) and US-01-008 (invoice sending) — both are, by their own descriptions, load-bearing pieces of the same Journey 4.4 chain (nothing about "sending an invoice" or "the invoice table existing" is breadth outside the skeleton), which is exactly what a passing instance of Walking Skeleton Independence looks like: every dependency a tagged story leans on is itself part of the same thin path, not a capability borrowed from outside it. Neither US-01-001 nor US-01-008 is reproduced as a full worked file in `story.example.md`, so treat this as the shape a clean pass takes, not as a claim about tags recorded in files this repo doesn't show.

[`sprint-plan.example.md`](../projects/TEMPLATE/sprints/sprint-plan.example.md) is worth reading for its Story Board and Delivery Sequence shape — six stories sequenced by technical dependency (schema before form, reference number generation before the detail page that displays it) rather than by the order they appear in the stories index — and for its Sprint Notes entry, a real example of the kind of pre-sprint operational note (seed data needed before a story is reviewable) that belongs in Section 9, not invented as an acceptance criterion.
