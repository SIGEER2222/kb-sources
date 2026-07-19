# SC2 Mutator System

Mutators are per-game modifiers that change gameplay rules in Co-op missions.
Each mutator is shipped as a self-contained mod that follows a fixed
convention; the framework (LibCOMU) registers and toggles them at match start.

## Directory layout

Mutator mods live under `mods/mutators/<name>.sc2mod/`. Two flavors:

- **Single mutators**: `mutator<name>.sc2mod` — one effect (e.g.
  `mutatorblizzard.sc2mod`, `mutatoravenger.sc2mod`).
- **Combo mutators**: `mutatorcombo<name>.sc2mod` — bundles several mutators
  with a themed name (e.g. `mutatorcomboblizzard.sc2mod`,
  `mutatorcomboburningtide.sc2mod`).

There are ~100 Blizzard mutators total. The full list of quick-select IDs is
in `LibCOMU_h.galaxy` under the `libCOMU_ge_CT_MutatorQuickList_*` constants
(see `sc2-data-trigger/mods/alliedcommanders.sc2mod/base.sc2data/LibCOMU_h.galaxy`).

## Mutator mod file structure

A single mutator mod has this layout (using `mutatorblizzard` as an example):

```
mutatorblizzard.sc2mod/
├── base.sc2data/
│   ├── GameData/
│   │   └── GameData.xml              # Catalog entries (CMutator, etc.)
│   ├── Lib9B7202B2.galaxy             # Implementation
│   ├── Lib9B7202B2_h.galaxy           # Declarations
│   └── TriggerLibs/                   # Optional, for nested imports
├── enus.sc2data/
│   └── LocalizedData/
│       ├── GameStrings.txt            # Display strings
│       └── TriggerStrings.txt         # Trigger string table
└── Triggers                           # Binary - DO NOT OVERWRITE
```

### `Lib<Hex>.galaxy` pattern

The library filename uses an obfuscated hex ID (e.g. `Lib9B7202B2` for the
Blizzard mutator) instead of a readable name. This is the editor's default
when a library is created without an explicit name; the hex is just a stable
identifier. You can rename the file (and update includes) without losing
functionality.

### Minimal mutator library

```galaxy
include "TriggerLibs/NativeLib"
include "LibCOMU"
include "Lib9B7202B2_h"

void lib9B7202B2_InitLibraries () {
    libNtve_InitVariables();
    libCOMU_InitVariables();
}

bool lib9B7202B2_gt_Initialization_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    libCOMU_gf_CT_EnableDisableMutator(true, "Blizzard");
    return true;
}

void lib9B7202B2_gt_Initialization_Init () {
    lib9B7202B2_gt_Initialization = TriggerCreate("lib9B7202B2_gt_Initialization_Func");
    TriggerAddEventMapInit(lib9B7202B2_gt_Initialization);
}

void lib9B7202B2_InitTriggers () {
    lib9B7202B2_gt_Initialization_Init();
}

bool lib9B7202B2_InitLib_completed = false;
void lib9B7202B2_InitLib () {
    if (lib9B7202B2_InitLib_completed) { return; }
    lib9B7202B2_InitLib_completed = true;
    lib9B7202B2_InitLibraries();
    lib9B7202B2_InitTriggers();
}
```

The `MapInit` trigger calls `lib9B7202B2_InitLib`, which initializes
dependencies (`libNtve`, `libCOMU`) and then registers the mutator's triggers.

## Mutator catalog entry

The mutator itself is declared as a `CMutator` catalog entry in the mod's
`GameData.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Catalog>
    <CMutator id="Blizzard" parent="CMutatorBase"/>
</Catalog>
```

Each mutator has properties like `Name` (display string), `Description`,
`Icon`, `Category`, `DisabledInDifficulty` (lockout by difficulty). See
`CommanderData.xml` / `GameData.xml` in any `mutator*.sc2mod` for a complete
field set.

## Registration with LibCOMU

A mutator registers itself with the framework in two ways:

1. **Implicit registration via `CMutator` catalog entry**: the framework
   enumerates all `CMutator` entries on init and adds them to the mutator
   pool.
2. **Explicit registration via `libCOMU_gf_CT_RegisterMutator`**: lets the
   mod attach init/shutdown triggers. Used when the mutator has init-time
   side effects beyond simple data overrides.

```galaxy
void libCOMU_gf_CT_RegisterMutator (string lp_mutator, trigger lp_initTrigger, trigger lp_shutdownTrigger);
void libCOMU_gf_CT_MutatorShutdownTriggerSet (string lp_mutator, trigger lp_shutdownTrigger);
void libCOMU_gf_CT_EnableDisableMutator (bool lp_enableDisable, string lp_mutator);
```

## Per-mutator hook functions

LibCOMU exposes hook functions called from central places (unit creation,
unit death, damage). Mutators override or wrap these via triggers:

```galaxy
void libCOMU_gf_CT_ApplyAvenger (unit lp_killedUnit);
void libCOMU_gf_CT_ApplySpawnBroodling (unit lp_killedUnit);
void libCOMU_gf_CT_ApplyDeathFire (unit lp_killedUnit);
void libCOMU_gf_CT_ApplyHybridDeathNuke (unit lp_killedUnit);
void libCOMU_gf_CT_ApplyConcussiveAttacks (unit lp_damagedUnit, unit lp_damagingUnit);
void libCOMU_gf_CT_ApplyHardenedWill (unit lp_createdUnit);
void libCOMU_gf_CT_ApplyInspiration (unit lp_createdUnit);
void libCOMU_gf_CT_ApplyPermaCloak (unit lp_createdUnit);
void libCOMU_gf_CT_ApplyUnitSpeed (unit lp_createdUnit);
```

These are no-ops when the corresponding mutator is disabled, so custom maps
can safely call them on every relevant event; the framework gates them.

## Combo mutator pattern

Combo mutators (`mutatorcombo*.sc2mod`) bundle 2-4 single mutators and ship
no library code of their own — only a `GameData.xml` declaring a
`CMutatorCombo` entry that lists the bundled mutator IDs. The framework
enables all bundled mutators when the combo is enabled.

## Toggle timing

`libCOMU_gf_CT_EnableDisableMutator` must be called during `MapInit`. If a
mutator registers hooks on unit events, units created before the toggle
will not have hooks applied to them — so custom mutator mods must defer unit
creation until after mutator init, or re-trigger hook application for
pre-existing units.

## Common pitfalls

- **`Triggers` file is binary**: a mutator mod has a top-level `Triggers` file
  (not a directory) that is a compiled binary blob. Never overwrite it with
  text; the engine reads it for pre-compiled trigger data.
- **Hex library name collisions**: each mod must use a unique hex ID for its
  library. The hex is generated by the editor from the library name; if you
  rename the library, regenerate the hex (or accept a different one).
- **`CMutator` parent**: always inherit from `CMutatorBase` to get default
  difficulty/UI/category fields; overriding them is optional.
- **Hook trigger subscription**: mutator hooks subscribe to `UnitCreated`,
  `UnitDied`, `UnitDamaged` during `MapInit`. If your map spawns units
  before `MapInit` finishes (e.g. preplaced units), they won't trigger the
  mutator's `UnitCreated` hook — use `libCOMU_gf_CT_ApplyPermaCloak(u)`
  etc. manually on preplaced units after mutator init if needed.
- **Localization**: mutator name/description must be in
  `enus.sc2data/LocalizedData/GameStrings.txt` to display correctly in the
  UI. The `Triggers.txt` table is for trigger string IDs.

## See also

- `coop/commander-framework.md` (mutator registration with LibCOMU)
- `triggers/system.md` (library load pattern)
- `catalog/format.md` (CMutator catalog entries)
