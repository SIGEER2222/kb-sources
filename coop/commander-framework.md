# SC2 Co-op Commander Framework

The Co-op commander framework is shipped in the `alliedcommanders.sc2mod`
dependency and is implemented across four Galaxy libraries. Maps opt into the
framework by depending on `alliedcommanders.sc2mod` and following the
initialization contract below.

## Library layout

All four libraries live at
`mods/alliedcommanders.sc2mod/base.sc2data/Lib<Name>.galaxy` (+ `_h` headers):

| Library | Purpose | Depends on |
|---------|---------|-------------|
| `LibCOOC` (Core) | Commander selection, mastery levels, mission progress, perk state, hero unit lookup, player-commander binding. | `NativeLib` only. |
| `LibCOMU` (Mutators) | Mutator registration, enable/disable toggles, per-mutator hook functions (`CT_Apply*`). | `LibCOOC`. |
| `LibCOMI` (Mission) | Mission lifecycle, objectives, transmission portraits, AI attack waves, Co-op AI tech level buckets. | `NativeLib`, `LibCOOC`. |
| `LibCOUI` (UI) | Commander select screen, mission select, army customization UI, story scenes. | `LibCOMI`, `LibCOOC`. |

Load order is determined by `include` chains. Always include the `_h` variant
only at the top of your library — `include "LibCOOC"` loads both header and
implementation; `include "LibCOOC_h"` would only load declarations and produce
"function declared but not defined" errors at link time.

## Commander data model

A commander is a `CCommander` entry in `CommanderData.xml`. Each commander has:

- `id`: short identifier (e.g. `Raynor`, `Kerrigan`, `Alenger`).
- `Name`: display name string reference.
- `Race` / `SpawnRace`: race for tech tree and initial spawn.
- `HeroUnit`: the hero unit type the player controls.
- `HeroStructureType`: hero reviver building (e.g. Barracks-Hero, Portal-Hero).
- `HeroReviveLink`: link used to look up revival cost/time.
- `PortraitModel` / `PortraitActor`: UI portrait assets.
- `ConversationLink`: character conversation reference.
- `VoiceOver`: VoiceOver entry played on selection.
- `SelectCutscene`: cutscene played on commander select.
- `ScoreCoopStatistic` / `ScoreSelfStatistic`: score result IDs for scoring.

Query these via the `libCOOC_gf_CC_Commander*` family:

```galaxy
string  libCOOC_gf_CC_CommanderRace (string lp_commander);
string  libCOOC_gf_CC_CommanderHeroUnitType (string lp_commander);
string  libCOOC_gf_CC_CommanderHeroStructureType (string lp_commander);
int     libCOOC_gf_CC_CommanderLevel (int lp_player, string lp_commander);
string  libCOOC_gf_CC_CommanderData (string lp_commander);
string  libCOOC_gf_CC_CommanderUserInstance (string lp_commander);
```

## Player-commander binding

The engine initializes each player's selected commander from the lobby. Map
code reads it via:

```galaxy
string  libCOOC_gf_CC_PlayerCommander (int lp_player);
int     libCOOC_gf_CC_CommanderPlayer (string lp_commander);
int     libCOOC_gf_CC_PlayerAlly (int lp_player);
void    libCOOC_gf_CC_PlayerCommanderSet (int lp_player, string lp_commander);
```

For custom maps that don't use the Co-op lobby (e.g. 7vs1 single-player maps),
the map must write the player's commander into a Bank before
`libCOOC_InitLib` runs, or call `libCOOC_gf_CC_PlayerCommanderSet` manually
during `OnAfterPlayersInit`. The `RebornMapAdapter` pattern is a worked
example: it reads a `PrimaryCommander` bank key, calls
`libCOOC_gf_CC_PlayerCommanderSet`, then calls `libCOOC_InitLib` and
`libCOMI_InitLib` manually.

## Mastery system

Per-commander mastery points are stored in a Bank. Standard API:

```galaxy
int   libCOOC_gf_CC_PlayerMasteryLevel (int lp_player);
int   libCOOC_gf_CC_PlayerMasteryCategory (string lp_masteryUpgrade);
int   libCOOC_gf_CC_PlayerMasteryUpgradeLevel (int lp_player, string lp_masteryUpgrade);
void  libCOOC_gf_CC_PlayerMasteryUpgradeLevelSet (int lp_player, string lp_masteryUpgrade, int lp_level);
void  libCOOC_gf_CC_ApplyMasteryTech (int lp_player);
void  libCOOC_gf_CC_SaveMasteryToBank (int lp_player, bank lp_bank, string lp_section);
void  libCOOC_gf_CC_LoadMasteryFromBank (int lp_player, bank lp_bank, string lp_section);
```

`libCOOC_gf_CC_ApplyMasteryTech` applies all mastery effects to the player's
units. It must be called after the player's units are spawned, otherwise newly
spawned units won't have mastery modifiers applied.

## Mutator registration

Custom mutators register with the framework via `LibCOMU`:

```galaxy
void libCOMU_gf_CT_RegisterMutator (string lp_mutator, trigger lp_initTrigger, trigger lp_shutdownTrigger);
void libCOMU_gf_CT_MutatorShutdownTriggerSet (string lp_mutator, trigger lp_shutdownTrigger);
void libCOMU_gf_CT_EnableDisableMutator (bool lp_enableDisable, string lp_mutator);
```

A mutator mod's `Lib<Name>.galaxy` file follows this pattern (see
`mutatorblizzard.sc2mod/Lib9B7202B2.galaxy` for a real example):

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

The mutator ID (e.g. `"Blizzard"`) must match a `CMutator` entry registered
elsewhere (typically in the mutator mod's `GameData.xml` or via
`libCOMU_gf_CT_RegisterMutator` in `MapInit`).

## Per-mutator hooks

`LibCOMU` provides hook functions called from central places (unit creation,
unit death, unit damage). A mutator that wants to react to these events should
not subscribe directly; it should provide its logic via the corresponding
`CT_Apply*` API or wrap the framework's hook. Examples:

```galaxy
void libCOMU_gf_CT_ApplyAvenger (unit lp_killedUnit);
void libCOMU_gf_CT_ApplySpawnBroodling (unit lp_killedUnit);
void libCOMU_gf_CT_ApplyConcussiveAttacks (unit lp_damagedUnit, unit lp_damagingUnit);
void libCOMU_gf_CT_ApplyHardenedWill (unit lp_createdUnit);
void libCOMU_gf_CT_ApplyInspiration (unit lp_createdUnit);
```

These are no-ops when the corresponding mutator is disabled; mutator mods
override them with `void`-returning triggers via `TriggerCreate` and event
subscriptions on `UnitCreated` / `UnitDied` / `UnitDamaged`.

## Mission lifecycle (LibCOMI)

`LibCOMI` runs the mission lifecycle. Key entry points:

```galaxy
void libCOMI_gf_<Phase>_Start ();
```

Where `<Phase>` is one of:
- `InitGame` — early game setup, after player commanders are bound.
- `InitStart` — mission start cinematic / briefing.
- `ObjectiveRegister` — register main / bonus objectives.
- `Victory` / `Defeat` — mission end states.

Objectives register via `libCOOC_gf_CC_ObjectiveRegister` and update state
through `ObjectiveShow` / `ObjectiveSetState` natives. The
`libCOOC_ge_MissionObjectiveState` preset has `Undiscovered`,
`Available`, `Completed`, `Failed`.

## Common pitfalls

- **`libCOOC_gf_CC_CommanderIsDeveloping` does not exist** in the official
  library. Custom CMRE mods that referenced this function produced
  `ScriptError: function not defined`. The closest official API is
  `libCOOC_gf_CC_CommanderLevel(player, commander)`; check level > 0 to test
  "unlocked". See `sc2-porting-workspace/projects/cmre-porting/stages/04-runtime-baseline/issues.json`
  for the resolved case study.
- **`_h` include mistake**: `include "LibCOOC_h"` loads only declarations.
  Use `include "LibCOOC"` to get both declarations and implementation.
- **Mastery not applied**: if you spawn units before
  `libCOOC_gf_CC_ApplyMasteryTech` runs, mastery modifiers won't be applied
  to them. Either defer unit spawning to after mastery application, or re-run
  `ApplyMasteryTech` after spawning.
- **Commander binding on non-Co-op maps**: maps without the Co-op lobby must
  call `libCOOC_gf_CC_PlayerCommanderSet` from `OnAfterPlayersInit` before
  any commander-dependent code runs.
- **Mutator toggle timing**: `libCOMU_gf_CT_EnableDisableMutator` must be
  called during `MapInit`, not later — mutator hook triggers subscribe to
  unit events during init and won't pick up units created before they're
  registered.

## See also

- `triggers/system.md` (library load pattern)
- `catalog/format.md` (commander catalog data)
- `mutator/system.md` (mutator mod structure)
- `runtime-contracts/observer.md` (CMRE commander contract)
