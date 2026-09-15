# Practitioner's Guide

The Ascendra Framework's own docs — [`CLAUDE.md`](../CLAUDE.md), [`SDLC.md`](../SDLC.md), [`PROJECT-LIFECYCLE.md`](../PROJECT-LIFECYCLE.md) — tell you **what** happens at each stage and **which command** runs it. This guide answers a different question: **what do you actually write, how much, and what breaks three phases downstream if you get a judgment call wrong.** That gap — real, not imagined — was found by a full read-through of every command, template, and example file in the framework, preserved in [`FRAMEWORK-AUDIT.md`](../FRAMEWORK-AUDIT.md) at the repo root. This guide is what came out of closing it.

**Start here:** [`00-orientation.md`](00-orientation.md) — how this guide relates to the rest of the framework's docs, the running example every chapter leans on, and the terminology map that resolves the most common source of confusion in the whole framework (Scope vs. Priority vs. Layer vs. Core/Extension architecture — four different things that sound related and aren't).

## Status

This guide is being built phase by phase, checkpointed against real feedback rather than delivered all at once. Chapters below are either **live** or **planned** — planned chapters aren't linked yet, so you won't hit a dead link.

| # | Chapter | Covers | Status |
|---|---------|--------|--------|
| — | [`00-orientation.md`](00-orientation.md) | How to use this guide; the terminology map | ✅ Live |
| 1 | [`01-intake.md`](01-intake.md) | Stage 1 — `/init-project`, `/run-intake`, the brief | ✅ Live |
| 2 | [`02-domain-discovery.md`](02-domain-discovery.md) | Stage 2 (domain half) — `/gen-domain-knowledge` and the optional discovery sub-flow | ✅ Live |
| 3 | [`03-brd-discovery-and-scope-lock.md`](03-brd-discovery-and-scope-lock.md) | Stage 2 (BRD half) + Stage 3 — BRD discovery, scope lock | ✅ Live |
| 4 | [`04-product-structuring-epics.md`](04-product-structuring-epics.md) | Stage 4 — `/gen-epics`, epic boundaries and sizing | ✅ Live |
| 5 | [`05-screen-design.md`](05-screen-design.md) | Stage 5 — `/gen-screen-design`, `/gen-ui-mocks` | ✅ Live |
| 6 | `06-architecture.md` | Stage 6 — `/gen-architecture`, the 35-check review | ⏳ Planned |
| 7 | `07-sprint-planning.md` | Stage 7 — `/gen-stories`, wave vs. sprint, sizing | ⏳ Planned |
| 8 | `08-development.md` | Stage 8 — `/gen-story-plan`, `/implement-story`, `/verify-story` | ⏳ Planned |
| 9 | `09-qa-uat.md` | Stage 9–10 — QA, UAT | ⏳ Planned |
| 10 | `10-release-support-hotfix.md` | Stage 11–12 — release, support, hotfix | ⏳ Planned |
| 11 | `11-cross-cutting.md` | `/assess-change`, `/update-status`, `/project-status` | ⏳ Planned |

## Case studies

- [`case-studies/harder-cases.md`](case-studies/harder-cases.md) — the judgment calls the framework's own running example (Harborview) never exercises: a regulated/niche domain, an extension project, multi-portal RBAC. Built out alongside the phase chapters — check back as later phases add a complex architecture review and other harder scenarios.

## How each chapter is structured

Every phase chapter follows the same shape, so once you know one you know them all: **purpose** (why this stage exists) → **decision heuristics** (a real test for every fork the audit found under-specified, not vague advice) → **information-density guidance** (how much to write, with a good/bad worked example) → **downstream impact** (what breaks later if you get it wrong) → **terminology recap** (links back to the orientation map rather than repeating it) → **common mistakes** → **Harborview in practice** (a short pointer into the real worked example, not a duplicate of it).
