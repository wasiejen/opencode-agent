# FST logging — usage guide (worker-11, 2026-10-05, #131 build)

The five `DEBUG*` source flags are gone. Controls: start arguments, numpad
(gated by `-debug_numpad`), and CLI menu option 0.

## Files (both in the working directory, both append-only — old runs survive)
| file | what's in it |
|---|---|
| `fst.log` | everything at INFO+ from all areas (lifecycle, always-on WARNING/ERROR); DEBUG records when debug level is on. Format: `time - area - LEVEL - message` |
| `fst_flow.log` | the per-key flow stream only (the old DEBUG4, arrow vocabulary), one line per pipeline step. Only exists/writes while flow is enabled. Format: `time LEVEL message` |

## Start arguments
- `-log=debug|info|warning|error` — sets the main areas (`fst`, `fst.eval`,
  `fst.state`, `fst.macro`, `fst.config`). Default `info`.
  - `warning`/`error`: quietest — only problems.
  - `debug`: full diagnostics of state/eval/macro/config in `fst.log` — use for
    "why did the constraint misbehave" / first-call timing questions.
- `-flow` — enables `fst.flow` at DEBUG into its own `fst_flow.log`. This is the
  old `-debug4`/menu-0 stream; it does NOT go into `fst.log` (volume: ~30–60
  lines/s while tapping). Combine: `-log=debug -flow` for a full session trace.
- `-debug_numpad` — unchanged: required for the alt+num* toggles to work at all.
- `-debug` no longer exists — it is now an unknown-argument warning.

## Runtime toggles (numpad, needs `-debug_numpad`)
- `alt+num1` — toggle the root debug level on/off (D1+D2+D3 in one: state, eval,
  macro, config areas). `alt+num2`/`num3` do the same toggle (the separate D2/D3
  channels are gone).
- `alt+num4` — toggle the flow stream on/off (old DEBUG4 toggle).
- `alt+num5` — internal repr dump to console (unchanged).
- `alt+num7`/`num8` — real / all key-state dict dumps — now logged to
  `fst.state` at DEBUG, so they land in `fst.log` only while a debug level is on.
- CLI menu option 0 — same as `alt+num4` (flow toggle).

## Reading `fst_flow.log` for lag
Each key event appears as an IN/OUT pair (relative ms from start):
```
--> IN (12345) real ke(0x41, press)
-- rebind ke(0x41, press) -> ke(0x42, press)
--> OUT (12348) real ke(0x42, press)
```
Per-key pipeline latency = `OUT.t − IN.t`. Simulated events carry the `-- |`
marker and a `sim` field (`grep 'sim ' fst_flow.log`). `delayed by N ms` lines
sit next to the tap-group sends, and the macro `> playing macro:` lines next to
playback — so a slow macro shows its delay budget in the same file.

## Errors
- WARNING/ERROR are always on regardless of level and always land in `fst.log`
  (and on the console at WARNING+): unknown constraint names (config typos),
  unknown repeat aliases, rebind-not-found, no-macro-sequence, config-load
  errors, crashed macro/repeat tasks (`check_result` no longer swallows them),
  failed async dispatch.
- GUI mode: config errors still surface as toasts; headless: console + file.
