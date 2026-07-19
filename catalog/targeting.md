# SC2 Targeting System

The targeting system (`CTargetFind` + `CTargetSort`) decides what a
unit auto-acquires as a target, and in what order. Abilities and weapons
reference these to drive auto-cast, auto-attack, and AoE enumeration
ordering.

Sources:
- `sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/TargetFind.galaxy`
- `sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/TargetSort.galaxy`

## CTargetFind

A `CTargetFind` entry defines how to find a target. Subclasses:

| Subclass | Purpose |
|----------|---------|
| `CTargetFindBestPoint` | Find the best point for an AoE (eg. maximize enemy count). |
| `CTargetFindRallyPoint` | Find a rally point for production buildings. |
| `CTargetFindWorkerRallyPoint` | Find a worker rally point (eg. near mineral line). |
| `CTargetFindEnumArea` | Enumerate targets in an area. |
| `CTargetFindEffect` | Find a target relevant to an effect. |
| `CTargetFindLastAttacker` | Find the unit that last attacked the source. |
| `CTargetFindOffset` | Find a point offset from source. |
| `CTargetFindOrder` | Find the order target (current order's target). |
| `CTargetFindSet` | Composite — runs multiple finds in sequence. |

### Common fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Subject` | enum | Who is searching (Source / Caster / Target / Outer). |
| `Filters` | filter | Unit filters (enemy / ally / ground / air / etc.). |
| `SearchFlags` | flag array | Search options (AllowCorpse, RequireVisible, etc.). |
| `MaxCount` | int | Maximum targets to find (0 = unlimited). |
| `SearchRadius` | fixed | Search radius. |
| `MaxRadius` | fixed | Max radius. |
| `MinRadius` | fixed | Min radius (ring search). |
| `Sorts` | targetsort link array | Sort orders applied. |
| `ValidatorArray` | validator link array | Per-target validators. |
| `Effect` | effect link | Effect to run on found targets. |

### `CTargetFindBestPoint`

For AoE abilities that want to auto-target the densest enemy cluster:

```xml
<CTargetFindBestPoint id="FindBestAoECenter">
    <SearchRadius value="8"/>
    <Filters value="Enemy,Ground"/>
    <MaxCount value="1"/>
    <BestPointRadius value="3"/>  <!-- The AoE radius to optimize for -->
    <BestPointMaxResults value="16"/>  <!-- Max candidates to evaluate -->
</CTargetFindBestPoint>
```

Returns a point (not a unit) — best used for point-targeted abilities
like Psionic Storm.

### `CTargetFindEnumArea`

Enumerate all units matching filters in an area. Used by auto-cast
heals, AoE auras, and persistent effect search:

```xml
<CTargetFindEnumArea id="FindNearbyAllies">
    <SearchRadius value="5"/>
    <Filters value="Ally,Organic"/>
    <SearchFlags value="AllowSelf"/>
    <Sorts value="SortByLowestHP"/>  <!-- CTargetSortVital -->
    <MaxCount value="5"/>
    <ValidatorArray index="0" value="NeedsHealingValidator"/>
    <Effect value="HealEffect"/>
</CTargetFindEnumArea>
```

### `CTargetFindEffect`

Re-targets based on an effect's result. Useful for chain effects (eg.
Lightning Chain):

```xml
<CTargetFindEffect id="FindNextChainTarget">
    <Effect value="PreviousChainHitEffect"/>
    <Filters value="Enemy,Organic"/>
    <SearchRadius value="6"/>
    <MaxCount value="1"/>
    <ExcludedSubjects value="AlreadyHitSubjects"/>  <!-- subject set -->
</CTargetFindEffect>
```

### `CTargetFindSet`

Composite — runs multiple finds in sequence and combines results:

```xml
<CTargetFindSet id="FindPrimaryOrSecondary">
    <FindArray index="0" value="FindPrimaryTarget"/>
    <FindArray index="1" value="FindSecondaryTarget"/>
</CTargetFindSet>
```

## CTargetSort

A `CTargetSort` defines how to sort found targets. Multiple sorts can be
chained — first sort dominates, ties are broken by next sort, etc.

Subclasses:

| Subclass | Sorts by |
|----------|---------|
| `CTargetSortAlliance` | Alliance (enemies first, then allies, then neutral). |
| `CTargetSortAngle` | Angle to source. |
| `CTargetSortBehaviorCount` | Behavior stack count. |
| `CTargetSortBehaviorDuration` | Behavior remaining duration. |
| `CTargetSortChargeCount` | Ability charge count. |
| `CTargetSortChargeRegen` | Charge regen rate. |
| `CTargetSortCooldown` | Cooldown remaining. |
| `CTargetSortDistance` | Distance to source. |
| `CTargetSortField` | Generic catalog field. |
| `CTargetSortInterruptible` | Interruptible state. |
| `CTargetSortMarker` | Marker count (eg. how many times tagged). |
| `CTargetSortPowerSourceLevel` | Power source level. |
| `CTargetSortPowerUserLevel` | Power user level. |
| `CTargetSortPriority` | Priority field. |
| `CTargetSortRandom` | Random order. |
| `CTargetSortVeterancy` | Veterancy level. |
| `CTargetSortVital` | Vital (HP / shields / energy). |
| `CTargetSortVitalFraction` | Vital fraction (current / max). |

### Common fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Subject` | enum | Source for sort comparisons (Source / Caster / Target / Outer). |
| `Filters` | filter | Unit filters. |
| `Ascending` | bool | Sort ascending (false = descending). |
| `Iceland` | int | Iceland (tie-breaker level). |

### Sort chaining

Multiple sorts combine via a `Sorts` array on `CTargetFind` or `CAbil`:

```xml
<CAbilAttack id="SmartAutoAttack">
    <TargetSorts index="0">
        <SortArray index="0" Sort="SortByClosestEnemy"/>     <!-- Primary: distance -->
        <SortArray index="1" Sort="SortByLowestHP"/>          <!-- Tiebreak: low HP -->
        <SortArray index="2" Sort="SortByHighestThreat"/>     <!-- Tiebreak: threat -->
    </TargetSorts>
</CAbilAttack>
```

Each sort runs after the previous one — so the result is sorted by
`SortByClosestEnemy`, then ties broken by `SortByLowestHP`, then ties
broken by `SortByHighestThreat`.

## How targeting flows in an ability

When a unit auto-casts or auto-attacks:

1. **Filter**: apply `TargetFilters` to find candidate units.
2. **Sort**: apply `TargetSorts` to order candidates.
3. **Validate**: run `ValidatorArray` on each candidate (top-down).
4. **Acquire**: pick first candidate that passes validators.

For manual casts (player picks a target), only Filter and Validate
apply — the player's pick wins.

## Target filters

Filters use a set of flag constants from `c_unitFilter*`:

- **Alliance**: `Enemy` / `Ally` / `Neutral` / `Self` (one required).
- **Plane**: `Ground` / `Air` / `Structure` (one or both).
- **Attribute**: `Light` / `Armored` / `Biological` / `Mechanical` /
  `Psionic` / `Massive` / `Heroic` / `Worker` / `Summoned` / `Unkillable`.

A filter is a comma-separated list. Multiple attributes are ANDed (must
match all); alliance and plane are independent filters (must match
at least one in each category).

```xml
<!-- Targets enemies that are ground, biological, light or armored -->
<Filters value="Enemy,Ground,Biological"/>
```

For more complex filter logic, use `CValidatorUnitFilters` instead — it
supports both `Require` and `Exclude` arrays.

## CAbil AttackPriority

`CAbilAttack.AttackPriority` references a `CPriority` entry that defines
auto-acquire priority. The priority entry has `Bonus` per-attribute
entries (eg. +10 vs Armored) that bias auto-targeting toward certain
unit types.

```xml
<CPriority id="AntiAirPriority">
    <BonusArray index="0" Bonus="20" Attribute="Air"/>
    <BonusArray index="1" Bonus="10" Attribute="Armored"/>
    <BonusArray index="2" Bonus="-10" Attribute="Worker"/>
</CPriority>
```

Higher bonus = more likely to auto-target. Negative = avoid.

## CWeapon filters and auto-acquire

Each `CWeapon` has its own `Filters` and `AttackPriority`. These combine
with the unit's `CAbilAttack` for auto-attack:

- The unit's `CAbilAttack` decides WHEN to attack.
- Each `CWeapon` decides WHICH weapon to fire based on target filters
  (eg. ground weapon vs ground targets, air weapon vs air targets).

## Common pitfalls

- **`MaxCount = 0` is unlimited**: if you forget to set `MaxCount`, an
  enum-area can enumerate the whole map. Always set a sane max.
- **`Sorts` order matters**: the first sort dominates. If you put
  `SortByRandom` first, the rest of the sorts are useless.
- **Filters are AND-attribute, OR-alliance**: a filter `Enemy,Ground`
  targets enemies that are on the ground. To target either ground OR
  air, use `Enemy,Ground,Air` — alliance is OR, plane is OR.
- **`CTargetFindBestPoint` returns a point, not a unit**: it can't be
  used as a target for unit-targeted abilities. Use it only for
  point-targeted abilities.
- **`CTargetFindEffect` ExcludedSubjects**: chain effects need to
  track which units have already been hit. The `ExcludedSubjects`
  field references a subject set (created by `CEffectSet` with the
  `AddToSubjectSet` flag).
- **`CTargetSortVeterancy` requires veterancy behavior**: only works
  if the unit has `CBehaviorVeterancy`. Without it, sorts treat veterancy
  as level 0.
- **Validator passes per-target**: in `CTargetFindEnumArea`, validators
  run on each enumerated target. Slow validators in this slot are
  multiplied by target count — use cheap checks.
- **AttackPriority doesn't replace TargetSorts**: priority biases
  auto-acquire toward certain targets, but `TargetSorts` orders the
  candidates first. Use both for best results.

## See also

- `catalog/fields-reference.md` §CTargetFind / §CTargetSort
- `catalog/effects.md` (CEffectEnumArea uses CTargetFind internally)
- `catalog/validators.md` (ValidatorArray on TargetFind)
- `requirement/system.md` (Requirements gate visibility, not targeting)
