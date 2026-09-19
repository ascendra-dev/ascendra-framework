# QA & UAT

Covers Stage 9 (Testing & QA) and Stage 10 (UAT) in [`SDLC.md`](../SDLC.md): QA confirmation, [`/gen-uat-checklist`](../.claude/commands/gen-uat-checklist.md), and defect handling via [`reference/defect-severity.md`](../reference/defect-severity.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't. This chapter picks up exactly where [`08-development.md`](08-development.md) leaves off — every sprint story `Merged`, every PR closed — and covers the two gates between "code is merged" and "sprint is Complete."

---

## Purpose

Stage 9 and Stage 10 are two separate gates, not one step with two names, and the distinction is worth holding onto before you read further. Stage 9 (QA) asks a technical question: *does the merged code actually do what the stories said it would do, measured against the automated test suite and the `/verify-story` reports already produced during development?* Stage 10 (UAT) asks a different question entirely: *does the running system, operated by the Product Owner's (or client's) own hands, actually deliver the business outcome the BRD promised?* A sprint can clear Stage 9 cleanly — full suite green, zero open Critical/High, coverage threshold met — and still fail Stage 10, because a feature can be technically correct against its acceptance criteria and still feel wrong, be missing an obvious step, or contradict something the client actually meant when the requirement was written. Collapsing the two into one step means either skipping the technical confirmation (UAT starts on code nobody has confirmed is stable) or skipping the business confirmation (a sprint ships on `/verify-story` passes alone, with no one but the AI ever having operated the actual feature). Keep them separate on purpose.

---

## Defect severity and the deferral rule, worked

`/verify-story` classifies every defect it finds during Stage 8 against [`reference/defect-severity.md`](../reference/defect-severity.md); the Product Owner does the same for anything found fresh during UAT. Four levels, briefly:

| Severity | What it means | Blocks release? |
|---|---|---|
| **Critical** | System unusable, data at risk, or a security control bypassed. No workaround. | Always — blocks UAT sign-off and production deployment, no exceptions |
| **High** | A core feature is broken with no viable workaround — the user cannot complete a primary task. | Blocks UAT sign-off unless formally deferred (see below) |
| **Medium** | Feature is degraded but a workaround exists. | Does not block sign-off; must be documented if a test execution report is produced |
| **Low** | Cosmetic or minor. No functional impact. | Does not block any gate |

Two automatic escalations worth remembering because they're easy to under-apply in the moment: any defect found in production is escalated one severity level (Medium → High, High → Critical), and any security-related or cross-tenant data-access defect is always Critical regardless of how small the functional impact looks. Don't classify a cross-tenant leak as Medium because only one field was exposed — the rule doesn't have a proportionality exception.

### The Deferred Defect Process — the current, corrected rule

This is the part worth slowing down for. A defect can be set to **Deferred** only when every one of these four conditions holds:

1. It is **Medium or Low severity**, **or** it is **High severity with the explicit Product Owner agreement required by condition 3** — **Critical defects can never be deferred, under any condition or agreement.**
2. If the project has an external client, the client has been **informed** of the defect and its deferral.
3. The **Product Owner has approved** the deferral — and for a High-severity defect specifically, that approval must be **explicit and individually recorded against that one defect**, not covered by a blanket sprint sign-off.
4. A **target sprint** for the fix is agreed and recorded in the defect log.

Read condition 3 literally: a High defect does not get deferred by the PO signing the UAT checklist's overall sign-off line at the bottom, or by approving the sprint as a whole with the defect merely listed somewhere in it. It gets deferred by the PO looking at *that specific defect* and recording, against *that defect's own entry*, a decision to defer it — with a reason. The QA sign-off checklist in [`test-execution-report.template.md`](../projects/TEMPLATE/test-reports/test-execution-report.template.md) Section 5 operationalizes this exactly: its one exception line reads *"a High defect can be overridden to Deferred status only by explicit Product Owner decision — record it in the Defects Found table with Status 'Deferred' and a note of who made the decision and when."* That's condition 3, mechanically enforced at the row level, not the document level.

**A worked contrast, both against the same kind of defect — a broken export button:**

- **Correctly deferred:** DEF-14, High — "Export to CSV on the Overdue Report screen returns a 500 error." The underlying data is fully visible and actionable on-screen (the Overdue Report table itself works; only the export action fails), the client's own Finance Manager has been told directly and has said the CSV export isn't needed until month-end close, three weeks out, and the PO records against DEF-14 specifically: *"Deferred — Finance Manager confirmed CSV export isn't used until month-end (in 3 weeks); on-screen data is fully usable as a workaround in the meantime. Target: Sprint 6. — Z. Shah, 2026-07-18."* That's every one of the four conditions met, individually, for this one defect.
- **Should not be deferred:** DEF-15, High — "The Void Invoice action silently fails for invoices with more than 10 line items; no error is shown, the invoice stays in Sent status, and the Finance Manager has no way to know the void didn't happen." There's no viable workaround (the user has no signal the action failed at all — they can't even manually correct it because they don't know it's wrong), and deferring it risks a client sending a collections notice on an invoice they believe is voided. Even if a PO is tempted to defer it because the fix touches a tricky bulk-line-item code path, condition 1's "explicit PO agreement" isn't a rubber stamp — it exists to force the PO to actually weigh the risk, and a silent, undetectable failure on a money-moving action is exactly the kind of High defect that should fail that weighing test.

The difference between the two isn't the severity label — both are High. It's whether a real workaround exists and whether the PO can honestly say, in writing, why shipping without the fix is safe for the time between now and the target sprint. If you can't write that sentence honestly, don't defer it — fix it now.

---

## When the full test execution report is warranted

*(A process decision about which artifact gets produced at all — not a check against content already on the page.)*

Stage 9's own guidance says the fuller [`test-execution-report.md`](../projects/TEMPLATE/test-reports/test-execution-report.template.md) is "required for client-facing projects" and the template says "generate this fuller report only when the sprint needs a formal, client-facing QA record." Neither states a crisp trigger, and the framework's own audit flagged this as the one real gap in an otherwise well-explained relationship (the aggregation relationship between the report and individual `/verify-story` reports, and the report's optionality, are all consistently stated elsewhere — it's specifically the *when* that's missing an explicit permission-to-skip).

**The trigger test:** ask two questions.

1. **Is there an external client or stakeholder who will actually see quality evidence for this sprint** — someone outside the delivery team who expects to be shown, not just told, that testing happened?
2. **Is there a compliance, contractual, or audit reason a formal QA record has to exist** — a regulator, an audit trail requirement, a contract clause naming test evidence as a deliverable?

If the answer to both is no — a purely internal project, or a client-facing project where the client has never asked to see QA artifacts and none are contractually owed — the one-line confirmation in the sprint plan's Sprint Notes section is sufficient. Write it, move on. Producing a six-section formal report nobody on the client side will ever open is ceremony, not QA — it costs real PO time synthesizing a document, and that time comes from somewhere.

If the answer to either is yes, produce the full report. It's the gate artifact UAT actually reads in that case: `/verify-story` reports alone give you per-story detail with no sprint-wide rollup, and a client or auditor asking "was this sprint tested" wants the aggregated answer — pass rate, defects, blocked items, one sign-off recommendation — not six separate story-level files to reconcile themselves.

One thing worth being honest about either way: skipping the full report doesn't mean skipping Stage 9's actual exit criteria (below). The one-line confirmation is a lighter *artifact*, not a lighter *bar* — the full test suite still has to pass, Critical/High defects still have to be at zero (or properly deferred), and coverage still has to clear the project's threshold before that one line gets written.

---

## UAT scenario density — turning an AC into something a client can run

`uat-checklist.template.md` already contains the right teaching example — worth pulling out directly and explaining why it works, because you'll be writing this translation yourself the first time you hit an AC the template's own worked case doesn't cover.

**The AC, as written for engineers:**

> "The system rejects a payment attempt when the invoice status is not Open."

**The UAT translation:**

> 1. Navigate to the Payments screen.
> 2. Find an invoice that has already been paid (status: Paid).
> 3. Attempt to record a new payment against it.
> 4. Submit the form.
>
> **Expected result:** An error message is displayed. The payment is not recorded. The invoice status remains unchanged.

Three things make this translation work, and all three generalize to any AC you translate yourself:

- **Numbered, single-action steps.** Each line is one physical action a non-technical person can perform without interpretation — "Navigate to X," "Find Y," "Submit." Nothing in the steps requires the reader to understand *why* the system behaves this way, only what to click and type. Compare that to a step like "trigger the status-guard validation path" — technically accurate, useless to a PO or client running the test.
- **A concrete, reproducible test condition, not an abstract one.** The AC says "when the invoice status is not Open" — a category. The UAT steps pick one concrete instance of that category (an already-Paid invoice) and tell the tester exactly how to get their test data into that state. A tester left to invent their own "an invoice that's not Open" might pick a Draft invoice instead of a Paid one and exercise a completely different code path than the one the AC is actually about.
- **An expected result that's observable, not internal.** "An error message is displayed. The payment is not recorded. The invoice status remains unchanged" — every clause is something the tester can literally see on screen or verify by re-opening the invoice. Nothing says "the API returns a 409" or "the service throws a BusinessRuleViolationException" — that's real, correct engineering detail, but it's invisible to the person running UAT and would make the checklist untestable by exactly the audience it's written for.

The template's own rules generalize this pattern for the cases that don't map cleanly onto "click a button, see a message": a scheduled-job AC gets written as what the user observes the *next business day*, not how the job runs; an email/notification AC gets written as what the *recipient receives*, not how the email is sent. The underlying principle in both cases is the same as the payment example — describe the world as the tester can observe it, never the mechanism that produced that world.

When you're writing UAT steps yourself for a new AC, run this check before you consider it done: could someone who has never seen the codebase, and doesn't know what an API or a database is, execute every step and judge Pass/Fail purely from what's in front of them on screen? If any step or any clause of the expected result requires technical knowledge to act on or evaluate, rewrite it.

---

## The UAT fail-and-return loop — what it actually costs

State the cycle plainly, because "it loops back" undersells what actually happens: a Critical or High defect found during UAT sends the *affected story* — not the whole sprint, but everything about that story's fix cycle — back through the same sequence it already went through once: Stage 8 (fix the code, re-verify with `/verify-story`, re-raise and re-approve the PR), then Stage 9 again (the full suite has to pass again, the defect count has to clear again), then Stage 10 again (a new or updated UAT checklist scenario has to be re-run and re-signed).

That's not a quick patch-and-continue. It's the entire development-to-QA pipeline re-run for that story, and it costs real calendar time on top of the engineering time: a fix has to be written and reviewed, `/verify-story` has to re-execute every AC for that story (not just the one that failed — the whole story's AC set, because a fix can regress an AC that previously passed), the PR cycle repeats, Stage 9's suite-wide checks re-run project-wide (not just for the one story), and only then does UAT resume — and UAT resuming often means re-running the *whole checklist* from the point the PO left off, not just the one failing scenario, because a fix landing mid-checklist can change behavior the PO already marked Pass earlier in the same session.

**Why this matters for sprint sizing:** if a PO budgets UAT as "half a day at the end of the sprint," a single Critical finding on day one of that half-day can consume the rest of the sprint's remaining calendar time — the fix-verify-PR-QA-UAT cycle for one story routinely takes longer than the entire first UAT pass did. This is the concrete reason to budget UAT with slack at the end of a sprint rather than scheduling it flush against a hard release date, and it's also the practical argument for taking Stage 9's exit criteria seriously before UAT starts at all (next section) — every Critical or High defect Stage 9 catches and resolves *before* UAT begins is a full fail-and-return cycle you never have to pay for.

---

## What QA confirmation actually requires before UAT can start

Stage 9's exit criteria, stated in [`SDLC.md`](../SDLC.md), are three things: full test suite passing, zero open Critical or High defects, and coverage threshold met. The first two are self-explanatory. The third one is easy to read past without knowing where the number actually comes from — and it's worth being explicit about this, because a PO reading only the command files (`gen-uat-checklist.md` never mentions coverage at all) could reasonably assume "80%" is a framework-wide constant.

It isn't. **The coverage threshold is the project's own number, defined in `projects/{CODE}/standards/test-strategy.md`.** SDLC.md's stated "≥ 80%" is a *target default*, not a fixed rule — it's the number the framework suggests when a project's test strategy doesn't say otherwise, but the actual gate for any given project is whatever that project's own `test-strategy.md` says. If your project's test strategy sets a stricter bar (say, 90% for a payments-adjacent codepath) or a looser one for a low-risk internal tool, that project-specific number is the one Stage 9 actually checks against — not the SDLC.md default. Before you sign off QA confirmation on a sprint, know which number applies to *this* project, not the framework's generic figure, and go read `standards/test-strategy.md` yourself if you're not certain.

Put the three exit criteria together and Stage 9's actual job is: confirm the automated suite is green, confirm every `/verify-story` report for the sprint shows zero open Critical/High (or, for High, a properly-deferred entry per the rule above), and confirm coverage clears *this project's* threshold — then record that confirmation, either as the sprint plan's one-line note or the full test execution report, per the trigger test above. Only once that's recorded does Stage 10 formally begin.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md). New terms specific to this phase:

- **Deferred (defect status)** — a defect that has been allowed to cross the UAT sign-off gate without being fixed, under the four-condition process above. Never available to Critical defects; available to High only with individually-recorded PO agreement against that specific defect, not a blanket sprint sign-off.
- **QA confirmation** — the Stage 9 checkpoint itself (full suite passing, zero open Critical/High, coverage threshold met), recorded either as a one-line sprint plan note or the full test execution report. Not the same thing as UAT sign-off, which is a separate, later confirmation by the Product Owner operating the running system.
- **Coverage threshold** — a per-project number from `standards/test-strategy.md`, not a framework-wide constant. SDLC.md's "≥ 80%" is the default suggestion, not a fixed rule every project is held to.

---

## Common mistakes

- Treating a High defect's deferral as approved because the PO signed the sprint's overall sign-off line, instead of recording an individual, explicit decision against that specific defect.
- Deferring a High defect where no real workaround exists, just because the fix is inconvenient to schedule this sprint.
- Producing the full `test-execution-report.md` for a purely internal project with no external stakeholder who will ever read it — ceremony instead of QA.
- Skipping any form of QA confirmation because "the one-line note felt like overkill too" — the lighter artifact is still gated on the same exit criteria, not a lighter bar.
- Writing a UAT test step around an internal mechanism ("verify the API returns 409") instead of what the tester can actually observe on screen.
- Letting a tester invent their own test data for an AC's condition instead of specifying a concrete, reproducible instance of it.
- Budgeting UAT as a flush-to-deadline half-day, without slack for the fact that a single Critical finding can trigger a full Stage 8 → 9 → 10 re-run.
- Assuming the coverage threshold is a fixed "80%" for every project instead of checking this project's own `standards/test-strategy.md`.

---

## Harborview in practice

[`uat-checklist.example.md`](../projects/TEMPLATE/sprints/uat-checklist.example.md) is Harborview's real Sprint 01 checklist — read the two `US-03-002` scenarios directly: TC-1 (happy path — create and save a draft invoice) shows the full observable-expected-result pattern in practice (a success toast with exact wording, a total that recalculates to a specific figure, the new invoice appearing in a list); TC-2 (rejection — Client field required) shows the same pattern applied to a validation-failure AC, the same shape as the deferred-payment example walked through above.

[`test-execution-report.example.md`](../projects/TEMPLATE/test-reports/test-execution-report.example.md) is Harborview's Sprint 01 execution report, worth reading for two things specifically: Section 3 (Defects Found) shows DEF-1, a Medium defect (a validation error message that doesn't clear correctly), logged and left Open rather than blocking anything — the routine case the deferral rule doesn't even need to engage for Medium severity. And Section 5 (QA Sign-off Checklist) shows the checklist's own item 3 — "No High defects are Open (or all High defects have been explicitly Deferred by the Product Owner — confirm in Defects Found table)" — checked off because Harborview's Sprint 01 genuinely had zero High defects, not because one was quietly waved through. If you want to see what a *properly deferred* High defect's row looks like in this same table, that's the shape to build from: a Status of "Deferred," not "Open," with the individually-recorded reasoning this chapter's worked example walks through living either in the row's own note or the sprint plan's carry-forward list.
