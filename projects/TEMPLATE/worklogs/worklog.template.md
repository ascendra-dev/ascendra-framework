# Worklog: {Person}

> **[AI Guide — Document Level]**
> One person's record of every change they want made to the project's main documents, kept while two or more people work on one project at the same time (worklog mode, `FW-053`). Each person writes only their own worklog. The main documents (the decision log, the build index, approved plans, the design, the architecture, the domain) are changed only by `/sync-worklogs`, which reads every worklog, finds the conflicts between them, has the PO resolve them, and applies everything in one session. Then each worklog is emptied of what was applied.
>
> Worklog mode is switched on in the project's own `CLAUDE.md`, in a section `## Worklog mode`, which names each person, their `git config user.name`, their worklog file, their ID prefix, and who runs the sync (the PO). A command that would change a main document checks that section first: if worklog mode is on, it writes an entry here instead.
>
> **The hold** (`projects/{PROJECT_CODE}/worklogs/hold.md`): whoever holds it edits the main documents directly, as if the project weren't in worklog mode, and writes no worklog entries; everyone else keeps writing their worklog meanwhile. One person holds it at a time. To take it: pull, check the file is empty, write your name, the time and why ("working alone today", "planning with Claude"), commit and push that file alone. To release it: commit and push your direct edits, empty the file, push. `/sync-worklogs` takes it for the length of the sync. The file holds one table, `| Held by | Since | Why |`, with one row, or none when free. Edits made under the hold are checked by the next sync like any other change to a target: an entry written against the older text shows as a stale base.
>
> **When writing this file (any command in worklog mode, or the person by hand):** append one entry per change, at the end of Entries, using the entry shape below. Never edit another person's worklog, never edit or delete an entry another command or person may rely on, and never edit a main document directly. Take the next ID from the header and raise "Next ID". Fill "Based on" with the version or commit of the target you read, so the sync can tell whether the target moved since. An entry that changes what another person is building right now is marked Urgent: say so to them as well, and ask the PO for an early sync.
>
> **When reading this file (`/sync-worklogs`; any command in worklog mode that needs the current state):** a target's true current state is the main document plus every unapplied entry on it, in every worklog. `/build-checkpoint` reads earlier Built entries for the same chunk here; `/sync-worklogs` reads every entry and applies it.
>
> **What this document is not:**
> - Not the decision log. Decisions here are proposals with provisional IDs (`Z-3`) until the sync numbers them into the project's decision log.
> - Not a journal or diary. Every entry targets a document and says exactly what changes there; working notes stay in your own scratch space.
> - Not for code. Code goes through branches and pull requests in the code repos; a Built entry only records what the plan should say about it.
>
> **File location:** `projects/{PROJECT_CODE}/worklogs/{name}.md`, one per person, named in the project's `CLAUDE.md` (for example `worklogs/zaka.md`). Created when worklog mode is switched on, and never deleted: the sync empties it.

| Field | Value |
|---|---|
| Person | [Name, role] |
| Git user | [`git config user.name`, exactly] |
| ID prefix | [One or two letters, e.g. `Z`] |
| Next ID | [Prefix and number, e.g. `Z-1`. IDs never restart, across syncs] |
| Last sync | [Date and the project repo commit of the last `/sync-worklogs`, or "none yet"] |

---

## Entries

> **[AI Guide]** One H3 per entry, oldest first. The heading is `### {ID} · {Kind} · {YYYY-MM-DD}`, with ` · Urgent` appended when it is. Then the fields table, then the body for that kind. Kinds are a closed list:
>
> | Kind | Use it for | Body |
> |---|---|---|
> | **Decision** | A choice with a real alternative, for the decision log or a plan's decisions table | **Question**, **Answer**, **Why** (naming the alternative rejected) |
> | **Built** | A checkpoint built (`/build-checkpoint`) | The Built block exactly as it goes into the plan, and the status change (`Planned → Built`) |
> | **Status** | A status change: a plan Approved, a unit Closed, a chunk accepted | From, to, and on whose say-so |
> | **Change** | Any other edit to a main document: a fix, a correction, a new row, a proposal for a document someone else owns | The exact text to replace (quoted) and the exact new text, or the exact text to add and where |
> | **Finding** | Something a later unit must act on | What, and which unit or document it goes to |
> | **New file** | A file you created that a main document should list (a new plan, a new reference) | The path, and the rows to add to which index |
> | **Question** | Something only the PO, or the other person, can answer before an entry can apply | The question, your recommendation, and which entries wait on it |
>
> **Correct:** "### A-4 · Change · 2026-11-26 — Target: `build/group-03-staff-and-roles.md` › B03.1.1 Endpoint block `GET /staff`, Performance — Based on: plan v1.0 (PAY-004 `8a65bc8`) — Replace 'Index `idn_staff (tenant_id, status)`' with 'Index `idn_staff (tenant_id, status, last_signed_in_at desc)`' — Why: the default sort reads it"
> **Incorrect:** "### A-4 — fixed the index thing in the plan" (no target, no base, no exact text: the sync can't apply it or check it)

### {ID} · {Kind} · {YYYY-MM-DD}

| Field | Value |
|---|---|
| Target | [Document path › section, or "new" for a New file] |
| Based on | [The target's version and the project repo commit you read, e.g. "plan v1.0 at `8a65bc8`"] |
| Summary | [One line] |
| Refs | [Commits in code repos, other entry IDs this depends on, the checkpoint or chunk] |
| Affects | [Work in progress this may change, yours or another person's; or "none"] |

[Body, by kind]
