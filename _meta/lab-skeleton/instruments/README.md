# Instruments

Instruments the agents may use. Each entry is a **lightweight pointer**: the
folder or repo named in its `source` property owns the driver, API, procedures
and verification record. The entry here keeps only what a researcher needs:
what it gives the lab, how to start, and its guards and limits.

```
lab/instruments/
└── <name>/
    └── INSTRUMENT.md     template: the scaffold's _meta/templates/INSTRUMENT.md
```

- `scripts/vm instruments` lists every entry, warns when a `source` is missing
  on this machine, and shows the source repo's commit.
- `scripts/vm run <node> <slug> --instrument <name>` records that commit in the
  run's `instrument_version`, so results can be traced to the driver version
  that produced them.
- Deeper use (commands, settings, wiring, verification) happens in the source,
  under that repo's own rules. Update facts there first, then refresh the
  summary here.
- Anything that drives an output (supply, AFG, RF transmit, motion) needs
  explicit human permission, whatever the source driver allows.
