# veri-map

A scaffold for letting LLM agents explore many research paths, driving
external experimental instruments, without stepping on each other or redoing
dead ends. Humans steer and read it through [Obsidian](https://obsidian.md):
the whole repo is one vault, so every question is visible at once.

## Layout

```
veri-map/
├── CLAUDE.md                 agent protocol (AGENTS.md points here)
├── .claude/skills/go/        the /go command: find where to pick up, then work it
├── INDEX.md                  generated map of every question and node (scripts/vm index)
├── _meta/
│   ├── conventions.md        field formats, statuses, naming
│   ├── dashboard.md          live status tables (needs the Dataview plugin)
│   └── templates/            DIRECTIVE, NOTEBOOK, FINDINGS, entry, run, INSTRUMENT
├── instruments/<name>/       registered external instruments (INSTRUMENT.md each)
├── questions/
│   ├── cache-tuning/         ← one question = one top-level folder (a "root node")
│   │   ├── DIRECTIVE.md
│   │   ├── NOTEBOOK.md
│   │   ├── FINDINGS.md
│   │   ├── baseline/         ← sub-path (node)
│   │   │   ├── DIRECTIVE.md, NOTEBOOK.md, FINDINGS.md
│   │   │   └── runs/20261020-1432-sweep/run.md
│   │   └── cold-start/
│   │       └── …
│   └── tokenizer-drift/      ← an unrelated question
└── scripts/vm                CLI helper (Python 3 stdlib)
```

- **Question** = a folder directly under `questions/`. Questions are
  independent of each other; agents working on one never touch another.
- **Node** = any folder inside a question (including the question folder
  itself). The directory tree *is* the research tree: each sub-folder is a
  narrower sub-path of its parent.

Every node holds the same three files:

| File | Purpose | Edit rule |
|---|---|---|
| `DIRECTIVE.md` | The node's question, what to do (*In scope*), what not to do (*Restricted*), exit criteria, budget, and its sub-paths. | Humans steer here. Agents may only *tighten* it. |
| `NOTEBOOK.md` | Lab notebook: dated, append-only entries of what was tried and seen. | Append only, enforced by a pre-commit hook. |
| `FINDINGS.md` | The current distilled answer, with evidence and negative results. | Rewritten as understanding improves. |

Plus optional `runs/` (instrument records) and child folders (sub-paths).

## How non-redundant work is enforced

1. **Restrictions inherit.** A node's effective rules are the union of its own
   *Restricted* list and every ancestor's. `scripts/vm context <node>` prints
   them all, so an agent cannot miss one.
2. **Siblings declare scope.** A parent's *Sub-paths* table gives each child a
   one-line scope. A new child must not overlap an existing row, and gets the
   siblings' ground added to its own *Restricted* list.
3. **Claims.** A node being worked on has `status: active` and `claimed_by`.
   Other agents pick a different node (`scripts/vm open`).
4. **Closed paths are recorded.** Finished or abandoned children are summarised
   in the parent's *Closed paths*, and their negative results in `FINDINGS.md`.
5. **History is append-only.** The pre-commit hook rejects edits to past
   notebook entries and warns when a restriction is removed.

Agent rules in full: [CLAUDE.md](CLAUDE.md). Field formats:
[_meta/conventions.md](_meta/conventions.md).

## Quick start

```bash
scripts/vm init                     # once per clone: enables the git hooks

# start a question
scripts/vm question cache-tuning "Which cache settings minimise p99 latency on workload W?"

# split it into sub-paths
scripts/vm path questions/cache-tuning baseline "What is a reproducible baseline p99 for W?"

# everything an agent must read before working a node
scripts/vm context questions/cache-tuning/baseline

# claim, take notes, record a run, release
scripts/vm claim   questions/cache-tuning/baseline agent-a
scripts/vm log     questions/cache-tuning/baseline agent-a "First sweep"
scripts/vm run     questions/cache-tuning/baseline sweep --agent agent-a --instrument loadgen
scripts/vm release questions/cache-tuning/baseline agent-a --status concluded

# steer and pick up
scripts/vm flag    questions/cache-tuning/baseline "Re-check with warm cache"
scripts/vm next           # where an agent should pick up (what /go uses)

# overview
scripts/vm questions      # one line per question
scripts/vm tree           # every node with status and claim
scripts/vm open           # nodes free to claim
scripts/vm index          # rewrite INDEX.md for Obsidian
scripts/vm check          # lint the tree
```

## Obsidian

Open the repo folder as a vault. The committed `.obsidian/` config:

- uses relative Markdown links, which also work on GitHub and for agents;
- points the core Templates plugin at `_meta/templates/` (use *Insert template →
  entry* to add a notebook entry by hand);
- colours the graph: directives red, notebooks blue, findings green, runs grey,
  with `_meta/` and `scripts/` filtered out, so each question shows as its own
  tree;
- hides `scripts/` from the file explorer.

`INDEX.md` is a plain-Markdown map of everything (regenerate with
`scripts/vm index`). If you install the community **Dataview** plugin,
[_meta/dashboard.md](_meta/dashboard.md) gives live tables of claims and status.

## Steering, then `/go`

You steer from Obsidian. An agent picks up your changes when you type `/go`
in Claude (desktop Code tab or CLI) with this repo open.

- **Flag a node.** In the node's `DIRECTIVE.md`, fill in the `flag` property
  (Obsidian's Properties panel) with what you want done, e.g.
  `flag: re-run the sweep with the warm cache; the cold numbers look off`.
  From a terminal: `scripts/vm flag <node> "…"`. A flag on a question root
  can point at a sub-path.
- **Or just edit a directive**: change *In scope*, add a *Restricted* line,
  rewrite *Done when*, set a *Budget*, or answer a blocked node's
  `**Proposal:**`. Each claim/release stamps a fingerprint (`seen`) of
  those sections, so `vm` notices edits made since an agent last looked,
  committed or not. Edits to *Sub-paths* / *Closed paths* don't count, since
  agents maintain those.
- **Then run `/go`** (optionally `/go cache-tuning` to stay in one question).
  The [go skill](.claude/skills/go/SKILL.md) runs `scripts/vm next`, which
  ranks flagged nodes first, then blocked nodes whose directive changed, then
  edited directives, stale claims, parents ready to synthesise, and ordinary
  open work. The agent claims the top node, works it, releases it (which
  clears the flag), and reports back.

`INDEX.md` opens with a *Needs attention* list (⚑ flagged, ✎ edited,
blocked), and `scripts/vm tree` shows the same markers.

## Instruments

External experimental tools are registered under [instruments/](instruments/).
Each has an `INSTRUMENT.md` saying how to invoke it, what it costs, and what it
outputs. Each use is recorded in the node's `runs/<timestamp>-<slug>/run.md`;
large raw output goes in that run's `raw/` folder, which git ignores.

## Git

Everything lives on `main`; git is history, not structure. Agents commit only
the paths they own (`git commit -- <node>`), so several can share one
checkout. For heavier isolation an agent can work in a worktree on a
short-lived branch and merge back; `NOTEBOOK.md` files use git's `union` merge
driver, so concurrent appends merge without conflicts.
