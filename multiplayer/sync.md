# SC2 Multiplayer & Synchronization

SC2 multiplayer is deterministic lockstep: every client runs the same
Galaxy script and the same game simulation, and the engine syncs player
inputs (orders, ability commands, selections). Game state diverges only
if scripts produce non-deterministic results. This page covers what
Galaxy code must do to stay correct in multiplayer.

## Game time

```galaxy
int SynchronousGameStartTimeGet ();
int CurrentSynchronousGameTimeGet ();
```

`SynchronousGameStartTimeGet` returns the game time at which all clients
finished loading. `CurrentSynchronousGameTimeGet` returns the current
synced game time in milliseconds. Both are guaranteed identical across
all clients — use them instead of `GameGetTime()` for cross-client
synchronization logic.

`Wait(time, c_timeGame)` blocks for the given game-time duration. During
cinematics or pause, game time stops, so `c_timeGame` waits also pause.
Use `c_timeReal` for wall-clock pacing that ignores game state.

## Player join / leave events

```galaxy
void TriggerAddEventPlayerLeft (trigger inTrigger, int player, int inResult);
```

`inResult` is one of `c_playerLeaveReason*` (e.g. `Quit`, `Disconnect`,
`Defeat`, `Victory`). This event fires on every client when a player
drops, so cleanup code (e.g. transferring units to allies) runs everywhere.

There's no `PlayerJoined` event — the lobby sets up the player roster
before `MapInit` fires. Player slots are fixed for the duration of the
match.

## Player groups

Use `PlayerGroupAll()` for "every player including observers", or filter
by `PlayerGroupOwner` for active players. `PlayerGroupAllActive()` skips
observers and defeated players.

```galaxy
int PlayerGroupCount (playergroup group, int type);
int PlayerGroupPlayer (playergroup group, int type, int index);
// type: c_playerGroupAll / c_playerGroupActive / c_playerGroupAllOrObs
```

## Trigger synchronization

Trigger execution is host-driven by default — actions run on every client
when the trigger fires. For host-authoritative logic (e.g. spawning
special units, awarding bonuses), gate with:

```galaxy
bool PlayerIsLocal (int player);
int  PlayerLocal ();
```

`PlayerLocal()` returns the local player's ID. Standard pattern:

```galaxy
bool gt_MyEvent_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    if (PlayerLocal() != <host player>) {
        return true;
    }
    // Host-only logic here.
    // Unit creation, catalog changes, etc. are auto-replicated to all clients.
    return true;
}
```

Unit creation, orders, and catalog changes done by the host are
automatically replicated — you don't need to manually sync them.

## Network-aware natives

Some natives are explicitly network-aware:

```galaxy
void   SoundLengthQuery (soundlink info);  // Broadcast a length query
void   SoundLengthQueryWait ();             // Wait for all clients to respond
fixed  SoundLengthSync (soundlink info);    // Retrieve the synced result

fixed  AnimLengthSync (generichandle h);        // Synced animation length
fixed  AnimLengthRemainingSync (generichandle h);
```

These block until all clients have reported, then return the same value
on every client. Use them whenever you need to wait on a duration that
varies by locale (sound files) or model variant (animations).

## Bank file synchronization

Bank files are per-player per-map save files. In multiplayer:

- Each client has its own bank for the local player.
- Banks for non-local players are received from the network when the
  player joins.
- `BankPreload(key, player)` initiates a network sync; `BankWait(key)`
  blocks until the bank arrives.
- `BankLoad(key, player)` returns a `bank` handle if the file exists
  locally or after `BankWait` completes.

```galaxy
void   BankPreload (string name, int player);
bool   BankWait (string name);
bank   BankLoad (string name, int player);
bool   BankExists (string name, int player);
string BankLastRestoredName ();
int    BankLastRestoredPlayer ();
```

For multiplayer-critical Bank reads (e.g. commander mastery levels),
always `BankPreload` then `BankWait` in `MapInit` before reading.

## UI sync frames

Many UI elements are "synced frames" — visible to all clients and updated
by the host. Use `c_syncFrameType*` constants to identify them:

```galaxy
const int c_syncFrameTypeMenuBar                = 0;
const int c_syncFrameTypeObjectivePanel          = 3;
const int c_syncFrameTypeSupply                  = 5;
const int c_syncFrameTypeResourcePanel           = 6;
const int c_syncFrameTypeVictoryPanel            = 12;
const int c_syncFrameTypeHeroPanel               = 15;
const int c_syncFrameTypeBattleUI                = 20;
const int c_syncFrameTypeMinimapPanel            = 21;
const int c_syncFrameTypeCommandPanel            = 22;
const int c_syncFrameTypeInventoryPanel          = 23;
const int c_syncFrameTypeMissionTimePanel        = 28;
const int c_syncFrameTypeControlGroupPanel       = 29;
// ... ~30 total
```

UI changes to these frames are automatically replicated. Non-synced
frames (custom dialogs) are local-only and require explicit
synchronization if other clients need to know the dialog state.

## TriggerSync (custom data sync)

For custom cross-client data, use TriggerSync:

```galaxy
// Pattern: host computes data, attaches to a trigger, fires it.
// Clients receive the trigger event with the attached data.
```

This is the manual escape hatch when no built-in native provides sync.
Use sparingly — it adds per-frame network traffic.

## Common pitfalls

- **Don't use `RandomInt`/`RandomFixed` in deterministic code**: unless
  seeded identically across clients, random numbers diverge and cause
  desyncs. Use `RandomIntSeed` and `RandomFixedSeed` to set a fixed seed
  agreed upon at match start, or use pre-determined tables.
- **`UnitCreate` is auto-replicated**: don't call it on every client —
  the host's call is replicated automatically.
- **Dialog UI is local by default**: changes to dialog controls aren't
  replicated unless the underlying frame is a sync frame. Custom UI
  requires explicit sync via TriggerSync.
- **`GameGetTime()` may diverge**: use `CurrentSynchronousGameTimeGet`
  for cross-client timing.
- **Preplaced units are sync-critical**: changing the preplaced unit
  list in the editor invalidates sync with older map versions. Always
  bump the map version when changing preplaced units.
- **`TriggerAddEventPlayerLeft` cleanup is per-client**: when a player
  drops, every client must run cleanup (e.g. remove their units,
  transfer resources). Don't gate this with `PlayerIsLocal`.
- **Bank file size limits**: banks are capped at 4 KB per section, 64 KB
  total per file (Blizzard maps get larger limits via special permission).
  Compress your data and split across sections.
- **Preloading assets**: `PreloadAsset` / `PreloadUnit` / `PreloadModel`
  queue assets for loading. In multiplayer, every client must preload
  the same assets — otherwise one client may stutter while others
  don't, causing sync issues during the loaded asset's first use.
- **Region enter/leave events are per-client**: each client fires its
  own `TriggerAddEventUnitEnterRegion` callbacks based on what that
  client's simulation sees. They're already synced via the lockstep
  simulation; no manual sync needed.

## See also

- `bank/format.md` (Bank file format, encryption, sections)
- `triggers/system.md` (initialization order)
- `performance/tuning.md` (per-tick cost considerations)
- `runtime-contracts/observer.md` (multiplayer testing contract)
