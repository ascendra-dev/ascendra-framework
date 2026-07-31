# FW-032 — Screen Design Coverage in `/assess-change`

| Field | Value |
|-------|-------|
| **ID** | FW-032 |
| **Date** | 2026-07-16 |
| **Status** | Decided — implemented |
| **Area** | `.claude/commands/assess-change.md` |
| **Amends** | FW-026 — Screen Design Phase |

---

## Decision

`/assess-change` now recognizes `screens/screen-design.md`, its mock files, and architecture Section 3.1.3.1–3.1.3.3 as artifacts it can be asked to change. A missing or incorrect screen for an already-approved story/journey classifies as Pattern 2 (Gap, Informal) — the same tier as a missing story or missing AC. Because architecture Section 3.1.3.1–3.1.3.3 carries Screen Design's content over as-is (`FW-026`), any accepted change to `screen-design.md` triggers a new **Screen Design sync rule**: if architecture exists, Section 3.1.3 is updated to match in the same session, with a Change History row recording the amendment — without reopening the full `/review-architecture` gate or moving the architecture document's `Status` off `Locked`. `screen-design.md` itself stays `Approved`; a Document Control row is sufficient for an Informal gap-fill, mirroring how Pattern 2 already treats epic Document Control.

---

## The Problem That Triggered This Decision

While running `/gen-story-plan` for `US-02-001` (Sign up as a new Issuer, `ASCENDRA-PAY-001`), a Web Plan couldn't be built: no signup screen exists anywhere — not in `screen-design.md`'s Screen Inventory, not in architecture's carried-over Section 3.1.3.3, and no mock covers it. All three defined portals (Issuer Staff App, Payer Payment Link, Ascendra Reviewer Console) assume an Issuer already exists.

This is a Pattern 2 gap by every definition already in `assess-change.md` — `REQ-003/038/064` are approved, `EPIC-002` is approved, `US-02-001` is already `Reviewed` — nothing about scope or requirements is changing, a screen was simply never generated. But `assess-change.md` had no path for it: Screen Design does not appear anywhere in its Phase A/B reading lists, its four Pattern definitions, its blast-radius table's file-type examples, or its Document Control templates. The command was written entirely around the BRD → Epics → Stories cascade, before `FW-026` introduced Screen Design as an independently-gated artifact with a direct, as-is carryover into architecture.

`review-screen-design.md` already states the correct entry point for changing an `Approved` screen-design.md — *"run `/assess-change` and describe the change needed"* — but the destination command had nothing built to receive that call. Re-running `/gen-screen-design` instead would regenerate all screens from scratch, discard the existing `Approved` document, reset it to `Draft`, and force a full 4-section re-review for a one-screen gap — wildly disproportionate for an Informal-tier fix, and not what `FW-026` intended `/assess-change`'s redirect to mean.

---

## What Changed in `assess-change.md`

- **Phase B reading list:** added a case for screen/UI-level gaps — read `screen-design.md` in full, the affected mock(s), and architecture Section 3.1.3 if architecture exists.
- **Pattern 2 (Gap) definition:** added "missing screen in `screen-design.md` for a story/journey already in approved scope" as an example, with its own formality bullet (no version bump, no re-approval, mock regenerated/patched, Screen Design sync rule applied if architecture exists).
- **Step 4 blast radius table:** added `screen-design.md`, the mock file, and `arch-v1.md` as example rows.
- **Step 6 order of application:** inserted Screen Design between BRD and Architecture files (position 2 of 7), matching its real place in the lifecycle (`FW-026`: Epic Approval → Screen Design → Architecture).
- **New Screen Design sync rule** (alongside the existing Architecture file sync rule): defines exactly how `screen-design.md` changes propagate into a `Locked` (or any-status) architecture document without reopening its full review/lock gate.
- **New Document Control template** for the Pattern 2 screen-added case, matching the existing per-pattern templates.
- **New post-change verification check:** confirms `arch-v1.md` Section 3.1.3 has no drift from `screen-design.md` after any change touching either.

---

## Why This Was Needed (and why now, not deferred)

This gap would recur on every future project the moment `/gen-screen-design` misses a screen after Screen Design is already `Approved` — a plausible, ordinary occurrence, not an edge case. Without this fix, `/assess-change` would either silently under-scope the blast radius (leaving `screen-design.md` and `screen-design.md`'s architecture carryover out of sync with no warning) or the operator would have to rediscover and improvise the same reasoning each time, undocumented, with no guarantee of consistency across sessions or projects.

The fix is scoped narrowly to the demonstrated gap — it does not attempt to model every possible Screen Design change under Pattern 3/4 (e.g. an epic scope shift that removes a screen), since no concrete case has surfaced yet requiring that. Extend further only when a real scenario demands it, consistent with this framework's own practice of fixing what broke rather than speculatively covering every theoretical case.

---

## Implementation Status

Applied immediately during this session — `.claude/commands/assess-change.md` edited directly; no separate tracking needed.
