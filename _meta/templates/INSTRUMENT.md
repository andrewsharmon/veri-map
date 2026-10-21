---
type: instrument
name:
source:
driver:
interface:
activate:
outputs:
status:
last_checked:
cost:
tags: [vm/instrument]
---
<!-- A lab instrument entry is a lightweight pointer. `source` is the folder (or URL)
     that owns the instrument: its driver, API, procedures and verification record.
     Keep here only what a researcher needs to decide whether and how to use it;
     everything deeper stays in the source. `outputs` = what it can drive (a voltage,
     an AFG, RF TX, motion), or "none" for read-only meters. -->

## Gives this lab
<!-- What it can measure or source, in research terms: ranges, resolution, speed. -->

## Use it
<!-- Where the authoritative docs are (source README/API), how to activate the
     environment, and which source rules apply (e.g. that repo's AGENTS.md). -->

## Guards and limits
<!-- Default permissions, what needs explicit human permission, known
     discrepancies or unverified paths, safety class. -->
