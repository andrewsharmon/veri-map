# Conventions

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

## Frontmatter

`DIRECTIVE.md`

| Field | Value |
|---|---|
| `type` | `directive` |
| `node` | Repo-relative path, e.g. `questions/cache-tuning/baseline` |
| `status` | See above |
| `claimed_by` | Agent id, or empty |
| `claimed_at` | `YYYY-MM-DDTHH:MM` local time, or empty |
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

`Question`, `Context`, `In scope`, `Restricted`, `Done when`, `Budget`,
`Sub-paths`, `Closed paths`. `scripts/vm` finds them by their `## ` heading, so
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

Heading = date, time, agent, title. Body fields: *Intent*, *Did*, *Observed*,
*Interpretation*, *Next*, *Refs*. Refer to an entry by its heading text. Use a
`**Proposal:**` line to ask a human to change a directive.

## Runs

`<node>/runs/<YYYYMMDD-HHMM>-<slug>/run.md`, plus small outputs alongside it and
large raw output in `raw/` (git-ignored).

## Commits

`[<node path>] <verb>: <message>`, e.g.
`[questions/cache-tuning/baseline] log: first sweep`. Verbs: `question`, `path`,
`claim`, `release`, `log`, `run`, `findings`, `directive`.
