# veri-map

A workspace where LLM agents explore many research paths, driving external
experimental instruments, without stepping on each other or redoing dead
ends. **You run it entirely from Obsidian**: the repo is one vault. Agents do
the work when you type `/go` in Claude.

## Layout

```
veri-map/
├── Research.base             live, editable tables: needs attention, all nodes, findings, runs
├── INDEX.md                  plain map of every question (refreshed by agents)
├── CLAUDE.md                 agent protocol (AGENTS.md points here)
├── .claude/skills/go/        the /go command
├── _meta/
│   ├── conventions.md        field formats, statuses, naming
│   └── templates/            DIRECTIVE, NOTEBOOK, FINDINGS, entry, run, INSTRUMENT
├── instruments/<name>/       registered external instruments (INSTRUMENT.md each)
├── questions/
│   ├── cache-tuning/         ← one question = one folder (a "root node")
│   │   ├── DIRECTIVE.md
│   │   ├── NOTEBOOK.md
│   │   ├── FINDINGS.md
│   │   ├── baseline/         ← sub-path (node)
│   │   │   ├── DIRECTIVE.md, NOTEBOOK.md, FINDINGS.md
│   │   │   └── runs/20261020-1432-sweep/run.md
│   │   └── cold-start/ …
│   └── tokenizer-drift/      ← an unrelated question
└── scripts/vm                CLI the agents use (Python 3 stdlib)
```

- **Question**: a folder directly under `questions/`. Questions are
  independent; agents working on one never touch another.
- **Node**: any folder inside a question, including the question folder
  itself. The folder tree *is* the research tree: each sub-folder is a narrower
  sub-path of its parent.

Every node holds three notes:

| Note | Purpose | Who edits |
|---|---|---|
| `DIRECTIVE.md` | The node's `question` (a property), what to do (*In scope*), what not to do (*Restricted*), *Done when*, *Budget*, and its sub-paths. | You steer here. Agents may only *tighten* it. |
| `NOTEBOOK.md` | Lab notebook: dated, append-only entries of what was tried and seen. | Agents, plus your notes to them. Never rewritten. |
| `FINDINGS.md` | The current answer, with evidence and negative results. | Agents. |

## Working from Obsidian

Open this folder as a vault (**Open folder as vault**), then open
**Research** from the bookmarks. Nothing below needs a terminal.

| You want to… | Do this in Obsidian |
|---|---|
| **Start a question** | In `questions/`, create a new note. Give it a short name (it becomes the folder name), make the first line the question, and optionally add `## In scope`, `## Restricted`, `## Done when`, `## Budget` sections. |
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
   directive. These changes are committed.
2. **Ranks** where to pick up: flagged nodes, then blocked nodes whose
   directive you changed, then nodes where your note is the last notebook
   entry, then directives you edited since an agent last looked, then
   abandoned claims, parents ready to write up their children, and ordinary
   open work.

The agent claims the top node, logs your flag, note or edit in its notebook,
works under [CLAUDE.md](CLAUDE.md), releases the node (which clears the flag),
refreshes `INDEX.md`, and reports back.

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
  only `questions/` is shown, so each question appears as its own tree.
- `scripts/` hidden from the file explorer.

Your edits sit uncommitted until an agent's `/go` commits them along with its
work. If you want every edit committed as you go, add the community
*Obsidian Git* plugin; that's optional.

## How non-redundant work is enforced

1. **Restrictions inherit.** A node's effective rules are its own *Restricted*
   list plus every ancestor's. Agents read them all via `scripts/vm context`.
2. **Siblings declare scope.** A parent's *Sub-paths* table gives each child a
   one-line scope; a new child must not overlap one.
3. **Claims.** A node being worked has `status: active` and `claimed_by`;
   other agents skip it.
4. **Closed paths are recorded** in the parent, and negative results in
   `FINDINGS.md`.
5. **History is append-only.** A pre-commit hook rejects edits to past
   notebook entries and warns when a restriction is removed.

## Instruments

External experimental tools are registered under [instruments/](instruments/),
each with an `INSTRUMENT.md` (how to invoke it, costs, outputs). Each use is
recorded in the node's `runs/<timestamp>-<slug>/run.md`; large raw output
goes in that run's `raw/` folder, which git ignores.

## CLI (agents, or you in a terminal)

```bash
scripts/vm init                      # once per clone: enables the git hook
scripts/vm next                      # sync Obsidian edits, then rank where to pick up
scripts/vm context <node>            # everything an agent reads before working
scripts/vm claim|release <node> <agent>
scripts/vm log <node> <agent> "<title>"
scripts/vm run <node> <slug> --agent <id> --instrument <name>
scripts/vm path <parent> <slug> "<question>"
scripts/vm question <slug> "<question>"
scripts/vm flag <node> "<text>"      # same as the flag property
scripts/vm tree | open | questions | index | sync | check
```

Everything lives on `main`; git is history, not structure. Agents commit only
their own paths, so several can share one checkout. `NOTEBOOK.md` files use
git's `union` merge driver, so appends made on parallel branches merge cleanly.
