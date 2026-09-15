# Framework Audit — Practitioner Guide Research

**Date:** 2026-09-15
**Purpose:** This document is the research base for the Practitioner Guide project. It was produced by simulating an experienced Product Owner running the Ascendra Framework end to end — six parallel deep-dive passes, one per SDLC phase group, each reading every command, template, `.example.md`, and relevant decision record in full and reporting concrete friction, not summaries.

**Status:** Point-in-time research artifact, not a living document. Do not update this file as the framework evolves — file new findings as new ADRs or against the Practitioner Guide directly. Kept here so the research isn't lost and can be re-consulted while building the guide.

---

## How to read this document

1. **Executive Synthesis** — the findings sorted into two categories: framework defects to fix at the source, and documentation gaps to address in the Practitioner Guide. Start here.
2. **Six full phase reports** — the raw output of each research pass, preserved in full for reference when writing the guide (exact file/line citations, full reasoning, all strengths noted).

---

# Executive Synthesis

## A key structural finding, not part of the original six passes

Every `.example.md` file across all seventeen template pairs in `projects/TEMPLATE/` traces **the same fictional project**: Harborview Consulting Ltd, Invoice Management System, project code `HARBORVIEW-INV-001` — from `brief.example.md` through `release-notes.example.md`. Confirmed by direct grep across every example file. This means a coherent, continuous case study already exists in the framework; it has just never been narrated as one story. The Practitioner Guide should thread this existing example rather than invent a new one — reserving new worked material only for the harder cases the reports below found completely unexampled (extension projects, regulated/niche domains, multi-portal RBAC, cross-layer epics).

---

## Category A — Framework defects found (not documentation gaps)

These are places the framework's own files disagree with each other, with themselves, or reference something that doesn't exist. All are additive/corrective fixes — no structural redesign, no change to the framework's working model.

| # | Defect | Where | Fix shape |
|---|--------|-------|-----------|
| 1 | `brd.template.md` has no Section 5.21 (Future Capabilities), though `gen-brd.md` and `FW-024` both instruct POs to file deferred-but-discovered ideas there | `brd.template.md`, `gen-brd.md:130`, `FW-024:14,34-42` | Add Section 5.21 to the template with the same rigor as Section 15 (Parking Lot); add a worked row to `brd.example.md`; add a §13.1 verification check |
| 2 | `defect-severity.md` self-contradicts on whether a High defect can ever be deferred (one line says never, two lines later says yes with PO agreement; the test-execution-report template operationalizes "yes") | `reference/defect-severity.md:87-89,96`; `test-execution-report.template.md:133` | Rewrite condition 1 to: "Critical defects can never be deferred; High defects can only be deferred with explicit PO agreement" |
| 3 | Hotfix Cycle step 8 instructs merging into "the current sprint's working branch" — a branch the documented git model (story branches → `main` directly) never creates anywhere | `PROJECT-LIFECYCLE.md:718` vs. `implement-story.md` branch model | Either drop the merge-back step (rely on `main` as the single integration point) or define the sprint branch explicitly at Sprint Activation if one is genuinely intended |
| 4 | `init-project.md`'s code validator rejects 4-segment project codes (`CLIENT-DOMAIN-EXT-NNN`) as a structural error, while `FW-008` gives 4-segment codes as its own canonical example | `init-project.md:22-25` vs. `FW-008:48-49` | Update the validator to accept an optional extension segment, or correct FW-008's examples to match current practice (extension identity lives in the brief's Extension Context, not the code) |
| 5 | Five files (SDLC.md, PROJECT-LIFECYCLE.md, `gen-stories.md`, `review-stories.md`, `gen-epics.md` implicitly) treat a "story-level interleaving note" in `epics/index.md` as a reliable, checkable artifact — but `gen-epics.md` is never instructed to produce it, and no `epics/index.md` template exists to define its shape | `gen-stories.md:80`, `review-stories.md:288`, `SDLC.md:156`, `gen-epics.md:148` (silent) | Add explicit trigger criteria and output shape for the interleaving note to `gen-epics.md` Step 6; consider adding an `epics/index.md` template |
| 6 | `PROJECT-LIFECYCLE.md` claims `/review-epics` runs checks for epic "size" and "no gold-plating" — neither check exists in `review-epics.md`'s actual 11 checks | `PROJECT-LIFECYCLE.md:307` vs. `review-epics.md` Step 5B | Either add the missing checks to `review-epics.md`, or correct the claim in `PROJECT-LIFECYCLE.md` |
| 7 | `gen-ui-mocks.md` never reads `hard-instructions.md` (the corrections registry for `ascendra-ui`'s own docs), unlike `implement-story.md` which does — so a PO can approve a mock built on a pattern the framework already knows is wrong, then implementation builds something structurally different. FW-040 admits this gap | `gen-ui-mocks.md` Step 2; `FW-040` "Scope of Application" | Add `reference/ascendra-ui/hard-instructions.md` to `gen-ui-mocks.md`'s Step 2 reading list, both base and patch mode |
| 8 | `arch.example.md` is stale against `arch.template.md` — missing Section 3.1.3 (Screen & Nav carry-over), Section 6.5 (Logging, FW-041), correct actor-column shapes (FW-037), `If-Match` optimistic-concurrency headers (FW-045); section numbering has drifted (6.3/6.4/6.5 vs. example's 6.3) | `arch.example.md` vs. `arch.template.md` (diffed in full) | Regenerate `arch.example.md` against the current template (re-run `/gen-architecture` against the same Harborview BRD) so it demonstrates every currently-required section |
| 9 | The Gate Summary table in `PROJECT-LIFECYCLE.md` omits the Story Plan Confirmed gate entirely, despite it being a documented hard gate with no override | `PROJECT-LIFECYCLE.md:874-892` vs. `:862,489-501` | Add a "6a — Story Plan (per story)" row to the Gate Summary table |
| 10 | Stale cross-reference: `run-brd-discovery.md` sends POs to "Section 5 (Open Issues)" of the discovery-state file — that section is actually Integration Confirmation; Open Questions is Section 7 | `run-brd-discovery.md:189` vs. `brd-discovery-state.template.md` | Correct the section reference |
| 11 | `projects/TEMPLATE/standards/` has zero `.example.md` files for any of its three templates, unlike every other template folder in the framework | `projects/TEMPLATE/standards/` (directory listing) | Add one worked `technical-standard.example.md` and one worked `architectural-pattern.example.md` |
| 12 | `PROJECT-LIFECYCLE.md` itself flags, in its own text, that the architecture-lock confirmation ("lock"/"not yet") may have regressed from a richer design with rationale capture, and the discrepancy was never resolved | `PROJECT-LIFECYCLE.md:406-408` vs. `review-architecture.md:446-450` | Resolve the open decision: either restore a rationale prompt or explicitly close the question confirming bare `lock` is intended |
| 13 | FW-001 calls the standard versioning table "Document History" — every actual template calls it "Document Control" (or "Change History" specifically on architecture docs) | `FW-001-document-versioning.md:16` vs. every template's actual heading | Update FW-001's text to match what's actually on the page |

**Recommendation carried into execution:** fix all 13 as one batched, mechanical pass before writing new documentation against these artifacts — otherwise the guide has to caveat around known-stale reference material.

---

## Category B — Documentation gaps (recurring patterns across all six phases)

Rather than list every individual finding here (see the six full reports below for that), these are the *patterns* that recur across phases, since they should shape how the Practitioner Guide is structured rather than being treated as one-off fixes:

1. **Binary forks with no decision test.** "Is my domain niche enough for the optional sub-flow?", "live session or AI-synthesis?", "Pattern 2 gap or Pattern 3 scope shift?", "Known Variation or just a generic question?" — offered as judgment calls with no test to apply. `FW-024`'s Parking Lot test ("can you write one factual sentence backed by something already in the domain doc?") is the one place in the framework this is done right and should be the template for the others.
2. **Confidence/completion scores with undefined formulas.** Domain Confidence Score and Requirement Confidence Score are both "estimate a percentage" with no stated weighting — a PO cannot reproduce or sanity-check the number they're given.
3. **Worked examples show only the easy case.** The domain doc's generalization filter, the epic template's cross-layer/Extension rules, screen design's Drawer/Tabs/multi-portal guidance — every one has an AI Guide note warning about a hard case, and zero worked examples of it. Harborview itself is single-portal, Core-layer-only, contradiction-free.
4. **"PO reviews it" steps that are structurally unauditable.** Architecture's 35 checks and Screen Design's catalog citations both report only a pass/fail summary — a PO has no lightweight way to independently verify the claim without redoing the AI's own work.
5. **Terminology defined once, locally, never collected.** Portal/Screen/Surface/UI Pattern; wave/sprint/epic; Content Status vs. Status; and — flagged independently by the PO before this research began — Core/Extension (architecture) vs. BRD Scope (in/out) vs. Priority (Must/Should/Nice) vs. Layer (Core/Ext) are four genuinely independent axes with no single place that draws the map between them.
6. **Fixed thresholds that don't scale with stakes.** The 70% domain-confidence gate and the 85% requirement-confidence gate apply identically to a throwaway internal tool and a regulated cross-border-payments project, with no guidance to raise the personal bar for the latter.
7. **Silent-failure chains with no warning.** The clearest example: a domain rule misclassified as "universal" (Section 5, UBR) rather than "variable" (Section 10, Known Variation) is never asked about again during BRD discovery and never appears in the BRD unless the client spontaneously contradicts an assumption they were never shown — and even then, client disagreement is treated as a scope risk to flag, not evidence of misclassification. Nothing in the framework warns a PO this is the highest-stakes call in domain modeling.

---

# Full Phase Reports

## Phase 1 — Domain Discovery (Stage 2, domain half)

*Files read: `gen-domain-knowledge.md`, `gen-domain-playbook.md`, `run-domain-discovery.md`, `run-mock-discovery.md`, all `domain/*.template.md` and `*.example.md` pairs, `FW-013`, `FW-014`, cross-checked against `FW-016`, `brief.template.md`, `gen-brd-playbook.md`.*

### 1. No test for "is my domain niche enough to warrant the sub-flow"
**File/line:** `gen-domain-knowledge.md:52-66`; `FW-013:64-68`.
The warning names two example buckets (well-known: invoicing, payroll, school fees, HR / niche: regulated, unfamiliar) with no criteria for anything not on the list. "HR" is named well-known, but *HR leave management with jurisdiction-specific statutory entitlements* is a different animal. The PO is left making the exact judgment call the framework exists to remove.
**Fix:** Replace the two example lists with a 3-question self-test: "Can you name the 3 most consequential business rules in this domain from memory, with confidence? Would a wrong assumption here cost >1 sprint to unwind? Is there a regulator/auditor/legal requirement that would reject wrong output?" Any "no/unsure" → run the sub-flow.

### 2. Domain Confidence Score has no defined formula
**File/line:** `domain-discovery-state.template.md:40`; `run-domain-discovery.md:115`.
"Estimate what percentage..." is the entire instruction. The worked example shows 55% after 6/10 sections Covered — not a naive ratio, and never explained why it undershoots. A PO cannot reproduce the number, so cannot independently judge whether a 70% is trustworthy.
**Fix:** Publish the actual weighting (e.g., each section = 10%, but Section 12 Known Variations = double weight since it directly feeds the BRD playbook).
**Downstream gap:** nothing tells the PO what a 70% vs. 95% score means for what to personally double-check before `/gen-brd-playbook`.

### 3. The 70% threshold doesn't scale with stakes
**File/line:** `domain-discovery-state.template.md:13`.
One fixed number governs both a throwaway internal tool and a cross-border-payments compliance domain. Nothing raises the personal bar for regulated domains.
**Fix:** "For regulated/compliance-bearing domains, do not proceed below 90% regardless of the 70% floor; verify Section 9 (Regulatory Baseline) and Section 12.3 (Factual Accuracy Log) are fully resolved."

### 4. "Unverified" facts have a label but no enforced consequence
**File/line:** `gen-domain-knowledge.md:160`; `domain.template.md:240-242`.
The entire enforcement mechanism for a regulatory fact the AI couldn't confirm is a string in a table cell. Nothing requires clearing it before the doc feeds the BRD, and the Step 8 closing report doesn't surface an "Unverified: N" count the way it surfaces Open Issues.
**Fix:** Add an "Unverified facts: [N]" line to the Step 8 report.

### 5. Misclassifying a variable rule as universal is a silent, high-stakes failure mode with no warning
**File/line:** `domain.template.md:116-118`; document-level guide line 18; `gen-brd-playbook.md:63,95`.
BRD discovery questions are generated only from Section 10 (Known Variations) — Sections 5/6 (Universal Business Rules) generate none. A wrongly-universal rule is never asked about, never appears in the BRD unless the client spontaneously contradicts it, and even then is treated as a scope risk, not evidence of misclassification. This is the sharpest finding across the whole audit — see Category B §7 above.
**Fix:** Explicit callout in Section 5's AI Guide: "A rule wrongly marked universal will never be asked about again. When in doubt, put it in Section 10 instead — a variation that turns out universal costs one redundant question; a UBR that turns out variable costs a missed requirement."

### 6. Section 4's generalization filter is the hardest cognitive task in the flow, taught with exactly one example
**File/line:** `gen-domain-playbook.md:91`; `domain-playbook.template.md:82,91`.
Separating "genuinely universal domain concepts" from "project-specific product decisions" has one worked example (invoice templating). Misapplying it in either direction has real cost.
**Fix:** Add 2-3 more worked examples spanning different domain types directly in the template's AI Guide.

### 7. Sub-domain decomposition (FW-014) trigger conditions are under-specified for anything short of obviously-separate portals
**File/line:** `FW-014:53-60`; `domain.template.md:38`.
Clear for admin/teacher/parent/student portals; unclear for e.g. "is Loyalty & Rewards a separate sub-domain from Order Management." Consequences are described in architecture terms a PO won't translate into an observable signal.
**Fix:** Add a concrete rule ("if you can't describe this domain's scope in 2-3 sentences without joining two user roles or two unrelated-lifecycle entities with 'and', split it") and translate the consequence into PO-observable terms ("if a discovery session runs long and answers feel shallow, that's the tell").

### 8. Extension document naming (country vs. sector) has no field that actually decides which name to use
**File/line:** `gen-domain-knowledge.md:166-171` vs. `FW-16:65` (`Extension Adds` is explicitly unstructured free text, not a category).
Three naming conventions are offered as if mechanically selectable, but the only upstream signal is free text. No PO has a way to predict or correct the resulting filename.
**Fix:** Either add an explicit ask in Step 4 ("country-specific, sector-specific, both, or neither?") or state plainly that the AI infers from free text and the PO should verify the filename.

### 9. SDLC's "skip straight to /gen-brd-playbook" contradicts the actual hard gate
**File/line:** SDLC.md framing vs. `gen-brd-playbook.md:26` (hard-gates on a domain knowledge file existing).
`/gen-domain-knowledge` is never actually optional — only its two feeder commands are. A PO following the SDLC framing literally wastes a round trip.
**Fix:** Reword to "skip the discovery sub-flow and go straight from brief to `/gen-domain-knowledge` with AI-knowledge-only."

### 10. Binary yes/no on the discovery sub-flow offers no proportionate middle path
**File/line:** `gen-domain-knowledge.md:45-69`.
A PO who's confident on 80% but unsure on 2-3 specific things has only "trust AI fully" or "run the entire structured session" — nothing lightweight in between.
**Fix:** Offer a third option: "answer a few targeted questions now instead of a full session," letting the AI ask only about its least-confident sections without a persisted playbook file.

### 11. Mock discovery is self-graded by the same model that produced the artifacts being tested
**File/line:** `run-mock-discovery.md` Steps 5-6.
A gap the AI didn't think to include in Known Variations is, by construction, unlikely to be probed for by "the simulated client" either — same blind spot, same source. Never disclosed to the PO.
**Fix:** Add to the Step 8 pass message: "Passing all 5 checks confirms internal consistency — it does not substitute for a human domain expert's own read of Section 10, since both were generated by the same AI."

### 12. "Covered" in the discovery state file conflates "asked" with "adequately answered"
**File/line:** `domain-discovery-state.template.md:184`.
"Substantive" is undefined. A fatigued PO's blanket "yes to all" could get marked Covered the same as a detailed, quoted answer.
**Fix:** "A single blanket 'yes to all' does not qualify as Covered — mark it Partial and note which items still need individual confirmation."

### 13. Content Status vs. Status is only explained in a different file than where the gate lives
**File/line:** `gen-domain-knowledge.md:31` (warns not to confuse the two fields) vs. `brief.template.md:53` (actually explains them).
Minor but real — the file a PO is looking at when blocked doesn't carry the explanation.
**Fix:** One-line addition explaining both fields inline at the gate check.

### Strengths (Domain phase)
- The warning-and-pause pattern before proceeding without discovery (clear trade-off, named examples, safe default, explicit confirmation) is well-designed even though the underlying test needs work.
- The `[Client-Stated]` / priority-rule override mechanism is unambiguous and correctly ordered.
- `domain.example.md` / `domain-playbook.example.md` are detailed, internally consistent, and the Overdue → Overdue (Escalated) correction in `domain-discovery-state.example.md:68-70` is the single best piece of pedagogy in the file set — a real correction shown end-to-end from question to session answer to state file to downstream impact.
- The example state file deliberately shows a 55%, in-progress, two-session state rather than a clean finish — exactly what a first-time PO needs to see.
- Section 12's verification checklist (12.1-12.6) is concrete and checkable, applied with real reasoning in the example.
- The generalization filter's existence is the right instinct, just under-taught.
- Command-to-command handoff is thoughtful about *what* to pass forward (e.g. offering to use live session context over the saved file since it's richer).

---

## Phase 2 — BRD (Stage 2 BRD half, Stage 3)

*Files read: `gen-brd-playbook.md`, `run-brd-discovery.md`, `gen-brd.md`, `review-brd.md`, all `brds/*.template.md`/`*.example.md` pairs, `FW-019`, `FW-024`, `FW-016`, `domain.template.md` §10.*

### 1. No decision rule for live session vs. AI-synthesis vs. skip-to-gen-brd
**File/line:** `run-brd-discovery.md` (whole file) vs. CLAUDE.md's stated AI-synthesis mode for no-client projects.
The command has zero text implementing internal-project mode — no branch on client presence, no instruction to propose an answer first rather than ask cold. A PO on an internal project has to manually improvise being both interviewer and client.
**Fix:** Add a Step 0 branch: "If no external client exists, run in AI-Synthesis mode: propose the domain-default answer first, ask the PO to confirm/correct."

### 2. Requirement Confidence Score formula is ambiguous on its own denominator
**File/line:** `run-brd-discovery.md:110` ("coverage of known-variation topics from domain knowledge Section 10") vs. `:157` ("topics marked Covered / total topics" from the discovery state's Section 3, which also includes journeys).
Materially changes whether a PO hits 85%. Also unstated: does a topic answered "I don't know" flip to Covered or stay Pending?
**Fix:** State explicitly: denominator = all rows in Discovery State §3 (topics + journeys combined); confidence measures coverage of conversation, not resolution of every answer — then separately warn that high confidence with many unresolved Open Questions isn't low-risk.

### 3. "Stuck below 85%" has no prescribed action beyond "schedule a follow-up"
**File/line:** `run-brd-discovery.md:181`.
Coverage (topics asked) and depth (answers actually resolved) are conflated — a client who answers everything with "it depends" can hit 100% coverage with near-zero real confidence, and the scoring mechanic can't tell the difference.
**Fix:** "A topic answered with genuine uncertainty should be marked Pending, not Covered, and logged as an Open Question with an owner." Add a second gate: don't offer to generate the BRD if scope-blocking Open Questions exceed 2, even at ≥85%.

### 4. "Known Variation" vs. "generic question" has no operational threshold where it matters most
**File/line:** `run-brd-discovery.md:108` ("probe immediately" on contradiction) with no rule for whether the contradiction reveals a structurally new domain variation (→ should trigger a domain knowledge update) vs. just an inconsistent client answer (→ resolve and move on).
**Fix:** "If an answer reveals a variation not represented by any existing playbook topic — a new axis of variation, not just an unusual value — flag it as a candidate for a domain knowledge update, not just a BRD Open Question."

### 5. REQ granularity ("one sentence, plain English") has no generative rule, only a retrospective test
**File/line:** `brd.template.md §5:122-145`; `gen-brd.md §5:96-107`; the only operative test is §13.3 ("specific enough a developer could write a story without follow-up questions") — found out only after the fact.
No rule for when multiple facts belong in one REQ vs. several — directly determines epic/story granularity downstream, since `/gen-epics` treats REQ IDs as atomic.
**Fix:** Add a worked *bad* example (over-fat and over-thin versions of the same REQ) and a positive rule: "One REQ = one independently testable, independently story-able capability. An 'and' joining two separately-shippable things → split. A sub-detail of another requirement's behavior → fold in, don't create a new REQ."

### 6. Section 5.21 (Future Capabilities) is referenced three times but does not exist in the template — this is Category A defect #1
**File/line:** `gen-brd.md:130`, `brd.template.md:386,388`, `FW-024:14,34-42` all reference it as real and in production use; `brd.template.md` has no Section 5.21 anywhere, and `brd.example.md` never demonstrates it.
Any PO who reaches the scenario FW-024 describes will be told to "add it to Section 5.21," open the template, and find nothing there.

### 7. Scope (In/Out) vs. Priority (Must/Should/Nice) vs. Layer (Core/Ext) are three independent axes never named as independent
**File/line:** `brd.template.md §1.4:39-45`, `§5:141-145`.
No stated relationship between "In Scope but Should Have" (ships this phase, might get cut under pressure) and "Out of Scope — deferred" (explicitly Phase 2, tracked via REQ-ID per §13.1). No worked example exercises the deferred-item tag format at all. Layer (technical sequencing) and Priority (business value) can conflict with no resolution rule (a Must-Have that's also Ext:PK — built last despite being business-critical).
**Fix:** Add an explanatory subsection near BRD §5's top distinguishing all three axes explicitly, plus one worked deferred REQ with the `[Client-Stated — Phase 2]` tag.

### 8. The BRD→Epic traceability chain is real and mechanical but never told to the PO at BRD-draft time
**File/line:** `gen-brd.md` never states that Section 5 feature-area headings become epics 1:1 (`gen-epics.md:87,147` confirms this is exactly what happens). A PO drafting feature-area boundaries is unknowingly pre-designing the epic list.
**Fix:** One sentence in both `brd.template.md` §5's AI Guide and `gen-brd.md` Step 4: "Each feature-area heading becomes one epic 1:1 — choose boundaries the way you'd choose epic boundaries."

### 9. `/review-brd`'s response table has no "PO uncertain, defer to client" path
**File/line:** `review-brd.md:173-181` — only Confirm/Change/Remove/Too vague, all requiring the PO to personally resolve an `[Assumed]` REQ right now, even when they genuinely weren't on the call that would answer it.
**Fix:** Add a fifth row: "PO uncertain / not equipped to decide on the client's behalf → move to BRD Section 12 (Open Questions) with client as owner."

### 10. Stale cross-reference plus no worked revision example — this is Category A defect #10
**File/line:** `run-brd-discovery.md:189` says record revisions in "Section 5 (Open Issues)" — actually Integration Confirmation; Open Questions is Section 7. The worked example is a clean, zero-contradiction run with no revision ever modeled.

### 11. No circuit breaker for endless discovery iteration with a shifting client
**File/line:** "Resuming across sessions," `run-brd-discovery.md:185-189` — every revision is just recorded and moved past, with no signal for when to stop iterating and escalate.
**Fix:** "If the same topic is revised more than twice across sessions, or a Scope Risk Flag flips more than once, escalate as a scope blocker requiring a decision-maker above the immediate contact."

### Strengths (BRD phase)
- Source tagging (`[Client-Stated]`/`[Domain-Default]`/`[Assumed]`) is one of the best-specified mechanisms in the framework — a PO can learn it from the example alone.
- The Scope Risk Flag mechanic with a verbatim "Response to client" line is excellent judgment support.
- `review-brd.md`'s "framing guide by requirement type" table is a genuinely reusable teaching tool.
- The Layer membership test ("would a UK retailer using this system need this requirement?") is a rare, crisp, repeatable heuristic in a document set where most judgment calls are prose-only.
- Cross-section consistency checks (§13.2) are unusually rigorous.
- `brd.example.md` is genuinely well-built — REQ-002/003, REQ-016 (tagged Assumed), and the Approval Record's "Deviates from domain standard: Yes" are good models of expected specificity.

---

## Phase 3 — Epics & Screen Design (Stage 4, Stage 5)

*Files read: `gen-epics.md`, `review-epics.md`, `epics/*`, `gen-screen-design.md`, `review-screen-design.md`, `screens/*`, `gen-ui-mocks.md`, `mocks/*`, `reference/ascendra-ui/*`, `FW-026`.*

### A1. No guidance on ideal epic count/size — and PROJECT-LIFECYCLE.md claims a check that doesn't exist (Category A defect #6)
**File/line:** `gen-epics.md §4:83-98` gives zero granularity guidance; `PROJECT-LIFECYCLE.md:307` claims `/review-epics` checks "size" and "no gold-plating" — the actual 11 checks in `review-epics.md` contain neither.
**Fix:** Add a rule-of-thumb (e.g. "≈4-10 stories per epic; if a draft would clearly produce more, propose a split at Step 4") and either add the missing checks or correct the claim.

### A2. The planning-table step is the only real pushback point, and it's one unstructured prompt
**File/line:** `gen-epics.md §4:83-98` — a PO is asked to eyeball a 60+ REQ table in one pass, with no signal that this is a coarse check and `/review-epics` is the rigorous one.
**Fix:** One sentence clarifying the division of labor between the two passes.

### A3. Section 8 Notes leaks architecture decisions into an artifact whose entire premise is "no architecture dependency"
**File/line:** `epic.template.md §8:128-132` ("include architectural constraints") vs. document-level guide line 9 ("if it describes how something is built, it belongs in architecture, not here") vs. `epic.example.md:119-121`, which states authorization is "enforced by filtering all Payer queries by `orgId` from the JWT" — a specific implementation mechanism asserted before architecture exists to decide it. Neither the Verification checklist nor `review-epics.md`'s Template Compliance check scope the no-implementation-detail rule to Section 8 — both explicitly exempt it.
**Fix:** Either scope Section 8 to domain/business constraints only, or explicitly state Section 8 is the one place pre-architecture technical hints are allowed and may be revised.

### A4. The capability walkthrough's story-count estimate has no stated basis
**File/line:** `review-epics.md §5C:159` — "approximately [N] stories," with no defined heuristic, presented as fact before `/gen-stories` has ever run.
**Fix:** State the heuristic explicitly (e.g. "one In Scope bullet ≈ one story") or drop the number.

### A5. Dependency ordering is AI-derived with no PO-facing method to double check it
**File/line:** `gen-epics.md §4` item 4 — no falsifiable test for what makes Epic B depend on Epic A.
**Fix:** "Epic B depends on Epic A only if a story in B cannot be implemented or meaningfully demoed without a capability A delivers — not merely 'A is conceptually foundational.'"

### B1. A PO with no ascendra-ui catalog knowledge has almost no way to sanity-check Surface/UI Pattern matches
**File/line:** `screen-design.template.md §3:60-69`; the mechanical check ("does the citation resolve to a real catalog entry") is something only the AI can run against a 1,499-line reference doc — not something a PO reviewing a markdown table can do. The one real PO-facing verification surface (open the mock, does it look right) only checks shape, not whether the citation itself is real.
**Fix:** Give the PO a concrete low-effort check ("ask the AI to show you the exact catalog row it matched — a real match names a specific row, not a category"), or have `/review-screen-design` grep-verify every citation automatically and flag failures before presenting to the PO.

### B2. The "closest:" prefix is an undocumented escape hatch used in the canonical example itself
**File/line:** `screen-design.example.md:42,44` uses `closest:` on two of five rows — meaning "not actually the right entry, nearest approximation" — with no definition anywhere of when this is legitimate vs. a quiet rigor downgrade.
**Fix:** Formally define `closest:` (when legitimate, require a one-line reason) or eliminate it and fold any imperfect-fit caveat into Notes instead.

### B3. What's locked at Screen Design vs. what Architecture can still change is asserted uniformly, but the practical boundary is fuzzy
**File/line:** `screen-design.template.md:8-10` — Sections 1-3 are carried over verbatim; Section 4 (Data Requirements) merely "feeds" architecture. Nothing addresses what happens when Architecture discovers Section 4's data shape was structurally wrong (new entity implied) — is that a normal refinement or does it invalidate the Screen Design approval?
**Fix:** One line distinguishing firmness: Sections 1-3 locked; Section 4 a best-effort forecast, and getting it wrong during Architecture is a normal refinement, not a regression requiring `/assess-change`.

### B4. Portal merge/split criterion is thin at the edges a PO will actually hit
**File/line:** `screen-design.template.md §1:42` — "same app, navigation differs only by visibility" doesn't address personas sharing 90% of nav but with genuinely different auth flows (SSO vs. magic link).
**Fix:** "Two personas sharing the same nav shell but entering via genuinely different auth flows are two portals, even if nav items are otherwise identical."

### C1. Two-pass mock generation is mechanically well-explained but the *why* lives only in an ADR
**File/line:** `gen-ui-mocks.md:3-8` states what triggers each pass; the reasoning (data-model correctness depends on real screen data needs; Section 6 genuinely doesn't exist yet) lives only in `FW-026`, not in the files a PO is actually looking at.
**Fix:** One sentence in `gen-ui-mocks.md`'s intro carrying the rationale forward.

### C2. `gen-ui-mocks` doesn't read `hard-instructions.md` — Category A defect #7, self-admitted in FW-040
See Category A table above.

### D. Terminology: Portal/Screen/Surface/UI Pattern — each defined at first use, never collected, and "Screen" itself is never explicitly defined
**File/line:** No glossary exists anywhere; "Screen" is used as self-evident but is simultaneously the inventory table's name, a row key, and (implicitly, never stated) a superset of Surface — every row, Page or Dialog alike, is "a Screen" with equal weight, but this equivalence is never stated as a rule.
**Fix:** A short terminology block: Portal = nav shell; Screen = one Section 3 row regardless of weight; Surface = container mechanism; UI Pattern = internal content shape, independent of Surface.

### E1/E2. Both epic and screen-design examples demonstrate only the easy case
`epic.example.md` is entirely single-layer/Phase-1/no-REQ-splits despite the AI Guide repeatedly warning about cross-epic REQ splits and layer mixing. `screen-design.example.md` is single-portal, no Drawer, no Tabs, despite both being called out with specific criteria in the guide.
**Fix:** Add a second worked example to each showing the harder case (an Ext-layer epic depending on a named Core epic; a two-portal project with a Drawer and a legitimate Tabs case).

### F1/F2. Tier 3 ("PO decides") gives no decision criteria; targeted PO questions are themselves generic despite instructions not to be
**File/line:** `review-epics.md:190-198,202-211`; `review-screen-design.md:113-119`. Tier 3 flags a fork with no rails for how to decide. The four example targeted questions are fully generic and would fit any epic verbatim — with no worked before/after showing what a genuinely epic-specific question looks like.
**Fix:** One sentence of decision guidance per Tier 3 category; a worked before/after question pair.

### Strengths (Epics & Screen Design)
- The base/patch mode determination table for mocks is unambiguous and self-diagnosing.
- FW-026's "Why This Was Needed" reasoning is unusually well-documented, with a grep-verified finding as evidence.
- The Surface/UI Pattern citation discipline is correctly motivated, and the one Correct/Incorrect pair in the template is genuinely clarifying — more sections should use this pattern.
- `review-screen-design.md`'s instruction to open the mocks alongside the table is the single best PO-accessibility mechanism in this phase — routes around the catalog-knowledge problem by giving the PO something they *can* judge.
- `hard-instructions.md`'s append-only, dated, source-cited entry format is excellent institutional memory design.
- Resumability ("find the first Pending row") is handled consistently and well across both review commands.

---

## Phase 4 — Architecture (Stage 6)

*Files read: `gen-architecture.md` (full), `review-architecture.md`, `architecture/*`, all three `standards/*.template.md`, `reference/architecture/*`, `reference/api-contract/contract.md`, `reference/form-validation-messages.md`, `FW-025`, `FW-033`, `FW-034`, `FW-040`, `FW-041`, `FW-042`, `FW-045`.*

### 1. CLAUDE.md tells the PO to prepare standards before running the command — the command itself treats an empty folder as the default path
**File/line:** CLAUDE.md Workspace Setup step 5 vs. `gen-architecture.md:59-61,147` — pre-authored files are used as-is if present, but Steps 4-7 derive everything conversationally by default when the folder is empty. A first-time PO following CLAUDE.md literally will stall trying to write 8+ documents from an abstract template with no worked example.
**Fix:** State explicitly that pre-authoring is optional and exists only to lock in a non-default choice; name the one or two files actually worth hand-authoring.

### 2. No worked `.example.md` for any standards template — Category A defect #11
`technical-standard.template.md` is pure placeholder shape with no worked instance anywhere, unlike every paired architecture/domain reference document.

### 3. Step 7's standards-generation confirmation shows filenames, never content — the real judgment calls happen off-screen
**File/line:** `gen-architecture.md:262-282` — the PO's one confirmable moment is a `File | Tier | Why` table. Actual rules (actor-identity shape, JWKS vs. shared secret, exception-filter pattern) are generated after, with no further checkpoint. `/review-architecture` never quotes or summarizes a rule from these files directly to the PO — only their downstream structural effect.
**Fix:** Surface one plain-language "the one non-default judgment call this file makes" sentence per generated standards file during review.

### 4. Deviation detection (Section 9) is self-referential on a first run
**File/line:** `arch.template.md:631` — precisely defined ("differs from `system-architecture.md`"), but that baseline file is written from the same PO confirmation minutes earlier (one dense 12-row table). The mechanism structurally cannot catch a bad choice rubber-stamped in that same confirmation — only later drift.
**Fix:** Surface a "why this and not X" one-liner per row in the Step 5 table by default, not only on request.

### 5. The Actor Identity Shape derivation is the densest judgment call bundled into a single compound prompt
**File/line:** `gen-architecture.md:109,118,122-129` — a consequential, hard-to-reverse schema decision (every Master table's audit-column shape, project-wide) folded into one clause of a four-question compound prompt, citing reasoning that lives in an 85-line ADR (`FW-037`) never surfaced in-command.
**Fix:** Pull this into its own standalone confirmation with a two-sentence plain-language practical explanation.

### 6. `arch.example.md` is stale against `arch.template.md` — Category A defect #8
See Category A table. Missing Section 3.1.3, Section 6.5 (FW-041), correct actor columns (FW-037), `If-Match` headers (FW-045); section numbering drifted.

### 7. The 35-check review is well-scaffolded to use, structurally impossible to audit
**File/line:** `review-architecture.md` Concerns A-E — genuinely good UX (plain-language framing before checks, one targeted question per concern), but the 35 checks themselves run entirely off-screen with only a pass/fail summary ("No flags — all Concern B checks passed"). A PO cannot verify the claim without redoing the AI's own cross-referencing work.
**Fix:** Print the actual check list (even abbreviated) alongside the summary so a skeptical PO has something concrete to spot-check.

### 8. Screen Design carry-over and fresh Architecture content are mixed in the same review section without telling the PO which is which
**File/line:** `gen-architecture.md:492,494` correctly distinguishes 3.1.3.1-3.1.3.3 (carried over) from 3.1.3.4-3.1.3.5 (fresh) internally, but `review-architecture.md`'s Concern A framing never states this to the PO — risking either re-litigating already-approved content or skimming past the two genuinely new subsections.
**Fix:** One sentence flagging the boundary explicitly in the Concern A framing script.

### 9. Core/Extension methodology is well-written but the PO is never told to go read it
**File/line:** `reference/architecture/layered-domain-architecture.md` is genuinely PO-accessible (a crisp "One Rule," a testable "Membership Test," a fully worked example) — but every reference to it is AI-facing only. The PO-facing Extension Points table is a terse two-column citation with no pointer to the readable methodology that could let them apply the same test themselves.
**Fix:** Invite the PO directly to read and apply the Membership Test themselves when reviewing the Extension Points table.

### 10. Section numbering is never mapped as "PO-reviewable vs. must-trust"
**File/line:** No document states which sections a PO can genuinely evaluate (plain-English narration) vs. which are effectively unreviewable without engineering background (raw Drizzle/DTO code blocks) — `review-architecture.md`'s framing prose implicitly does this translation by never quoting code, but never says so as a deliberate design choice.
**Fix:** One paragraph stating explicitly that the PO isn't expected to read raw code blocks — the framing and FLAGS are the reviewable surface.

### 11. Composable capability model detection is a silent, one-shot heuristic with no confirmation of its own
**File/line:** `gen-architecture.md:576` — a significant structural fork (JWT shape, RBAC shape, every guard downstream) decided by pattern-matching BRD wording ("independently assignable"), surfaced to the PO only after the fact as settled context, not as a decision point.
**Fix:** Explicit "I detected X — correct?" confirmation, matching the pattern already used well for comparably consequential forks elsewhere in the command.

### 12. The lock confirmation gap — Category A defect #12
`PROJECT-LIFECYCLE.md` itself flags this as unresolved. See Category A table.

### 13. "No undocumented deviations" has no standing PO-facing mechanism — every known instance was caught by manual code review, not a structural gate
**File/line:** The ADR trail (`FW-033`, `FW-041`, `FW-040`, `FW-034`) shows the same pattern repeatedly: a standard says X, code does something else, discovered only by a PO manually reading source or by `/verify-story` hitting a real failure. No command's job is "confirm running code still matches the Locked architecture" on an ongoing basis — `/implement-story`'s embedded self-check runs invisibly, once, per story, at implementation time only.
**Fix:** Either document explicitly that manual PO code review is the intended backstop, or add a lightweight periodic conformance check (even a coarse grep-based one).

### Strengths (Architecture)
- `layered-domain-architecture.md` + its worked example is the single best-explained artifact in the whole audit — a genuinely PO-accessible test, traced through a real INSERT sequence.
- `review-architecture.md`'s "Frame this concern" pattern is a real UX achievement for a non-architect audience.
- The Tier 1-4 fix-tier system correctly scopes where PO attention is actually needed.
- `reference/api-contract/contract.md` grounds its design in real RFCs rather than inventing bespoke shapes, and explicitly draws a boundary around what's fixed vs. per-project.
- Section 9's deviation definition is precise and testable, not vague — its only limitation is what it structurally cannot catch (finding 4), not the definition itself.
- Step 4's "present what you derived, ask only what can't be inferred" pattern reduces PO cognitive load well.
- The FW-025 defaults table gives a genuine mental model of when the framework deviates from its own defaults and why.

---

## Phase 5 — Sprint Planning & Development (Stage 7, Stage 8)

*Files read: `gen-stories.md`, `review-stories.md`, `gen-sprint-plan.md`, `gen-story-plan.md`, `implement-story.md`, `verify-story.md`, `gen-pr-description.md`, `stories/*`, `story-plans/*`, `sprints/*`, `pr/*`, `reference/estimation/*`, `FW-031`, `FW-011`, `FW-029`, `FW-030`.*

### 1. Estimation model choice — real guidance exists, but a silent gap between the reference file and the PO-facing prompt
**File/line:** `reference/estimation/time-based.md:5` explicitly self-disqualifies for AI-implemented projects ("AI implementation speed does not map linearly to human hours") — since every project here is AI-implemented, this is effectively always disqualifying. But `gen-stories.md §1.6:47-55` presents all three models as parallel legitimate options with a terse one-liner, so the caveat only surfaces if the PO reads the full reference file *after* choosing — the wrong order.
**Fix:** Surface the caveat inline in the Step 1.6 prompt itself for option (c).

### 2. Wave-vs-epic boundary — the single biggest gap in this phase; the underlying artifact isn't actually specified to exist
**File/line:** Five files (`gen-stories.md:80`, `review-stories.md:288`, `gen-epics.md:148`, `SDLC.md:156`, `PROJECT-LIFECYCLE.md:416,423,427`) all treat a "story-level interleaving note" in `epics/index.md` Section 3 as reliable and checkable. But `gen-epics.md` Step 6 — the only place `epics/index.md` is written — never instructs the AI to detect or produce such a note, "interleav" appears nowhere in `gen-epics.md`/`review-epics.md`, and no `epics/index.md` template exists at all. This is Category A defect #5 — not a PO trust issue, a genuine spec gap.
**Fix:** Add explicit trigger criteria and markdown shape for the note to `gen-epics.md` Step 6.

### 3. Sprint Assignment logic (`/review-stories` Part F) is one of the strongest-explained mechanisms in the framework, with one gap
**File/line:** `review-stories.md:284-310` — transparent math, sourced capacity, honest handling of leftover unassigned stories. The "above target" split-boundary branch gives no worked heuristic for *where* to split, unlike the estimation references which all give a concrete "find the seam" rule.
**Fix:** Point back to the split-trigger guidance already in `standards/estimation.md`.

### 4. The four-command chain's purpose is well-argued in the ADR, invisible in the command itself
**File/line:** `FW-031:45` explains precisely what wrong interpretations the story-plan step is meant to catch (auth guard names, tenant-scoping, screen routes) — but `gen-story-plan.md §9:121-132`'s confirmation prompt is a bare "does this look right?" with no checklist porting that reasoning into the moment the PO actually needs it.
**Fix:** Add a short "what to check before confirming" list to the Step 9 prompt itself.

### 5. Repo scaffolding-on-first-touch is thoroughly specified once inside the command, invisible before it
**File/line:** `implement-story.md §3:101-186` is careful and well-gated once triggered, but nothing upstream (story plan confirmation, gate-check messages) warns the PO in advance that the first `/implement-story` run on a project also scaffolds an entire repo, logging, exception handling, and an auth module — none of which appears in the story's own ACs.
**Fix:** One line in the gate-check success path when no repo exists yet: "this is the first story touching {repo} — expect a larger first diff than usual."

### 6. Recovery loop is asymmetric: implementation failure has a clear bound, verification failure doesn't
**File/line:** `implement-story.md:425` gives a clean two-attempt ceiling before an explicit stop. `verify-story.md:225`'s failure message ("resolve these, re-run") doesn't say *how* — re-running `/implement-story` hits a status gate (story is `In Progress`), leaving the PO to intuit an off-framework fix, undercutting the "nothing invented ad hoc" philosophy elsewhere.
**Fix:** Explicit recovery instruction: fix directly on the same branch without re-running `/implement-story`; if the same AC fails twice, suspect the plan and regenerate it.

### 7. Story sizing guidance is thorough for the "too big" case, reactive rather than proactive for shaping the initial breakdown
**File/line:** `tshirt-sizing.md:85-90` gives a target sprint composition skewed toward S/M, but nothing in `gen-stories.md` Step 4 (before any file is written) prompts the PO to weigh scope boundaries against that composition — sizing happens per-story only after the list is fixed, so corrective splits happen reactively at `/review-stories` rather than proactively at breakdown time.
**Fix:** "If more than 2 stories in this table look L/XL at this stage, propose splits now, before generating files."

### 8. Terminology: wave/sprint/epic well-distinguished in the reference docs, genuinely confusing in command output itself
**File/line:** `SDLC.md`/`PROJECT-LIFECYCLE.md` get the distinction right on first use; but `review-stories.md:290` says "this wave" once then switches entirely to "Sprint {NN}" with no re-anchoring, and `gen-stories.md`'s Step 8 report never uses "sprint" at all.
**Fix:** A one-line parenthetical the first time each command uses "wave" in its own output.

### 9. Three concrete template/command drift points
- **(a)** Story template's tie-break rule for two personas needing the same capability is never exercised in any of the three worked examples.
- **(b)** Story-plan template's Web Plan "Data layer" field is hardcoded to React Query (`story-plan.template.md:87`), while `gen-story-plan.md:83` explicitly requires conditional language for hookless approaches — the template is less careful than the command that's supposed to follow it, and no worked hookless example exists either.
- **(c)** PR description template's Pre-Merge Checklist hardcodes stack-specific assumptions (`orgId` from JWT, React Query cache invalidation) that FW-029/FW-030 explicitly ban from unconditional framework rules.
**Fix:** Add the missing worked example (a); align the template's field language with the command's conditional instruction (b); apply the FW-029/FW-030 conditioning pattern to the PR checklist (c).

### Strengths (Sprint Planning & Development)
- FW-031's extraction-vs-invention framing and the hard-gate-no-override rule are well-justified, not dogmatic — the ADR explains precisely why this gate is stricter than others.
- The fix-tier system in `/review-stories` is an excellent piece of judgment-call scaffolding and should be the template other reviewable-artifact steps are made to match.
- Stack/project agnosticism (FW-029/FW-030) is executed thoroughly throughout `implement-story.md`, not just declared.
- The Step 3 structural repo-existence check (refusing to build on a directory that merely exists) is real defensive design.
- Sprint Assignment's three-branch design is honest about uncertainty and surfaces leftover unassigned stories rather than losing them.

---

## Phase 6 — QA/UAT/Release/Support & Cross-Cutting Commands

*Files read: `gen-uat-checklist.md`, `close-sprint.md`, `gen-release-notes.md`, `assess-change.md`, `update-status.md`, `project-status.md`, `init-project.md`, `sprints/*`, `releases/*`, `test-reports/*`, `reference/defect-severity.md`, `PROJECT-LIFECYCLE.md` (full), `conventions/*`, `FW-001`, `FW-002`, `FW-024`.*

### 1. `/assess-change` — best-documented command in the set, with one real gap
**Pattern 2 (Gap, informal) vs. Pattern 3 (Scope Shift, formal)** have no decision heuristic — both are "new epic content, no BRD change," differentiated only by whether it was "already implied," a subjective call. Contrast `FW-024`'s crisp Parking Lot test. Under time pressure a PO will default to the cheaper Pattern 2 even when it's really Pattern 3, eroding the formal-change discipline the pattern system exists to enforce.
**Fix:** "If you can point to the specific BRD sentence that already implies this behavior, it's Pattern 2. If you're inferring intent rather than citing text, it's Pattern 3."
Also: the Promotion Gate (Parking Lot items) triggers mid-session with no upfront check — a PO burns a full read-classify cycle before discovering they needed to check BRD Section 15 first.

### 2. `update-status` — no single reference for inline vs. manual, even inside the command meant to answer that
**File/line:** `update-status.md` itself contains no disambiguation table; it's scattered across five separately-phrased write-ups in `PROJECT-LIFECYCLE.md`. Even the Gate Summary table only shows the Approved-by-inline row — it doesn't flag the other transitions on the same artifact (Epic → In Progress/Done, Story → Merged, Sprint → Active/Complete) that are manual-only, risking a PO wrongly assuming all status changes on a gated artifact are automatic.
**Fix:** Add a short "handled automatically" vs. "always requires this command" table to the top of `update-status.md` itself.

### 3. `/project-status` ATTENTION section — one generic follow-up sentence doesn't map to all five example triggers
**File/line:** `project-status.md:127-157` — two of the five triggers (sprint slip risk, stale In Progress epic) have no defined remedy via either of the two commands the single follow-up sentence points to.
**Fix:** Give each ATTENTION category its own one-line recommended action.

### 4. Hotfix Cycle merge-back — Category A defect #3
See Category A table — references a "sprint branch" the documented git model never creates.

### 5. `/init-project` — genuinely strong Step 2 (project boundary), but the code-format validator contradicts its own governing ADR
Category A defect #4 — 4-segment extension codes rejected as errors despite being FW-008's own canonical example.

### 6. FW-001/FW-002 — terminology drift and a self-admitted partial-implementation gap that never surfaces to the PO
Category A defect #13 (FW-001 naming) plus: `FW-002:7` explicitly states `/gen-brd`, `/gen-architecture`, `/gen-sprint-plan`, `/gen-release-notes`, and `/update-status` do NOT yet auto-maintain the brief's Section 9 Related Artifacts index — meaning it's known-stale for most artifact types. Nothing in `CLAUDE.md` or `PROJECT-LIFECYCLE.md` carries this caveat forward to where a PO would actually see it.
**Fix:** Surface FW-002's partial-implementation caveat wherever Section 9 is mentioned in PO-facing docs.

### 7. Gate Summary table gap — Category A defect #9
See Category A table — Story Plan Confirmed gate entirely missing.

### 8. "Content Status vs. Status" isn't actually a recurring pattern — worth correcting the framing
Only `brief.md` has two distinct status fields; every other gated artifact (BRD, Architecture, Epic, Story) uses one field for both readiness and gate state. Nothing states this explicitly, so a PO who just learned the two-field split from the Brief could reasonably (wrongly) look for the same split elsewhere.
**Fix:** One sentence in `update-status.md`'s intro: "No other artifact type uses a two-field split — Brief is the only exception."

### 9. Test execution report vs. individual verify-story reports — genuinely well-explained, no confusion found
Three independent places (template AI Guide, `PROJECT-LIFECYCLE.md` Step 23, `artifact-naming-conventions.md`) consistently state the aggregation relationship, optionality, and trigger. Minor gap: "when the sprint needs a formal, client-facing record" has no explicit permission-to-skip statement for internal projects.

### 10. Defect severity self-contradiction — Category A defect #2, highest-impact finding in this phase
See Category A table — direct contradiction on whether High defects can be deferred, with the actual sign-off checklist operationalizing the reading the rule text denies.

### Strengths (QA/UAT/Release/Support & Cross-Cutting)
- `/assess-change`'s blast-radius table and per-pattern formality rules, with concrete Document Control snippets and a story-state-rules table, is the strongest piece of judgment-call scaffolding in the whole framework — a model for the rest.
- `/init-project` Step 2's project-boundary question puts the hardest early judgment call directly in front of the PO with a worked disambiguation rule.
- The UAT checklist's AC-to-scenario translation guide gives a real before/after example applicable mechanically.
- The "What this document is / is not" block on every template consistently heads off cross-artifact confusion (UAT checklist vs. test execution report vs. release notes vs. defect log) — a single convention doing a lot of quiet work.
- `PROJECT-LIFECYCLE.md`'s self-auditing honesty (flagging its own drift rather than presenting stale content as settled) is good practice — the fix is making these flags visible from the command files a PO actually uses, not only from a 67KB cross-reference doc or a buried ADR.
- `update-status.md`'s per-artifact status tables are genuinely comprehensive once you're inside the command — the gap is discoverability of *whether* you need it, not clarity once there.
