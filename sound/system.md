# SC2 Sound System

SC2's audio system has three layers:

1. **Sound** — individual sound assets played via `SoundPlay*` natives.
2. **Soundtrack** — music tracks with categories, cues, and indexes.
3. **Sound channel** — per-category volume control (combat, dialogue, music, UI, etc.).
4. **Reverb / DSP** — environment effects applied to sound channels.
5. **VoiceOver** — data-driven narrative voice clips, catalog type `CVoiceOver`.

## Sound playback

### Sound links

A sound is identified by a `soundlink` handle, built from a catalog sound ID
and a sound index. Sounds defined in `SoundData.xml` have stable string IDs
that you pass to `SoundLink`:

```galaxy
soundlink SoundLink (string soundId, int soundIndex);
string   SoundLinkId (soundlink soundId);
int      SoundLinkAsset (soundlink soundId);
```

`soundIndex` selects one variant from a multi-variant sound entry (e.g.
random unit response quotes). Use `c_soundIndexAny` (-1) to let the engine
pick a variant.

### Playing sounds

```galaxy
void SoundPlayForPlayer (
    soundlink link, int inOwningPlayer, playergroup audibleMask,
    fixed volume, fixed offset);
void SoundPlayAtPointForPlayer (
    soundlink link, int inOwningPlayer, playergroup audibleMask,
    point inPoint, fixed height, fixed volume, fixed offset);
void SoundPlayOnUnitForPlayer (
    soundlink link, int inOwningPlayer, playergroup audibleMask,
    unit inUnit, fixed height, fixed volume, fixed offset);
void SoundPlaySceneForPlayer (
    soundlink link, int inOwningPlayer, playergroup audibleMask,
    unitgroup units, string animProps);
sound SoundLastPlayed ();
```

`SoundPlayAtPointForPlayer` and `SoundPlayOnUnitForPlayer` attach positional
information so the engine can apply 3D attenuation, doppler, and reverb
based on the listener's camera position.

`SoundPlaySceneForPlayer` synchronizes a group of units' animations with
the sound's duration — used for choreographed cutscenes where unit
animations must match audio beats.

### Sound control

```galaxy
void SoundPause (sound s, bool pause);
void SoundStop (sound s, bool fade);
void SoundStopAllModelSounds ();
void SoundStopAllTriggerSounds (bool fade);
void SoundSetVolume (sound s, fixed volume);
void SoundSetPosition (sound s, point position, fixed height);
void SoundSetOffset (sound s, fixed offset, int offsetType);  // c_soundOffsetStart / c_soundOffsetEnd
void SoundWait (sound s, fixed offset, int offsetType);
void SoundAttachUnit (sound s, unit u, fixed height);
```

### Sound length synchronization

Sound length varies by locale (localized audio files differ). To get a
synchronous length value across all players, use the three-step query:

```galaxy
void   SoundLengthQuery (soundlink info);
void   SoundLengthQueryWait ();
fixed  SoundLengthSync (soundlink info);
```

1. Call `SoundLengthQuery(link)` for every sound whose length you need.
2. Call `SoundLengthQueryWait()` once to block until all results return.
3. Call `SoundLengthSync(link)` to retrieve each length.

This is essential for synchronizing transmissions to sound duration in
multiplayer.

## Sound channels

Channels are independent volume sliders for sound categories. The engine
predefines channels matching `c_soundChannel*` (see `GameData/Sound.galaxy`
for the full list — `Dialogue`, `Mission`, `Music`, `Voice`, `UI`,
`Combat`, `Alert`, `Death`, `Ready`, `Spell`, `Build`, `Gather`, etc.).

```galaxy
void SoundChannelSetVolume (playergroup players, int channel, fixed volume, fixed duration);
void SoundChannelMute (playergroup players, int channel, bool mute);
void SoundChannelPause (playergroup players, int channel, bool pause);
void SoundChannelStop (playergroup players, int channel);
void SoundChannelDSPInsert (playergroup players, int channel, string dsp);
void SoundChannelDSPRemove (playergroup players, int channel, string dsp);
```

Volume is 0-100 percent. `duration` is the fade time in game seconds.

DSP (digital signal processing) effects are per-channel — useful for
applying low-pass filters during cinematics or distortion during
mind-control effects.

## Soundtracks (music)

A soundtrack is a music entry defined in `SoundtrackData.xml`. Soundtracks
have a `category`, a `cue`, and an `index`. The `cue` selects the
intro/loop/outro segments; the `index` picks a variant.

```galaxy
void SoundtrackDefault (playergroup players, int category, string soundtrack, int cue, int index);
void SoundtrackPlay (playergroup players, int category, string soundtrack, int cue, int index, bool makeDefault);
void SoundtrackPause (playergroup players, int category, bool pause, bool fade);
void SoundtrackSetContinuous (playergroup players, int category, bool continuous);
void SoundtrackSetDelay (playergroup players, int category, fixed delay);
void SoundtrackStop (playergroup players, int category, bool fade);
void SoundtrackStopCurrent (playergroup players, int category, bool fade);
void SoundtrackWait (string soundtrack);
```

- `SoundtrackDefault` sets the music that plays when nothing else is active.
- `SoundtrackPlay` plays a specific track immediately, optionally replacing
  the default.
- `SoundtrackPause` pauses the music without resetting state.
- `SoundtrackSetContinuous` makes the soundtrack loop indefinitely.
- `SoundtrackWait` blocks until the named soundtrack finishes.

### Cue and index

- `c_soundtrackCueAny` (-1) — engine picks based on context.
- `c_soundtrackIndexAny` (-1) — random variant.

A `CSoundtrack` entry in `SoundtrackData.xml` defines cue segments
(`Intro`, `Loop`, `Outro`) and how they map to actual sound assets. The
engine crossfades between segments automatically.

## Sound mix snapshots

A `SoundMixSnapshot` (catalog type `CSoundMixSnapshot`) is a saved set of
per-channel volume values. Use it to apply a complete mix change (e.g.
"cinematic mode" quiets combat / environment while boosting dialogue).

The Co-op framework defines `libCOOC_ge_VolumeChannelModeCampaign_*`
presets (`Cinematic`, `Speech`, `Game`, `Victory`, `LowHealthBegin`,
`LowHealthEnd`, `MissionLaunchUI`, etc.) that wrap the snapshot API. See
`LibCOOC_h.galaxy` for the full list.

## Reverb

Reverb applies environmental echo/decay to a sound channel. Each `CReverb`
entry (catalog `ReverbData.xml`) defines a reverb preset.

```galaxy
void SoundSetReverb (string inReverbLink, fixed inDuration, bool inAmbient, bool inGlobal);
void SoundSetReverbForPlayers (playergroup players, string inReverbLink, fixed inDuration, bool inAmbient, bool inGlobal);
void SoundSetFactors (fixed distance, fixed doppler, fixed rolloff);
```

- `inDuration` — fade time in seconds.
- `inAmbient` — apply to ambient sounds only.
- `inGlobal` — apply to all sounds regardless of position.
- `SoundSetFactors` adjusts 3D sound properties globally: distance scaling,
  doppler intensity, and rolloff curve.

## VoiceOver

`CVoiceOver` entries (catalog `VoiceOverData.xml`) are data-driven narrative
voice clips. They bundle a sound asset, subtitle text, portrait, and
speaker identity — primarily used by the campaign and Co-op commander
select screens.

Query VoiceOver data via the standard `CatalogFieldValueGet` natives
with `c_catalogScopeVoiceOver`. LibCOMI's `libCOOC_gf_CC_CommanderVoiceOver`
returns the VoiceOver entry for a commander's selection audio.

## Common pitfalls

- **Sound length varies by locale**: a sound's length is not known
  synchronously across multiplayer clients. Always use the three-step
  `SoundLengthQuery` / `SoundLengthQueryWait` / `SoundLengthSync` pattern
  before computing transmission durations.
- **Sound channel mute is per-player group**: `SoundChannelMute` mutes
  only for players in the group. Pass `PlayerGroupAll()` for global mute.
- **Soundtrack category**: each category has an independent default and
  active soundtrack. Use different categories for different layers (e.g.
  ambient music on `Music`, action music on `Mission`).
- **Reverb global flag**: `SoundSetReverb` with `inGlobal=true` affects
  positional and non-positional sounds. With `inGlobal=false`, only
  positional sounds get the reverb based on their location.
- **DSP and channel interaction**: DSPs are channel-scoped. A single
  channel can have multiple DSPs active simultaneously; remove them by
  name when no longer needed.
- **`SoundStopAllModelSounds` is global**: stops every model-attached
  sound on the map. Use sparingly — usually you want
  `SoundChannelStop(players, c_soundChannelCombat)` instead.
- **Streaming vs preloaded**: long sound assets stream from disk; the
  `SoundSourceSetStreamingAllowed` flag on transmissions lets you block
  streaming for shorter clips that must play immediately.
- **VoiceOver is data**: `CVoiceOver` entries are pure data. To play one,
  look up its `Sound` field via `CatalogFieldValueGet` and pass the
  result to `SoundPlayForPlayer` or `TransmissionSendForPlayer`.

## See also

- `cutscene/system.md` (Transmission API for in-game dialogue)
- `campaign/system.md` (campaigns use voiceovers heavily)
- `coop/commander-framework.md` (commander select VO)
- `catalog/format.md` (CSound, CSoundtrack, CVoiceOver catalog entries)
