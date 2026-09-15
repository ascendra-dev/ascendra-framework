# Project Brief Template

> **[AI Guide — Document Level]**
> The project brief is the stable, top-level context document for a project. It is created once at intake and updated only when the project's scope, client, or strategic direction changes materially.
>
> **What this document is:** the *why* and *for whom* — the business context, goals, and constraints that frame all decisions downstream. The BRD captures *what the system must do*. The brief captures everything before that.
>
> **What this document is not:** a requirements document, a technical specification, or a feature list. If a sentence describes what the system will do rather than why it is being built or who it is for, it belongs in the BRD, not here.
>
> **When to create it:** before the first BRD discovery session. Load this brief as context when running the discovery session so the AI has full client context without the client needing to re-explain it.
>
> **When to update it:** only when the project's scope, client relationship, strategic direction, or key constraints change. Minor updates (new artifact added to Section 9, open question resolved in Section 8) are expected. Structural rewrites of Sections 1–7 signal a project direction change — flag it explicitly in Document Control.
>
> **Where it lives:** `projects/{PROJECT_CODE}/brief.md` — one file per project, in the project folder alongside the BRD files. Never in the framework's own root-level docs — briefs are project-specific, not framework knowledge.
>
> **Loading:** read directly from disk. Do not retrieve via the RAG pipeline.
>
> **One product rule:** This brief covers one user-facing product or capability. A product may have multiple technical components (API, web app, mobile app, workers) — those are defined in the architecture, not counted as separate projects. The boundary is at the user-facing purpose level: a teacher portal and a parent portal are two projects; a web app and its API are one project. If the engagement spans multiple products, each gets its own project and brief.
>
> **When used by `/init-project`:** Read Part 1 for the section structure and heading names only. Create `projects/{PROJECT_CODE}/brief.md` as a stub: pre-fill header fields (Project Name, Project Code, Client, Domain, Created) from the `/init-project` conversation and record identity in `projects/index.md` — there is no separate `project.json` file (`FW-027`) — then insert all section headings with awaiting-intake placeholder bodies for Sections 1–9. Do not fill any section content — that is the role of `/run-intake`.
>
> **When used by `/run-intake`:** Read the `[AI Guide]` note for each section — these are the intake session questions, correct/incorrect examples, and field-level guidance the session follows. Run the Part 2 Verification checklist at the end of intake before marking the brief "Brief complete."

---

## Document Control

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version from intake |

---

## Part 1 — Template

---

**Project Name:** [Name]
**Project Code:** [e.g. HARBORVIEW-INV-001 — CLIENT-DOMAIN-NNN format, assigned by /init-project]
**Client:** [Use "Internal" if building for your own organisation (internal team or own product). Use the client's full organisation name if building for a third party (e.g. Harborview Consulting Ltd).]
**Domain:** [e.g. Finance / Healthcare / Education / Logistics / Internal tooling]
**Status:** Active / On Hold / Complete / Cancelled
**Content Status:** Draft / Under Review / Brief complete / Approved / Locked / Superseded
**Created:** [YYYY-MM-DD]
**Last Updated:** [YYYY-MM-DD]
**Project Type:** Standalone / Extension
**Extends:** [PROJECT_CODE of the project this extends, or — if Standalone]
**Extension Adds:** [What capability this project adds to the parent, or — if Standalone]

> **[AI Guide — Two distinct status fields]**
> **Status** tracks the *project's* overall lifecycle — set once at `/init-project` (`Active`) and only changes for the whole project (paused, delivered, cancelled). It mirrors `projects/index.md`'s Status column; `update-status` keeps both in sync.
>
> **Content Status** tracks whether *this document's own content* is ready — `Draft` (placeholders only) → `Brief complete` (`/run-intake` finished) → `Approved` (gates `/gen-domain-knowledge`) → `Locked` / `Superseded` if ever revised formally. This is the field `/run-intake` and `/gen-domain-knowledge`'s gate check actually read and write — never confuse it with **Status** above; they track different things and are never set to the same value at the same time.

> **[AI Guide — Extension Context]**
> **Project Type** classifies whether this project stands alone or builds on a prior project's domain knowledge and requirements baseline.
>
> **Standalone:** No dependency on another Ascendra project's artifacts. Full domain discovery and BRD run from scratch.
>
> **Extension:** This project extends a prior Ascendra project. Fill in the parent's project code in **Extends**. Commands automatically load the parent project's domain knowledge, BRD baseline, and architecture extension points before generating artifacts for this project. Specify the direct parent only — the full chain is resolved automatically by reading each project's brief.
>
> **One product rule:** Regardless of type, this brief covers one user-facing product or capability. A product may have multiple technical components (API, web, mobile) — those are defined in the architecture, not counted as separate projects. The boundary is about distinct user groups or purposes: a teacher portal and a parent portal are two projects; a web app and its API are one project.

---

### 1. Problem Statement

> **[AI Guide]** 2–4 sentences. Describe the current-state problem — what exists today that is broken, missing, or insufficient. Answer three questions in order:
>
> 1. What is happening now that should not be, or what is not happening that should be?
> 2. What does that cost — in time, money, risk, missed opportunity, or human effort?
> 3. What would be true if the problem were solved?
>
> **Write from the client's perspective**, not the system's. Do not describe the solution or the system being built — that belongs in the BRD.
>
> **Correct:** "Harborview tracks all client invoices in a shared spreadsheet. As the team has grown, the spreadsheet fails to flag overdue invoices automatically, leaving the Finance Manager to manually chase payments every week. Unpaid invoices are regularly missed until the finance close."
>
> **Incorrect:** "We are building an invoice management system with automated reminders." — This describes the solution, not the problem.

---

### 2. Client Context

> **[AI Guide — Client definition]** "Client" in this framework means whoever commissioned or owns the project. Three valid cases:
> - **(a) External client** — a third-party organisation paying for the system (e.g. Harborview Consulting Ltd)
> - **(b) Internal team** — a team within the same organisation commissioning the system for their own use
> - **(c) Own product** — the delivery organisation itself building a product for its own use or market
>
> Adjust your language accordingly. In case (a) write as if describing a third party. In cases (b) and (c) the PO is effectively describing their own organisation — use first-person-plural framing where natural ("we process 40–60 invoices per month").
>
> **[AI Guide]** Enough background that a developer or AI agent joining mid-project would understand the client's business without asking follow-up questions. Cover:
>
> - Who the client is and what their business does
> - How large they are (team size, transaction volume, client base — whatever is relevant to system design)
> - Their current situation: what process, tool, or system they have today and why it is no longer sufficient
> - Any relevant background that would affect design decisions — e.g. they operate in multiple countries, they have a compliance team, their clients are also regulated entities
>
> **Write only what is relevant to building the system.** This is not a company profile or a sales document. If a fact would not change a design decision, omit it.

---

### 3. Strategic Goals

> **[AI Guide]** What the client wants to achieve beyond the immediate system. These are outcomes, not features. A goal describes a state the client wants to be in; a feature describes something the system does.
>
> **Test for each goal:** could the client achieve this goal without the system, in principle? If yes, it is a goal (the system is one way to achieve it). If no, it is a feature or requirement (the system is the only way to do it).
>
> **Correct:** "Automate payment reminders so the Finance Manager spends zero time manually chasing overdue invoices."
>
> **Incorrect:** "Add automated email reminders to the system." — This is a feature, not a goal. It belongs in the BRD.
>
> 2–5 goals. If the client gave vague goals ("be more efficient"), sharpen them into something observable: "Finance Manager can see the full AR position without leaving the system."

1. [Goal — outcome the client wants to be in, not a feature]
2. [Goal]

---

### 4. Key Stakeholders

> **[AI Guide]** List only people whose involvement affects delivery — decision-makers, primary users, and anyone who can block or derail the project. Do not list every person at the client organisation.
>
> **Decision Authority options:**
> - Final sign-off — approves BRD, sprint plans, and UAT. Nothing ships without their approval
> - Input only — consulted on requirements; does not approve deliverables
> - Daily user — will use the system every day; their workflow drives requirements
> - End user only — uses the system but is not involved in delivery or decisions
>
> **Involvement:** describe their expected involvement in delivery — weekly calls, UAT participation, available for questions only, etc. This is used to plan communication and gate dependencies.

| Name | Role | Decision Authority | Involvement |
|------|------|--------------------|-------------|
| [Name] | [Job title] | Final sign-off / Input only / Daily user / End user only | [How involved in delivery] |

---

### 5. Constraints

> **[AI Guide]** Hard limits that cannot be changed through negotiation or design. If a constraint can be relaxed with a conversation, it is a preference — put it in Section 6.
>
> **Budget:** state whether it is fixed (no overruns permitted), capped at a range, or not stated. If fixed, note the implication: scope changes require a formal change request.
>
> **Deadline:** state the exact date if known and the reason it cannot move (e.g. regulatory deadline, client event, financial close). A deadline without a reason is likely a preference, not a constraint.
>
> **Technology:** only hard mandates go here — systems the client already pays for and has a contractual obligation to use, or systems that must not be replaced. Technology preferences go in Section 6.
>
> **Regulatory:** name the specific regulation (GDPR, PCI DSS, FCA, HIPAA). "Privacy regulations" is not specific enough to design to.
>
> **Other:** any other non-negotiable. Write "None" for any category that has no constraint — do not leave blank rows.

- **Budget:** [Fixed / Capped at [range] / Not stated]
- **Deadline:** [Date + reason it cannot move, or "Not stated"]
- **Technology:** [Named mandatory systems, or "None"]
- **Regulatory:** [Named regulations, or "None identified"]
- **Other:** [Any other non-negotiable, or "None"]

---

### 6. Technology Preferences

> **[AI Guide]** The client's existing technology landscape and stated preferences. These are inputs for the architecture phase — they inform decisions but do not constrain them the way Section 5 does.
>
> **Existing systems:** systems already in use that the new system must integrate with or replace. These are factual, not preferences.
>
> **Preferred stack:** any language, framework, database, or cloud the client has expressed a preference for. Note whether this is a strong preference (previous investment, in-house expertise) or a mild one (heard good things about it).
>
> **Hosting:** cloud vs on-premise, and which provider if specified.
>
> **Authentication:** existing identity provider (Google Workspace, Azure AD, Okta) or standalone. This affects every screen in the system — capture it here even if it seems obvious.
>
> If the client has no preference on any of these, write "No preference — client defers to delivery team."

- **Existing systems:** [Systems that must be integrated with or replaced]
- **Preferred stack:** [Language / cloud / database preference, or "No preference"]
- **Hosting:** [Cloud / On-premise / Client-managed, and provider if specified]
- **Authentication:** [Existing SSO / identity provider, or "Standalone auth"]

---

### 7. Success Criteria

> **[AI Guide]** How the client will know the project was successful. Specific and observable where possible.
>
> **Test for each criterion:** could a Product Owner verify this during UAT without asking the client for their opinion? If yes, it is a good criterion. If the client would need to say "yes, that feels right," it is too vague.
>
> **Correct:** "Finance Manager can create and send an invoice in under 5 minutes from login — verified in UAT."
>
> **Incorrect:** "The system is easy to use." — This is an expectation, not a success criterion. It cannot be verified objectively.
>
> 3–7 criteria. Fewer than 3 means the client has not thought carefully about what success looks like. More than 7 means the list is diluted with obvious expectations.

1. [Specific, observable outcome — verified during UAT]
2. [Specific, observable outcome]
3. ...

---

### 8. Risks and Open Questions

> **[AI Guide]** Known unknowns at intake time. Two types — keep them distinct:
>
> **Risk:** something that might go wrong and affect delivery, even if it does not go wrong. It exists whether or not anyone acts on it. Risks need a mitigation or monitoring plan.
>
> **Open Question:** something that must be answered before or during BRD discovery. Without the answer, a requirement cannot be written or a constraint cannot be confirmed. Open questions have a due date and an owner — unowned questions never get resolved.
>
> **Type column:** use "Risk" or "Open Question" — do not combine them in one row.
>
> Every item must have an Owner. An unowned risk or question is an unmanaged one. Update the Notes column when items are resolved — do not delete rows.

| Item | Type | Owner | Notes |
|------|------|-------|-------|
| [Risk or question] | Risk / Open Question | [Named owner] | [Current status or resolution] |

---

### 9. Related Artifacts

> **[AI Guide]** A navigation index to all documents produced for this project. Update this section each time a new artifact is created or an existing one changes status. Someone reading the brief should be able to find every related document from this table without searching the file system.
>
> Standard artifact types: BRD, Discovery State, Epic, Sprint Plan, Architecture Decision Record, Sprint Roadmap.
>
> Status options: Draft / Under Review / Approved / Locked / Superseded.

| Artifact | Location | Status |
|----------|----------|--------|
| Project Brief | `projects/{PROJECT_CODE}/brief.md` | [Status] |
| BRD v1 | `projects/{PROJECT_CODE}/brds/brd-core-v1.md` | [Status] |

---

### 10. Density & Judgment Findings

> **[AI Guide]** Written only by `/judgment-check` — never filled at `/run-intake` time, and left out of the document entirely until that command has actually run once. This section holds one run's worth of findings against `practitioner-guide/01-intake.md`'s own tests (the Problem-Statement-as-solution test, the Strategic-Goals outcome-vs-feature test, the Constraints-vs-Technology-Preferences boundary, the Risk-vs-Open-Question distinction, and the rest of that chapter) — questions raised for the Product Owner to weigh, never verdicts. It is replaced wholesale by each new run, never appended to — see Section 11 for the durable record of how each finding was actually resolved. If `/judgment-check` has not yet run against this brief, this section does not appear in the document at all; do not pre-create it empty.

---

### 11. Density & Judgment Resolution

> **[AI Guide]** Written only by `/judgment-check`, appended to on every run, never overwritten — the same append-only discipline as Document Control above. Each row records one finding from Section 10's history and the Product Owner's actual response to it (Confirmed as-is / Revised / Acknowledged tradeoff), dated. A later run's finding covering the same spot in the document gets its own new row, never an edit to an earlier one. If `/judgment-check` has not yet run against this brief, this section does not appear in the document at all; do not pre-create it empty.

---

## Part 2 — Verification

> **[AI Guide]** Run every check below before this brief is used as context for a BRD discovery session. A brief that fails any check will produce a BRD with wrong or missing context.

- [ ] Problem Statement describes the current-state problem, not the solution — no sentence begins with "We are building..."
- [ ] Client Context includes only information relevant to system design — no company history or marketing content
- [ ] Every Strategic Goal is an outcome, not a feature — each goal could in principle be achieved without the system
- [ ] Every stakeholder has a named Decision Authority from the defined options
- [ ] Every Constraint in Section 5 is genuinely non-negotiable — preferences have been moved to Section 6
- [ ] Every named regulation in Section 5 is specific (e.g. UK GDPR, PCI DSS Level 1) — not generic ("privacy laws")
- [ ] Technology Preferences in Section 6 are distinguished from hard mandates in Section 5
- [ ] Every Success Criterion is observable — a Product Owner can verify it in UAT without asking the client for a subjective opinion
- [ ] Every item in Section 8 has a named owner — no row with "TBD" in the Owner column
- [ ] Related Artifacts in Section 9 reflects the current state of all project documents — no stale "Draft" statuses on approved documents
- [ ] If Project Type is Extension: the Extends field names a valid project code and `projects/{EXTENDS_CODE}/` exists in this workspace
