# Instruments

External experimental tools that agents drive: benchmarks, simulators, lab
APIs, eval harnesses, anything that produces evidence.

Register each one in its own folder:

```
instruments/
└── <name>/
    ├── INSTRUMENT.md     how to invoke it, costs, outputs, pitfalls (from _meta/templates/INSTRUMENT.md)
    └── …                 wrapper scripts or config, if any
```

Rules:

- An agent uses only instruments registered here, unless a directive says
  otherwise.
- Every use is recorded as a run in the node that needed it:
  `scripts/vm run <node> <slug> --agent <id> --instrument <name>`.
- If an instrument costs money or quota, say so under *Limits and costs*, and
  set a *Budget* in the directives that use it.
- Changes to an instrument go in its `INSTRUMENT.md` with the date, so old runs
  can be read against the version that produced them.
