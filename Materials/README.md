# Materials

One library entry per substance. A material file describes what it *is* —
machining numbers, print numbers, and a density — and the job says which domain
it is being used in, on the Material Builder component's **Use** input
(Grackle `DECISIONS.md`, 2026-08-19).

## Folders

| Folder | Holds |
|---|---|
| `Additive/` | Feedstock an effector consumes — filaments, and anything else a job deposits. |
| `Subtractive/` | Stock a tool removes — metals, plywood, foam, wax. |
| `Materials/` (bare) | Presets authored before the split. Still scanned. |

Folders are **organization only**: every folder loads the same unified
`Material` schema, exactly as every `Tools/` category loads the same `Tool`.

All three folders are scanned, one folder deep. The Material Library's
**Libraries…** browser picks between them one at a time, the way the Tool
Library picks between `Endmills/` and `Nozzles/`; the component's own Name
dropdown merges all three, so picking a preset never depends on remembering
which folder it came from.

A material that is genuinely both printed and milled can be filed in both
folders — the additive entry is the one a shared name resolves to, so keep
the two in step rather than letting them drift. Filing it once and letting
the job's **Use** input designate the domain is the simpler option, and the
one `DECISIONS.md` (2026-08-19) describes.

Presets must sit **directly** inside one of these folders. A folder nested
deeper (`Materials/Additive/Prusament/`) is not scanned, and a preset there
is invisible to the dropdown.

## Fields

| Field | Unit | Notes |
|---|---|---|
| `name` | — | What the dropdown lists. |
| `nozzleTemperature` | °C | Resolves an unwired `Temperature` op on the hotend. |
| `bedTemperature` | °C | Resolves an unwired bed `Temperature` op. |
| `fanSpeed` | raw post units | 0–255 Marlin-style. `0` means the fan stays off (ABS); absent means unauthored. |
| `chamberTemperature` | °C | **Absent = needs no heated chamber.** Carried for enclosed printers; no gcode is emitted for it yet. |
| `density` | g/cm³ | Drives printed mass on Generate Code and stock mass on a Workpiece. |
| `surfaceSpeed` | m/min | Machining only. |
| `chipload` | mm/tooth | Machining only. |
| `use` | — | Normally **omitted**. Presets ship undesignated so they work in either direction; the Use input designates on load. |

## Provenance of the additive numbers

The temperatures, fan speeds and densities in `Additive/` are **conservative,
vendor-neutral starting points**, not values calibrated against printed parts.
Density in particular feeds mass estimates directly — check it against your own
spools before trusting a number. Per-brand entries are welcome as separate
files (e.g. `Prusament-PLA.json`).
