# SC2 Catalog Data Format

The Catalog is the structured game-data layer of StarCraft II. Every unit, ability,
weapon, effect, behavior, actor, upgrade, requirement, and similar entry lives in a
Catalog XML document. The Galaxy Editor shows this data through its Data Editor GUI;
underlying files are XML.

## Document location

Catalog XML lives inside `Base.SC2Data/GameData/` of a mod or map. The main entry
point is `Base.SC2Data/GameData.xml`, which uses `<Includes>` to reference individual
catalog files:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Catalog>
    <Includes>
        <Catalog path="GameData/Units.xml"/>
        <Catalog path="GameData/Abilities.xml"/>
        <Catalog path="GameData/Commanders.xml"/>
    </Includes>
</Catalog>
```

This is called **data space mode** and is the only supported layout for project mods
in this workspace. See `data-spaces/usage-guide.md` for the full guide.

## Catalog scopes

Each catalog scope has its own XML root element. The most common scopes:

| Scope | Root element | Typical content |
|-------|--------------|-----------------|
| Units | `<Units>` | Unit definitions, HP, movement, race, build time. |
| Abilities | `<Abilities>` | Command-card actions, cast effects, cooldowns. |
| Weapons | `<Weapons>` | Damage, period, range, target filters. |
| Effects | `<Effects>` | Damage / create unit / set behavior / etc. sub-actions. |
| Behaviors | `<Behaviors>` | Buffs/debuffs, auras, periodic effects. |
| Actors | `<Actors>` | Visual/sound layer between data and model. |
| Upgrades | `<Upgrades>` | Tech-tree upgrades that modify other entries. |
| Requirements | `<Requirements>` | Tech-tree gating for production and research. |
| Buttons | `<Buttons>` | Command-card button UI. |
| Commanders | `<Commanders>` | Coop commander definitions (mods like CMRE). |
| CommanderAbilities | `<CommanderAbilities>` | Commander GP-tier abilities. |

## Entry structure

```xml
<CUnit id="Marine">
    <Face value="Marine"/>
    <Race value="Terr"/>
    <Radius value="0.375"/>
    <LifeStart value="45"/>
    <LifeMax value="45"/>
    <FlagArray index="Movable" value="1"/>
    <Attributes index="Light" value="1"/>
</CUnit>
```

- `id` is the entry's unique key within its catalog scope.
- Child elements are **fields** identified by their tag name.
- `index="..."` selects a specific slot in an array-like field.
- `<FlagArray index="X" value="1"/>` toggles a flag bit.

## Inheritance and override

```xml
<CUnit id="MarineRaynor" parent="Marine">
    <LifeMax value="60"/>
    <LifeStart value="60"/>
</CUnit>
```

- `parent="..."` declares inheritance from another entry.
- Re-declaring a field overrides the parent's value.
- Catalog merges by entry ID: later-loaded documents with the same `id` overlay
  the earlier ones at the field level, not the entry level. Unmentioned fields
  keep the parent's value.

## Load order and merge semantics

Mods/maps declare dependencies in `DocumentHeader` (binary) and `DocumentInfo` (XML).
The engine loads dependencies in declared order, then the current document last.
Later loads overlay earlier loads.

Rules:
- Field-level override: only redeclared fields change; sibling fields are untouched.
- `id` collisions merge by field. To remove a field, use the appropriate removal
  syntax (varies by field; consult the editor's "raw data" view).
- A unit cannot exist partially across documents unless each document declares
  the unit with `parent=` pointing to a complete base entry.
- Editor "View Raw Data" (Ctrl+Shift+Alt+D) shows internal IDs and effective values.

## Validation

- The Galaxy Editor saves only structurally valid Catalog XML; invalid XML fails
  the save with a generic error.
- Cross-catalog references (e.g. an ability referencing an effect `id`) must resolve
  in the effective merged catalog at load time, not at edit time.
- Removing a unit entry that another entry references (via `parent` or ability
  link) will produce a load-time error.

## See also

- `data-spaces/usage-guide.md`
- `editor/editor-overview.md` §6 (Data Editor overview)
- `document-structure/overview.md`
