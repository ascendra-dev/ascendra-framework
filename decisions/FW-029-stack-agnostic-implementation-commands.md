# FW-029 — Stack-Agnostic Implementation Commands

| Field | Value |
|-------|-------|
| **ID** | FW-029 |
| **Date** | 2026-07-13 |
| **Status** | Decided |
| **Area** | `implement-story.md`, `verify-story.md`, and any future command that acts on already-confirmed architecture |
| **Extends** | `FW-025` (On-the-Fly Architecture and Standards Generation) |

---

## Decision

`FW-025` already established that architecture *generation* is stack-agnostic — no fixed "Ascendra standard stack," the tech stack is recommended per project and recorded in `system-architecture.md` / arch-v1.md Section 2. This decision extends the same principle to commands that *consume* an already-confirmed architecture: `/implement-story`, `/verify-story`, and any future command in that category must read the confirmed stack and act on it dynamically — never hardcode a specific framework's syntax, tooling, or commands as if it were universal.

**Concretely:** NestJS + Next.js/Ascendra UI may appear as a fully-worked reference path in these commands, since it's the framework's common default and this project's actual confirmed stack — but every stack-specific instruction must be explicitly conditioned on "if X is confirmed," with a stated fallback: apply the same underlying principle using whatever framework Section 3.1/5/6 of that project's *own* arch-v1.md actually documents.

This is a hard requirement, not a style preference: a specialist command per architecture type (`implement-story-java.md`, `implement-story-angular.md`, etc.) is explicitly rejected as the alternative. That would be exactly the "predefined, curated library" `FW-025` already rejects for architecture generation, and would duplicate the same maintenance burden the framework avoids everywhere else (one `/gen-epics` regardless of domain, one `/gen-architecture` regardless of stack).

---

## Rationale

Found 2026-07-13 during a critical review of `implement-story.md` (never yet run against a real story): the command hardcoded `nest new`, npm commands, and NestJS decorators (`@UseGuards`, `class-validator`) as if they were the only possible path, with no branch for a different confirmed backend or frontend. This directly contradicted `FW-025`, which the architecture-generation side of the framework already takes seriously — `arch.template.md`'s own AI Guide for Section 3.1.1 already says *"If a different backend framework is confirmed: define the equivalent structure for that framework, following the same principles the NestJS layout below demonstrates"* — but nothing on the implementation-command side carried that same principle forward.

Checked `verify-story.md` for the same class of bug: it was already largely compliant, since it tests at the black-box level (`curl` against running HTTP endpoints, browser navigation, HTTP-triggered job invocation) rather than stack-specific unit-test syntax — inherently more portable. The one gap found: its `curl` examples hardcoded `/api/v1/` as a literal base path rather than sourcing it from the architecture document's Section 5 — fixed to reference "this project's confirmed base path," never assumed.

---

## What changes

- `implement-story.md`: Step 2 now establishes the confirmed-stack-first principle explicitly. Step 3's bootstrap branches on confirmed backend/frontend (NestJS/Next.js path vs. "scaffold with that framework's own tool, then match Section 3.1's actual documented structure"). Step 4's API/Web rule sections open with an explicit note that the shown syntax illustrates the confirmed stack, not a universal one. Step 5's check commands state the confirmed npm commands as this project's answer, with an explicit substitute-the-real-toolchain instruction.
- `verify-story.md`: `curl` examples reference the confirmed base path from Section 5 instead of hardcoding `/api/v1/`.
- Both fixes keep the NestJS/Next.js content fully worked out — nothing about this decision removes detail for the common case, it only adds the explicit fallback for the uncommon one.

## When adding a future command in this category

Any command that reads a Locked architecture document and acts on its confirmed stack (implementation, verification, deployment tooling, etc.) must state its stack-specific instructions as "if X is confirmed, do Y" with an explicit fallback to the architecture document's own real, per-project structure — never as an unconditional assumption.
