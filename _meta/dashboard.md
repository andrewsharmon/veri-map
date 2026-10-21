# Dashboard

Live tables. Requires the community **Dataview** plugin; without it, see
[INDEX.md](../INDEX.md) or run `scripts/vm tree`.

## Flagged (picked up first by `/go`)

```dataview
TABLE flag AS "Flag", status
FROM "questions"
WHERE type = "directive" AND flag
SORT file.path ASC
```

## Active claims

```dataview
TABLE claimed_by AS "Agent", claimed_at AS "Since"
FROM "questions"
WHERE type = "directive" AND status = "active"
SORT claimed_at ASC
```

## Blocked (needs a human)

```dataview
TABLE updated AS "Updated"
FROM "questions"
WHERE type = "directive" AND status = "blocked"
SORT file.path ASC
```

## Open nodes

```dataview
TABLE updated AS "Updated"
FROM "questions"
WHERE type = "directive" AND status = "open"
SORT file.path ASC
```

## Findings by confidence

```dataview
TABLE confidence, updated
FROM "questions"
WHERE type = "findings" AND confidence != "none"
SORT confidence DESC, updated DESC
```

## Recent runs

```dataview
TABLE instrument, agent, outcome, started
FROM "questions"
WHERE type = "run"
SORT started DESC
LIMIT 20
```
