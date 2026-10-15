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
  **Exception: the Bambu posts**, whose firmware changes the extrusion mode
  during its own start sequence (see below).
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
| `Bambu Lab X1C.json`, `Bambu Lab H2S.json` | a Bambu machine built on the canvas, no start/end operations | **Self-contained** |
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

## Bambu Lab (X1C, H2S)

Both Bambu posts are **self-contained**: their start and end scripts are Bambu
Studio's own machine start/end gcode, taken from files Bambu Studio
`02.06.00.51` exported for each printer with PLA on a Textured PEI plate. There
is no Bambu machine preset in `Machines/` yet. If one later carries its own
`startOperations`, cut the matching lines out of the post so the job doesn't
home and level twice.

### What changed from Bambu Studio's output, and why

- **Material temperatures are variables.** Every nozzle and bed setpoint that
  Bambu Studio filled from the filament (220 °C / 55 °C in the source files) is
  now `{hotendTemp}` / `{bedTemp}`, so it comes from the Material wired into
  Machine Setup. Bambu's *fixed* temperatures stay as they were: 140 °C for
  probing, 75 °C to start the hotend fan, and on the X1C 250 °C for the flush and
  200 °C for the post-flush shrink.
- **Bed levelling covers the whole plate** (`G29 … X0 Y0 I256 J256` on the X1C,
  `I340 J320` on the H2S). Bambu Studio levels only the first layer's bounding
  box, and Grackle has no first-layer extent to give it.
- **Removed:**
  - `G21`, which the emitter writes in its preamble.
  - `M73` progress lines, because Grackle has no time estimate.
  - The start-up tune.
  - The X1C's first-layer lidar scan (`M977`, `M976`), and the H2S's tall-part
    obstacle check under the bed (`G3811 Z<print height>`). Bambu Studio leaves
    both out for some jobs too: the scan when first-layer inspection is off,
    the check when the print is under 145 mm. The scan is dropped because it
    expects a slicer-shaped first layer. The check is dropped because it needs
    the job's height, which a post can't know.
  - The spaghetti detector (`M981 S1`), which Bambu Studio turns on after its
    start gcode. Non-planar and wire-printed paths are what it is trained to
    flag.
- **The end scripts lift and drop the bed by fixed amounts.** Bambu Studio sizes
  those moves from `max_layer_z`, which a post can't know either, so they're
  relative lifts plus Bambu's own tall-part branch: drop the bed to the bottom.
  Moving the bed down only moves the part away from the nozzle.

### `M83` stays in, and Relative Extrusion must stay on

These are the only posts here whose scripts write `M83`, against the rule at the
top of this file. On the first X1C test, with Bambu's `M83` lines removed, the
emitter's single `M83` came before the start sequence. The job then ran every
move in the right place and extruded nothing. Something in Bambu's start sequence
(the AMS load, `T1000`/`T1100`, `M18 E`) leaves the extruder in absolute mode.
Each `E1.4` then means "go to position 1.4", which extrudes once and never again.
Bambu's own file sends `M83` again after each of those steps and once more before
the print. The posts now keep every one of those lines and end their start script
with `G90` + `M83`.

The cost is the one the rule warns about: the body must also be relative-E.
**Leave Relative Extrusion on** (the default) for these posts. With it off, the
emitter would write absolute `E` values into a file the start script has just put
in relative mode, and the print would over-extrude badly.

### What still assumes PLA

Lines Bambu Studio rendered from the filament profile, other than temperatures,
are kept as exported: `M1002 set_filament_type:PLA`, the PLA anti-jam chamber
fan (`M142`), the H2S chamber auto-cooling setting, and the flush and
calibration feed rates. For another filament, export a file from Bambu Studio
with that filament and regenerate the scripts. The same goes for the Textured
PEI `G29.1 Z` trim when you use a different plate.

### AMS, fans, arcs and getting the file to the printer

- **AMS:** the start script keeps Bambu's `M620 M` / `M620 S0A … T0 … M621 S0A`
  block, so the filament comes from whichever slot the printer screen maps to
  filament 1. The end script unloads it back to the AMS, as Bambu Studio does.
- **Fans:** a bare `M106 S…` is the part-cooling fan, which is what Bambu Studio
  writes. `P2` is the aux fan and `P3` the chamber fan.
- **Arcs:** both printers take `G2`/`G3`. Bambu Studio's exports use them
  throughout the print body on the X1C and on the H2S (a curved H2S part had
  about 300,000).
- **Start and End operations are required.** A post's start script runs only
  when the toolpath *begins* with a `Start` operation, and its end script only
  when the toolpath *ends* with an `End` operation. Generate Code adds neither,
  so a job without them has no bring-up: no homing, no heating, no levelling.
- **Delivery:** the file goes to the printer on a memory stick or SD card
  (whichever the printer takes). Bambu printers don't take gcode streamed over serial,
  so Grackle's Machine Control can't drive them.

Run the first job from each post with the printer attended.
