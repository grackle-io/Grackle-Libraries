# Machines

One JSON per machine, loaded by Grackle's **Libraries** button. A machine that
has geometry keeps it in a `.3dm` of the same name, whose layer names match the
`parts` entries; a machine with no `.3dm` still works for toolpath validation
and code generation, it just does not appear in preview.

Anything in here is edited **in this repo** — that is the file Grackle reads.
(A copy of the same filename under `Documents/Grackle/Libraries` wins over the
downloaded cache, which is how a shop keeps a local change without losing it to
a re-install.)

## Start and end operations

`startOperations` runs once at the top of a job, before any tool segment;
`endOperations` runs once at the bottom. Both are the *machine's* band — the
hotend heat belongs to the effector, which runs after. See
`docs/start-end-scripts-and-material-handoff.md` in the plugin repo for the
three-layer split.

The two Prusa printers here start the same way, and the order is the point:

```
bed to temperature (wait)   ← the mesh is measured hot, as the slicer does it
home                        ← MK3 needs G28 W: a plain G28 re-applies the
                              stored mesh the next line is about to replace
mesh bed levelling
```

The bed heat carries **no number**. A temperature is a fact about the material,
not about the printer — `"wait": true` with no `target` resolves from the job's
`Material` at emit time, so one machine file serves PLA and PETG instead of
needing a copy of itself per filament. If no material is wired, Generate Code
says so rather than guessing.

### The mesh line must match what Machine Control sends

| File | Firmware | Home + mesh |
|---|---|---|
| `Prusa-i3-MK3.json` | `Prusa-Firmware` (8-bit) | `G28 W` then `G80` |
| `Prusa-MK4.json` | `Prusa-Firmware-Buddy` (32-bit) | `G28` then `G29` |

Grackle's **Home** button spells these from the board's own `M115` answer, so a
file that disagrees with the table above will level a bed one way from the job
header and another way from the button. If you add a Prusa variant, copy the row
that matches its firmware, not the one that matches its name.

## What is deliberately absent from the Prusa files

- **`M862.3 P "MK4"`** (printer-model check). Worth adding by hand once you know
  which variant you have — it turns "wrong printer" into a clean stop instead of
  a crash — but it is variant-exact, and the firmware string does not
  distinguish an MK4 from an MK4S, so a shipped line would hard-stop half the
  machines it was meant to protect.
- **`M115 U<version>`** (firmware-version check). PrusaSlicer emits it; do not
  copy it into a hand-authored header. If the version you declare is newer than
  the one running, the printer pauses for 30 seconds asking you to upgrade, and
  a host has no way to abort that.
- **`G21` / `G90` / `M82` / `M83`.** The emitter derives these from the same
  `EmitOptions` flag that decides how E is written, so a copy pasted from a
  slicer header can only disagree with the body.

## Numbers you may want to change

Travel limits are the published build volumes (MK3S+ 250 × 210 × 210, MK4
250 × 210 × 220). Feed caps are deliberately conservative — 12 000 mm/min on
X/Y, 720 on Z — and are used to *validate* a toolpath, not to drive it. Raise
them if you run an MK4 with input shaping and want the envelope checked against
what the machine can really do.

`Prusa-MK4.json` has no `.3dm` and no `parts`: it carries axis travel, feed
caps, and the start/end operations only. Add a model later if you want the
machine in preview.
