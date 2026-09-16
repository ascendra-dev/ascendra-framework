Harborview Consulting Ltd — right after `/init-project` created `HARBORVIEW-INV-001`, James Okafor dropped a rambling voice-note transcript and a couple of old Excel sheets into `source-material/`, ahead of any formal discovery. This example is deliberately **partial** — several sections are `Pending`, one Flag is unresolved — because 100% coverage was never the bar, and a fully-filled example would teach the wrong calibration. Compare against `brief.example.md`, `domain-discovery-state.example.md`, and `brd-discovery-state.example.md` — the real, fuller sessions that ran afterward and closed every gap this file left open.

---

## 1. Session Context

| Field | Value |
|-------|-------|
| Project Code | HARBORVIEW-INV-001 |
| Client | Harborview Consulting Ltd |
| Source material folder | `projects/HARBORVIEW-INV-001/source-material/` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Touched |
|-----------|------|--------------|-------------------|
| 1 | 2026-06-08 | Ascendra AI (voice-note transcript + Excel sheets) | 4, 5 (partial), 6 (partial) |

---

## 3. Coverage Snapshot

| Section | Status |
|---------|--------|
| 4 — Brief Material | Substantially covered |
| 5 — Domain Material | In progress |
| 6 — BRD Material | In progress |

---

## 4. Brief Material

**Problem Statement** — Status: Covered
PO stated: "We're doing everything in a shared spreadsheet right now. I don't know what's outstanding until I go dig for it, and reminders only go out when I remember to send them — which is basically never on time."

**Client Context** — Status: Covered
PO stated: Harborview Consulting Ltd, small consultancy. James is the only finance person. Three directors, all involved in approving anything "big."

**Strategic Goals** — Status: Covered
PO stated: "I want to stop opening Excel every morning to figure out who owes us money. And the directors are sick of me forwarding them invoices by email for sign-off."

**Key Stakeholders** — Status: Partial
PO stated: James (Finance Manager) runs it day to day. Three directors approve high-value invoices — only got one name (Sarah Chen) in this session, the other two weren't mentioned.

**Constraints** — Status: Covered
PO stated: "Everything's in pounds, we don't do other currencies." Small team — "just me on the finance side." No mention of a budget ceiling or deadline pressure in this session.

**Technology Preferences** — Status: Pending
PO stated: *(nothing raised — James didn't have an opinion on this yet)*

**Success Criteria** — Status: Partial
PO stated: "If I could just see, at a glance, what's overdue and who I still need to chase, that alone would save me hours." No measurable target given yet (e.g. a time figure) — that got sharpened later during `/run-intake`.

---

## 5. Domain Material

**Domain Scope** — Status: Covered
PO stated: Invoicing and collecting payment from clients. Not trying to do payroll or expenses — "that's a totally different system, don't touch it."

**Core Entities** — Status: Partial
PO stated: Talked about "clients" and "invoices" a lot. Didn't get into what data is actually tracked per client — deferred to the real domain session.

**Lifecycle & Status Model** — Status: Pending
PO stated: *(not raised — the Excel sheets implied a Draft → Sent → Paid flow, but James never said this explicitly, so it's not recorded as stated)*

**Universal Business Rules** — Status: Partial
PO stated: VAT gets added to everything — "20%, always, no exceptions I can think of." No mention of exemptions.

**Standard Validations** — Status: Pending
PO stated:

**Common Personas & Roles** — Status: Partial
PO stated: James (creates/sends invoices), the three directors (approve above some threshold — exact number not yet given), clients (receive and pay — "they don't log into anything, I just email them").

**Typical Integration Points** — Status: Partial
PO stated: "We use Stripe for basically everything else already, would make sense to use it here too."
Flag: `AI Knowledge Correction` — James specifically wants Stripe, not a generic "some payment processor" — worth carrying forward as a stated preference, not a domain default.

**Regulatory Baseline** — Status: Pending
PO stated:

**Known Variations** — Status: Pending
PO stated:

**Typical Scope Boundaries** — Status: Partial
PO stated: "No app for clients — they're not going to log into anything, they just get an email." No mention of accounting-system integration in this session (surfaced later, during BRD discovery, as a Phase 2 item).

---

## 6. BRD Material

**Core requirement topics** — Status: Partial
PO stated: Create and send invoices; directors approve the expensive ones; clients pay somehow; remind people who haven't paid. No detail yet on exactly how creation works (line items? PDF? reference numbers?) — that's what `/run-brd-discovery` dug into properly.

**Journey Walkthroughs** — Status: Partial
PO stated: "I make the invoice, send it, and then I'm just... waiting, checking my bank statement to see if it landed." Approval flow mentioned only in passing — "sometimes I have to email Sarah and wait for her to say yes."

**Walking Skeleton (FW-048)** — Status: Pending
PO stated: *(Not clear yet from this session which flow is the true thin path — create/send/collect looks likely given how James talked about it, but this wasn't confirmed directly, so it's left Pending rather than guessed. `/run-brd-discovery` confirmed it properly — see `brd-discovery-state.example.md` Section 6, Walking Skeleton note.)*

**Integration Confirmation** — Status: Partial
PO stated: Stripe — "yes, definitely." Nothing else raised.

**Output Confirmation** — Status: Pending
PO stated:

---

## 7. Terms As Used

| Term | Where Used | Flag |
|------|-----------|------|
| "clients" | Throughout | Used consistently — no synonym issue, but worth confirming this is the term that sticks rather than "customers" or "payers," since the domain default for this kind of system sometimes uses "Payer." |

---

## 8. Source Material Index

| File | Kind | Purpose | Reference When |
|------|------|---------|-----------------|
| `source-material/james-voice-note-transcript.txt` | Informal transcript | Primary source for everything above | Cross-reference during `/run-brd-discovery` if a captured answer here seems ambiguous — the original wording may resolve it |
| `source-material/old-invoice-tracker.xlsx` | Legacy Excel sheet | Implies a Draft → Sent → Paid status flow and shows real invoice numbering in use (`INV-2025-001000` style) — not confirmed verbally, so not recorded as PO-stated fact above | Reference during Domain Discovery when confirming the lifecycle/status model and the reference-number format |

---

## 9. Open / Uncategorized

| # | What was discovered | Why it doesn't fit above | Where it probably belongs |
|---|---------------------|---------------------------|----------------------------|
| PU-001 | James mentioned in passing that Harborview is "hopefully going to start using some accounting software next year, not sure which one" | Too vague to be a real Domain or BRD fact yet — no specifics | Revisit during BRD discovery; likely lands as a Section 5.21 (Future Capabilities) candidate once a real system is named, matching what actually happened (Xero, confirmed later) |

---

## 10. Resume Instructions

**Next session starts at:** Section 5 (Domain Material) — Lifecycle & Status Model, Standard Validations, Regulatory Baseline, Known Variations all still Pending; Section 6 (BRD Material) — Output Confirmation still Pending

**Before resuming:**
- [x] Load: this file
- [x] Load: `source-material/james-voice-note-transcript.txt`, `source-material/old-invoice-tracker.xlsx`
- [x] Review Section 3 (Coverage Snapshot)

**Topics still Pending (do not skip silently):**
- Technology Preferences (Brief)
- Lifecycle & Status Model, Standard Validations, Regulatory Baseline, Known Variations (Domain)
- Output Confirmation (BRD)
- Walking Skeleton confirmation (BRD) — likely candidate exists, not yet confirmed directly

**Flags still needing a real discovery session:**
- Stripe as a stated integration preference (Section 5)
- "clients" vs. domain-default "Payer" terminology (Section 7)
- PU-001, accounting software (Section 9)

---

## 11. Pre-Handoff Verification

- [x] Section 1: Project Code, Client, and Source material folder all filled
- [x] Every `Covered`/`Partial` topic in Sections 4–6 has a substantive PO-stated line
- [x] Every Flag in Sections 5–7 is typed (`AI Knowledge Correction`, or a plain terminology note)
- [x] Section 8 lists both files actually referenced above — no orphaned citations
- [x] Section 9 is populated, not left as a bare "None"
- [x] Section 10 names a real next section to resume at
