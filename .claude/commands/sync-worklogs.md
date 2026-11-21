# Sync Worklogs

Brings every person's worklog into the project's main documents in one session, for a project in worklog mode (`FW-053`), where two or more people work at once and each writes only their own `worklogs/{name}.md`. It reads every unapplied entry, finds what clashes (the same section edited twice, two answers to one question, a decision that undoes someone's built work, an entry written against a document that has since moved), puts every conflict to the PO at once with a recommendation, and then applies everything: decisions numbered into the decision log, Built blocks and statuses into plans, changes, findings and new files into their documents, the build index and status recounted.

Run it at each chunk's acceptance, and early for any entry marked Urgent. Only the PO runs it to apply; anyone may run it as `check`, which reports the conflicts and writes nothing. It commits nothing: it proposes the commit, and after it is pushed every person pulls before writing again.

---

## Arguments received

`$ARGUMENTS`

---

## Step 1 — Parse arguments

Required: a project code. Optional: the mode.

- **Project code:** e.g. `ASCENDRA-PAY-004`
- **Mode:** `apply` (default) or `check` (read only: report what would apply and every conflict, write nothing)

If the project code is missing:

> "Which project are these worklogs for? Provide the project code (e.g. `ASCENDRA-PAY-004`)."

Wait for PO response.

---

## Step 1.5 — Gate check

| Check | How to verify | Failure message |
|-------|--------------|----------------|
| Worklog mode is on | The project's `CLAUDE.md` has a `## Worklog mode` section naming each person, git user, worklog file, ID prefix and the sync owner | > "`{CODE}` isn't in worklog mode: its `CLAUDE.md` has no `## Worklog mode` section. Every document is edited directly there, so there's nothing to sync." |
| Up to date | `git fetch`, then the project repo has no uncommitted changes outside the worklogs and isn't behind its remote | > "The project repo isn't up to date: {what}. Pull, and commit or set aside other changes, so the sync sees every person's latest worklog." |
| The sync owner, for `apply` | `git config user.name` matches the sync owner named in `## Worklog mode` | > "Only {owner} applies a sync, so the main documents keep one author. Run `/sync-worklogs {CODE} check` to see the conflicts without writing anything." |
| Something to sync | At least one worklog has an entry | > "Every worklog is empty: nothing to sync." |
| The hold, for `apply` | `worklogs/hold.md` is empty or names the sync owner | > "{PERSON} holds the hold since {WHEN} ({WHY}), so they may be editing the main documents now. Ask them to release it (commit, push, empty `worklogs/hold.md`), then run the sync." |

On any failure, show the message and wait for PO response.

For `apply`, if the hold is free, take it before reading: write the sync owner, the time and "sync" into `worklogs/hold.md`, and ask:

> "I've taken the hold for the sync. Shall I commit and push `worklogs/hold.md` now, so nobody edits the main documents meanwhile?"

Wait for PO response. Commit and push only that file, and only if the PO says so.

Edits made directly by whoever held the hold since the last sync are part of the main documents: the stale-base check (Step 3, check 5) compares every entry against them.

---

## Step 2 — Read files

1. `projects/{CODE}/CLAUDE.md`: the `## Worklog mode` section (people, files, prefixes, the sync owner, the decision log's path and numbering), and the project's rules for each main document.
2. `projects/TEMPLATE/worklogs/worklog.template.md`: the entry shape and the closed list of kinds (its "When reading this file" block addresses this command).
3. **Every worklog**, in full: header and every entry.
4. **Each target, in parts:** for every entry, the section it names, as it is now. For a plan, also its header, Document Control and Status; for the decision log, its last number and the rows any Decision entry touches.
5. **What moved since each entry was written:** `git log` on each target since the commit in the entry's "Based on", and the diff of the named section.
6. **For each Built entry:** the commit it names, in the code repo the plan names (`git cat-file -e`), and the checkpoint's current Status and Depends on in the plan.
7. The build index (`build/index.md` or the project's equivalent) and the decision log's status section (for a handoff, its "Start here"), which the sync refreshes.

Don't read whole documents an entry doesn't target.

---

## Step 3 — Check every entry

Give each entry one result, and gather the reasons:

- **Applies:** well formed, its target unchanged since "Based on" (or changed elsewhere in the document only), and nothing below applies.
- **Needs the PO:** it changes an Approved document (a plan's change after approval is recorded in its Document Control and needs the PO's say), or it is a Question.
- **Conflict:** any of the checks below.
- **Back to the author:** it can't be applied as written: no target, no base, a body that doesn't give the exact text, a kind outside the list. Never guessed at.

The checks, every one run on every entry:

1. **Same section, two people:** two entries from different worklogs change the same section of the same document.
2. **Same question, two answers:** two Decision entries, from any worklogs, answer the same question differently, or one entry's answer makes another's untrue. Read for meaning, not wording.
3. **Against what is settled:** an entry contradicts a numbered decision, an approved design, or the architecture note, without saying it replaces it.
4. **Against built work:** an entry changes something another Built entry, or a Built block already in a plan, relies on (an endpoint, a table, a rule, a text). Name the code that would have to change.
5. **Stale base:** the target section changed since the entry's "Based on". Say whether the entry still makes sense on the new text.
6. **Order:** a Built entry whose checkpoint depends on one not yet built, in the plan or in any worklog; a commit that doesn't exist; a checkpoint already Built.
7. **Dangling reference:** an entry depends on another entry (by ID) that is back with its author or in conflict.

Urgent entries are checked and listed first.

---

## Step 4 — Ask the PO, at once

In one message: every conflict, every entry needing the PO, and every entry going back to its author, each numbered with a recommendation; then what applies cleanly, grouped by document, for information.

> "Sync of {CODE}: {N} entries from {people}.
>
> **Conflicts:**
> 1. [Z-3 and A-5, `build/group-03-staff-and-roles.md` › A15: Z-3 answers '…', A-5 answers '…'. A-5's checkpoint B03.1.2 is built on its answer.] My recommendation: [keep A-5, and turn Z-3 into a Change for the next chunk].
>
> **Needs your decision:**
> 2. [A-7 changes the approved plan's B03.2.2 Form block: … ] My recommendation: [approve; Document Control row 1.1].
>
> **Back to the author:**
> 3. [A-9: no exact text to replace.] It stays in Ali's worklog for him to fix.
>
> **Applies cleanly:** [N] entries
> - `handoff.md`: decisions Z-1, A-2 (become 190, 191)
> - `build/group-03-staff-and-roles.md`: Built B03.1.1, B03.1.2 (A-1, A-3); status Approved (Z-2)
> - `build/index.md`: recount
>
> Answer by number, or say 'yes to all'. Where a conflict touches another person's work, they should be here, or agree to the answer first."

Wait for PO response.

In `check` mode, show the same message without the last line, then go straight to Step 7's report: write nothing.

A resolution can turn an entry into a different one (a Decision into a Change, two entries into one): note it, and apply the result. An answer that opens a new question is asked as its own numbered item.

Then confirm the plan:

> "Does this look right? I'll apply {N} entries to {M} documents, number {K} decisions from {first} to {last}, record the sync as decision {last + 1}, and empty the applied entries from each worklog. Say what to change if not."

Wait for PO response. Do not write any files until confirmed.

---

## Step 5 — Apply

In this order, so every later step can refer to what earlier ones wrote:

1. **Decisions:** number them in worklog order, oldest first across people, from the decision log's next number. Write each in the log's own format, with "{Person}, {date}, through the sync of {today}" as how it was agreed. Keep a map of provisional ID → number.
2. **Rewrite references:** in every entry still to apply, replace each provisional ID with its number.
3. **New files:** add each to the index it names.
4. **Status:** plans Approved, units Closed, chunks accepted, in each document's own place (the plan's header and B0, the build index).
5. **Built:** each Built block directly above its checkpoint's `**Status:**` line, and that line set to Built; in checkpoint order.
6. **Changes:** each exact edit. A change to an Approved document adds a Document Control row naming the entry, the PO's approval and the date.
7. **Findings:** into the target unit's findings section (a plan's C3), or the document named.
8. **Recount:** the build index (checkpoints built per unit, statuses), and the decision log's status section ("Start here": where things stand and what comes next).
9. **Record the sync:** one more decision in the log: the date, who took part, the map of provisional IDs to numbers, each conflict and how it was resolved, the entries returned to their authors.

Then **empty each worklog**: remove exactly the entries applied or turned into others, keep any added after Step 1's pull and any returned to its author, set "Last sync" to today (the commit follows in Step 6), and leave "Next ID" as it is.

Never apply an entry that is in conflict or returned. Never edit text outside an entry's target.

---

## Step 6 — Update cross-references

1. The decision log's own index or status lines, if it keeps them apart from its table.
2. Every plan whose status changed: its row in the build index.
3. Any document whose links an applied change moved (a renamed section, a new file): the links pointing to it, as real markdown links (C-043).

Empty `worklogs/hold.md` (unless the PO says they keep the hold to go on working directly). Commit nothing. Propose the commit:

> "The sync is applied and the hold released. Proposed commit: `Sync worklogs {date}: {N} entries, decisions {first} to {last}`. Commit and push it, then each person pulls before writing to their worklog again."

Wait for PO response. Commit and push only if the PO says so.

---

## Step 7 — Report

```
WORKLOGS SYNCED — {PROJECT_CODE}
─────────────────────────────────────────
Mode:          {apply | check}
Entries:       {N} read from {people}
Applied:       {N} ({by kind})
Decisions:     {first} to {last}; the sync recorded as {number}
Conflicts:     {N}, resolved: {list, or none}
Returned:      {entry IDs and authors, or none}
Documents:     {each document changed}
Worklogs:      emptied of {N}; {N} kept
─────────────────────────────────────────
Next: commit and push, then every person pulls. The next sync is at
the next chunk's acceptance, or sooner for an Urgent entry:
/sync-worklogs {PROJECT_CODE}
```

In `check` mode, "Applied" reads "Would apply", "Documents" lists what would change, and nothing is written.
