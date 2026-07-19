# SC2 Runtime Observer Contract

A runtime observer captures dynamic evidence from a running SC2 instance. The
workspace is backend-neutral: a Neuro-compatible WebSocket is one transport, but
Bank watchers, game-log scanners, replay analysis, or a purpose-built observer
are all valid when they provide equivalent evidence.

## Run directory layout

```
evidence/runtime/<run-id>/
  run.json              # composition, commands, lock holder, run start/stop time
  events.jsonl          # ordered raw events observed during the run
  process.json          # SC2 process state: pid, alive, responsiveness
  script-errors/       # captured ScriptError*.txt files from GameLogs
  screenshots/         # optional screenshots captured during the run
  verdict.json         # acceptance criteria -> observed evidence + pass/fail
```

## Acceptance criteria

Each `runtime-verdict.json` entry maps an acceptance criterion to evidence:

- `passed`: evidence shows the criterion was met.
- `failed`: evidence shows the criterion was not met.
- `not_observed`: the observer did not capture evidence for the criterion.

A stage result is `passed` only when all required criteria are `passed`. An
empty observer result is never proof of absence.

## Readiness gate

The launcher waits for the readiness signal before collecting evidence. A
typical readiness check combines:

- SC2 process is alive and responsive (PID recorded, ping succeeds).
- No ScriptError files were created during the grace period (default 20 seconds
  after launch).
- The expected map-script init events fired (e.g. `MapInit` -> `InitMap`).

## ScriptError scanning

SC2 writes ScriptError files to `Documents/StarCraft II/GameLogs/` with a
timestamp prefix. The launcher scans this directory before and after the run to
detect errors created during the run. A non-empty diff fails the
`script-error-free` criterion.

Common ScriptError patterns observed in this project:

- `cmui_customization.galaxy:1890` — `libCOOC_gf_CC_CommanderIsDeveloping` is
  not a boolean expression in the effective CMRE dependency chain. See
  `projects/cmre-porting/stages/04-runtime-baseline/issues.json` `CMRE-ALENGER3-001`.
- Native `AchievementTermQuantitySet` restricted after init — disable the
  calling official StarCoop achievement triggers via the adapter.
- `Function declared but not defined` — a `LibXxx_*.galaxy` was not injected as
  an `include` into `MapScript.galaxy`.

## Process gate

After the readiness gate and the observation window, the launcher performs a
final liveness check:

- Process PID matches the recorded PID.
- Process is responsive (responds to a ping within 5 seconds).
- No new ScriptError files appeared during the observation window.

A failed process gate fails the `mission-runtime` criterion.

## Backend neutrality

The workspace registers several runtime-capable tools in `config/workspace.json`:

- `sc2-neuro-api` — Neuro-compatible WebSocket protocol.
- `sc2-neuro-wol` — Wings of Liberty runtime integration.
- `gary` — Neuro-compatible backend with manual action and test engine.

Any of these may serve as the observer. The run directory layout above is the
contract, not the transport.

## See also

- `triggers/system.md`
- `bank/format.md`
- `galaxy/script-error-codes.md`
- `projects/<active>/stages/<stage>/plan.md` for the project-specific runtime plan.
