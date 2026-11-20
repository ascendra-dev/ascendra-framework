# Build Plan: Group {GG}, [Group Name]

> **[AI Guide — Document Level]**
> The living plan for building one build unit: usually one approved design group, sometimes a unit with no screens (such as a money spine). It holds every implementation decision and the build order, written so that three readers can use the same document: the **PO** (requirements, screens, rules, the demo), the **developer** (patterns, contracts, IDs, data, order) and the **AI** that builds it. All three read the same words; nothing is written for the AI alone.
>
> Produced by `/plan-build-group`, which reads the sources, takes every decision, and writes each one down with its reason. The PO and the developer review and approve it (Status: Approved). `/build-checkpoint` then builds it one checkpoint at a time, and adds a Built block under each checkpoint. The close (`B{GG}.X`) completes it.
>
> **How it is organised:**
> - **Part A, the group:** everything decided once for the whole group. It is read before any checkpoint, and every checkpoint relies on it.
> - **Part B, the checkpoints:** in build order, grouped into chunks. Each checkpoint is self-contained: with Part A, it is everything needed to build, test and review that step.
> - **Part C, the close:** the group's demo, regression and deploy, and what later groups must know.
> - **Block library:** the standard blocks Part B is built from (page, listing, form, dialog, endpoint, migration, money command…). Each block has fixed fields, so every plan specifies the same things the same way.
>
> **Rules for every plan:**
> - **Complete, not by reference.** A fact the build needs is written here, copied from its source, with a source stamp: `(source: handoff decision 131)`, `(source: Staff.tsx)`, `(source: Blueprint §27)`. A link alone is never enough: "see decision 131" makes the reader go and look.
> - **Every decision is written as question → answer → why.** Simple facts go in the tables. Any choice with a real alternative goes in Section A15, and the table cell points to it (`D4`). Nothing is left for the reader to decide: no "TBD", "to be confirmed" or "the builder decides".
> - **The design is approved and doesn't change silently.** Where the plan needs something different from the artboards, or something they don't draw, it goes in Section A13 as a proposal, which the PO has agreed before the plan is written.
> - **Every concern in Section A2 is answered**, either "applies, see X" or "not applicable, because…". A missing answer can't be told apart from a forgotten one.
> - **Names and IDs follow one scheme**, so code and plan can be searched for each other (Section A3, "IDs used in this plan").
> - **A screen reads only an error's `code` and `message`.** The error body is the API contract's flat shape (Rule 1), with nothing else in it. The screen branches on `code` to decide where a message goes, and shows the API's `message` where the text needs data the screen doesn't hold (such as another person's name). If the screen needs data to show a refusal (such as when a session opened), that data comes from a query the screen already makes (the record's row), never from the error.
> - **Roles, never people.** Write "the PO", "the developer", "a School admin". Sample people from the design (such as "Rashid Ahmed") are seed data and may appear as such.
> - **Artboards are named by their canvas title**, with the canvas page they are on, never by their source file name and never pictured. The reviewer has the design canvas open and finds an artboard by its title.
> - **Effects go through domain events.** No module calls the audit or notification module directly. An audit entry is a critical event, raised inside the transaction so it commits or rolls back with the change. An email or notification is an after-commit event, sent by a reaction in the applications layer. Money between modules stays a direct contract call inside one transaction.
> - **Nothing shows a placeholder.** A part that later work feeds shows its true current state (real data, even if empty), or is disabled with its reason. Only a whole page or sidebar item of an unbuilt group is hidden. Later work replaces only what sits behind a seam, whose honest default is stated here. No sample data from a real endpoint.
> - **Modules and folders come from the project's architecture note.** A plan names no module the note doesn't list, and writes every file path inside the folder of the module that owns it.
>
> **What this document is not:**
> - Not the design. Screen text and layout come from the artboards; the plan copies the text it needs and names the artboard. A new screen or text goes to the design first, as a Section A13 proposal.
> - Not the architecture. It works inside the architecture note. A change to the note goes to Part C, "Proposed changes for their owners".
> - Not a story list. Chunks and checkpoints state the job and its proof; there are no user stories.
> - Not a code dump. Code snippets appear only where a shape must be exact (a type, a response, a migration). The diff is reviewed with each commit.
>
> **When used by `/plan-build-group`:** the structure to generate. Fill every section and remove every AI Guide. Copy the blocks a checkpoint needs from the Block library into the checkpoint, and delete the library from the written plan. Run the Plan checks before writing, and fix every failure. Status starts at `Draft`; every checkpoint starts at `Planned`, with no Built blocks.
>
> **When used by `/build-checkpoint`:** read the header, Part A in full, and the one checkpoint being built (or Part C for the close), plus the Built blocks of earlier checkpoints in the same chunk. The checkpoint's blocks are binding. Write only that checkpoint's Built block and Status cell, and append to Part C's Findings. If the plan is wrong or missing a fact, stop and propose the change to the PO. Never fill it in from other sources silently, and never rewrite another section.
>
> **File location:** `projects/{PROJECT_CODE}/build/group-{GG}-{slug}.md`. `{GG}` is the two-digit design group number from the screen inventory, or a short name for a unit without a group (`spine`).

**Status:** Draft

| Field | Value |
|---|---|
| Unit | [Group {GG}, Group Name] |
| Design | [Screens (SCR IDs), the canvas and page holding the artboards, and the decision that approved the design, linked] |
| Depends on | [Build units that must be closed first, linked to their plans, or "none"] |
| Reviewers | PO: requirements, screens, rules, demo. Developer: patterns, contracts, IDs, data, order |
| Repos and base | [Each code repo, the main line the plan builds on, and the commit it was reconciled against. Unmerged branches are reference only: any code lifted from one is listed in A14] |
| Reconciled | [Date the plan was checked against the code. `/build-checkpoint` compares it with later commits] |
| Design parity | [How an artboard is rendered, and how the built page is rendered at the same size with the seed] |
| Size | [N chunks, N checkpoints] |

## Document Control

> **[AI Guide]** Add a row whenever the plan changes after its first version. A change made after approval says so, and names who approved it. Built blocks don't need a row.

| Version | Date | Author | Summary of Changes |
|---------|------|--------|--------------------|
| 1.0 | [Date] | [Author] | Initial version |

---

# Part A — The group

---

## A1. Sources read

> **[AI Guide]** Every source the plan was written from, and which part of it was read. This lets a reviewer see what the plan is based on, and what it ignored. Facts copied into this plan carry a source stamp that matches a row here.

| Source | Parts read | Why |
|---|---|---|
| [Document or code path] | [Sections, rows or files] | [What it gave the plan] |

---

## A2. Considered for every group

> **[AI Guide]** The fixed list. Answer every row, for every group, in the same order. "Applies" points to where the plan handles it. "Not applicable" gives the reason in a few words. Never delete a row, and never leave one blank. If a row applies only to a later group's part of this work, say "Deferred" and point to Section A3's deferred list.
>
> **Correct:** "Approval by a second person | Not applicable: staff changes take effect directly (source: handoff decision 131)"
> **Incorrect:** "Approval by a second person | N/A" (no reason, so the reviewer can't tell it was considered)

| # | Concern | Answer |
|---|---|---|
| 1 | Server listing (DataTableQuery) | [Applies, see … / Not applicable, because …] |
| 2 | Static listing (DataTable) | |
| 3 | Form or wizard | |
| 4 | Row menu built from the server's allowed actions, with reasons | |
| 5 | Status lifecycle | |
| 6 | Approval by a second person | |
| 7 | Money written in one transaction | |
| 8 | Import from a file | |
| 9 | Preview, then commit | |
| 10 | Background or batch job | |
| 11 | Printed document | |
| 12 | Email or notification | |
| 13 | Report or dashboard figures | |
| 14 | Public link page (no staff sign-in) | |
| 15 | Operator or cross-school access | |
| 16 | A school setting that changes behaviour | |
| 17 | Platform-fixed values | |
| 18 | Outside provider (login, email, bank or rail, file storage) | |
| 19 | Depends on work in a later group | |
| 20 | Implied by the design but not drawn | |
| 21 | Differs from the design | |
| 22 | New module, or a new public contract function | |
| 23 | A call between modules (and its direction) | |
| 24 | Commons module copied or changed | |
| 25 | New or changed table, and existing rows to update | |
| 26 | Reference numbers | |
| 27 | Retry-safe commands (idempotency) | |
| 28 | Two people changing one record (concurrency) | |
| 29 | Audit entries (critical events, inside the transaction) | |
| 30 | Domain events (after-commit events and their reactions; "not applicable" only if the group has no audit entry, email or other effect on another concern) | |
| 31 | A pattern or part extracted for reuse | |
| 32 | Seed additions | |
| 33 | Phone width | |

---

## A3. Scope

> **[AI Guide]** Everything the group builds, everything it defers, and everything the design implies but doesn't draw. Walk every artboard of the group: each page, listing, card, menu, sheet, drawer, form, dialog and tab is a surface. Give each surface an ID and the checkpoint that builds it. No surface may be missing from all three tables.
>
> **Notices, toasts and states are not surfaces** and get no ID. Each is described inside the surface it appears on: a notice in its block's Notices field, a toast in the form or dialog's "After submitting" table, a state in the checkpoint's states table.

**IDs used in this plan:**
- Surfaces: `<kind>.<record>.<purpose>`, in kebab-case. Kinds: `page`, `table`, `query`, `form`, `dialog`, `sheet`, `drawer`, `tab`, `menu`, `card`. Example: `dialog.staff.change-roles`. In code, the surface's root element carries the ID as `data-testid`.
- **A new kind** is never invented quietly. The plan proposes it as a decision in A15 (question, answer, why), and C2 carries it to this template. The same applies to a new block.
- **In the web client,** the function that calls an endpoint is named after its action (`staff.change-roles` → `staffChangeRoles`), and a query's key is its query ID's record and purpose (`query.staff.list` → `['staff', 'list', …params]`). A search for an action or query ID then finds the screen, the client, the endpoint and the test.
- Screens, documents and emails keep their `SCR-NNN` IDs.
- Endpoints are named by their action (`staff.change-roles`) plus the method and path.
- Contract functions are `package.function` (`identity.getMe`).
- A business rule's ID is its error `code` (`STAFF_ROLES_EMPTY`). A field's validation rule is named by the form and field (`form.staff.invite › email: required`).
- Migrations follow the project's convention (`0003_identity__add_staff_roles.sql`). Events are past tense, in PascalCase (`StaffInvited`).
- Checkpoints are `B{GG}.{chunk}.{n}`; chunks are `B{GG}.{chunk}`; the close is `B{GG}.X`. Decisions in this plan are `D1`, `D2`…; design proposals are `P1`, `P2`….

**Surfaces built in this group:**

| Surface ID | Kind | SCR | Artboards | What it is | Built in |
|---|---|---|---|---|---|
| [page.record.purpose] | [Kind] | [SCR-NNN] | [Artboard canvas titles] | [One line] | [B{GG}.c.n] |

**Deferred: depends on later work.**

> **[AI Guide]** Something drawn in this group that can't be finished yet, because it needs a module, provider or group not yet built. Say what shows in its place now, and which group wires it later. What shows is never a placeholder or sample data: it is the part's **true current state** (real data, even if empty, such as "Not set up"), or the action **disabled with its reason** (exact text). Only a whole page or sidebar item of an unbuilt group is **hidden**. Where a seam stands in for the later work, name it and its honest default. Each row is also a Section A13 difference, because the built screen won't match the artboard until then.

| What | Why it waits | Shown meanwhile | Wired by |
|---|---|---|---|
| [Item] | [Needs …] | [True current state: … / disabled, with "exact reason" / hidden (whole page or sidebar item only)] | [Group or unit, and the seam it replaces] |

**Implied but not drawn.**

> **[AI Guide]** A surface the design clearly needs but has no artboard for, such as the dialog behind a drawn menu item. The plan proposes its content (title, text, buttons, outcome) in Section A13, and the PO agrees it before the plan is written. Then it is built like any other surface.

| What | Implied by | Proposal | Built in |
|---|---|---|---|
| [Surface] | [The drawn item that needs it] | [P{n}] | [B{GG}.c.n] |

---

## A4. Rules

> **[AI Guide]** Every business rule the group enforces, decided once here, because one rule often drives several surfaces and endpoints. Copy each rule's statement from its source, with a source stamp. "Enforced in" names every layer: the API always enforces it; the screen may only hint (hide, disable or explain); the database enforces it with a constraint where it can. Every rule has at least one test, named by its code.
>
> **Correct:** "`STAFF_LAST_ADMIN` | The school always keeps at least one active School admin | API: change roles, deactivate. Screen: the menu note and the Sheet notice. Database: none, because the rule spans rows | handoff decision 131 | `STAFF_LAST_ADMIN: refuses removing the last School admin`"
> **Incorrect:** "Admin rule | Can't remove admin | Backend" (no code, no statement a test can check, no surfaces)

**Business rules:**

| Code | Rule | Enforced in | Surfaces | Source | Test |
|---|---|---|---|---|---|
| [CODE] | [Statement] | [API / screen hint / database] | [Surface IDs] | [Stamp] | [Test name] |

**Field validation:**

> **[AI Guide]** One row per field rule. Messages come from the project's validation-message reference, required first ("Email is required", then "Email is invalid"). The same rule is checked in the form and again by the API, which returns 400.

| Form › field | Rule | Message |
|---|---|---|
| [form.record.purpose › field] | [required / format / length / range] | [Exact message] |

---

## A5. Actions and roles

> **[AI Guide]** Every action a person can take in this group, who may take it, and where it appears. The action name is the permission the server checks, and the endpoint's name. The screen never decides permissions by itself: it shows what the server allows (Section A6).

| Action | What it does | Roles | Surfaces | Endpoint | Second person? |
|---|---|---|---|---|---|
| [record.verb] | [One line] | [Roles] | [Surface IDs] | [METHOD /path] | [No / Yes, see B1 approval block] |

**Opening the group's pages:** [Who may open each page, and what everyone else sees: the page is missing from navigation, and a direct link shows the no-permission state]

---

## A6. Lifecycle and allowed actions

> **[AI Guide]** For each record with a status: every status, its badge, the actions allowed in it, and where each action takes it. Then every reason an action isn't allowed, with the exact text the person sees. The server works out the allowed actions and their reasons and sends them with each record. The screen only shows them, so status and role rules live in one place.
>
> Write "Not applicable: no record in this group has a status" if so.

**[Record] lifecycle:**

| Status | Badge | Allowed actions | Each action leads to |
|---|---|---|---|
| [Status] | [Label, colour] | [Actions] | [Action → new status] |

**When an action isn't allowed:**

| Action | Not allowed when | Reason code | Text shown | Where it shows |
|---|---|---|---|---|
| [Action] | [Condition] | [CODE] | [Exact text] | [Menu note / Sheet notice / disabled with hint / item hidden] |

**How allowed actions reach the screen:** [The response field that carries them, with its type, for example `allowedActions: { action: string; allowed: boolean; reasonCode?: string; reason?: string }[]`, sent with each row of the listing and with the single record]

---

## A7. Modules

> **[AI Guide]** Every module the group touches, including one that only gains configuration (actions added to the role map, an email added to the catalogue). Names come only from the project's architecture note; a module the note lacks is proposed as a decision in A15, and C2 carries it to the note. Each module has two kinds, from the note: **where its code comes from** (commons, commons-to-be, product) and **what it is for** (business, platform, integration, front end). Its **layer** decides which modules it may call. The API's top layer is always called **Applications**, with its folder, never given a module name.
>
> Every call between modules is one of four kinds: a **direct contract call** (downward only), a **critical event** (inside the transaction: audit), an **after-commit event** (emails, notifications) or a **registration** (a higher module registers a check or listener with a lower one, for an upward need).

| Module | Kind | Purpose | Layer | Folder | New or existing | What it gains in this group | Contract functions and events |
|---|---|---|---|---|---|---|---|
| [module] | [Commons / commons-to-be / product] | [Business / platform / integration / front end] | [Applications / orchestration / business layer / platform / infrastructure] | [Path] | [New / existing] | [Records, rules, endpoints, configuration] | [package.function; events raised, critical ones marked] |

**Calls between modules:**

| From | To | What for | Allowed by the layers? | How |
|---|---|---|---|---|
| [module] | [module] | [Purpose] | [Yes / no, so a registration] | [Direct contract call / critical event / after-commit event / registration: …] |

---

## A8. Data

> **[AI Guide]** The tables, and where every displayed value comes from. The field-source table is filled from the artboards: every column, field and fact a person sees gets a row, because a value from a provider or another module is easy to miss until the build. Migrations are specified in full in the checkpoint that writes them (the migration block); this is the summary.

**Tables:**

| Table | Module | New or changed | Key columns | Row security | Migration |
|---|---|---|---|---|---|
| [prefix_table] | [module] | [New / adds …] | [Columns] | [School-scoped / platform] | [File name, checkpoint] |

**Where each displayed value comes from:**

| Surface › field | Shown as | Source | Fetched by | Available now? |
|---|---|---|---|---|
| [table.staff.list › Last sign-in] | [Format] | [Our table / login provider / other module / calculated] | [Query or contract function] | [Yes / no: see deferred] |

---

## A9. Settings and constants

> **[AI Guide]** Every value that changes how the group behaves, and how it is loaded. Rules for every project, decided once: the "who am I" response carries only what the app shell needs; a page's settings come with that page's own query; school policy is owned by the configuration module; platform-fixed values are constants in the module that owns them. Write "None" only after checking the artboards' text for numbers such as days, limits and amounts.

| Value | Kind | Owner | Default | Loaded with | Used by |
|---|---|---|---|---|---|
| [Name] | [School policy / platform-fixed / shell] | [Module] | [Value] | [Query or constant] | [Surfaces, rules] |

---

## A10. What each command causes

> **[AI Guide]** For every command (every endpoint that changes data), everything that happens besides the change itself. A blank cell is an unanswered question; write "none" where nothing happens. An audit entry is a critical event, raised inside the transaction. An email or notification is sent by a reaction in the applications layer to an after-commit event.

| Endpoint | Audit entry (critical event) | Events after the commit | Emails or notifications | Retry-safe | Concurrency |
|---|---|---|---|---|---|
| [action] | [What is recorded] | [EventName, payload] | [SCR-NNN, to whom; the reaction that sends it] | [Idempotency key: required / not needed, why] | [Version check / not needed, why] |

---

## A11. Seed

> **[AI Guide]** The sample records this group adds, so every built state can be compared with its artboard using the same school, people and figures. Seeded through the modules' real contracts, never by inserting rows directly. Earlier groups' seed is assumed. Every state in the artboards needs a record that produces it.

| Records | Produces which state | Artboards | Through |
|---|---|---|---|
| [What, how many] | [For example, "an Invited person"] | [Artboard canvas titles] | [Contract function] |

---

## A12. Patterns and components

> **[AI Guide]** Every pattern and part the group uses, named from the project's pattern catalogue. For each, one decision: **reuse** (it exists), **extract now** (the design shows it twice or more, so it is built once here for later groups), or **one-off** (seen once; built inline). Say where an extracted part lives.

| Pattern or part | Catalogue key | Decision | Lives in | Used by |
|---|---|---|---|---|
| [Name] | [Key] | [Reuse / extract now / one-off] | [Path] | [Surface IDs] |

---

## A13. Differs from the design

> **[AI Guide]** Every place where the built group will differ from the approved artboards, and every proposal for something the design implies but doesn't draw. Each is a proposal the PO agreed before this plan was written, so the "Agreed" column is never empty in a written plan. Deferred items from Section A3 appear here as well, with what shows meanwhile.
>
> Write "None" only after comparing every surface in Section A3 with its artboard.

| # | Where | Design shows | Built shows | Why | Agreed |
|---|---|---|---|---|---|
| P1 | [Surface ID, artboard canvas title] | [What is drawn] | [What will be built] | [Reason] | [PO, date; decision log number if recorded] |

---

## A14. Reconciled with existing code

> **[AI Guide]** Where existing code or documents disagree with the approved design, and the fix. Name the file, table or section. Write "None found" only after looking at the code repos, not just the documents. Reconcile against the main line only. Code on an unmerged branch is reference: if the plan lifts any of it, list it here (what, from which branch and commit, and why), and it is reviewed with the plan like new code.

| # | What disagrees | Where | Fix | Fixed in |
|---|---|---|---|---|
| 1 | [Description] | [File, table or document section] | [Fix] | [B{GG}.c.n] |

---

## A15. Decisions

> **[AI Guide]** Every choice with a real alternative, written as question, answer and why. The "why" names the alternative that was rejected and the reason, so a reviewer can disagree with something specific. Tables elsewhere point here by number. Don't re-decide anything the architecture note or the design decision log already settled; copy it into the right table with its source stamp instead.
>
> **Correct:** "D3 | How does the staff list get Last sign-in? | Our identity module records it when it first sees a new sign-in session, and the list query reads it with the other columns | Asking the login provider per page would mean a network call on every list load, and would fetch every school's users from one shared project; joining the provider's tables would tie our code to the provider (architecture note §1, Portability)"
> **Incorrect:** "D3 | Last sign-in | From Supabase | Simple" (no question, no alternative)

| # | Question | Answer | Why |
|---|---|---|---|
| D1 | [Question] | [Answer] | [Reason, and the alternative rejected] |

---

# Part B — The checkpoints

---

## B0. Build order

> **[AI Guide]** How the group is cut. **A checkpoint is one behaviour a reviewer can see work:** usually one user action, built from the surface down to the database (screen, data loading, endpoint, module, migration), with its own tests. A checkpoint with nothing to see (such as a security foundation) stands alone only when it is security- or data-critical, and is proved through the API. Order checkpoints so each one builds only on earlier ones: first the listing that shows the records, then the action that creates them, then the actions that change them.
>
> **Chunks:** 2 to 5 per group, each one capability a named role can use, with a one-sentence demo. A chunk is one pull request; a checkpoint is one commit. The PO accepts each chunk by its acceptance steps before its pull request merges.
>
> **Size:** a checkpoint is small enough for the developer to review in one sitting: about 600 changed lines, not counting tests, generated files and migrations. Split a larger one by surface or by action, never by layer.
>
> **Where checks sit:** automated tests at each checkpoint, written in plain language here and approved with the plan; the developer's manual check at each checkpoint; the PO's acceptance steps at each chunk; the demo, the regression of every closed group, the financial checks and the deploy at the close.

| Checkpoint | Goal: what a reviewer sees work | Surfaces | Endpoints | Depends on | Status |
|---|---|---|---|---|---|
| **B{GG}.1 [Chunk name]** | **Demo:** [Role] [does what] | | | | |
| B{GG}.1.1 | [One sentence] | [Surface IDs] | [Actions] | [Checkpoint or unit] | Planned |

---

## B{GG}.1 [Chunk name]

**Demo:** [Role] [does what, end to end, in one sentence]

### B{GG}.1.1 [Goal, as a short title]

> **[AI Guide]** One section per checkpoint, in build order. Start with the four lines below, then copy in, from the Block library, every block this checkpoint needs, in this order: screen blocks (page, then listing, card, menu, sheet, drawer, form, dialog, in the order a person meets them), the states table, data loading, endpoints, then the module and migration. Add a non-screen block (money command, approval, import, job, document, email, report, public link, cross-school) when the checkpoint does that kind of work. Finish with Verification. A block that doesn't apply is left out, not written as empty. Each block is filled in full: the build reads only Part A and this section.

**Goal:** [What a reviewer can see working when this checkpoint is done]
**Depends on:** [Earlier checkpoints and units]
**Builds:** [Surface IDs, endpoints, migrations]
**Rules enforced here:** [Codes from Section A4, each with its statement]

[Blocks from the Block library]

**Verification**

[Verification block]

**Status:** Planned

---

**B{GG}.1 acceptance (PO)**

> **[AI Guide]** At the end of each chunk: the steps the PO follows on the running app before the chunk's pull request merges. Each step has its expected result and, where there is one, the artboard to compare with. Written for the PO, in the screen's own words, with the seed's sample people and figures.

| # | Step | Expected | Compare with |
|---|---|---|---|
| 1 | [Sign in as … and do …] | [What shows] | [Artboard canvas title] |

---

# Part C — The close

---

## C1. Close, B{GG}.X

> **[AI Guide]** Built last by `/build-checkpoint`. The group's demo end to end, the browser tests of every closed group (regression), the financial checks if money moved, the merge, and the deploy to staging. The PO and the developer both sign off.

**Demo:**

| # | Step | Expected |
|---|---|---|
| 1 | [Step] | [Result] |

**Regression:** [Every closed group's browser tests, by name]

**Financial checks:** [Which, or "Not applicable: no money moves in this group"]

**Sign-off:** PO: [date]. Developer: [date].

**Built B{GG}.X:** [Added at close]

---

## C2. Proposed changes for their owners

> **[AI Guide]** What this group's planning or build showed should change in documents it doesn't own: the architecture note (a new module), the pattern catalogue, commons copies, this template (a new surface kind or block). Proposed here; each owner applies them.

| Document | Change | Why | Applied |
|---|---|---|---|
| [Document] | [Change] | [Reason] | [Date, or "Not yet"] |

---

## C3. Findings for later groups

> **[AI Guide]** Anything found while planning or building that a later group must act on, with its destination. `/build-checkpoint` appends here; `/plan-build-group` reads this section of every earlier plan.

| Found in | What | Goes to |
|---|---|---|
| [Checkpoint ID] | [Description] | [Group or document] |

---

# Block library

> **[AI Guide]** The standard blocks Part B's checkpoints are built from. Copy the blocks a checkpoint needs into it and fill every field; a field that doesn't apply says "Not applicable" and why. The library itself is removed from the written plan. Every block names the catalogue pattern it follows, so the developer knows which existing code it extends.
>
> **Each surface kind has its block:** `page` → Page block; `tab` → the Page block's Tabs field; `table` → Static listing or Server listing block; `card`, `menu`, `sheet`, `drawer`, `form`, `dialog` → their own blocks; `query` → Data loading block. A block is never invented in a plan: a new one is proposed as a decision in A15, and C2 carries it to this template.

---

**Page block** (catalogue: page recipe and app shell)

| Field | Value |
|---|---|
| Surface ID | [page.record.purpose] |
| SCR and artboards | [SCR-NNN; artboard canvas titles] |
| Route | [Path, its path parameters, and its query parameters] |
| Opened from | [Navigation item, link or button] |
| Who can open it | [Action, roles; everyone else sees …] |
| Recipe | [Recipe name from the design system] |
| Title and subtitle | [Exact text] |
| Header actions | [Button text → action → where it goes] |
| Notices | [Exact text, and the rule or setting that shows it] |
| Tabs | [Tab IDs (`tab.record.purpose`) and labels, the default, and whether the tab is in the URL; or "none". Each tab's contents are the blocks that follow, in tab order] |

**Static listing block** (catalogue: static listing)

> **[AI Guide]** For a list loaded in full, searched and filtered in the browser. Say why that is enough and its limit (how many rows it can hold before it must become a server listing).

| Field | Value |
|---|---|
| Table ID | [table.record.purpose] |
| Why static | [Reason, and the limit: "a school has at most a few dozen staff"] |
| Loaded by | [Query key, from the data loading block] |
| Search | [Fields searched] |
| Filters | [Field, options, default] |
| Sort | [Default, and which columns sort] |
| Selection and bar actions | [None / checkboxes, select all, bar actions] |
| Row click | [Opens which sheet or page] |
| Row actions | [Menu ID, see the menu block] |
| Empty | [Text with no rows; text when a search or filter finds nothing] |

| # | Column | Header | Cell | Format | Frozen | Shown by default | Source |
|---|---|---|---|---|---|---|---|
| 1 | [key] | [Header text] | [Part, catalogue key] | [How it's formatted, e.g. "first two roles, then +N"] | [Yes / no] | [Yes / no] | [Row in A8's field sources] |

**Server listing block** (catalogue: server listing, screen half and server half)

> **[AI Guide]** For a list loaded page by page from the server, with queries, filters and sort done there. Every query is its own row, and each has its own server-side line. Follow the API contract's list rules (shape, query parameters, export).

| Field | Value |
|---|---|
| Table ID | [table.record.purpose, also the `tableId` that remembers each person's settings] |
| Why server | [Reason: row count, or filters across data the browser can't hold] |
| Page size | [Default and options] |
| Selection and bar actions | [None / checkboxes, select all across pages or not, bar actions] |
| Row click | [Opens which sheet or page] |
| Row actions | [Menu ID, see the menu block] |
| Export | [Formats, columns, row limit; or "none"] |
| Empty | [Text with no rows; text when a query or filter finds nothing] |

Columns:

| # | Column | Header | Cell | Format | Frozen | Shown by default | Sortable | Source |
|---|---|---|---|---|---|---|---|---|
| 1 | [key] | [Header] | [Part] | [Format] | [Yes / no] | [Yes / no] | [Yes / no] | [A8 row] |

Queries:

| Query ID | Label | Shows | Fixed parameters | Default sort | Default query? |
|---|---|---|---|---|---|
| [query.record.purpose] | [Label on screen] | [Which records] | [Parameters it always sends] | [Column, direction] | [Yes / no] |

Advanced filter fields:

| Field | Type | Operators | Options from |
|---|---|---|---|
| [Field] | [Text / number / date / choice] | [Operators] | [Fixed list / lookup endpoint] |

Server side:

| Query ID | Endpoint | Parameters | How filters map to the data | Index needed |
|---|---|---|---|---|
| [query.record.purpose] | [METHOD /path] | [Parameters] | [Filter → column or join] | [Index, or "none, because …"] |

**Card block**

| Field | Value |
|---|---|
| Surface ID | [card.record.purpose] |
| What it shows | [Contents, copied from the artboard] |
| Loaded by | [Query key; every list on a page has its own query, even one that returns fixed values today] |
| States | [In the states table] |

**Menu block** (catalogue: row menu by status and role; allowed actions from the server)

| Field | Value |
|---|---|
| Menu ID | [menu.record.purpose] |
| Opens from | [Row's ⋯ button / Sheet header] |

| Item | Action | Shown when | Not allowed when | Reason shown | Opens |
|---|---|---|---|---|---|
| [Item text] | [Action] | [Status and role] | [Reason code from A6, or "—"] | [Exact text] | [Dialog, page or sheet ID] |

**Sheet block** (catalogue: read-only Sheet)

| Field | Value |
|---|---|
| Sheet ID | [sheet.record.purpose] |
| Opens from | [Row click / menu item / a cell] |
| Title | [Exact text, with the record's fields it uses] |
| Loaded by | [The list row's data, or its own query, and why] |
| Changes | [Read-only: changes are made from the row menu] |

| Section | Contents | Source |
|---|---|---|
| [Navigator label] | [Fields and facts, exact labels] | [A8 rows] |

Notices: [Exact text, and the rule that shows each one]

**Drawer block** (catalogue: drawer)

| Field | Value |
|---|---|
| Drawer ID | [drawer.record.purpose] |
| Opens from | [Button, row or menu item] |
| Side and width | [Side, width, and its phone layout] |
| Title | [Exact text, with record fields] |
| Loaded by | [Query key, or the opener's data, and why] |
| Contents | [Sections and fields, exact labels; or the form block it holds] |
| Closes | [How it closes, and what happens to unsaved input] |

Notices: [Exact text, and the rule that shows each one]

**Form block** (catalogue: the form rule; form, end to end)

> **[AI Guide]** In "After submitting", every 422 row says which text shows: the screen's own (fixed text) or the API's `message` (text with data). Nothing is read from an error besides its `code` and `message`. A 400 shows the API's messages as a bulleted list in the form's error alert: the contract's 400 carries no field names, and the form checks the same rules before sending, so a 400 is rare.

| Field | Value |
|---|---|
| Form ID | [form.record.purpose] |
| Kind | [Page / dialog / sheet / wizard, and why, by the form rule] |
| Submit | [Button text → action → endpoint] |
| Retry-safe | [Idempotency key sent / not needed, why; the button is disabled while sending] |
| Leaving with changes | [What happens] |

| Field | Label | Control | Required | Rules | Hint | Default |
|---|---|---|---|---|---|---|
| [field] | [Label] | [Input / choice cards / select…] | [Yes / no] | [Rules from A4's field validation] | [Exact hint text] | [Value] |

Form-level rules: [Rules across fields, with their messages; or "none"]

After submitting:

| Response | What the person sees | Where | Goes to | Refreshes |
|---|---|---|---|---|
| Success | [Toast text, exact] | [Toast / inline] | [Page] | [Query keys] |
| 400 validation | [The API's messages, as a bulleted list] | [Form alert] | [Stays] | [—] |
| 422 [CODE] | [Exact text] | [Field or form alert] | [Stays] | [—] |
| 403 | [Text] | [Form alert] | [Stays] | [—] |
| 409 version | [Text] | [Form alert, with reload] | [Stays] | [The record] |
| Network or server error | [Text and request ID] | [Form alert] | [Stays] | [—] |

**Dialog block** (catalogue: consequence confirmation)

| Field | Value |
|---|---|
| Dialog ID | [dialog.record.purpose] |
| Opens from | [Menu item or button] |
| Title | [Exact text, with record fields] |
| Description | [Exact text] |
| Inputs | [None / see the form block] |
| Buttons | [Cancel; confirm button text and style] |
| Calls | [Endpoint] |

After confirming: [The same response table as the form block]

**States table** (catalogue: four states)

> **[AI Guide]** Every surface that loads data gets its own row. With parallel queries each surface fails on its own; with one combined query the page has one row.

| Surface | Loading | Empty | Error | No permission |
|---|---|---|---|---|
| [Surface ID] | [Skeleton shape] | [Exact text] | [Exact text, Retry] | [Exact text] |

**Data loading block**

| Field | Value |
|---|---|
| One query or several | [One combined query / parallel queries per surface; decision D{n}] |

| Query ID | Query key | Service function | Endpoint | Response type | File | Used by | Refetched when |
|---|---|---|---|---|---|---|---|
| [query.record.purpose] | [['record', 'purpose', …params]] | [Named after the action, e.g. `staffList`] | [METHOD /path] | [Type name] | [Path in the module's folder, e.g. `platform/identity/api.ts`] | [Surface IDs] | [After which actions] |

**Endpoint block** (catalogue: error contract, named actions, idempotency, optimistic locking, tenancy)

| Field | Value |
|---|---|
| Action | [record.verb] |
| Method and path | [METHOD /api/v1/path] |
| Lives in | [The controller's folder: the module's `.api` package for a commons-to-be module; the applications layer for a business module] |
| Who | [Action; roles] |
| Tenancy | [School-scoped through row security / platform / cross-school, see that block] |
| Request | [Type name, fields, and their 400 validation] |
| Response | [Type name and shape; lists follow the contract's list shape] |
| Rules | [422 codes from A4, each with when it is returned] |
| Other errors | [403, 404, 409: when each happens] |
| Retry-safe | [Idempotency key required, and what a repeat returns / not needed, why] |
| Concurrency | [Version field checked / not needed, why] |
| Audit | [What is recorded, as a critical event inside the transaction] |
| Events | [After the commit: EventName, and its payload] |
| Emails | [SCR-NNN, to whom; the reaction that sends it, on which event] |
| Settings read | [From A9] |
| Performance | [Expected rows, and the index used] |

```ts
// Only where the shape must be exact: request and response types
```

**Module block** (catalogue: module package)

| Field | Value |
|---|---|
| Module | [Name; kind, purpose and layer from A7] |
| Folder | [Path, by the architecture note's layout] |
| Contract functions | [package.function: what each does] |
| Events raised | [EventName: critical or after the commit, and its payload] |
| Service methods | [What each does, and the rules it checks] |
| Data access | [Queries and writes, and the table each touches] |
| Calls to other modules | [From A7: direct contract call, critical event, after-commit event or registration] |

**Migration block**

| Field | Value |
|---|---|
| File | [Name, by the project's convention] |
| Creates or changes | [Table] |
| Row security | [Policy] |
| Grants | [To the app role] |
| Existing rows | [How they are updated, or "none exist yet"] |
| Reversible | [How, or why not] |

| Column | Type | Null | Default | Constraint |
|---|---|---|---|---|
| [name] | [type] | [Yes / no] | [Value] | [Check, unique, foreign key] |

Indexes: [Each, and the query it serves]

**Money command block** (catalogue: unit of work and the money pattern)

| Field | Value |
|---|---|
| Command | [Action] |
| Written in one transaction | [Every record written, in order] |
| Counted once | [How a payment or charge can't be counted twice] |
| Retry-safe | [Idempotency key, with the unit of work] |
| Audit | [A critical event, recorded inside the transaction] |
| Events | [Published after the commit, through the unit of work's after-commit step or the outbox] |
| Financial checks | [Outstanding recomputable from records; allocations never exceed payments; others] |
| Breaks to prove the tests | [Each money rule broken on purpose, and the test that must fail] |

**Approval block** (catalogue: approval by a second person)

| Field | Value |
|---|---|
| Request type | [Name] |
| Raised by | [Roles] |
| Approved by | [Roles; never the person who raised it] |
| On approve | [What happens] |
| On reject | [What happens, and the reason asked for] |
| Expires | [When, or "never"] |
| Shown in | [Surfaces] |
| Handler | [Module that applies it] |

**Import block** (catalogue: import; preview, then commit)

| Field | Value |
|---|---|
| File | [Type, columns, the template offered for download] |
| Row checks | [Each check and its message] |
| Preview | [What shows before committing] |
| Commit | [All rows or none / valid rows only; and why] |
| Result | [What is reported after, and where] |
| Runs as | [In the request / background job] |

**Background job block** (catalogue: background jobs with progress and retry)

| Field | Value |
|---|---|
| Job | [Name] |
| Started by | [Action or schedule] |
| Unit of work | [What one step does] |
| Progress shown | [Where and how] |
| Retry | [What is retried, how often] |
| Partial failure | [What happens to the steps that succeeded] |
| Retry-safe | [How a repeated step is ignored] |

**Document block** (catalogue: documents; reference numbers)

| Field | Value |
|---|---|
| Document | [SCR-NNN, name] |
| Made when | [Trigger] |
| Number | [Reference number scheme, or "none"] |
| Fields | [Each field and its source] |
| Kept | [Stored, and for how long; reprint rules] |

**Email block** (catalogue: emails)

| Field | Value |
|---|---|
| Email | [SCR-NNN, name] |
| Sent when | [The after-commit event, and the reaction in the applications layer that sends it] |
| To | [Who] |
| Subject | [Exact text] |
| Body fields | [Each field and its source] |
| Link | [What it opens, and how long it works] |
| If it fails | [Bounce or failure handling, and where it shows] |

**Report block** (catalogue: report frame)

| Field | Value |
|---|---|
| Report | [SCR-NNN, name] |
| Measures | [Each figure, and exactly how it is calculated] |
| As at | [The moment the figures are taken] |
| Filters | [Each filter and its default] |
| Export | [Formats] |
| Performance | [Rows read, and how it stays fast] |

**Public link block**

| Field | Value |
|---|---|
| Page | [SCR-NNN] |
| Link | [How the link identifies the record, and how long it works] |
| Shows | [What, and what it never shows] |
| Abuse limits | [Rate limits, guessing protection] |
| Phone width | [Layout at phone width] |

**Cross-school block** (catalogue: support access mode)

| Field | Value |
|---|---|
| Who | [Operator roles] |
| Scope | [Which schools, and how a school is chosen] |
| Recorded | [Audit of every access] |
| Never allowed | [What an operator can't do] |

**Verification block**

> **[AI Guide]** Written in plain language and approved with the plan. Every rule enforced in this checkpoint has a test named by its code. At least one test fails without this checkpoint's change.

Automated tests:

| Test | Level | Proves |
|---|---|---|
| [CODE: behaviour, or "surface: behaviour"] | [Unit / database / API / browser] | [One line] |

Developer's check:

| # | Step | Expected |
|---|---|---|
| 1 | [Step] | [Result] |

Design parity: [Each artboard compared, by canvas title, the state it shows, and the seed that produces it]

**Built block** (added by `/build-checkpoint`, at most 10 lines)

**Built B{GG}.c.n** ([Date], [time taken]): [Result in one sentence]
- Files: [By area, linked]
- Differences from the plan: [Each, or "none"]
- Tests: [N passed, where; parity result; breaks caught if run]
- Commit: `[proposed message]`

---

## Plan checks

> **[AI Guide — Verification]** `/plan-build-group` runs these twice: on its decisions before it asks the PO anything (its Step 5), so everything needing the PO is asked at once, and again on the written plan. It fixes every failure. The written plan keeps this section, ticked, so the reviewers see the checks were made.

- [ ] A2: every row answered, none blank, none deleted
- [ ] A3: every surface on the group's artboards is built, deferred or implied, each with an ID and a checkpoint
- [ ] A3: IDs follow the scheme; no new surface kind or block without a decision in A15
- [ ] A3: no notice, toast or state has its own ID
- [ ] A3: nothing deferred shows a placeholder or sample data; each shows its true current state, or is disabled with its reason, or (a whole page or sidebar item only) is hidden
- [ ] A4: every rule has a code, a statement, where it is enforced, a source stamp and a test
- [ ] A4: every form field rule has its exact message
- [ ] A5: every action has its roles, its surfaces and its endpoint
- [ ] A6: every status has its allowed actions; every refusal has its exact text and where it shows
- [ ] A7: every module touched is listed, by its name in the architecture note, with its kinds, layer and folder; the top layer is called Applications
- [ ] A7: every call between modules is a direct contract call allowed by the layers, a critical event, an after-commit event or a named registration
- [ ] A8: every displayed value has a source; every value not available yet is deferred
- [ ] A9: every number in the artboards' text (days, limits, amounts) has an owner
- [ ] A10: every command has all five columns answered; every audit entry is a critical event and every email has its after-commit event and reaction; no module calls audit or notification directly
- [ ] A11: every artboard state has a seed record
- [ ] A13: every difference and proposal has the PO's agreement
- [ ] A15: every decision has a question, an answer, and a why naming the rejected alternative
- [ ] Part B: every checkpoint is one visible behaviour, self-contained, within the size limit, with every block it needs filled in full
- [ ] Part B: every list on a page has its own query; every page states one query or parallel; every surface that loads has its states
- [ ] Part B: every server listing has every query, filter field and server-side line
- [ ] Part B: every chunk has acceptance steps
- [ ] Every response table reads only an error's `code` and `message`; data a refusal needs comes from a query, not the error; a 400 shows as a list in the form alert
- [ ] Every artboard is named by its canvas title; every client function and query key carries its action or query ID; every file path is in its module's folder
- [ ] No "TBD", no "to be decided", no fact given only as a link; no person named except as seed data
