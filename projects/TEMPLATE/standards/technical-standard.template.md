# Technical Standard Template

> **[AI Guide — Document Level]**
> This is the shared shape for every framework/language/cross-cutting convention document
> `gen-architecture` generates — `api-standards.md`, `database-standards.md`,
> `security-standards.md`, `test-strategy.md`, `git-standards.md`, `ci-cd-pipeline.md`,
> `environment-strategy.md`, and any technology-specific standard named after whatever was actually
> confirmed in `system-architecture.md` (e.g. `nestjs-standards.md`, `nextjs-standards.md`,
> `typescript-standards.md` for this project's stack — `spring-standards.md`, `angular-standards.md`
> for a different one). One template, many instances — the shape stays the same, only the topic and
> the confirmed-technology-specific content changes.
>
> **File location:** `projects/{PROJECT_CODE}/standards/{standard-name}.md`

---

# {Standard Name}

## Purpose

{One or two sentences: what this document governs, and which confirmed technology/technologies it
applies to. State the deviation rule: deviations require a documented decision in the project's
Architecture Document.}

---

## Rules / Standards

> **[AI Guide]** Numbered topical sections. Use `### N. {Topic}` for each rule area — this is what
> every archived reference standard does (NestJS, Next.js, TypeScript, Security, Git standards all
> follow this numbering). For a standard that's more naturally a flat set of conventions with
> tables rather than numbered code-pattern rules (e.g. database column-naming conventions, CI/CD
> pipeline stages), topical `## {Topic}` sections without the numbered wrapper are equally valid —
> match whichever shape best fits the actual content; do not force numbering onto tabular
> reference material or vice versa.

### 1. {Topic}

{The rule, convention, or pattern. Include a code block if the rule is a code pattern — show it
exactly as it should appear in generated code, not as pseudocode.}

### 2. {Topic}

{...}

[Repeat for every rule area this standard covers]

---

## Examples

> **[AI Guide]** Optional — include when a worked, complete code example clarifies the rules above
> better than the individual rule snippets do (e.g. a fully validated DTO showing every rule from
> the Security or TypeScript standard applied together). Omit this section entirely if the rules
> above are already self-contained; do not pad with a redundant example.

### {Descriptive example title}

```{language}
{Complete, realistic code example — not a fragment}
```
