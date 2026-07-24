# Grackle Libraries

Data only — no plugin code. This branch is what the Grackle Grasshopper
plugin's "Libraries…" buttons browse and install from (Material Library, Tool
Library, Machine Library, Effector Library, Post-Processor Library), the same
way visose/Robots' "Libraries" button installs from its own `libraries`
branch.

## Layout

```
Machines/                      Machine presets (+ optional paired .3dm geometry)
Tools/
  Nozzles/                     GKL_Tool presets, type: nozzle
  Endmills/                    GKL_Tool presets, type: endMill
  Grippers/                    GKL_Tool presets, type: gripper
  Saws/                        GKL_Tool presets, type: saw
Effectors/                     Effector presets (+ optional paired .3dm) — flat, no subfolders
PostProcessors/                Post-processor presets
Materials/                     Material cutting-parameter presets
```

Every `Tools/` category shares one unified `GKL_Tool` schema (Name, Number,
TCP, plus the fields relevant to its `"type"`) — the folder is organizational
only, so a shop can browse by kind. `Effectors/` is flat: every effector
(spindle, extruder, gripper, saw) loads the same `Effector` schema, so there's
nothing to categorize by. A `.json` may have a paired `.3dm` of the same name
(Machines, Effectors, Tools) carrying its Rhino geometry — for
Machines/Effectors, keyed by layer name to `BodyPart.Name`; for Tools, the
tip mesh (any layer).

Each preset is a `.json` file matching the schema in `Grackle.Core` (see the
main branch's `presets/` folder, which remains the human-authored source for
these files, and `docs/architecture-handoff.md` for the JSON field
reference).

Contribute a preset by opening a PR against this branch with a new `.json`
(and `.3dm`, if it carries geometry) in the appropriate category folder.
