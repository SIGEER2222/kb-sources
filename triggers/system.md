# SC2 Trigger System

Triggers are the primary logic layer of an SC2 map. The Trigger Editor GUI lets
authors compose triggers as Event-Condition-Action (ECA) trees; on save, the GUI
compiles to Galaxy script (`MapScript.galaxy` plus `TriggerLibs/*.galaxy`).

## ECA structure

- **Event**: when the trigger fires (e.g. `UnitDied`, `KeyPressed`, `TimerPeriodic`).
- **Condition**: a boolean expression that must evaluate true for actions to run.
  Multiple conditions combine with `And` / `Or` / `Not`.
- **Action**: side effects executed when conditions hold (create units, issue
  orders, display messages, modify bank, etc.).

### Parameter types

Each ECA parameter has a declared type. Common parameter value sources:

- **Preset**: enumerated constants from `c_*` (e.g. `c_unitCountAlive`).
- **Variable**: a global or local variable declared in the GUI.
- **Function**: another ECA returning a value (nested as a sub-parameter).
- **Custom Script**: inline Galaxy expression. Used when the GUI lacks a wrapper.
- **Value**: literal numbers, strings, fixed-point.

Sub-parameters (parameters of parameters) nest recursively; the GUI renders them
as indented tree nodes. Each compiled parameter is a Galaxy function call.

### Custom Script elements

Insert raw Galaxy in any parameter slot via "Custom Script" element type. At
the top level, **Custom Script** action elements let authors embed arbitrary
statements between GUI actions. They compile verbatim into `MapScript.galaxy`.

Custom scripts can `include "TriggerLibs/<name>"` to pull in external Galaxy
files imported via the Importer module. Avoid `_h` suffix — the include directive
loads both the declaration header and the implementation when given the bare
basename.

## GUI vs custom script

- The GUI covers approximately 99.9% of native functionality. Custom script
  elements (`数据 > 新建 > 新建自定义脚本`) let advanced authors embed raw Galaxy
  in any trigger.
- GUI custom functions (动作元素 and 条件元素) compile to Galaxy functions and
  can be called across triggers.
- Custom scripts can `include "TriggerLibs/<name>"` to pull in external Galaxy
  files imported via the Importer module.

## Trigger library auto-loading

A mod's trigger library is auto-loaded by the engine when the mod's data editor
sets the `TriggerLibs Identifier` (识别符) field on `SC2 Gameplay Defaults`:

```
数据模块 > 对象类型: 游戏性数据 > 游戏性值名称: SC2游戏性默认设定
> 右侧 触发器库-识别符: 添加 library ID
```

The library ID is the file's basename (without `_h` / `.galaxy`). Loaded
libraries' `Init` functions are called during map init.

A library file (`Lib<Name>.galaxy`) follows this pattern:

```galaxy
include "TriggerLibs/NativeLib"
include "LibCOMU"
include "Lib<Name>_h"

// External Library Initialization
void lib<Name>_InitLibraries () {
    libNtve_InitVariables();
    libCOMU_InitVariables();
}

// Trigger function declarations (one per trigger in the GUI)
bool lib<Name>_gt_<TriggerName>_Func (bool testConds, bool runActions) {
    if (!runActions) { return true; }
    // Actions
    return true;
}

void lib<Name>_gt_<TriggerName>_Init () {
    lib<Name>_gt_<TriggerName> = TriggerCreate("lib<Name>_gt_<TriggerName>_Func");
    TriggerAddEventMapInit(lib<Name>_gt_<TriggerName>);
}

void lib<Name>_InitTriggers () {
    lib<Name>_gt_<TriggerName>_Init();
}

// Library Initialization (idempotent)
bool lib<Name>_InitLib_completed = false;

void lib<Name>_InitLib () {
    if (lib<Name>_InitLib_completed) { return; }
    lib<Name>_InitLib_completed = true;
    lib<Name>_InitLibraries();
    lib<Name>_InitTriggers();
}
```

This pattern is used by all official Blizzard libraries (`LibCOMI`, `LibCOMU`,
`LibCOOC`, `LibCOUI`, `Lib9B7202B2`, etc.). The `_InitLib` function is the
entrypoint called by the engine during map init.

## Initialization order

1. Mod dependencies load in declared order (innermost first).
2. Map's own data loads.
3. `MapInit` triggers fire (data fully resolved at this point).
4. Trigger libraries' `Init` functions run.
5. `InitMap` runs user-defined init code.

Order within a single init phase is non-deterministic across triggers unless
explicitly sequenced via `TriggerEnable` / `TriggerExecute`.

The compiled `MapScript.galaxy` calls these stages in this fixed order; you
cannot reorder them from the GUI.

## Common pitfalls

- `TriggerAddEventTimePeriodic` does not reliably fire on 7vs1 campaign maps.
  Use a `while (true) { Wait(period, c_timeReal); ... }` loop inside a function
  triggered by `MapInit` instead.
- Triggers created during the init phase can fire before all preplaced units
  exist. Defer trigger creation to a separate init stage or use `Wait`.
- Variables in triggers are global by default; locals must be explicitly declared
  as local in the GUI.
- The minimum `Wait` granularity is 1/16 second (one game tick). Shorter waits
  are silently rounded up.
- `Wait` with `c_timeGame` pauses during cinematic / dialog; use `c_timeReal`
  for real-time pacing regardless of game state.
- `TriggerExecute` is synchronous and runs in the calling thread; it does not
  fork a new thread. Use `TriggerCreate` + event attachment for async behavior.
- Trigger functions return `bool`; returning `false` from `testConds=true` call
  aborts the action list without running it.

## Categories and libraries

- **Categories** organize triggers inside one map for editing only; they have
  no runtime effect.
- **Libraries** are reusable trigger collections that can be saved to a mod and
  auto-loaded into any consuming map. They are the primary mechanism for sharing
  behavior across maps.

## See also

- `galaxy/syntax.md`
- `editor/editor-overview.md` §5
- `runtime-contracts/observer.md`
- `coop/commander-framework.md` (Co-op commander library pattern)
