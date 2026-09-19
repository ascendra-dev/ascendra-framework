# Cross-Reference Conventions

Verifiable rules for how any file in this repository cites another file. Apply these when writing a new file and when adding a cross-reference to an existing one. Each rule is independently checkable with a yes/no.

**Governing principle:** the split below tracks who actually reads a citing file, not a stylistic preference. A file whose primary reader is Claude mid-execution, or whose citations don't resolve to a real path in this repo, gains nothing from a markdown link. A file a human browses in a rendered view (GitHub, an IDE preview) benefits genuinely from one. Where a file's category is ambiguous, follow the reader test above rather than the letter of the rule below.

---

## Section 1 — Plain Backtick Citations Only

**XR-001** Every file under `.claude/commands/` cites another file with plain backtick text (`` `path/to/file.md` ``), never a markdown link. A command's primary reader is Claude executing it, not a human clicking through a rendered view — a link adds formatting complexity for a reader that gets no benefit from it.

**XR-002** Every file under `projects/TEMPLATE/**` — both `.template.md` files and `.example.md` files — cites another file with plain backtick text, never a markdown link, with one exception (`XR-003`). An `.example.md` file's citations reference a fictional project (`HARBORVIEW-INV-001`) that has no corresponding folder in this repo — a link would be a guaranteed dead link. A `.template.md` file's `[AI Guide]` prose citing another framework file for guidance is stripped from generated output and never resolves to a real path either.

**XR-003** **Exception to `XR-002`:** a template's own generation skeleton — the literal section content, using `{PROJECT_CODE}`-style substitution syntax, that gets copied directly into every real generated document — follows `conventions/command-conventions.md` C-043 instead, the same as any other generated content, since it resolves to a real path the moment `{PROJECT_CODE}` is substituted for a real project. `brief.template.md` Section 9's own example rows are the current case of this.

---

## Section 2 — Real Markdown Links

**XR-010** Every other framework-authoring document in this repo — `CLAUDE.md`, `SDLC.md`, `PROJECT-LIFECYCLE.md`, `README.md`, every `decisions/*.md`, every `conventions/*.md`, every `observations/*.md`, and every `practitioner-guide/*.md` — cites another file in this repo with a real markdown link, computed as a correct relative path from the citing file's own location, never plain backtick text.

**XR-011** Link text is the same backtick-styled path a plain citation would use, wrapped in link syntax: `` [`path/to/file.md`](relative/path/to/file.md) ``. Never bare prose as the link text (e.g. not `[the priming conventions](...)`), so a reader scanning for a literal filename can still find it visually.

**XR-012** A section-qualifier suffix (e.g. `§5.16`, `PC-020`) stays outside the link, as plain text immediately after the closing `)` — never inside the brackets or the parenthesized path, since the link target is the file, not the section.

**XR-013** Cross-references written into a *generated project document* (`brief.md` Section 9 and equivalent) follow `conventions/command-conventions.md` C-043 instead of this section — that rule already covers the generated-output case in full; this file governs the framework's own authoring documents.

---

## Checklist for Verifying a File's Cross-References

- [ ] XR-001  Every citation in a `.claude/commands/*.md` file is plain backtick text
- [ ] XR-002  Every citation in a `projects/TEMPLATE/**` template or `.example.md` file is plain backtick text, except a generation skeleton (`XR-003`)
- [ ] XR-003  A template's generation skeleton follows `C-043`, not `XR-002`
- [ ] XR-010  Every citation in `CLAUDE.md` / `SDLC.md` / `PROJECT-LIFECYCLE.md` / `README.md` / `decisions/*.md` / `conventions/*.md` / `observations/*.md` / `practitioner-guide/*.md` is a real markdown link with a correct relative path
- [ ] XR-011  Link text is backtick-styled, matching the path — never bare prose
- [ ] XR-012  A section-qualifier suffix sits outside the link, never inside it
- [ ] XR-013  A generated document's own cross-references follow `C-043`, not this file
