---
name: go
description: Find where to pick up research work in this veri-map repo and do it. Checks human flags, directives edited since an agent last looked, stale claims and unfinished nodes, then claims the best node and works it under the CLAUDE.md protocol. Use when the user says "go", "go work", "pick up", "continue the research", or names a question to work on.
argument-hint: "[question-or-node]  (optional: limit to one question or subtree)"
---

# Go work

Pick the most useful node and work it, following [CLAUDE.md](../../../CLAUDE.md)
exactly. Scope from the user, if any: `$ARGUMENTS`. An empty scope means all
questions.

## 1. Choose an agent id

Use `claude-<MMDD-HHMM>` from the current time (e.g. `claude-1020-1932`) and
keep it for the whole session.

## 2. Find the work

```bash
scripts/vm next $ARGUMENTS --agent <id>
```

It ranks candidates, highest first:

1. **flagged** – a human set `flag:` on the node (in Obsidian, or with `vm flag`).
   The flag text is your instruction for this session.
2. **blocked, directive updated** – a human probably answered a `**Proposal:**`.
3. **directive updated** – a human changed the scope, restrictions, exit
   criteria or budget since an agent last claimed or released the node.
4. **stale claim** – an agent stopped mid-work over 24h ago; resume it.
5. **all sub-paths closed** – write up the parent's findings from its children.
6. **in progress / not started / plan sub-paths** – ordinary open work, deepest
   first.

Take the `PICK UP:` node. Choose a lower one only if the top one is outside
the user's scope or you cannot do it (e.g. it needs an instrument that isn't
registered). If it prints `NOTHING TO PICK UP`, tell the user, list anything
under *Waiting on a human*, and stop. Do not invent work.

## 3. Orient

```bash
scripts/vm context <node>
```

Read all of it. If it has a **From the human** section, deal with that first:

- A flag is the instruction for this session. If it's about a different node,
  e.g. a flag on a question root naming a sub-path, work that node instead and
  mention the redirect in both notebooks.
- If the directive changed, read the diff it points to (`git diff` /
  `git log -p`) so you know exactly what's new.

If the node is `concluded` or `abandoned` and flagged, the flag reopens it:
claim with `--force`.

## 4. Claim and work

```bash
scripts/vm claim <node> <id>
```

This logs the flag and the directive change to the notebook and commits.
Then work under the protocol: log before and after each step, record
instrument runs with `vm run`, stay within the effective restrictions and
budgets, spawn sub-paths with `vm path` when the node splits, and commit only
your node's paths.

Keep going until one of these happens:

- the node's *Done when* is met,
- a budget is reached,
- you need a human decision (write a `**Proposal:**` line), or
- you have finished one solid unit of progress and the next step is a new
  line of work best done in a fresh session.

## 5. Wrap up

1. Update `FINDINGS.md`. If the node is closing, add the roll-up to the
   parent's notebook and *From sub-paths*.
2. Release with the right status. This clears the flag; pass `--keep-flag`
   only if you did not address it.
   ```bash
   scripts/vm release <node> <id> --status <open|blocked|concluded|abandoned> [--summary "..."]
   ```
3. Commit your paths. Run `scripts/vm index` and commit `INDEX.md`
   (`git commit -- INDEX.md`) so Obsidian's map stays current.
4. Report to the user in a few lines: which node and why it was picked, what
   you did, what was found, the node's status now, and what `vm next` would
   pick next. Put anything that needs their decision first.

Work one node per `/go` unless the user asked for more.
