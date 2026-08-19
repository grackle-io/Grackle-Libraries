# Effectors

One JSON per effector, loaded by Grackle's **Libraries** button. An effector
that has geometry keeps it in a `.3dm` of the same name, whose layer names match
the `parts` entries; an effector with no `.3dm` and no `parts` still works for
code generation, it just does not appear in preview.

## Start and end operations

`startOperations` runs at the top of every tool segment this effector works in;
`endOperations` runs when a segment hands off to a different effector, and at the
end of the job. This is the *effector's* band — it runs after the machine's
(bed heat, home, mesh) and before the job body, which is exactly the order a
hotend wants: come up to temperature once the mesh has been measured, not
before it.

They are authored as **operations**, never as gcode text. An effector is saved
to the library and reused across dialects and machines; a gcode string on it
would break on the first machine swap, while an operation renders through
whichever post the job runs, animates in preview, and is costed in the time
estimate.

The hotend heat carries **no number**, for the same reason the machine's bed
heat does not: a temperature is a fact about the filament, not about the
hardware. `"wait": true` with no `target` resolves from the job's `Material` at
emit time, so one extruder file serves PLA and PETG. If no material is wired,
Generate Code says so rather than guessing.

## What is here

| File | Notes |
|---|---|
| `Prusa-i3-Extruder.json` | MK3S extruder, with geometry. Heats the hotend. |
| `Prusa-MK4-Nextruder.json` | MK4 Nextruder. No geometry yet — bring-up only. |
| `Schunk-PSH32.json` | Gripper, with geometry and jaw states. |
| `UH-Pen-Holder.json` | Pen holder for the plotter. |

Neither Prusa extruder turns its own heater **off** in `endOperations`: the
machine band already cools the hotend and bed at the end of the job, and a
second `M104 S0` would just be noise. An effector that hands off mid-job to a
different effector — a real tool change — is the case where a tear-down entry
earns its place.

## The prime belongs here, not on the machine

The effector's band runs **after** the machine's, and that is the whole reason
the prime lives here. A purge authored into the machine's `startOperations`
emits between the mesh levelling and the hotend heat — it extrudes cold, which
grinds filament instead of priming. Put it below this effector's own
`Temperature`, which is the first point in the job where the nozzle is hot:

```jsonc
"startOperations": [
  { "$type": "temperature", "heater": "hotend", "wait": true },
  { "$type": "raw",
    "gcode": "G1 Z0.2 F720\nG1 Y-2 F1000\nG92 E0\nG1 X60 E9 F1000",
    "name": "intro line" }
]
```

The band re-runs on a material change as well as a tool change, which is what
you want: new filament genuinely needs purging through.

### A purge can read its own temperature

Four `{token}` names resolve from the job's `Material` inside a raw block —
`{hotendTemp}` (or `{nozzleTemp}`), `{bedTemp}` and `{fanSpeed}` — the same
names a post's start script uses. That lets a purge be self-contained, bringing
the nozzle up itself instead of relying on a `Temperature` above it:

```jsonc
{ "$type": "raw",
  "gcode": "M109 S{hotendTemp:.0f}\nG1 Z0.2 F720\nG1 Y-2 F1000\nG92 E0\nG1 X60 E9 F1000",
  "name": "intro line" }
```

One authored block, and the MK4 emits `M109 S210` under PLA and `M109 S240`
under PETG. Write `{hotendTemp:.0f}` to control decimals.

**Every other brace is left exactly as typed**, so a conditional pasted from a
slicer (`{if layer_z < max_print_height}…{endif}`) still emits unchanged. The
one thing that is an error rather than literal text is a *known* token the job
cannot resolve — `{hotendTemp}` with no material wired — because shipping that
brace to the firmware would fail on the machine instead of on the canvas.

## What is deliberately absent

- **The purge / intro line itself.** The shape is above, but no coordinates
  ship. It is real motion at bed-edge coordinates, and fragment operations run
  outside the solve context that resolves motion, so a preset cannot know where
  your machine's origin frame puts `Y-2` — and a wrong one scrapes the bed.
  Author it against your own setup, where you can see it in preview first.
- **Tool and machine tokens in `Raw Gcode`.** The four *material* tokens above
  resolve; a raw block still cannot reach the tool (`{nozzleDiameter}`,
  `{filamentDiameter}`) or the machine, which a linear-advance `M900 K…` would
  want. Kept deliberately to the names already contracted rather than opened
  into a general expression language.
