# Run Priming Session

You are running a priming session for the Ascendra framework — a free-form, informal conversation with the PO, optionally alongside raw material they drop into `source-material/`, that produces a `priming-package.md`: a distilled, structurally-organized head start for `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery`. This is not a discovery session in the formal sense — there is no playbook, no fixed question order, and no requirement to cover everything. It exists so one long, informal session can materialize whatever facts are available before the real, rigorous sessions run, so those sessions start from a head start instead of a blank page.

Full design rationale for this command lives in `ANCHOR-PROJECT-DESIGN.md` §5 at the repo root. This file is its executable form.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments

Extract `PROJECT_CODE` from `$ARGUMENTS`. Required — this command has no standalone-project variant; it always runs against an existing project.

If missing, ask:
> "Which project is this priming session for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

Wait for PO response.

---

## Step 2 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Project exists | `projects/{PROJECT_CODE}/` resolves | "`{PROJECT_CODE}` doesn't exist yet. Run `/init-project` first — it creates the folder structure this command reads and writes into." |
| `source-material/` folder exists | `projects/{PROJECT_CODE}/source-material/` resolves | "This project was created before `source-material/` was added to `/init-project`'s skeleton. Create `projects/{PROJECT_CODE}/source-material/` manually, or re-run `/init-project` if this is otherwise a fresh project." |

---

## Step 3 — Read what already exists

Before saying anything to the PO:

1. `projects/{PROJECT_CODE}/source-material/` — list every file present. Do not deep-read any of them yet; this is inventory, not triage (triage happens in Step 5).
2. `projects/{PROJECT_CODE}/priming/priming-package.md` — if it exists, this is a **resumed** session. Read its Section 10 (Resume Instructions) and Section 3 (Coverage Snapshot) to know what's already captured before asking about anything.
3. `projects/{PROJECT_CODE}/brief.md`, `projects/{PROJECT_CODE}/domain/*-core.md`, `projects/{PROJECT_CODE}/brds/brd-core-v1.md` — whichever of these already exist. **Anything already Approved/Locked in a real artifact is out of scope for this session** — do not re-ask about it. If the Brief is already Approved and the BRD is already Locked, this command has nothing left to usefully do; say so plainly rather than running a pointless session.

---

## Step 4 — Open the session

Keep this short — one message, not a form:

> "I can work from files you drop in `source-material/`, from talking it through, or both. What have you got?"

Wait for PO response. Branch based on what they say:
- **Files only, no interest in talking:** skip to Step 5's triage pipeline, then present what was extracted for confirmation (Step 6) rather than running a live conversation.
- **Wants to talk, no files:** skip triage, go straight to the live conversation (Step 5's conversational half).
- **Both:** triage the files first — it's cheaper to ask about genuine gaps once you know what the files already answered, rather than asking the PO something a file already told you.

---

## Step 5 — Conduct the session

This step has two distinct halves — triaging any dropped material, and the live conversation — used together, separately, or not at all, per Step 4's branch.

### 5a — Triaging dropped material (the three-pass pipeline)

For every file in `source-material/` the PO points to (or all of them, if they said "just use what's there"):

1. **Map** — skim only, no deep reading yet. Build a lightweight outline: what's in it, roughly where. A source with real structure (an SRS, a spec with real headings) — build the outline from that structure. A source with no reliable structure (raw notes, a handwritten dump, fragments) — cluster by inferred topic instead; there are no headings to skim by.
2. **Triage** — against that map and the priming package's own sections (Brief/Domain/BRD material), mark each unit relevant / redundant / irrelevant / ambiguous. Drop irrelevant units without deep-reading them — this is what keeps a large or noisy file cheap. Flag redundant units so the same fact doesn't get extracted twice.
3. **Extract** — deep-read only what survived, and write it into the matching priming package section, compressed and deduplicated. Never a verbatim transcription.

**Legacy material** — if a file describes an existing/legacy system: where it matches how this kind of business normally operates, capture it as ordinary content. Where it diverges for a real, stated business reason, flag it `AI Knowledge Correction`. Where it diverges and looks like a legacy design flaw rather than a real constraint, do not carry it forward as fact — flag it `Legacy Design Question`, framed as a negotiation ("the old system does X this way — real constraint, or something to leave behind?"), and raise it with the PO directly rather than silently deciding.

**Sensitive content** — recognize obviously credential-shaped strings (API keys, tokens, passwords) and flag them to the PO rather than filing them into the package.

**Volume** — for a genuinely large file, checkpoint progress in the priming package's own Section 2 (Session History) rather than holding the whole source in context across turns. It is entirely normal for one file to take more than one session to fully triage.

Persist the Map + Triage decisions themselves in Section 8 (Source Material Index) — which files were used, for what, and where to reference them later. This is what makes the session legible rather than a black box.

### 5b — The live conversation

**No script, no fixed order.** The PO raises whatever they want to raise, in whatever order it occurs to them. Your job is to listen for what maps onto the priming package's sections and capture it there — not to march through a checklist out loud.

**Question density — compression, not simplification.** The PO is not a layman being onboarded; use domain terms and the PO's own vocabulary freely. Compress what you need to ask into one or two sentences, not a paragraph of setup. An example or a short set of options helps when a question is genuinely abstract, not by default.

- **Correct:** "Who signs off on the expensive invoices — anyone in particular, or any one of a group?"
- **Incorrect:** "I'd like to understand your organization's approval workflow in more detail — could you walk me through, in as much detail as you're comfortable with, exactly who is involved in reviewing and approving invoices, what the criteria are for escalation, and whether there are any exceptions to this process?"

**Bounded question loops — draft and correct when a topic drags.** Ask up to roughly three short questions on one topic; if it's still not resolving, stop asking and state what you think you heard as a short draft: *"Sounds like: any one of the three directors can approve, no specific order — right?"* Correcting a draft is less effort for the PO than continuing to answer questions, and it closes topics that would otherwise stall the whole session.

**When to use which mode:** short Q&A for anything the PO clearly knows and can state in a sentence (who does what, what a term means, what's out of scope). Draft-and-correct for anything the PO is talking around without quite landing on ("it's complicated," a long tangent) — reflect back a short draft rather than keep probing. Most real sessions mix both within the same conversation; don't commit to one mode for the whole session.

**No Ubiquitous Language exists yet.** There is no domain glossary to conform to at this stage. Capture the PO's own terms exactly as used. If the same concept seems to get two different names across the conversation, note it in Section 7 as a flag — do not silently pick one or ask the PO to standardize on the spot. Resolving it is domain discovery's job, later.

**100% coverage is never the goal.** If a topic genuinely doesn't come up, leave it `Pending` and move on — do not manufacture a question for every empty section just to fill it. A short, honest session that leaves real gaps clearly marked is a better outcome than a long one that pads out low-value detail to look complete.

---

## Step 6 — Write or update the priming package

**Output path:** `projects/{PROJECT_CODE}/priming/priming-package.md`

Follow `projects/TEMPLATE/priming/priming-package.template.md` exactly. Do not include the `[AI Guide]` blocks in the output — same convention as every other generated document in this framework.

**If a package already exists** (a resumed session): merge new material into it. Update `Pending` topics that now have an answer, add new Session History and Source Material Index rows, never overwrite a `Covered` topic's existing content without the PO having actually revisited it.

**Density check before writing:** every entry should read like a discovery-state file's "PO stated" line — one to three sentences. If an entry reads like it belongs in the actual BRD or domain knowledge document, compress it back down before writing it.

---

## Step 7 — Run Pre-Handoff Verification

Work through `priming-package.template.md`'s own Section 11 checklist before presenting the session as done. A failed check does not mean the session failed — it means the gap gets recorded in Section 10 (Resume Instructions) rather than silently dropped.

---

## Step 8 — Report and offer handoff

```
PRIMING SESSION — {PROJECT_CODE}
─────────────────────────────────────────
Sections substantially covered: [list]
Sections still Pending:         [list]
Flags recorded:                 [N] (AI Knowledge Correction / Legacy Design
                                 Question / Scope Risk / terminology)
Open/Uncategorized items:       [N]
─────────────────────────────────────────
Next step:
Run /anchor-project {PROJECT_CODE} to have this package automatically feed
/run-intake, /run-domain-discovery, and /run-brd-discovery as anchor reaches
each phase — this is the only path that reads it automatically. Running any
of those commands directly instead still works, but none of them are aware
this package exists; reference projects/{PROJECT_CODE}/priming/priming-package.md
manually during that session if going that route.
```

> "Continue now with `/anchor-project {PROJECT_CODE}`, so it picks this package up automatically? Or stop here for now."

Wait for PO response.
