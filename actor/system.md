# Actor System

Actors are the visual/sound layer between Catalog data and the in-game model. They
translate data events (UnitBirth, WeaponStart, AnimationStart) into model actions
(play animation, attach model, emit sound).

## Source references

- `galaxy/natives/catalog/Actor.galaxy`: native function declarations for the Actor
  catalog scope (auto-generated from `Actor.xml`).
- `catalog/reference/ActorData.xml`: full official Actor catalog (Blizzard assets,
  reference only).
- `editor/editor-overview.md` §6 (Data Editor overview).

## Actor kinds

Actor catalog has dozens of `CActor*` subclasses. The most commonly used:

| Class | Description |
|-------|-------------|
| `CActorModel` | Primary visible model. Most units have one. |
| `CActorSiteOp` | Positioning operation (offset, rotation, snap). |
| `CActorSound` | One-shot sound on event. |
| `CActorBeamSimple` / `CActorBeamStandard` | Beam visuals (Lurker attack, Colossus). |
| `CActorLight` / `CActorLightOmni` / `CActorLightSpot` | Dynamic lights. |
| `CActorDoodad` | Doodad actor. |
| `CActorPortrait` | Portrait-model actor for UI. |
| `CActorSplat` | Decal/splat on terrain (rally point,Psistorm). |
| `CActorEventMacro` | Reusable event-driven macro actor. |
| `CActorGlobalConfig` | Global actor configuration. |

See `Actor.galaxy` constants `c_classIdCActor*` for the full list.

## Actor events

Actors are event-driven. Common events sent by the engine to a unit's actor:

- `UnitBirth` — unit created and becomes visible.
- `UnitDeath` — unit dying animation starts.
- `WeaponStart` / `WeaponStop` — weapon firing begins/ends.
- `AnimationStart` / `AnimationDone` — animation lifecycle.
- `AnimationTagUpdate` — animation tag (e.g. moving, attacking) changed.
- `SoundDone` — sound finished playing.
- `ModelLoad` / `ModelUnload` — model load/unload.

Custom events sent via `ActorSend`, `UnitSendActorMsg`, `ActorSendAsText` allow
data-driven communication between triggers and actors. Custom event names are
free-form strings defined by the actor.

## Actor message chain

A typical attack sequence:

1. Unit issues order → catalog resolves ability → effect chain executes.
2. `CEffectDamage` or similar fires `ActorMsg` to the unit's model actor.
3. Actor receives `WeaponStart` → plays attack animation.
4. At animation event `Attack`, actor triggers a `CActorSound` child → plays
   attack sound.
5. Effect chain reaches `CEffectCreateUnit` (for projectiles) or `CEffectDamage`
   (for instant damage).

## Cross-actor queries

- `ActorGetText` / `ActorGetUnit` / `ActorGetRole` — actor→property queries.
- `ActorSetFilterState` — programmatically toggle an actor filter.
- `ActorSend` / `ActorSendAsText` — broadcast a message by name or string.

## Pitfalls

- Actor messages are async; sending one and immediately checking the model state
  will miss the change. Use a trigger on the corresponding event instead.
- Custom actors referenced by name from a Catalog `Actor` field must exist in the
  merged catalog at load time. Missing actors silently no-op rather than
  ScriptError.
- Heavy per-unit actors (e.g. many child `CActorSound`) cause performance issues
  in large battles. Use `CActorEventMacro` to share work across actor instances.

## See also

- `catalog/format.md`
- `galaxy/natives/catalog/Actor.galaxy`
- `catalog/reference/ActorData.xml`
