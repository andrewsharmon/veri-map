# Agent protocol

You are working in a veri-map research repository. Each folder directly under
`questions/` is one research **question**; every folder inside a question is a
**node** (a research path) with a `DIRECTIVE.md`, `NOTEBOOK.md` and
`FINDINGS.md`. Read [README.md](README.md) for the model and
[_meta/conventions.md](_meta/conventions.md) for field formats.

Tooling: `scripts/vm` (Python 3 stdlib, no install). Run `scripts/vm -h`.

## Scope

- You are assigned a question, and usually a node within it. If you were only
  given a question, run `scripts/vm open questions/<slug>` and pick a node, or
  ask.
- **Never touch another question's folder.** Questions are independent.
- Outside `questions/`, change nothing (templates, scripts, this file,
  `.obsidian/`) unless a human asks you to.

## Before you work on a node

1. Run `scripts/vm context <node>` and read all of it. It prints, root-first,
   every ancestor's question, the accumulated **Restricted** list, sibling
   scopes, closed paths, and this node's directive, findings and recent
   notebook entries.
2. The effective restrictions are the **union** of every ancestor's
   *Restricted* list and this node's. They are hard limits. If you think one is
   wrong, say so in the notebook and stop that line of work; do not ignore it.
3. If the node is `active` and claimed by someone else, pick another node.
   (`vm claim` refuses unless the claim is stale, over 24h old.)
4. `scripts/vm claim <node> <your-id>`. It commits the claim.

## While working

- **Log as you go.** Each meaningful step (a hypothesis, an instrument run, a
  surprising observation, a decision to stop) gets a notebook entry:
  `scripts/vm log <node> <your-id> "<title>"`, then fill in the skeleton it
  appended. Write *Intent* before running an instrument and the rest after.
- **Notebooks are append-only.** Never edit or delete an earlier entry,
  including your own. To correct one, add a new entry that names the heading
  it corrects. The pre-commit hook rejects rewrites.
- **Stay inside your node.** You may write in your node and its descendants.
  Outside it you may only: append to the parent's `NOTEBOOK.md`, and add rows
  to the parent's *Sub-paths* / lines to its *Closed paths*.
- **Instrument runs:** `scripts/vm run <node> <slug> --agent <id> --instrument <name>`
  creates `runs/<YYYYMMDD-HHMM>-<slug>/run.md`. Fill in the exact command,
  instrument version and parameters. Put large raw output in that run's `raw/`
  folder (git-ignored); commit summaries, not gigabytes. Check
  `instruments/<name>/INSTRUMENT.md` for costs and limits first.
- **Respect budgets.** If any directive in the chain sets a *Budget*, stop when
  you reach it and log that you did.
- **Commit only your paths:** `git add <node> && git commit -m "[<node>] <verb>: <msg>" -- <node>`.
  Other agents may share this checkout; never `git add -A` or commit their
  files. Commit after every few entries.

## Spawning a sub-path

Split a node when it holds two or more questions that can be pursued
independently. Before creating one:

1. Check it is not covered by an existing sibling's scope, not listed in
   *Closed paths*, and not forbidden by any inherited restriction.
2. `scripts/vm path <parent> <slug> "<question>"`. This creates the folder from
   templates, adds a row to the parent's *Sub-paths* table, and logs it in the
   parent's notebook.
3. Fill in the child's *In scope*, *Restricted* and *Done when*. The child's
   *Restricted* must name the ground its siblings cover, so the two cannot
   drift into each other. Restrictions may be added, never removed.

## Finishing or stopping

1. Rewrite `FINDINGS.md` to reflect what is now known, including negative
   results. Every claim points to the notebook entry or run that supports it.
2. Append a short roll-up entry to the **parent's** notebook: what this node
   concluded and a link to its `FINDINGS.md`; add a line under the parent
   findings' *From sub-paths*.
3. `scripts/vm release <node> <your-id> --status <open|blocked|concluded|abandoned>`.
   For `concluded` / `abandoned` it also adds a line to the parent's
   *Closed paths*; edit that line to state the result.
4. Commit.

Leaving mid-task is fine: log where you stopped and what you would do next,
then release with `--status open`.

## What you must not do

- Loosen, delete or reword a restriction, or change a directive's *Question*.
  Those are human decisions. Propose a change in the notebook with a
  `**Proposal:**` line, and release with `--status blocked` if you cannot
  continue without it.
- Edit another node's notebook entries or findings (except the parent roll-ups
  above).
- Redo work recorded under *Closed paths* or *Negative results* unless a
  directive tells you to.
- Push, force-push, rewrite history, or bypass hooks (`--no-verify`) unless a
  human asks.
