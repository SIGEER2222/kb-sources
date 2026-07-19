# SC2 Document Structure

A `.SC2Map` or `.SC2Mod` file is an MPQ archive containing a fixed set of document
files. This page lists the files the editor and engine read at runtime, and which
are authoritative for which concerns.

## Files inside a `.SC2Map`

| File | Format | Authority for |
|------|--------|---------------|
| `DocumentHeader` | Binary | Effective dependency list. The engine reads this, not `DocumentInfo`. Copying files between maps without preserving the target's `DocumentHeader` corrupts dependency resolution. |
| `DocumentInfo` | XML | Human-readable dependency declaration; advisory, used by the editor UI. Treat as a mirror of `DocumentHeader`, not the source of truth. |
| `MapInfo` | Binary | Map dimensions, tileset, players, starting locations. |
| `Objects` | Binary | Preplaced units, doodads, points, regions, cameras. |
| `Triggers` | Binary | GUI trigger definitions, categories, parameters, custom scripts. Compiles to `MapScript.galaxy` on save. |
| `Attributes` | Binary | Map attribute values (difficulty, commander slot, etc.). |
| `CustomAI` | Binary | AI script data. |
| `t3HardTile` / `t3VertCol` / `t3Water` / `t3Terrain.xml` / `t3HeightMap` / ... | Binary / XML | Terrain data layers. |
| `Minimap.tga` | TGA | Minimap preview image. |
| `Preload.xml` | XML | Asset preload list. |
| `BankList.xml` | XML | Banks used by the map. |
| `Base.SC2Data/GameData.xml` + `GameData/*.xml` | XML | Catalog data (data-space mode). |
| `MapScript.galaxy` | Text (Galaxy) | Compiled trigger script. Re-generated on save; do not hand-edit unless using custom script elements. |
| `TriggerLibs/*.galaxy` | Text (Galaxy) | External libraries included by `MapScript.galaxy`. |
| `GameText.txt` / `GameText.xml` | Text / XML | Localized strings referenced from Catalog and triggers. |

## Files inside a `.SC2Mod`

A `.SC2Mod` is the same MPQ archive layout but typically omits map-specific files:

| File | Format | Authority for |
|------|--------|---------------|
| `DocumentHeader` | Binary | Effective dependency list. |
| `DocumentInfo` | XML | Human-readable dependency mirror. |
| `Base.SC2Data/GameData.xml` + `GameData/*.xml` | XML | Catalog data. |
| `TriggerLibs/*.galaxy` + `TriggerLibs/*_h.galaxy` | Text (Galaxy) | Trigger libraries auto-loaded via the data editor field `TriggerLibs Identifier` (识别符) on `SC2 Gameplay Defaults`. |
| `GameText.txt` / `GameText.xml` | Text / XML | Localized strings. |

## DocumentHeader vs DocumentInfo

This is the single most important distinction for porting work:

- `DocumentHeader` is **binary** and contains the dependency list as embedded
  C-style strings. The engine reads this file at load time.
- `DocumentInfo` is **XML** and declares the same dependencies for the editor UI.
- When copying a map's files into another map, NEVER overwrite the target's
  `DocumentHeader` with the source's. The target's effective dependencies must
  remain authoritative. Overwriting silently changes what mods load.
- A typical corruption symptom is "missing unit" or "missing ability" errors at
  runtime that the editor does not show.

## Dependency direction

Dependencies are the parent of the map/mod. The map inherits all of the parent's
data. To change a value defined in a dependency, either:

1. Modify the dependency mod itself.
2. Override the specific field in the map/mod by re-declaring the entry with the
   same `id` and overriding only the desired fields.

The map cannot remove a dependency's entry, only override fields.

## 7vs1 campaign map caveat

A 7vs1 map is structurally a **campaign map** (自由之翼 / 虫群之心 / 虚空之遗
campaign dependency), not a custom melee map. Campaign maps have:

- Additional trigger libraries (`CampaignLib`).
- Strict tech-tree triggers that gate unit production.
- Save-breaking triggers that must be removed before editing (e.g. some tech
  research triggers in 自由之翼).
- Different dependency resolution semantics compared to custom melee maps.

Treat 7vs1 maps as campaign maps for all porting decisions.

## See also

- `mpq/format.md`
- `catalog/format.md`
- `data-spaces/usage-guide.md`
- `triggers/system.md`
