# SC2 Galaxy Native API Index

This page indexes the SC2 native function library by category. For full
signatures and constants, search `sc2-data-trigger/mods/core.sc2mod/base.sc2data/TriggerLibs/natives.galaxy`
(~1500 functions, 254 KB).

## How natives are declared

Natives are declared in `natives.galaxy` as:

```galaxy
native void   FunctionName (params);
native int    FunctionName (params);
native fixed  FunctionName (params);
native string FunctionName (params);
native bool   FunctionName (params);
```

Each returns a typed value. Native calls are synchronous — they block the
game thread until the function returns.

## Categories (from `natives.galaxy` table of contents)

The native library is organized into 56 categories:

1. **About Types** — type system overview
2. **Common** — utility (`UnitLastCreated`, etc.)
3. **Achievements**
4. **Actor** — actor creation / messaging
5. **Animation** — animation control
6. **Automation** — automation API
7. **Bank** — Bank file I/O
8. **Battle Report**
9. **BitMask**
10. **Boards**
11. **Camera** — camera control
12. **Campaign** — campaign progress
13. **Catalogs** — Catalog field get/set
14. **Cinematics** — cinematic playback
15. **Conversions** — type conversions
16. **Conversation** — branching dialogue
17. **Data Table** — named storage
18. **Dialogs** — UI dialogs
19. **Effect History**
20. **Environment** — fog, weather
21. **Game** — game state
22. **Game User**
23. **Looping** — loop utilities
24. **Markers**
25. **Math** — math functions
26. **Melee** — melee init
27. **Minimap**
28. **Movie** — movie playback
29. **Objectives** — mission objectives
30. **Orders** — unit orders
31. **Path** — pathing
32. **Ping** — minimap pings
33. **Planet**
34. **Players** — player state
35. **Player Groups**
36. **Points** — point geometry
37. **Portraits**
38. **Preload** — asset preloading
39. **PurchaseItems**
40. **Regions** — regions
41. **Sound** — sound playback
42. **Story Mode**
43. **Strings** — string manipulation
44. **Tech Tree** — tech tree queries
45. **Text Tags** — floating text
46. **Timing** — Wait, timers
47. **Transmission** — in-game dialogue
48. **Triggers** — trigger creation / events
49. **Units** — unit control
50. **Unit Filters** — filter construction
51. **Unit Groups** — group enumeration
52. **Unit Reference**
53. **Unit Selection**
54. **User Data** — UserData.xml
55. **User Interface** — UI panels
56. **Visibility** — vision control

## Key natives by category

Below are the most frequently used natives per category. For full lists,
grep `natives.galaxy` for the function name.

### Common

```galaxy
unit        UnitLastCreated ();
unit        UnitLastRestored ();
unitgroup   UnitGroupLastCreated ();
order       OrderLastCreated ();
bool        GameIsTestMode ();
void        TriggerExecute (trigger t, bool testConds, bool waitActions);
```

### Units

```galaxy
unit    UnitCreate (int player, string unitType, int flags, int color, point pos, fixed facing, string skinLink);
unit    UnitFromUnit (unit u);  // clone
unit    UnitRefFromUnit (unit u);
void    UnitRemove (unit u);
void    UnitKill (unit u);
bool    UnitIsAlive (unit u);
int     UnitGetOwner (unit u);
string  UnitGetType (unit u);
point   UnitGetPosition (unit u);
fixed   UnitGetFacing (unit u);
fixed   UnitGetHeight (unit u);
fixed   UnitGetProperty (unit u, int property, bool current);  // c_unitProp*
fixed   UnitGetSpeed (unit u);
void    UnitSetSpeed (unit u, fixed speed, int type);  // c_speedChangeCurrent/Max/etc.
void    UnitSetProperty (unit u, int property, fixed value);
int     UnitGetStatus (unit u, int status);  // c_unitStatus*
void    UnitSetStatus (unit u, int status, bool on);
int     UnitGetState (unit u, int state);  // c_unitState*
void    UnitSetState (unit u, int state, bool on);
text    UnitGetName (unit u);
void    UnitSetName (unit u, text name);
int     UnitGetTag (unit u);
void    UnitSetTeamColor (unit u, color c);
```

### Unit flags (control)

```galaxy
// c_unitFlag* constants are in GameData/Unit.galaxy.
// Runtime equivalents:
const int c_unitStatusIdle = 1;
const int c_unitStatusActive = 2;
// ... etc.

// Selectability is set via state:
void UnitSetState (unit u, int state, bool on);
// where state is one of c_unitState*
// (e.g. c_unitStateSelectable, c_unitStateTargetable, c_unitStateVisible,
//        c_unitStateSuppressDrop, etc.)
```

### Unit vitals

```galaxy
fixed   UnitGetVital (unit u, int vital);  // c_unitVitalLife / Shields / Energy
fixed   UnitGetVitalMax (unit u, int vital);
fixed   UnitGetVitalRegen (unit u, int vital);
fixed   UnitGetVitalFraction (unit u, int vital);
void    UnitSetVital (unit u, int vital, fixed value);
void    UnitSetVitalMax (unit u, int vital, fixed value);
void    UnitSetVitalRegen (unit u, int vital, fixed value);
void    UnitDamage (unit u, int damageType, fixed amount, unit attacker, unit causer, string damageName);
void    UnitHeal (unit u, fixed amount, unit healer);
```

### Unit behaviors

```galaxy
void        UnitBehaviorAdd (unit u, string behavior, unit caster, int stacks);
void        UnitBehaviorAddPlayer (unit u, string behavior, int player, int stacks);
void        UnitBehaviorRemove (unit u, string behavior, int count);
int         UnitBehaviorCount (unit u, string behavior);
bool        UnitBehaviorHas (unit u, string behavior);
fixed       UnitBehaviorGetDuration (unit u, string behavior);
fixed       UnitBehaviorGetDurationRemaining (unit u, string behavior);
void        UnitBehaviorSetDuration (unit u, string behavior, fixed time);
void        UnitBehaviorSetDurationRemaining (unit u, string behavior, fixed time);
void        UnitBehaviorSetStackCount (unit u, string behavior, int count);
int         UnitBehaviorGetStackCount (unit u, string behavior);
int         UnitBehaviorCountAll (unit u);
string      UnitBehaviorGetAll (unit u, int index);
void        UnitBehaviorRemoveGroup (unit u, behaviorgroup group);
```

### Unit XP / Veterancy

```galaxy
void   UnitXPAddXP (unit u, int xpType, fixed amount);
fixed  UnitXPGetCurrentXP (unit u, int xpType);
void   UnitXPSetCurrentXP (unit u, int xpType, fixed value);
int    UnitXPGetCurrentLevel (unit u, int xpType);
void   UnitXPSetCurrentLevel (unit u, int xpType, int level);
fixed  UnitXPGetXPForLevel (unit u, int xpType, int level);
void   UnitXPSetXPForLevel (unit u, int xpType, int level, fixed value);
int    UnitXPGetNumLevels (unit u, int xpType);
void   UnitXPGainEnable (unit u, behavior b, bool enable);
void   UnitXPSync (unit u);  // sync XP with the simulation
int    UnitXPLastXPType ();   // last XP type queried
```

### Unit orders

```galaxy
order  OrderCreate (int player, abilcmd cmd, int flags, point targetPoint, bool queue);
order  OrderCreateTargetingUnit (int player, abilcmd cmd, unit target, int flags, bool queue);
order  OrderCreateTargetingPoint (int player, abilcmd cmd, point target, int flags, bool queue);
order  OrderCreateTargetingItem (int player, abilcmd cmd, unit target, int flags, bool queue);
bool   OrderIsValid (int player, order inOrder);
void   UnitOrderIssue (unit u, order inOrder, int orderQueueType);  // c_orderQueue*
void   UnitOrderIssueOverlay (unit u, order inOrder);
order  UnitOrderGet (unit u, int index);
int    UnitOrderCount (unit u, bool overlay);
abilcmd OrderGetAbilCmd (order inOrder);
point  OrderGetTargetPoint (order inOrder);
unit   OrderGetTargetUnit (order inOrder);
int    OrderGetFlag (order inOrder, int flag);
```

### Unit groups

```galaxy
unitgroup UnitGroup (string unitType, int player, region r, unitfilter filter, int maxCount);
unitgroup UnitGroupSearch (string unitType, int player, point p, fixed r, unitfilter filter, int maxCount);
unitgroup UnitGroupAlliance (int player, int alliance, region r, unitfilter filter, int maxCount);  // c_alliance*
int       UnitGroupCount (unitgroup group, int type);  // c_unitGroupAll/Active/AllOrObs
unit      UnitGroupUnit (unitgroup group, int index);
int       UnitGroupIndex (unitgroup group, unit u);
bool      UnitGroupHasUnit (unitgroup group, unit u);
void      UnitGroupAdd (unitgroup group, unit u);
void      UnitGroupRemove (unitgroup group, unit u);
void      UnitGroupClear (unitgroup group);
unitgroup UnitGroupCopy (unitgroup group);
void      UnitGroupIssueOrder (unitgroup group, order inOrder, int orderQueueType);
```

### Unit filters

```galaxy
unitfilter UnitFilter (int required, int excluded, int requiredExcluded, int requiredState);
           // c_unitFilter* constants
unitfilter UnitFilterStr (string filterString);  // "Enemy,Ground,Alive" etc.
void       UnitFilterSetState (unitfilter filter, int state, int flag, int value);
int        UnitFilterMatch (unit u, int player, unitfilter filter);
```

### Catalog

```galaxy
int    CatalogEntryCount (int catalog);  // c_catalogScope*
string CatalogEntryGet (int catalog, int index);
int    CatalogEntryScope (string entry);
bool   CatalogEntryExists (string entry);
int    CatalogFieldCount (int catalog);
string CatalogFieldGet (int catalog, int index);
string CatalogFieldIsArray (string entry, string field);
string CatalogFieldValueGet (int catalog, string entry, string field, int player);
string CatalogFieldValueGetAsText (int catalog, string entry, string field, int player);
string CatalogFieldValueGetTokens (int catalog, string entry, string field, int player, int tokenIndex);
int    CatalogFieldValueGetAsInt (int catalog, string entry, string field, int player);
fixed  CatalogFieldValueGetAsFixed (int catalog, string entry, string field, int player);
bool   CatalogFieldValueGetAsFlag (int catalog, string entry, string field, int player);
void   CatalogFieldValueSet (int catalog, string entry, string field, int player, string value);
void   CatalogFieldValueSetFlags (int catalog, string entry, string field, int player, string value);
string CatalogReferenceGet (string reference, int player);
string CatalogReferenceGetAsText (string reference, int player);
int    CatalogReferenceGetAsInt (string reference, int player);
fixed  CatalogReferenceGetAsFixed (string reference, int player);
void   CatalogReferenceSet (string reference, int player, string value);
bool   CatalogValidatorEvaluate (string validator, int player, unit source, unit target);
```

### Triggers

```galaxy
trigger TriggerCreate (string funcName);
void   TriggerDestroy (trigger t);
void   TriggerEnable (trigger t, bool enabled);
bool   TriggerIsEnabled (trigger t);
int    TriggerGetEvalCount (trigger t);
int    TriggerGetExecCount (trigger t);
void   TriggerExecute (trigger t, bool testConds, bool waitActions);
void   TriggerExecuteOnUnit (trigger t, bool testConds, bool waitActions, unit u);
void   TriggerAddEventMapInit (trigger t);
void   TriggerAddEventTimePeriodic (trigger t, fixed time);  // NOTE: 7vs1 caveat
void   TriggerAddEventPlayerLeft (trigger t, int player, int result);
void   TriggerAddEventUnitCreated (trigger t, int player, string unitType);
void   TriggerAddEventUnitDied (trigger t, int player, string unitType);
void   TriggerAddEventUnitDamaged (trigger t, unitref u);
void   TriggerAddEventUnitAcquiredTarget (trigger t, unitref u);
void   TriggerAddEventUnitTargetable (trigger t, unitref u);
void   TriggerAddEventUnitBehaviorChange (trigger t, unitref u);
void   TriggerAddEventUnitGainExperience (trigger t, unitref u);
void   TriggerAddEventUnitGainLevel (trigger t, unitref u);
void   TriggerAddEventUnitSelected (trigger t, int player, unitref u);
void   TriggerAddEventUnitRemoved (trigger t);
void   TriggerAddEventKeyDown (trigger t, int player, int key);
void   TriggerAddEventMouseClicked (trigger t, int player, int button);
// ... ~80 trigger event attachers total
```

### Event queries

```galaxy
unit      EventUnit ();
int       EventPlayer ();
string    EventUnitType ();
point     EventUnitPosition ();
unit      EventUnitTarget ();
unit      EventUnitDamager ();
string    EventUnitBehavior ();
int       EventUnitBehaviorChange ();
fixed     EventUnitDamageTaken ();
fixed     EventUnitDamageDealt ();
point     EventMouseClickedPoint ();
int       EventMouseClickedButton ();
int       EventKey ();
abilcmd   EventAbilCmd ();
```

### Timing

```galaxy
void   Wait (fixed time, int timeType);  // c_timeReal / c_timeGame
int    CurrentSynchronousGameTimeGet ();  // ms
int    SynchronousGameStartTimeGet ();
fixed  GameGetTime ();  // seconds, may diverge in MP
fixed  GameGetMissionTime ();
fixed  GameGetLocalTime ();
fixed  TimerGetElapsed (timer t);
fixed  TimerGetRemaining (timer t);
void   TimerStart (timer t, fixed initial, int type, bool recurring);  // c_timerType*
timer  TimerCreate ();
void   TimerRestart (timer t);
void   TimerPause (timer t, bool pause);
```

### Points / Regions

```galaxy
point  PointCreate (fixed x, fixed y);
point  PointFromXY (fixed x, fixed y);
fixed  PointGetX (point p);
fixed  PointGetY (point p);
fixed  PointDistance (point a, point b);
fixed  PointAngle (point a, point b);
point  PointWithOffset (point p, fixed dx, fixed dy);
point  PointWithOffsetPolar (point p, fixed distance, fixed angle);
bool   PointIsPassable (point p);
region RegionCircle (point center, fixed radius);
region RegionRect (fixed minx, fixed miny, fixed maxx, fixed maxy);
region RegionFromId (int regionId);
region RegionPlayableMap ();
bool   RegionContainsPoint (region r, point p);
void   RegionAddPoint (region r, point p);
void   RegionAddRect (region r, fixed minx, fixed miny, fixed maxx, fixed maxy);
```

### Players

```galaxy
int    PlayerLocal ();
int    PlayerCount ();
string PlayerName (int player);
bool   PlayerIsLocal (int player);
int    PlayerStatus (int player);  // c_playerStatus*
int    PlayerRace (int player);
int    PlayerColor (int player);
fixed  PlayerGetProperty (int player, int property);  // c_playerPropMinerals / Vespene / etc.
void   PlayerSetProperty (int player, int property, fixed value);
void   PlayerSetColor (int player, int color);
int    PlayerStartLocation (int player);
void   PlayerSetAlliance (int source, int target, int alliance, bool ally);  // c_allianceId*
```

### Camera

```galaxy
void   CameraLookAt (int player, point where, int whichCamera);  // c_camera*
void   CameraPan (int player, point where, fixed duration, int gap, int bound);
void   CameraShake (int player, int magnitude, int type);
void   CameraShakeStop (int player);
void   CameraSetValue (int player, int value, fixed val, fixed duration, int inOutType);
fixed  CameraGetValue (int player, int value);
void   CameraLock (int player, unit u, bool lock, bool follow);
point  CameraGetTarget (int player);
```

### Bank

```galaxy
bank   BankLoad (string name, int player);
bool   BankWait (string name);
bool   BankExists (string name, int player);
void   BankPreload (string name, int player);
void   BankSave (bank b, bool sync);
void   BankDelete (bank b);
string BankKeyCount (bank b, string section);
string BankKey (bank b, string section, int index);
bool   BankKeyExists (bank b, string section, string key);
int    BankValueGetAsInt (bank b, string section, string key);
fixed  BankValueGetAsFixed (bank b, string section, string key);
string BankValueGetAsString (bank b, string section, string key);
bool   BankValueGetAsFlag (bank b, string section, string key);
void   BankValueSetFromInt (bank b, string section, string key, int value);
void   BankValueSetFromFixed (bank b, string section, string key, fixed value);
void   BankValueSetFromString (bank b, string section, string key, string value);
void   BankValueSetFromFlag (bank b, string section, string key, bool value);
bool   BankConditionEvaluate (bank b, string section, string key);
```

### Sound

```galaxy
soundlink SoundLink (string soundId, int soundIndex);
sound    SoundPlayForPlayer (soundlink link, int owner, playergroup audible, fixed volume, fixed offset);
sound    SoundPlayAtPointForPlayer (soundlink link, int owner, playergroup audible, point pos, fixed height, fixed volume, fixed offset);
sound    SoundPlayOnUnitForPlayer (soundlink link, int owner, playergroup audible, unit u, fixed height, fixed volume, fixed offset);
sound    SoundLastPlayed ();
void     SoundStop (sound s, bool fade);
void     SoundSetVolume (sound s, fixed volume);
void     SoundChannelSetVolume (playergroup players, int channel, fixed volume, fixed duration);
void     SoundChannelMute (playergroup players, int channel, bool mute);
void     SoundtrackDefault (playergroup players, int category, string soundtrack, int cue, int index);
void     SoundtrackPlay (playergroup players, int category, string soundtrack, int cue, int index, bool makeDefault);
void     SoundtrackPause (playergroup players, int category, bool pause, bool fade);
void     SoundtrackStop (playergroup players, int category, bool fade);
void     SoundLengthQuery (soundlink link);
void     SoundLengthQueryWait ();
fixed    SoundLengthSync (soundlink link);
void     SoundSetReverb (string link, fixed duration, bool ambient, bool global);
```

### Actor

```galaxy
actor     ActorLastCreated ();
actor     ActorFromScope (actorscope as, string refName);
actorscope ActorScopeLastCreated ();
actorscope ActorScopeFromActor (actor a);
actor     ActorRefGet (actor a, string refName);
void      ActorSend (actor a, string msg);
void      ActorSendAsText (actor a, text msg);
void      ActorSendTo (actor a, string refName, string msg);
void      ActorSendToAsText (actor a, string refName, text msg);
void      ActorScopeSend (actorscope as, string msg);
void      ActorScopeKill (actorscope as);
void      ActorScopeOrphan (actorscope as);
void      ActorKill (actor a);
void      ActorLookAtStart (actor s, string lookAt, int weight, fixed time, actor t);
void      ActorLookAtStop (actor s, string lookAt, int weight, fixed time);
```

### Cinematic

```galaxy
int  CutsceneCreateNew (string inFilePath, point pos, fixed facing, playergroup players, bool inAutoPlay);
int  CutsceneLastCreated ();
void CutscenePlay (int id);
void CutscenePause (int id);
void CutsceneStop (int id);
void CutsceneSetTime (int id, int time);
void CutsceneGoToBookmark (int id, string name);
void CutsceneSetFilter (int id, string filter);
void WaitForCutsceneToEnd (int id);
void TriggerAddEventCutsceneBookmarkFired (trigger t, int id, string bookmark);
int  EventCutsceneId ();
string EventCutsceneBookmark ();
```

### Transmission

```galaxy
transmissionsource TransmissionSource ();
transmissionsource TransmissionSourceFromUnit (unit u, bool flash, bool overridePortrait, string anim);
transmissionsource TransmissionSourceFromUnitType (string unitType, bool overridePortrait);
transmissionsource TransmissionSourceFromModel (string modelLink);
transmissionsource TransmissionSourceFromMovie (string assetLink, bool subtitles);
int  TransmissionSendForPlayer (playergroup players, transmissionsource source, string portraitActor,
                                 string soundLink, string speakerName, string subtitleText,
                                 int durationType, fixed duration, int waitOption);
int  TransmissionLastSent ();
bool TransmissionIsComplete (int t);
void TransmissionClear (int t);
void TransmissionClearAll ();
```

### Dialogs (UI)

```galaxy
int  DialogCreate (int width, int height, bool modal);
int  DialogLastCreated ();
void DialogDestroy (int d);
void DialogSetVisible (bool visible, playergroup players, int d);
bool DialogIsVisible (int d, int player);
void DialogSetPosition (int d, int anchor, fixed x, fixed y);
int  DialogControlCreate (int d, int type);  // c_triggerControlType*
int  DialogControlLastCreated ();
void DialogControlSetVisible (int c, playergroup players, bool visible);
void DialogControlSetText (int c, playergroup players, text t);
text DialogControlGetText (int c, int player);
void DialogControlSetSize (int c, playergroup players, int width, int height);
void TriggerAddEventDialogControl (trigger t, int player, int c, int event);
int  EventDialogControl ();
```

### Visibility

```galaxy
revealer VisibilityCreate ();
void VisibilityDestroy (revealer v);
void VisibilitySetVisible (revealer v, region r, int player, bool visible);
bool VisibilityIsVisible (int player, point p);
bool VisibilityIsVisibleForPlayer (int player, point p);
void RegionVisEnable (region r, bool enable);
void RegionVisAddRevealer (region r, int player, bool visible);
```

## Catalog scopes (reference)

For the `c_catalogScope*` constants used by `CatalogEntryCount` etc., see
`catalog/fields-reference.md` §Complete scope list.

## See also

- `galaxy/syntax.md` (Galaxy syntax basics)
- `galaxy/natives-reference.md` (curated native reference)
- `catalog/fields-reference.md` (catalog field reference)
- `triggers/system.md` (trigger events)
