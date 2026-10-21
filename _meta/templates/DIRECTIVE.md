---
type: directive
node:
question:
status: open
flag:
claimed_by:
claimed_at:
seen:
created: {{date:YYYY-MM-DD}}
updated: {{date:YYYY-MM-DD}}
tags: [vm/directive]
---
Parent: [../DIRECTIVE.md](../DIRECTIVE.md) · [NOTEBOOK.md](NOTEBOOK.md) · [FINDINGS.md](FINDINGS.md)

<!-- The `question` property above is this node's question, in one sentence.
     Set `flag` to tell the next agent what to do; `/go` picks flagged nodes first. -->

## Context
<!-- Why this question matters and what is already known. Link the parent's
     findings or the notebook entries that motivated it. -->

## In scope
<!-- Concrete directions to pursue here, most promising first. One bullet each. -->

## Restricted
<!-- Directions NOT to pursue here or anywhere below, each with a reason.
     Ancestors' restrictions also apply (`scripts/vm context` lists them all).
     Agents only ever add restrictions; removing one is a human decision.
     Format:  - R1: <direction> — <reason>
     Reasons: "covered by sibling `x`", "settled, see ../FINDINGS.md", "over budget", … -->

## Done when
<!-- Exit criteria: the evidence that would answer the question, or the point at
     which to give up and record a negative result. -->

## Budget
<!-- Optional limits on instrument use: runs, wall-clock, money. Leave empty if none. -->

## Sub-paths
<!-- One row per child folder; scopes must not overlap. Maintained by agents. -->

| Path | Scope |
|---|---|

## Closed paths
<!-- Children that are concluded or abandoned, so nobody redoes them:
     - `slug` — concluded: one-line result. [findings](slug/FINDINGS.md) -->
