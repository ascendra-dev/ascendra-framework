# Implement Story

You are the Engineering Agent for Ascendra. Your job is to implement exactly one user story from the project backlog into the correct code repository. You implement what the story defines — nothing more, nothing less. The architecture document is the law: every table name, column name, endpoint path, DTO shape, security rule, and screen shape must match it exactly.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Read the story and gate check

Parse `$ARGUMENTS` to find the story file path (e.g. `projects/ASCENDRA-PAY-001/stories/EPIC-001/US-01-001-scaffold-api-project.md`).

Read the story file in full.

**Gate check — story must be Reviewed:**

Check the `**Status:**` field in the story header.

- If Status is `Reviewed` → proceed
- If Status is `Draft` → stop:
  > "Story `{ID}` has status `Draft`. Stories must pass review before implementation. Run `/review-stories` and then `update-status {story-path} Reviewed`."
- If Status is `In Progress`, `PR Created`, `Merged`, or `Done` → stop:
  > "Story `{ID}` has status `{status}`. This story has already been implemented or is past that stage. Check the story file for current state."
- If Status is `Deprecated` → stop:
  > "Story `{ID}` is Deprecated and should not be implemented."

**Gate check — architecture must be Locked:**

Read `projects/{PROJECT_CODE}/architecture/arch-v1.md`'s `**Status:**` field.
- If `Locked` → proceed
- If anything else → stop:
  > "Architecture is not yet Locked (current status: `{status}`). `/implement-story` treats the architecture document as law — it must be Locked before any code is written. Run `/review-architecture {PROJECT_CODE}`."

**Gate check — dependencies must be cleared:**

Read the story's Section 6 (Dependencies). For each entry that resolves to a Story ID (Pass 2 format, e.g. `US-01-003`), check that story's status.
- If every dependency is `Merged` or `Done` (or the section says "None") → proceed
- If any dependency is not yet `Merged`/`Done` → stop:
  > "Story `{ID}` depends on `{dependency-story-id}`, which is currently `{status}` — not yet Merged or Done. Implementing now risks building against tables, endpoints, or mechanisms that don't exist yet. Implement the dependency first, or confirm you want to proceed anyway (e.g. the dependency is a documentation-only formality)."
  Wait for PO response. Proceed only on explicit override.

**Gate check — implementation plan must be Confirmed (`FW-031`):**

Read `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md`.
- If the file does not exist → stop:
  > "No implementation plan found for `{ID}`. `/implement-story` requires a Product-Owner-confirmed plan before writing any code — run `/gen-story-plan {story-path}` first."
- If it exists but **Plan Status** is `Draft` → stop:
  > "Implementation plan for `{ID}` exists but is not yet Confirmed. Review `projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md` and run `update-status projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md Confirmed`, or re-run `/gen-story-plan` first if it needs changes."
- If **Plan Status** is `Confirmed` → proceed. This is a hard gate, no PO override — unlike the dependency and Open Decision gates above, a plan is cheap enough to produce and review that there's no legitimate case for implementing without one.

This plan is the authoritative task list for Step 4 — read it in full now, alongside the reference files in Step 2.

Extract from the story file:
- **Story ID** (e.g. `US-01-001`)
- **Title**
- **Project code** — derived from the file path (e.g. `projects/ASCENDRA-PAY-001/...` → `ASCENDRA-PAY-001`)
- **Acceptance criteria** — the numbered list in Section 3; these define exactly what you must deliver
- **Out of scope** — the list in Section 4; do not implement anything listed here even if it seems natural

---

## Step 2 — Read mandatory reference files

The Confirmed plan (read in Step 1) already states *what* to build — which endpoints, tables, and screens, in what order. The files below are where you confirm *how* to build it in this project's confirmed stack idiom, and where you resolve any plan detail that's abbreviated or missing. Read all of these before writing a single line of code. Do not skip any.

1. **Architecture quick-reference (read first):** `projects/{PROJECT_CODE}/architecture/arch-v1-ref.md`
   - Primary source for all implementation decisions: module-to-table-to-endpoint mappings, column names and types, enum values, auth guards and roles, file naming conventions, and critical constraints
   - This file covers the vast majority of what implementation requires. Read it before opening the full architecture document.
   - **If `arch-v1-ref.md` does not exist** (architecture locked before the ref file was generated): state that explicitly, read the full `arch-v1.md` as the primary source for this run, and recommend regenerating the ref file via `/review-architecture` before the next story. Do not silently proceed as if the ref file had been read.
   - **Determine the confirmed stack first, from Section 2 (Tech Stack):** per `FW-025`, there is no fixed Ascendra stack — every project's backend framework, frontend framework, database, and ORM are recommended per-project and recorded in `system-architecture.md` / arch-v1.md Section 2. **NestJS + Next.js/Ascendra UI is this project's own confirmed stack and the framework's common default — it is not a universal assumption.** Steps 3–5 below give the full, concrete NestJS/Next.js path since that's what's actually confirmed here. If you are ever running this command against a project whose Section 2 confirms something different (e.g. a Java/Spring backend, an Angular or plain React frontend, a different ORM): do not follow the NestJS/Next.js-specific commands and syntax literally — apply the same underlying principle (module boundary, migration tooling, DTO validation, RBAC enforcement, orgId scoping, sensitive-field exclusion, check/test commands) using that framework's own idiomatic mechanism, and follow arch-v1.md Section 3.1's actual documented structure for that stack, which `/gen-architecture` always writes out concretely regardless of which stack was confirmed.

2. **Every file in `projects/{PROJECT_CODE}/standards/` relevant to this story's target(s) (always read, not a fallback):** `api-standards.md`, `database-standards.md`, `security-standards.md`, any auth-provider-specific standard (e.g. `supabase-auth-standards.md`), `test-strategy.md`, and any per-technology standard for the stack this story touches. These are generated once by `/gen-architecture` and hold binding, specific rules that are never restated inline in `arch-v1.md`/`arch-v1-ref.md` — e.g. exactly how a token's signature is verified, a trigger required on every table with a given column, a required cross-origin policy. Treat `arch-v1.md`/`arch-v1-ref.md` as necessary but not sufficient: a rule that lives only in a standards file is still binding, and silently defaulting to generic framework knowledge instead of this project's own documented convention is a defect, not a reasonable judgment call.

3. **Architecture document (read only when ref file is insufficient):** `projects/{PROJECT_CODE}/architecture/arch-v1.md`
   - Read the full document only for: first-run bootstrap (repo scaffolding and README/CLAUDE.md creation — needs Sections 1, 2, 3.1, 7.3); DTO field-level validation detail not shown in the ref; seed data row values (Section 4.4); environment variable descriptions (Section 7.3); delegated-access mechanics (Section 6.3) for any story touching a delegated-access capability the architecture defines (an acting-on-behalf-of or assumed-role mechanism — only if this project's Section 6.3 defines one); Open Decisions (Section 8) relevant to this story's capability
   - When falling back, state explicitly: *"Reading full arch-v1.md for [specific detail] not covered by ref file"* — do not silently fall back
   - If a detail appears in neither file: it is not defined yet — state the gap rather than inventing a solution

4. **Open Decisions check (Section 8):** if this story's capability area has an open, unresolved decision recorded in Section 8, stop and flag it:
   > "Architecture Section 8 has an open decision affecting this story: '{decision text}'. Implementing now risks building against a choice that may still change. Resolve it first (`/assess-change` if it needs to become a formal decision), or confirm you want to proceed with a stated assumption."
   Wait for PO response before proceeding on override.

5. **Domain knowledge** — read any file referenced in the story's Notes section or its parent epic. Domain knowledge files are located in `projects/{PROJECT_CODE}/domain/`.

6. **UI component reference** (Web stories only) — check `projects/{PROJECT_CODE}/standards/` for a UI reference file (e.g. `ui-reference.md`). If present, scan it for the components relevant to this story before building any UI. If absent, use the UI library specified in the project's `system-architecture.md`.
   - **If the confirmed UI library is Ascendra UI:** also read `reference/ascendra-ui/hard-instructions.md` in full before building any UI. This file is a living list of corrections to how `ascendra-ui` is actually used in practice (found by PO review or direct investigation against `../ascendra-ui`'s real component source and real pages) — read *in addition to*, never instead of, `ascendra-ui/docs/ui-reference.md`/`showcase-reference.md`. Where a hard instruction and a docs template disagree, the hard instruction wins — it exists specifically to record cases where the docs were wrong or incomplete.
   - **Any story building a client-side validation schema:** also read `reference/form-validation-messages.md` before writing it — a general, UI-library-agnostic convention for validation message wording and check ordering.

7. **Screen design and approved mock** (Web stories only) — before building any UI:
   - Find this story's screen(s) in `projects/{PROJECT_CODE}/screens/screen-design.md` Section 3 (or architecture Section 3.1.3.3, once carried over) and read its Surface (Page/Dialog/Sheet/Drawer), UI Pattern, and Matched Reference.
   - **Then open the actual approved mock file** for this screen's portal — `projects/{PROJECT_CODE}/mocks/{portal-slug}-mock.html` — and locate this specific screen inside it. The mock's real markup (layout, component structure, copy, field order) is the visual ground truth the PO already approved — build the screen to match it, adapting only what's needed to wire it to real data and real endpoints. Do not redesign or reinterpret — a text description (Surface/Pattern) is a category, not a substitute for the actual approved shape.
   - Read `projects/{PROJECT_CODE}/architecture/arch-v1.md` Section 3.1.3.2 (Navigation Map) for this portal. Build the persistent nav exactly as listed — same items, same grouping, same visibility conditions — not a fresh navigation design.
   - If the story's screen is not found in the mock file either, state the gap rather than inventing a shape.

---

## Step 3 — Determine the target repository, bootstrap if needed, and branch

**Read the confirmed repo topology:**

Repository paths are an architecture decision, recorded only in the architecture document (`FW-027`). Read `projects/{PROJECT_CODE}/architecture/arch-v1.md` Section 3.1 (Project Structure) — already loaded in Step 2 — to find the confirmed repo name(s) (e.g. "Repository topology: `ascendra-pay-api` (NestJS backend) + `ascendra-pay-web` (Next.js frontend)", plus any additional repo declared per `FW-025` — mobile, worker, etc., each in its own unnumbered `####` subsection after 3.1.2).

Apply convention-over-configuration for the local filesystem path: each confirmed repo is a **sibling directory to this framework repo**, named exactly as stated in Section 3.1 — `../{repo-name}` (e.g. `../ascendra-pay-api`, `../ascendra-pay-web`, `../{project-code}-worker`).

If Section 3.1 does not name a repo for the target this story needs, stop and tell the user:

> "Architecture Section 3.1 does not name a [API/Web/Worker] repository for {PROJECT_CODE}. Repo topology must be confirmed in the architecture document before implementation — it is not decided at implementation time."

**Determine the target:**

Read the story header's `**Target:**` field first — `/gen-stories` populates it from the Locked architecture (values: `API`, `Web`, `Worker`, or a `+`-joined combination like `API+Web`). Cross-check it against the Confirmed plan's own Target field and its API Plan / Web Plan / Worker Plan sections (Section 2/3/4) — the plan was built from this same Target, so the two should already agree; a mismatch means the plan is stale (regenerate via `/gen-story-plan`) rather than something to silently resolve here.

**Fallback — only for stories generated before the Target field existed:** derive the target from the story content:

| Story content | Target | Directory |
|--------------|--------|-----------|
| Backend API, database schema, migration, API endpoint, background job triggered synchronously, seed script | **API** | `../{api-repo-name}` per Section 3.1 |
| Frontend page, screen, component, middleware | **Web** | `../{web-repo-name}` per Section 3.1 |
| Scheduled/async background job running in a dedicated worker process (only when Section 3.1 declares a worker repo) | **Worker** | `../{worker-repo-name}` per Section 3.1 |
| Requires more than one of the above | **Multiple** | Implement in dependency order: API (or Worker, whichever the others depend on) first, then Web |

If the target is genuinely ambiguous, state your interpretation and ask for confirmation before proceeding. If a story needs a worker repo but Section 3.1 declares none, stop and flag it as an architecture gap — do not invent a worker mechanism inline in the API repo.

**For each target directory this story needs, in order:**

**Bootstrap the repository if it doesn't exist yet:**

Check `../{repo-name}`. Three cases — a directory merely existing is not proof it was scaffolded correctly:

- **Does not exist** → this is the first story touching this repo. Scaffold it now, following the confirmed stack from Section 2 (Tech Stack) and Section 3.1 (instructions below).
- **Exists and passes the structural check** → skip to "Branch the story" below. The structural check: it is a git repository (`.git` present), it contains the confirmed stack's manifest at the root (e.g. `package.json` for a Node stack, `pom.xml`/`build.gradle` for Java, `pyproject.toml` for Python), and the top-level source layout matches what Section 3.1 documents for this repo (e.g. `src/modules/` exists for the NestJS layout).
- **Exists but fails the structural check** (missing `.git`, missing manifest, or layout doesn't match Section 3.1 — typically an interrupted or manual scaffold) → **stop and report exactly what is missing.** Do not build story code on a half-scaffolded base, and do not delete or re-scaffold over an existing directory without explicit PO confirmation.

**API — if the confirmed backend (Section 2) is NestJS:**
1. `cd ..` (to the sibling level, outside this framework repo) and run `nest new {api-repo-name}` (or `npx @nestjs/cli new {api-repo-name}` if the Nest CLI is not installed globally — do not treat a missing global CLI as a blocker) — use the package manager and options matching `system-architecture.md`
2. Restructure `src/` to match the module layout in Section 3.1.1 exactly (`src/modules/`, `src/db/`, `src/common/`)
3. **Wire logging and exception handling now, as part of scaffolding — never left for whichever story happens to touch `main.ts` first (`FW-041`).** Per `standards/logging-standards.md`, `reference/api-contract/contract.md` (`FW-042`), and the relevant Tier 2 standard's exception-handling rule: install the confirmed logger (e.g. `nestjs-pino` + its confirmed sink), add a global exception filter (`src/common/filters/`) and a typed business-rule-violation exception (`src/common/exceptions/`), wire both plus a `ClassSerializerInterceptor` into `main.ts`/`app.module.ts`, and initialize the confirmed error-tracking SDK if one is confirmed. The filter's response body must match `reference/api-contract/contract.md` Rule 1 exactly (field names included) — this is what a `/implement-story` run for the frontend repo will later generate its client against. This must exist before the first story's own endpoint is written, not retrofitted afterward. **If — and only if — `standards/database-standards.md` uses optimistic locking on any table (`FW-046`):** also add a second typed exception for a stale-version conflict (e.g. `StaleVersionException`, `409`, `code: 'STALE_VERSION'` per contract Rule 8) and give the filter an explicit branch for it (mirroring the business-rule-violation branch) — a generic `HttpException` fallback silently drops a custom `code` to `null`, which is wrong for a typed exception meant to carry one. Omit entirely, with no dead exception class, if no table uses optimistic locking.
4. Create `.env.example` from Section 7.3's backend environment variables (including the logging/error-tracking variables Section 7.3 lists)
5. Ensure the default branch is named `main`

**API — if a different backend is confirmed** (e.g. Spring Boot, Django, Express): scaffold using that framework's own standard tool (e.g. `spring init`, `django-admin startproject`), then restructure to match whatever layout Section 3.1.1 documents for this project — `/gen-architecture` always writes that section out concretely for the actual confirmed stack, never just for NestJS. Create `.env.example` from Section 7.3. Ensure the default branch is `main`.

**Web — if the confirmed frontend is Next.js + Ascendra UI (Section 2):** built on `ascendra-ui`'s component library, never scaffolded from scratch — but **not** a wholesale clone of the `ascendra-ui` repo, and **not** via `ascendra-ui`'s own `create-project.js`/`npm run upgrade` scripts. Those exist for standalone `ascendra-ui` consumers with no project-management system of their own; this framework already has one (sprints, releases, PR descriptions, its own `.claude/commands`), so shipping `ascendra-ui`'s generic project bookkeeping (`ASCENDRA.md`, `BACKLOG.md`, `CHANGELOG.md`, `scripts/`, `.ascendra-ui/`, its own `.claude/commands/`) would duplicate or conflict with it. Do this instead:
1. `git init` in a new `../{web-repo-name}` directory, default branch `main`
2. Copy `../ascendra-ui/ascendra-ui/` (the component library folder), excluding `template/`, into `{web-repo-name}/ascendra-ui/` — this folder is managed: never hand-edit files inside it; refresh it wholesale when a sync is needed (see "Keeping Ascendra UI current" below)
3. Copy `../ascendra-ui/ascendra-ui/template/app/{layout.tsx,globals.css,favicon.ico}` into `{web-repo-name}/app/` as the root shell only — do **not** copy `(app)/` (that's `ascendra-ui`'s own showcase shell — a getting-started page and a blank sandbox nobody in a real deliverable needs). This project's real route groups come from Section 3.1.2, added as this and later stories build them.
4. Copy root config needed to build — `components.json`, `next.config.ts`, `tsconfig.json`, `postcss.config.mjs`, `eslint.config.mjs`, `global.d.ts` — from `../ascendra-ui`'s repo root, with two required edits to `eslint.config.mjs`: drop the override block referencing `create-project.js`/`ascendra-ui/template/scripts/**/*.js` (those paths only exist inside the `ascendra-ui` repo itself and are dead config in a consumer), and add `"ascendra-ui/**"` to `globalIgnores` (the source repo lints its own `ascendra-ui/` as first-party code; in a consumer it's a vendored, never-hand-edited folder and must not be linted — the source config does not have this ignore, it must be added here).
5. Copy `../ascendra-ui/docs/` into `{web-repo-name}/ascendra-ui/docs/` (nested under the library folder — it's library reference documentation, not this project's own docs, which live in this framework repo under `projects/{PROJECT_CODE}/`)
6. Create empty `components/`, `hooks/`, `lib/`, `providers/`, `utils/` folders for this project's own code. **Project-specific hooks go in `hooks/` — never nested under `lib/hooks/`.**
7. Set up the test tooling `standards/test-strategy.md` specifies for this repo (e.g. Vitest config + setup file + test script) — `ascendra-ui`'s scaffold ships no test tooling of its own, so this is never inherited, only ever set up fresh from the project's own test strategy.
8. Write `package.json`: `name` set to `{web-repo-name}`, `dependencies`/`devDependencies` copied from `../ascendra-ui`'s `package.json` plus whatever test tooling Step 7 requires, `scripts` limited to `dev`/`build`/`start`/`lint`/test (per Step 7) — no `upgrade`/`changelog`, there's no script to run them
9. Write `.gitignore` (standard Next.js — `node_modules/`, `.next/`, `*.tsbuildinfo`, `next-env.d.ts`, `.env*` — **with an explicit `!.env.example` exception**, otherwise the blanket `.env*` pattern silently untracks the one env file that must be committed) and `.env.example` from Section 7.3's frontend environment variables
10. **Generate the frontend auth session module now, before the API client depends on it (`FW-043`).** Per this project's confirmed auth provider and its auth-provider-specific standard (e.g. `supabase-auth-standards.md`) if one exists, otherwise `security-standards.md`'s frontend session/token-retrieval convention: generate `lib/auth.ts` (browser-only — session/token retrieval and any session-claims decoding used by Client Components, e.g. a `getBrowserAccessToken()`-style function) and, if the framework has a client/server execution split, a separate `lib/auth.server.ts` (Server Component/Route Handler session access) — split into two files specifically so a server-only import never leaks into a Client Component's bundle. This must exist before item 11 below, whose request interceptor calls directly into this module's token getter.
11. **Wire error handling now, as part of scaffolding — never left for whichever story happens to render the first screen (`FW-041`).** Per `standards/logging-standards.md`, `reference/api-contract/contract.md` (`FW-042`), and the relevant Tier 2 standard's error-handling rule:
    - Add `app/error.tsx`, `app/global-error.tsx`, `app/not-found.tsx` (Next.js App Router's error-boundary convention — using `ascendra-ui`'s own `Empty`/`EmptyHeader`/`EmptyMedia`/`EmptyTitle`/`EmptyDescription` primitives, not a hand-rolled layout).
    - Generate `lib/api/error.ts` (or equivalent) — a typed error class implementing `reference/api-contract/contract.md` Rule 1's exact shape (`statusCode`, `code`, and a `requestId` read from the `X-Request-Id` response header), with a `displayMessage`-style getter that shows a generic message + the `requestId` as a reference for `statusCode === 0`/`>= 500`, and the real `message` otherwise. This is written fresh, against this specific project's confirmed backend contract — there is no shared client library to copy it from.
    - Generate `lib/api/client.ts` (or equivalent) — the axios (or equivalent) instance for this project's confirmed `NEXT_PUBLIC_API_BASE_URL`. This is the shared client only (interceptors, no per-domain functions — see `FW-044` and the "API calls"/"Hooks" Web implementation rules below for where per-domain code goes as stories add it). Request interceptor: attaches `Authorization: Bearer <token>` using item 10's `lib/auth.ts` token getter (never assume a specific auth library inline here), and generates an `Idempotency-Key` (e.g. `crypto.randomUUID()`) on every `POST`/`PATCH`/`PUT` per contract Rule 7. Response interceptor: normalizes every failure through the typed error class above, and reports `statusCode === 0`/`>= 500` failures to the confirmed error-tracking SDK if one is confirmed (a 4xx is an expected UI state, not a bug worth paging anyone over — never report those).
    - **If — and only if — the backend's `standards/database-standards.md` uses optimistic locking on any table (`FW-046`):** add an `isStaleVersion`-style getter (`code === 'STALE_VERSION'`) to `lib/api/error.ts`'s typed error class — every version-guarded mutation across every domain branches on this the same way, so it belongs here once, not re-checked as a string comparison at each call site — and export a `withIfMatch(version: number)` helper from `lib/api/client.ts` (contract Rule 8) returning the request-config shape carrying the `If-Match` header, quoted (`"3"`) to match what the backend emits/expects, so every domain's version-guarded mutation attaches the header identically instead of hand-rolling the header name/quoting per call site. Unlike `Idempotency-Key`, this is never applied blindly by the interceptor to every request — only some resources are version-tracked, and the value is resource-specific, not generated fresh per call — so it stays an explicit opt-in helper each version-guarded API function calls, not interceptor logic. Omit both entirely, with no dead code, if the backend uses no optimistic locking anywhere.
    - Initialize the confirmed error-tracking SDK's instrumentation files if one is confirmed.
    A route group's own error boundary (e.g. `app/(issuer-app)/error.tsx`) is added once that route group itself is scaffolded, not before.
12. Write `CLAUDE.md` and `README.md` from this project's actual context (`standards/system-architecture.md`, the domain glossary, `standards/git-standards.md`) — not `ascendra-ui`'s generic templates. Cover what's genuinely load-bearing: `ascendra-ui/` is managed and must never be hand-edited, the `@/ascendra-ui` import convention, where the reference docs live (`ascendra-ui/docs/ui-reference.md`, `ascendra-ui/docs/showcase-reference.md`), and this project's actual branch/release model — not a generic feat/fix/chore workflow.
13. Do not copy: `.claude/commands/`, `ASCENDRA.md`, `BACKLOG.md`, `CHANGELOG.md`, `scripts/`, `.ascendra-ui/`, `.vscode/`, `next-env.d.ts` (Next regenerates it; it's git-ignored, copying it accomplishes nothing)
14. `npm install`
15. Commit: `chore: scaffold {web-repo-name} from ascendra-ui`

**Keeping Ascendra UI current:** no version manifest is tracked, so currency is determined by direct comparison, not a version number. When a story or `/assess-change` determines the project needs a newer `ascendra-ui` (a new/changed component, a shadcn primitive update): diff `../ascendra-ui/ascendra-ui/` (excluding `template/`) against `{web-repo-name}/ascendra-ui/` directly, and show the PO the diff before applying anything. On confirmation, re-copy the changed files from `../ascendra-ui/ascendra-ui/` wholesale (always safe — that tree is fully managed) and re-copy `../ascendra-ui/docs/` the same way. Then diff `../ascendra-ui/package.json` dependencies against `{web-repo-name}/package.json` and flag any differences for the PO to approve before running `npm install`. Never touch `components/`, `hooks/`, `lib/`, `providers/`, `utils/`, or any file outside `ascendra-ui/`/`ascendra-ui/docs/` during a sync.

**Web — if the confirmed frontend is Next.js *without* Ascendra UI** (Section 2 confirms Next.js but the PO chose a different foundation — recorded as a Section 9 deviation by `/gen-architecture`): do **not** clone `ascendra-ui`. Scaffold with `npx create-next-app@latest {web-repo-name}` using the options matching `system-architecture.md`, install only the component library Section 2 actually confirms, then structure route groups to match Section 3.1.2. Create `.env.example` from Section 7.3. Ensure the default branch is `main`.

**Web — if a different frontend is confirmed** (e.g. Angular, Vue, a plain React SPA): there is no `ascendra-ui`-equivalent component library to clone unless Next.js + Ascendra UI was actually confirmed for this project. Scaffold using that framework's own standard tool (e.g. `ng new`, `npm create vite@latest`), then build to match whatever structure Section 3.1.2 documents. Create `.env.example` from Section 7.3. Ensure the default branch is `main`.

**Worker (only when Section 3.1 declares one):** follow that unnumbered `####` subsection's own structure the same way — scaffold per its confirmed stack (never assume Node/NestJS), restructure to match its documented layout, `.env.example` from Section 7.3, fresh `git init` if not cloned from an existing base.

After scaffolding any repo, commit the scaffold (`chore: scaffold {repo-name}`) before proceeding to branch for the story itself — the scaffold and the story's own changes are separate commits.

**Branch the story:**

Per `standards/git-standards.md`'s branch model (trunk-based, one branch per story): in the target directory, check if `story/{STORY-ID}-{short-slug}` already exists locally.
- **Exists** (resuming interrupted work): check it out.
- **Does not exist:** ensure `main` is up to date, then create and check out `story/{STORY-ID}-{short-slug}` from `main` (e.g. `story/US-04-001-register-payer-manually`).

Do not write any story code directly on `main`.

---

## Step 4 — Implement the story

Build exactly the artifacts declared in the Confirmed plan's API Plan / Web Plan / Worker Plan sections (`API-1`, `WEB-1`, `WORKER-1`, etc.) — that is your task list. The rule sections below tell you *how* to build each one in this project's confirmed stack idiom; the plan already told you *what*. Stop at the story's Section 4 (Out of Scope) boundary — do not implement anything listed there even if the plan's predicted file paths seem to invite it.

**If you discover mid-implementation that you must diverge from the plan** (a technical constraint the plan didn't anticipate, an assumption in the plan that turns out wrong): do not silently build something different. Add an entry to the plan file's Section 7 (Deviations from Plan) stating what changed and why, then proceed with the corrected approach. `/verify-story`'s Plan Conformance check reads this section — a documented deviation is normal engineering judgment; an undocumented one is a defect.

---

### First-run bootstrap (README/CLAUDE.md — applies to any target repo, in addition to the scaffolding in Step 3)

Before writing any story-specific code, check whether the target repository root contains `README.md` and `CLAUDE.md`. If either is missing, create it now.

The two templates below show this project's actual confirmed stack (Node.js/npm) — if a different stack is confirmed elsewhere, replace the Prerequisites and every command shown (install, migrate, dev server, lint/build/test) with that stack's own real equivalents; do not paste npm commands into a repo that doesn't use npm.

**`README.md`** — project-level documentation:
```
# {Project Name} API          (or Web / Worker)

{One sentence from Section 1 of arch-v1.md describing what the system does.}

## Stack
{Copy the tech stack table from Section 2 of arch-v1.md, keeping only the rows relevant to this repo.}

## Development setup

### Prerequisites
- Node.js 20+
- Docker + Docker Compose

### Start local services
\`\`\`bash
docker compose up -d
\`\`\`

### Install dependencies
\`\`\`bash
npm install
\`\`\`

### Apply database migrations   (API only)
\`\`\`bash
npm run db:migrate
npm run db:seed
\`\`\`

### Run the development server
\`\`\`bash
npm run dev
\`\`\`

### Run checks
\`\`\`bash
npm run lint
npm run build
npm test
\`\`\`

## Project structure
{Copy the relevant Section 3.1.x subsection from arch-v1.md.}

## Environment variables
Copy \`.env.example\` to \`.env\` and fill in values. See Section 7.3 of the architecture document for descriptions.
```

**`CLAUDE.md`** — Engineering Agent guard file. Replace `{framework-repo-name}` with the actual directory name of this framework repo (the repo you are running `/implement-story` from — check its own folder name, do not assume a fixed name):
```
# {Project Name} API          (or Web / Worker)

This is a code-only output repository. All implementation work is orchestrated from
the sibling `{framework-repo-name}` repository using the `/implement-story` command.

Do not add features, endpoints, pages, or schema changes without a corresponding
approved user story in `{framework-repo-name}/projects/{PROJECT_CODE}/`.

If you have opened this repo directly, the right place to start is `{framework-repo-name}`.

## Architecture reference
Quick-reference (read first): `../{framework-repo-name}/projects/{PROJECT_CODE}/architecture/arch-v1-ref.md`
Full document (fallback): `../{framework-repo-name}/projects/{PROJECT_CODE}/architecture/arch-v1.md`

## Development commands
npm run dev          Start development server
npm run db:migrate   Apply database migrations     (API only)
npm run db:seed      Run seed script               (API only)
npm run lint         ESLint
npm run build        TypeScript compile
npm test             Vitest test suite
```

---

### API implementation rules

> The concrete syntax below (decorators, class-validator, Drizzle) illustrates NestJS — this project's confirmed backend (Section 2). If a different backend is confirmed for the project you're running this against, apply the same principle each rule states (module boundary, DTO validation, RBAC enforcement, orgId scoping, sensitive-field exclusion) using that framework's own idiomatic mechanism — check Sections 3.1/5/6 of that project's arch-v1.md for the real names and structure to use, never the NestJS syntax below.

**Module structure** (Section 3.1.1 of arch doc)
- Every module lives at `src/modules/{module-name}/`
- Files within: `{name}.controller.ts`, `{name}.service.ts`, `{name}.module.ts`, `dto/` subfolder
- Register every new module in `src/app.module.ts`

**Database schema** (Section 4.2)
- Schema files live at `src/db/schema/{module-name}.ts`
- Column names, types, indexes, and nullability must match the arch doc verbatim — do not add or remove columns without a documented decision
- Every `updatedAt` column requires the SQL trigger shown in the arch doc — include it as a comment or in a migration file
- Every insert this story performs must set `createdBy`; every update must set `updatedBy`; every soft-delete must set `deletedBy` alongside `deletedAt` — these are never left null by default (`database-standards.md`, `FW-035`/`FW-036`/`FW-037`). Thread the acting value from the verified session (`@CurrentUser()` or this project's own equivalent) explicitly through controller → service → repository — there is no ORM- or trigger-level mechanism that sets these automatically. **The exact shape of that value is whatever this project's own `database-standards.md` Rule 4 states, never assumed** — it may be an actor id (FK-based projects) or a self-contained actor label (projects with more than one actor identity space, `FW-037`/`FW-038`); read that file, don't default to one or the other. The one common exception either way: an entity's own "self-signup" flow, where the actor's own record doesn't exist yet when the transaction starts — pre-generate that record's id and self-reference it (the same pattern as a self-referential tenant/org id); for the FK shape, never derive the actor id from the identity provider's own user id, which will not satisfy the foreign key.
- After creating or modifying a schema file, run `npm run db:migrate` in the target directory

**Enums** (Section 4.3)
- Use the exact enum values listed — do not add values

**Seed data** (Section 4.4)
- If this story requires seed data rows, add them to `src/db/seed.ts` using upsert logic so the script is idempotent — never destructive (never truncate/delete existing rows)
- Run `npm run db:seed` after `db:migrate` and confirm it is safe to re-run

**Endpoints and DTOs** (Section 5)
- HTTP method, URL path, auth requirement, request body shape, and response shape must match Section 5 exactly
- DTO classes use class-validator decorators as shown; field types follow the project's documented data-representation standards exactly (e.g. where the architecture stores money as integer minor units, the DTO uses `@IsInt()` — never `@IsNumber()` or `@IsDecimal()`)
- Apply `ValidationPipe` globally — do not add it per-controller

**Authentication** (Section 6.1)
- Use the exact guard name(s), decorator(s), and token-verification mechanism documented in this project's `security-standards.md` (and its auth-provider-specific standard, e.g. `supabase-auth-standards.md`, if one exists) — never a generic placeholder guard name. In particular, exactly how a token's signature is verified (a shared secret vs. an identity provider's JWKS/public-key endpoint) is a specific, binding detail stated in that file, not something to default from general framework familiarity.
- Public endpoints use whichever exemption mechanism that same file documents (e.g. a `@Public()` decorator), only if this project's `security-standards.md` defines one — otherwise every endpoint is protected by default

**Authorisation / RBAC** (Section 6.2)
- Apply `@Roles(...)` exactly as defined in Section 6.2 for each endpoint
- If the architecture defines a tenant/organisation boundary (Section 6.2), the tenant identifier is **always** extracted from the JWT payload via `@CurrentUser()` — never from the request body, URL param, or query string
- If Section 6.1/6.2 defines portal-specific or role-specific guards, use those exact guard names for the matching endpoints — never invent a guard name or substitute the baseline guard where a specific one is defined

**Delegated access** (Section 6.3 — applies only if this project's architecture defines a delegated-access mechanism: acting-on-behalf-of, assumed-role, support-access grants, or similar. Required for any story touching such a capability; skip this block entirely if Section 6.3 defines none)
- A delegated session's token carries the *assumed* identity's tenant scope and the assumed role/capability's permissions only — never merge or union with the acting user's own identity permissions
- Every action taken during an active grant must write to the audit log table specified in 6.3, unconditionally — in addition to, not instead of, standard request logging
- Check grant revocation state on every request under a delegated token, not just at session start — a mid-flight revocation must reject the next request

**Queries**
- Write Drizzle queries directly in the service — no repository classes
- If the architecture defines a tenant/organisation boundary: every query that reads tenant data must filter on the tenant column exactly as Section 6.2 names it (e.g. `.where(eq(table.orgId, orgId))`), using the scoping helper the architecture defines when one exists (e.g. `forOrg()`)
- Soft-deleted rows must be excluded via `isNull(table.deletedAt)` or `isNull(table.archivedAt)` as appropriate

**Sensitive fields** (Section 6.4)
- Apply `@Exclude()` to every field listed in Section 6.4 for this module
- Import `ClassSerializerInterceptor` in the module and apply it via `APP_INTERCEPTOR`

---

### Web implementation rules

> This project's confirmed frontend (Section 2) is Next.js + Ascendra UI, scaffolded from `ascendra-ui`'s component library per the steps above — the rules below assume that. If a different frontend is confirmed for the project you're running this against, apply the same principle (route/screen structure matching the architecture doc, using the confirmed component library, never hardcoding API URLs, keeping privileged calls server-side) using that framework's own idiomatic mechanism and whatever Sections 3.1.2/5 document for it.

**Route structure** (Section 3.1.2 of arch doc)
- Pages live under the correct Next.js route group per portal, matching Section 3.1.2's folder names exactly
- Follow the exact folder names listed — do not invent a new route group

**Screens** (Section 3.1.3.3, cross-checked against the approved mock per Step 2 item 7)
- Build the screen to match the approved mock's actual markup and layout — the mock is the PO-approved visual reference, not a starting suggestion
- Wire the mock's static structure to real data (API calls) and real form submission — do not change the visual shape while doing so

**Validation messages** — read `reference/form-validation-messages.md` before writing any client-side validation schema (Zod or whatever the project confirms). Every message names the actual field (its visible label text, not the schema key) — never a bare "Required"/"Invalid X". Within one field's chain, the required check always comes first, so an empty field shows a "required" message, never a format/length message about nothing having been entered.

**Navigation** (Section 3.1.3.2, per Step 2 item 7)
- Build the portal's persistent nav exactly as the Navigation Map lists it — same items, same grouping, same visibility conditions per role/capability
- Any cross-cutting UI rule in Section 3.1.3.5 (e.g. a blanket read-only mode for a review role, a delegated-access indicator banner) applies across every screen in the portal, not just the ones this story touches directly — check whether this story's screen needs a gating caption or banner

**Components**
- Use the UI component library defined in `projects/{PROJECT_CODE}/standards/system-architecture.md` — from `ascendra-ui`, copied locally into `ascendra-ui/` at scaffold time; do not install additional UI libraries without a documented decision
- Check `ascendra-ui/docs/ui-reference.md` (component API) and `ascendra-ui/docs/showcase-reference.md` (page patterns) before writing any UI code — or `projects/{PROJECT_CODE}/standards/ui-reference.md` if this project maintains its own
- **Also read `reference/ascendra-ui/hard-instructions.md`** (Step 2 item 6) before writing any UI code, and follow it over the docs above where the two disagree
- When a form, dialog, sheet, or drawer resembles an existing pattern under `../ascendra-ui/components/{forms,dialogs,sheets,drawers}/`, copy that real page's structural nesting exactly (per AUI-007) — do not hand-tune spacing or invent a structure the docs only describe in prose
- New project-specific hooks go in the web repo's root `hooks/` folder — never nested under `lib/hooks/`, and never inside `ascendra-ui/` (that tree is managed and gets overwritten on a sync)

**API calls**
- Use the endpoint paths from Section 5
- Always call through this repo's own `lib/api/client.ts` — generated once at scaffold time (Step 3 item 11) against `reference/api-contract/contract.md` (`FW-042`). Never create a new axios/fetch instance, never hardcode the base URL or an Authorization header inline — the client's interceptors already handle both.
- **Domain-namespaced files, created as each domain is first touched (`FW-044`):** this story's first endpoint in a given domain creates `lib/api/{domain}.api.ts` (the calling functions) and `lib/api/{domain}.types.ts` (its request/response interfaces) — e.g. `lib/api/staff.api.ts`/`lib/api/staff.types.ts`. Never append a new domain's functions to an existing unrelated domain's file, and never add per-domain functions to `lib/api/client.ts` itself. If a call needs something `lib/api/client.ts` doesn't yet expose (a new interceptor behavior), extend that file directly — don't work around it at the call site.
- Every caught API failure is the typed error class from `lib/api/error.ts` (via the client's response interceptor) — never a raw `catch (err)` on an untyped error.
- **Version-guarded resources (contract Rule 8, `FW-046`):** if `arch-v1.md` Section 5 marks an endpoint "Requires `If-Match`," its `.api.ts` function takes `version: number` as a parameter and calls the client with `lib/api/client.ts`'s `withIfMatch(version)` — never hand-construct the header inline. The matching `.types.ts` response interface(s) for that domain include a `version: number` field on every version-tracked object (a single-resource shape mirrors the `ETag` header's value into the body the same way; a list shape carries it per-row, per contract Rule 8) — this is what a later mutation call reads `version` from, there is no separate fetch-the-current-version step.

**Hooks** (only if this project's confirmed State Management / Data Fetching choice, `system-architecture.md` Step 5, uses hooks — e.g. React Query, SWR; skip this rule entirely if Step 5 confirmed none)
- One file per domain: `hooks/{domain}.hook.ts`, holding every hook for that domain (e.g. `useStaffList`/`useAddStaff`/`useRemoveStaff` all in `hooks/staff.hook.ts`) — never one file per hook, never a subfolder per domain (`FW-044`).
- A cross-cutting, non-domain-specific hook (e.g. session-claims decoding) still gets its own single-concern file under the same convention (e.g. `hooks/session.hook.ts`) — never left unnamespaced at the `hooks/` root.
- **Version-guarded mutations (`FW-046`):** the mutation hook takes whatever object the caller already has in hand (from a prior list/detail query, which already carries `version` per the API-calls rule above) — never a bare id — and passes its `version` straight through to the `.api.ts` call; a hook never re-fetches just to obtain a version. On a `STALE_VERSION` error (`ApiError.isStaleVersion`), invalidate that resource's query in `onError` in addition to surfacing `displayMessage` — the cached copy is now known-wrong, so the generic recovery is "show the rejection and refresh the view," not a silent retry.

**Auth**
- Server components and route handlers read the session from the Next.js auth session
- Never call a privileged API endpoint from a client component — use a server action or API route handler as a proxy

---

### Worker implementation rules (only when Section 3.1 declares a worker repo)

**Job structure** (per the worker's own `####` subsection in Section 3.1)
- Follow the confirmed job/queue framework's own idiomatic structure as documented there — one file/handler per job type
- Register every new job in whatever central registry the confirmed framework uses (e.g. a queue module, a cron registry)

**Data access**
- Same tenant-isolation and soft-delete rules as the API's Queries rules above — a worker reading or writing tenant data is not exempt from tenant scoping
- If the worker shares the database with the API, use the same schema definitions (Section 4.2) — do not redefine them locally

**Idempotency**
- Every job must be safe to run twice (retries, at-least-once delivery) — check for existing effect before applying it (e.g. check a status field before transitioning it) rather than assuming single delivery

**Observability**
- Log job start, completion, and failure with enough context (org, entity ID, job name) to trace a single run — background failures have no user-facing error to surface otherwise

---

### Standards Compliance Self-Check (run before Step 5)

Before running automated checks, re-read every file in `projects/{PROJECT_CODE}/standards/` relevant to the target(s) this story touched (API stories: `api-standards.md`, `database-standards.md`, `security-standards.md`, `logging-standards.md`, any auth-provider-specific standard, plus any per-technology standard; Web stories: `logging-standards.md`, the frontend per-technology standard). For each rule stated as a requirement (a "must," "every," "always," or a named required pattern), check the actual diff against it now, by re-reading the rule text — not from memory of what was written earlier in this same session. Fix anything not satisfied before proceeding to Step 5.

This check exists because standards-file rules are easy to satisfy in the plan and miss in the code, and automated lint/build/test cannot catch them: a missing database trigger, an unwired CORS policy, or a token-verification mechanism that diverges from what the project's own standard specifies will all pass lint, build, and a naive test suite. Only re-reading each rule against the actual diff catches these.

**Web stories on Ascendra UI — also re-check against `reference/ascendra-ui/hard-instructions.md`:** re-read every entry and check the actual UI diff against it (e.g. `FieldLegend` vs a raw heading, `FieldHint` vs `FieldError`, table-page action placement). Same reasoning as above — these are exactly the kind of detail lint/build/test cannot catch, and are easy to get right in one screen and drift on the next unless re-checked against the diff each time.

**Any story with a validation schema — also re-check against `reference/form-validation-messages.md`:** confirm every message names its field and that required checks come first in each field's chain. Same reasoning — a naming slip or wrong check order compiles and passes tests cleanly.

**Every story — also re-check against `standards/logging-standards.md` (`FW-041`):** did this story throw a business-rule rejection via the confirmed typed exception (not a generic malformed-input error)? Does any new log call risk a sensitive field going out unredacted? Does a new Web screen's failure path go through the centralized API-error handling, not a one-off `try/catch`? This file governs the same kind of easy-to-satisfy-once-and-drift-on-the-next-story detail as the two checks above.

**Every story — also re-check against `reference/api-contract/contract.md` (`FW-042`):** on the API side, does every new endpoint's success/error response match the contract's field names exactly (not just "has a `code` field")? Is a business-rule rejection genuinely `422` via the typed exception, not a repurposed `400`? On the Web side, does every new/touched call go through `lib/api/client.ts`'s centralized client — never a one-off `axios`/`fetch` call — and does every new error path produce the typed error class from `lib/api/error.ts`, not a raw caught exception? **If this story touched an endpoint `arch-v1.md` Section 5 marks "Requires `If-Match`" (Rule 8, `FW-045`/`FW-046`):** does the handler actually require and compare `If-Match` (not just increment `version` on a successful write, which satisfies the column but not the concurrency check itself)? Does a mismatch reject via the project's stale-version exception, mapped to `409` with its own `code`, not a repurposed `422`/`400`? On the Web side, does the call site pass `version` through `withIfMatch` rather than a hand-rolled header, and does the mutation hook take the object it already has rather than re-fetching? Same reasoning as the checks above: a call site that bypasses the centralized client compiles and passes tests cleanly, and only re-reading the contract against the actual diff catches it.

**Every Web story that added a new domain's API code — also re-check placement against `FW-044`:** did this story's new functions/types land in that domain's own `lib/api/{domain}.api.ts`/`.types.ts` (and, if hooks apply, `hooks/{domain}.hook.ts`) rather than an existing unrelated domain's file or the shared client itself? Same reasoning as the checks above — a function added to the wrong file compiles and passes tests cleanly, and only re-checking placement against the diff catches it.

---

## Step 5 — Run checks and commit

For each target directory this story touched, run this project's confirmed lint / build / test commands (per `test-strategy.md` / `system-architecture.md`). For this project's confirmed Node/TypeScript stack, that's:

```bash
cd {target-directory} && npm run lint
cd {target-directory} && npm run build
cd {target-directory} && npm test -- --run
```

If a different toolchain is confirmed (e.g. Maven/Gradle for a Java backend), run that toolchain's equivalent commands instead — never assume npm scripts exist.

Resolve all lint errors and build/type errors before reporting. If a test fails, fix it. If you cannot fix a failure after two attempts, report it explicitly — state exactly what failed and why — rather than suppressing or skipping it.

If no test file exists yet for a new module, write a minimal smoke test that confirms the module is registered and the controller responds to at least one endpoint.

Before committing, confirm none of `git-standards.md`'s "what never goes in git" items are staged (`.env` files with real values, secrets, generated build output, database dumps with real data).

Once checks pass in every touched directory, commit on the story branch using `git-standards.md`'s format: `{type}({scope}): {summary}` (e.g. `feat(invoices): add void-and-reissue endpoint`). One commit per repo touched — if this story spans API and Web, commit each separately in its own repo.

---

## Step 6 — Report to the user

Report one block per target repository touched (repeat for each if the story spanned more than one), then the AC coverage summary:

```
STORY IMPLEMENTED — {Story ID}: {Story Title}
─────────────────────────────────────────
Repo:          {repo-name} ({API / Web / Worker})
Branch:        story/{STORY-ID}-{short-slug}
Bootstrapped:  Yes — {what was scaffolded} / No
Files created: {list, paths relative to repo root}
Files modified:{list, paths relative to repo root}
Checks:        Lint ✓  Build ✓  Tests ✓  (or: describe failure)
Committed:     {commit message used}
─────────────────────────────────────────
Plan:          projects/{PROJECT_CODE}/story-plans/US-{epic}-{seq}-plan.md
Artifacts built: {API-1, WEB-1, ... — list which plan artifacts were built}
Deviations:    {count — "None" or list, each also recorded in the plan's Section 7}
─────────────────────────────────────────
AC coverage:
  AC1 — {met / not met and why}
  AC2 — {met / not met and why}
─────────────────────────────────────────
Next step:
Update story status when satisfied with the implementation (PO decision — not
done automatically). Then /verify-story to test against the running application,
then /gen-pr-description to raise the PR from the branch above.
```

Do not change the story status yourself. Status updates are always made by the Product Owner.
