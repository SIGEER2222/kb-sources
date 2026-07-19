# Galaxy Native Functions Reference

This is a curated index of commonly used Galaxy native functions. For the full
canonical list, consult:

- The SC2 API docs at https://mapster.talv.space (Talv's mirror).
- `galaxy/CampaignLib_h.galaxy` for campaign library declarations.
- `galaxy/natives-missing.galaxy` for natives known to be missing from the
  editor's auto-completion but still callable from custom scripts.

## Categories

### Game and player

- `PlayerGetColor(int player)` → `int`
- `PlayerGetRace(int player)` → `string` (returns `Terr` / `Zerg` / `Prot`, not full names)
- `PlayerGetProperty(int player, int property)` → `int`
- `PlayerSetPropertyInt(int player, int property, int value)`
- `PlayerStatus(int player)` → `int`
- `PlayerGetCooldown(int player, string cooldownId)` → `fixed`

### Units

- `UnitCreate(int player, string unitType, int createFlags, int color, point p, int facing)` → `unit`
- `UnitKill(unit u)`
- `UnitRemove(unit u)` — silently no-ops during init; defer with a trigger.
- `UnitIsAlive(unit u)` → `bool`
- `UnitGetType(unit u)` → `string`
- `UnitGetOwner(unit u)` → `int`
- `UnitGetPosition(unit u)` → `point`
- `UnitGroupCreate()` / `UnitGroupAdd(unitgroup g, unit u)` / `UnitGroupRemove(g, u)`
- `UnitGroupCount(unitgroup g, int countType)` → `int`
- `UnitGroupUnitFromIndex(unitgroup g, int index)` → `unit`
- `UnitGroupIssueOrder(unitgroup g, order o, int orderQueueFlags)`

### Orders

- `OrderTargetingPoint(int abilityId, point target)` → `order`
- `OrderTargetingUnit(int abilityId, unit target)` → `order`
- `Order(int abilityId)` → `order` (no-target order)
- `UnitIssueOrder(unit u, order o, int orderQueueFlags)`

### Points and regions

- `PointCreate(fixed x, fixed y)` → `point`
- `PointGetX(point p)` / `PointGetY(point p)` → `fixed`
- `RegionCreate()` → `region`
- `RegionAddRect(region r, int placement, fixed minx, fixed miny, fixed maxx, fixed maxy)`
- `RegionContainsPoint(region r, point p)` → `bool`

### Triggers

- `TriggerCreate(string funcName)` → `trigger`
- `TriggerAddEventUnitDied(trigger t, unit u)`
- `TriggerAddEventUnitConstructed(trigger t, int player)`
- `TriggerAddEventPlayerEffect(trigger t, int player, string effectId)`
- `TriggerAddEventTimePeriodic(trigger t, fixed period)` — unreliable on 7vs1
  campaign maps; prefer `while (true) { Wait(period, c_timeReal); ... }`.
- `TriggerExecute(trigger t)`
- `TriggerEnable(trigger t, bool enable)`

### Catalog (data) access

- `CatalogEntryCount(int catalogScope)` → `int`
- `CatalogEntryGet(int catalogScope, int index)` → `string`
- `CatalogFieldValueGet(int catalogScope, string entry, string field, int player)` → `string`
- `CatalogFieldValueGetAsInt(int catalogScope, string entry, string field, int player)` → `int`
- `CatalogFieldValueGetAsFixed(int catalogScope, string entry, string field, int player)` → `fixed`
- `CatalogFieldCategoryCount(int catalogScope, string category)` → `int`

Catalog scope constants: `c_catalogScopeUnits`, `c_catalogScopeAbilities`,
`c_catalogScopeUpgrades`, `c_catalogScopeBehaviors`, `c_catalogScopeEffects`, etc.

### Bank (persistence)

- `BankLoad(string name, int player)` → `bank`
- `BankSave(bank b, int player)` → `bool`
- `BankKeyCount(bank b, string section)` → `int`
- `BankKeyExists(bank b, string section, string key)` → `bool`
- `BankValueGetAsInt(bank b, string section, string key)` → `int`
- `BankValueSetFromInt(bank b, string section, string key, int value)`
- `BankLastRestoredBank(int player)` → `bank`
- `BankWait(bank b, int player)` — block until save completes.

### Waiting and timing

- `Wait(fixed seconds, int timeType)` — `timeType` is `c_timeGame` or `c_timeReal`.
  Minimum useful resolution is 1/16 second (one game tick) for `c_timeGame`.
- `TimerCreate()` → `timer`
- `TimerStart(timer t, fixed initial, bool repeating, int timeType)`

### Display and UI

- `UIDisplayMessage(int playerGroup, int messageArea, text message)`
- `TriggerDebugOutput(int level, text msg, int type)`
- `GameOver(int player, int result, bool showMission)`

## Constants

- Races: `Terr`, `Zerg`, `Prot` (returned by `PlayerGetRace` and Catalog race fields).
- Alliance: `c_allianceId Ally`, `c_allianceId Enemy`, `c_allianceId Neutral`.
- Player: `c_playerAny`, `c_playerGroupActive`, `c_playerGroupCasting`.
- Unit count type: `c_unitCountAlive`, `c_unitCountAll`, `c_unitCountCompleted`.

## See also

- `galaxy/syntax.md`
- `galaxy/script-error-codes.md`
- `legacy/galaxy-tutorial.txt`
