# SC2 Validator System

Validators (`CValidator`) are reusable boolean checks used throughout the
catalog. They gate abilities, requirements, effects, behaviors, and
targeting. Defining a check once as a validator lets you reuse it
wherever a "Condition" field appears.

Source declarations:
`sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/Validator.galaxy`.
Catalog data: `ValidatorData.xml`.

## Where validators are used

| Field | Where it appears |
|-------|------------------|
| `Requirements` | `CAbil`, `CButton`, `CUpgrade`, `CBehavior` — visibility / cast gate. |
| `ValidatorArray` | `CAbil` (cast-time), `CEffect` (pre-effect), `CWeapon` (attack-time). |
| `RemoveValidatorArray` | `CBehavior` — auto-remove when validator fails. |
| `ValidatorArray` on `CEffectEnumArea` | Per-enumerated-target check. |
| `CaseArray` on `CEffectSwitch` | Branch decision. |

Validators return true / false. A failing validator on `CAbil.ValidatorArray`
prevents the cast. A failing validator on `CEffect.ValidatorArray` skips
the effect. A failing `RemoveValidatorArray` on `CBehavior` removes the
behavior.

## Validator subclass list

SC2 5.0.16 ships 80+ validator subclasses. Grouped by what they check:

### Logic combinators

| Subclass | Purpose |
|----------|---------|
| `CValidatorCombine` | Combine multiple validators with AND / OR / NOT logic. |
| `CValidatorCondition` | General condition tree. |
| `CValidatorFunction` | Custom Galaxy function call. |

### Effect-related

| Subclass | Purpose |
|----------|---------|
| `CValidatorEffect` | Run an effect and check if it succeeded. |
| `CValidatorEffectCompare` | Compare effect result value. |
| `CValidatorEffectCompareDodged` | Check if target dodged. |
| `CValidatorEffectCompareEvaded` | Check if target evaded. |
| `CValidatorEffectTreeUserData` | Check effect tree user data. |

### Location-based

| Subclass | Purpose |
|----------|---------|
| `CValidatorLocation` | Base. |
| `CValidatorLocationCompareCliffLevel` | Compare cliff level. |
| `CValidatorLocationComparePower` | Check power source coverage. |
| `CValidatorLocationCompareRange` | Check distance to a point/unit. |
| `CValidatorLocationArc` | Check angle arc. |
| `CValidatorLocationCreep` | Check creep coverage. |
| `CValidatorLocationCrossChasm` | Check chasm crossing. |
| `CValidatorLocationCrossCliff` | Check cliff crossing. |
| `CValidatorLocationEnumArea` | Enumerate area and check. |
| `CValidatorLocationPathable` | Pathability check. |
| `CValidatorLocationInPlayableMapArea` | In playable bounds. |
| `CValidatorLocationPlacement` | Placement check (build). |
| `CValidatorLocationShrub` | Shrub check. |
| `CValidatorLocationType` | Location type. |
| `CValidatorLocationVision` | Vision check. |

### Player-based

| Subclass | Purpose |
|----------|---------|
| `CValidatorPlayer` | Base. |
| `CValidatorPlayerAlliance` | Alliance check (eg. enemy of). |
| `CValidatorPlayerRequirement` | Player requirement check. |
| `CValidatorPlayerTalent` | Player talent check. |
| `CValidatorPlayerCompare` | Generic compare. |
| `CValidatorPlayerCompareDifficulty` | Difficulty compare. |
| `CValidatorPlayerCompareFoodAvailable` | Food available compare. |
| `CValidatorPlayerCompareFoodMade` | Food made compare. |
| `CValidatorPlayerCompareFoodUsed` | Food used compare. |
| `CValidatorPlayerCompareRace` | Race compare. |
| `CValidatorPlayerCompareResource` | Resource compare. |
| `CValidatorPlayerCompareResult` | Game result compare. |
| `CValidatorPlayerCompareType` | Player type compare. |

### Unit-based (state)

| Subclass | Purpose |
|----------|---------|
| `CValidatorUnit` | Base. |
| `CValidatorUnitInWeaponRange` | In weapon range. |
| `CValidatorUnitAI` | AI check. |
| `CValidatorUnitCombatAI` | Combat AI state. |
| `CValidatorUnitAbil` | Has ability. |
| `CValidatorUnitBehaviorState` | Behavior state. |
| `CValidatorUnitState` | Unit state (alive, paused, etc.). |
| `CValidatorUnitDetected` | Detection status. |
| `CValidatorUnitFilters` | Filter check (Light / Armored / etc.). |
| `CValidatorUnitFlying` | Flying check. |
| `CValidatorUnitInventory` | Inventory check. |
| `CValidatorUnitInventoryIsFull` | Inventory full. |
| `CValidatorUnitInventoryContainsItem` | Inventory contains item. |
| `CValidatorUnitLastDamagePlayer` | Last damage source. |
| `CValidatorUnitKinetic` | Kinetic state. |
| `CValidatorUnitMissileNullified` | Missile nullified. |
| `CValidatorUnitMover` | Mover check. |
| `CValidatorUnitOrder` | Order state. |
| `CValidatorUnitOrderQueue` | Order queue. |
| `CValidatorUnitOrderTargetPathable` | Order target pathable. |
| `CValidatorUnitOrderTargetType` | Order target type. |
| `CValidatorUnitPathable` | Pathing check. |
| `CValidatorUnitPathing` | Pathing check. |
| `CValidatorUnitScanning` | Scanning state. |
| `CValidatorUnitType` | Unit type check. |
| `CValidatorUnitWeaponAnimating` | Weapon animating. |
| `CValidatorUnitWeaponFiring` | Weapon firing. |
| `CValidatorUnitWeaponPlane` | Weapon plane (air / ground). |

### Unit-based (compare)

| Subclass | Purpose |
|----------|---------|
| `CValidatorUnitCompare` | Generic compare. |
| `CValidatorUnitCompareAIAreaEvalRatio` | AI area eval ratio. |
| `CValidatorUnitCompareAbilLevel` | Ability level. |
| `CValidatorUnitCompareAttackPriority` | Attack priority. |
| `CValidatorUnitCompareBehaviorCount` | Behavior stack count. |
| `CValidatorUnitCompareCargo` | Cargo contents. |
| `CValidatorUnitCompareChargeUsed` | Charge used. |
| `CValidatorUnitCompareCooldown` | Cooldown remaining. |
| `CValidatorUnitCompareDamageDealtTime` | Time since dealt damage. |
| `CValidatorUnitCompareDamageTakenTime` | Time since took damage. |
| `CValidatorUnitCompareDeath` | Death state. |
| `CValidatorUnitCompareDetectRange` | Detect range. |
| `CValidatorUnitCompareField` | Generic field compare. |
| `CValidatorUnitCompareKillCount` | Kill count. |
| `CValidatorUnitCompareMarkerCount` | Marker count. |
| `CValidatorUnitCompareMoverPhase` | Mover phase. |
| `CValidatorUnitCompareOrderCount` | Order count. |
| `CValidatorUnitCompareOrderTargetRange` | Order target range. |
| `CValidatorUnitComparePowerSourceLevel` | Power source level. |
| `CValidatorUnitComparePowerUserLevel` | Power user level. |
| `CValidatorUnitCompareRallyPointCount` | Rally point count. |
| `CValidatorUnitCompareResourceContents` | Resource contents. |
| `CValidatorUnitCompareResourceHarvesters` | Resource harvesters. |
| `CValidatorUnitCompareSpeed` | Speed. |
| `CValidatorUnitCompareVeterancyLevel` | Veterancy level. |
| `CValidatorUnitCompareVital` | Vital compare (HP / shields / energy). |
| `CValidatorUnitCompareVitality` | Vitality (combined). |
| `CValidatorUnitCompareHeight` | Height compare. |

### Game-based

| Subclass | Purpose |
|----------|---------|
| `CValidatorGameCompareTimeOfDay` | Time of day. |
| `CValidatorGameCompareTerrain` | Terrain type. |
| `CValidatorGameCommanderActive` | Co-op commander active. |

## Common validator fields

Every validator subclass shares:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Find` | enum | What to look up (`Location` / `Source` / `Target` / `Outer`). |
| `Value` | string | Compare value. |
| `Compare` | enum | Compare op (`Eq` / `Ne` / `Gt` / `Ge` / `Lt` / `Le`). |

`Compare` is the comparison operator. `Value` is the comparison value
(string, parsed per validator type). `Find` selects which subject's data
to inspect.

## `CValidatorCombine` (logic combiner)

The most-used validator for complex conditions:

```xml
<CValidatorCombine id="IsArmoredAndNotHeroic">
    <CombineArray index="0" Operator="Require" Validator="IsArmoredValidator"/>
    <CombineArray index="1" Operator="Require" Validator="IsNotHeroicValidator"/>
</CValidatorCombine>
```

Operators (`c_validatorCombine*`):

- `Require` — AND: this validator must pass.
- `Exclude` — AND NOT: this validator must fail.
- `Disable` — disable this branch entirely (skip).

Multiple entries combine left-to-right. For OR logic, use multiple
`CValidatorCombine` entries nested inside a parent that uses `Require`
on each (but combine them via a separate OR-rooted combine).

## Compare validator pattern

`CValidatorUnitCompareVital` and similar:

```xml
<CValidatorUnitCompareVital id="HealthBelow50">
    <Value value="50"/>
    <Compare value="Lt"/>
    <Find value="Target"/>
</CValidatorUnitCompareVital>
```

Returns true when target's current vital is less than 50.

For vital fraction (percent), use `CValidatorUnitCompareVitalFraction`
(which doesn't exist — use `CValidatorUnitCompareVital` with the
`VitalType` and `Compare` against max value via a wrapper).

## Filter validator

`CValidatorUnitFilters` checks if a unit matches a filter set:

```xml
<CValidatorUnitFilters id="IsEnemyGround">
    <Filters value="Enemy,Ground"/>
    <Exclude value="Missile"/>
</CValidatorUnitFilters>
```

Filters use the same flag set as `CWeapon.Filters` and
`CEffectEnumArea.SearchFlags`.

## Effect-result validator

`CValidatorEffectCompare` runs an effect and compares its result:

```xml
<CValidatorEffectCompare id="TargetWillDieFromDamage">
    <Effect value="MyDamagePreview"/>
    <Compare value="Ge"/>
    <Value value="0"/>  <!-- VitalChange result -->
</CValidatorEffectCompare>
```

This is the basis for "kill secured" effects — check if the damage would
kill the target, and only apply a "execute" bonus if so.

## Use in abilities

To gate an ability cast:

```xml
<CAbilEffectTarget id="MyAbility">
    <ValidatorArray index="0" value="TargetIsEnemyValidator"/>
    <ValidatorArray index="1" value="CasterHasEnergyValidator"/>
</CAbilEffectTarget>
```

All validators in `ValidatorArray` must pass for the cast to succeed.

## Use in effects

`CEffect.ValidatorArray` runs pre-effect. If any fails, the effect is
skipped (chain continues with the next effect).

```xml
<CEffectApplyBehavior id="ApplyStunIfNotBoss">
    <ValidatorArray index="0" value="IsNotBossValidator"/>
    <Behavior value="Stun"/>
</CEffectApplyBehavior>
```

## Use in CEffectSwitch

```xml
<CEffectSwitch id="DamageByTargetType">
    <CaseArray index="0">
        <Validator value="IsArmoredValidator"/>
        <Effect value="AntiArmorDamage"/>
    </CaseArray>
    <CaseArray index="1">
        <Validator value="IsLightValidator"/>
        <Effect value="AntiLightDamage"/>
    </CaseArray>
    <Default value="GenericDamage"/>
</CEffectSwitch>
```

The first matching case wins. `Default` runs if none match.

## Use in behaviors (auto-remove)

```xml
<CBehaviorBuff id="CloakBuff">
    <RemoveValidatorArray index="0" value="IsAttackedValidator"/>
    <Duration value="0"/>  <!-- permanent until removed -->
</CBehaviorBuff>
```

When `IsAttackedValidator` returns true (the unit takes damage), the
behavior is automatically removed.

## Use in requirements

`CRequirementNode` entries can reference validators as their condition
node. See `requirement/system.md` for the requirement node tree pattern.

## Runtime check

You can evaluate a validator from Galaxy:

```galaxy
bool CatalogValidatorEvaluate (string validator, int player, unit source, unit target);
```

Useful for trigger-driven logic that needs to reuse catalog-defined checks.

## Common pitfalls

- **Default `Find` is `Target`**: if you want to check the source / caster,
  set `Find` to `Source` explicitly.
- **`Compare` operator semantics**: `Gt` is strictly greater, `Ge` is
  greater-or-equal. Mixing them is a common bug.
- **`CValidatorCombine` AND vs OR**: a single combine is AND across all
  `Require` entries. For OR, use nested combines or fall back to a
  `CValidatorFunction` calling Galaxy code.
- **`CValidatorUnitFilters` accepts both `Require` and `Exclude`**: use
  `Require` for inclusion and `Exclude` for exclusion. Mixing in the same
  `Filters` field doesn't work — split into two entries.
- **Effect-result validators are slow**: `CValidatorEffectCompare` runs
  the effect chain synchronously, so don't use it in per-tick hot paths.
- **Validators don't short-circuit**: a `CValidatorCombine` evaluates all
  entries even if the first one fails. For perf-critical paths, structure
  combines so the cheapest check is first.
- **`Find` subject for `CEffectSwitch`**: the case validators run with
  the current effect chain's subject as their `Find` default. Make sure
  the subject is what you expect — set `Find` explicitly if unsure.

## See also

- `catalog/fields-reference.md` §CValidator (subclass list)
- `catalog/effects.md` (CEffectSwitch, ValidatorArray usage)
- `catalog/targeting.md` (CTargetFind uses validators)
- `requirement/system.md` (CRequirementNode references validators)
