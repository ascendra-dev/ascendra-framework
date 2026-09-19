# Priming Package — [Project Name]

> **[AI Guide — Document Level]**
> This file is written by `/run-priming-session` from whatever raw material and free-form conversation a PO can offer *before* formal discovery starts — a legacy SRS, scattered notes, an old requirements doc, a rambling explanation of the business, any combination, in any order. It is not a transcript and not a deliverable — it is a distilled, structurally-organized capture of facts that gives `/run-intake`, `/run-domain-discovery`, and `/run-brd-discovery` a head start instead of a blank page.
>
> **It is scoped to Problem Domain only — Brief, Domain, BRD.** Epics, Screen Design, and Architecture are AI synthesis performed *from* an approved BRD, not raw PO knowledge — there is nothing here for those phases, and nothing here should try to anticipate them.
>
> **100% coverage is never the bar.** Capture whatever surfaces in one informal session; mark the rest `Pending`. The real discovery sessions still run in full afterward and fill every gap this file leaves — nothing downstream is shortened, skipped, or made less rigorous by this file existing. Its only job is to materialize facts as a head start.
>
> **Density: match this file's own model, not the documents it feeds.** Every entry here is a short "PO stated" distillation — one to three sentences, in the PO's own words where possible — never final-artifact prose. If you find yourself writing a paragraph that reads like it belongs in `brd-core-v1.md`, stop and compress it back down. This file is not the BRD, not the domain knowledge document, and not the brief — it is raw material for the sessions that produce those.
>
> **No Ubiquitous Language exists yet.** The domain glossary this framework normally enforces conformance against doesn't exist until domain discovery establishes it. Capture the PO's own terms exactly as used — do not normalize, standardize, or silently pick one term over another. Where the same concept seems to get two different names across the conversation, record it in Section 7 as a flag, not a decision.
>
> **Flag types — the only recognized set, used consistently in every section below:**
> - `AI Knowledge Correction` — source material diverges from generic domain understanding for a real, stated business reason.
> - `Legacy Design Question` — source material diverges and looks like a legacy design flaw rather than a real constraint; framed as a negotiation, never resolved here.
> - `Scope Risk` — something that could expand or threaten the scope as currently understood.
> - A plain synonym/terminology note (Section 7 only) — the same concept getting two different names.
> - `Conflicting Source` — two or more captured statements about the same topic disagree with each other, regardless of origin: two places in the same source file, two different source files, or the priming package disagreeing with something the PO says live. Never silently pick a side — record every conflicting statement in full, each attributed to where it came from.
>
> Every Flag, of any type, on any topic, means that topic is **not** a known/complete answer — see the "When reading this file" note below for what this means downstream.
>
> **When writing this file (`/run-priming-session`):** distil what the PO actually said into the matching section below. Do not write a transcript. Do not invent content to fill a gap — an honest `Pending` is always correct where nothing surfaced. Sessions can repeat: if this file already exists, merge new material into it rather than starting over or overwriting what's already captured.
>
> **When reading this file (`/anchor-project`):** this file is never read directly by `/run-intake`, `/gen-domain-knowledge`, `/run-brd-discovery`, or `/gen-brd` — none of them are modified to know it exists. When anchor reaches each of those phases, it decomposes the matching section below into that phase's own expected input (pre-filling `brief.md`'s fields, or `domain-discovery-state.md` / `brd-discovery-state.md` before invoking the command that reads them) at that moment — not speculatively ahead of time. A PO running any of those commands directly, without anchor, is entirely unaffected by whether this file exists. Sections 7, 8, and 10 are never pre-filled into a phase's field — they carry no answer to substitute, only reference material. Section 10 (Source-Specific Material) is handed along as background reference material to whichever phase command is about to run; Section 8 (Source Material Index) the same way; Section 7 (Terms As Used) specifically to `/run-domain-discovery`, the one phase where a flagged possible-synonym is actually actionable — see `conventions/priming-command-conventions.md` PC-041/PC-042. **A topic carrying any unresolved Flag, of any type, is never pre-filled either, regardless of its own Status field** — Status and Flag are independent, and a Flag always means the topic surfaces as a live question when its owning phase command reaches it, never a silent substitution; see `conventions/priming-command-conventions.md` PC-054. **Section 9 (Open/Uncategorized) is handled like a `Pending` topic, not like reference material** — each unresolved row surfaces as a live question at the phase named in its own "where it probably belongs" column, never an automatic write into any artifact; see `conventions/priming-command-conventions.md` PC-043.
>
> **More than one command can write into this file.** `/run-priming-session` is the only one today, and maps everything it captures onto Sections 4-6. A future command reading a different source mechanic (e.g. legacy code) may also contribute to Section 10 — see `conventions/priming-command-conventions.md`, which governs every priming-producing command uniformly.

---

## 1. Session Context

> **[AI Guide]** `/init-project` must have already run — this command reads `Project Code` from it and writes into `projects/{PROJECT_CODE}/source-material/`, a folder `/init-project` creates. There is no pre-project variant of this session.

| Field | Value |
|-------|-------|
| Project Code | |
| Client | |
| Source material folder | `projects/{PROJECT_CODE}/source-material/` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Touched |
|-----------|------|--------------|-------------------|
| 1 | | | |

---

## 3. Coverage Snapshot

> **[AI Guide]** No numeric confidence score here, deliberately — a single blended percentage across three very different sections (Brief, Domain, BRD) would imply a precision this session was never meant to produce. `/run-domain-discovery` and `/run-brd-discovery` each compute their own real confidence score later, against their own template's actual criteria; this table just shows where things stand right now, at a glance, before anyone reads the detail below.

| Section | Status |
|---------|--------|
| 4 — Brief Material | Not started / In progress / Substantially covered |
| 5 — Domain Material | Not started / In progress / Substantially covered |
| 6 — BRD Material | Not started / In progress / Substantially covered |

---

## 4. Brief Material

> **[AI Guide]** Mirrors `brief.template.md`'s own sections — Problem Statement, Client Context, Strategic Goals, Key Stakeholders, Constraints, Technology Preferences, Success Criteria. One block per topic, same format throughout this file:
>
> **[Topic name]** — Status: Covered / Partial / Pending
> PO stated: [distillation, the PO's own words where possible]
> Flag (only if applicable): [use one of the recognized types from the document-level AI Guide above — never an untyped or improvised flag]
>
> Do not chase brief.template.md's own Correct/Incorrect examples for polished, measurable phrasing here — that rigor belongs to `/run-intake`, which will sharpen a vague goal into something testable. This session's job is to capture what the PO actually said, not to pre-write the brief.

**Problem Statement** — Status: Pending
PO stated:

**Client Context** — Status: Pending
PO stated:

**Strategic Goals** — Status: Pending
PO stated:

**Key Stakeholders** — Status: Pending
PO stated:

**Constraints** — Status: Pending
PO stated:

**Technology Preferences** — Status: Pending
PO stated:

**Success Criteria** — Status: Pending
PO stated:

---

## 5. Domain Material

> **[AI Guide]** Mirrors `domain-discovery-state.template.md` Section 4's topic list — Domain Scope, Core Entities, Lifecycle & Status Model, Universal Business Rules, Standard Validations, Common Personas & Roles, Typical Integration Points, Regulatory Baseline, Known Variations, Typical Scope Boundaries. Same block format as Section 4 above.
>
> **Legacy material — don't let a bad system distort the domain model.** If source material describes an existing/legacy system: where it matches how this kind of business normally works, capture it as ordinary content. Where it diverges for a real, stated business reason, note it as a Flag tagged `AI Knowledge Correction` — the real domain discovery session turns this into a proper AI Knowledge Corrections entry. Where it diverges and looks like a legacy design flaw rather than a real constraint (bad normalization, an accidental workaround that calcified), do **not** silently carry it forward as fact — tag the Flag `Legacy Design Question` and frame it as a negotiation: *"the old system does X this way — real constraint, or something to leave behind in the upgrade?"* Never resolve this one yourself.

**Domain Scope** — Status: Pending
PO stated:

**Core Entities** — Status: Pending
PO stated:

**Lifecycle & Status Model** — Status: Pending
PO stated:

**Universal Business Rules** — Status: Pending
PO stated:

**Standard Validations** — Status: Pending
PO stated:

**Common Personas & Roles** — Status: Pending
PO stated:

**Typical Integration Points** — Status: Pending
PO stated:

**Regulatory Baseline** — Status: Pending
PO stated:

**Known Variations** — Status: Pending
PO stated:

**Typical Scope Boundaries** — Status: Pending
PO stated:

---

## 6. BRD Material

> **[AI Guide]** Mirrors `brd-discovery-state.template.md`'s shape — core requirement topics, journey walkthroughs, integration confirmation, output confirmation, and scope risk flags — at this session's lighter density, not that file's full rigor. Same block format again. Do not attempt to assign REQ-level Priority or Layer here — those require real REQ granularity that only exists once `/gen-brd` runs; if the PO says something that clearly signals one ("this is only needed for the Pakistan rollout," "this can wait, it's not essential for launch"), capture it inside the PO-stated distillation itself, in their own words — the signal survives into the real BRD discovery session without you pre-deciding the tag.
>
> **Scope risk flags** get a Flag line tagged `Scope Risk`, same mechanism as everywhere else in this file — not a separate table. The real `/run-brd-discovery` session is what actually works through a triggered flag; this session only needs to notice and record that something risky came up.

**Core requirement topics** — Status: Pending
PO stated: [Whatever the PO described about what the system needs to do — capture as it comes, do not force it into BRD feature-area groupings yet]

**Journey Walkthroughs** — Status: Pending
PO stated:

**Walking Skeleton (FW-048)** — Status: Pending
PO stated: [If it's clear from the conversation which journey — or trimmed, cross-persona combination of journeys — is the thinnest real end-to-end path, name it here. If it isn't clear yet, leave Pending; `/run-brd-discovery` asks directly and this is not something to guess at.]

**Integration Confirmation** — Status: Pending
PO stated: [Which integrations the PO mentioned, confirmed in, or confirmed out]

**Output Confirmation** — Status: Pending
PO stated: [Reports, notifications, dashboards the PO described]

---

## 7. Terms As Used

> **[AI Guide]** No normalization, per the document-level note above. One row per term worth flagging — not every term the PO used, only the ones where something is genuinely ambiguous (a possible synonym, an unusual usage, a term that might not survive contact with the real domain glossary).

| Term | Where Used | Flag |
|------|-----------|------|
| [term] | [topic/section] | [e.g. "Also called 'Customer' once — same thing, or a real distinction?"] |

---

## 8. Source Material Index

> **[AI Guide]** One row per file in `source-material/` or otherwise fed into this session. This is a pointer index, not a summary — later phases cross-reference the original file for the stated purpose, they do not re-read it in full from here.

| File | Kind | Purpose | Reference When |
|------|------|---------|-----------------|
| [filename] | [SRS / notes / legacy docs / etc.] | [what it's actually useful for] | [which future phase/topic should pull from it, and why] |

---

## 9. Open / Uncategorized

> **[AI Guide]** Same discipline as the BRD template's Parking Lot (`FW-024`) — anything discovered that doesn't fit a section above still gets recorded here, not discarded. If none, write "None." Fill in "Where it probably belongs" with a real phase (Domain or BRD) whenever you can make a reasonable guess — `/anchor-project` uses that column to decide which live discovery session raises this row as a question later; see `conventions/priming-command-conventions.md` PC-043. A row with no reasonable guess is fine left blank — it defaults to surfacing at BRD discovery.

| # | What was discovered | Why it doesn't fit above | Where it probably belongs |
|---|---------------------|---------------------------|----------------------------|
| PU-001 | | | |

---

## 10. Source-Specific Material

> **[AI Guide]** This section holds facts a source-specific priming command routinely produces that never map onto Brief/Domain/BRD Material (Sections 4-6) — e.g. a code-reading command's existing API surface, schema-as-implemented, or technical-debt inventory. It is **not** a second overflow bucket alongside Section 9 (Open/Uncategorized) — Section 9 is for a genuine one-off with nowhere to go; this section is for an expected category of fact from a given source mechanic. See `conventions/priming-command-conventions.md` Section 3 (Mapping Priority) for the exact distinction, and Section 4 (The Source-Specific Section Mechanism) for how a contributing command adds its own subsection here.
>
> `/run-priming-session` owns no subsection here — prose, an SRS, raw notes, and live conversation always have a home in Sections 4-6. Leave this section at its default until a source-specific command (e.g. a future legacy-code-priming command) actually contributes.
>
> A contributed subsection uses this heading pattern, naming its owning command inline:
> `### 10.N Source-Specific — [Descriptive Name] (owned by /command-name)`

None — every fact captured this session mapped onto Sections 4-6.

---

## 11. Resume Instructions

> **[AI Guide]** First thing to read when this session (or anchor, decomposing this file) resumes.

**Next session starts at:** [Section and topic — or "Handoff ready" if nothing further is pending]

**Before resuming:**
- [ ] Load: this file (already done if you are reading this)
- [ ] Load: every file listed in Section 8 (Source Material Index)
- [ ] Review Section 3 (Coverage Snapshot) to rebuild context on what's already captured

**Topics still Pending (do not skip silently):**
- [List, pulled from Sections 4–6]

**Flags still needing a real discovery session (Sections 4–7):**
- [List]

**Open/Uncategorized items still unresolved (Section 9):**
- [List, pulled from Section 9 — each one surfaces as a live question at the phase named in its own "where it probably belongs" column; see `conventions/priming-command-conventions.md` PC-043]

---

## 12. Pre-Handoff Verification

> **[AI Guide — Verification]** Run these checks before treating this file as ready to hand off. Failing a check does not mean the session failed — it means the corresponding gap gets carried into Resume Instructions or the real discovery session, not silently dropped.

- [ ] Section 1 (Session Context): Client and Source material folder are filled, even if Project Code is still blank
- [ ] Every topic marked `Covered` or `Partial` in Sections 4–6 has a substantive "PO stated" line — not a placeholder
- [ ] Every Flag recorded in Sections 4–7 is tagged with a type from the document-level canonical list (`AI Knowledge Correction` / `Legacy Design Question` / `Scope Risk` / `Conflicting Source` / synonym note) — no untyped flags
- [ ] Section 8 (Source Material Index) lists every file actually referenced in Sections 4–6 — no orphaned citations to a file not indexed here
- [ ] Section 9 (Open/Uncategorized) is populated or explicitly "None" — nothing discovered was silently dropped
- [ ] Section 10 (Source-Specific Material) is either "None" or every subsection present is attributed to its owning command per `conventions/priming-command-conventions.md` PC-031 — no unlabeled entries
- [ ] Section 11 (Resume Instructions) names a real next step, or states "Handoff ready"
