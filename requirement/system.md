# Requirement, Upgrade, and Tech Tree

The tech-tree gating system: Requirements decide what units/abilities/upgrades a
player can use or research; Upgrades modify other catalog entries' fields when
researched.

## Source references

- `galaxy/natives/catalog/Requirement.galaxy`: native function declarations.
- `galaxy/natives/catalog/Upgrade.galaxy`: upgrade natives.
- `catalog/reference/RequirementData.xml`: full official Requirement catalog.
- `catalog/reference/UpgradeData.xml`: full official Upgrade catalog.

## Requirement (`CRequirement`)

A Requirement is a boolean expression over `Node` entries. Each `CRequirement`
has a tree of nodes (`CRequirementNode`):

- `CRequirementCountUnits` — count units owned by a player.
- `CRequirementCountBehaviors` — count behaviors on units.
- `CRequirementCountUpgrade` — count upgrades researched.
- `CRequirementCountAbilCmd` — count abilities on command card.
- `CRequirementLogic` — `And`, `Or`, `Not`, `Equals`, `GreaterThan`, etc.

`NodeArray` field references `Node` entries by their `id`. The root node's result
is the requirement's effective boolean.

## Use in Catalog

- `CUnit.Requirements` — gates whether the unit can be produced (visible on the
  build panel, and produces / fails the order).
- `CAbility.Requirements` — gates whether the ability appears / casts.
- `CButton.Requirements` — gates whether the button is enabled.
- `CUpgrade.Requirements` — gates whether the upgrade can be researched.

`Flags` distinguishes:
- `VisibleWhenRequirementNotMet` — show but greyed out.
- `HiddenWhenRequirementNotMet` — hide entirely.

## Upgrade (`CUpgrade`)

An Upgrade is a researchable tech-tree entry that modifies catalog values when
researched. Each `CUpgrade` declares:

- `Level` — max level (1 for normal upgrades, multiple for ranks).
- `Effect` — effect chain to execute on research.
- `Cost` per level (`Minerals`, `Vespene`, `Time`).
- `Data` changes — per-level value changes applied to other catalog entries.

### Field modification via `CAffect`

When an Upgrade is researched at level N, the engine applies every `CAffect`
referenced by the upgrade. Each `CAffect` modifies a specific field of a specific
entry:

```xml
<CUpgrade id="TerranInfantryWeaponsLevel1">
    <Effect value="TerranInfantryWeaponsUpgradeSet"/>
</CUpgrade>

<CUpgradeSet id="TerranInfantryWeaponsUpgradeSet">
    <AmountArray index="0" Amount="1"/>
    <!-- more levels... -->
</CUpgradeSet>
```

Internally, a `CUpgradeSet` carries per-level amounts. Each `CAffectDamageDealt`
(or similar) references the `CUpgradeSet` and applies its amounts to a specific
weapon/effect field.

## Tech tree patterns

### Production gate

```xml
<CUnit id="Factory">
    <Requirements>
        <Value arrayIndex="0" value=" BarracksExists"/>
    </Requirements>
</CUnit>

<CRequirement id="BarracksExists">
    <NodeArray index="Use">
        <Value value=" BarracksCount"/>
    </NodeArray>
</CRequirement>

<CRequirementCountUnits id="BarracksCount">
    <UnitLink value="Barracks"/>
    <Count value="1"/>
</CRequirementCountUnits>
```

### Tier-gated research

```xml
<CUpgrade id="Stimpack">
    <Requirements>
        <Value arrayIndex="0" value=" BarracksTechLabExists"/>
    </Requirements>
</CUpgrade>
```

### Multi-level upgrade

A `CUpgrade` with `Level="3"` modifies the same field three times. Each level's
value comes from a `CUpgradeSet` indexed 0..2.

## Pitfalls

- Requirement nodes are referenced by `id` (string) and merged like other catalog
  entries. Removing a node without updating references breaks the requirement
  silently.
- `HiddenWhenRequirementNotMet` hides the button entirely, which can hide the
  production building from the build panel. This is the most common cause of
  "missing unit on build panel" issues.
- Upgrades do not re-evaluate requirements after research; you cannot gate
  `CUpgrade X` on `CUpgrade Y researched` via the Requirements field alone if
  both are researched in the same tick. Use a `CRequirementLogic` node and a
  1-frame delay.
- 7vs1 campaign maps have additional tech-tree triggers that may override these
  rules; see `document-structure/overview.md`.

## See also

- `catalog/format.md`
- `catalog/reference/RequirementData.xml`
- `catalog/reference/UpgradeData.xml`
- `galaxy/natives/catalog/Requirement.galaxy`
- `galaxy/natives/catalog/Upgrade.galaxy`
