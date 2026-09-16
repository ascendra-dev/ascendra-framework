# Product Structuring — Epics

Covers Stage 4 (Product Structuring) in [`SDLC.md`](../SDLC.md): [`/gen-epics`](../.claude/commands/gen-epics.md) and [`/review-epics`](../.claude/commands/review-epics.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't — this chapter assumes you already have its terminology table (Scope vs. Priority vs. Layer) in hand.

This chapter picks up exactly where [`03-brd-discovery-and-scope-lock.md`](03-brd-discovery-and-scope-lock.md) left off. That chapter established that each BRD Section 5.N feature-area heading becomes an epic boundary, and that you're already drawing epic boundaries the moment you draft Section 5 headings. This chapter is about what happens once the AI actually turns those headings into epic files: where the 1:1 mapping holds, where it correctly doesn't, and the judgment calls in between.

---

## Purpose

Epics are the layer between the BRD and stories. `/gen-stories` reads an epic's Section 3.1 (In Scope) and Section 8 (Notes) to derive the story list for that epic — nothing about how a story is broken down comes from anywhere else. That's what makes epic quality load-bearing: an epic that bundles two unrelated capabilities together produces one bloated, undemoable wave of stories; an epic with a fuzzy Out of Scope boundary produces stories that quietly duplicate work another epic already owns; a wrong dependency order produces a sprint plan that schedules an epic before the capability it actually needs exists.

Epics carry no architecture dependency ([`decisions/FW-022-problem-solution-domain-boundary.md`](../decisions/FW-022-problem-solution-domain-boundary.md)) — that's deliberate, it's why product structuring can happen for the whole approved BRD at once, before any technology decision is made. But "no architecture dependency" is a promise about what an epic is *allowed* to assume, not a guarantee that every sentence an AI writes into Section 8 will honor it. More on that below.

Two commands, two different jobs: `/gen-epics` proposes the breakdown and generates the files; `/review-epics` is the rigorous per-REQ audit that has to pass before any epic reaches `Approved`. Knowing which one is supposed to catch which kind of mistake is most of what this chapter teaches.

---

## Epic sizing and boundaries: one capability, or two glued together?

[`epic.template.md`](../projects/TEMPLATE/epics/epic.template.md)'s Document Level AI Guide states the rule of thumb plainly: **an epic typically decomposes into roughly 4–10 stories.** That number by itself isn't the useful part — it's a symptom, not a cause. The actual test, the one worth applying every time the AI presents a candidate epic (at `/gen-epics` Step 4's planning table, or later at `/review-epics` check #12), is the same cohesion test Chapter 3 used for REQ density, one level up:

> Could this feature area ship and be demoed as a complete, independently useful capability on its own — one scenario, one Definition-of-Done walkthrough, no "and separately, at a different time, a different person does X"?

If yes, it's one epic no matter how many REQs it contains. If you find yourself writing a Section 6 walkthrough that has to switch personas or switch triggers partway through, that's the tell that two capabilities got welded together for the convenience of one BRD heading.

### Harborview's epic set, and the one place the 1:1 mapping correctly doesn't hold

Four of Harborview's BRD feature areas ([`brd.example.md`](../projects/TEMPLATE/brds/brd.example.md)) map cleanly 1:1 to a single epic, exactly as Chapter 3 described:

| BRD section | REQs | Epic |
|---|---|---|
| 5.1 Payer Management | REQ-001–005 (5) | EPIC-001 Payer Management |
| 5.3 Payment Collection | REQ-014–016 (3) | EPIC-005 Payment Collection |
| 5.4 Overdue Management | REQ-017–019 (3) | EPIC-006 Overdue Management |
| 5.5 Dashboards | REQ-020–021 (2) | EPIC-007 Dashboard and Reporting |

But Section 5.2 (Invoice Management) is the interesting one, and it's the case worth actually studying. It holds eight requirements — REQ-006 through REQ-013 — under one BRD heading. Read what [`epic.example.md`](../projects/TEMPLATE/epics/epic.example.md)'s own Section 3.2 (Out of Scope) and Section 5 (Dependencies) reveal: Harborview's real epic set treats this heading as **two** epics, not one — EPIC-003 Invoice Management (REQ-006, 007, 008, 009, 013 — creation, automatic totals, reference numbering, PDF generation, void) and EPIC-004 Director Approval (REQ-010, 011, 012 — submission above threshold, Director notification and decision, rejection-and-resubmit). *(The epic numbering visible in the example jumps from EPIC-001 to EPIC-003 — nothing in the REQ coverage map depends on an EPIC-002 existing, since REQ-001 through REQ-021 are fully accounted for across the six epics named here; treat the gap as a drafting artifact, not something to chase.)*

Apply the cohesion test to see why the split is correct, not arbitrary:

- **Different trigger.** Invoice Management is triggered by the Finance Manager choosing to create and send an invoice. Director Approval is triggered by an invoice crossing a value threshold — an event, not a user choosing to start a workflow.
- **Different persona doing the acting.** Finance Manager acts in one; a Director acts in the other. Neither persona's actions in their half depend on the other's Section 3.1 bullets to be meaningful.
- **Independently demoable.** You can build and demo "create, calculate, number, PDF, void an invoice" end to end with the approval threshold set so high nothing ever crosses it — the Director never touches the system. You can equally build and demo "submit → notify all three Directors → any one approves or rejects → rejection reason recorded → Finance Manager resubmits" against a stubbed invoice that already exists. Neither half needs the other's full Definition of Done to be a complete, watchable scenario.
- **Size.** Eight REQs on one heading is already outside the comfortable end of "4–10 stories" once you account for REQ-007's three calculated fields and REQ-010–012's three-step approval state machine each plausibly costing more than one story — exactly the signal Step 4 item 5 tells you to catch before any file is generated, not after.

If 5.2 had **not** been split — imagine a hypothetical `EPIC-BAD: Invoice Management & Approval` carrying all eight REQs — the tell would show up immediately in Section 6: the Definition of Done walkthrough would have to read "Finance Manager creates and sends an invoice... **and separately**, a Director later approves a different invoice... **and separately**, the Finance Manager resubmits a rejected one." That "and separately" is the same signal Chapter 3 taught you to catch in an over-fat REQ, just one layer up.

**What to actually do at the planning table:** for each candidate epic, ask yourself the two questions above — one trigger/one persona-cluster, and would the Section 6 walkthrough read as one continuous scene. Don't count REQs and stop there; REQ count is a proxy that catches size outliers, not boundary correctness. A four-REQ epic can still be two capabilities glued together, and an eight-REQ epic can still be genuinely one (Harborview's own REQ-002's dedup logic — three sentences, one capability — is proof density alone isn't the test).

---

## The two-pass review model: don't do `/review-epics`'s job at Step 4

`/gen-epics` Step 4 gives you a planning table — Epic ID, proposed title, REQ IDs, dependencies, notes — and one prompt: *"Does this epic breakdown look right?"* `/review-epics` gives you 13 structural checks per epic, run one epic at a time, against the actual generated file, the BRD, and the domain knowledge document all loaded in full. These are not the same kind of pass, and treating them as the same kind of pass wastes your attention at exactly the point where you have the least material to work with.

At Step 4, you do not yet have epic files. You have REQ IDs and titles in a table — no Section 3.1 In Scope bullets, no Section 8 Notes, no verbatim REQ text sitting next to the epic that's supposed to cover it. There is no way to rigorously cross-check REQ-by-REQ accuracy at this point even if you tried, because the thing you'd be checking against doesn't exist yet.

**What Step 4 is actually for — spend your attention here:**
- Do the groupings feel like real, independently-shippable capabilities? (the cohesion test above)
- Does the dependency order look right at a glance — is the epic everything else needs listed first?
- Is there a REQ you personally remember from discovery that looks obviously misfiled into the wrong feature area?

**What to explicitly *not* do at Step 4:** don't open the BRD side by side and verify each REQ ID landed in the row you expect, don't check whether In Scope wording will be specific enough to derive a story title, don't verify Section 8 will carry every applicable domain rule. None of that surface exists yet to check. `/review-epics`' checks #1 (REQ Coverage), #2 (Verbatim Accuracy), #4 (Story Granularity), and #8 (Domain Rule Coverage) do exactly this, exhaustively, epic by epic, once real files exist. Doing it twice doesn't make it more correct — it just means you spent real time at Step 4 on a check you're going to redo properly a few minutes later with better material.

---

## Dependency ordering: the falsifiable test

`/gen-epics` Step 4 asks you to "draft a dependency ordering" with no test for what actually makes Epic B depend on Epic A. Left ungrounded, this collapses into vibes — "Payer Management feels foundational" — which is true here, but "feels foundational" isn't a test you can apply to a case where it *isn't* obviously true. Use this instead:

> **Epic B depends on Epic A only if a story in B cannot be implemented or meaningfully demoed without a capability A delivers — not merely "A is conceptually foundational."**

Walk it through on Harborview, where it's supposed to hold, and then on a case where the easy answer is actually wrong.

**EPIC-001 Payer Management as the dependency root — the test, not vibes.** `epic.example.md` Section 5 states it plainly: *"Payer Management is the foundational epic — no other epic needs to be complete before it can begin. All invoice epics depend on this one."* That's not an assertion you have to take on faith — apply the test yourself against REQ-006: *"The system must allow the Finance Manager to create an invoice by selecting a Payer..."* You cannot implement "select a Payer" as a working screen, and you cannot demo it to a client, if no Payer exists to select. A story in EPIC-003 (Invoice Management) is structurally blocked without a specific capability EPIC-001 delivers — "register a Payer" — landing first. That's a passing instance of the test, not an assumption.

**Where the test earns its keep — catching a dependency that only *sounds* right.** Someone reviewing this epic set might reasonably propose "EPIC-007 Dashboard and Reporting depends on EPIC-001 Payer Management — after all, the dashboard shows Payer-related figures." Apply the test before accepting it: REQ-020's dashboard shows *"total outstanding balance, total overdue balance, invoices due within the next 7 days, and recently paid invoices."* None of those figures come from a raw Payer record — they come from Invoice, Payment, and Overdue state. You cannot demo REQ-020 with only EPIC-001 built; you need EPIC-003 (invoices to be outstanding), EPIC-005 (payments to be recorded), and EPIC-006 (overdue state to exist). Payer Management is a real ancestor of Dashboards, but only *transitively*, through those three epics — it is not a *direct* dependency, because no story in Dashboards is blocked on a Payer Management capability specifically. Listing "EPIC-001" directly in EPIC-007's Section 5 because it "feels foundational" would pass an eyeball check and fail the test — the honest Section 5 for Dashboards names EPIC-003, EPIC-005, and EPIC-006 directly, not EPIC-001.

**The practical move:** every time an epic's Section 5 names a dependency, ask "which specific story in this epic actually can't be built or demoed without it?" If you can point to that story, the dependency is real and direct. If the honest answer is "well, it all connects back eventually," that's a transitive ancestor, not a Section 5 entry — naming it directly just pads the dependency graph with an edge nothing in the epic actually needs.

---

## The Walking Skeleton: a harder constraint than ordinary dependency ordering (FW-048)

Dependency ordering (above) answers "what must exist before this epic can begin." The Walking Skeleton answers a related but stricter question: "which epic proves the whole system actually works, end to end, before anything else gets built to full breadth." The two can point the same direction — often the Walking Skeleton epic *is* the dependency root — but they're not the same test, and conflating them is exactly how the failure this convention exists to prevent happens.

The failure is real, not hypothetical: a prior Ascendra project shipped its first attempt without one. Its own retrospective names the root cause plainly — *"the core of the system was not established first. Rather than delivering the product core first and adding capability in phases, breadth was built out across many epics before a single thin end-to-end path was proven."* Everything downstream of that one decision got worse for it: stories couldn't be built or verified independently, because nothing had ever proven the thin path actually held together; authorization and notifications both got built out to a depth nobody had confirmed was needed yet, because there was no working core to test the *actual* need against.

**What to check, concretely:** by the time epics reach you, the BRD already carries a `Walking Skeleton: Yes` tag on exactly the requirements that compose one complete, thin, real, end-to-end path (Section 5 — see Chapter 3 for how it got there). Your job at this phase is narrower than re-deciding what the skeleton is — it's making sure the epic set doesn't quietly lose the thread:

- Every `Yes`-tagged REQ landed in some epic — it cannot fall through Step 3's REQ-to-epic assignment.
- Whichever epic(s) carry those REQs are sequenced first in the build order — ahead of any epic that carries none of them, even one that would otherwise sort earlier by an ordinary dependency read.
- If a Walking Skeleton REQ ends up split across two epics, that's a signal worth pausing on: a REQ set that was deliberately kept to "one complete chain, no gaps" at BRD time splitting across epic boundaries risks the fragmentation the whole convention exists to prevent. It isn't automatically wrong — sometimes a chain genuinely crosses a natural epic boundary (e.g. Payer Management's registration REQ feeding an Invoice Management epic's creation REQ, as it does on Harborview below) — but it's exactly the kind of split `/review-epics`' Walking Skeleton Build Order check is there to make you look at directly, not wave through by habit.

**The real enforcement doesn't finish here.** Epics only assign REQs to a container; stories are what actually get built and merged independently. The harder, sharper check — do the Walking Skeleton *stories* actually avoid depending on any non-Walking-Skeleton story — belongs to `/review-stories` (Chapter 7 covers it). What you're protecting at this phase is upstream of that: making sure the epic set hands stories a Walking Skeleton that's still intact, not one that's already been diluted by REQ-to-epic assignment decisions made for other reasons.

---

## The story-level interleaving note

Section 5 dependencies are epic-level: "EPIC-B needs all of EPIC-A's Definition of Done done first." Sometimes that's too coarse — the *true* build order is finer-grained than epic-before-epic: one specific story in A has to land before one specific story in B, but the rest of A's stories can build in parallel with the rest of B's. `/gen-epics` Step 6 now has a real trigger test for exactly this case, written into the epics index rather than left to guesswork:

> Can you name one *specific* story in the depended-on epic — not the whole epic — that must land before a specific story in the dependent epic can be built or meaningfully demoed? If yes, and the dependent epic doesn't need the rest of the depended-on epic's Definition of Done first, the pair's true build order is finer than epic-before-epic.

**Where you'll see it:** `projects/{PROJECT_CODE}/epics/index.md`, Section 3 (Dependency Graph notes), directly beneath the build-order list, as a `Story-Level Interleaving` subsection. `/gen-epics` writes it for every dependency pair that passes the trigger — and writes it even when nothing passes, as `"None — every dependency in this set resolves at the epic level."` `/gen-stories` and `/review-stories` both read this subsection later to decide whether one wave spans a single epic or an interleaved pair — this is the mechanism Chapter 0's Wave vs. Sprint vs. Epic table pointed forward to.

**Harborview doesn't exercise this** — every dependency in its epic set resolves cleanly at the epic level (EPIC-003 genuinely needs all of EPIC-001's Definition of Done, not just one story from it, before invoice creation is meaningfully demoable — a Finance Manager needs the full Payer registration *and* dedup-check flow working, not half of it). So its `index.md` Section 3 would read the "None" case. Here's a brief, clearly-labeled hypothetical that does exercise it — **not** drawn from Harborview:

> **Hypothetical: Staff Invite / Staff Accept-Invite.** `EPIC-STAFF-INVITE` lets an Org Admin invite a new staff member by email. `EPIC-STAFF-ACCEPT-INVITE` lets the invited person open the emailed link and set a password to activate their account. At the epic level, Accept-Invite obviously depends on all of Staff Invite — you'd naturally say "build the whole invite feature first." But the *true* first blocker is finer: Accept-Invite's very first story — "staff member opens the invite link and lands on a set-password screen" — only needs **one** specific Staff Invite story to exist first: "system generates and emails a unique, time-limited invite link." It does **not** need the rest of Staff Invite's Definition of Done — "Org Admin can revoke a pending invite," "Org Admin can resend an expired invite," "Org Admin sees an invite status list" — to be built before Accept-Invite's first story can be built or demoed. Scheduling these as two fully sequential waves would needlessly block Accept-Invite's first story behind capabilities it never actually touches. The index entry would read:
>
> ```
> **Story-Level Interleaving:**
> - EPIC-STAFF-INVITE ⇄ EPIC-STAFF-ACCEPT-INVITE — the "generate and email a unique, time-limited invite link" story in Staff Invite must land before Staff Accept-Invite's "staff member sets a password from the invite link" story can be built or demoed; the remaining Staff Invite stories (revoke, resend, status list) are not required first.
> ```

If you see this subsection on a real project, read it as permission to interleave two waves at the *story* level around exactly the named pair — not as license to interleave the whole epic pair freely.

---

## Section 8 Notes: a real, sharp boundary — not smoothed over

Section 8's own AI Guide invites domain rules, validation rules, business rules, security requirements, and — in the same breath — "any architectural constraints or open questions the AI must know when generating stories." That second half sits uneasily next to the document-level premise stated three lines above it in the same template: *"If a sentence describes how something is built rather than what it delivers to a user, it belongs in the architecture document or story acceptance criteria — not here."* Read `epic.example.md` Section 8 and the tension isn't theoretical — it's on the page:

> *"Payer data is scoped to the organisation. A Finance Manager from one organisation must not be able to view, edit, or import Payers belonging to another organisation. This is enforced by filtering all Payer queries by `orgId` from the JWT."*

The first two sentences are a domain/business constraint — genuinely epic-appropriate, no architecture opinion in them at all. The third sentence is a specific implementation mechanism: JWT-based `orgId` filtering. That's an authentication-and-tenancy shape decision, and epics have no architecture dependency — `/gen-architecture` hasn't run yet when this sentence is written. Nothing catches this mechanically either: both the epic template's own Part 2 Verification checklist and `/review-epics`' check #10 (Template Compliance) scope the "no implementation detail" rule to Sections 1 through 5 (or 1 through 6, in `review-epics.md`'s own phrasing) — Section 8 sits outside that rule in both places, by design, because Section 8 is exactly where domain rules requiring some technical framing are supposed to live. There is no check anywhere in this phase that will flag a Section 8 sentence for pre-empting an architecture decision.

**How to use Section 8 correctly, given that this gap is real and by design, not a bug you should expect fixed:**

- **Business/domain constraints belong here with full confidence.** A validation rule, a threshold, an exemption, a security *requirement* ("Payer data must be scoped to the organisation") — these are true regardless of what the architecture turns out to be. Write them plainly.
- **A technical *mechanism* belongs here only as a labeled, revisable forward-looking hint — never as settled fact.** If you or the AI find yourselves writing "this is enforced by X," ask whether X is a requirement (scoping must exist) or a guess at implementation (it'll be a JWT claim). If it's the latter, it's fine to leave the hint in — it can genuinely help `/gen-architecture` land faster on an already-reasonable default — but hold it loosely. Expect it to be revised, and don't be surprised if it's flatly contradicted once the real authentication and tenancy shape gets decided.
- **The test to apply on any Section 8 sentence:** *is this a rule about what the business requires, or a rule about how the system will be built?* The first is Section 8's actual job. The second is a hint on loan from a decision nobody has made yet — treat it accordingly, and don't let a specific phrase like "filtering by `orgId` from the JWT" quietly harden into a fixed architecture decision in your own head before `/gen-architecture` has actually run and confirmed it.

---

## The story-count estimate: a sanity check, never a commitment

`/review-epics` Step 5C's capability walkthrough states, for every epic: *"This epic covers [N] BRD requirements and will produce approximately [N] stories."* No formula backs that second number anywhere in the command file — it's presented with the same confidence as the REQ count, which is an exact, checkable fact, right next to a number that is not.

Treat it exactly the way `00-orientation.md`'s general warning about ungrounded scores tells you to: as a rough signal, not a measurement. The one check you can actually run yourself, cheaply, in your head: cross-reference it against the ~4–10-stories-per-epic rule of thumb relative to the epic's REQ count. A 5-REQ epic estimated at "approximately 6 stories" is unremarkable. A 5-REQ epic estimated at "approximately 15 stories" is worth one clarifying question — not because 15 is necessarily wrong (a REQ with a multi-step state machine, like Director Approval's submit/notify/approve-reject/resubmit, can genuinely cost more than one story each), but because if the AI can't give you a reason when you ask, you should discount the number to zero. It commits nothing — the real, authoritative story count only exists once `/gen-stories` actually runs against a Locked architecture and each story is individually written. Nothing about sprint capacity, velocity, or scheduling should be decided off this number.

---

## Tier 3 ("PO decides"): what it actually asks of you

`/review-epics`' fix-tier system correctly routes REQ reassignment, scope boundary changes, and layer boundary corrections to Tier 3 — a PO decision before execution — but gives no criteria for making that decision. Two real decision aids, worked against Harborview.

### REQ reassignment

> **Reassign a REQ to a different epic only if that requirement's acceptance criteria would naturally be written and tested as part of the OTHER epic's capability — not merely because it feels thematically closer.**

Worked test: suppose, during generation, REQ-013 ("the system must allow the Finance Manager to void a Sent or Approved invoice... a voided invoice cannot be edited, reactivated, or resent") had been proposed for EPIC-004 Director Approval instead of EPIC-003 Invoice Management, on the reasoning that "voiding relates to invoices that went through approval." Apply the test: would REQ-013's acceptance criteria naturally be written and tested as part of the Approval capability — submit, notify, approve/reject, resubmit? No. Voiding is a Finance-Manager-triggered lifecycle action on an already-existing invoice record, and it's testable on an invoice that reached Sent status without ever crossing the approval threshold at all — Director Approval never has to be exercised for REQ-013's acceptance criteria to be fully verified. It belongs with EPIC-003's other lifecycle actions (create, calculate, number, PDF, void), which is exactly where the real epic set puts it. "Thematically closer to approval" loses to "acceptance-criteria home is invoice lifecycle" — that's the test doing its job.

The same logic scales down to scope boundary changes: ask "if I had to write the Section 6 Definition-of-Done walkthrough step exercising this capability, which epic's walkthrough would it naturally belong inside?" That's the same acceptance-criteria-home question, one level up.

### Targeted PO questions: generic vs. sharpened

`/review-epics` Step 5E's four example questions are explicitly generic and interchangeable across any epic — read them and notice none of them names anything specific to the epic in front of you:

> *"Is there any capability you discussed with the client in this area that is not covered by the In Scope list?"*

That question is real and worth asking, but it's a checklist item wearing a question mark, not an interrogation of this epic's actual risk. A sharpened, epic-specific version of the same underlying concern, grounded in EPIC-001 Payer Management:

> *"REQ-002 says the deduplication key is 'configurable per organisation.' If an Org Admin changes that configured key — say from phone number to email — after Payers already exist under the old key, what happens to those existing Payers? Are they silently exempt from dedup protection under the new key until someone happens to re-register them, or does the system need to re-run a dedup pass retroactively across existing records? Is that answer actually in REQ-002 or REQ-003, or is this a gap the BRD never closed?"*

What makes the second version sharper, concretely: it interrogates a specific state transition (a setting changed mid-lifecycle, after data already exists under the old setting) that the REQ text as written simply doesn't address either way; it can't be answered by re-reading the epic file, which is what makes it worth a PO's time rather than a structural check's; and it can't be copy-pasted into a different epic verbatim — a question about what happens when a dedup key changes after Payers exist makes no sense asked of, say, Overdue Management's reminder schedule. That non-transferability is the actual test for whether a targeted question is doing its job: if it would read naturally pasted into any other epic's review, it's still generic no matter how specific it sounds.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s terminology map — go there for Scope/Priority/Layer and the other base pairs. New terms specific to this phase:

- **Epic Size / No Gold-Plating (`/review-epics` checks #12–13)** — Epic Size flags an epic whose REQ count is a clear outlier against its siblings in the index's Epic Registry, relative to the ~4–10-stories-per-epic rule of thumb. No Gold-Plating flags any In Scope item or Section 8 Notes entry that doesn't trace back to a REQ in Section 2 or an approved domain rule — invented scope beyond what the BRD or domain doc actually calls for.
- **Story-Level Interleaving** — a note in `epics/index.md` Section 3 marking a dependency pair whose true build order is finer than epic-before-epic: one named story in epic A must land before one named story in epic B, without B needing the rest of A's Definition of Done first. See the worked hypothetical above.
- **Fix tiers (Tier 1–4)** — `/review-epics`' fix-tier system: Tier 1 applies silently, Tier 2 confirms before applying, Tier 3 is a PO decision with no automatic action, Tier 4 halts the review entirely (structurally incomplete coverage, misaligned scope, a Layer violation spanning multiple items, or an Epic Size outlier requiring a split-and-regenerate).

---

## Common mistakes

- Treating REQ count alone as the sizing test — a small epic can still glue two capabilities together; a large one can still be genuinely cohesive.
- Doing `/review-epics`'s REQ-by-REQ verification work at the `/gen-epics` Step 4 planning table, where the material to check it against doesn't exist yet.
- Listing a dependency because an epic "feels foundational" instead of naming the specific story in the dependent epic that's actually blocked.
- Letting a Walking Skeleton REQ silently land in whichever epic feels like the closest fit, without checking whether the resulting epic set still gets sequenced first in the build order (FW-048).
- Treating every epic-to-epic dependency as fully sequential when the real blocker is one specific story — check the Story-Level Interleaving subsection before scheduling waves as strictly sequential.
- Letting a Section 8 technical hint (a specific token shape, a specific filtering mechanism) quietly become a fixed architecture decision in your head before `/gen-architecture` has actually run.
- Treating `/review-epics`' "approximately N stories" as a commitment for sprint planning rather than a rough, unverifiable sanity check.
- Reassigning a REQ because it feels thematically closer to another epic, instead of asking where its acceptance criteria would actually be written and tested.
- Asking one of the four generic example questions verbatim instead of sharpening it into something that couldn't be pasted into a different epic's review unchanged.
- Leaving Section 3.2 (Out of Scope) thin or generic — every excluded capability needs a named epic or an explicit "Phase X / deferred," not a vague gesture.

---

## Harborview in practice

[`epic.example.md`](../projects/TEMPLATE/epics/epic.example.md) (EPIC-001, Payer Management) is the worked model for everything in this chapter, and — because the full Harborview epic set was never generated as a standalone directory in this repo, only this one example file — its own Section 3.2 and Section 5 are also the best evidence available for what the rest of the set looks like:

- **Section 3.2 (Out of Scope)** names EPIC-003 (Invoice Management), EPIC-004 (Director Approval), EPIC-005 (Payment Collection), EPIC-006 (Overdue Management), and EPIC-007 (Dashboard and Reporting) by ID — the concrete evidence that BRD Section 5.2 (eight REQs, one heading) became two epics, not one, per the cohesion test walked through above.
- **Section 5 (Dependencies)** states Payer Management's role as the dependency root in plain language ("All invoice epics depend on this one") — the assertion the falsifiable test above confirms rather than takes on faith, grounded in REQ-006's "selecting a Payer."
- **Section 8 (Notes)** is the canonical instance of both a correct use (the deduplication and Opening Balance business rules, traced to REQ-002/003/005) and the sharp edge (the JWT-based `orgId` filtering sentence) discussed above — read it with both in mind, not just as a positive example.
- **The filled Verification checklist** at the bottom is worth reading once for its own sake: every check cites the specific REQ IDs or BRD section it verified against, which is the standard of specificity `/review-epics`' own 13 checks hold every subsequent epic to.
- **Section 2's Walking Skeleton column** shows the split case named above directly: REQ-001 (register a Payer) is tagged `Yes` here in EPIC-001, while the rest of Harborview's Walking Skeleton — create the invoice, calculate its total, send it, collect real payment — lands in EPIC-003 (Invoice Management, named in this epic's own Section 3.2, per `brd.example.md` Section 5.2). A Payer has to exist before an invoice can be created, so the chain genuinely does cross this epic boundary; that's the legitimate version of the split case, not the fragmentation the check exists to catch.
