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

## What is deliberately absent

- **The purge / intro line.** It is real motion at bed-edge coordinates, and
  fragment operations run outside the solve context that resolves motion, so a
  preset cannot know where your machine's origin frame puts `Y-2`. Author the
  purge as a `Raw Gcode` operation at the head of your job instead, where you
  can see it in preview before it runs.
