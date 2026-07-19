# SC2 Cutscene / Transmission / Conversation System

Three cooperating systems drive SC2's narrative and cinematic content:

1. **Cutscene** — pre-rendered cinematic scenes built in the Cutscene Editor
   (`.sc2cutscene` files, native `Cutscene*`).
2. **Transmission** — in-game dialogue overlay (portrait + sound + subtitles),
   native `Transmission*`.
3. **Conversation** — interactive branching dialogue (reply tree + state),
   native `Conversation*` and `ConversationData*`.

## Cutscenes

Cutscenes are authored in the Cutscene Editor module (a separate editor
panel inside the Galaxy Editor). A cutscene is a timeline of camera moves,
model animations, effects, sound, and text; it's saved as a
`<name>.sc2cutscene` asset and played back at runtime.

### Native API

```galaxy
int  CutsceneCreateNew (string inFilePath, point pos, fixed inFacing,
                        playergroup players, bool inAutoPlay);
int  CutsceneCreate (string inFilePath, point pos, playergroup players,
                     bool inAutoPlay);
int  CutsceneLastCreated ();
void CutscenePlay (int inCutscene);
void CutscenePause (int inCutscene);
void CutsceneStop (int inCutscene);
void CutsceneSetTime (int inCutscene, int inTime);
void CutsceneGoToBookmark (int inCutscene, string inBookmarkName);
void CutsceneGoToNextBookmark (int inCutscene);
void CutscenePlayCutsceneRangeOverTime (int inCutscene,
        string inBookmarkStart, string inBookmarkEnd, fixed inDuration);
void CutsceneSetGlobalFilter (string inFilter);
void CutsceneSetFilter (int inCutscene, string inFilter);
int  CutsceneGetTriggerControl (int inControlId);
void WaitForCutsceneToEnd (int inCutscene);
```

### Events

```galaxy
void TriggerAddEventCutsceneBookmarkFired (trigger t, int inCutscene, string inBookmarkName);
void TriggerAddEventCutsceneEndSceneFired (trigger t, int inCutscene);
void TriggerAddEventCutsceneConversationLine (trigger t, int inCutscene, string inConversationLine);

int    EventCutsceneId ();
string EventCutsceneBookmark ();
```

A cutscene fires `BookmarkFired` events as the playhead crosses named
bookmarks defined in the editor. This is the primary way to synchronize
Galaxy logic with cinematic timing — e.g. spawning a unit when the camera
reaches a specific frame.

### Filter

`CutsceneSetGlobalFilter` / `CutsceneSetFilter` toggle visibility of cutscene
tracks based on a filter string. Tracks in the editor are tagged with filter
names; at runtime, only tracks whose filter matches the active filter set are
played. Use this to swap language audio, hide non-critical effects in
multiplayer, or branch a single cutscene for multiple story outcomes.

### Cutscene vs Trigger Control

`CutsceneGetTriggerControl(int inControlId)` returns the cutscene ID
associated with a dialog control of type `c_triggerControlTypeCutscene` (22)
or `c_triggerControlTypeHeroCutscene` (24). Use this to bind a cutscene
playback to a UI panel.

## Transmissions

A transmission is the standard in-game dialogue: a portrait, a sound asset,
and a text string shown in the bottom-left dialog. The player can click to
skip; multiplayer peers see the same transmission synchronized.

### Transmission sources

```galaxy
transmissionsource TransmissionSource ();
transmissionsource TransmissionSourceFromUnit (unit u, bool flash,
        bool overridePortrait, string anim);
transmissionsource TransmissionSourceFromUnitType (string unitType, bool overridePortrait);
transmissionsource TransmissionSourceFromModel (string modelLink);
transmissionsource TransmissionSourceFromMovie (string assetLink, bool subtitles);

void TransmissionSourceSetPauseAllowed (transmissionsource source, bool allowed);
void TransmissionSourceSetStreamingAllowed (transmissionsource source, bool allowed);
void TransmissionSourceSetBypassMessageLog (transmissionsource source, bool bypass);
```

A `transmissionsource` describes where the portrait comes from. Use
`FromUnit` for an in-world unit's portrait, `FromModel` for a standalone
model, `FromMovie` for a pre-rendered video file.

### Sending transmissions

```galaxy
int TransmissionSendForPlayer (
    playergroup players,
    transmissionsource source,
    string portraitActorLink,
    string soundLink,
    string speakerName,
    string subtitleText,
    int durationType,            // c_durationTypeAdd / c_durationTypeSet
    fixed duration,
    int waitOption               // c_waitOptionWait / c_waitOptionDontWait
);
int  TransmissionLastSent ();
void TransmissionClear (int t);
bool TransmissionPlayerHasActiveTransmission (int inPlayerId);
void TransmissionClearAll ();
void TransmissionClearGroup (playergroup players);
void TransmissionSetOption (int inOptionIndex, bool inValue);
void TransmissionWait (int t, fixed offset);
bool TransmissionIsComplete (int t);
```

`TransmissionSendForPlayer` is the workhorse: it plays the sound, shows the
portrait and subtitle, and returns an ID you can later `Wait` on. Pass
`c_waitOptionWait` to block until the transmission finishes (typical for
scripted dialogue), or `c_waitOptionDontWait` to fire-and-forget.

### Co-op transmission options

LibCOMI provides Co-op-specific transmission helpers that auto-pick portrait
based on the active commander. See `libCOMI_gf_MissionTransmission` and
related wrappers in `LibCOMI_h.galaxy`. These wrap `TransmissionSendForPlayer`
with the mission's `VolumeChannelMode` presets and cue sound behavior.

## Conversations

Conversations are interactive branching dialogues. They have:

- A **state** tree (key-value pairs of strings → integers).
- **Replies** (clickable options presented to the player).
- **Lines** (spoken dialogue, each tied to a state and a transmission).
- **Choices** (decisions persisted across conversation sessions).

### Native API (data-driven)

```galaxy
int    ConversationDataStateIndexCount (string inStateId);
string ConversationDataStateIndex (string inStateId, int inIndex);
text   ConversationDataStateName (string stateIndex);
text   ConversationDataStateText (string stateIndex, string inInfoName);
fixed  ConversationDataStateFixedValue (string stateIndex, string inInfoName);
string ConversationDataStateImagePath (string stateIndex);
int    ConversationDataStateImageEdge (string stateIndex);
string ConversationDataStateAttachPoint (string stateIndex);
string ConversationDataStateMoviePath (string stateIndex);
string ConversationDataStateModel (string stateIndex, string inInfoName);
string ConversationDataStateUpgrade (string stateIndex, string inInfoName);
abilcmd ConversationDataStateAbilCmd (string stateIndex, string inInfoName);

void   ConversationDataRegisterCamera (string camIndex, string charIndex,
        camerainfo c, trigger t, bool wait);
void   ConversationDataRegisterUnit (string stateIndex, unit u);
void   ConversationDataRegisterPortrait (string stateIndex, int p);

void   ConversationDataStateSetValue (string stateIndex, int value);
int    ConversationDataStateGetValue (string stateIndex);

int    ConversationDataChoiceCount (string convId);
string ConversationDataChoiceId (string convId, int index);
void   ConversationDataChoiceSetState (string convId, string choiceId, int state);
int    ConversationDataChoiceGetState (string convId, string choiceId);
void   ConversationDataChoiceSetPicked (string convId, string choiceId, bool picked);
bool   ConversationDataChoiceGetPicked (string convId, string choiceId);
void   ConversationDataChoiceSetPickedCount (string convId, string choiceId, int count);
int    ConversationDataChoiceGetPickedCount (string convId, string choiceId);

int    ConversationDataLineCount (string convId);
```

### Native API (UI)

```galaxy
int  ConversationCreate (bool visible);
int  ConversationLastCreated ();
void ConversationDestroy (int intId);
void ConversationDestroyAll ();
void ConversationShow (int intId, playergroup to, bool visible);
bool ConversationVisible (int intId, int player);

int  ConversationReplyCreate (int intId, text inText);
int  ConversationReplyLastCreated ();
void ConversationReplyDestroy (int intId, int replyId);
void ConversationReplyDestroyAll (int intId);
void ConversationReplySetText (int intId, int replyId, text inText);
text ConversationReplyGetText (int intId, int replyId);
void ConversationReplySetState (int intId, int replyId, int state);
int  ConversationReplyGetState (int intId, int replyId);
int  ConversationReplyGetIndex (int intId, int replyId);

void TriggerAddEventConversationReplySelected (trigger t, int player, int intId, int replyId);
int  EventConversation ();
int  EventConversationReply ();
```

### State model

`ConversationDataStateGetValue` / `SetValue` is the persistent state
key-value store. State IDs are strings like `Mission_Raynor_01_State_PlayerChoseB`
and values are integers (0/1/2/...). The Co-op campaign uses these to gate
story branches and persist decisions across maps via Bank files.

### Camera registration

`ConversationDataRegisterCamera` binds a `camerainfo` to a (camIndex, charIndex)
pair; the conversation then uses that camera when displaying a line for the
matching character. This lets a single conversation flow swap cameras
dynamically based on which character is speaking.

## Common pitfalls

- **Transmission in multiplayer**: `TransmissionSendForPlayer` syncs only
  to the players in the `players` group. If the group is empty, the
  transmission is silent. Use `PlayerGroupAll()` for global dialogue.
- **Transmission duration**: `c_durationTypeAdd` adds the duration to the
  sound's natural length; `c_durationTypeSet` overrides it. Use `Set` when
  you need a fixed minimum display time for short sound bites.
- **Cutscene pauses game by default**: cutscenes auto-pause game time. To
  allow gameplay during a cutscene, set the cutscene's "Pause Game" flag off
  in the editor and use `Wait(c_timeReal)` in Galaxy instead of `Wait(c_timeGame)`.
- **Conversation state persistence**: state set via
  `ConversationDataStateSetValue` is in-memory only. To persist across
  sessions, save the relevant state values to a Bank and restore on
  next map load.
- **Cutscene bookmark events** fire on every player's machine independently.
  If your Galaxy logic is host-authoritative, gate it with a
  `PlayerGroupOwner` check.
- **TransmissionSourceFromMovie subtitles**: when `subtitles=true`, the
  movie's subtitle track is displayed in the transmission UI; otherwise
  only the speaker name is shown.
- **Sound asset must exist**: if the `soundLink` passed to
  `TransmissionSendForPlayer` doesn't resolve, the transmission will display
  text but be silent, and `TransmissionIsComplete` may never report true.

## See also

- `sound/system.md` (soundtrack, sound mix, voiceover)
- `campaign/system.md` (campaign structure that wraps these systems)
- `coop/commander-framework.md` (LibCOMI transmission helpers)
