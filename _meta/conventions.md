# Conventions

## Notes made in Obsidian

`vm sync` (also run by `vm next`, hence `/go`) adopts:

- **A note in `lab/questions/` or in a node folder** (other than the three node
  files) → a new child node. Folder = slug of the note's name. Question = the
  `question` property, else the note's first paragraph, else its name. The
  remaining lead text becomes *Context*; `## In scope`, `## Restricted`,
  `## Done when`, `## Budget`, `## Context` sections carry over. The note is
  deleted once folded into the new `DIRECTIVE.md`.
- **A folder** → renamed to a slug if needed. Missing node files are created,
  and the folder is listed in its parent's *Sub-paths*. A folder with no
  `DIRECTIVE.md` and exactly one note uses that note as its directive.

Notes named `Untitled…` or starting with `_` are left alone. Put notes that
are not requests in `assets/` or a `_`-prefixed folder.

## Names

- Question and node folders: lowercase kebab-case slugs (`cold-start`,
  `p99-baseline`). Keep them short; the path carries the context.
- Folders inside a node that are **not** nodes: `runs/`, `assets/`, and
  anything starting with `_` or `.`. Use `assets/` for figures and notes that
  belong to the node but are not runs.
- Agent ids: any stable short string (`agent-a`, `claude-7f3`, `alice`). Use
  the same id for the whole session.

## Status

Set in `DIRECTIVE.md` frontmatter.

| Status | Meaning |
|---|---|
| `open` | Free to claim. |
| `active` | Claimed; `claimed_by` and `claimed_at` are set. Stale after 24h. |
| `blocked` | Waiting on a human (a proposal, a budget, an instrument). |
| `concluded` | Done criteria met; answer in `FINDINGS.md`. |
| `abandoned` | Stopped without an answer; reason in `FINDINGS.md`. |

Markers in `vm tree`, `vm next` and `INDEX.md`: **⚑** flagged, **✎** directive
edited since an agent last looked (its steering sections no longer match
`seen`).

## Frontmatter

`DIRECTIVE.md`

| Field | Value |
|---|---|
| `type` | `directive` |
| `node` | Node name: path under `lab/questions/`, e.g. `cache-tuning/baseline`. Set by `vm`. |
| `question` | The node's question, one sentence. Canonical; edit it here. |
| `status` | See above |
| `claimed_by` | Agent id, or empty |
| `claimed_at` | `YYYY-MM-DDTHH:MM` local time, or empty |
| `flag` | Human instruction for the next agent, or empty. Set by humans only; cleared by `vm release`. |
| `seen` | Fingerprint of the steering sections when an agent last claimed or released. Set by `vm` only. |
| `created`, `updated` | `YYYY-MM-DD` |

`FINDINGS.md`: `type: findings`, `node`, `confidence` (`none` / `low` /
`medium` / `high`), `updated`.

`NOTEBOOK.md`: `type: notebook`, `node`. Nothing that changes, so the file
stays append-only.

`run.md`: `type: run`, `node`, `run`, `agent`, `instrument`,
`instrument_version`, `commit`, `started`, `outcome` (`ok` / `partial` /
`failed`).

Keep frontmatter values on one line; `scripts/vm` reads `key: value` lines only.

## Directive sections

`Context`, `In scope`, `Restricted`, `Done when`, `Budget`, `Sub-paths`,
`Closed paths` (the question is the `question` property). `scripts/vm` finds them by their `## ` heading, so
keep the names. Guidance lives in `<!-- -->` comments, which Obsidian hides in
reading view.

Restrictions are bullets with an id and a reason:

```
- R1: Tuning the kernel scheduler — out of scope for this question (human, 2026-10-20)
- R2: Eviction policy experiments — covered by sibling `eviction`
```

## Notebook entries

```
## 2026-10-20 14:32 · agent-a · First sweep
```

Heading = date, time, author, title. The author `human` is reserved for the
person steering from Obsidian (the *entry* template uses it); an agent never
writes as `human`. Body fields: *Intent*, *Did*, *Observed*,
*Interpretation*, *Next*, *Refs*. Refer to an entry by its heading text. Use a
`**Proposal:**` line to ask a human to change a directive.

## Runs

`<node>/runs/<YYYYMMDD-HHMM>-<slug>/run.md`, plus small outputs alongside it and
large raw output in `raw/` (git-ignored).

## Commits

All in the lab repo: `[<node>] <verb>: <message>`, e.g.
`[cache-tuning/baseline] log: first sweep` (what `vm commit` writes). Verbs: `question`, `path`,
`claim`, `release`, `log`, `run`, `findings`, `directive`, plus `[sync]` and
`[index]` from `vm`.
