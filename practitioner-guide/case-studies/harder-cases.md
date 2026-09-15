# Harder Cases

Harborview — the project threaded through every `.example.md` file in the framework, and through most of this guide — is deliberately the easy path: one client, one well-understood domain, one portal, Core layer only, a discovery session with no contradictions. That's exactly why it can't teach you the judgment calls that actually cause trouble on a real engagement. This file holds worked vignettes for the cases Harborview doesn't cover, added alongside the guide phase by phase. It's not a second full project traced end to end — it's targeted, just deep enough to show the specific judgment call in action.

Each case names which phase-doc it supports, so you can jump to it from there rather than reading this file start to end.

---

## Case 1 — A niche, regulated domain: cross-border remittance compliance

**Supports:** [`02-domain-discovery.md`](../02-domain-discovery.md) — applying the niche-domain self-test and the raised confidence bar for real.

Imagine a brief for a system that lets a licensed money-transfer operator send remittances from the UK to recipients in Pakistan, the Philippines, and Nigeria — the kind of domain an AI has partial, plausible-sounding training knowledge of, which is exactly what makes it dangerous to wave through.

**Applying the self-test from `02-domain-discovery.md`:**

1. *Can you name the 3 most consequential business rules in this domain from memory, with confidence?* You might guess "large transactions need extra checks" — but you can't state the actual threshold, which regulator sets it, or whether it differs by corridor. Low confidence.
2. *Would a wrong assumption here cost more than a sprint to unwind?* Yes — a wrongly-modeled compliance threshold isn't a UI bug, it's a regulatory exposure that could require re-architecting the transaction-approval flow after the fact.
3. *Is there a regulator or legal requirement that would reject wrong output?* Yes, several, and they don't agree with each other across corridors.

Three "run the sub-flow" answers out of three. This is not a borderline call the way Harborview's invoicing domain or a retail loyalty program might be — it's a clean trigger.

**Where the Section 5 vs. Section 10 misclassification risk gets concrete:** a plausible-sounding "fact" here is *"transactions over $3,000 require enhanced due diligence."* Written into Section 5 (Universal Business Rules) as stated, this becomes binding and — per `02-domain-discovery.md` — **stops generating BRD discovery questions entirely**, because only Section 10 topics do that. But the rule isn't universal: it's the FinCEN threshold for US-based money service businesses; the UK's threshold under the Money Laundering Regulations is different, and Nigeria's CBN threshold is different again. This is precisely the shape of fact that belongs in Section 10 (Known Variations) — "Topic: enhanced due diligence threshold, Common Options: [varies by corridor/regulator], Must Ask: which corridors are in scope and what's each one's actual threshold" — never Section 5. Get this wrong and the framework won't ask the client which threshold applies in any corridor; it'll just build against whichever number the AI guessed.

**What the raised 90% bar actually looks like in practice for this domain:** don't just watch the number. Before treating discovery as sufficient:
- Read Section 9 (Regulatory Baseline) line by line and confirm every named regulation is cited to something specific (a real Act, a real regulator's named threshold) — "AML regulations apply" is not sufficient; "UK MLR 2017 Regulation 33 enhanced due diligence" is.
- Read Section 12.3 (Factual Accuracy Log) and treat every row still marked Unverified as a hard blocker, not a note — for a domain like this, an unverified compliance fact reaching the BRD is the exact failure mode `02-domain-discovery.md` warns about.
- Confirm Section 10's Known Variations table has a row for every corridor genuinely in scope, each with its own confirmed threshold — not one row with "varies" as the value.

Nothing in the framework enforces any of this beyond a bare 70% gate. On a domain like this, treating that gate as sufficient is how a wrong compliance assumption ends up in a locked architecture.

---

## Case 2 — An extension project: Ascendra Pay's Pakistan extension

**Supports:** [`01-intake.md`](../01-intake.md) (Standalone vs. Extension, project codes) and [`03-brd-discovery-and-scope-lock.md`](../03-brd-discovery-and-scope-lock.md) (Layer, in full working contact with the Core/Extension architecture methodology).

This reuses the framework's own existing worked example — `reference/architecture/layered-domain-architecture.example.md` already models **Ascendra Pay** with two extension dimensions (Market/Country: Pakistan; Sector: School) — rather than inventing an unrelated project. Read that file alongside this one; this case walks through the Intake and BRD-level decisions that would produce the inputs that file's architecture-level example assumes already exist.

**The project family, and how it's actually expressed:**

- `ASCENDRA-PAY-001` (or `ASCENDRA-PAY-CORE-001`) — the base platform. `Project Type: Standalone`. Its BRD Section 1.6 (Designed for Extension) names the two dimensions: *Market — Pakistan* (FBR e-invoicing, IRIS submission) and *Sector — School* (academic-year rollover, fee-blocking). Naming these doesn't build them — per the brief and BRD templates' own guidance, it signals to the architecture phase where to leave seams, without designing the extension itself yet.
- `ASCENDRA-PAY-PKN-001` — the Pakistan extension, run as its own project with its own brief. `Project Type: Extension`, `Extends: ASCENDRA-PAY-001` (or `-CORE-001`), `Extension Adds: FBR e-invoicing compliance, IRIS integration` — one free-text sentence, not a structured category (per `FW-016`; see `01-intake.md`'s project-code section for why the code's optional `-PKN-` segment is cosmetic and this brief field is what actually drives every downstream command's chain resolution).

**What Layer looks like in this BRD, in contrast to Harborview's BRD where every requirement is `Core`:** the Pakistan extension project's own BRD will have requirements like *"the system must capture and validate a 7-digit NTN for every Issuer registered under this deployment"* — tagged `Ext:PK`, not `Core`, because the membership test fails it cleanly: *"would a UK retailer using this system need this requirement?"* No — NTN only exists because the issuer is Pakistani. Compare a requirement like *"the system must generate an invoice PDF"* — that one's `Core` regardless of which project's BRD it's written in, because every deployment needs it.

**Where Layer and the Core/Extension architecture methodology meet, and where they stay separate:** Layer is a label on *this project's own* BRD requirements — a bookkeeping device telling `/gen-stories` and sprint planning which requirements can be built now versus which wait on a seam. The Core/Extension methodology (`layered-domain-architecture.md`) is what the *base* project's architecture document actually builds to make that seam exist — the `issuer_pakistan_compliance` table shown in the reference example, the strategy-pattern Payment Adapter that lets Pakistan's Safepay integration plug in without Core ever importing anything Pakistan-specific. A PO working the Pakistan extension project doesn't design that seam — `ASCENDRA-PAY-001`'s own architecture document already did, in its Section 3.2 (Extension Points), before this project ever started. The extension project's own architecture document just implements *against* that seam. If you're the PO on the extension project and you find yourself needing to invent a new column on a Core table rather than working through an existing extension point, that's a signal to go back to the base project's architecture — not to route around it.

**What this means for a PO's actual workflow on the extension project:** when `/gen-architecture` runs for `ASCENDRA-PAY-PKN-001`, it reads the parent project's architecture document specifically for its Extension Points section (per `FW-016`'s loading table) — you're not starting from a blank page, and you're not supposed to be able to freely redesign Core. If a PO on the extension project finds themselves arguing for a requirement that would require changing something in Core, that requirement doesn't actually belong to this project — it belongs to a change request against the base project, assessed through `/assess-change` there, not smuggled in through the extension's own BRD.

---

## Case 3 — Multi-portal RBAC: Ascendra Pay's School extension

**Supports:** [`05-screen-design.md`](../05-screen-design.md) (portal boundaries, in a case where the simple rule genuinely isn't enough) and [`04-product-structuring-epics.md`](../04-product-structuring-epics.md) (dependency ordering across portals).

Same project family as Case 2, but now the Sector extension: `ASCENDRA-PAY-SCH-001`, `Extends: ASCENDRA-PAY-001`, adding the School domain's `Student`, `AcademicYear`, and `FamilyAccount` entities (per `layered-domain-architecture.example.md`). Where Harborview's Screen Design is genuinely simple — one portal, two personas, sharing a standing session — this project has to actually work through the boundary test that Harborview never stresses.

**The personas:** a School Admin (manages the school's account, staff, fee structures — logs in via the school's own SSO, Google Workspace), a Bursar/Finance role (creates and sends fee invoices — same SSO, same app, different permissions), and a Parent (views and pays fees for their own children only — logs in via a standalone magic-link flow, has never heard of and will never touch the school's Google Workspace).

**Applying `05-screen-design.md`'s portal boundary test:** the base rule — "same app, navigation differs only by visibility" — already fails immediately for School Admin/Bursar vs. Parent, before you even get to nav items: they don't share an authentication flow at all. SSO vs. magic-link is exactly the sharpened case the screen-design chapter calls out. Two portals, full stop — a Staff App (Admin + Bursar sharing one portal, since *they* do share both the app and an auth flow, differing only by what's visible) and a Family Portal (Parent). This is a genuinely three-persona, two-portal structure, and it's the two-persona-one-portal grouping (Admin + Bursar) plus the auth-flow split (Staff vs. Parent) working together that Harborview's simpler two-persona-one-portal case never has to demonstrate.

**Where the RBAC complexity actually bites, past the portal boundary:** a School Admin sees every student's fee record. A Parent must see *only* their own children's — and a family with two children in the same school, or two parents (separated, both with legitimate access to the same student's fees) is the normal case, not an edge case. This isn't a Screen Design–level decision — Screen Design's Section 3 (Screen Inventory) records that a "Family Fee Dashboard" screen exists and what data it plausibly shows; it does not decide *how* that screen's query gets scoped to the logged-in parent's own family. That's an Architecture-phase decision (the future architecture chapter covers this in depth — the shape is a per-row data-scoping rule tied to `FamilyAccount`, plus a Delegated Access concept for the two-parents-one-student case, both properties of Section 6 of the architecture document, not of Screen Design). What Screen Design *is* responsible for getting right here: making sure the Family Portal's screens are modeled as scoped-by-default in their Per-Screen Data Requirements (Section 4) — "this Parent's children's fees," not "fees" — so the scoping requirement is visible on paper before Architecture has to invent it from scratch.

**The practical takeaway:** the moment a project has more than one persona who might plausibly share a portal, don't reach for the base rule alone — check auth flow explicitly, even if nav items look like they'd overlap. And the moment a portal serves users who should each see only their own slice of the data (not just their own *permissions*, their own *data*), flag that in Screen Design's Data Requirements even though the actual scoping mechanism isn't yours to design yet — it's the difference between Architecture discovering the requirement on its own and Architecture confirming a requirement you already surfaced.
