# SC2 Effect System

Effects (`CEffect`) are the workhorse of SC2 ability design. They form
directed acyclic graphs (DAGs) that describe "what happens" when an
ability is cast, a weapon hits, a behavior ticks, or an upgrade is
researched. Effects chain together via the `Effect` link field.

This page describes the effect system in depth. For the field list per
subclass, see `catalog/fields-reference.md` §CEffect.

## Effect chain pattern

A typical ability is built like this:

```
CAbilEffectTarget
    Effect: MyAbilityInitial  (CEffectSet or CEffectDamage)
                ↓
        CEffectDamage  (direct damage to primary target)
                ↓
        CEffectEnumArea  (search nearby enemies)
                ↓
        CEffectApplyBehavior  (apply debuff to each found)
```

Each effect runs in sequence. The chain ends when an effect has no `Effect`
link set, or when the chain is broken by a `CEffectSwitch` with no matching
case.

## Subject vs Source vs Caster vs Target

Effects operate on a "subject" — the unit / player / location the effect
applies to. Effects distinguish:

- **Source**: the unit that originated the chain (e.g. the caster). Stays
  the same throughout the chain.
- **Caster**: same as Source in most cases. For chained effects (eg. a
  projectile launched by an effect), the Caster becomes the projectile.
- **Target**: the current target. Each effect in the chain can change the
  Target for the next effect (eg. `CEffectEnumArea` enumerates units and
  sets each as the Target of the next effect).
- **Outer**: the outer effect's Target (eg. for nested `CEffectSet`, the
  outer's Target is passed to inner effects).

Configure via the `Subject` field on each effect. Default is `Target`.

## Effect kinds

Each effect has a `Kind` field:

- `Damage` — deals damage.
- `Heal` — heals (negative damage to allied vitals).
- `Recycle` — recycles resources.
- `Splash` — splash damage (uses `SplashRadius` and `SplashDamageFraction`).
- `Army` — army-wide effect.

`Kind` is metadata — it doesn't change runtime behavior, but the engine
uses it for damage attribution, score tracking, and UI.

## Effect response flags

`ResponseFlags` on `CEffectDamage` controls how the target responds:

- `Stun` — apply stun response behavior.
- `Knockback` — apply knockback.
- `Flinch` — apply flinch.
- `Death` — apply death response.
- `Invulnerable` — invulnerable check.
- `Touch` — touch response.
- `Redirected` — already redirected.

These flags map to a `CEffectResponse` catalog entry that defines the
actual response behavior to apply.

## CEffectDamage deep dive

### Direct damage

```xml
<CEffectDamage id="MyDamage">
    <Amount value="20"/>
    <Effect value="MyFollowupEffect"/>
    <Kind value="Damage"/>
    <Death value="Normal"/>
    <ResponseFlags index="0" value="Flinch"/>
</CEffectDamage>
```

### Fractional damage

For "10% of max HP" patterns:

```xml
<CEffectDamage id="PercentDamage">
    <Amount value="0"/>
    <VitalFraction index="Life" value="0.1"/>
    <VitalFractionMultiplier value="1"/>
</CEffectDamage>
```

`VitalFraction` is indexed by vital type (`Life` / `Shields` / `Energy`).
`VitalFractionMultiplier` scales the fraction.

### Splash damage

```xml
<CEffectDamage id="SplashDamage">
    <Amount value="50"/>
    <SplashRadius value="2"/>
    <SplashDamageFraction value="0.5"/>
</CEffectDamage>
```

This deals 50 damage to the primary target, 25 damage (50 × 0.5) to
everything within 2 radius.

### Attribute bonus damage

```xml
<CEffectDamage id="AntiArmorDamage">
    <Amount value="10"/>
    <AttributeBonus index="Armored" value="10"/>
</CEffectDamage>
```

Deals 10 base + 10 bonus against Armored targets.

### Death type

`Death` controls the death animation / response:

- `Normal` — standard death.
- `Splash` — splash death.
- `Blast` — explosive death.
- `Fire` — burning death.
- `Disintegrate` — disintegration.
- `Cleave` — cleaving.
- `Critical` — critical death.
- `Suicide` — suicide death.

The unit's actor selects the death animation based on this value.

## CEffectEnumArea deep dive

`CEffectEnumArea` is the engine's AoE / search mechanism. It enumerates
units matching filters within an area and applies an effect to each.

```xml
<CEffectEnumArea id="AoEDamage">
    <AreaArray index="0" Radius="2.0"/>
    <SearchFlags index="0" value="AllowEnemy"/>
    <MaxCount value="10"/>
    <ValidatorArray index="0" value="NotHeroicValidator"/>
    <Effect value="MyDamage"/>
</CEffectEnumArea>
```

- `AreaArray` — list of radius/shape pairs to search. Multiple areas can
  be combined (eg. inner circle + outer ring with different effects).
- `SearchFlags` — `AllowEnemy` / `AllowAlly` / `AllowSelf` / `AllowNeutral`
  / `AllowCorpse` / `AllowMissile` etc.
- `MaxCount` — limit number of targets. 0 = no limit.
- `ValidatorArray` — per-target validators. Skips units that fail.
- `Effect` — applied to each enumerated unit, with that unit as Target.

### Search flags

From `c_effectSearchFlag*` in `natives.galaxy`:

- `AllowEnemy` / `AllowAlly` / `AllowSelf` / `AllowNeutral` / `AllowCorpse`
- `AllowMissile` / `AllowHidden` / `AllowDead` / `AllowPreplaced`
- `RequireVisible` / `RequireDetected` / `RequireInside`
- `ExcludeAir` / `ExcludeGround` / `ExcludeStructural`
- `Behind` / `InFront` — spatial filters relative to source facing.

## CEffectSwitch deep dive

`CEffectSwitch` branches the effect chain based on validators:

```xml
<CEffectSwitch id="BranchByTarget">
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

The first matching case's effect runs; if none match, `Default` runs.
This is the standard way to implement conditional ability behavior (eg.
different damage to different unit types, or bonus damage when target is
stunned).

## CEffectCreatePersistent deep dive

`CEffectCreatePersistent` creates a persistent effect that ticks over
time. It's the basis for AoE auras, debuff clouds, and timed zones.

```xml
<CEffectCreatePersistent id="MyAura">
    <PeriodArray index="0" Period="2.0"/>
    <PeriodOffsetArray index="0" Offset="0"/>
    <PeriodicEffectArray index="0" Effect="MyPeriodicEffect"/>
    <InitialEffect value="MyInitialEffect"/>
    <FinalEffect value="MyFinalEffect"/>
    <ExpireEffect value="MyExpireEffect"/>
    <Duration value="10"/>
    <PeriodicOffsetArray index="0" Offset="0"/>
    <PeriodicValidatorArray index="0" Validator="MyValidator"/>
    <SearchFlags index="0" value="AllowEnemy"/>
</CEffectCreatePersistent>
```

- `PeriodArray` — tick intervals.
- `PeriodicEffectArray` — effects to fire each tick (paired with
  `PeriodArray` by index).
- `InitialEffect` — fires once on creation.
- `FinalEffect` — fires when duration ends naturally.
- `ExpireEffect` — fires when removed early (eg. dispelled).
- `PeriodicValidatorArray` — per-tick validators.
- `Duration` — total lifetime in seconds.

For aura-like effects, pair with `CEffectEnumArea` as the
`PeriodicEffectArray` entry to enumerate and apply effects to nearby units
each tick.

## CEffectLaunchMissile deep dive

`CEffectLaunchMissile` creates a missile unit that travels from the caster
to the target, then runs an effect on impact.

```xml
<CEffectLaunchMissile id="MyMissileLaunch">
    <AmmoUnit value="MyMissile"/>  <!-- CUnit link -->
    <Movers index="0" value="MissileMover"/>
    <PayloadEffect value="MyImpactEffect"/>
    <FinalEffect value="MyFinalEffect"/>
    <LaunchEffect value="MyLaunchEffect"/>
    <Range value="6"/>
    <Speed value="10"/>
    <DeleteOnImpact value="1"/>
</CEffectLaunchMissile>
```

The missile unit (`AmmoUnit`) must have `CMoverMissile` as its `Mover`. The
missile is created, given a mover, and launched toward the target. On
impact, `PayloadEffect` runs. `DeleteOnImpact` removes the missile unit.

## CEffectCreateUnit deep dive

`CEffectCreateUnit` spawns a unit at a location. Used for summons,
projectiles that aren't missiles (eg. spawns that move themselves), and
decoration.

```xml
<CEffectCreateUnit id="SummonImp">
    <Unit value="Imp"/>
    <Count value="3"/>
    <Owner value="Caster"/>
    <SpawnOwner value="Caster"/>
    <Location value="Target"/>
    <Facing value="0"/>
    <Offset index="0" X="1" Y="0"/>
    <InitialUnitState index="0" value="Alive"/>
    <TimeScaleActive value="1"/>
    <TimeScaleInactive value="0"/>
</CEffectCreateUnit>
```

- `Owner` — long-term owner.
- `SpawnOwner` — initial owner (often `Caster`); use for delayed ownership
  transfer.
- `Offset` — multiple offsets spawn multiple units at different positions.
- `InitialUnitState` — `Alive` / `Hidden` / `Paused` / `Inside`.

## CEffectModifyUnit / CEffectModifyPlayer

These effects let you change catalog fields at runtime:

```xml
<CEffectModifyUnit id="BoostMaxHP">
    <Unit value="Target"/>
    <Field value="LifeMax"/>
    <Value value="500"/>
</CEffectModifyUnit>
```

For player fields:

```xml
<CEffectModifyPlayer id="GiveMinerals">
    <Player value="Caster"/>
    <Field value="Minerals"/>
    <Value value="100"/>
</CEffectModifyPlayer>
```

Use these sparingly — they're powerful but less transparent than
behavior modifications, and they don't stack cleanly.

## CEffectIssueOrder

Issues an order to the target unit:

```xml
<CEffectIssueOrder id="ForceMove">
    <Order value="Move"/>
    <TargetType value="Point"/>
    <Target value="CasterLocation"/>
    <QueueType value="Replace"/>
    <ForceFlags value="IgnoreCount"/>
</CEffectIssueOrder>
```

- `Order` — abilcmd like `Move`, `Attack`, `Stop`, `Patrol`, `HoldPosition`,
  or custom abilcmds like `MyAbility,Execute`.
- `TargetType` — `None` / `Point` / `Unit` / `Item`.
- `QueueType` — `Replace` / `Prepend` / `Append`.

## Effect chain performance

Effect chains are evaluated synchronously on the game thread. Long chains
(>50 effects in one cast) can cause frame drops. Common patterns:

- Cache effect results — if an ability does the same thing multiple times,
  use `CEffectSet` with a single computation followed by reuse.
- Use `CEffectEnumArea` instead of recursive `CEffectCreateUnit` + `Wait`.
- Avoid deep `CEffectSwitch` chains — prefer a single switch with cases
  or a validator on the damage effect.

## Common pitfalls

- **`Effect` link is recursive**: if you set `Effect` to point back to
  the same effect, you get infinite recursion → stack overflow.
- **`Subject` defaults**: if `Subject` is unset, it defaults to `Target`.
  Use `Source` when you want the original caster.
- **`CEffectEnumArea` with no `Effect`**: enumerates but does nothing.
  Always set the `Effect` field.
- **`CEffectSwitch` with no `Default` and no matching case**: chain ends
  silently. Set `Default` for fallback behavior.
- **`CEffectDamage` `VitalFraction`**: uses fraction of MAX vital, not
  current. Use `VitalFractionMultiplier` for dynamic scaling.
- **`CEffectLaunchMissile` requires `AmmoUnit`**: omit it and the effect
  does nothing. The missile unit must have a `CMoverMissile`.
- **`CEffectCreatePersistent` `PeriodArray` count**: must match
  `PeriodicEffectArray` count. Mismatch causes silent no-op.
- **`CEffectModifyUnit` `Field` is field name**: use the exact catalog
  field name (e.g. `LifeMax`), not the UI display name.
- **`ResponseFlags` requires `CEffectResponse`**: the response behavior
  comes from a `CEffectResponse` catalog entry referenced by the damage
  kind. Without one, the flag does nothing.

## See also

- `catalog/fields-reference.md` §CEffect (field tables per subclass)
- `catalog/validators.md` (used by `CEffectSwitch` and `ValidatorArray`)
- `catalog/targeting.md` (target find / sort used by `CEffectEnumArea`)
- `actor/system.md` (actor events fired by effects)
- `requirement/system.md` (requirements gate visibility, not cast)
