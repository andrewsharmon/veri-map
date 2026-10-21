# veri-map

A workspace where LLM agents explore many research paths, driving external
experimental instruments, without stepping on each other or redoing dead
ends. **You run it entirely from Obsidian**: the repo is one vault. Agents do
the work when you type `/go` in Claude.

The scaffold is public; your research is not. Everything you and your agents
produce lives in `lab/`, a separate private git repo that the scaffold ignores
(see [Public scaffold, private lab](#public-scaffold-private-lab)).

## Setup

```bash
git clone <this scaffold> veri-map && cd veri-map
scripts/vm init                                     # creates lab/ as a private git repo, enables hooks
scripts/vm init --remote git@github.com:<you>/<private-lab>.git   # optional: back the lab up privately
```

Then open the `veri-map` folder in Obsidian (**Open folder as vault**).

## Layout

```
veri-map/                     PUBLIC scaffold (this repo)
├── Research.base             live, editable tables: needs attention, all nodes, findings, runs
├── CLAUDE.md                 agent protocol (AGENTS.md points here)
├── .claude/skills/go/        the /go command
├── _meta/
│   ├── conventions.md        field formats, statuses, naming
│   ├── templates/            DIRECTIVE, NOTEBOOK, FINDINGS, entry, run, INSTRUMENT
│   └── lab-skeleton/         what `vm init` copies into a new lab/
├── scripts/vm                CLI the agents use (Python 3 stdlib)
└── lab/                      PRIVATE: its own git repo, ignored by the scaffold
    ├── INDEX.md              plain map of every question (refreshed by agents)
    ├── instruments/<name>/   your registered instruments (INSTRUMENT.md each)
    └── questions/
        ├── cache-tuning/     ← one question = one folder (a "root node")
        │   ├── DIRECTIVE.md
        │   ├── NOTEBOOK.md
        │   ├── FINDINGS.md
        │   ├── baseline/     ← sub-path (node)
        │   │   ├── DIRECTIVE.md, NOTEBOOK.md, FINDINGS.md
        │   │   └── runs/20261020-1432-sweep/run.md
        │   └── cold-start/ …
        └── tokenizer-drift/  ← an unrelated question
```

- **Question**: a folder directly under `lab/questions/`. Questions are
  independent; agents working on one never touch another.
- **Node**: any folder inside a question, including the question folder
  itself. The folder tree *is* the research tree: each sub-folder is a narrower
  sub-path of its parent. A node's name is its path under `lab/questions/`,
  e.g. `cache-tuning/baseline`.

Every node holds three notes:

| Note | Purpose | Who edits |
|---|---|---|
| `DIRECTIVE.md` | The node's `question` (a property), what to do (*In scope*), what not to do (*Restricted*), *Done when*, *Budget*, and its sub-paths. | You steer here. Agents may only *tighten* it. |
| `NOTEBOOK.md` | Lab notebook: dated, append-only entries of what was tried and seen. | Agents, plus your notes to them. Never rewritten. |
| `FINDINGS.md` | The current answer, with evidence and negative results. | Agents. |

## Working from Obsidian

Open **Research** from the bookmarks. Nothing below needs a terminal.

| You want to… | Do this in Obsidian |
|---|---|
| **Start a question** | In `lab/questions/`, create a new note. Give it a short name (it becomes the folder name), make the first line the question, and optionally add `## In scope`, `## Restricted`, `## Done when`, `## Budget` sections. |
| **Add a sub-path** | Same, but create the note inside the parent node's folder. (With a node's note open, ⌘N creates the note in that folder.) |
| **Tell an agent what to do next** | Type into the node's `flag` property, either in the **Research › All nodes** table or in the note's Properties panel. |
| **Change direction** | Edit the directive: *In scope*, *Restricted*, *Done when*, *Budget*, or the `question` property. |
| **Leave a note on a node** | At the end of its `NOTEBOOK.md`, run *Templates: Insert template → entry* and write your note. |
| **Unblock, reopen or stop a node** | Set its `status` property to `open` or `abandoned`. |
| **See what needs you** | **Research › Needs attention**: flags, blocked nodes, nodes with no question. |

Then type **`/go`** in Claude (desktop Code tab or CLI) with this repo open,
or `/go cache-tuning` to stay in one question.

### What `/go` does

The [go skill](.claude/skills/go/SKILL.md) runs `scripts/vm next`, which:

1. **Syncs** what you made: each new note becomes a node folder with all
   three files and is listed in its parent's *Sub-paths*. Folder names are
   turned into slugs, and a folder that has only a note uses it as its
   directive. These changes are committed to the lab.
2. **Ranks** where to pick up: flagged nodes, then blocked nodes whose
   directive you changed, then nodes where your note is the last notebook
   entry, then directives you edited since an agent last looked, then
   abandoned claims, parents ready to write up their children, and ordinary
   open work.

The agent claims the top node, logs your flag, note or edit in its notebook,
works under [CLAUDE.md](CLAUDE.md), releases the node (which clears the flag),
refreshes `lab/INDEX.md`, and reports back.

Edits are detected by a fingerprint (`seen`) of each directive's steering
sections, stamped whenever an agent claims or releases the node. Your edits
are picked up whether or not they are committed. Changes to *Sub-paths* and
*Closed paths* don't count, since agents maintain those.

### Vault settings (committed in `.obsidian/`)

- **Research.base**: core Bases plugin, no community plugins needed. Its
  tables are editable, so `flag` and `status` can be changed in place.
- Property types for every field, so `flag`, `question` and `status` show as
  plain text fields.
- Relative Markdown links, which also work on GitHub and for agents.
- Core Templates plugin pointed at `_meta/templates/`.
- Graph colours: directives red, notebooks blue, findings green, runs grey;
  only `lab/questions/` is shown, so each question appears as its own tree.
- `scripts/` and `_meta/lab-skeleton/` hidden from the file explorer.

## Public scaffold, private lab

| | Scaffold (this repo) | Lab (`lab/`) |
|---|---|---|
| Contains | protocol, templates, `vm`, Obsidian config, `/go` | questions, notebooks, findings, runs, your instruments, `INDEX.md` |
| Git | this repo; publish it anywhere | its own repo, created by `vm init`; give it a **private** remote |
| Who commits | you, when you change the scaffold | `vm` and agents (`vm claim`, `vm commit`, `/go`) |

How the two stay apart:

- The scaffold's `.gitignore` ignores `/lab/`, so `git add -A` here can't pick
  up research.
- The scaffold's pre-commit hook refuses any commit containing a `lab/` path,
  even a force-added one.
- `vm` only ever commits inside `lab/`. Agents commit with `vm commit`, never
  with raw `git` in the scaffold.
- Instrument runs record both commits: the lab's and the scaffold version
  they ran with.

Common tasks:

```bash
git pull                                    # update the scaffold; lab/ is untouched
git -C lab push                             # back up your research to its private remote
git clone <scaffold> veri-map && git clone <private-lab> veri-map/lab && veri-map/scripts/vm init
                                            # same setup on another machine
```

Before publishing, check what the scaffold contains with `git ls-files`.
Nothing under `lab/` should ever appear.

## How non-redundant work is enforced

1. **Restrictions inherit.** A node's effective rules are its own *Restricted*
   list plus every ancestor's. Agents read them all via `scripts/vm context`.
2. **Siblings declare scope.** A parent's *Sub-paths* table gives each child a
   one-line scope; a new child must not overlap one.
3. **Claims.** A node being worked has `status: active` and `claimed_by`;
   other agents skip it.
4. **Closed paths are recorded** in the parent, and negative results in
   `FINDINGS.md`.
5. **History is append-only.** The lab's pre-commit hook rejects edits to past
   notebook entries and warns when a restriction is removed.

## Instruments

Register external experimental tools under `lab/instruments/<name>/`, each
with an `INSTRUMENT.md` (template in `_meta/templates/`) covering how to invoke
it, its costs and its outputs. Each use is recorded in the node's
`runs/<timestamp>-<slug>/run.md`; large raw output goes in that run's `raw/`
folder, which git ignores.

## CLI (agents, or you in a terminal)

```bash
scripts/vm init [--remote <url>]     # create lab/ if missing; enable hooks in both repos
scripts/vm next                      # sync Obsidian edits, then rank where to pick up
scripts/vm context <node>            # everything an agent reads before working
scripts/vm claim|release <node> <agent>
scripts/vm log <node> <agent> "<title>"
scripts/vm run <node> <slug> --agent <id> --instrument <name>
scripts/vm commit <node> "<verb>: <msg>"
scripts/vm path <parent> <slug> "<question>"
scripts/vm question <slug> "<question>"
scripts/vm flag <node> "<text>"      # same as the flag property
scripts/vm tree | open | questions | index | sync | check
```

Several agents can share one checkout, since each commits only its own
node. `NOTEBOOK.md` files use git's `union` merge driver, so appends made on
parallel lab branches merge cleanly.
