# BRD Discovery State — [Project Name]

> **[AI Guide — Document Level]**
> This file is written by the AI at the end of every requirement discovery session where the BRD is not yet complete.
> It is the sole source of continuity between sessions and the primary input to `/gen-brd` when generating the BRD in a separate session.
>
> **When writing this file (`/run-brd-discovery`):**
> - Distil what the client stated into each section. Do not write a transcript — write what `/gen-brd` needs to draft BRD requirements.
> - On resume: load this file, the domain document(s) listed in section 1, and the BRD playbook. Then follow the Resume Instructions at the bottom.
>
> **When reading this file (`/gen-brd`):**
> - Section 8 (Captured Client Answers) is the primary source for all BRD requirements.
> - Section 4 (Scope Risk Flags) feeds BRD dependencies and open items.
> - Section 7 (Open Questions) maps directly to BRD Section 12.
>
> Do not treat this file as a conversation transcript. It is a distilled handoff.

---

## 1. Project & Session Context

| Field | Value |
|-------|-------|
| Project Name | |
| Project Code | |
| Client | |
| Domain | |
| Sector | |
| Domain docs loaded | |
| BRD playbook used | `projects/{PROJECT_CODE}/brds/{slug}-brd-playbook.md` |

---

## 2. Session History

| Session # | Date | Completed By | Sections Covered |
|-----------|------|-------------|-----------------|
| 1 | | | |

---

## 3. Playbook Progress

> **[AI Guide]** Mirror section 4 (Core Discovery Questions) and section 6 (Journey Walkthroughs) of the BRD playbook used. Mark each topic as covered or pending.
>
> For every covered topic: write one-line distillation of the client's answer — enough to draft a BRD requirement without re-asking.
> For pending topics: leave the answer blank.

### Section 4 — Core Discovery Questions

| # | Topic | Status | Client Answer (distilled) |
|---|-------|--------|--------------------------|
| 4.1 | [Topic name] | Covered / Pending | |
| 4.2 | [Topic name] | Covered / Pending | |
| 4.3 | [Topic name] | Covered / Pending | |

### Section 6 — Journey Walkthroughs

| # | Journey | Status | Key Points from Client |
|---|---------|--------|------------------------|
| 6.1 | [Journey name] | Covered / Pending | |
| 6.2 | [Journey name] | Covered / Pending | |

**Walking Skeleton (FW-048):** [Which journey — or trimmed, cross-persona combination of journeys above — is the thinnest real end-to-end path the PO confirmed. `/gen-brd` reads this directly to tag Section 5 requirements; do not leave blank once Section 6 is Covered.]

---

## 4. Scope Risk Flags

> **[AI Guide]** List every scope risk flag (from BRD playbook section 5) that triggered during discovery. If none triggered, write "None."

| Flag ID | Flag Name | Triggered By (client statement) | Client Response | Status |
|---------|-----------|--------------------------------|----------------|--------|
| | | | | Open / Resolved / Deferred |

---

## 5. Integration Confirmation

> **[AI Guide]** Record the status of each integration from BRD playbook section 7. If section 7 has not been reached, mark all as Pending.

| Integration | Status | Notes |
|------------|--------|-------|
| [Name] | Confirmed In / Confirmed Out / Pending | |

---

## 6. Output Confirmation

> **[AI Guide]** Record the status of section 8 (Output Confirmation). If that section has not been reached, mark as Not Yet Run.

**Status:** Not Yet Run / Partially Completed / Completed

| Output | Confirmed? | Notes |
|--------|-----------|-------|
| [Report / notification name] | Yes / No / Pending | |

---

## 7. Open Questions

> **[AI Guide]** List every question raised during discovery that was not resolved. These will be carried into BRD section 12. If none, write "None."

| # | Question | Raised In | Owner | Due | Notes |
|---|---------|---------|-------|-----|-------|
| OQ-001 | | | | | |

---

## 8. Captured Client Answers (BRD Input)

> **[AI Guide]**
> Write the substantive answers from the client here, distilled per topic. The goal: a colleague who was not in the room can draft BRD sections 1–10 from this section alone.
>
> Do not write a transcript. Do not write your inferences. Write what the client said.
> Do not write domain knowledge that came from the domain doc — only client-stated information.
>
> Structure: one block per topic covered. Start each block with the topic name from section 3 above.

---

### [Topic Name from Section 4.1]

[What the client said, distilled. Include any client-specific deviations from domain defaults, any constraints they mentioned, any terminology they used that differs from the domain doc.]

---

### [Topic Name from Section 4.2]

[...]

---

### [Journey Name from Section 6.1]

[Key steps the client described, any variations from the standard journey, any exceptions or edge cases they mentioned, any approval steps or handoffs they named.]

---

## 9. Resume Instructions

> **[AI Guide]**
> This section is the first thing to read on resume. It tells you exactly where to pick up.

**Next session starts at:** [Section number and topic name from the BRD playbook, e.g. "Section 4.3 — Payment Method Confirmation"]

**Before resuming with the client:**
- [ ] Load: [list domain docs]
- [ ] Load: [BRD playbook file path]
- [ ] Load: this file (already done if you are reading this)
- [ ] Review section 8 above to rebuild context on what the client has already told you
- [ ] Review section 4 (Scope Risk Flags) if any flags are marked Open

**Brief the client at the start of the session:**
Summarise in 3–4 bullet points what was covered last session. Name the topics that are still pending. Ask if anything has changed since the last session before continuing.

**Topics still pending (do not skip):**
- [List pending topics from section 3]

**Things the client committed to follow up on:**
- [Any follow-up the client mentioned they would come back with]

**Scope risk items still open:**
- [Any SRF marked Open in section 4]

---

## 10. Pre-Generation Verification

> **[AI Guide — Verification]** Run these checks before passing this file to `/gen-brd`. Do not generate if any check fails.

- [ ] Section 1 (Project & Session Context): Project Name, Project Code, Client, Domain, and BRD playbook path are all filled
- [ ] Section 3 (Playbook Progress): Every topic marked Covered has a substantive one-line client answer — not a placeholder
- [ ] Section 4 (Scope Risk Flags): All Open flags are reviewed — either Resolved or explicitly Deferred with a reason recorded
- [ ] Section 5 (Integration Confirmation): Every integration is marked Confirmed In, Confirmed Out, or Pending with a stated reason
- [ ] Section 6 (Output Confirmation): Completed, or marked Not Yet Run with a note that outputs will be derived from domain defaults
- [ ] Section 7 (Open Questions): All unresolved questions are listed with an owner — they map to BRD Section 12 and must not be silently dropped
- [ ] Section 8 (Captured Client Answers): Every Covered topic has a substantive block — this is the primary source for all BRD requirement generation
