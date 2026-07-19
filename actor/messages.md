# Actor Messages

Actors in SC2 are event-driven visual nodes. They communicate with each other
and with galaxy scripts through **messages** — free-form strings sent via
`ActorSend`, `ActorSendTo`, `ActorScopeSend`, or `ActorRegionSend`. This
manual documents the message API surface and the most useful message names.

## Source references

- `sc2-data-trigger/.../TriggerLibs/natives.galaxy`: `ActorSend*`,
  `ActorScope*`, `ActorFrom*`, `ActorLookAt*`, `ActorTextureGroup*`.
- `sc2-data-trigger/.../TriggerLibs/GameData/Actor.galaxy`: `c_classIdCActor*`
  for the full subclass list (~100 subclasses).
- `actor/system.md`: actor kinds, lifecycle, event names.
- `catalog/fields-reference.md` §CActor entries.

## Actor handles vs actor scopes

A `unit` has one **actor scope** (`actorscope`) that owns all its actors.
Inside that scope, actors are referenced by **refName** (a string).

```c
native actorscope ActorScopeFromUnit   (unit u);
native actorscope ActorScopeFrom        (string name);    // by scope name
native actorscope ActorScopeFromActor   (actor a);
native actorscope ActorScopeFromPortrait(int p);
native actorscope ActorScopeFromDialogControl(int p);
native actorscope ActorScopeCreate     (string optionalCreateWithActorName);
```

Once you have a scope, you can pick out an actor by refName:

```c
native actor ActorScopeRefGet (actorscope as, string refName);
native void  ActorScopeRefSet (actorscope as, string refName, actor aValue);
native actor ActorFromScope   (actorscope as, string name);
```

You can also walk the actor graph from another actor:

```c
native actor ActorFromActor (actor a, string name);   // child by refName
native actor ActorFrom      (string name);             // global by name
```

### Common refNames

The unit's primary model actor is conventionally named `_` (underscore) —
this is the implicit "self" actor. Other common refNames include:

| refName | Meaning |
|---------|---------|
| `_` | Primary model actor (the unit itself). |
| `Attack` | Attack sound/model actor child. |
| `Birth` | Birth animation actor. |
| `Death` | Death animation actor. |
| `Hit` | Damage-hit actor. |
| `Wpn` | Weapon attachment point. |
| `Head`, `Chest`, `Hand_L`, `Hand_R`, `Foot_L`, `Foot_R`, `Origin` | Standard attachment points. |
| `Ref_1`, `Ref_2`, ... | User-defined reference actors. |
| `Overlay1`, `Overlay2` | Overlay actors (e.g. cloak shimmer). |

You can also reference actors by their `ActorName` attribute (the `Name`
field on `<CActorModel>`), in which case the engine resolves the scope
automatically.

## Sending messages

There are four primary send APIs:

```c
native void ActorSend         (actor a, string msg);          // to one actor
native void ActorSendTo       (actor a, string refName, string msg); // to refName child
native void ActorScopeSend    (actorscope as, string msg);    // broadcast to scope
native void ActorRegionSend   (actor a, int intersect, string msg, string filters, string terms);
```

`ActorSend` and `ActorSendTo` are the workhorses — `ActorSendTo` is
necessary when you don't have an `actor` handle for the child but you have
the parent and know its refName.

`ActorScopeSend` is rare — used to fan-out a message to *every* actor in
the scope (e.g. hide everything during a cutscene).

`ActorRegionSend` is used to message every actor inside an `ActorRegion`
matching a filter — useful for mass visual updates.

### Text vs string

```c
native void ActorSend        (actor a, string msg);
native void ActorSendAsText  (actor a, text msg);
native void ActorSendTo      (actor a, string refName, string msg);
native void ActorSendToAsText(actor a, string refName, text msg);
```

The `*AsText` variants resolve `text` localization before sending. Use them
when the message body contains a player-facing string (rare).

## Message grammar

A message is a single string of the form:

```
MsgName Param1 Param2 Param3
```

Parameters are space-separated. Quoted strings are supported when a
parameter contains a space:

```
AnimPlay Stand "Working"
```

The engine parses this and routes to the actor's message handler. Unknown
messages are silently dropped (no ScriptError).

## Useful message names

These are the message names most commonly used from galaxy or via the
`Action` catalog field on `CActorModel`.

### Visibility / lifecycle

| Message | Effect |
|---------|--------|
| `SetVisibility Show` / `SetVisibility Hide` | Show/hide the model without destroying the actor. |
| `Destroy` | Kill the actor immediately. |
| `AnimPlay <name>` | Play a one-shot animation. |
| `AnimPlay <name> <variant>` | Play with a named variant. |
| `AnimPlayTimeScale <name> <scale>` | Play with time scale. |
| `AnimClear <name>` | Stop the named animation. |
| `AnimGroupTimeScale <scale>` | Set global anim time scale. |
| `ModelSwap <modelId>` | Swap the model to another model id. |
| `ModelAdd <modelId>` | Add a model as child. |
| `TextureSwap <textureProps>` | Swap a texture group. |

### Status / state

| Message | Effect |
|---------|--------|
| `SetTeamColor <index>` | Override team color index. |
| `SetColor <color>` | Override tint. |
| `SetTint <color> <weight> <time>` | Apply tint with weight/duration. |
| `SetScale <x> <y> <z>` | Scale model. |
| `SetRotation <yaw> <pitch> <roll>` | Rotate model. |
| `SetPosition <x> <y> <z>` | Move model. |
| `SetHeight <height>` | Set hover height. |
| `SetFacing <facing>` | Set facing. |

### Attachment / sounds

| Message | Effect |
|---------|--------|
| `AttachModel <refName> <modelId>` | Attach a child model at a refName. |
| `AttachModelToObject <refName> <modelId>` | Attach to object query. |
| `SoundPlay <soundId>` | Play a sound from the actor. |
| `SoundStop` | Stop sound. |
| `MoverMove <moverId>` | Start a mover (for projectiles). |

### Combat / weapon sync

| Message | Effect |
|---------|--------|
| `WeaponStart <weaponId>` | Mark weapon as firing (drives attack animation tag). |
| `WeaponStop <weaponId>` | Mark weapon stopped. |
| `AttackStart <target>` | Start attack sequence toward a target unit actor. |
| `AttackStop` | End attack sequence. |

### Custom events

| Message | Effect |
|---------|--------|
| `CustomName` | Fires a custom event handler defined in the actor's `<On>` events. |

Custom events are how data designers expose actor customization: define
`<On Name="CustomRedAlert" Action="..."/>` in the actor XML, then trigger
it from galaxy via `ActorSend(actor, "CustomRedAlert")`.

## Querying actor state

```c
native text   ActorGetText    (actor a);     // display name
native actor  ActorLastCreated();            // most recently created actor
native actor  ActorLastCreatedSend();         // create + lastCreated convenience
native actorscope ActorScopeLastCreated();
native actorscope ActorScopeLastCreatedSend();
native text   ActorScopeGetText (actorscope as);
```

There is no general "actor get current animation" query — actor state is
fire-and-forget. If you need state sync between galaxy and actor, use a
**custom event** pattern: the actor sends a `<Action>` `CustomName` back to
galaxy via a `libNtve_CustomEvent` trigger when its animation completes,
and galaxy listens for that event.

## Creating actors

```c
native actor ActorCreate(
    actorscope as,
    string actorName,        // link to a CActor entry, or empty for anonymous
    string content1Name,
    string content2Name,
    string content3Name
);
```

`content1Name`/`content2Name`/`content3Name` are typically the model id,
the alternate model (for variants), and the sound link — but their exact
meaning depends on the actor's `Content` array in XML. Most CActorModel
entries accept `content1Name` = model id.

```c
// Spawn a fire visual at a point
actorscope scope = ActorScopeCreate("MyFireScope");
ActorCreate(scope, "FireSmall", "", "", "");
ActorLastCreatedSend();
ActorSend(ActorLastCreated(), "SetPosition 12.5 7.0 0");
ActorSend(ActorLastCreated(), "SetScale 2.0 2.0 2.0");
```

When you're done with a scope, kill it to free resources:

```c
native void ActorScopeKill     (actorscope as);
native void ActorScopeOrphan   (actorscope as);   // detach but don't kill
```

`Orphan` is useful when you want the actor to live until its model finishes
its death animation — the engine will garbage-collect it when ready.

## LookAt system

LookAt is the actor system that makes a unit's head/eyes track a target
(e.g. marines turning to look at a passing Banshee).

```c
native void ActorLookAtStart (actor s, string lookAt, int weight, fixed time, actor t);
native void ActorLookAtStop  (actor s, string lookAt, int weight, fixed time);
native void ActorLookAtTypeStart (actor s, string type, actor t);
native void ActorLookAtTypeStop  (actor s, string type);
```

- `s` = the source actor (e.g. the unit's primary model).
- `lookAt` = the look-at target name defined on the actor (e.g. `Head`).
- `weight` = blend weight (0–100); higher overrides lower.
- `time` = blend in/out duration in seconds.
- `t` = the target actor to look at.

`LookAtTypeStart` uses the actor's `LookAtType` definitions instead of a
specific refName — useful for stance-based looks (e.g. `Alerted`,
`Engaged`).

### LookAt in XML

```xml
<CActorLookAt id="Head">
    <Subject value="Target"/>
    <Type value="Head"/>
</CActorLookAt>
```

Multiple look-at actors can coexist on one unit; the engine blends them by
weight.

## Texture groups

Texture groups let you swap multiple textures atomically (e.g. team color,
damage states, owner-specific overlays).

```c
native void ActorTextureGroupApplyGlobal (string textureProps);
native void ActorTextureGroupRemoveGlobal(string textureProps);
native void ActorTextureGroupPush();   // save current state
native void ActorTextureGroupPop();     // restore
```

`Apply/Remove` are global — they affect every model using that texture
group. `Push/Pop` let you set a temporary override and restore it.

Example: tint everything red during a cinematic event:

```c
ActorTextureGroupPush();
ActorTextureGroupApplyGlobal("RedOverlay");
Wait(3.0, c_timeReal);
ActorTextureGroupPop();
```

## Actor regions

`ActorRegion` is the actor-side equivalent of `region` — a region of space
that detects actor entry/exit. Useful for triggering visual effects when
actors enter a zone (e.g. fog of war props, region tinting).

```c
native actor ActorRegionCreate (actorscope as, string actorName, region r);
native void  ActorRegionSend    (actor a, int intersect, string msg,
                                  string filters, string terms);
```

- `a` = the actor region's actor handle.
- `intersect` = a bitfield (usually `1`).
- `filters` = unit filter string applied to the actors (same grammar as
  `UnitFilterStr`).
- `terms` = additional matching terms (advanced).

When an actor enters the region matching the filter, it receives the
`msg`.

## Wiring messages from catalog (the `Action` field)

Most actor messages aren't sent from galaxy — they're sent by the actor
itself in response to events, via the `<Action>` field on `<CActorModel>`
or `<CActorSound>`:

```xml
<CActorModel id="Marine">
    <On Terms="WeaponStart.*" Action="SoundPlay AttackSound"/>
    <On Terms="WeaponStop"    Action="SoundStop"/>
    <On Terms="UnitDeath"     Action="AnimPlay Death"/>
</CActorModel>
```

The `Action` value is the same string you'd pass to `ActorSend`. The most
commonly used actions are `SoundPlay`, `AnimPlay`, `AnimClear`,
`ModelSwap`, `SetVisibility`, and `Custom<Name>`.

## Common patterns

### Spawn a temporary visual effect at a point

```c
// Standard recipe
actorscope scope = ActorScopeCreate("FX");
ActorCreate(scope, "PsiStormSmall", "", "", "");
ActorSend(ActorLastCreated(), "SetPosition 12.0 8.0 0.5");
ActorSend(ActorLastCreated(), "AnimPlay Stand");
// Auto-cleanup after the model's animation completes
ActorScopeOrphan(scope);
```

### Attach a glow to a unit while a buff is active

```c
// In a trigger that fires when the buff is applied
unit u = EventUnit();
actor a = ActorFromActor(ActorLastCreated(), "_");   // unit's primary
ActorSendTo(a, "Overlay1", "AttachModel Overlay1 FireGlowModel");

// When the buff expires
ActorSendTo(a, "Overlay1", "Destroy");
```

### Hide all of a unit's actors during a cinematic

```c
actorscope scope = ActorScopeFromUnit(u);
ActorScopeSend(scope, "SetVisibility Hide");
// ... cinematic ...
ActorScopeSend(scope, "SetVisibility Show");
```

### Trigger a custom event defined in XML

Define the event in XML:

```xml
<CActorModel id="CustomEventBus">
    <Events index="0">
        <Id value="CustomRedAlert"/>
        <Action value="AnimPlay Alerted RedAlert"/>
    </Events>
</CActorModel>
```

Then trigger it from galaxy:

```c
actor bus = ActorFrom("CustomEventBus");
ActorSend(bus, "CustomRedAlert");
```

### Play a portrait animation on a UI dialog

```c
int portrait = PortraitCreate(0, c_playerNeutral, ...);
actorscope scope = ActorScopeFromPortrait(portrait);
actor a = ActorFromScope(scope, "_");
ActorSend(a, "AnimPlay Idle");
```

## Pitfalls

- **`ActorSend` is async.** The message is queued and processed on the
  next actor tick — checking the actor state immediately after sending
  will not reflect the change.
- **Unknown messages silently drop.** A typo in the message name doesn't
  raise a ScriptError — the actor simply ignores it. Use `ActorGetText`
  or `ActorFrom` queries to verify the actor exists before sending.
- **`ActorLastCreated` is global.** Between an `ActorCreate` and your next
  call, any other actor creation in another trigger will overwrite
  `ActorLastCreated`. Use `ActorLastCreatedSend` immediately after
  `ActorCreate`, or capture the handle in a local variable.
- **Actor handles are not ref-counted.** If you hold an `actor` handle
  after the underlying actor is destroyed (`ActorScopeKill` or natural
  death), the handle becomes a dangling pointer — sending to it is a
  silent no-op, not a crash, but your logic may go wrong.
- **refName lookups are case-sensitive.** `_` is convention; the actual
  refName is defined by the actor's `RefName` field. Check `ActorData.xml`
  or use the SC2 Editor's Actor panel to verify the exact refName.
- **Custom events must be defined on the actor before they're sent.** A
  `CustomRedAlert` message to an actor that doesn't declare a matching
  `<On>` is silently ignored.
- **TextureGroupApplyGlobal affects every model.** If you only want to
  tint one unit, use `ActorSend` with `SetTint` instead.
- **`ActorRegionSend`'s `filters` parameter uses unit-filter grammar,
  not actor-filter grammar.** This is a common confusion — the filter is
  applied to the *units* associated with the actors in the region.
- **Heavy `AttachModel` usage is expensive.** Each attached model is a
  full actor with its own animation tick. Prefer `TextureSwap` for simple
  visual variations.
- **`ActorScopeOrphan` does not prevent the actors from being killed by
  `ActorScopeKill`.** Use either orphan (for natural cleanup) or kill (for
  immediate cleanup), not both.

## See also

- `actor/system.md` — actor kinds, lifecycle events, message chain.
- `catalog/fields-reference.md` — CActor entries, On/Action fields.
- `galaxy/native-index.md` — Actor/ActorScope native signatures.
- `cutscene/system.md` — cutscene-specific actor integration.
- `sound/system.md` — `CActorSound` and `SoundPlay` integration.
