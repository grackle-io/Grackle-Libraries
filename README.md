# Grackle-Libraries

Preset data for the [Grackle](https://github.com/akudless/Grackle) Grasshopper
plugin — machines, effectors, tools, post-processors, and materials, as plain
JSON (plus a paired `.3dm` where a preset has geometry). Data only:
nothing here is code, and Grackle never executes anything it downloads from this
repo.

Each folder has its own README describing that category's schema and
conventions. Start there before adding a preset.

## How the plugin reads this repo

Every `*Library` component in Grackle has a **Libraries…** button. It lists what
is in this repo, installs the presets you pick into a local cache, and the
component's Name dropdown then offers them by the `name` field *inside* each
JSON file.

The plugin reads presets from three local roots, first match by filename
winning:

| | Path | Written by |
|---|---|---|
| **Override** | `<Documents>/Grackle/Libraries` | you, by hand |
| **Home override** | `~/Grackle/Libraries` | you, by hand |
| **Cache** | `<Grasshopper app data>/Grackle/Libraries` | the **Libraries…** button |

Right-click any Library component ▸ **Report Library Folders** to print those
three paths, as resolved on that machine, to the Rhino command line.

## Installing by hand (and the macOS shortcut)

The folder layout in this repo *is* the layout the plugin scans, so cloning it
into a library root installs everything at once:

```bash
git clone https://github.com/grackle-io/Grackle-Libraries.git ~/Grackle/Libraries
```

`~/Grackle/Libraries` is the root to prefer on macOS. `<Documents>` there
resolves to `~/Documents`, which macOS keeps behind the Privacy & Security ▸
Files and Folders consent prompt and iCloud Drive syncs by default — an
un-consented folder cannot be listed at all, and iCloud can evict a preset to a
`.Foo.json.icloud` stub that no file scan can read. The home folder is subject
to neither. On Windows the same argument applies to OneDrive's redirected
Documents folder.

Two rules if you lay the folders out yourself:

- **The scan is one level deep per category.** `Machines/foo.json` is found;
  `Machines/Prusa/foo.json` and a doubled `Libraries/Libraries/…` level are not.
  The scanned prefixes are `Machines/`, `Effectors/`, `PostProcessors/`,
  `Materials/` (plus `Materials/Additive/` and `Materials/Subtractive/`), and
  `Tools/{Nozzles,Endmills,Grippers,Saws,Custom}/`.
- **A preset appears under its `name` field, not its filename.** The two need
  not match, and a file with no `name` never reaches a dropdown.

## Contributing a preset

Open a PR against `main` with the JSON (and `.3dm`, if the part has geometry)
in the right category folder, named to match. Keep the pair's base name
identical — `Prusa-MK4.json` / `Prusa-MK4.3dm` — since that is how geometry is
found, and check the category README for what the schema expects.
