# SC2 Hero / Talent System

The Hero and Talent systems are the framework for hero-based modes (Co-op
commander heroes, custom hero arenas). Heroes are special units with
additional catalog data; talents are upgrade entries unlocked through a
tiered tree.

## Hero data (CHero)

`CHero` is the catalog type for hero definitions. Each entry lives in
`HeroData.xml` (see Blizzard's `core.sc2mod/base.sc2data/GameData/`). Key
fields:

- `id` — hero ID (must match a `CUnit` entry of the same name).
- `HeroUnit` — backing CUnit link (often the same as `id`).
- `Flags` — bit flags from `c_heroFlag*`:
  - `Flyer`, `Random`, `UsesMount`, `ExcludeFromMapIntro`,
    `GoodForModeA/B`, `AllowAIRandomSelection`, `ShowInStore`.
- `Role` — `c_heroRoleWarrior` / `Damage` / `Support` / `Specialist`.
- `Difficulty` — `c_heroDifficultyEasy` / `Medium` / `Hard` / `VeryHard`.
- `CutsceneSize` — `Small` / `Medium` / `Large` (selects cutscene variant).
- `State` — `c_heroStateLocked` / `Unlocked` (player-progression driven).

### Hero natives

```galaxy
string PlayerHero (int inPlayer);
void   PlayerSetHero (int inPlayer, string inHero);
void   SetHeroLeaderPanelEnabled (bool inRequired);
bool   GameAreHeroDuplicatesAllowed ();
```

`PlayerHero` returns the player's selected hero ID; `PlayerSetHero` binds
one. `GameAreHeroDuplicatesAllowed` reflects the lobby setting for whether
two players can pick the same hero.

## Talent tree

Each hero has a talent tree: a 2D grid of (tier, column) → talent link.
Players pick one talent per tier; the panel auto-shows when a tier becomes
available (unless disabled).

### Native API

```galaxy
string TalentTreeGetHeroTalentLink (int inPlayer, int inIndex);
bool   TalentTreeCanSelectHeroTalentTree (int inPlayer, int inTalentTreeIndex);
void   TalentTreeSetSelectedHeroTalentTree (int inPlayer, int inTalentTreeIndex);
int    TalentTreeGetSelectedHeroTalentTree (int inPlayer, int inTier);
int    TalentTreeGetSelectedHeroTalentTreeColumn (int inPlayer, int inTier);
void   TalentTreeClearTier (int inPlayer, int inTier);
bool   TalentTreeAllowed (int inPlayer, int inTalentTreeIndex);
void   SetTalentTierEnabled (int inTier, bool inEnabled);
void   SetTalentUpgradeRequired (bool inRequired);
void   SetTalentTreeHeroLevel (int inPlayer, int inLevel);
void   SetTalentsEnabled (bool inRequired);
void   SetTalentTreeSelectionPanelDismissAllowed (bool inAllowed);
void   SetTalentTreeSelectionPanelAutoShow (bool inAutoShow);
void   SetTalentTreePauseGameWhenSelectionPanelShown (bool inPauseGame);

void   TriggerAddEventHeroTalentTreeSelected (trigger t, int player);
void   TriggerAddEventHeroTalentTreeSelectionPanelShown (trigger t, int player);
void   TriggerAddEventHeroTalentTreeSelectionPanelHidden (trigger t, int player);
```

### Typical flow

1. `PlayerSetHero(player, heroId)` — bind hero.
2. `SetTalentsEnabled(true)` / `SetTalentUpgradeRequired(true)` — enable
   the tree.
3. `SetTalentTreeHeroLevel(player, level)` — set the hero's level; each
   tier unlocks at a specific level defined by the talent tree.
4. Listen for `TriggerAddEventHeroTalentTreeSelected` — fires when the
   player picks a talent.
5. Query selection via `TalentTreeGetSelectedHeroTalentTree(player, tier)`.

### Talent (CTalent)

A `CTalent` entry (catalog `TalentData.xml`) is an upgrade-like modifier
applied when a talent is unlocked. Each talent has:

- `id` — talent ID.
- `Modification` — `c_talentModification*`:
  - `None` — no stat change, just an unlock flag.
  - `CooldownReduction` — flat cooldown reduction on abilities.
  - `FlatModification` — flat numeric modifier.
  - `MultiplyLevelModification` — multiplier that scales with hero level.
  - `StringReplacement` — replaces a string field in another catalog entry.
  - `CatalogReplacement` — replaces a catalog field reference (e.g. swap
    a unit's weapon).

Talent modifications stack with the hero's base stats and behaviors. The
engine applies them when `PlayerAddTalent` is called; undo with
`PlayerRemoveTalent`.

```galaxy
void PlayerAddTalent (int inPlayer, string inTalent);
void PlayerRemoveTalent (int inPlayer, string inTalent);
bool PlayerHasTalent (int inPlayer, string inTalent);
```

## Hero abilities (CHeroAbil)

A hero ability (`CHeroAbil`, catalog `HeroAbilData.xml`) groups multiple
ability variants of the same skill — e.g. one ability with three talent
variants (Q1, Q2, Q3). The hero's ability bar shows the base ability;
selecting a talent swaps to the variant.

- `id` — base ability ID.
- `Abil` — backing `CAbil` link.
- `TalentTree` — talent tree entries that unlock this ability's variants.

The engine automatically swaps the active ability variant based on the
player's selected talents. Custom maps can override via
`CatalogFieldValueSet` if they need scripted variant swaps.

## Hero stats (CHeroStat)

`CHeroStat` (catalog `HeroStatData.xml`) defines named stat tracks like
`Strength`, `Agility`, `Intellect`, `Vitality`. Each entry has:

- `id` — stat ID.
- `Name` — display name.
- `Type` — `c_heroStatType*` (Primary / Secondary / Hidden).
- `Icon` — UI icon.

Stats are queried via `libCOOC_gf_CC_PlayerMasteryCategory*` APIs in
the Co-op framework, or via `CatalogFieldValueGet` with the `HeroStat`
scope.

## Hero cutscenes

A `CHeroCutscene` (no separate catalog file — cutscenes are referenced
inline) is a per-hero intro/outro cinematic. The
`c_heroCutsceneSize*` constants select which size variant to play.

## Common pitfalls

- **`PlayerSetHero` must be called before talent tree use**: if you skip
  it, `TalentTreeGetHeroTalentLink` returns empty strings and the panel
  shows nothing.
- **Talent tree tier gating**: each tier has a minimum hero level. The
  tree won't unlock until `SetTalentTreeHeroLevel` sets the level high
  enough. Don't forget to increment the level as the hero gains XP.
- **`PlayerHasTalent` checks runtime state**: returns whether the talent
  is currently applied via `PlayerAddTalent`. It doesn't check catalog
  existence — that's `CatalogEntryExists`.
- **`SetTalentTreePauseGameWhenSelectionPanelShown(true)` is the default**:
  the panel auto-pauses the game. Set to `false` for real-time hero
  modes where pausing isn't acceptable.
- **Talent modification types**: `MultiplyLevelModification` scales with
  `SetTalentTreeHeroLevel`. If you change the level after applying talents,
  the multiplier is recomputed automatically.
- **`TalentTreeClearTier` refunds the pick**: calling this for a tier
  that's already selected refunds the pick and lets the player re-pick.
  Useful for "respec" features.
- **Hero ability variant swap timing**: when a talent is added, the
  engine swaps the ability variant at the next `UnitRefreshAbilities`
  tick. If you need the swap to be instant, call
  `UnitAbilityReset` (or issue a no-op order to force a refresh).

## See also

- `coop/commander-framework.md` (commander hero unit lookup)
- `catalog/format.md` (CHero / CTalent / CHeroAbil catalog entries)
- `triggers/system.md` (talent selection events)
