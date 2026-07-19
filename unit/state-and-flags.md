# Unit State and Flags

Units in SC2 expose two distinct layers of toggleable configuration:

- **Unit Flags** (`c_unitFlag*`) — set in the `CUnit.Flags` catalog field at data
  design time. They describe the unit's *category* and *static behavior*
  (worker, hero, missile, unselectable, invulnerable, ...). Most flags are
  read-only at runtime, but a subset has a matching `c_unitState*` runtime
  switch that can be flipped with `UnitSetState`.
- **Unit State** (`c_unitState*`) — runtime switches that can be toggled on a
  live unit via `UnitSetState(unit, state, bool)`. Some are read-only mirrors
  of derived conditions (Buried, Cloaked, InsideTransport, ...) and only
  reflect the underlying mechanics; the rest are writeable.

A third related layer — `unitfilter` — drives runtime unit selection by
filtering `unitgroup` queries by attribute flags (worker, mechanical, heroic,
...).

## Source references

- `sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/Unit.galaxy`:
  `c_unitFlag*` definitions (90+ flags).
- `sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/natives.galaxy`:
  `c_unitState*` definitions, `UnitSetState`, `UnitFilter*` natives,
  `UnitFilterMatch`.
- `catalog/fields-reference.md` §CUnit: `Flags` and `Attributes` block.
- `catalog/validators.md`: `CValidatorUnitFilters`, `CValidatorUnitState`.

## Unit flag list (catalog-time, `c_unitFlag*`)

All flags are declared in `GameData/Unit.galaxy` and surface as the
`<Flags index="..."/>` element inside `<CUnit>`. The corresponding XML
attribute name is the suffix after `c_unitFlag` (e.g. `c_unitFlagUnselectable`
→ `<Flags index="Unselectable"/>`).

### Interaction / targeting

| Flag | Meaning |
|------|---------|
| `Unclickable` | Cannot be clicked by mouse (no selection ring). |
| `Uncommandable` | Cannot receive player orders (still selectable for inspection). |
| `Unhighlightable` | Cannot be highlighted (no highlight ring). |
| `Untooltipable` | Tooltip never shows when hovering. |
| `Unselectable` | Cannot be selected. Implies `Unclickable`/`Unhighlightable`. |
| `Untargetable` | Cannot be picked as an ability/attack target. |
| `Uncursorable` | Custom cursor suppressed when hovering. |
| `HiddenCargoUI` | Cargo contents hidden in transport UI. |
| `IndividualSubgroups` | Each unit forms its own subgroup (used for carriers). |
| `ArmySelect` | Selectable via Army Select hotkey. |
| `TreatStructureAsUnitForSelection` | Structure behaves like a unit in selection. |
| `SelectableWhileDead` / `TargetableWhileDead` | Still selectable/targetable during death anim. |
| `ResumeLastMoveOrder` | After non-move order ends, resumes the previous move. |
| `PreferLastAttackTarget` | Auto-acquire prefers last attack target. |
| `IgnoreAttackAlert` | No "we're under attack" alert when attacked. |
| `AcquireRally` | Acquires rally-point target as attack target. |

### Visibility / cloaking

| Flag | Meaning |
|------|---------|
| `CreateVisible` | Created visible (flashes briefly on the minimap). |
| `PreventReveal` | Cannot be revealed by anything. |
| `Cloaked` | Is cloaked (subject to detector rules). |
| `Uncloakable` | Cannot cloak even if has cloak ability. |
| `Buried` | Is buried. |
| `Undetectable` | Cannot be detected. |
| `Unradarable` | Cannot show on radar. |
| `UseLineOfSight` | Vision blocked by LOS (default for ground units). |
| `VisionTestCenterOnly` | Vision test uses origin instead of footprint. |
| `NoDraw` | No model rendering. |
| `PreventDefeat` | Owner can't lose while this unit is alive. |
| `PreventDestroy` | Cannot be destroyed by damage. |

### Combat

| Flag | Meaning |
|------|---------|
| `Invulnerable` | Cannot take damage. |
| `Unstoppable` | Immune to most forms of interruption (knockbacks, etc.). |
| `Resistant` | Resistant (alternate interrupt-resistance rules). |
| `Destructible` | Treated as destructible doodad. |
| `KillCredit` | Killing it grants credit (bounty / objective). |
| `NoDeathEvent` | No `UnitDeath` event fired. |
| `NoScore` | Doesn't count toward score. |
| `LeechBehaviorShieldDamage` | Lifesteal applies to shield damage. |
| `ArmorDisabledWhileConstructing` | Loses armor while constructing. |
| `IgnoreAttackAlert` | No "under attack" ping. |
| `StatTrackAbilities` / `StatTrackDamageDone` / `StatTrackDamageReceived` / `StatTrackCreation` | Stat tracking. |

### Movement

| Flag | Meaning |
|------|---------|
| `Movable` | Can move at all. |
| `Turnable` | Can rotate in place. |
| `TurnBeforeMove` | Must turn before moving. |
| `Bounce` | Bounces off cliffs (e.g. reaper jump). |
| `Missile` | Treated as a missile for collision and targeting. |
| `ForceCollisionCheck` | Always run collision checks. |
| `ClearRallyOnDeath` / `ClearRallyOnTargetLost` | Self-explanatory. |
| `CameraFollow` | Default camera follow subject. |
| `IgnoreTerrainZInit` / `UseLineOfSight` | Terrain Z behavior. |

### Economy / construction

| Flag | Meaning |
|------|---------|
| `Worker` | Treated as a worker. |
| `HideFromHarvestingCount` | Excluded from worker harvesting count. |
| `Pawnable` | Can be sent to town hall via Pawn ability. |
| `ShowResources` | Shows resource contents in UI. |
| `BuiltOnOptional` | Can be built on optional terrain. |
| `ShareControl` | Control shared with allies. |
| `TownAlert` | Triggers town-alert when attacked. |
| `TownCamera` | Centering camera snaps here. |
| `TownStructureWall` / `Gate` / `TownHall` / `CannonTower` / `Moonwell` / `Core` | Town-structure role. |
| `PlayerRevivable` | Can be revived (hero). |
| `Hero` | Is a hero (xp, levels). |

### AI

| Flag | Meaning |
|------|---------|
| `AIThreatGround` / `AIThreatAir` | Threatens ground/air. |
| `AILifetime` | Counts against AI lifetime cap. |
| `AISplash` | AI considers splash damage. |
| `AIHighPrioTarget` / `AIDefense` / `AICaster` / `AISupport` / `AISplitter` | AI role. |
| `AICantAddToWave` | AI can't draft into waves. |
| `AIFleeDamageDisabled` / `AIPressForwardDisabled` | AI toggles. |
| `AIObservatory` | AI observer role. |
| `AIChangeling` | Treated as changeling. |
| `AIAllowSuicideOverride` | AI may suicide-attack. |
| `AIForceTactical` | Force tactical AI. |
| `AIPreferBurrow` | Prefer to burrow when idle. |
| `AIResourceBlocker` | Blocks resource pathing. |
| `AIMakeIgnore` | Build queue ignores it. |
| `AlwaysThreatens` | Always considered threatening. |

### Misc

| Flag | Meaning |
|------|---------|
| `Bounce`, `NoPortraitTalk`, `PenaltyRevealed`, `NoScore`, ... | Specialized flags; consult `Unit.galaxy` for the exhaustive list. |

## Unit state list (runtime, `c_unitState*`)

All `c_unitState*` constants are integers in `natives.galaxy`. Use:

```c
native void UnitSetState (unit inUnit, int inState, bool inVal);
native bool UnitGetState (unit inUnit, int inState);
```

States marked **Read-only** in the source are derived conditions — calling
`UnitSetState` on them is a no-op.

| Constant | Writeable? | Meaning |
|----------|-----------|---------|
| `Buried` | RO | Buried (mirror of `c_unitFlagBuried` + mechanic). |
| `Cloaked` | RO | Cloaked. |
| `Detector` | RO | Detector. |
| `Radar` | RO | Radar. |
| `VisionSuppressed` | RO | Vision blocked. |
| `AttackSuppressed` | W | Cannot attack. |
| `InStasis` | RO | In stasis (frozen). |
| `Hallucination` | RO | Hallucination. |
| `Invulnerable` | W | Invulnerable. |
| `Paused` | W | Frozen (no AI, no orders, no animations). |
| `Hidden` | W | Hidden (not rendered, not selectable). |
| `Highlightable` | W | Highlightable. |
| `IgnoreTerrainZ` | W | Ignore terrain Z. |
| `UnderConstruction` | RO | Under construction. |
| `InsideTransport` | RO | Inside a transport. |
| `InsideUnitTransport` | RO | Inside a unit transport. |
| `InsidePlayerTransport` | RO | Inside a player transport. |
| `Idle` | RO | Idle. |
| `Fidget` | W | Fidgeting animation. |
| `Selectable` | W | Selectable. |
| `Targetable` | W | Targetable. |
| `StatusBar` | W | Status bar shown. |
| `Tooltipable` | W | Tooltip shown. |
| `Cursorable` | W | Cursor override shown. |
| `IsDead` | RO | Dead. |
| `IsTransport` | RO | Is a transport. |
| `MoveSuppressed` | W | Cannot move. |
| `TurnSuppressed` | W | Cannot turn. |
| `Highlighted` | W | Currently highlighted. |
| `UsingSupply` | W | Counts toward supply. |
| `Revivable` | RO | Revivable. |
| `Detectable` | W | Detectable (overrides `Undetectable`). |
| `Radarable` | W | Radarable. |
| `Stunned` | W | Stunned (no orders, no attacks, no abilities). |
| `Stoppable` | W | Stoppable. |
| `Resistant` | W | Resistant. |
| `Silenced` | W | Cannot use abilities. |
| `Dazed` | W | Dazed (silenced + disarmed). |
| `IsChangeling` | RO | Changeling. |
| `Sleeping` | W | Sleeping (idle but visually so). |

### Toggle pattern (canonical)

To make a unit temporarily unselectable, untargetable, and invulnerable:

```c
void EnterStasis (unit u) {
    UnitSetState(u, c_unitStateSelectable, false);
    UnitSetState(u, c_unitStateTargetable, false);
    UnitSetState(u, c_unitStateInvulnerable, true);
    UnitSetState(u, c_unitStatePaused, true);
    UnitSetState(u, c_unitStateHidden, true);     // optional: also hide model
}

void ExitStasis (unit u) {
    UnitSetState(u, c_unitStateSelectable, true);
    UnitSetState(u, c_unitStateTargetable, true);
    UnitSetState(u, c_unitStateInvulnerable, false);
    UnitSetState(u, c_unitStatePaused, false);
    UnitSetState(u, c_unitStateHidden, false);
}
```

### Runtime vs catalog flag

`c_unitFlagUnselectable` (catalog-time) sets the **default** value of the
selectable state on unit creation. `c_unitStateSelectable` (runtime) is the
live value. Setting the catalog flag in XML is equivalent to calling
`UnitSetState(u, c_unitStateSelectable, false)` once when the unit is born.

If you need to flip a flag at runtime for *every* unit of a type, prefer
setting the catalog flag — runtime `UnitSetState` only affects the one unit.

## UnitFilter system

`unitfilter` is a runtime matcher that lets you query unit groups by
attribute. Unlike `c_unitState*`, it filters by inherent traits (race,
attribute, behavior presence, etc.) and is used by `UnitGroupSearch`,
`UnitGroupFilter`, and `CValidatorUnitFilters`.

### Creating a filter

There are two equivalent construction paths:

```c
// 1. From constants (most flexible, supports precomputed bitfields)
native unitfilter UnitFilter (int inRequired1, int inRequired2,
                              int inExcluded1, int inExcluded2);

// 2. From a filter string (more readable, parser-driven)
native unitfilter UnitFilterStr (string filters);
```

Filter string grammar (used by `UnitFilterStr`):

```
"[required1, required2];[excluded1, excluded2]"
```

Each token is a filter attribute name (case-insensitive), e.g.:

| Filter attribute | Matches |
|------------------|---------|
| `Ground` / `Air` | Movement plane. |
| `Mechanical` / `Biological` / `Robotic` / `Light` / `Armored` / `Massive` | Armor attribute. |
| `Structure` / `Worker` / `Heroic` / `Missile` | Unit class. |
| `Psionic` / `Undead` / `Demon` / `Elemental` | Bonus attribute. |
| `Self` / `Ally` / `Enemy` / `Neutral` | Alliance (toward the querying player). |
| `Visible` / `Hidden` | Visibility. |
| `Alive` / `Dead` | Life state. |
| `Detector` / `Cloaked` / `Buried` | Tactical state. |
| `Resource` / `Cargo` / `Transport` | Economy/transport. |
| `PreventDefeat` / `PreventDestroy` | Special. |
| `Unselectable` / `Untargetable` | Mirror of the catalog flag. |

Example:

```c
unitfilter f = UnitFilterStr("[Air; Mechanical; Enemy], [Missile; Hallucination]");
// Match living enemy air mechanical units that are not missiles or hallucinations.
bool match = UnitFilterMatch(someUnit, queryingPlayer, f);
```

### Runtime mutation

```c
native void UnitFilterSetState (unitfilter inFilter, int inType, int inState);
native int  UnitFilterGetState (unitfilter inFilter, int inType);

const int c_unitFilterAllowed   = 0;
const int c_unitFilterRequired  = 1;
const int c_unitFilterExcluded  = 2;
```

`inType` here is the filter attribute index (see `GameData/Unit.galaxy`
`c_targetFilter*` constants, e.g. `c_targetFilterDead`,
`c_targetFilterMissile`, `c_targetFilterHeroic`).

### Wiring into a unit group query

```c
unitgroup g = UnitGroup(
    /*type*/      null,                       // any type
    /*player*/   c_playerAny,
    /*region*/   someRegion,
    /*filter*/   UnitFilterStr("[Enemy; Visible], [Missile]"),
    /*maxCount*/ 0                            // 0 = unlimited
);
```

This is the most common use: enumerate all enemy visible units in a region,
excluding missiles. Note that `c_playerAny` plus an "Enemy" filter token is
the standard idiom for "everyone who is hostile to the querying player".

### Wiring into a catalog validator

`<CValidatorUnitFilters>` consumes the same filter string and returns true
when the unit matches:

```xml
<CValidatorUnitFilters id="EnemyAirOnly">
    <Filters value="Enemy;Air"/>
</CValidatorUnitFilters>
```

This validator can then gate `CAbilEffectTarget.ValidatorArray`,
`CEffectDamage.ValidatorArray`, `CBehaviorBuff.ValidatorArray`, etc.

## Runtime modification of catalog flag fields

You can change catalog flag fields at runtime via `CatalogFieldValueSet`.
This affects *all units of that entry*, not just one unit:

```c
// Make 'Marine' untargetable globally for player 1
int player = 1;
int entry  = CatalogGetEntry(c_gameCatalogUnit, /* Marine link */);
CatalogFieldValueSet(
    c_gameCatalogUnit, entry, "Flags[Untargetable].value",
    player, "1"
);
```

This is rarely needed — usually you want a `CBehaviorBuff` with a
`Modification` block that toggles state, applied via an effect or ability.

## Modification-based state toggles

`CBehaviorBuff` exposes a `Modification` block with sub-fields that map
directly to runtime state changes. See `catalog/fields-reference.md`
§Modification.

Example: a buff that makes the carrier untargetable while channelling:

```xml
<CBehaviorBuff id="ChannelUntargetable">
    <Modification>
        <States index="Untargetable" value="1"/>
        <States index="Unselectable"  value="1"/>
    </Modification>
</CBehaviorBuff>
```

The `States` array on `CBehaviorModification` accepts the same index names
as `c_unitState*` (`Selectable`, `Targetable`, `Invulnerable`, `Stunned`,
`Silenced`, `Dazed`, `Resistant`, `Sleeping`, `Hidden`, ...).

This is the **preferred** approach for state toggles tied to an ability or
effect chain — it's data-driven, no galaxy code required, and stacks
correctly with other buffs.

## Common patterns

### Toggle selectable via ability

```xml
<!-- Ability that toggles selectable on/off -->
<CAbilBehavior id="ToggleSelectable">
    <BehaviorId value="SelectOffBuff"/>
    <CmdButtonArray index="On"  Face="ToggleSelectableOn"/>
    <CmdButtonArray index="Off" Face="ToggleSelectableOff"/>
</CAbilBehavior>

<CBehaviorBuff id="SelectOffBuff">
    <Modification>
        <States index="Unselectable" value="1"/>
    </Modification>
</CBehaviorBuff>
```

When toggled on, the unit becomes unselectable; the buff is removed when the
ability is toggled off.

### Temporary invulnerability on damage taken

```xml
<!-- Triggered by an ApplyBehavior effect in the damage response chain -->
<CEffectApplyBehavior id="ApplyInvuln">
    <WhichUnit Value="Target"/>
    <Behavior value="InvulnBuff"/>
</CEffectApplyBehavior>

<CBehaviorBuff id="InvulnBuff">
    <Duration value="2.0"/>
    <Modification>
        <States index="Invulnerable" value="1"/>
    </Modification>
</CBehaviorBuff>
```

### Stunned debuff that also disables abilities and movement

```xml
<CBehaviorBuff id="Stun">
    <Modification>
        <States index="Stunned"    value="1"/>
        <States index="Silenced"   value="1"/>
        <States index="MoveSuppressed"  value="1"/>
        <States index="AttackSuppressed" value="1"/>
    </Modification>
</CBehaviorBuff>
```

Note: `Stunned` already implies all of the above as a derived mechanic; the
explicit states are belt-and-suspenders for abilities that check them
individually.

### Auto-acquire disable for "passive" units

```xml
<CBehaviorBuff id="PassiveGuard">
    <Modification>
        <States index="AttackSuppressed" value="1"/>
    </Modification>
</CBehaviorBuff>
```

Applied permanently (no duration) — the unit won't auto-acquire but can still
be ordered to attack manually.

## Pitfalls

- **Read-only states silently no-op.** `UnitSetState(u, c_unitStateCloaked, true)`
  does nothing — cloaking is driven by the cloak ability or the `Cloaked`
  catalog flag. Check the `// Read-only` comments in `natives.galaxy`.
- **`Paused` ≠ `Stunned`.** Paused freezes AI/animation/orders entirely (used
  by cinematic freezes). Stunned is the gameplay-facing equivalent that still
  shows animations and allows passive effects to fire.
- **`Hidden` removes the model.** If you only want to prevent interaction,
  use `Selectable=false; Targetable=false` — don't hide unless you want the
  model gone.
- **Filters ignore dead units by default.** Add `Dead` to required set if
  you want to enumerate corpses.
- **`UnitFilterStr` is parsed once.** Mutating the resulting `unitfilter`
  with `UnitFilterSetState` is more efficient than re-parsing the string
  every tick.
- **`CatalogFieldValueSet` on flag fields is not the same as
  `UnitSetState`.** Catalog mutation changes the *blueprint* and propagates
  to all existing + future units of that entry; `UnitSetState` is per-unit.
- **`c_unitStateInvulnerable` doesn't ignore `CEffectDamage.Kind` set to
  `Death` overrides.** Some "execute" effects bypass invulnerability via
  `<DeathType>` overrides in `CEffectDamage`.
- **Buff-stacked state counts.** If two buffs both set
  `States[Unselectable]=1`, removing one still leaves the unit
  unselectable until the second is also removed. The state value is the
  sum across all active modifications.
- **Custom UI selection logic** must check both the catalog flag *and* the
  runtime state — checking only one misses units that were toggled at
  runtime via `UnitSetState` or a behavior.

## See also

- `catalog/fields-reference.md` — CUnit.Flags block, CBehaviorModification.States
- `catalog/validators.md` — CValidatorUnitFilters, CValidatorUnitState
- `catalog/effects.md` — CEffectApplyBehavior for buff-driven state toggles
- `galaxy/native-index.md` — UnitSetState, UnitFilter, UnitFilterStr
- `triggers/system.md` — UnitDeath / UnitCreated events for state-driven logic
