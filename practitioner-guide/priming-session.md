# Priming Session

Covers [`/run-priming-session`](../.claude/commands/run-priming-session.md), governed by [`conventions/priming-command-conventions.md`](../conventions/priming-command-conventions.md). Read [`00-orientation.md`](00-orientation.md) first if you haven't. This command has no stage of its own — it's scoped to Problem Domain only (Brief, Domain, BRD material) and is meant to run once, early, right after `/init-project`, before `/run-intake` ever starts. Everything it produces is a head start for three later commands, never a substitute for them.

---

## Purpose

Every real discovery session starts from a blank page unless something fills it first. `/run-priming-session` is that fill: a free-form, optional, repeatable conversation — with or without raw material the PO drops into `source-material/` — that materializes whatever facts already exist into one file, `priming-package.md`. `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery` all read it as a head start when it exists. None of them are modified to require it, and none of them treat anything in it as confirmed — a priming package is provisional by design, re-confirmed for real once the actual discovery session reaches each topic.

This is deliberately not a discovery session in the formal sense. There is no playbook, no fixed question order, and no requirement to cover everything. That's the whole point: a rigid script forces structure onto raw, half-formed material before it's ready for one. Priming exists to materialize facts cheaply, in whatever order they arrive, so the real sessions start from something instead of nothing.

---

## Two ways in, and when to use which

*(A process choice made at the very start of the session — not something checkable against the finished package.)*

Step 4 opens with one question: "I can work from files you drop in `source-material/`, from talking it through, or both. What have you got?" The answer branches into two genuinely different halves, used together, separately, or not at all.

**Triaging dropped material** is a three-pass pipeline — Map, Triage, Extract — run per file. Map is a skim, no deep reading yet: build a lightweight outline of what's in the file and roughly where. Triage marks each unit relevant, redundant, irrelevant, or ambiguous against the priming package's own sections, and against whatever a topic already holds — checking for disagreement, not just redundancy. Extract deep-reads only what survived and writes it in, compressed, never a verbatim transcription.

**The live conversation** has no script and no fixed order — the PO raises whatever they want, in whatever order it occurs to them, and the job is to listen for what maps onto the priming package's sections rather than marching through a checklist out loud.

**If both exist, triage the files first.** It's cheaper to ask about genuine gaps once you already know what the files answered, than to ask the PO something a file already told you.

---

## Question density: compression, not simplification

The PO is not a layman being onboarded — use domain terms and the PO's own vocabulary freely. The discipline is compression, not simplification:

- **Correct:** "Who signs off on the expensive invoices — anyone in particular, or any one of a group?"
- **Incorrect:** "I'd like to understand your organization's approval workflow in more detail — could you walk me through, in as much detail as you're comfortable with, exactly who is involved in reviewing and approving invoices..."

**Bounded question loops.** Ask up to roughly three short questions on one topic. If it's still not resolving, stop asking and state a short draft instead: "sounds like — any one of the three directors can approve, no specific order — right?" Correcting a draft is less effort for the PO than continuing to answer questions, and it closes topics that would otherwise stall the whole session. Short Q&A works for anything the PO clearly knows and can state in a sentence; draft-and-correct works for anything they're talking around without quite landing on. Most real sessions mix both.

**100% coverage is never the goal.** If a topic genuinely doesn't come up, leave it Pending and move on — don't manufacture a question for every empty section just to fill it. A short, honest session that leaves real gaps clearly marked beats a long one that pads out low-value detail to look complete.

---

## "Out of scope" governs asking, not noticing

This is the one rule in this command worth its own section, because getting it wrong silently swallows real information. Anything already Approved or Locked in a real artifact — the Brief, the Domain document, the BRD — is out of scope for this session's own questions. Don't re-ask about it.

But that's a rule about what you go looking for, not about what you do when something surfaces anyway. If triage or the live conversation turns up a genuine new fact that happens to touch an already-settled topic — not a re-ask, something volunteered on its own — it still gets recorded. It goes into Section 9 (Open/Uncategorized), with a note naming the affected artifact and its status (e.g. "Domain — already Locked, needs `/assess-change`") rather than a phase to raise it in later, since there's no open discovery session left to raise it in. The distinction matters because the alternative — silently dropping it because the topic is "out of scope" — means a fact that should trigger a formal change review never reaches anyone.

---

## Handling disagreement: the Conflicting Source flag

When a unit's content disagrees with something already captured for the same topic — two places in one file, two different files, or a file against something the PO already said — never silently pick a side or overwrite the earlier entry. Write both statements in full, each attributed to its origin, and tag the flag `Conflicting Source`.

**If the PO is live when the disagreement surfaces, ask directly, right then** — compressed like any other question: "your notes say X, but you just said Y — which is it?" This is the cheap case, since resolution is available immediately. Only fall back to a recorded flag when no live resolution is possible in the moment — say, two dropped files disagree and the PO hasn't been asked about either yet.

**Harborview's own worked instance is real, not hypothetical.** James stated, live, that VAT is "20%, always, no exceptions I can think of." But `old-invoice-tracker.xlsx`, one of the files he dropped in, shows a historical invoice — `INV-2025-000412` — at a 0% VAT line item. Nobody asked James about that specific invoice, and the spreadsheet gives no reason for the difference. The session didn't guess which one was right. It recorded both, flagged `Conflicting Source`, and left it for `/run-domain-discovery` to resolve directly — a zero-rated client, or a data-entry error in the old sheet, are both plausible, and only James can say which.

---

## Legacy material: two different flags, two different postures

If a dropped file describes an existing or legacy system, not everything in it gets the same treatment. Where it matches how this kind of business normally operates, capture it as ordinary content. Where it diverges for a real, stated business reason, flag it `AI Knowledge Correction` — this is a genuine, worth-remembering fact about this client, not domain noise. Where it diverges and looks like a legacy design flaw rather than a real constraint, don't carry it forward as fact at all — flag it `Legacy Design Question`, framed as a negotiation ("the old system does X this way — real constraint, or something to leave behind?"), and raise it with the PO directly rather than silently deciding either way.

**Harborview's real instance of the first case:** James said Stripe specifically — "we use Stripe for basically everything else already, would make sense to use it here too." That's not a generic domain default (some payment processor); it's a stated preference worth carrying forward as `AI Knowledge Correction` precisely because a different client in the same domain might have said something else entirely.

---

## Density check before writing

Every entry in the priming package should read like a discovery-state file's "PO stated" line — one to three sentences. If an entry reads like it belongs in the actual BRD or domain knowledge document, it's too dense; compress it back down before writing. This is the same density discipline Chapter 3 taught you to apply to a REQ, one phase earlier and at a lower bar — a priming package entry is provisional, not a final requirement, and writing it at final-artifact density overstates how settled it actually is.

---

## Terminology recap

- **Covered / Partial / Pending** — a priming package topic's own status values. Partial means something real was captured but not the whole picture; it surfaces as an informed live question later ("here's what's known, what's missing"), never silently treated as complete the way a bare Pending vs. Covered binary would suggest.
- **Section 9 (Open/Uncategorized) vs. Section 10 (Source-Specific Material)** — Section 9 is for genuine one-off leftovers that don't fit anywhere else. Section 10 is reserved for a whole *expected* category of fact from a different extraction mechanic — this command owns no subsection there at all; everything it extracts has a home in Sections 4–6. See [Priming Code Session](priming-code-session.html) for the command that actually uses Section 10.
- **No Ubiquitous Language exists yet.** There's no domain glossary to conform to at this stage. Capture the PO's own terms exactly as used; if the same concept gets two names, flag it in Section 7 rather than picking one. Resolving it is domain discovery's job, later.

---

## Common mistakes

- Re-asking about a topic that's already Approved or Locked in a real artifact, instead of checking first.
- Silently dropping a genuine new fact because it touches an already-settled topic, instead of routing it to Section 9.
- Picking a side on a Conflicting Source disagreement instead of recording both statements with their origins.
- Carrying forward a legacy system's quirk as if it were a universal domain fact, without checking whether it's a real constraint or a design flaw.
- Writing a priming package entry at BRD or domain-document density instead of a one-to-three-sentence "PO stated" line.
- Manufacturing a question for every empty section to make the session look complete, instead of leaving a genuine gap marked Pending.
- Treating a rushed "yes, that's right" the same as a real, reasoned answer — the same rubber-stamp risk Chapter 2 warned about, one phase earlier.

---

## Harborview in practice

[`priming-package.example.md`](../projects/TEMPLATE/priming/priming-package.example.md) is a real, deliberately partial session — James dropped a rambling voice-note transcript and two old Excel sheets into `source-material/` right after `/init-project`, before any formal discovery ran. Worth reading end to end:

- **Section 4 (Brief Material)** shows the Partial/Covered mix in practice — Key Stakeholders is Partial because only one director's name (Sarah Chen) came up, not all three; Technology Preferences is Pending because James simply had no opinion yet.
- **Section 5 (Domain Material)** carries both flags this chapter walked through: the VAT `Conflicting Source` flag, and the Stripe `AI Knowledge Correction` flag.
- **Section 9 (Open/Uncategorized)** holds PU-001 — James mentioning, in passing, that Harborview might adopt accounting software "next year, not sure which one." Too vague for a real fact at the time. Read Chapter 3's REQ-022 (Accounting System Export, Xero) to see what this vague mention actually became, once real discovery confirmed it — the clearest evidence in this whole guide that priming's job is to materialize a lead, not to settle it.
- **Section 11 (Resume Instructions)** shows what an honest, unfinished handoff looks like — five Pending topics named explicitly, not glossed over, so the next session (or the real discovery sessions that followed) picks up exactly where this one left off.
