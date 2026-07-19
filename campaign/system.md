# SC2 Campaign System

The campaign system provides the data and runtime support for SC2's
story-driven mission arcs: Wings of Liberty (Liberty), Heart of the Swarm
(Swarm), Legacy of the Void (Void), Nova Covert Ops (Nova), and the Co-op
campaign (allied commanders). Custom maps adopt the same pattern to
deliver multi-mission story content.

## Catalog types

### CCampaign

`CCampaign` entries (catalog `CampaignData.xml`) define top-level campaign
metadata: ID, name, description, race, mission list, story scene links.
Each campaign has a unique ID that the engine uses to track progress
across Bank files.

### CCharacter

`CCharacter` (catalog `CharacterData.xml`) is a named story character:
Raynor, Kerrigan, Zeratul, etc. Each entry has portrait model, voice over,
conversation link, default ability bar, and a "hero unit" link used when
the character appears in-mission as a controllable hero.

### CLocation

`CLocation` (catalog `LocationData.xml`) is a named point on the campaign
map (star map). Each location has a position, associated mission ID, icon,
and tooltip. The campaign travel UI uses these to render the star map
and let the player pick the next mission.

### CObjective

`CObjective` (catalog `ObjectiveData.xml`) is the data-driven objective
definition. Each entry has name, description, primary/secondary flag, and
default state. At runtime, these are converted to live objective instances
via the `Objective*` natives.

## Runtime: Objective API

Objectives are the player-visible mission goals shown in the upper-left
panel. Each objective has a state and a player group it's visible to.

### States

```galaxy
const int c_objectiveStateUnknown   = -1;
const int c_objectiveStateHidden    = 0;
const int c_objectiveStateActive    = 1;
const int c_objectiveStateCompleted = 2;
const int c_objectiveStateFailed    = 3;
```

### Creation and lifecycle

```galaxy
int ObjectiveCreate (text inName, text inDescription, int inState, bool inPrimary);
int ObjectiveCreateForPlayers (text inName, text inDescription, int inState, bool inPrimary, playergroup inPlayers);
int ObjectiveLastCreated ();
void ObjectiveDestroy (int inObjective);
void ObjectiveDestroyAll (playergroup inPlayers);

void ObjectiveShow (int inObjective, playergroup inPlayers, bool inShow);
bool ObjectiveVisible (int inObjective, int inPlayer);

void ObjectiveSetName (int inObjective, text inName);
text ObjectiveGetName (int inObjective);
void ObjectiveSetDescription (int inObjective, text inText);
text ObjectiveGetDescription (int inObjective);

void ObjectiveSetState (int inObjective, int inState);
int  ObjectiveGetState (int inObjective);

void ObjectiveSetPlayerGroup (int inObjective, playergroup inPlayers);
playergroup ObjectiveGetPlayerGroup (int inObjective);

void ObjectiveSetPrimary (int inObjective, bool inPrimary);
bool ObjectiveGetPrimary (int inObjective);

void ObjectiveSetPriority (int inObjective, int inPriority);
int  ObjectiveGetPriority (int inObjective);

void ObjectiveSetFirst (int inObjective);
void ObjectiveSetLast (int inObjective);
void ObjectiveSetAfter (int inObjective, int inAfterObjective);
void ObjectiveSetBefore (int inObjective, int inBeforeObjective);
```

### Primary vs secondary

- Primary objectives (`inPrimary=true`) appear at the top and their failure
  triggers mission failure. Their completion triggers mission victory when
  all primary objectives are complete.
- Secondary objectives (`inPrimary=false`) appear below primary and are
  optional. They typically reward bonus XP or unlocks.

The `c_primaryObjectivesId` / `c_secondaryObjectivesId` constants are
reserved IDs for the auto-created primary/secondary groups.

### Ordering

By default, objectives appear in creation order. `ObjectiveSetPriority`
sets an explicit priority; `ObjectiveSetFirst` / `ObjectiveSetLast` /
`ObjectiveSetAfter` / `ObjectiveSetBefore` allow fine-grained reordering
based on another objective's ID.

## Runtime: Campaign progress API

```galaxy
void CampaignInitAI ();
void CampaignProgressSetText (playergroup players, string campaignId, text inText);
void CampaignProgressSetImageFilePath (playergroup players, string campaignId, string inFilePath);
void CampaignProgressSetTutorialFinished (playergroup players, string campaignId, bool inFinished);
void CampaignProgressSetCampaignFinished (playergroup players, string campaignId, bool inFinished);
void CampaignProgressDeleteCampaignSave (playergroup players);
void CampaignProgressEnableCampaignSaves (playergroup players, bool inDisable);
void CampaignProgressEnableCampaignCompletedSaves (playergroup players, bool inDisable);

void GameSetNextCampaignIndex (playergroup inPlayerGroup, int inCampaignIndex);

void BankDeleteCampaignBanks (int player, int index);
```

Many of these are marked "Blizzard maps only" — they manipulate the
engine's built-in campaign progress UI which assumes the official
campaign IDs. Custom campaigns should use Bank files directly for
their progress persistence.

`CampaignInitAI` initializes the AI for campaign-mode maps (uses
`CampaignAI.galaxy` in TriggerLibs). This is different from melee AI.

## Co-op campaign integration (LibCOMI)

The Co-op campaign wraps these primitives in `LibCOMI` (Mission library).
Key entry points:

- `libCOMI_gf_InitGame_*` — mission lifecycle init.
- `libCOMI_gf_ObjectivePing_*` — objective pings on the minimap.
- `libCOMI_gf_MissionTransmission*` — Co-op-specific transmission
  helpers (auto-portrait based on commander).
- `libCOMI_gf_CoopAI_*` — Co-op AI attack waves and tech level buckets.

The `libCOOC_gf_CC_ObjectiveRegister` registers a CObjective entry with
the framework so it integrates with the Co-op mission UI.

## Campaign mission types

The Co-op framework recognizes several mission types via
`libCOOC_gf_CC_CampaignMapTypeCheck`:

- `c_campaignMapType*` — base campaign map types.
- Co-op-specific types inherit from these.

Custom maps can declare their mission type in the map's
`DocumentInfo.xml` `<MapType>` field to opt into Co-op framework features.

## Common pitfalls

- **`ObjectiveDestroyAll` is per-player group**: pass the right group to
  avoid destroying other players' objectives in shared-screen Co-op.
- **Objective state sync**: in multiplayer, objective state changes are
  not auto-synced. Each client must call `ObjectiveSetState` for the
  objectives it tracks; coordinate via TriggerSync or by calling from
  host-only code with replicated state.
- **`BankDeleteCampaignBanks` is destructive**: deletes the player's
  entire campaign progress for the given index. Use only when the player
  explicitly requests a reset.
- **Campaign progress natives are Blizzard-only**: `CampaignProgressSet*`
  natives are marked "Blizzard maps only" and may not work in custom
  maps. Use Bank files for custom campaign progress.
- **Mission type binding**: a map's mission type is set in
  `DocumentInfo.xml`. Changing it after the map is saved may corrupt
  the binary DocumentHeader — always re-save the map in the editor after
  changing mission type.
- **`ObjectiveSetPrimary` at runtime**: changing primary flag mid-mission
  is supported but visually disrupts the panel order. Prefer creating
  objectives with the correct primary flag from the start.
- **Campaign AI vs melee AI**: `CampaignInitAI` initializes AI for
  campaign maps which uses `CampaignAI.galaxy` and assumes preplaced
  units, build queues, and scripted attack waves. Don't mix with
  `MeleeInitAI` — they use different initialization paths.

## See also

- `cutscene/system.md` (cutscenes, transmissions, conversations)
- `sound/system.md` (campaign voice overs and soundtrack)
- `coop/commander-framework.md` (LibCOMI mission lifecycle)
- `bank/format.md` (campaign progress persistence)
- `catalog/format.md` (CCampaign, CCharacter, CLocation, CObjective entries)
