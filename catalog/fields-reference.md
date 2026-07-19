# Catalog Field Reference

Quick reference for the most-used fields on the most-used Catalog scopes. For
exhaustive field lists, consult the matching XML file under
`catalog/reference/` (Blizzard assets) or the matching `.galaxy` file under
`galaxy/natives/catalog/` (native declarations).

## CUnit

Source: `catalog/reference/UnitData.xml`, `galaxy/natives/catalog/Unit.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID, unique within the Units scope. |
| `parent` | string | Inheritance parent unit ID. |
| `Name` | text | Display name (localized). |
| `Race` | string | Race (`Terr`, `Zerg`, `Prot`). |
| `Radius` | fixed | Collision radius. |
| `InnerRadius` | fixed | Inner collision radius (push-out). |
| `LifeMax` | fixed | Maximum HP. |
| `LifeStart` | fixed | HP at creation (usually equals LifeMax). |
| `ShieldsMax` | fixed | Maximum shields (Protoss). |
| `EnergyMax` | fixed | Maximum energy (casters). |
| `EnergyStart` | fixed | Initial energy. |
| `Speed` | fixed | Movement speed. |
| `Acceleration` | fixed | Acceleration. |
| `Deceleration` | fixed | Deceleration. |
| `TurnRate` | fixed | Turn speed (radians/sec). |
| `Sight` | fixed | Sight radius. |
| `FlagArray` | int array | Flags from `EUnitFlag` (see `Unit.galaxy` `c_unitFlag*`). |
| `Attributes` | int array | Light/Biological/Armored/Massive/Psionic/Mechanical/Structure/Worker/Heroic. |
| `CardAbilities` | ability link array | Abilities placed on the command card. |
| `Weapons` | weapon link array | Weapons this unit can use. |
| `Behaviors` | behavior link array | Default behaviors applied on creation. |
| `EditorCategories` | string | Editor categories for grouping. |
| `Race` | string | Race ID. |
| `Requirements` | requirement link | Production gate. |
| `CostResource` | fixed array | Mineral/Vespere/Time cost array. |
| `Footprint` | footprint link | Pathing footprint. |
| `Model` | model link | Default model. |
| `Actor` | actor link | Primary actor for this unit. |
| `Button` | button link | Build button. |
| `Score` | int | Score value. |

## CAbil (ability)

Source: `catalog/reference/AbilData.xml`, `galaxy/natives/catalog/Abil.galaxy`.

`CAbil` is the base; subclasses `CAbilBuild`, `CAbilEffect`, `CAbilTrain`,
`CAbilResearch`, `CAbilAttack`, `CAbilMove`, `CAbilStop`, `CAbilHoldPosition`,
`CAbilPatrol`, `CAbilAcquire`, etc.

Common fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `CmdFlags` | int array | Ability flags (auto-cast, hostile, neutral, etc.). |
| `CmdCardArray` | int array | Command card slot. |
| `Buttons` | button link array | Per-command buttons. |
| `Cost` | cost array | Mineral/Vespene/Time cost per command. |
| `Effect` | effect link | Effect chain triggered on cast. |
| `Range` | fixed | Cast range. |
| `Cooldown` | fixed | Cooldown duration. |
| `AutoCastEnable` | bool | Whether auto-cast is on by default. |
| `Requirements` | requirement link | Visibility / cast gate. |

`CAbilEffect` is the generic "cast effect" ability. `CAbilTrain` builds units,
`CAbilResearch` researches upgrades, `CAbilBuild` places a building.

## CWeapon

Source: `catalog/reference/WeaponData.xml`, `galaxy/natives/catalog/Weapon.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Parent` | string | Inheritance parent. |
| `Effect` | effect link | Effect chain fired on hit. |
| `Range` | fixed | Attack range. |
| `Period` | fixed | Time between attacks. |
| `InitialPeriod` | fixed | First attack period (often longer). |
| `DamagePoint` | fixed | Windup before attack lands. |
| `DamagePointExchange` | fixed | Animation timing point. |
| `MinimumRange` | fixed | Minimum range (for siege units). |
| `Editor` | string | Editor display name. |
| `Filters` | filter | Target filters (enemy, ground, air, etc.). |
| `Tenders` | turret link | Turret required to fire. |
| `Options` | int array | Weapon flags (slow, splash, blink, etc.). |

## CEffect

Source: `catalog/reference/EffectData.xml`, `galaxy/natives/catalog/Effect.galaxy`.

`CEffect` is the base; subclasses include:

- `CEffectDamage` — deals damage.
- `CEffectCreateUnit` — spawns a unit (projectiles, summon).
- `CEffectSetBehavior` — applies or removes a behavior.
- `CEffectEnumArea` / `CEffectEnumSearch` — area and search enumerators for AoE.
- `CEffectSwitch` — branches based on a validator.
- `CEffectLaunchUnit` — launches a unit (projectile parent).
- `CEffectDamageSet` — modifies damage amount for chained effects.

Common fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Effect` | effect link | Next effect in chain. |
| `Amount` | fixed (Damage) | Damage amount (for `CEffectDamage`). |
| `Behavior` | behavior link (SetBehavior) | Behavior to apply (for `CEffectSetBehavior`). |
| `Count` | int | Number of effects to apply. |
| `SearchFilter` | filter | Target filters for enumeration. |
| `Validators` | validator link array | Conditions to check before applying. |

## CBehavior

Source: `catalog/reference/BehaviorData.xml`, `galaxy/natives/catalog/Behavior.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Parent` | string | Inheritance parent. |
| `Duration` | fixed | Duration in seconds (0 = permanent). |
| `MaxStacks` | int | Maximum stacks (e.g. stimpack, broodling speed). |
| `Modification` | modification | Stat changes (damage, armor, speed, etc.). |
| `Period` | fixed | Period for `BehaviorEnumPeriodic` effects. |
| `PeriodicEffect` | effect link | Effect to fire each period. |
| `Flags` | int array | Behavior flags (positive, negative, hidden, etc.). |

Subclasses: `CBehaviorBuff`, `CBehaviorUnit`, `CBehaviorStacking`, etc.

## CUpgrade

Source: `catalog/reference/UpgradeData.xml`, `galaxy/natives/catalog/Upgrade.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Level` | int | Maximum upgrade level (1 for normal). |
| `Effect` | effect link | Effect chain to execute on research. |
| `Minerals` | int array | Mineral cost per level. |
| `Vespene` | int array | Vespene cost per level. |
| `Time` | int array | Build time per level. |
| `Requirements` | requirement link | Visibility / research gate. |

See `requirement/system.md` for the upgrade-requirement relationship.

## CActor

Source: `catalog/reference/ActorData.xml`, `galaxy/natives/catalog/Actor.galaxy`.

Actor is a large scope with many subclasses. See `actor/system.md` for the
system overview.

## CButton

Source: `catalog/reference/ButtonData.xml`, `galaxy/natives/catalog/Button.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Name` | text | Tooltip title. |
| `Icon` | string | Icon path. |
| `Tooltip` | text | Tooltip body. |
| `Hotkey` | string | Default hotkey. |
| `Flags` | int array | Button flags. |
| `Requirements` | requirement link | Button visibility gate. |

## CMover

Source: `catalog/reference/MoverData.xml`, `galaxy/natives/catalog/Mover.galaxy`.

Movers describe unit movement models: `Fly`, `Ground`, `Hover`, `Missile`,
`Build`, etc.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `MoverType` | enum | Movement type (Fly, Ground, ...). |
| `MaxSpeed` | fixed | Maximum speed. |
| `MinSpeed` | fixed | Minimum speed. |
| `Acceleration` | fixed | Acceleration. |
| `Deceleration` | fixed | Deceleration. |
| `TurnRate` | fixed | Turn rate (radians/sec). |

## CValidator

Source: `catalog/reference/ValidatorData.xml`, `galaxy/natives/catalog/Validator.galaxy`.

Validators are boolean checks reusable in effects, requirements, and abilities.

Common subclasses:
- `CValidatorUnitCompareVital` — compare HP / energy / shields.
- `CValidatorUnitCompareBehaviorCount` — check behavior stacks.
- `CValidatorLocationRange` — distance check.
- `CValidatorUnitTypeMatch` — unit type match.

## See also

- `catalog/format.md`
- `catalog/reference/*.xml` (Blizzard assets)
- `galaxy/natives/catalog/*.galaxy` (native declarations per scope)
- `requirement/system.md`
- `actor/system.md`
