# Catalog Field Reference

This page lists every Catalog scope shipped by SC2 5.0.16 and the most-used
fields per scope. Use this as a navigational map: for exhaustive field
lists, open the matching Blizzard `.galaxy` declaration file under
`sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/<Scope>.galaxy`.

## How to read Catalog entries

Every Catalog entry is XML of the form:

```xml
<Catalog>
    <CUnit id="Marine">
        <FaceArray index="0" value="1"/>
        <CardLayouts index="0">
            <LayoutButtons index="0" Face="Stop" Type="AbilCmd" AbilCmd="Stop"/>
        </CardLayouts>
    </CUnit>
</Catalog>
```

- Each Catalog entry (`C<Scope id="...">`) inherits from a `parent`
  attribute (or from the implicit base class).
- Fields are either attributes (`id="Marine"`) or child elements
  (`<FaceArray index="0" value="1"/>`).
- Array fields use `index="0|1|2|..."` to address elements.
- Link fields (`unitLink`, `abilLink`, `behaviorLink`, etc.) reference
  other catalog entries by ID.
- Flags fields are comma-separated flag names from the matching `c_<scope>Flag*`
  constants.

## Complete scope list

SC2 5.0.16 ships 105 catalog scopes. Each has a `c_catalogScope<Name>`
constant in `natives.galaxy`. Below is the full list grouped by category:

### Core data scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CUnit` | `UnitData.xml` | Units (Marine, Barracks, etc.). |
| `CAbil` | `AbilData.xml` | Abilities (attack, build, train, cast, etc.). |
| `CBehavior` | `BehaviorData.xml` | Behaviors (buffs, debuffs, auras, veterancy). |
| `CEffect` | `EffectData.xml` | Effects (damage, heal, spawn, apply behavior, etc.). |
| `CWeapon` | `WeaponData.xml` | Weapons (range, period, damage-point). |
| `CUpgrade` | `UpgradeData.xml` | Upgrades (level-based stat modifications). |
| `CRequirement` | `RequirementData.xml` | Visibility / production gates. |
| `CValidator` | `ValidatorData.xml` | Boolean checks reused across catalog. |
| `CButton` | `ButtonData.xml` | UI button (icon, tooltip, hotkey). |
| `CMover` | `MoverData.xml` | Movement model (Fly, Ground, Hover, Missile). |
| `CFootprint` | `FootprintData.xml` | Pathing footprints. |
| `CTurret` | `TurretData.xml` | Turret (turnable weapon mount). |
| `CActor` | `ActorData.xml` | Visual actor (model, light, sound, etc.). |
| `CModel` | `ModelData.xml` | 3D model assets. |
| `CSound` | `SoundData.xml` | Sound assets. |
| `CSoundtrack` | `SoundtrackData.xml` | Music tracks. |
| `CVoiceOver` | `VoiceOverData.xml` | Narrative voice clips. |
| `CLight` | `LightData.xml` | Light actors. |
| `CReverb` | `ReverbData.xml` | Reverb presets. |
| `CSoundMixSnapshot` | `SoundMixSnapshotData.xml` | Saved mix state (per-channel volume). |
| `CCamera` | `CameraData.xml` | Camera presets. |

### Targeting scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CTargetFind` | `TargetFindData.xml` | Target acquisition (best point, rally, enum area). |
| `CTargetSort` | `TargetSortData.xml` | Target sorting (distance, priority, vital). |

### Tactical / AI scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CTactical` | `TacticalData.xml` | AI tactical hints (auto-cast, targeting). |
| `CTacCooldown` | `TacCooldownData.xml` | Tactical cooldown. |
| `CHerd` / `CHerdNode` | `HerdData.xml` / `HerdNodeData.xml` | Wildlife herd AI. |

### Hero / Campaign / Co-op scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CHero` | `HeroData.xml` | Hero definitions (see `hero-talent/system.md`). |
| `CHeroAbil` | `HeroAbilData.xml` | Hero ability variants. |
| `CHeroStat` | `HeroStatData.xml` | Named stat tracks. |
| `CTalent` | `TalentData.xml` | Talent tree entries. |
| `CTalentProfile` | `TalentProfileData.xml` | Talent tree layouts. |
| `CCommander` | `CommanderData.xml` | Co-op commanders (see `coop/commander-framework.md`). |
| `CCampaign` / `CCharacter` / `CLocation` / `CObjective` | `CampaignData.xml` / `CharacterData.xml` / `LocationData.xml` / `ObjectiveData.xml` | Campaign system (see `campaign/system.md`). |
| `CConversation` / `CConversationState` | `ConversationData.xml` / `ConversationStateData.xml` | Conversation data (see `cutscene/system.md`). |
| `CMutator` | `MutatorData.xml` | Mutator definitions (see `mutator/system.md`). |
| `CArmyCategory` / `CArmyUnit` / `CArmyUpgrade` | `ArmyCategoryData.xml` / `ArmyUnitData.xml` / `ArmyUpgradeData.xml` | Co-op army composition (UI). |

### UI / Score / Reward scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CGameUI` | `GameUIData.xml` | UI panel layouts. |
| `CAlert` | `AlertData.xml` | UI alert messages. |
| `CPing` | `PingData.xml` | Minimap pings. |
| `CScoreResult` / `CScoreValue` | `ScoreResultData.xml` / `ScoreValueData.xml` | End-game score. |
| `CReward` / `CBundle` / `CLoot` | `RewardData.xml` / `BundleData.xml` / `LootData.xml` | Reward systems. |
| `CAchievement` / `CAchievementTerm` | `AchievementData.xml` / `AchievementTermData.xml` | Achievements. |
| `CConsoleSkin` / `CSkin` / `CSkinPack` | `ConsoleSkinData.xml` / `SkinData.xml` / `SkinPackData.xml` | Console skins. |
| `CCursor` / `CSprayPack` / `CPortraitPack` / `CDecalPack` / `CEmoticonPack` / `CWarChest*` | various | Cosmetic unlocks. |

### Other scopes

| Scope | File | Purpose |
|-------|------|---------|
| `CRace` | `RaceData.xml` | Race definitions. |
| `CMap` | `MapData.xml` | Map metadata. |
| `CGame` | `GameData.xml` | Top-level game defaults. |
| `CFog` | `FoWData.xml` | Fog of war settings. |
| `CWater` / `CCliff` / `CCliffMesh` / `CShape` / `CTerrain*` / `CTile` | various | Terrain. |
| `CPhysicsMaterial` | `PhysicsMaterialData.xml` | Physics surfaces. |
| `CItem` / `CItemClass` / `CItemContainer` | `ItemData.xml` / `ItemClassData.xml` / `ItemContainerData.xml` | Items. |
| `CKinetic` | `KineticData.xml` | Physics kinetics (ragdoll, impact). |
| `CAttachMethod` | `AttachMethodData.xml` | Model attach points. |
| `CBeam` | `BeamData.xml` | Beam actors. |
| `CBoost` | `BoostData.xml` | Speed boosts (eg. stimpack). |
| `CBankCondition` | `BankConditionData.xml` | Bank-driven conditions. |
| `CError` | `ErrorData.xml` | Error messages. |
| `CGame` | `GameData.xml` | Global defaults. |
| `CLocation` (game) | `LocationData.xml` | Spawn locations. |
| `CMap` | `MapData.xml` | Map metadata. |
| `CMount` | `MountData.xml` | Hero mounts. |
| `CPreload` | `PreloadData.xml` | Preload hints. |
| `CPremiumMap` | `PremiumMapData.xml` | Premium marketplace maps. |
| `CTrophy` | `TrophyData.xml` | Trophy data. |
| `CUser` | `UserData.xml` | User data. |

For full scope enumeration see
`sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/` (105 `.galaxy` files,
one per scope).

---

## CUnit (full field list)

Source: `sc2-data-trigger/mods/core.sc2mod/base.sc2data/GameData/UnitData.xml`,
declarations: `TriggerLibs/GameData/Unit.galaxy`.

### Identity & race

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID, unique within Units scope. |
| `parent` | string | Inheritance parent. |
| `Name` | text | Display name (localized). |
| `Race` | string | Race: `Terr` / `Zerg` / `Prot`. |
| `EditorName` | string | Editor display name. |
| `EditorCategories` | string | Editor categorization. |
| `Footprint` | footprint link | Pathing footprint. |
| `PlacementPrevent` | string array | Placement restrictions. |

### Vitals & combat

| Field | Type | Description |
|-------|------|-------------|
| `LifeMax` / `LifeStart` / `LifeArmorName` | fixed / fixed / string | HP. |
| `ShieldsMax` / `ShieldsStart` | fixed | Protoss shields. |
| `EnergyMax` / `EnergyStart` | fixed | Caster energy. |
| `ShieldsRegenDelay` / `ShieldsRegenRate` | fixed / fixed | Shield regen. |
| `LifeRegenDelay` / `LifeRegenRate` | fixed / fixed | HP regen. |
| `EnergyRegenDelay` / `EnergyRegenRate` | fixed / fixed | Energy regen. |
| `LifeArmor` / `ShieldArmor` | int | Armor values. |
| `LifeArmorName` / `ShieldArmorName` | string | Display strings. |

### Movement

| Field | Type | Description |
|-------|------|-------------|
| `Speed` / `SpeedMin` / `SpeedMax` | fixed | Movement speed (min/max for randomization). |
| `Acceleration` / `Deceleration` | fixed | Linear accel. |
| `TurnRate` / `TurnSpeed` | fixed | Angular speed. |
| `TurnPitch` / `TurnRoll` | fixed | Banking. |
| `HeightBlendTime` | fixed | Height transition smoothing. |
| `Mover` | mover link | Movement model. |
| `MoverAcceleration` / `MoverDeceleration` | fixed | Mover overrides. |
| `MoverMaxSpeed` | fixed | Mover max speed. |
| `MoverTurnRate` | fixed | Mover turn rate. |
| `CargoSize` | int | Transport slot usage. |
| `CargoExtra` | int | Extra cargo (eg. hero). |

### Sight & detection

| Field | Type | Description |
|-------|------|-------------|
| `Sight` | fixed | Vision radius. |
| `SightBonus` | fixed | Vision bonus (eg. high-ground). |
| `DetectRange` | fixed | Detector range. |
| `KillRadius` | fixed | Kill credit radius. |

### Flags (`FlagArray`)

Each flag corresponds to a `c_unitFlag*` constant in `Unit.galaxy`. Set
multiple via comma-separated list:

- `Bounce`, `Turnable`, `Movable`, `Worker`, `CreateVisible`
- `Unclickable`, `Uncommandable`, `Unhighlightable`, `Untooltipable`
- `Unselectable`, `Untargetable`, `Uncursorable`
- `Hero`, `HiddenCargoUI`, `IndividualSubgroups`, `NoDraw`, `PreventReveal`
- `PreventDefeat`, `PreventDestroy`, `PenaltyRevealed`, `Uncloakable`
- `Missile`, `Undetectable`, `Unradarable`, `UseLineOfSight`, `KillCredit`
- `TownAlert`, `Invulnerable`, `Destructible`, `Cloaked`, `Buried`, `NoScore`
- `IgnoreTerrainZInit`, `TurnBeforeMove`, `AlwaysThreatens`, `NoDeathEvent`
- `NoPortraitTalk`, `TownCamera`, `AIThreatGround`, `AIThreatAir`
- `AILifetime`, `AISplash`, `AIHighPrioTarget`, `AISplitter`, `AIDefense`
- `AICaster`, `AISupport`, `AICantAddToWave`, `ShowResources`
- `ArmorDisabledWhileConstructing`, `Pawnable`, `AIFleeDamageDisabled`
- `AIPressForwardDisabled`, `AIObservatory`, `ForceCollisionCheck`
- `AIChangeling`, `ShareControl`, `BuiltOnOptional`, `AcquireRally`
- `AIAllowSuicideOverride`, `AIForceTactical`, `VisionTestCenterOnly`
- `Unstoppable`, `AIPreferBurrow`, `ClearRallyOnDeath`
- `ClearRallyOnTargetLost`, `SelectableWhileDead`, `TargetableWhileDead`
- `IgnoreAttackAlert`, `PreferLastAttackTarget`, `ResumeLastMoveOrder`
- `AIResourceBlocker`, `ArmySelect`, `Resistant`, `PlayerRevivable`
- `AIMakeIgnore`, `StatTrackAbilities`, `StatTrackDamageDone`
- `StatTrackDamageReceived`, `StatTrackCreation`, `CameraFollow`
- `LeechBehaviorShieldDamage`, `TownStructureWall`, `TownStructureGate`
- `TownStructureTownHall`, `TownStructureCannonTower`, `TownStructureMoonwell`
- `TownStructureCore`, `HideFromHarvestingCount`
- `TreatStructureAsUnitForSelection`

### Attributes (`Attributes` array)

`Armored`, `Biological`, `Light`, `Massive`, `Mechanical`, `Psionic`,
`Structure`, `Worker`, `Heroic`, `Summoned`, `Unkillable`.

### Abilities & weapons

| Field | Type | Description |
|-------|------|-------------|
| `CardAbilities` | ability link array | Abilities placed on the command card. |
| `Weapons` | weapon link array | Weapons this unit can use (Turret-indexed). |
| `Turrets` | turret link array | Turrets mounted on this unit. |
| `Behaviors` | behavior link array | Default behaviors applied on creation. |
| `TechTree` | upgrade link array | Upgrades that apply to this unit. |

### Cost & build

| Field | Type | Description |
|-------|------|-------------|
| `CostResource` | fixed array | Resource costs (`Minerals` / `Vespene` / `Time`). |
| `CostSupplies` | fixed | Supply cost. |
| `BuildTime` | fixed | Time to build. |
| `Requirements` | requirement link | Production gate. |
| `ProductionLink` | unit link | Reverse-link to producer (build panel). |
| `Producer` | unit link array | Buildings that produce this. |
| `BuiltOn` | unit link | Required host (eg. add-on). |
| `BuiltOnOptional` | bool | BuiltOn is optional. |
| `PlaceVars` | string | Placement variables. |
| `PlacementPrevent` | string array | Placement restrictions. |

### Model & UI

| Field | Type | Description |
|-------|------|-------------|
| `Model` | model link | Default model. |
| `Actor` | actor link | Primary actor. |
| `Button` | button link | Build button. |
| `Portrait` | model link | Portrait model. |
| `Minimap` | string | Minimap blip color. |
| `Score` | int | Score value. |
| `KillScore` | int | Kill score. |
| `SelectScore` | int | Select score. |
| `BuildScore` | int | Build score. |

### Veterancy

A `CUnit` does NOT contain veterancy info directly. Veterancy is a separate
`CBehaviorVeterancy` linked via `Behaviors`. See `unit/veterancy-and-xp.md`
(not yet written — use the Blizzard `BehaviorData.xml` entries with
`CBehaviorVeterancy` parent for examples).

---

## CAbil (full subclass list)

Source: `AbilData.xml`, declarations: `TriggerLibs/GameData/Abil.galaxy`.

### Subclasses

| Subclass | Purpose |
|----------|---------|
| `CAbilAttack` | Attack command (approach / attack / loiter stages). |
| `CAbilBuild` | Place a building (per-race structure list). |
| `CAbilBuildable` | Indicates this is a buildable unit. |
| `CAbilEffect` | Base for effect-casting abilities. |
| `CAbilEffectInstant` | Cast effect on self or point (no target). |
| `CAbilEffectTarget` | Cast effect on a target unit / point. |
| `CAbilBehavior` | Toggle behavior (Untoggled / Toggled stages). |
| `CAbilHarvest` | Harvest resources. |
| `CAbilInteract` | Interact with target (eg. mineral field). |
| `CAbilInventory` | Inventory management (open / close / drop). |
| `CAbilLearn` | Learn ability (talent tree / hero). |
| `CAbilMerge` | Merge two units (eg. Archon). |
| `CAbilMergeable` | Indicates this unit can be merged. |
| `CAbilMorph` | Morph between two unit types (eg. SiegeTank). |
| `CAbilMorphPlacement` | Morph with placement (eg. Viking land). |
| `CAbilMove` | Move command. |
| `CAbilPawn` | Pawn item. |
| `CAbilQueue` | Queue production. |
| `CAbilRally` | Set rally point. |
| `CAbilResearch` | Research an upgrade. |
| `CAbilRevive` | Revive a hero (Co-op). |
| `CAbilSpecialize` | Switch between variants (eg. BC Tactical Mode). |
| `CAbilStop` | Stop command. |
| `CAbilTrain` | Train a unit. |
| `CAbilTransport` | Load / unload transport. |
| `CAbilWarpable` | Warp-gate variant of Build. |
| `CAbilWarpTrain` | Warp-gate variant of Train. |
| `CAbilArmMagazine` | Arm magazine (eg. Carrier interceptors). |
| `CAbilAugment` | Augment another ability. |
| `CAbilBattery` | Battery (eg. Nexus Chrono Boost). |
| `CAbilBeacon` | Place a beacon (eg. Sensor Tower). |
| `CAbilRedirect` | Redirect to another ability (instant / target). |
| `CAbilQueueable` | Queueable effect-cast. |
| `CAbilProgress` | Progress display ability. |

### Common fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `CmdFlags` | flag array | Ability flags from `c_abilCmdFlag*` (AutoCastOn, AutoCastPlayerOff, AutoCastNeutral, AutoCastAlly, AutoCastEnemy, AutoCastHidden, ShowInEditor, Disabled, Passive, Cancelable, AutoRecord, NoResultAllowed, Cancellable, NoDisplay, IgnoreFacing, IgnoreRange, Offcooldown, Oncooldown, NotInScript, NoGenerator, NoDecay, NoKillCredit, AiAutoCastRepeat, NoRangeCheck, AiAllowCastOff, ToggleAutoCastOn, RequirePlacement, RequireRally, RequirePoint, AOEEffect, IgnoreAutoCast, AutoCastRemovable). |
| `CmdCardArray` | int array | Command card slot index. |
| `Buttons` | button link array | Per-command buttons. |
| `Cost` | cost array | Per-command resource cost. |
| `Effect` | effect link | Effect chain on cast. |
| `Range` | fixed | Cast range. |
| `RangeBuffer` | fixed | Range overshoot buffer. |
| `Cooldown` | fixed | Cooldown duration. |
| `AutoCastEnable` | bool | Auto-cast on by default. |
| `Requirements` | requirement link | Visibility / cast gate. |
| `TargetSort` | targetsort link | Targeting priority. |
| `TargetFilters` | filter array | What can be targeted (enemy / ground / air / etc.). |
| `ValidatorArray` | validator link array | Cast-time validators. |
| `Minimap` | bool | Show on minimap. |
| `Cursor` | cursor link | Cursor shown while targeting. |
| `CursorConfirmation` | cursor link | Cursor after target picked. |

### `CAbilEffectTarget` specific fields

| Field | Type | Description |
|-------|------|-------------|
| `TargetFilters` | filter array | Targetability filters. |
| `TargetSorts` | targetsort link array | Sort orders applied. |
| `TargetFind` | targetfind link | Target find logic. |
| `RangeIndex` | fixed array | Per-index ranges. |
| `EffectArray` | effect link array | Per-stage effect (Approach / Cast / Launch / Finish). |
| `EffectFlags` | flag array | Effect flags. |

### `CAbilEffectInstant` specific fields

| Field | Type | Description |
|-------|------|-------------|
| `Effect` | effect link | Effect chain to run. |
| `EffectFlags` | flag array | Effect flags. |

### `CAbilBehavior` specific fields (toggle abilities)

| Field | Type | Description |
|-------|------|-------------|
| `Effect` | effect link | Effect chain when toggled ON. |
| `EffectOff` | effect link | Effect chain when toggled OFF. |
| `Behavior` | behavior link | Behavior to apply/remove. |
| `Flags` | flag array | Behavior toggle flags. |

### `CAbilAttack` specific fields

| Field | Type | Description |
|-------|------|-------------|
| `Effect` | effect link | Effect chain on attack. |
| `AttackStage` | enum | Approach / Attack / Loiter (see `c_abilAttackStage*`). |
| `Range` | fixed | Attack range. |
| `RangeBuffer` | fixed | Range overshoot. |
| `AttackPriority` | priority link | Auto-acquire priority. |
| `TargetFilters` | filter array | What can be attacked. |
| `TargetSorts` | targetsort link array | Sort orders. |
| `WeaponArray` | weapon link array | Weapons used. |
| `OnlyFireWhenInAttackRange` | bool | Self-explanatory. |
| `IgnoreTargetPoint` | bool | Self-explanatory. |
| `AttackNotify` | trigger | Custom notify. |

### `CAbilTrain` / `CAbilResearch` / `CAbilBuild` specific fields

| Field | Type | Description |
|-------|------|-------------|
| `InfoArray` | info link array | Buildable items (units / upgrades / buildings). |
| `InfoIcon` | button link | Per-info button. |
| `BuildTime` | fixed | Per-info build time. |
| `ProduceUnit` | unit link | Per-info unit produced. |
| `ProduceUpgrade` | upgrade link | Per-info upgrade researched. |

---

## CBehavior (full subclass list)

Source: `BehaviorData.xml`, declarations: `TriggerLibs/GameData/Behavior.galaxy`.

### Subclasses

| Subclass | Purpose |
|----------|---------|
| `CBehaviorAttribute` | Attribute modifier (Light, Armored, etc.). |
| `CBehaviorBuff` | Generic buff/debuff with modification. |
| `CBehaviorClickResponse` | Triggers effect on unit click. |
| `CBehaviorConjoined` | Joins multiple units (eg. broodlord). |
| `CBehaviorCreepSource` | Generates creep. |
| `CBehaviorJump` | Jump movement behavior. |
| `CBehaviorPowerSource` | Provides power (Protoss). |
| `CBehaviorPowerUser` | Consumes power (Protoss). |
| `CBehaviorResource` | Resource drop-off. |
| `CBehaviorReveal` | Reveal area. |
| `CBehaviorSpawn` | Spawn subordinate units. |
| `CBehaviorVeterancy` | Veterancy/level (see `unit/veterancy-and-xp.md`). |
| `CBehaviorWander` | Wander AI. |

### Common fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Name` | string | Display name. |
| `Tooltip` | text | Tooltip text. |
| `Duration` | fixed | Duration in seconds (0 = permanent). |
| `Period` | fixed | Period for periodic effects. |
| `PeriodicEffect` | effect link | Effect fired each period. |
| `ExpirationEffect` | effect link | Effect fired when duration expires. |
| `InitialEffect` | effect link | Effect fired on application. |
| `StackCountMax` | int | Max stacks. |
| `StackFlags` | flag array | How stacking works. |
| `StackBehaviorReplace` | bool | Replace on re-apply. |
| `BehaviorCategories` | enum array | Categories (see `c_behaviorCategory*`). |
| `DisplayFlags` | flag array | UI display flags. |
| `Flags` | flag array | Behavior flags (Hidden, Permanent, Damaged, etc.). |
| `Modification` | modification | Stat changes (see below). |
| `InfoIcon` | button link | UI icon. |
| `InfoName` | text | UI name. |
| `InfoTooltip` | text | UI tooltip. |
| `InfoTooltipPriority` | int | Tooltip priority. |
| `Requirements` | requirement link | Visibility gate. |
| `RemoveValidatorArray` | validator link array | Auto-remove validators. |
| `Buffs` / `Debuffs` | behavior link array | Linked buffs. |

### Categories (`BehaviorCategories`)

`Permanent`, `Restorable`, `Temporary`, `Cloak`, `Invulnerable`, `Slow`,
`Fast`, `Stun`, `Reveal`, `User1`–`User15`.

### `Modification` sub-fields (the key field)

`Modification` is a complex sub-element. All available sub-fields (from
`BehaviorData.xml` `Modification` complex type):

#### Vital modifications

| Sub-field | Type | Description |
|-----------|------|-------------|
| `VitalMaxArray[Life\|Shields\|Energy]` | fixed | Max vital cap. |
| `VitalRegenArray[Life\|Shields\|Energy]` | fixed | Regen rate. |
| `VitalRegenDelayArray[Life\|Shields\|Energy]` | fixed | Regen delay. |
| `VitalArmorArray[Life\|Shields]` | int | Armor. |
| `VitalArmorFractionArray[Life\|Shields]` | fixed | Armor fraction. |
| `VitalArmorPenetrationArray[Life\|Shields]` | fixed | Armor penetration. |

#### Combat modifications

| Sub-field | Type | Description |
|-----------|------|-------------|
| `DamageDealtFraction` | fixed | Damage dealt multiplier. |
| `DamageDealtRange` | fixed array | Damage dealt range. |
| `DamageDealtAttribute[Light\|Armored\|Biological\|Mechanical\|Psionic\|Massive\|Structure\|Worker\|Heroic]` | fixed | Bonus damage to attribute. |
| `DamageDealtVital[Life\|Shields\|Energy]` | fixed | Bonus damage to vital. |
| `DamageDealtKind[Army\|Splash\|Damage\|Heal\|Recycle]` | fixed | Bonus damage to kind. |
| `DamageTakenFraction` | fixed | Damage taken multiplier. |
| `DamageTakenAttribute[...]` | fixed array | Same dimensions as `DamageDealtAttribute`. |
| `DamageTakenVital[...]` | fixed array | Same dimensions. |
| `DamageTakenKind[...]` | fixed array | Same dimensions. |
| `DamageSplashFraction` | fixed | Splash damage multiplier. |
| `DamageSplashOutgoingFraction` | fixed | Outgoing splash. |
| `DamageSplashIncomingFraction` | fixed | Incoming splash. |

#### Weapon modifications

| Sub-field | Type | Description |
|-----------|------|-------------|
| `WeaponCooldownFraction` | fixed | Attack speed multiplier (1 = normal, 0.5 = double speed). |
| `WeaponCooldownPercent` | fixed | Same as fraction. |
| `WeaponDamageFraction` | fixed | Weapon damage multiplier. |
| `WeaponRangeBonus` | fixed | Range bonus. |
| `WeaponRangeMultiplier` | fixed | Range multiplier. |
| `WeaponTargerPriorityBonus` | int | Target priority bonus. |

#### Movement modifications

| Sub-field | Type | Description |
|-----------|------|-------------|
| `MovementSpeedFraction` | fixed | Speed multiplier. |
| `MovementSpeedMultiplier` | fixed | Same. |
| `MovementSpeedAbsolute` | fixed | Set absolute speed. |
| `MovementSpeedMinimum` | fixed | Min speed. |
| `MovementSpeedMaximum` | fixed | Max speed. |
| `AccelerationMultiplier` | fixed | Accel multiplier. |
| `DecelerationMultiplier` | fixed | Decel multiplier. |
| `TurnRateMultiplier` | fixed | Turn rate multiplier. |
| `MovementDelayBonus` | fixed | Movement delay. |

#### Other modifications

| Sub-field | Type | Description |
|-----------|------|-------------|
| `SightBonus` | fixed | Sight radius bonus. |
| `DetectRangeBonus` | fixed | Detect range bonus. |
| `ResourceDropOffBonus` | fixed array | Drop-off bonus. |
| `SupplyCostBonus` | int | Supply cost adjustment. |
| `BuildTimeBonus` | fixed | Build time multiplier. |
| `BuildableExtraRadius` | fixed | Build radius bonus. |
| `CargoSizeDelta` | int | Cargo size change. |
| `FogVisibleTimeBonus` | fixed | Fog vision time. |
| `StatScaleFactor[...]` | fixed array | Generic stat scale. |

The same `Modification` complex type is used by `CBehaviorBuff` and
`CUpgrade.Upgrade.Bonus` (apply on upgrade research).

---

## CEffect (full subclass list)

Source: `EffectData.xml`, declarations: `TriggerLibs/GameData/Effect.galaxy`.

### Subclasses

| Subclass | Purpose |
|----------|---------|
| `CEffectDamage` | Deal damage to target. |
| `CEffectApplyBehavior` | Apply a behavior. |
| `CEffectRemoveBehavior` | Remove a behavior. |
| `CEffectTransferBehavior` | Transfer behavior between units. |
| `CEffectCreateUnit` | Spawn a unit (summon, projectile). |
| `CEffectLaunchMissile` | Launch a missile unit. |
| `CEffectRedirectMissile` | Redirect a missile. |
| `CEffectCreatePersistent` | Persistent AoE effect. |
| `CEffectCreateHealer` | Create healer link. |
| `CEffectDestroyHealer` | Destroy healer link. |
| `CEffectDestroyPersistent` | Destroy persistent effect. |
| `CEffectEnumArea` | Enumerate area units. |
| `CEffectEnumMagazine` | Enumerate magazine (eg. carrier interceptors). |
| `CEffectEnumTransport` | Enumerate transport cargo. |
| `CEffectEnumInventory` | Enumerate inventory items. |
| `CEffectIssueOrder` | Issue an order to target unit. |
| `CEffectCancelOrder` | Cancel current order. |
| `CEffectModifyPlayer` | Modify player state. |
| `CEffectModifyUnit` | Modify unit field. |
| `CEffectReleaseMagazine` | Release magazine units. |
| `CEffectReturnMagazine` | Return magazine units. |
| `CEffectUseCalldown` | Trigger a calldown. |
| `CEffectUseMagazine` | Use magazine units. |
| `CEffectSet` | Run multiple effects in sequence. |
| `CEffectSwitch` | Branch based on validators. |
| `CEffectTeleport` | Teleport a unit. |
| `CEffectApplyForce` | Apply force (kinetic). |
| `CEffectApplyKinetic` | Apply kinetic effect. |
| `CEffectRemoveKinetic` | Remove kinetic effect. |
| `CEffectUserData` | User-data effect (custom). |
| `CEffectResponse` | Effect response handler. |

### Common fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `Effect` | effect link | Next effect in chain (recursive). |
| `ValidatorArray` | validator link array | Conditions to check before running. |
| `Subject` | subject | Who is the target (Source, Caster, Target, Outer). |
| `LocationFilter` | filter | Spatial location filter. |
| `PlayerFilter` | filter | Player filter. |
| `EntityFilter` | filter | Entity type filter. |
| `Kind` | enum | Effect kind (Damage / Heal / Recycle / Splash / Army). |
| `Flags` | flag array | Effect flags (`Hidden`, `Effect2`, `Linkable`, etc.). |

### `CEffectDamage` specific

| Field | Type | Description |
|-------|------|-------------|
| `Amount` | fixed | Damage amount. |
| `AttributeBonus` | fixed array | Per-attribute bonus damage. |
| `AttributeTable` | bool | Use attribute bonus table. |
| `Behavior` | behavior link | Behavior to apply as part of damage. |
| `Death` | int | Death type (Normal, Splash, etc.). |
| `KillShare` | fixed | Kill share fraction. |
| `VitalFraction` | fixed | Fractional damage (eg. 0.1 = 10% max HP). |
| `VitalFractionMultiplier` | fixed | Multiplier on fractional damage. |
| `Fraction` | fixed | Damage as fraction of source's max vital. |
| `Kind` | enum | Damage kind. |
| `Loader` | string | Loader type. |
| `ResponseFlags` | flag array | Response flags. |
| `ShieldModifier` | fixed | Bonus to shields. |
| `SplashRadius` | fixed | Splash radius. |
| `SplashDamageFraction` | fixed | Fraction of damage that splashes. |
| `Tier` | int | Damage tier (for layered armor). |

### `CEffectApplyBehavior` specific

| Field | Type | Description |
|-------|------|-------------|
| `Behavior` | behavior link | Behavior to apply. |
| `Stacks` | int | Initial stacks. |
| `Duration` | fixed | Override duration. |
| `DurationArray` | fixed array | Per-tier duration override. |
| `Unit` | unit link | Override unit. |
| `ValidatorArray` | validator link array | Pre-application validators. |

### `CEffectCreateUnit` specific

| Field | Type | Description |
|-------|------|-------------|
| `Unit` | unit link | Unit to create. |
| `Count` | int | Number of units. |
| `Owner` | player | Owner (default = caster). |
| `SpawnOwner` | player | Spawn-time owner. |
| `Location` | point | Spawn location (or offset). |
| `Facing` | fixed | Initial facing. |
| `Offset` | point | Spawn offset. |
| `TimeScaleActive` | fixed | Time scale when active. |
| `TimeScaleInactive` | fixed | Time scale when inactive. |
| `InitialUnitState` | flag array | Initial state flags. |
| `PersistMinerals` | bool | Use mineral persistence. |

### `CEffectEnumArea` specific

| Field | Type | Description |
|-------|------|-------------|
| `AreaArray` | area | Area definitions. |
| `SearchFlags` | flag array | Search flags (eg. AllowEnemy, AllowAlly). |
| `MaxCount` | int | Max units to enumerate. |
| `Offset` | point | Offset from origin. |
| `ValidatorArray` | validator link array | Per-target validators. |
| `Effect` | effect link | Effect to apply per enumerated target. |

### `CEffectSwitch` specific

| Field | Type | Description |
|-------|------|-------------|
| `CaseArray` | case | Cases (validator + effect pairs). |
| `Default` | effect link | Default effect if no case matches. |
| `Else` | effect link | Else branch. |

### `CEffectSet` specific

| Field | Type | Description |
|-------|------|-------------|
| `EffectArray` | effect link array | Effects to run in sequence. |

### `CEffectModifyUnit` specific

| Field | Type | Description |
|-------|------|-------------|
| `Field` | string | Catalog field to modify (eg. `LifeMax`). |
| `Value` | string | New value. |
| `Unit` | unit link | Target unit. |

### `CEffectModifyPlayer` specific

| Field | Type | Description |
|-------|------|-------------|
| `Player` | player | Target player. |
| `Field` | string | Player field (eg. `Minerals`). |
| `Value` | string | New value. |

---

## CWeapon (full field list)

Source: `WeaponData.xml`, declarations: `TriggerLibs/GameData/Weapon.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Name` | string | Display name. |
| `Effect` | effect link | Effect chain on hit. |
| `EffectFlags` | flag array | Effect flags. |
| `Range` | fixed | Attack range. |
| `MinimumRange` | fixed | Min range (siege). |
| `RangeBuffer` | fixed | Range overshoot. |
| `Period` | fixed | Time between attacks. |
| `InitialPeriod` | fixed | First attack period. |
| `DamagePoint` | fixed | Windup before attack lands. |
| `DamagePointExchange` | fixed | Anim exchange for damage point. |
| `DamagePointPerRange` | fixed | Extra damage point per range. |
| `RefundThreshold` | fixed | Refund threshold for attack cancel. |
| `AcquireTime` | fixed | Time to acquire target. |
| `AcquireOrder` | int | Acquisition priority. |
| `AttackPriority` | priority link | Auto-acquire priority. |
| `Filters` | filter | Target filters (enemy, ground, air). |
| `Turrets` | turret link array | Required turrets. |
| `TurretOrigin` | point | Turret origin offset. |
| `Options` | flag array | Weapon flags (Slow, Splash, Blink, OnlyWhileInMoving, AllowMovement, DisableAutoAcquire, CanFireWhileMoving, NoDecay, NoKillCredit, NoAutoAttackReturn, etc.). |
| `ShowCounter` | bool | Show attack counter. |
| `Icon` | button link | UI icon. |
| `ValidatorArray` | validator link array | Attack-time validators. |
| `TargetSorts` | targetsort link array | Sort orders. |
| `TargetFind` | targetfind link | Target find logic. |
| `LegacyExchange` | string | Legacy data exchange. |
| `Minimap` | bool | Show on minimap. |
| `MoveTo` | point | Move to point. |
| `SpeedMultiplier` | fixed | Projectile speed multiplier. |
| `Kind` | enum | Weapon kind (eg. Artillery, Melee). |
| `Pricing` | pricing | Cost per attack. |

---

## CUpgrade (full field list)

Source: `UpgradeData.xml`, declarations: `TriggerLibs/GameData/Upgrade.galaxy`.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Name` | text | Display name. |
| `Level` | int | Maximum upgrade level. |
| `Minerals` | int array | Mineral cost per level. |
| `Vespene` | int array | Vespene cost per level. |
| `Time` | int array | Build time per level. |
| `Effect` | effect link | Effect chain on research (eg. apply modification). |
| `EffectArray` | effect link array | Per-level effects. |
| `ModifierArray` | modification array | Per-level stat modifications (alternative to Effect). |
| `Requirements` | requirement link | Visibility / research gate. |
| `BuildTime` | fixed array | Per-level build time. |
| `BuildCost` | cost array | Per-level cost. |
| `Score` | int | Score value. |
| `ScoreResult` | string | Score result ID. |
| `Icon` | button link | UI icon. |
| `Race` | string | Race (Terr / Zerg / Prot). |
| `Categories` | string array | Categories (eg. UpgradeTypeWeapon). |
| `Alert` | alert link | Alert on research. |
| `AlertCooldown` | alert link | Cooldown alert. |
| `ValidatorArray` | validator link array | Research validators. |
| `ProductionLink` | unit link | Producer link. |
| `Producer` | unit link array | Buildings that research. |
| `Flags` | flag array | Upgrade flags. |

`CUpgrade`'s `ModifierArray` uses the same Modification complex type as
`CBehaviorBuff` (see above). For each upgrade level, list the modification
that should be applied when the upgrade reaches that level.

---

## CButton (full field list)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `Name` | text | Tooltip title. |
| `Icon` | string | Icon path. |
| `Tooltip` | text | Tooltip body. |
| `Hotkey` | string | Default hotkey. |
| `Flags` | flag array | Button flags (eg. ClickDisabled, Cancelable, AutoCastable). |
| `Requirements` | requirement link | Button visibility gate. |
| `Alert` | alert link | Alert on click. |
| `Sister` | button link | Sister button (for toggle UI). |
| `Race` | string | Race. |
| `SortName` | text | Sort key. |
| `State` | enum | Default state (eg. Hidden, Disabled, Enabled). |
| `EditorSearchType` | string array | Editor search keywords. |

---

## CMover (full subclass list)

| Subclass | Purpose |
|----------|---------|
| `CMoverFly` | Flying movement. |
| `CMoverGround` | Ground movement. |
| `CMoverHover` | Hover movement (eg. Reaver). |
| `CMoverMissile` | Missile movement (projectile). |
| `CMoverBuild` | Building (no movement). |
| `CMoverNone` | No movement. |

Common fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `MoverType` | enum | Movement type. |
| `MaxSpeed` / `MinSpeed` | fixed | Speed range. |
| `Acceleration` / `Deceleration` | fixed | Accel. |
| `TurnRate` | fixed | Turn rate (radians/sec). |
| `BankRate` | fixed | Banking rate. |
| `PitchRate` | fixed | Pitch rate. |
| `BankScale` / `PitchScale` | fixed | Banking scales. |
| `HeightMin` / `HeightMax` | fixed | Height range (fly). |
| `HeightSmooth` | fixed | Height smoothing. |
| `Behavior` | behavior link | Behavior to apply. |
| `Properties` | flag array | Mover properties. |

For `CMoverMissile`:

| Field | Type | Description |
|-------|------|-------------|
| `MotionTopic` | string | Motion type. |
| `Acceleration` / `Deceleration` | fixed | Missile accel. |
| `MaxSpeed` / `MinSpeed` | fixed | Missile speed range. |
| `TurnRate` | fixed | Missile turn rate. |
| `Clearance` | fixed | Pathfinding clearance. |
| `Height` | fixed | Flight height. |
| `Align` | fixed | Alignment with terrain. |

---

## CRequirement (overview)

See `requirement/system.md` for the full system description.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Entry ID. |
| `parent` | string | Inheritance parent. |
| `NodeCatalog` | requirementnode array | Node tree (for building the logic). |
| `Flags` | flag array | Requirement flags. |
| `EditorCategories` | string | Editor categories. |

The `NodeCatalog` is a tree of `CRequirementNode` entries — see
`TriggerLibs/GameData/RequirementNode.galaxy` and `requirement/system.md`.

---

## CActor (overview)

Actor scope is very large (120+ subclasses — see
`sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/Actor.galaxy`
for the full list). See `actor/system.md` and `actor/messages.md` for the
actor system overview and message system.

---

## CTargetFind (full subclass list)

Source: `TargetFindData.xml`, declarations: `TargetFind.galaxy`.

| Subclass | Purpose |
|----------|---------|
| `CTargetFindBestPoint` | Find best point (eg. AoE center). |
| `CTargetFindRallyPoint` | Find rally point. |
| `CTargetFindWorkerRallyPoint` | Find worker rally point. |
| `CTargetFindEnumArea` | Enumerate area targets. |
| `CTargetFindEffect` | Find effect-relevant target. |
| `CTargetFindLastAttacker` | Find last attacker. |
| `CTargetFindOffset` | Find offset point from source. |
| `CTargetFindOrder` | Find order target. |
| `CTargetFindSet` | Set of target finds (for chained targeting). |

See `catalog/targeting.md` for the targeting system overview.

---

## CTargetSort (full subclass list)

Source: `TargetSortData.xml`, declarations: `TargetSort.galaxy`.

| Subclass | Purpose |
|----------|---------|
| `CTargetSortAlliance` | Sort by alliance (enemy first). |
| `CTargetSortAngle` | Sort by angle to source. |
| `CTargetSortBehaviorCount` | Sort by behavior stack count. |
| `CTargetSortBehaviorDuration` | Sort by behavior remaining duration. |
| `CTargetSortChargeCount` | Sort by ability charge count. |
| `CTargetSortChargeRegen` | Sort by charge regen rate. |
| `CTargetSortCooldown` | Sort by cooldown remaining. |
| `CTargetSortDistance` | Sort by distance to source. |
| `CTargetSortField` | Sort by catalog field. |
| `CTargetSortInterruptible` | Sort by interruptible state. |
| `CTargetSortMarker` | Sort by marker count. |
| `CTargetSortPowerSourceLevel` | Sort by power source level. |
| `CTargetSortPowerUserLevel` | Sort by power user level. |
| `CTargetSortPriority` | Sort by priority field. |
| `CTargetSortRandom` | Random sort. |
| `CTargetSortVeterancy` | Sort by veterancy level. |
| `CTargetSortVital` | Sort by vital (HP / shields / energy). |
| `CTargetSortVitalFraction` | Sort by vital fraction. |

See `catalog/targeting.md` for the targeting system overview.

---

## CValidator (full subclass list)

See `catalog/validators.md` for the full system description and per-subclass
field tables. The complete subclass list is at
`sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/GameData/Validator.galaxy`.

---

## CAction (trigger action)

Not a catalog scope — `CAction` is the Galaxy action descriptor generated by
the editor for each trigger action element. See `triggers/system.md`.

---

## See also

- `catalog/format.md` (catalog XML format basics)
- `catalog/effects.md` (CEffect system in depth)
- `catalog/validators.md` (CValidator system in depth)
- `catalog/targeting.md` (CTargetFind / CTargetSort system in depth)
- `actor/system.md`, `actor/messages.md`
- `requirement/system.md`
- `galaxy/native-index.md` (natives grouped by category)
- `unit/state-and-flags.md` (unit flag/state runtime API)
