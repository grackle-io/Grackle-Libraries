# Post-Processors

One JSON per dialect, loaded by Grackle's **Libraries** button. A post owns the
*vocabulary*: the command templates, the axis map, the comment format, and the
program-start/end boilerplate the controller expects. It does **not** own the
machine's bring-up, and it does not own any number that belongs to the material.

## The three layers of a header

A Grackle header is composed at emit time from three independently-owned bands,
in this order (see `docs/start-end-scripts-and-material-handoff.md` in the
plugin repo):

```
derived mode preamble     ← the emitter: G21, G90, M82/M83, G92 E0
post startScript          ← this folder: firmware limits and config
machine startOperations   ← Machines/: bed heat, home, mesh
effector startOperations  ← Effectors/: hotend heat, spin-up
  … the job …
machine endOperations     ← Machines/: heaters and fan off
post endScript            ← this folder: park, drain, steppers off
```

Two consequences worth having in front of you when authoring a post:

- **Never write `G21`, `G90`, `M82`, `M83` or `G92 E0` into a script.** Declare
  them as the `unitsMillimeters` / `absoluteCoordinates` / `extrusionAbsolute` /
  `extrusionRelative` / `resetExtrusion` templates instead and the emitter emits
  them, derived from the same flag that decides how `E` is written. A script
  copy can only ever disagree with the body — that mismatch caused a real
  over-extrusion, which is why the preamble is derived rather than authored.
- **A move belongs in a script, not in an operation band.** Fragment operations
  run outside the solve context that resolves motion, so the end-of-job park is
  written here, and the machine's `endOperations` stop before `M84` — a stepper
  disabled first would lose its position and park to the wrong place.

## Material numbers resolve themselves

A start or end script may reference `{hotendTemp}` (or `{nozzleTemp}`),
`{bedTemp}` and `{fanSpeed}`. Left unsupplied by the `Start`/`End` operation,
these four fill from the **Material** wired into the Machine Setup, the same way
an unwired `Temperature` target does. A value typed on the operation still wins.
Any *other* variable a script names is still the operation's to supply.

This is why the layered Prusa posts below carry no temperature at all: the
number is a fact about the filament, and it enters the job in exactly one
place — the Material socket on Machine Setup.

## What is here

| File | Pair it with | Shape |
|---|---|---|
| `Prusa MK4 (Buddy).json` | `Machines/Prusa-MK4.json` + `Effectors/Prusa-MK4-Nextruder.json` | Layered |
| `Prusa MK3S (Prusa-Firmware).json` | `Machines/Prusa-i3-MK3.json` + `Effectors/Prusa-i3-Extruder.json` | Layered |
| `prusa-marlin.json` | a hand-built machine with no start/end operations | **Self-contained** |
| `GRBL.json`, `grblHAL.json`, `GRBL Laser.json`, `GRBL Pen Plotter Servo.json` | any GRBL machine | Layered |
| `linuxcnc.json`, `multicam.json`, `onefinity-masso.json` | the matching control | Layered |

**`prusa-marlin.json` is the odd one out, deliberately.** Its start script still
homes and heats by itself, from before the machine and effector bands existed.
That makes it the right choice for a machine you built on the canvas that has no
bring-up of its own — and the wrong choice next to the shipped Prusa machine
presets, which would home and heat a second time. Its `{hotendTemp}`/`{bedTemp}`
now resolve from the Material like everything else, so it no longer needs those
numbers typed onto the `Start` operation.

## What is deliberately absent from the Prusa posts

- **`M862.3` / `M862.1` / `M862.5` (printer, nozzle and firmware checks).**
  Variant-exact, and the firmware string does not distinguish an MK4 from an
  MK4S — a shipped line would hard-stop half the machines it was meant to
  protect. Add one by hand once you know which variant you have.
- **`M115 U<version>`.** PrusaSlicer emits it; if the version you declare is
  newer than the one running, the printer pauses for 30 seconds asking you to
  upgrade, and a host has no way to abort that.
- **`M73` progress.** It reports *remaining* time, and Grackle does not yet
  estimate one. A file that opens by claiming zero minutes left is worse than a
  file that says nothing.
- **The purge / intro line.** A purge is real motion at bed-edge coordinates,
  and the right ones depend on where your machine's origin frame sits. Author it
  as a `Raw Gcode` operation at the head of your job, where you can see it in
  preview, rather than inheriting coordinates from a preset that cannot know
  your setup.
