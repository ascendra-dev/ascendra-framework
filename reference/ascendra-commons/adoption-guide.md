# Ascendra Commons — Adoption Guide

This is the framework's own annotation on `ascendra-commons` — the same relationship `reference/ascendra-ui/` has to `ascendra-ui`. It does not duplicate `ascendra-commons`'s own module catalog, `MODULE-STANDARD.md`, or any module's own `README.md` — those are read directly from the real repo when a module is actually being adopted. This file states three things the framework itself needs: the gate for whether `ascendra-commons` applies to a project at all, the two-part test for whether any individual module should be adopted, and the one rule that is never optional once a module is adopted.

`ascendra-commons` is not a framework dependency. Most projects — anything not on the NestJS/Drizzle backend stack — will never touch this file. `/gen-architecture` and `/implement-story` only consult it after the stack gate below is checked and passes.

---

## The stack gate

`ascendra-commons` is explicitly **not technology-agnostic** — its own root `README.md` states it's scoped to NestJS + Drizzle + Supabase/Postgres, consumed by Next.js frontends via `ascendra-ui`. The framework itself has no mandated stack (`FW-025`) — NestJS + Next.js is the common default and fallback, never a universal assumption. Those two facts combine into one hard rule:

> **`ascendra-commons` applies only when the confirmed backend stack (architecture Section 2) is NestJS + Drizzle, with Postgres or Supabase as the database. For any other confirmed stack — a different Node framework, Java, Python, anything else — `ascendra-commons` does not apply, full stop.**

This is a fail-closed gate, not a "translate the concepts" fallback. A module's value is in its shipped, tested NestJS DI wiring and its Drizzle schema — the pattern's *idea* (a domain-event emitter, a permission-table authorizer) may well generalize to another stack, but nothing in `ascendra-commons` today does, and this framework should never present an untested translation as if it were the vendored module.

Where this is checked: `/gen-architecture`, once Section 2 (Tech Stack) is confirmed — before any module is proposed. `/implement-story` never re-checks it independently; it trusts what the architecture document already recorded.

---

## The adoption test — per module, never as a package deal

Passing the stack gate makes `ascendra-commons` *reachable*, not adopted. Each of the fifteen modules is its own decision, and the test is two-part:

1. **Does this project's own BRD or domain knowledge document actually establish a need for this capability?** Not "might be useful" — a real requirement, a real domain rule, a real integration already named in the approved BRD. `ascendra-commons`'s own `MODULE-STANDARD.md` holds every module inside the library to the identical discipline: nothing gets built or split "eventually" or "when we scale," only against a named, concrete signal. Adopting a module speculatively here would violate the same principle the library itself was built on.
2. **Is the module's stated Model 1 sufficient for what this project needs, or does the project's own scale/requirements demand its Model 2?** Read the module's own README — every one states, explicitly, what Model 1 ships and the concrete trigger for Model 2. This is a real design decision per adopted module, not a checkbox.

Three real examples already sitting in the framework's own worked example (Harborview) that would pass the first half of this test, if Harborview's stack matched (it doesn't — that's exactly why `case-studies/harder-cases.md` is the right place for a worked commons-adoption case, not the Harborview example itself, which stays stack-neutral by design):

- Dunning/overdue reminder emails → `notification`
- The Stripe webhook integration → `payment-gateway-adapter`'s generic webhook mechanics (the concrete Stripe-specific handling stays project-owned per that module's own README)
- The `version`-column optimistic-concurrency contract `FW-045` already requires on every mutating endpoint → `optimistic-locking`'s shared atomic-update helper, rather than re-deriving the same mechanism per project

A module fails the test as often as it passes — most projects on the matching stack will still adopt only a handful of the fifteen, not all of them. Declining a module is a normal, correct outcome, not a gap to fill in later.

---

## The one rule that is never optional: `.core`-only imports

Every `ascendra-commons` module's public interface is its `.core` package (or, for a single-folder module with no split, the module itself) — the contract: interfaces, tokens, `forRoot()` wiring. Everything else — `.core.<capability>`, `.core.<family>.<stack>`, `.api` — is an implementation the vertical never touches directly. This is stated in every module's own README (see `domain-events/README.md`'s "Consuming this module" section for the canonical phrasing: *"any other module that depends on domain events depends on `domain-events.core` only"*) and it is the mechanism that makes a future Model 1 → Model 2 swap possible without touching business logic at all.

**This rule applies to project-owned code exactly as it applies to another vendored module depending on this one.** A story's own service, controller, or handler imports a commons module's `.core` package — never reaches past it into a concrete stack adapter, never imports from `.core.<family>.<stack>` or `.core.<capability>` directly, regardless of how convenient a shortcut might look. `/implement-story` treats this the same weight it gives any other hard, non-negotiable rule — the same class of thing `reference/ascendra-ui/hard-instructions.md` is for the frontend.

---

## What gets recorded, and where

An adopted module is documented the same way any other project-specific architectural pattern is: a `standards/{module}-pattern.md` file following `projects/TEMPLATE/standards/architectural-pattern.template.md`, generated by `/gen-architecture`, citing the module's own README rather than re-deriving its content. This is the existing Tier 3 standards mechanism — already used, for example, by Harborview's own `idempotent-stripe-webhook-handling.md` — extended to cover a vendored pattern instead of a project-invented one.

The generated standards file records which `ascendra-commons` git commit or date the module's README was read at, the same freezing discipline `arch-v1-ref.md` already applies to the architecture document itself. `ascendra-commons` has no release/versioning tooling yet — its own root `README.md` says so — so without this, a later change to the source repo could make an already-Locked project's citation silently stale. Freezing the reference is what prevents that.
