# Review Architecture

You are running the architecture review for an Ascendra client project. This is the mandatory quality gate between `/gen-architecture` and the Product Owner locking the document. No story generation or implementation begins until this review is complete and the architecture is Locked.

The review is a sequential, concern-by-concern walkthrough of five areas: module structure, data model, API contracts, security, and infrastructure. For each concern: the AI frames what that part of the architecture will produce in plain PO language, runs structural checks, surfaces FLAGS, applies fixes, and gets PO confirmation before moving to the next concern.

The architecture document is the most expensive artefact to get wrong. Every table name, column, endpoint, auth rule, and enum value becomes law for every story and every line of code. A requirement misunderstood here costs: BRD update → architecture revision → story updates → re-implementation → re-verify → re-PR → re-UAT. This review exists to catch those misalignments before that cascade can happen.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse the arguments and gate check

Extract from `$ARGUMENTS`:
- **PROJECT_CODE** — required (e.g. `HARBORVIEW-INV-001`)
- **Architecture path** — optional. Default: `projects/{PROJECT_CODE}/architecture/arch-v1.md`

If PROJECT_CODE is missing, ask:
> "Which project is this architecture review for? Provide the project code (e.g. `HARBORVIEW-INV-001`)."

**Gate check — architecture must not already be Locked:**

Read the `**Status:**` field from the architecture document header.
- If `Draft` or `Under Review` → proceed
- If `Locked` → stop: > "This architecture document is already Locked. To modify it, run `/assess-change` and describe the change needed."

---

## Step 2 — Read required files

Read every file below in full before starting the walkthrough.

1. The architecture document at the path resolved in Step 1 — read every section including Change History
2. The approved BRD in `projects/{PROJECT_CODE}/brds/` — source of truth for requirements, user roles, integrations, and business rules
3. Domain knowledge file(s) in `projects/{PROJECT_CODE}/domain/` — required for Concern B (state machine completeness)
4. `projects/TEMPLATE/architecture/arch.template.md` — specifically its Part 2 — Verification checklist. Every item in that checklist is either implemented by a numbered check below (cited inline as *(Part 2: "...")*) or explicitly marked generation-time-only (already verified in `gen-architecture.md` Step 11/12, not re-verified here). Nothing in Part 2 is silently uncovered — that silent-drift failure mode is exactly what this step exists to prevent, after two independently-maintained checklists were found to have drifted apart.
5. Every file present in `projects/{PROJECT_CODE}/standards/` — the confirmed tech stack, architecture pattern, and every generated standard (cross-cutting, per-technology, and any additional pattern document). Do not assume a fixed list of filenames; read whatever is actually there.
6. `projects/{PROJECT_CODE}/screens/screen-design.md`, if it exists — required for Concern A check 11, to confirm Section 3.1.3.1–3.1.3.3 were carried over correctly rather than re-derived.

If no standards files exist, the checks fall back to the universal technology-agnostic principles embedded in the architecture template.

---

## Step 3 — Review Record: initialise or resume

Check Section 11 of the architecture document.

---

**Fresh session (Section 11 does not exist):**

Append Section 11 to `projects/{PROJECT_CODE}/architecture/arch-v1.md` now:

```markdown
## Section 11 — Review Record

**Review started:** {today's date}
**Review status:** In Progress

| Concern | Description | Flags | Status |
|---------|-------------|-------|--------|
| A | Module Structure | 0 | Pending |
| B | Data Model | 0 | Pending |
| C | API Contracts | 0 | Pending |
| D | Security & Access Control | 0 | Pending |
| E | Infrastructure & Compliance | 0 | Pending |

## Open Flags

| # | Concern | Section | Issue | Tier | Status |
|---|---------|---------|-------|------|--------|

## Changes Applied

| Concern | Section | Change |
|---------|---------|--------|

## Review Summary

*(Filled on completion)*
```

Also update the architecture document `**Status:**` header to `Under Review`.

Proceed to Step 4 starting from Concern A.

---

**Resumed session (Section 11 exists, status `In Progress`):**

Read Section 11. Find the first concern with status `Pending` or `In Progress`. That is the resume point.

> "Resuming architecture review for {PROJECT_CODE}. Completed so far: [list Reviewed concerns]. Picking up from Concern [letter]: [Description]."

Jump to Step 4 at the resume point.

---

**Review already complete (Section 11 exists, status `Complete`):**

> "The architecture review is already complete per Section 11. Current document status: `{status}`. Run `/update-status projects/{PROJECT_CODE}/architecture/arch-v1.md Locked` if it is not yet Locked."

---

## Step 4 — Per-concern walkthrough

For each concern in sequence A through E, starting from the resume point if resuming.

**Fix tiers — apply within each concern as FLAGS are surfaced:**

**Tier 1 — Apply immediately, no pause:**
- Typo or label error in any field, heading, or table cell
- Section naming inconsistency between Section 3 module names and Section 4/5 references
- Missing `updatedAt` trigger on a table that clearly has an `updatedAt` column *(Part 2: "Every table with an `updatedAt` column has a trigger declaration" — handled here as an auto-fix, not as a separate numbered check below)*
- Part 2 verification item unticked that clearly passes

**Tier 2 — Present and confirm before applying:**
- Missing column that a stated requirement obviously needs
- Money column typed as `decimal` or `float` → correct to `integer`
- Missing enum value for a documented lifecycle state
- Missing endpoint for a module function clearly required by BRD scope
- `orgId` in a DTO input field → remove it
- State transition described as PATCH on a status field → convert to POST with action path segment
- BRD requirement with no covering module → assign to appropriate module and add endpoints
- Sensitive field not identified in Section 6.4 → add it with its protection mechanism

**Tier 3 — PO decides before executing:**
- Module boundary change (merge two modules or split one)
- New module addition for an unaddressed BRD feature area
- New lifecycle state not currently in any enum
- Role enum addition or removal
- External service added to or removed from scope
- Open decision in Section 8 that requires PO direction to resolve

**Tier 4 — Halt:**
- Multiple BRD requirements with no covering module (architecture structurally incomplete — rewrite needed)
- Multiple open decisions in Section 8 unresolved (PO must resolve before review continues)

---

### A — Module Structure

**Frame this concern:**

Present the module structure in plain PO language before running any checks.

> "**Concern A — What this system will contain:**
>
> This architecture defines [N] modules. Each one owns a distinct capability:
> [For each module in Section 3: module name — one sentence describing what business capability it owns and what it delivers, in plain language without technical jargon]
>
> Every BRD requirement maps to one of these modules:
> [For each FR-XXX in the approved BRD: FR-ID — the module that owns it]
>
> External services to be integrated: [list from Section 7.2 and which module handles each — or 'None']
>
> Repo topology and internal layering confirmed for this project: [state what's recorded in `system-architecture.md` — e.g. 'two repos (api + web), Controller → Service layering' — or a different confirmed pattern]
>
> [If Section 3.2 (Extension Points) is populated: "This project is designed for extension along [N] dimension(s): [list from BRD Section 1.6]. The seams this base system exposes for each: [one sentence per extension point]." If "None," omit this line.]
>
> [If Section 3.1.3 is populated: "This project also has [N] portal(s) with UI mocks generated: [list portal names]. Open each mock alongside its Screen Inventory rows below and confirm the layout matches your expectations, not just the row text."]
>
> Does this module breakdown match your mental model? Is any capability missing a home?"

Wait for PO confirmation.

**Run Concern A checks:**

1. **System Overview is jargon-free** *(Part 2: "System Overview contains no technical implementation detail")* — Section 1 contains no module names, API routes, table names, or technology choices. Flag any technical term found there.
2. **BRD FR coverage** — Every FR-XXX in the approved BRD has at least one module in Section 3 whose scope addresses it. Flag any FR-XXX with no covering module.
3. **External service module coverage** *(Part 2: "Every external service in Section 7.2 has a corresponding module in Section 3")* — Every external service in Section 7.2 has a dedicated module in Section 3. Flag any service without a module.
4. **Module-table coverage** — Every module in Section 3 owns at least one table in Section 4.2. Exception: a pure integration module that delegates all storage to another module's table (e.g. `EmailModule` writing only to `notification_logs`) is acceptable if the exception is noted explicitly.
5. **Module-endpoint coverage** — Every module in Section 3 has at least one endpoint in Section 5. Flag any module with no endpoints.
6. **Endpoint-module reverse coverage** *(Part 2: "Every endpoint in Section 5 references a module that exists in Section 3")* — Every endpoint in Section 5 belongs to a module actually listed in Section 3. Flag any endpoint under an unlisted or misspelled module name.
7. **Module single responsibility** *(Part 2: "Every module in Section 3 has a single, bounded responsibility")* — No module in Section 3 covers two unrelated domains, and no module is a catch-all (`Utils`/`Helper`/`Common` or equivalent). Flag any module that reads as two responsibilities merged.
8. **Project structure specified** *(Part 2: "Section 3.1 specifies both the backend folder structure and the frontend setup approach")* — Section 3.1 states both the backend structure (3.1.1) and the frontend setup approach (3.1.2), consistent with the confirmed stack. Flag if either is missing or left as template placeholder text.
9. **Architecture pattern stated explicitly** *(required by `gen-architecture.md`'s Architecture Pattern Discovery step, not a numbered Part 2 item)* — the confirmed internal layering pattern and repo topology are stated explicitly at the top of Section 3 / in Section 3.1, not left for the reader to assume the framework default. Flag if either is absent or only inferable from the tech stack table.
10. **Extension Points presence** *(Part 2: "Section 3.2 is present — either lists extension points derived from BRD Section 1.6, or explicitly states 'None'")* — Section 3.2 is present and, if BRD Section 1.6 names extension dimensions, applies `reference/architecture/layered-domain-architecture.md`'s Core/Extension model to identify a real technical seam per dimension — not a placeholder or a restatement of the BRD dimension without a seam. Flag if Section 1.6 names dimensions but Section 3.2 says "None," or if Section 3.2 is missing entirely.
11. **Screen & Navigation Map carried over correctly** *(Part 2: "Section 3.1.3 (Screen & Navigation Map) is present whenever a UI module is in scope")* — if `projects/{PROJECT_CODE}/screens/screen-design.md` exists, confirm Section 3.1.3.1–3.1.3.3 match it exactly (Portals, Navigation Map, Screen Inventory including Surface/UI Pattern/Matched Reference per screen) — this is a carry-over check, not a fresh derivation check; the underlying content was already reviewed and approved in `/review-screen-design`. "None" is only valid for a project with no UI module at all, confirmed at `/gen-screen-design`'s own gate. Flag any discrepancy between the carried-over content and the approved `screen-design.md`.
12. **Action Gating and Cross-Cutting UI Rules generated fresh** *(Part 2: "If Section 3.1.3 is populated, every journey in BRD Section 4 maps to at least one Screen ID in 3.1.3.3")* — 3.1.3.4 and 3.1.3.5 are the two subsections this document actually originates (everything else is carried over per check 11). Confirm 3.1.3.4 covers every multi-role screen needing finer-than-page gating, and 3.1.3.5 correctly reflects Section 6.1's `viewOnly` and Section 6.3's delegated-access status. Journey coverage itself (every journey has a screen) was already verified in `/review-screen-design` — do not re-derive it here, just confirm nothing was lost in the carry-over.
13. **UI mock patch pass completed** — if Section 3.1.3 is populated, every portal in 3.1.3.1 has a corresponding file at `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html` (generated in `/gen-screen-design`'s base pass), and that file now includes gating captions for every 3.1.3.4 row and a banner for every 3.1.3.5 rule that applies — confirming `/gen-architecture`'s Step 13.5 patch-mode invocation of `/gen-ui-mocks` actually ran. Flag any portal with no mock file, or a mock file missing gating/cross-cutting content that 3.1.3.4/3.1.3.5 define. If Section 3.1.3 is "None," this check does not apply.

**Present FLAGS:**

```
FLAGS — Concern A: Module Structure
[Numbered list. Each flag: check number, section in arch-v1.md, precise issue, fix tier.]
[If none: "No flags — all Concern A checks passed."]
```

Apply fixes by tier. For Tier 2, present the proposed change and wait for confirmation. For Tier 3, state the issue and wait for the PO to decide the approach.

**Targeted PO question:**

> "Is there any capability you discussed with the client during discovery that does not have an obvious home in any of the [N] modules listed? Think in the client's language — sometimes a feature they named maps to a module they would not recognise by its technical name.
>
> [If Section 3.1.3 is populated:] Does the screen inventory in 3.1.3 cover every portal and journey you expect, or is anything missing a screen?
>
> [If Section 3.2 is populated:] Do the extension seams listed match where you actually expect a future extension project to plug in?"

Wait for PO response. Apply any additions raised.

**Update Section 11:**

Set Concern A Status → `Reviewed`. Update Flags count and log any changes in Changes Applied.

> "Concern A complete. Moving to Concern B: Data Model."

---

### B — Data Model

**Frame this concern:**

Present the data model in plain PO language.

> "**Concern B — What this system will store:**
>
> The database will have [N] tables. The core entities and their lifecycles:
> [For each entity with a status enum in Section 4.3: entity name — states in lifecycle order, e.g. 'Invoice: Draft → Issued → Paid | Overdue → Voided']
>
> Supporting tables: [list lookup, reference, and audit tables and what they hold]
>
> Seed data required before the system can be used for the first time: [list from Section 4.4 — or 'None']
>
> Does the data model capture everything the system needs to store and track?"

Wait for PO confirmation.

**Run Concern B checks:**

14. **Table ownership completeness** *(Part 2: "Every table in Section 4.2 is owned by exactly one module listed in Section 3")* — Every table in Section 4.2 is assigned to exactly one module. No orphaned tables. No table claimed by two modules.
15. **State machine completeness** — For each entity with a status enum, the values in Section 4.3 cover every lifecycle state defined in the domain knowledge document. Flag any domain-knowledge state absent from the corresponding enum.
16. **User role enum completeness** — Every user role defined in BRD Section 3 appears as a value in the role enum in Section 4.3. Flag any missing role. *(Distinct from check 28 in Concern D, which checks the RBAC table in Section 6.2 — this check is about the Section 4.3 enum specifically.)*
17. **Money column types** *(Part 2: "No money column in any table uses `decimal` or `float`")* — Every column storing a monetary value uses `integer` type (smallest currency unit — paisa, pence, cents). Flag any money column typed as `decimal`, `numeric(x,y)`, `float`, or any non-integer type. Exception: FX rate columns store ratios, not amounts — `numeric(20,8)` is correct for those.
18. **Mandatory columns present** *(Part 2: "Every Master table in Section 4.2 includes all mandatory Master columns"; "Every Detail table in Section 4.2 includes all mandatory Detail columns")* — every Master table has `id`, `orgId`, `version`, `createdAt`, `updatedAt`, `createdBy`; every Detail table has `id`, `orgId`, `createdAt`, `updatedAt`. Flag any table missing one.
19. **Seed data section present** *(Part 2: "Section 4.4 (Seed Data) is present")* — Section 4.4 exists and either lists seed rows or explicitly states none are required. Flag if missing entirely.
20. **Lookup/Reference seed coverage** *(Part 2: "Every Lookup and Reference table in Section 4.2 has corresponding seed data entries in Section 4.4")* — every Lookup and Reference table has a matching entry in Section 4.4, or a stated reason none is needed at startup. Flag any without either.
20a. **Domain-specific actor attribution symmetry** *(`FW-039`)* — for every table with more than one domain-specific lifecycle timestamp (`assigned_at`/`revoked_at`, `requested_at`/`authorized_at`/`revoked_at`, `submitted_at`/`reviewed_at`, and similar — distinct from the generic `created_by`/`updated_by`/`deleted_by` triad checked in check 18), confirm each transition attributed to a specific human role in Section 6.2 has its own paired `_by` column, per `database-standards.md` Rule 4a's judgment. Flag a table where one transition on the same table has a paired `_by` column and a sibling transition doesn't, unless the unpaired one is `System`/webhook-triggered or a future deadline/schedule field (state the reason inline where flagging is not warranted). Do not flag a missing `_by` for a transition that's correctly unpaired under Rule 4a.

**Present FLAGS:**

```
FLAGS — Concern B: Data Model
[Numbered list. Each flag: check number, table or enum name, section in arch-v1.md, precise issue, fix tier.]
[If none: "No flags — all Concern B checks passed."]
```

Apply fixes by tier.

**Targeted PO question:**

> "For each entity with a lifecycle — [list entity names] — does the full state list match what you will need to report on, filter by, or manually transition in the product? A state missing here becomes a story gap later."

Wait for PO response. Apply any changes raised.

**Update Section 11:**

Set Concern B Status → `Reviewed`. Update Flags count and Changes Applied.

> "Concern B complete. Moving to Concern C: API Contracts."

---

### C — API Contracts

**Frame this concern:**

Present the API in plain PO language.

> "**Concern C — What this system will expose:**
>
> The API has [N] endpoints across [N] modules. By module:
> [For each module in Section 5: module name — list the key user actions in plain English, e.g. 'create invoice, issue invoice, record payment' — not HTTP verbs]
>
> Every user journey defined in the BRD maps to one or more of these actions.
>
> Does the endpoint set cover every action that users and integrated systems need to perform?"

Wait for PO confirmation.

**Run Concern C checks:**

21. **orgId not in DTO inputs** *(Part 2: "No DTO includes `orgId` as a field")* — `orgId` does not appear as a writable input field in any DTO in Section 5. Flag any endpoint where `orgId` is in the request body, query string, or URL path segment.
22. **State transition patterns** — All state transitions in Sections 5 and 6.2 use POST with an action path segment (e.g. `POST /invoices/:id/issue`). Flag any transition described as PATCH directly on a status field.
23. **DTOs on every write endpoint** *(Part 2: "DTOs are provided for every POST and PATCH endpoint in Section 5")* — every POST and PATCH endpoint in Section 5 has a corresponding DTO definition. Flag any write endpoint with no DTO.

**Present FLAGS:**

```
FLAGS — Concern C: API Contracts
[Numbered list. Each flag: check number, endpoint, section in arch-v1.md, precise issue, fix tier.]
[If none: "No flags — all Concern C checks passed."]
```

Apply fixes by tier.

**Targeted PO question:**

> "Walk through the most important end-to-end journey from the BRD — the one a client would demo on day one. Is every step of that journey covered by an endpoint in Section 5, or is any step missing?"

Wait for PO response. Apply any changes raised.

**Update Section 11:**

Set Concern C Status → `Reviewed`. Update Flags count and Changes Applied.

> "Concern C complete. Moving to Concern D: Security & Access Control."

---

### D — Security & Access Control

**Frame this concern:**

Present the permission model and sensitive data handling in plain PO language.

> "**Concern D — Who can do what:**
>
> [For each role in Section 6.2 RBAC table: role name — one sentence describing what they can do and what they explicitly cannot do, in plain language]
>
> [If Section 6.2 also has a Capability Permissions table: "This project also uses [N] independently assignable capabilities — a user can hold any combination: [for each capability, one sentence on what it grants]."]
>
> No login required for: [list public endpoints from Section 6.1 — or 'None, all endpoints require authentication']
>
> [If Section 6.3 (Delegated Access) is populated: "Delegated access: [one-sentence summary of who can act on whose behalf, and how it's logged/indicated]." If "Not applicable," omit this line.]
>
> Sensitive data in this system: [list each field or category from Section 6.4 and how it is protected]
>
> Does the permission model reflect how users will actually work in the product? Is there anything a role or capability should be blocked from doing that is not restricted here?"

Wait for PO confirmation.

**Run Concern D checks:**

24. **Sensitive fields identified** — Section 6.4 names every column in Section 4.2 that contains PII (names, emails, phone numbers, addresses), financial data (account numbers, payment references, balances), credentials (tokens, keys, hashed passwords), or regulated data. Flag any sensitive column in the data model that is absent from Section 6.4.
25. **Composable capability coverage** *(Part 2: "If Section 6.1 declares a composable capability model, every capability listed there appears as a row in Section 6.2's Capability Permissions table")* — if Section 6.1 declares a composable model, every capability listed there appears as a row in Section 6.2's Capability Permissions table. Flag any capability named in 6.1 but missing from the table, or vice versa.
26. **Delegated access completeness** — if Section 6.3 is populated (not "Not applicable"), confirm it names an audit-logging requirement and a persistent UI indicator. Flag either if missing.
27. **Delegated Access section present** *(Part 2: "Section 6.3 (Delegated Access) is present — either describing the delegated access pattern or explicitly stating 'Not applicable'")* — Section 6.3 exists at all, distinct from check 26's review of its content. Flag if the subsection is missing entirely rather than stating "Not applicable."
28. **BRD role coverage in RBAC table** *(Part 2: "Every role from BRD Section 3 appears in the RBAC table in Section 6.2")* — every role defined in BRD Section 3 appears as a row in the Section 6.2 RBAC table. Flag any BRD role missing from the table.
29. **Public endpoints listed** *(Part 2: "Public endpoints are explicitly listed in Section 6.1 — no unlisted public endpoints")* — every endpoint that does not require authentication is explicitly listed in Section 6.1. Flag any endpoint in Section 5 that looks public (no auth column, or explicitly marked `[public]`) but isn't listed in 6.1, or vice versa.

**Present FLAGS:**

```
FLAGS — Concern D: Security & Access Control
[Numbered list. Each flag: check number, field or section, precise issue, fix tier.]
[If none: "No flags — all Concern D checks passed."]
```

Apply fixes by tier.

**Targeted PO question:**

> "Is there any action in the system that a role should NOT be able to perform but is not explicitly restricted in the permission table in Section 6.2? A gap here becomes a security defect that is expensive to fix after implementation."

Wait for PO response. Apply any changes raised.

**Update Section 11:**

Set Concern D Status → `Reviewed`. Update Flags count and Changes Applied.

> "Concern D complete. Moving to Concern E: Infrastructure & Compliance."

---

### E — Infrastructure & Compliance

**Frame this concern:**

Present the infrastructure posture and compliance status in plain PO language.

> "**Concern E — How this system will be deployed:**
>
> Environments: [N] — [list each environment name and its approval authority from Section 7.1]
>
> External services confirmed: [list each from Section 7.2 with its purpose — or 'None']
>
> Environment variables required: [N] backend / [N] frontend
>
> Deviations from project tech standard: [list from Section 9, or 'None — all choices match the project standard']
>
> Unresolved design decisions: [list from Section 8, or 'None — all decisions resolved']
>
> Does the infrastructure setup match what was agreed for this project?"

Wait for PO confirmation.

**Run Concern E checks:**

30. **Open decisions resolved** *(Part 2: "Section 8 (Open Decisions) is empty")* — Section 8 must be empty before the document can be locked. Any recorded open decision is a flag. For each open decision: present it to the PO and record their resolution in Section 8 before marking the flag resolved.
31. **Deviation declarations consistent** *(Part 2: "Every deviation from what's confirmed in `system-architecture.md` has a corresponding entry in Section 9"; "Section 9 (Deviation Declarations) explicitly states either no deviations or lists all deviations with named approval")* — every tech stack choice in Section 2 that differs from `system-architecture.md` appears in Section 9. Flag any undeclared deviation. If no project standards file exists, confirm Section 9 states this clearly.
32. **Approval section not pre-filled** — Section 10 (Approval) Decision and Date fields are blank. Flag if any field is pre-filled.
33. **Tech Stack table completeness** *(Part 2: "Tech Stack table covers every layer confirmed in `system-architecture.md`; rows not in scope have been removed")* — Section 2 includes a row for every layer actually confirmed in `system-architecture.md`, and no row for a layer not in scope for this phase (removed, not marked N/A). Flag any mismatch either direction.
34. **Environment variables referenced** *(Part 2: "Every environment variable in Section 7.3 is referenced somewhere in the document")* — every variable in Section 7.3 is referenced somewhere else in the document (Section 7.2, Section 4, or Section 5). Flag any variable listed with no apparent use.

**Present FLAGS:**

```
FLAGS — Concern E: Infrastructure & Compliance
[Numbered list. Each flag: check number, section in arch-v1.md, precise issue, fix tier.]
[If none: "No flags — all Concern E checks passed."]
```

Apply fixes by tier. For any open decision flagged in check 30: ask the PO for their resolution, write it into Section 8, then mark the flag resolved before closing Concern E.

**Targeted PO question:**

> "Are there any external services or infrastructure requirements discussed with the client that do not appear in Section 7? Any agreed integration omitted here will be missing from the epic stories and sprint plan."

Wait for PO response. Apply any changes raised.

**Update Section 11:**

Set Concern E Status → `Reviewed`. Update Flags count and Changes Applied.

> "Concern E complete. All concerns reviewed — proceeding to final resolution."

---

## Step 5 — After all concerns are reviewed

**Resolve open flags:**

Check the Open Flags table in Section 11. If any flags remain `Open`:

> "The following items were not resolved during the concern walkthrough:
> [List each open flag: concern, section, issue]
> These must be resolved before the architecture can be locked."

Work through each with the PO. Apply agreed changes. Update the Open Flags table — set each to `Resolved` as it closes.

**Add Change History row:**

If any fixes were applied during the review, add one row to the architecture document Change History table:

```
| 1.1 | {today's date} | Ascendra AI | Review fixes: [one-sentence summary of all changes applied across all concerns] |
```

**Lock confirmation:**

> "All five concerns reviewed. [N] flags raised, all resolved. [N] changes applied.
>
> Ready to lock the architecture? Reply `lock: {one-line summary of what you specifically verified}` (e.g. `lock: confirmed RBAC matches every BRD role and the schema covers every screen's data requirements`) to set Status to Locked and complete the review, or `not yet` to leave it in Under Review for further changes."

Wait for PO response. A bare `lock` with no rationale is accepted too — do not block on it — but ask once: "Anything specific you want on record as verified before I lock it?" and capture the answer if given, or proceed with `lock` alone if the PO declines.

---

**If PO types `lock` (with or without a rationale):**

Update the architecture document:
- `**Status:** Locked`
- `**Locked on:** {today's date}`

In Section 10 (Approval):
- `**Decision:** Approved`
- `**Date:** {today's date}`
- `**Locked because:** {the PO's rationale, verbatim, if given — otherwise omit this line entirely rather than writing a placeholder}`
- Leave the PO identity field blank — the PO fills it if required

Update Section 11:
- `**Review status:** Complete`
- Add `**Review completed:** {today's date}`
- Fill the Review Summary

```
ARCHITECTURE REVIEW COMPLETE — {PROJECT_CODE}
─────────────────────────────────────────
Status:        Locked
Document:      projects/{PROJECT_CODE}/architecture/arch-v1.md
Ref file:      projects/{PROJECT_CODE}/architecture/arch-v1-ref.md
Concerns:      5 reviewed
Flags raised:  [N total] — all resolved
Changes:       [N applied]
─────────────────────────────────────────
Next step:
Run /gen-stories starting from EPIC-001 (or the first epic
with no unmet dependencies). /implement-story reads arch-v1-ref.md
on every story — confirm it exists at the path above.
No architecture changes permitted without /assess-change.
```

---

**If PO types `not yet`:**

> "Architecture remains in Under Review status. Progress is saved in Section 11 — return to this review at any time. Run `/review-architecture {PROJECT_CODE}` to resume."
