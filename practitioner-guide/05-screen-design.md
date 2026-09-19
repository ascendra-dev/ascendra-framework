# Screen Design

Covers Stage 5 (Screen Design) in [`SDLC.md`](../SDLC.md): `/gen-screen-design`, `/review-screen-design`, and `/gen-ui-mocks` — base mode in full here, since that's the pass this stage owns; patch mode only far enough to be honest about why the two-pass split exists, since its invocation from `/gen-architecture` belongs to the future architecture chapter. Read [`00-orientation.md`](00-orientation.md) first if you haven't — this chapter is responsible for the full depth of its Portal/Screen/Surface/UI Pattern row, which the orientation table only summarizes in one line.

If you haven't yet moved an approved BRD through epic generation, that's the previous chapter's job — see [`04-product-structuring.md`](04-product-structuring.md) for epic boundaries and sizing. This chapter picks up once every epic is `Approved`, which is `/gen-screen-design`'s own gate.

---

## Purpose

Before this phase existed, `/implement-story` never read the architecture section that was supposed to define screen shape — confirmed by grep, zero references ([`FW-026`](../decisions/FW-026-screen-design-phase.md)'s "Why This Was Needed" #1). Mocks and real implementation each independently guessed what a screen looked like, so a PO-approved mock carried no actual weight over what got built. Screen Design exists to close that gap: every screen gets an explicit Surface (Page/Dialog/Sheet/Drawer) and UI Pattern (Form/Dashboard/Report/Table-List/Detail), each cited to a real, inspectable entry in `ascendra-ui`'s component catalog — never an invented shape — and the mock generated from that decision (Step 8 of `gen-screen-design.md`) is the same mock `/implement-story` is later held to.

That's what approving Screen Design actually commits you to: not just "these are the right screens," but "this is the real shape each one will be built in." Get a Surface or UI Pattern wrong here and you're not fixing a labeling error later — you're catching implementation building against a mock nobody actually checked.

---

## Verifying a catalog citation without ascendra-ui knowledge

Every Screen Inventory row's Matched Reference is supposed to name a real row in `ascendra-ui/docs/ui-reference.md` Part 2, or — for Report/Table-List, which have no composite example — a primitive composition. The mechanical check ("does this citation actually exist in a 1,500-line reference doc") is something only the AI can run. You, reviewing a markdown table, cannot independently verify that "Customer Profile" or "Create Product Listing (multi-section, mixed grid)" are real catalog entries rather than something invented to sound plausible.

**The test you can actually run: open the mock, not just the table.** `/review-screen-design`'s own Section 3 framing already tells you to do this — "Open the base mocks now alongside this table — do the navigation and screen shapes match what you'd expect a user to see?" Treat that mock as the real verification surface for this phase. You don't need to know what `ui-reference.md` says a "Customer Profile" pattern looks like; you need to know whether the rendered HTML looks like something a Finance Manager would actually be shown when creating a Payer record. That's a judgment any PO can make — it's the same skill you already use reviewing a live product, just applied to a static file.

**When a screen's shape looks unfamiliar or suspicious anyway:** ask the AI directly to name the exact catalog row or section it matched — not "what pattern did you use" (which invites a restated category label) but "show me the row in `ui-reference.md` this came from." A real match names a specific row under a specific classification column (Forms: `Domain | Complexity | Layout`; Dashboards: `Domain | Chart types | KPIs`; and so on). A vague answer — "a standard form pattern," "a typical dialog" — with no row to point to is the failure mode this whole phase exists to prevent, restated instead of caught.

### The `closest:` prefix — still undocumented, know what it means

Look closely at the Matched Reference column in [`screen-design.example.md`](../projects/TEMPLATE/screens/screen-design.example.md) and you'll find two rows prefixed `closest:` — SCR-002 (`closest: Customer Profile`) and SCR-004 (`closest: Create Product Listing (multi-section, mixed grid)`). Neither [`screen-design.template.md`](../projects/TEMPLATE/screens/screen-design.template.md)'s AI Guide nor its Correct/Incorrect example pair defines this convention anywhere — it is not a formally recognized escape hatch, it's a convention the canonical worked example uses without ever being told to.

**What it means when you see it:** the AI didn't find a catalog entry that's actually a right fit — it's citing the nearest real example instead of a genuine match. This is different from an invented shape (which has no real citation behind it at all); `closest:` still points at something real, just not a precise fit.

**How to react:** don't wave it through, and don't treat it as automatically wrong either — ask why no exact match exists. There are two honest answers, and they call for different responses:
- **This screen shape is genuinely novel** — the domain content doesn't resemble anything already in `ascendra-ui`'s catalog. That's a legitimate outcome; note it and move on, since a first-of-its-kind screen has nowhere better to point.
- **The AI didn't look hard enough** — it stopped at the first plausible-ish Forms table row instead of checking the primitive compositions, or searching `showcase-reference.md` for something closer. Push back and ask it to look again before accepting the citation.

Because this isn't a formally defined convention yet, treat every `closest:` you see as worth a one-line reason from the AI even though `/review-screen-design`'s checks don't currently force one — it's the cheapest way to catch the second case before it ships as an approved mock.

---

## Portal boundaries: when two personas share one, and when they don't

The base rule, straight from `screen-design.template.md` Section 1: two personas share a portal if they log into the same app and their navigation differs only by which items are visible. Different visibility, same shell — one portal.

**The sharper addition worth knowing before a real project hits it:** two personas can share 90% of the same nav and still need separate portals, if their authentication flows are genuinely different — one enters via SSO through the client's own identity provider, another via a magic link, a third via an ordinary standing session. Nav-item overlap doesn't settle the question; the login mechanism does. **Two personas sharing the same nav shell but entering via genuinely different auth flows are two portals, even if the nav items are otherwise identical.** A shared portal implies one session model underneath it, not just one visual shell.

**Harborview is the simple baseline case, not the hard one.** Its Staff App has exactly one portal shared by two personas — Finance Manager and Account Owner (`screen-design.example.md` Section 1) — both on a standing session, both entering the same way. There's no auth-flow split to force a division, which is exactly why it's a clean teaching example and not a stress test of the rule. A real project would need to split the moment one of those personas authenticates differently — say, if Account Owner access came through the client's own corporate SSO while Finance Manager used an ordinary email/password session issued by this system. At that point "same nav, different visibility" stops being the whole test, because the two sessions aren't interchangeable underneath the shared-looking UI. See [`case-studies/harder-cases.md`](case-studies/harder-cases.md) for the multi-portal RBAC scenario Harborview doesn't exercise.

---

## What's locked at approval, and what Architecture can still change

Not every section of `screen-design.md` carries the same weight once you approve it, and the template doesn't say this plainly enough on its own — so state it here directly.

**Sections 1–3 (Portals, Nav Map, Screen Inventory) are structurally locked.** The document-level AI Guide says `/gen-architecture` carries these into architecture Section 3.1.3.1–3.1.3.3 "as-is, not re-derived" — the same treatment `standards/system-architecture.md` gets. Once you approve Screen Design, these three sections are the answer; architecture doesn't get a second, independent pass at deciding what the portals or screens are.

**Section 4 (Per-Screen Data Requirements) is a forecast, not a lock.** The template only says it "feeds" architecture's Data Model and API Contracts generation — a weaker verb than "carried over," and deliberately so. Architecture does real data modeling work: normalizing entities, resolving relationships, deciding what's a foreign key versus a join table. It is completely normal for that work to surface a data shape that doesn't match what Section 4 predicted — a new entity implied by two screens that turn out to share more structure than Section 4 assumed, say.

**The distinction that actually matters to you as the approving PO:** if Architecture's real data model ends up different from Section 4's forecast, that's a normal refinement of an educated guess — not a regression against something you locked, and it does not require running `/assess-change`. `/assess-change` exists for changes to things that were actually decided and approved; Section 4 was never that. Don't let an architecture-stage data model shift feel like your Screen Design approval is being violated — it isn't. What *would* require `/assess-change` is a change to Sections 1–3 after approval — a new screen, a portal split, a different Surface — because those are the parts you actually locked.

---

## The two-pass mock: why base and patch are separate

`/gen-ui-mocks` runs twice against the same file, in two different modes, invoked by two different upstream commands — never run standalone by the PO. This isn't an implementation quirk; it follows directly from what each pass actually needs to exist first. The mechanics of both passes, and the fixed skeleton every mock follows, are laid out in [`ui-mock.template.md`](../projects/TEMPLATE/mocks/ui-mock.template.md); [`ui-mock.example.md`](../projects/TEMPLATE/mocks/ui-mock.example.md) shows Harborview's own patched Staff App mock, gating caption and all.

**Base mode**, invoked by `/gen-screen-design` right after `screen-design.md` is written, generates full layout, navigation, and every screen's shape with real component fidelity — no gating captions, no cross-cutting banners. It can't add those yet for a real reason: Action Gating and Cross-Cutting UI Rules (architecture's 3.1.3.4–3.1.3.5) both depend on architecture Section 6, the security model — and at Screen Design time, that model genuinely does not exist. There's nothing to gate against yet, so base mode doesn't pretend to.

Base mode has to run — and be approved, inside `/review-screen-design` — *before* Architecture starts, for the same reason Section 4's Data Requirements exist at all: architecture's Data Model and API Contracts generation needs real, PO-reviewed screen data needs to design against, rather than inferring them a second time from BRD requirements alone, disconnected from what a screen actually renders (`FW-026`'s "Why This Was Needed" #3). The mock is what makes those data needs concrete enough to review before they become an API surface.

**Patch mode**, invoked by `/gen-architecture` once Section 6 is written, layers in Action Gating captions and Cross-Cutting UI Rules banners onto the existing files — additive only, never touching layout or content already reviewed in base mode. It's reviewed inside `/review-architecture` Concern A, not as its own gate. The full mechanics of that invocation belong to the architecture chapter; the point worth knowing here is just that the split exists because the security model the patch needs genuinely doesn't exist until then — not because of an arbitrary process boundary.

**One thing worth calling out as a real strength, not a caveat:** `/gen-ui-mocks` now reads [`hard-instructions.md`](../reference/ascendra-ui/hard-instructions.md) in *both* passes (`gen-ui-mocks.md` Step 2, base mode item 7 and patch mode item 3) — the same living registry of corrections `/implement-story` has always consulted, covering cases where `ascendra-ui`'s own docs describe a prop or pattern that doesn't match the real component source (e.g. AUI-004: `DialogContent`'s documented `showCloseButton` prop doesn't actually exist on the real component — a genuine TypeScript build error when used). Before this was fixed, a mock could be built from a documented-but-fictional pattern, get approved by the PO here, and then not match what implementation later correctly built against the real source — silently defeating the reason this phase exists at all. Now the mock you approve is built from the same corrected source `/implement-story` is held to, so an approved mock and the real implementation shouldn't diverge on anything `hard-instructions.md` already knows about.

---

## Portal, Screen, Surface, UI Pattern — the full distinction

`00-orientation.md`'s terminology table gives you the one-line version. This chapter owns the full depth, because the audit found the key equivalence here is never stated as an explicit rule anywhere in the framework's own files — so state it plainly:

- **Portal** — a navigation shell: one app, one session model. Harborview has one (Staff App).
- **Screen** — one row in Section 3's Screen Inventory. **Every row is a Screen, with equal weight, regardless of Surface.** An overlay Dialog or Sheet is exactly as much "a screen" as a routed Page — it is not a lesser or secondary thing hanging off a "real" screen. If you're counting how many screens a project has, count Section 3 rows, not routes: Harborview has five screens (SCR-001 through SCR-005), not three pages plus two afterthoughts.
- **Surface** — the screen's container mechanism: Page, Dialog, Sheet, or Drawer. This is *how* the screen is presented, not what it contains.
- **UI Pattern** — the screen's internal content shape: Form, Dashboard, Report, Table-List, or Detail. This is independent of Surface — a Form can live on a routed Page (Harborview's SCR-004, Form (Complex) on a Page) or inside an overlay Sheet (SCR-002, Form on a Sheet). Knowing a screen's UI Pattern tells you nothing about its Surface, and vice versa — both fields are mandatory on every row precisely because neither implies the other.

Keep these four cleanly separated and the rest of this phase reads correctly: a "screen" isn't a synonym for "page," a Surface choice isn't a UI Pattern choice, and nothing in Section 3 is structurally more important than anything else in it just because it happens to render as a full-width page instead of an overlay.

---

## Tabs: a criterion to apply carefully, not one to copy

The template's criterion for proposing Tabs is specific: only when the source journey or FR describes **2+ independent sub-views reachable from one entry point with no route change** — and even then, Tabs is layered onto whatever Surface/UI Pattern was already chosen, never a default choice on its own.

**Be honest with yourself about what Harborview's worked example can and can't teach you here: it doesn't demonstrate a real Tabs usage at all.** `screen-design.example.md`'s own Part 2 Verification checklist says so directly — "Tabs are proposed only where the source genuinely describes 2+ independent sub-views — not used in this example, no screen needed them." There is no row anywhere in the example with Tabs applied, so there's nothing to pattern-match from when you hit a real screen that might need it.

That means when Tabs comes up on your own project, you have to apply the template's criterion directly against your own journey text — "does this journey step genuinely describe two or more independent sub-views under one entry point, with no route change" — rather than reaching for "what did Harborview do here," because Harborview never had to make this call. Treat Tabs as a rule you test on the actual language in front of you, not a shape you can copy from the canonical example.

---

## Terminology recap

This chapter builds on [`00-orientation.md`](00-orientation.md)'s Portal/Screen/Surface/UI Pattern row — go there for the one-line version; the full depth is above. Two things specific to this phase, not covered in the orientation table:

- **Every Section 3 row is "a Screen" with equal weight, regardless of Surface** — an overlay Dialog or Sheet carries the same status as a routed Page. This chapter is where that rule is stated explicitly, because nothing else in the framework's own files does.
- **The `closest:` Matched Reference prefix** — an approximate catalog match, not an exact one, and still not formally defined anywhere in `screen-design.template.md`. Treat it as a prompt to ask why no exact match exists, not as a silent downgrade to wave through.

---

## Common mistakes

- Reviewing only the Screen Inventory table and never opening the generated mock — the table alone can't tell you whether a citation is real; the mock is the surface you can actually judge.
- Accepting a vague answer ("a standard form pattern") when you ask the AI what a screen matched to, instead of insisting on the specific catalog row.
- Waving through a `closest:` citation without asking whether the screen is genuinely novel or whether the AI just didn't look hard enough.
- Treating "same nav items" as sufficient to merge two personas into one portal, without checking whether their authentication flows are actually the same.
- Panicking when Architecture's real data model doesn't match Screen Design's Section 4 forecast — that's a normal refinement, not grounds for `/assess-change`.
- Treating a Dialog or Sheet as a lesser, secondary "sub-screen" instead of counting it as a full Screen Inventory row like any Page.
- Proposing Tabs because a screen "feels like" it should have them, instead of checking the source journey/FR against the actual 2+-independent-sub-views criterion.
- Forgetting that base-mode mocks intentionally have no gating captions yet — that's not a missing feature, it's because Section 6 doesn't exist at Screen Design time.

---

## Harborview in practice

`screen-design.example.md` is the worked model for this whole chapter — one portal (Staff App), five screens, deliberately the easy case. Specifically worth studying directly:

- **SCR-001 and SCR-003** (`Page`, `Table-List`, `Data Table + Empty State` / `Data Table + Page Bar`) — both cleanly matched to a primitive composition, since Table-List has no Part 2 composite example, exactly per the rule above.
- **SCR-002** (`Sheet`, `Form`, `closest: Customer Profile`) — the clearest example of the `closest:` prefix in the wild: an overlay Form for creating or editing a Payer, matched to the nearest real pattern rather than an exact one. Use this row to practice the "why no exact match" question on a real instance.
- **SCR-004** (`Page`, `Form (Complex)`, `closest: Create Product Listing (multi-section, mixed grid)`) — the same `closest:` pattern on a routed Page rather than an overlay, showing that the prefix isn't tied to any one Surface.
- **SCR-005** (`Page`, `Detail`, `Item + Card`) — an invoice detail screen citing a primitive pair rather than a named composite, and the one row whose Notes column shows a journey citation spanning two separate BRD journeys (4.1 and 4.2) for the same screen.

The document's own header note is worth reading directly too: it explicitly calls out that journey citations here reference a specific numbered step within a BRD journey (e.g. "Journey 4.1, step 1"), never an invented sub-heading like "4.1.1" — BRD journeys number their steps as a plain list, not as sub-headings, and this document only cites what's really there.
