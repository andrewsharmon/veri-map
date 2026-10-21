# Lab

Your private research. This folder is its own git repository, separate from
the public veri-map scaffold around it, which ignores it completely.

- `questions/` — one folder per research question (see the scaffold's README)
- `instruments/` — your registered instruments
- `INDEX.md` — map of every question, refreshed by agents

Keep this repo's remote **private**:

```bash
scripts/vm init --remote git@github.com:<you>/<private-lab-repo>.git
git -C lab push -u origin main
```
