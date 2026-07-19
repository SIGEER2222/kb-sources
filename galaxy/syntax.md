# Galaxy Script Syntax

Galaxy is the C-like scripting language used by StarCraft II's trigger system and
custom script elements. It replaces Warcraft III's JASS. Trigger GUI compiles to
Galaxy before execution.

## Source references

- `legacy/galaxy-tutorial.txt`: original community tutorial (Chinese).
- `galaxy/CampaignLib_h.galaxy`, `galaxy/MapScript-sample.galaxy`: real generated code samples.
- `editor/editor-overview.md` §7: language overview and C-differences summary.

## Top-level structure

A typical generated `MapScript.galaxy` contains, in order:

```galaxy
include "TriggerLibs/<LibName>"        // no '#', no .galaxy suffix
include "TriggerLibs/<LibName>_h"     // header-only declarations

// 1. global variable declarations (must be at top)
int gv_someCounter;
unit gv_lastAttacker;

// 2. function definitions
bool gv_IsUnitAlive (unit u) {
    return !UnitIsAlive(u) == false;
}

// 3. trigger creation and initialization
void gt_MyTrigger_Init () {
    gt_MyTrigger = TriggerCreate("gt_MyTrigger_Func");
    TriggerAddEventUnitDied(gt_MyTrigger, c_unitAny);
}

// 4. init entry point called from MapInit
void InitMap () {
    gt_MyTrigger_Init();
}
```

## Type system

| Type | Description |
|------|-------------|
| `int` | 32-bit signed integer. |
| `fixed` | Fixed-point decimal (replaces float/double). |
| `bool` | `true` / `false`. |
| `string` | Null-terminated string. |
| `text` | Localized text reference (separate from `string`). |
| `unit`, `player`, `region`, `point`, `unitgroup`, ... | Game object handles. |
| `unit[10] gv_units;` | Array. Dimensions follow the type, not the variable name. Max 4 dimensions. |
| `struct` | User-defined records; limited compared to C. |

No `float`, `double`, `unsigned`, `signed`, `switch`, `for`, `goto`, `sizeof`,
`++`, `--`, `/* */` block comments, `#include`, `#define`, or any preprocessor.

## Statements and blocks

- `if` / `else if` / `else` bodies and `while` bodies MUST be enclosed in `{}` even
  when single-line. Single-statement bodies without braces are a compile error.
- Only `//` line comments are supported.
- `break` / `continue` are valid inside `while`.
- No `for`; iterate with `while (i < n) { ... i = i + 1; }`.

## Includes

- Syntax: `include "TriggerLibs/MyLib"` (no `#`, no `.galaxy`).
- A library `MyLib.galaxy` and its header `MyLib_h.galaxy` are both included:
  - `include "TriggerLibs/MyLib_h"` provides declarations only.
  - `include "TriggerLibs/MyLib"` provides the implementation.
- The engine loads files by stripping the `_h` suffix; if you only include the header,
  the implementation file is NOT loaded. Always include both for libraries with code.
- Mod-loaded trigger libraries are auto-registered via the data editor field
  `TriggerLibs Identifier` (识别符) on `SC2 Gameplay Defaults` rather than via `include`.

## Functions

- Declaration: `<type> <name> (<args>) { ... }`.
- `void` for procedures.
- No function pointers; cannot cast between types via parentheses.
- Native functions (also called "natives") are provided by the engine; consult
  `galaxy/natives-reference.md` and the SC2 API docs at https://mapster.talv.space.
- GUI custom functions compile to Galaxy functions; cross-library calls require
  the called library to be included.

## Memory

- All local types are garbage-collected; no `malloc`, `new`, or manual `delete`.
- Handles (unit, player, ...) are reference-counted by the engine.
- String literals are interned; runtime concatenation uses `StringToText` /
  `TextExpression` to remain localized.

## Common pitfalls observed in this project

- `TriggerAddEventTimePeriodic` does not fire reliably in 7vs1 campaign maps; use
  `Wait(3.0, c_timeReal)` inside a `while (true)` loop instead.
- `CatalogFieldValueGet` returns race strings as `Terr` / `Zerg` / `Prot`, not
  `Terran` / `Zerg` / `Protoss`.
- `UnitRemove` during the map init phase silently no-ops; defer with a trigger and
  `Wait(2.0, c_timeReal)`.
- Chinese variable names break the editor's "get script id by name" lookup and
  cause map save failures; use ASCII identifiers only.
- Map paths containing CJK characters cannot store components reliably; use ASCII paths.

## See also

- `galaxy/natives-reference.md`
- `galaxy/script-error-codes.md`
- `editor/editor-overview.md` §7
