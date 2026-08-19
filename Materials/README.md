# Materials

One JSON per material, loaded by Grackle's **Libraries** button. A material
entry describes a substance **completely** — a filament's temperatures and a
metal's chipload live in the same schema — and the *job* says which half is
live via `use`.

```jsonc
{
  "name": "PLA",

  // Additive (print) half
  "nozzleTemperature": 210,   // °C
  "bedTemperature": 60,       // °C
  "fanSpeed": 255,            // the post's raw units; 0 means off, not unset

  // Subtractive (machining) half
  "surfaceSpeed": 213.4,      // m/min
  "chipload": 0.0571,         // mm per tooth

  // Both
  "density": 1.24,            // g/cm³ — as printed on the spool or datasheet
  "use": "additive"           // additive | subtractive | (omit for unspecified)
}
```

## Where a material enters a job

Wire it into the **Material** input on **Machine Setup** — alongside the
machine, effector and tool. Not on Generate Code, and not on the operations.
From there:

- A `Temperature` operation with no target, and a `Fan` with no speed, resolve
  from it. That is how the machine's bed heat and the effector's hotend heat get
  their numbers without either preset naming a filament.
- A post's start/end script resolves `{hotendTemp}` / `{nozzleTemp}`,
  `{bedTemp}` and `{fanSpeed}` from it, so those numbers need not be typed onto
  the `Start` operation either.
- `density` turns the job's extruded filament volume into deposited grams, and a
  workpiece's measured volume into stock grams.

Swapping PLA for PETG is one wire here; the effector, the job and the post are
untouched.

## `use`

`additive` means the print numbers are live and a machining value asking to
resolve from this material is an error. `subtractive` is the mirror. Omitting it
(`unspecified`) reads whichever fields are authored — the behaviour every file
saved before the designation existed had.

Designating is what turns a vague failure into a precise one: stock aluminium
wired into a print job is told it is designated subtractive and how to fix it,
rather than reporting only that a number was missing. The four filaments here
declare `"additive"`; the stock materials are undesignated, and adding
`"use": "subtractive"` to your own copies is worth doing for the same reason.
