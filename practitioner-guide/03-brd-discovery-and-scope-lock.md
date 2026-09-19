# BRD Discovery & Scope Lock

Covers the BRD half of Stage 2 (Discovery) and Stage 3 (BRD & Scope Lock) in [`SDLC.md`](../SDLC.md): `/gen-brd-playbook`, `/run-brd-discovery`, `/gen-brd`, `/review-brd`, and the BRD document itself. Read [`00-orientation.md`](00-orientation.md) first if you haven't — this chapter assumes you already have its terminology table (Scope vs. Priority vs. Layer) in hand and builds on it rather than re-explaining it from scratch.

If you haven't yet turned domain knowledge into a validated playbook, that's the previous chapter's job — see [`02-domain-discovery.md`](02-domain-discovery.md) for the Domain Confidence Score treatment this chapter's Requirement Confidence Score section leans on for comparison.

---

## Purpose

The BRD is the contract between Product and Engineering. Nothing downstream — no epic, no story, no architecture decision — gets generated without an Approved BRD to point at. `/gen-epics` hard-gates on it. That's what "locking scope" means in practice: once `/review-brd` sets Section 14's Status to `Approved`, every Section 5 requirement becomes a fixed unit of work that epics and stories will trace back to by REQ-ID, and every Section 5 feature-area heading becomes an epic boundary you don't get to redraw for free later (see [BRD → Epic traceability](#what-the-brd-commits-you-to-downstream-the-epic-boundary-you-are-already-drawing) below).

This is also why the two commands that produce the BRD — discovery and drafting — deserve more care than "ask some questions, write a document." A BRD requirement that's too vague costs you a follow-up conversation after epics already exist. A BRD requirement that's too dense or too thin costs you a broken story downstream. A scope boundary drawn carelessly at Section 1.4 or Section 5.21 costs you a client dispute about what "Phase 1" actually included. Everything in this chapter is aimed at catching those mistakes while they're still cheap — during discovery and drafting, not after approval.

---

## The discovery-mode decision: live session, AI-synthesis, or skip to `/gen-brd`

*(A process decision made before discovery starts — not something checkable against the finished BRD.)*

[`SDLC.md`](../SDLC.md) (Stage 2) already names two modes: a live requirement discovery session run via `/run-brd-discovery`, and — for projects with no external client — "AI-synthesis-and-review," where the AI proposes an answer grounded in the domain knowledge document's defaults and the PO reviews, corrects, or overrides it. That second mode is real and sanctioned. What's missing is that `run-brd-discovery.md` itself has no coded branch for it — no step that says "if there's no client, do X instead of Y." The command assumes a two-role interview: an AI asking questions and a client answering them. If you have no client, you have to *enact* AI-synthesis-and-review yourself, inside the same command, rather than expect the command to switch modes for you.

**The decision test:**

| Situation | Mode | How to run it |
|---|---|---|
| External client exists and is reachable for a session | **Live discovery** | Run `/run-brd-discovery` as written — two-role interview, client answers in their own words. |
| No external client (internal tool, or the client genuinely isn't available on the timeline you need), and the domain is well-understood (an existing, validated playbook; a small or simple project) | **AI-synthesis-and-review** | Run `/run-brd-discovery` yourself, but for each playbook question: propose the domain-default answer first, write down *why* a real client in this situation might answer differently, then record your own confirmed or corrected answer as if you were relaying it from a session. Don't just accept the default silently — the "why might this differ" step is what keeps this from becoming a rubber stamp. |
| No external client, but the domain is niche, unfamiliar, or high-stakes | **Live discovery is still worth insisting on** — find *someone* who knows the domain (a subject-matter advisor, a future user) even if they're not the paying client. AI-synthesis on an unfamiliar domain just launders your own guesses into a "confirmed" BRD. |
| You're tempted to skip discovery entirely and jump to `/gen-brd` | **Not actually available as a shortcut** — `/gen-brd`'s gate check (`gen-brd.md` Step 1.5) requires `brd-discovery-state.md` to exist with at least one topic marked Covered. You cannot generate a BRD with no discovery state. What "skip" really means in practice is: run the AI-synthesis pass solo and quickly, not that you bypass the artifact. |

Project size matters too: on anything beyond a small, fixed-scope internal tool, prefer live discovery whenever a client exists at all, even an internal stakeholder. The value of catching a Scope Risk Flag live — with the playbook's scripted "Response to client" line right in front of you — is highest early, before you've drafted anything. AI-synthesis is proportionate for small internal tools where the "client" is really just you or your own team confirming defaults.

---

## Density and judgment through the live session

### Requirement Confidence Score: what the number actually measures

`/run-brd-discovery` tracks this "mentally" during the session (Step 6) and calculates it formally at the end (Step 10): topics marked Covered in Discovery State Section 3, divided by total topics, as a percentage. 85% is the stated gate before the command offers to generate the BRD in the same session.

Two things to know before you trust that number:

**The denominator is topics *and* journeys together.** The command text at Step 6 describes the running score as "coverage of known-variation topics from domain knowledge Section 10," which sounds like it's Section 4 (Core Discovery Questions) alone. But the actual calculation at Step 10, and the discovery state file's real Section 3 structure (see [`brd-discovery-state.template.md`](../projects/TEMPLATE/brds/brd-discovery-state.template.md)), has two subsections — "Section 4 — Core Discovery Questions" *and* "Section 6 — Journey Walkthroughs" — both with their own Covered/Pending rows. Treat the denominator as every row across both subsections. If you're calculating the score yourself to sanity-check a number the AI hands you, count both.

**Coverage is not resolution — and the score can't tell the difference.** A "Covered" mark means a topic was asked about and answered. It says nothing about whether the answer was a real decision or a shrug. A client who responds to every question with "it depends" or "we'll figure that out later" can rack up 100% coverage while the BRD you'd draft from those answers would be nearly content-free. This is the single easiest way to be handed a confident-looking number that's hiding an unscoped project.

**The fix, concretely:** when a client's answer is genuine uncertainty — not "here's our answer and it's conditional on X" but "I don't actually know" — do not mark that topic Covered in Discovery State Section 3. Mark it Pending, and immediately open an entry in Section 7 (Open Questions) with a real owner and due date. This keeps the score honest at the cost of it moving slower — which is the right trade. And even at ≥85%, personally re-check Section 7 before agreeing to generate the BRD: if more than one or two of the remaining open questions are genuine **scope blockers** (the BRD's own §13.4 definition — "would materially change scope if answered one way vs. the other"), don't proceed yet regardless of what the percentage says. The threshold is a floor, not a green light on its own — see [`00-orientation.md`](00-orientation.md)'s general note on scores you can't independently reproduce, which applies here exactly as it does to the Domain Confidence Score.

### Known Variation vs. an ordinary follow-up

During a live session, Step 6's rule is "when an answer is ambiguous or contradicts something said earlier, probe immediately." That's right, but it doesn't tell you what to *do* with what you find once you've probed it. There are two different outcomes hiding behind "the client said something surprising," and they get handled completely differently:

- **An unusual value on an axis the playbook already has a topic for** → resolve it in the session, record it, move on. Nothing structural changes.
- **A structurally new axis of variation — a dimension the domain knowledge document doesn't model at all** → this needs a domain knowledge update, not just a BRD note, because the gap isn't specific to this client. The next project in this domain will hit the same undiscovered variation.

**The test:** does the answer fit inside a topic subsection that already exists in Section 4 of the playbook, or does it need a topic that isn't there?

Grounded in the Harborview playbook (`brd-playbook.example.md`): if James had said "our VAT rate actually depends on whether the client is in Northern Ireland or the rest of the UK," that's still inside 4.1 (VAT and Tax Model) — an unusual *value*, handled by probing and recording a business rule. But if he'd said "actually, for some clients we don't invoice them directly — we invoice their main contractor, and the client we're actually doing work for never sees an invoice at all," that's a new relationship pattern (indirect/third-party billing) that none of the playbook's eight topics address. That's a new axis. The right move isn't a BRD Open Question — it's flagging it as a candidate for a domain knowledge update (a job for `/gen-domain-knowledge` or a manual edit to the domain doc's Section 10, Known Variations) before you try to write a clean BRD requirement for it. Writing the requirement first, on an unmodeled concept, just means you're inventing domain rules mid-BRD-draft with no grounding — exactly the failure mode Section 5.21 vs. Parking Lot exists to prevent (see below).

### The circuit breaker: when a client won't converge

Nothing in `run-brd-discovery.md`'s "Resuming across sessions" guidance tells you when to stop scheduling more sessions. It tells you how to record a revision (update the saved answer, note it in Open Questions, update the Scope Risk Flag if one's affected) but treats every revision as equally routine.

It isn't. Two signals mean the actual problem isn't the discovery process — it's that you're talking to someone who can't give you a stable answer:

- **The same topic gets revised more than twice across sessions.** One correction is normal — people remember details differently on a second call. Three passes at the same topic means either the client hasn't actually decided, or the person in the room isn't the person who owns the decision.
- **A Scope Risk Flag flips status more than once** (Open → Resolved → Open again, or similar). This usually means the "resolution" wasn't real — someone agreed to something in the room that got overturned after the call.

When either happens, stop scheduling more discovery sessions and escalate: ask to bring in a decision-maker above the immediate contact — their manager, a director, whoever actually owns the scope call. This isn't about being difficult with the client; it protects the relationship. Endless re-litigating of the same point in session after session erodes trust in the process a lot faster than one honest "we need someone with authority to settle this" conversation.

---

## REQ-writing density: the single most important skill in this phase

Section 5's own guidance — "one sentence, plain English, what the system must do" — tells you the *form* a requirement takes. It says nothing about *how much* goes into that sentence. That gap matters more than almost anything else in this chapter, because `/gen-epics` treats REQ IDs as atomic units, and stories are written one-to-one (or close to it) against REQs. Get the density wrong and you're not fixing a wording problem later — you're re-splitting or re-merging work that epics and stories have already been built around.

### Two real examples that get the density right

**REQ-002** (`brd.example.md`, Section 5.1):

> "The system must detect duplicate Payers on registration. The deduplication key is configurable per organisation: phone number, email, or both (either match triggers detection). Default is phone number."

This is three sentences packed with detail — and it's correctly dense, not over-fat. Everything in it is a facet of **one** capability: duplicate detection at registration. The configurability, the three key options, and the default aren't separate features you could ship independently — they're the specification of the one feature. Strip any of them out and you no longer have a buildable requirement: without "configurable, three options, default = phone" a developer would have to come back and ask which key to check, exactly the follow-up question §13.3's downstream-readiness check exists to prevent.

**REQ-007** (`brd.example.md`, Section 5.2):

> "The system must calculate subtotal, VAT at 20%, and total automatically. These values must recalculate in real time as line items change and must not be editable by the user."

Same pattern. Three calculated fields, a real-time recalculation behavior, and a non-editability constraint — but this is still one capability (automatic invoice totals), not three. "Real time" and "not editable" are behavior details of that one capability, not independently shippable features.

### The rule these two examples are following

> **One REQ = one independently testable, independently story-able capability.** An "and" joining two separately-shippable things → split into two REQs. A sub-detail of another requirement's own behavior — a calculation rule, a default value, a field constraint — → fold it into the REQ it belongs to. Don't give it its own ID.

The test that operationalizes this: could each half of the sentence ship, get tested, and get demoed on its own, with the other half simply not built yet? If yes to both halves, split. If the second half is meaningless without the first — it's not a capability, it's a detail of one — fold it in, the way REQ-002 folds in "default is phone number."

### A deliberately bad over-fat version

Here's what an over-fat REQ looks like, constructed from Harborview's own invoice-creation flow:

> **REQ-BAD-FAT** — "The system must allow the Finance Manager to create an invoice with line items and a due date, submit it for Director approval if the total is above the threshold, and generate a PDF once approved."

This reads like a reasonable sentence, but it welds together three independently shippable capabilities: (1) invoice creation, (2) the approval-submission workflow, (3) PDF generation. Each already has its own real REQ in the actual Harborview BRD — REQ-006, REQ-010/REQ-011, and REQ-009 respectively — for exactly this reason. If this had shipped as one REQ instead of three: `/gen-epics` treats it as one atomic unit, so a story writer either has to awkwardly carve three stories out of a single REQ-ID (breaking the traceability the whole numbering scheme exists for), or builds one oversized story that tries to cover creation, approval, and PDF generation together — which blows past normal story sizing and can't be independently tested or demoed if, say, PDF generation slips a sprint but approval doesn't.

### A deliberately bad over-thin version

And here's the opposite failure, on the same underlying requirement as Harborview's real REQ-010:

> **REQ-BAD-THIN** — "Invoices above a threshold require Director approval."

This looks fine at a glance — it's one sentence, it's plain English, it passes a superficial "is this short enough" read. But it's missing the business rule that makes it usable: what's the threshold? £10,000? Inclusive or exclusive at exactly that figure? How many Directors need to approve — one, or all three? None of that is here. Compare it to the real REQ-010: "Invoices above the organisation's high-value threshold must be submitted for Director approval before they can be sent. The approval threshold is configurable per organisation. Harborview's threshold is £10,000 (inclusive)." A developer handed REQ-BAD-THIN can't write a single acceptance criterion — not because the sentence uses vague deferral language like "appropriate" or "standard" (the kind of phrase §13.3's checklist is built to catch), but because it silently omits a number that only a follow-up conversation can supply. That's a worse failure than the vague-language case, because nothing about the sentence's *shape* trips the checklist — it just quietly fails the test in practice.

**What to check for every REQ you write or review:** does it contain every number, threshold, default, and configuration detail that makes it a rule rather than a category? If a developer reading only this sentence would have to ask "compared to what?" or "how many?" or "which one by default?" — it's under-specified, no matter how short and clean it reads.

---

## Scope vs. Priority vs. Layer, in full working depth

[`00-orientation.md`](00-orientation.md) gives you the three-axis table and one worked example each. This section goes one level deeper on the place those axes are genuinely easy to confuse in a real BRD: the boundary between "in scope but might get cut" and "not in scope at all, but for a real, grounded reason."

### Should Have vs. Out of Scope — deferred: the real difference

These sound similar and are not. **Should Have** is a Priority value (BRD §5, per requirement) — it only applies to a requirement that is already **In Scope** (§1.4). It means: this ships in this phase if time allows, and it's the first thing cut under schedule pressure if it doesn't. The requirement still has a REQ-ID in a real Section 5.N feature area, still gets built into epics and stories for this phase, and its fate is a delivery-capacity question, not a scoping question.

**Out of Scope — deferred** is a different thing entirely: the capability is **not being built this phase at all.** It doesn't live in a real Section 5.N feature area. It lives in one of two holding places depending on how well it's understood — and which one is exactly the distinction this section exists to nail down.

### Section 5.21 (Future Capabilities) vs. Section 15 (Parking Lot): the actual test

[`FW-024`](../decisions/FW-024-parking-lot-deferred-capabilities.md) gives the exact wording, and it's worth learning verbatim because it's the cleanest single-sentence test in this whole phase:

> "Can you write one factual sentence describing exactly how it will behave, backed by something already in the domain knowledge document or this discovery session? If yes, Section 5.21. If you're guessing, Section 15."

Concretely:

| | Section 5.21 (Future Capabilities) | Section 15 (Parking Lot) |
|---|---|---|
| Domain modeling | Already exists — an entity, lifecycle, or rule in the domain knowledge doc backs it | Doesn't exist yet |
| Discovery | Already happened — client stated it, or it's a known domain variation | Hasn't happened |
| Gets a REQ-ID | Yes, immediately, sequential with every other REQ in the document | No — not until promoted |
| Priority / Source / Layer | Yes — full fields, same as a real requirement | No — these fields don't apply to an undiscovered idea |
| Can `/assess-change` write it directly | Yes | No — blocked until a discovery pass happens |

**Harborview's worked example is REQ-022** (`brd.example.md`, Section 5.21):

> REQ-022 — Accounting System Export (Xero). "The system must export invoice and payment records to Xero via its accounting API on a schedule the Finance Manager configures — the domain knowledge document already models Accounting System as a Typical Integration Point, and James Okafor confirmed Xero as Harborview's Phase 2 accounting system during discovery." Priority: Should Have. Source: Client-Stated. Layer: Core.

Walk through why this passes the test: the domain doc already models "Accounting System" as a Typical Integration Point (grounding exists), and James specifically named Xero during the live session (discovery happened). The description is a factual sentence, not a guess — it says what the export does and when it fires, based on something already on record. That's why it earns a real REQ-ID with full Priority/Source/Layer fields even though it's explicitly Out of Scope for Phase 1 (§1.4 lists "Accounting system integration (e.g. Xero, QuickBooks)" under Out of Scope, cross-referencing this same capability). This is the demonstration of §13.1's verification check: every deferred Out of Scope item either is a permanent exclusion or has a REQ-ID — REQ-022 is that REQ-ID.

**Contrast with Harborview's Parking Lot entry, PL-001** (`brd.example.md`, Section 15): the idea of letting an admin assistant submit invoices on the Finance Manager's behalf, with the Finance Manager approving before send. This surfaced as an aside during UAT prep — months after discovery closed — and nothing in the domain model or the BRD backs a "delegated/assisted authoring" concept for invoice creation. No REQ-ID. No Priority, Source, or Layer — those fields don't apply, because nothing has actually been scoped yet. It's recorded with a Promotion Path note instead: "Requires a domain discovery pass on delegated/assisted authoring before it can become a REQ." That promotion path is mandatory — per FW-024, a Parking Lot entry can never become a real REQ-ID through `/assess-change` alone. It has to go through an actual discovery pass first, the same modeling rigor every other requirement received.

**If you're mid-BRD-draft and genuinely unsure which section an item belongs in:** try to write the one factual sentence right now. If you can write it without inventing anything — pulling only from the domain doc or what was actually said in this session — it's 5.21. If writing it means guessing at behavior nobody has confirmed, it's 15, and it stays low-ceremony (no tags, no fields) until a real discovery pass earns it a REQ-ID.

### When Layer and Priority genuinely conflict

`00-orientation.md` already covers the resolution rule for a Must-Have `Ext:PK` requirement (Layer wins for build order, Priority governs sequencing within what's buildable). The one thing worth adding here: when you hit that conflict while drafting a BRD requirement, treat it as a signal to consider splitting the REQ itself — the same density judgment from the previous section applies. Often the part that's genuinely urgent is Core-buildable right now, and only the extension-specific part actually has to wait on the Ext seam. Splitting the REQ at that boundary, rather than leaving one requirement carrying a contradictory Priority/Layer pairing, keeps both axes honest.

---

## What the BRD commits you to downstream: the epic boundary you are already drawing

This is worth stating as plainly as the audit found it needed to be stated, because nothing in `gen-brd.md` says it explicitly: **each Section 5.N feature-area heading becomes one epic, 1:1, when `/gen-epics` runs.** This isn't an editorial convenience grouping you can rearrange freely later — it's a mechanical mapping (confirmed in `FRAMEWORK-AUDIT.md`'s Phase 2 findings, and consistent with `SDLC.md` Stage 4's description of epics as "cohesive delivery units, each representing a complete, independently useful capability").

What this means while you're still drafting Section 5: choose feature-area boundaries the way you'd choose epic boundaries, not the way you'd choose chapter headings in a document. Ask, for each heading, "could this feature area ship and be demoed as a complete, independently useful capability on its own?" If a heading is really covering two unrelated capabilities glued together for convenience — or worse, a catch-all "Miscellaneous" or "Other" heading — split it now, at BRD-draft time, while it costs you nothing but renaming a `###`. Fixing it after `/gen-epics` has already generated (and a PO has already reviewed) a bloated or incoherent epic costs a real re-scoping conversation.

Harborview's five feature areas — 5.1 Payer Management, 5.2 Invoice Management, 5.3 Payment Collection, 5.4 Overdue Management, 5.5 Dashboards — are five genuinely independent capabilities, and that's exactly why they became five clean epics. That's not an accident of how the example was written; it's the pattern to copy.

---

## Reviewing and approving: what "good" looks like, and what to do when you're uncertain

`/review-brd` (redesigned per [`FW-019`](../decisions/FW-019-review-model-redesign.md)) walks you through the BRD feature-area by feature-area, in real-world framing ("The Finance Manager will be able to register a new payer... does this match what was agreed with the client?"), not as a document to read cold and rubber-stamp. What "good" looks like at the end of that walkthrough:

- You can restate what each requirement will actually produce in the real world, in your own words, without looking back at the REQ text — this is the comprehension bar FW-019 exists to raise, not just approval-as-formality.
- Section 16's Flags table is empty, or every flag has been resolved, not just acknowledged.
- Section 12 has no unresolved scope blockers.
- Every `[Assumed]` tag has been actually looked at — confirmed as reasonable, corrected, or moved to Open Questions — not silently accepted because it was easier than stopping to think about it.

### The gap: what to do when you don't disagree, you just don't know

`/review-brd`'s response table (Confirm / Change / Remove / Too vague to frame) assumes you're equipped to make a call on every requirement. Sometimes you aren't — you weren't on the discovery call that produced a particular answer, or the requirement touches a business decision that's genuinely the client's to make, not yours to second-guess. Being forced into Confirm/Change/Remove in that situation means either rubber-stamping something you don't actually know is right, or guessing at a change the client never asked for.

**The move:** don't force yourself into one of those three boxes. Tell the AI directly that you're uncertain and want it routed to Section 12 (Open Questions) with the **client** as owner — not you. Mechanically this is just a "Change" applied through the normal review flow (the walkthrough already supports "discuss → agree → apply to BRD immediately"), but the content of the change is moving the item out of Section 5 and into Open Questions rather than editing the requirement text. This keeps the BRD honest about what's actually been confirmed vs. what's still waiting on the person who can actually answer it, and it's a legitimate outcome of the review — not a failure to complete it.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s Scope / Priority / Layer table and its "other pairs worth knowing" table — go there for the base definitions, not here. Two things specific to this phase, not covered in the orientation table:

- **Section 5.21 (Future Capabilities) vs. Section 15 (Parking Lot)** — both hold Out of Scope items, but only 5.21 items have real domain/discovery grounding and a REQ-ID. The test: can you write one factual sentence about it, backed by the domain doc or this session? See the full worked comparison above.
- **Requirement Confidence Score** — a coverage measure (topics + journeys asked and discussed), not a resolution measure (topics actually decided). Don't conflate the two, and don't trust the number alone above scope-blocking Open Questions.

---

## Common mistakes

- Marking a topic Covered when the client's real answer was "it depends" or "I don't know" — mark it Pending and open a question instead.
- Writing a REQ that's really two or three separately-shippable capabilities glued together with "and."
- Writing a REQ so thin it omits the number, threshold, or default that makes it usable — passing the "one sentence" test while failing the actual point of it.
- Confusing Should Have (in scope, ships if time allows) with Out of Scope — deferred (not being built this phase at all).
- Putting a genuinely ungrounded idea into Section 5.21 because it's convenient, instead of the honest Section 15 (Parking Lot) — you can't write the one factual sentence FW-024 requires without guessing.
- Noting a structurally new domain variation as just a BRD aside instead of flagging it for a domain knowledge update — the next project in this domain inherits the same gap.
- Letting a Section 5 feature-area heading sprawl into a "Miscellaneous" catch-all — it becomes an incoherent epic the moment `/gen-epics` runs.
- Approving a BRD with `[Assumed]` tags nobody actually re-examined during `/review-brd`.
- Scheduling a fourth or fifth discovery session on the same unresolved topic instead of escalating to a decision-maker above the immediate contact.

---

## Harborview in practice

`brd.example.md` is the worked model for everything in this chapter. Specifically worth studying directly:

- **REQ-002 and REQ-016** for source-tagging discipline — REQ-002 shows a correctly dense `[Client-Stated]` requirement (see the [REQ-writing density](#req-writing-density-the-single-most-important-skill-in-this-phase) section above); REQ-016 (partial Stripe payment handling) shows `[Assumed]` used sparingly and honestly, for a low-risk inference rather than something that should have been confirmed with the client.
- **Section 5.21, REQ-022** (Accounting System Export — Xero) as the canonical demonstration of a deferred-but-grounded capability: Out of Scope in §1.4, but with a real REQ-ID, full fields, and a description traceable to both the domain knowledge document and the actual discovery session.
- **Section 15, PL-001** as the contrasting Parking Lot entry: ungrounded, undiscovered, no REQ-ID, explicit Promotion Path — the item that failed the Section 5.21 test and correctly landed in the low-ceremony holding section instead.

The paired playbook (`brd-playbook.example.md`) and discovery state (`brd-discovery-state.example.md`) files are worth reading alongside the BRD itself — they show the full chain from a scope risk flag that *didn't* trigger (SRF-004, per-client dunning, closed out in session) through to the requirement that made it into the final document, which is the clearest way to see the density and judgment calls in this chapter applied end to end.
