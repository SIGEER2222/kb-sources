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

## Initialization order

1. Mod dependencies load in declared order (innermost first).
2. Map's own data loads.
3. `MapInit` triggers fire (data fully resolved at this point).
4. Trigger libraries' `Init` functions run.
5. `InitMap` runs user-defined init code.

Order within a single init phase is non-deterministic across triggers unless
explicitly sequenced via `TriggerEnable` / `TriggerExecute`.

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
