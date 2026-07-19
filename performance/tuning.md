# SC2 Performance Tuning

SC2's main game thread runs at 16 ms / frame (62.5 Hz game tick). Anything
that exceeds this budget causes frame drops; sustained overruns cause the
game to slow down (visible as "slow motion" gameplay). This page collects
the patterns that cause performance issues and the alternatives.

## Per-tick cost offenders

### 1. Per-second full-map scans

A common anti-pattern:

```galaxy
// ❌ Bad: every 3 seconds, enumerate every unit on the map
while (true) {
    Wait(3.0, c_timeReal);
    unitgroup g = UnitGroup(null, c_playerAny, RegionPlayableMap(), UnitFilter(0,0,0,0), 0);
    unit u = UnitGroupUnit(g, 1);
    while (u != null) {
        // do work per unit
        u = UnitGroupUnit(g, UnitGroupIndex(g, u) + 1);
    }
}
```

This works on small maps but stalls on large Co-op maps with hundreds of
units. Alternatives:

- Use specific `UnitFilter` to narrow the group before enumerating.
- Use `TriggerAddEventUnitCreated` / `TriggerAddEventUnitDied` to maintain
  a cached `unitgroup` instead of re-enumerating.
- Use `RegionCircle(center, radius)` to limit the spatial scope.
- If scanning for a specific unit type, use
  `UnitGroup(UnitType, player, region, filter, maxCount)` with `maxCount=1`.

### 2. Per-tick CatalogFieldValueGet

`CatalogFieldValueGet` is a hash lookup but still costs ~1 µs per call.
Calling it 1000× per frame for every unit's stats is detectable.

Cache the values in a Galaxy array indexed by unit type ID at map init:

```galaxy
// ✅ Good: cache unit stats at init
string[UnitTypeCount] g_unitMaxHpCache;

void CacheUnitStats () {
    int count = CatalogEntryCount(c_catalogScopeUnit);
    for (int i = 0; i < count; i += 1) {
        string entry = CatalogEntryGet(c_catalogScopeUnit, i);
        if (CatalogEntryScope(entry) == c_catalogScopeUnit) {
            g_unitMaxHpCache[entry] = CatalogFieldValueGet(
                c_catalogScopeUnit, entry, "MaxHealth", 1);
        }
    }
}
```

### 3. Per-tick CatalogEntryCount enumeration

`CatalogEntryCount` walks the catalog; calling it per frame to enumerate
upgrades is wasteful. Cache the count and entry IDs at init into a Galaxy
array.

### 4. BankSave per tick

`BankSave` writes a file to disk; the OS may block on the write. Calling
it every frame causes disk I/O stalls.

Only save on meaningful state changes:

- Game over.
- Major checkpoint reached.
- Player request (e.g. "Save and Quit" button).
- Periodic save every 60 seconds if your game has long sessions.

For incremental state that doesn't need persistence, keep it in Galaxy
arrays and only serialize to Bank on save events.

### 5. String concatenation in hot loops

Galaxy strings are immutable; `StringConcat` allocates. Doing this inside
a per-unit loop is slow. Build a single string at init and cache it.

## Patterns that are cheap

- `UnitGroupUnit(group, index)` — O(1) random access.
- `UnitGetCustomValue(unit, index)` — direct array lookup.
- `UnitGetProperty(unit, prop, current)` — native cached lookup.
- `UnitIsAlive(unit)` — boolean check.
- Catalog field access at init time only — batched is fast.
- Trigger subscription to events (`UnitCreated`, `UnitDied`, `EnterRegion`) —
  engine-optimized, near-zero overhead per subscriber.

## Wait granularity

```galaxy
const fixed c_timeReal = ...;
const fixed c_timeGame = ...;
```

- `Wait(time, c_timeReal)` — wall-clock time, always advances.
- `Wait(time, c_timeGame)` — game time, pauses during cinematic / pause.

Minimum `Wait` is 1/16 second (one game tick). Shorter waits are rounded
up silently. For finer-grained pacing, use `TriggerAddEventTimePeriodic`
with a 0.0625 second interval (but see `triggers/system.md` for the
`TriggerAddEventTimePeriodic` 7vs1 caveat).

## Async execution

The Galaxy VM is single-threaded; long computations block the main thread.
For work that doesn't need to run on the game thread:

- Use `TriggerCreate` + `TriggerAddEventTimePeriodic` to spread work
  across frames.
- Use `Wait(0.0625, c_timeReal)` between batches to yield back to the
  engine.

```galaxy
// Process 50 units per frame instead of all at once
int g_nextUnitIndex = 1;
unitgroup g_cachedUnits;

void ProcessBatched_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    int processed = 0;
    while (processed < 50) {
        unit u = UnitGroupUnit(g_cachedUnits, g_nextUnitIndex);
        if (u == null) {
            g_nextUnitIndex = 1;  // wrap around
            return true;
        }
        // ... do work ...
        g_nextUnitIndex += 1;
        processed += 1;
    }
    return true;
}
```

## DeferredCleanup pattern

For work that must happen after init (e.g. removing preplaced units), the
deferred cleanup pattern works around the init-phase `UnitRemove` no-op:

```galaxy
trigger g_deferredCleanup;

void DeferredCleanup_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    Wait(2.0, c_timeReal);  // Let init complete
    // ... do the cleanup ...
    return true;
}

void ScheduleDeferredCleanup () {
    g_deferredCleanup = TriggerCreate("DeferredCleanup_Func");
    TriggerExecute(g_deferredCleanup, false, false);
}
```

`TriggerExecute` with `false` for `wait` and `false` for `runActions`
starts the trigger asynchronously (doesn't block the caller).

## Profiling

SC2 doesn't expose a script profiler in shipped builds, but you can use
`TriggerAddEventTimePeriodic` with a 1-second trigger that prints
`GameGetTime()` and your own counters:

```galaxy
int g_unitsProcessed = 0;

void Profile_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    print("Units processed this second: " + IntToText(g_unitsProcessed));
    g_unitsProcessed = 0;
    return true;
}
```

Watch the GameLogs directory (`C:\Users\<user>\Documents\StarCraft II\GameLogs`)
for ScriptError reports — they include timing data when frames overrun.

## Mod size and load time

- Each mod dependency adds load time. Compress textures and sounds before
  importing.
- Use `.m3` model format instead of `.m3a` (animated) when animation
  isn't needed — smaller memory footprint.
- Group small textures into atlases; the engine batches draw calls better.
- Avoid deeply nested mod dependency chains — each level adds load time.

## Common pitfalls

- **`UnitGroup` enumeration on the entire map per tick**: see above.
- **Catalog field access per tick**: cache at init.
- **`BankSave` per tick**: serialize on events only.
- **String concatenation in hot loops**: precompute strings.
- **Long synchronous work in `MapInit`**: defer with `Wait(2.0, c_timeReal)`
  and `TriggerExecute`.
- **`UnitRemove` during init**: no-op; use the DeferredCleanup pattern.
- **Deep recursion**: Galaxy has a small stack (~256 frames). Use iteration
  for deep traversals.
- **`PlayerGroupAll()` in tight loops**: iterating every player every frame
  is wasteful when only 2 active players exist. Cache `PlayerGroupAllActive()`
  once per second.
- **`TriggerAddEventTimePeriodic` with sub-second intervals on 7vs1**:
  unreliable; use `Wait`-based loops instead.

## See also

- `triggers/system.md` (init order, Wait caveats)
- `galaxy/syntax.md` (string handling, arrays)
- `multiplayer/sync.md` (per-frame network costs)
- `runtime-contracts/observer.md` (runtime error scanning)
