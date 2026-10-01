# #131 — FST DEBUG → structured logging: research & design

Explorer: explorer-4, 2026-10-01, looprun 2026-10-01_03-27.
Read-only over FST repo `C:/Users/Wasiejen/Repos/Free-Snap-Tap` (checkout `fst_work3`,
b513bf2, clean at spec time). All line numbers refer to that checkout.
FST code was **not** modified.

Goal: instrument FST so logging helps find lag causes, documents errors, and
associates them to code areas — without overwhelming; control via start-argument
log level and/or separate log files per area; design the DEBUG4 successor (kept as
a formatted log); decide which DEBUG outputs die; recommend logging vs DEBUG
constants.

---

## 1. Current state (verified by this audit)

### 1.1 The five flags
`CONSTANTS.DEBUG / DEBUG2 / DEBUG3 / DEBUG4 / DEBUG_NUMPAD` — module-level booleans
in `fst_manager.py:26-44`, all default `False`.

Three toggling surfaces, all hand-wired:
1. **Source edit** — `free_snap_tap.py:30-41`: each flag has a commented `= True`
   variant right below. This is the maintainer's live workflow today.
2. **Start arguments** — `fst_manager.py:1363-1367` (`apply_start_arguments`):
   only `-debug` (sets `self.DEBUG` **and** `CONSTANTS.DEBUG`) and
   `-debug_numpad` (sets `CONSTANTS.DEBUG_NUMPAD`). **`-debug2/-debug3/-debug4`
   do not exist** — those flags are only reachable by source edit, numpad, or the
   CLI menu.
3. **Runtime numpad** — gated by `CONSTANTS.DEBUG_NUMPAD`
   (`fst_keyboard.py:684-685` → `check_debug_numpad_actions`,
   `fst_keyboard.py:1012-1027`): `alt+num1..4` toggle `DEBUG..DEBUG4`;
   `alt+num5` internal repr dump; `alt+num7/8` state-dict dumps.
4. **CLI menu** — `fst_manager.py:1881/1892`: menu option `0` toggles `DEBUG4`
   only.

### 1.2 Two parallel flag channels (latent bug, found by this audit)
`Argument_Manager.__init__` snapshots the flags at `fst_manager.py:1287-1290`
(`self.DEBUG = CONSTANTS.DEBUG` etc.). Some sites read the snapshot
(`self.DEBUG` in `apply_start_arguments` `fst_manager.py:1360,1374`;
`self._arg_manager.DEBUG2` in `macro_task` `fst_keyboard.py:953`), the rest read
`CONSTANTS.*` directly. The numpad/menu runtime toggles write **only** to
`CONSTANTS`, so after a runtime toggle:
- `-debug`-style reads of `self.DEBUG` are stale, and
- `macro_task`'s `self._arg_manager.DEBUG2` print (`fst_keyboard.py:953`) is
  invisible to the numpad-2 toggle.
Also `reset_global_variable_changes` deliberately does NOT reset the DEBUG
values (commented out at `fst_manager.py:1308-1310`) — so runtime toggles survive
focus changes.

### 1.3 Logging today (embryonic)
- Only config site: `logging.basicConfig(filename='fst.log', filemode='w',
  format=…, level=logging.DEBUG)` at `free_snap_tap.py:21-26`.
  - `filemode='w'` → **the log is overwritten on every launch** — crash/error
    evidence from a previous run is destroyed by the next start. This is the
    single biggest gap for "documenting errors".
  - One file, one level (DEBUG) for everything, no per-area separation.
  - Path is CWD-relative (`fst.log` next to the exe when run as nuitka onefile).
- ~14 live `logger.*` calls outside tests (fewer than the spec's estimate — see
  discrepancies §7): `free_snap_tap.py:151,164,178,212,216`;
  `fst_keyboard.py:95,102,107,817,921,931,946,967` (+ one commented `912`);
  `fst_manager.py:354,776`; `fst_tasks.py:29`.
- Module loggers exist in `free_snap_tap.py:27`, `fst_manager.py:24`,
  `fst_keyboard.py:22`.
- The hot path (win32 event filter, ~1138-line `fst_keyboard.py`,
  `fst_manager.py` 1943 lines) uses `print()` guarded by `CONSTANTS.DEBUG*` —
  ~96 flag-guarded sites outside tests.

### 1.4 Where errors are "documented" today (bad)
- `print(f"! Constraint {constraint} is not valid.")` — `fst_manager.py:116`,
  console only.
- `print(error)` in `constraint_evaluation` — `fst_manager.py:658`, console only.
- `print(f"ERROR: {error} \n -> in Macro: …")` — `fst_keyboard.py:391,394`
  (config load), console only.
- `print(f"--- No Macro Sequence with name …")` — `fst_keyboard.py:1090`,
  console only.
- `check_result` (`fst_keyboard.py:908-914`) is `pass` with the
  `logger.error(fut.result())` commented out — **repeat-task exceptions are
  swallowed** (they surface, if at all, as GC-time "Exception ignored" warnings).
- GUI mode: only `surface_config_error` (`fst_keyboard.py:157-165`) reaches the
  user (error toast). Headless: console print.
- Nothing is reliably persisted: `fst.log` is wiped per run (§1.3).

---

## 2. DEBUG-flag usage map & classification

Convention per site: **what it prints / when → disposition**
(keep-as-log / convert-to-level / kill).
"flow" = the per-key pipeline logger (§4), "eval" = constraint-evaluation logger,
"state" = key-state/timing logger, "args/config" = startup & config logger.

### 2.1 DEBUG (D1) — 24 sites
| file:line | what / when | disposition |
|---|---|---|
| `free_snap_tap.py:110-112` | startup dump of `tap_groups_hr` / `_tap_groups` | convert → INFO once at startup |
| `fst_keyboard.py:191` | constraints parsed from a config key string (config load, per line) | convert → `fst.config` DEBUG |
| `fst_keyboard.py:538-539` | mouse-event debug — **already commented out** | kill |
| `fst_keyboard.py:706-708` | rebind trigger fired but missing in `_rebinds_dict` + dict dump (**error-class**) | convert → WARNING, always on |
| `fst_keyboard.py:766` | macro `key_sequence` at fire | convert → `fst.macro` DEBUG |
| `fst_keyboard.py:777` | `> playing makro: <trigger>` (typo "makro" in source) | convert → `fst.macro` DEBUG |
| `fst_keyboard.py:788` | `tap group #0` (per key entering tap eval) | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:794` | `tap group {vk}` (per matching tap group) | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:941` | macro alias still alive — trying to stop | convert → `fst.macro` DEBUG |
| `fst_keyboard.py:1091-1092` | no macro sequence with name + all names (config typo) | merge with always-on print `:1090` → WARNING |
| `fst_keyboard.py:1014` | numpad toggle write | remap → logger level toggle (§4) |
| `fst_keyboard.py:639` | `PRINT_VK_CODES` print (guarded by arg, not a flag — related) | keep as-is (it's a feature, not a debug flag) |
| `fst_manager.py:264-266` | `last()` constraint timing internals (per eval) | convert → `fst.state`/eval DEBUG |
| `fst_manager.py:281` | `dc()` double-click timing (per eval) | convert → `fst.state`/eval DEBUG |
| `fst_manager.py:735` | crossover taken (per tap-group key) | convert → `fst.flow` DEBUG |
| `fst_manager.py:742` | `delayed by N ms` (per tap key) | convert → `fst.flow` DEBUG (lag-relevant value) |
| `fst_manager.py:754-756` | `last_key_send` / `key_to_send` (per tap key) | convert → `fst.flow` DEBUG |
| `fst_manager.py:1287-1289` | flag snapshot copies | kill (single channel, §4) |
| `fst_manager.py:1360-1361` | per start-arg echo | convert → `fst.config` DEBUG |
| `fst_manager.py:1363-1365` | `-debug` start arg handling | replace → `-log=LEVEL` (§4) |
| `fst_manager.py:1374-1375` | `-file=` echo | convert → `fst.config` DEBUG/INFO |
| `fst_manager.py:1856-1857` | **functional**: if any DEBUG flag on, CLI menu is NOT cleared (`D1: cli not cleared`) | keep the *behavior*; gate = "any area logger at DEBUG" (§4) |
| `fst_manager.py:1892` | menu option 0 toggle | remap → flow-logger level toggle |

### 2.2 DEBUG2 (D2) — 20 sites
All key-state/timing diagnostics — no error-class sites.
| file:line | what / when | disposition |
|---|---|---|
| `fst_manager.py:182-195` | time-released/pressed lookup + **KeyError first-call** (`no value yet for vk_code`) (per eval) | convert → `fst.state` DEBUG — first-call paths are exactly the lag-diagnosis signal |
| `fst_manager.py:1777-1778` | `D2: releasing all keys` (focus change / pause / menu) | convert → `fst.state` DEBUG (lifecycle-ish → INFO borderline) |
| `fst_manager.py:1785-1786` | per-key simulated release (focus change) | convert → `fst.state` DEBUG |
| `fst_manager.py:1798-1799` | per-key modifier release | convert → `fst.state` DEBUG |
| `fst_manager.py:1730-1761` | commented-out old implementation block | kill |
| `fst_manager.py:1288` | flag snapshot copy | kill |
| `fst_keyboard.py:860-861` | simulated key may contradict real input (per sim key) | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:864-865` | not suppressed because toggle | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:869-870` | `D2 suppressed … because it would release real key press state` — **stale: the actual suppression is commented out at `:873`**; the print lies | convert → `fst.flow` DEBUG **with corrected message** ("suppression disabled" or delete) |
| `fst_keyboard.py:953` | macro playback start dump — reads `self._arg_manager.DEBUG2` (stale snapshot channel, §1.2) | convert → `fst.macro` DEBUG |
| `fst_keyboard.py:1016` | numpad toggle write | remap → root-debug level toggle |

### 2.3 DEBUG3 (D3) — 22 sites
Mostly eval-area; two are error-class and should become always-on WARNINGs.
| file:line | what / when | disposition |
|---|---|---|
| `fst_keyboard.py:397-398` | `display_internal_repr_groups()` after config load | convert → `fst.config` DEBUG (also reachable via menu/numpad 5 — keep that path) |
| `fst_keyboard.py:661-662` | repeated key suppressed (per repeated trigger key) | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:664-665` | repeated rebind key let through | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:1033-1034` | return to menu + real-key state dump | convert → INFO (lifecycle) with state at DEBUG |
| `fst_manager.py:108-109` | string constraint found (per string constraint eval) | convert → `fst.eval` DEBUG |
| `fst_manager.py:310-311` | `stop_repeat` KeyError — unknown repeat alias (**config/bug-class**) | convert → WARNING, always on |
| `fst_manager.py:338-339` | `is_repeat_active` KeyError | convert → `fst.eval` DEBUG (harmless query) |
| `fst_manager.py:348-349` | `reset_repeat` KeyError — unknown alias (**error-class**) | convert → WARNING, always on |
| `fst_manager.py:363-364` | `stop_all_repeat` AttributeError/ValueError | convert → WARNING, always on |
| `fst_manager.py:403-404, 412-413, 436-437, 444-445` | `variable … not set` (first-call init of eval variables — 4 sites) | convert → `fst.eval` DEBUG (first-call paths) |
| `fst_manager.py:422-423` | commented `inc()` | kill |
| `fst_manager.py:684-685` | **NameError: constraint not recognized — ignored** (config references unknown macro; silently continues) | convert → WARNING, always on — this is the most valuable "your config has a typo" diagnostic in the whole map |
| `fst_manager.py:1289` | flag snapshot copy | kill |
| `fst_keyboard.py:884-885` | removed key from pressed keys after alias fire | convert → `fst.flow` DEBUG |
| `fst_keyboard.py:1018` | numpad toggle write | remap → eval/root debug level toggle |

### 2.4 DEBUG4 — 24 sites (the keep)
The formatted key-flow print. Full successor design in §4.
| file:line | what | disposition |
|---|---|---|
| `fst_keyboard.py:632-634` | `D4: -> IN (t): ke - real/simulated` — pipeline **entry**, relative ms | → `fst.flow` DEBUG (successor record `IN`) |
| `fst_keyboard.py:727-728` | `D4: -- rebind found: old -> new` | → `fst.flow` DEBUG (`rebind`) |
| `fst_keyboard.py:742-743` | `D4: -- toggle arrived: cur -> toggle_ke` | → `fst.flow` DEBUG (`toggle`) |
| `fst_keyboard.py:747-748` | `D4: -- toggle suppress: ke` (key-up after toggle) | → `fst.flow` DEBUG (`toggle-suppress`) |
| `fst_keyboard.py:886-887` | `D4: XX SUPPRESSED: ke` (with `-- |` sim marker) | → `fst.flow` DEBUG (`SUPPRESSED`) |
| `fst_keyboard.py:905-906` | `D4: <- OUT (t): ke - real/simulated` — pipeline **exit**, relative ms | → `fst.flow` DEBUG (`OUT`) |
| `fst_keyboard.py:1087-1088` | `D4: -- Macro (alias) reseted by ke` | → `fst.macro` DEBUG |
| `fst_keyboard.py:1020` | numpad toggle write | remap → flow level toggle |
| `fst_manager.py:360-361` | `D4: -- Eval: stopped all Repeat` | → `fst.eval` DEBUG |
| `fst_manager.py:375-376` | `D4: -- Eval: released all keys` | → `fst.eval`/state DEBUG |
| `fst_manager.py:393-394` | `variable … set to …` | → `fst.eval` DEBUG |
| `fst_manager.py:450-451` | `variable … cleared` | → `fst.eval` DEBUG |
| `fst_manager.py:501-502, 509-510` | `D4: -- Eval: made/restored backup` (already also console-print + toast) | → INFO (once per operation, low volume — main log) |
| `fst_manager.py:637-638` | `D4: received for eval: constraint : ke` — eval **entry** | → `fst.eval` DEBUG |
| `fst_manager.py:688-689` | `D4: evaluated … to: result` — eval **exit** | → `fst.eval` DEBUG |
| `fst_manager.py:1699-1700` | `D4: -- toggle state for ke updated` | → `fst.flow` DEBUG |
| `fst_manager.py:1290` | flag snapshot copy | kill |
| `fst_manager.py:1892` | menu option 0 toggle | remap → flow level toggle |

### 2.5 DEBUG_NUMPAD — 6 sites (safety gate)
| file:line | what | disposition |
|---|---|---|
| `fst_keyboard.py:684-685` | gate: only run numpad check when enabled | **keep** — it exists so `alt+num*` doesn't get swallowed during gaming |
| `fst_keyboard.py:1014-1020` | `alt+num1..4` toggles `DEBUG..DEBUG4` | remap targets → logger level toggles (§4) |
| `fst_keyboard.py:1021-1027` | `alt+num5` internal repr, `num7/8` state-dict dumps | keep (they're state dumps, not flag gates) → log at DEBUG via the right logger |
| `free_snap_tap.py:40-41` | source toggle | replace by `-debug_numpad` start arg (already exists) |
| `fst_manager.py:1290` | snapshot copy | kill |
| `fst_manager.py:1366-1367` | `-debug_numpad` arg | keep as-is |

---

## 3. Logging scheme design

Stdlib `logging` only — no new dependency. One design principle: **high-volume
per-key streams go to their own file; everything else goes to one main log that is
always appended (never wiped).**

### 3.1 Levels (what goes where)
| level | content |
|---|---|
| `ERROR` | unexpected exceptions (use `logger.exception`), failed async dispatch, invalid runtime constraint (`fst_manager.py:116`), repeat-task crashes (re-enable `check_result`, §5) |
| `WARNING` | config-suspect, fail-closed paths that today print-and-continue: unknown key in state constraint (`:653-660`), **constraint NameError ignored** (`:684`), rebind not found (`fst_keyboard.py:706`), no macro sequence name (`:1090`), repeat alias not found (`:310,348,363`), unknown start argument / out-of-range values (`:1461` etc.) — **always on, no flag** |
| `INFO` | lifecycle & user-visible ops: start/mode, focus change, pause/resume, menu enter/exit, control combos, backup/restore, start-arg application summary, file reload |
| `DEBUG` | everything per-key/per-eval from §2 (flow, state, eval, macro, config) |

### 3.2 Per-area loggers
```
fst                 (app root — lifecycle, WARNING+ errors; handlers: fst.log + console)
├── fst.flow        per-key pipeline: IN/rebind/toggle/SUPPRESSED/OUT + D1/D2 per-key lines
├── fst.eval        constraint evaluation + eval variables (D3/D4 eval-area lines)
├── fst.state       key-state dicts, press/release tracking, timings (D2 lines)
├── fst.macro       macro/repeat task lifecycle (playback, repeat, interrupt)
└── fst.config      start-arg parsing, config load, focus groups
```
- Sub-logger records propagate to `fst` (default) → they land in `fst.log` at
  their level.
- `fst.flow` gets **its own file handler** and is excluded from the main file at
  DEBUG volume (it is the only high-volume stream: 1–6 lines per key event).
  Implementation: `fst.flow` handler = own `FileHandler(fst_flow.log)`; set
  `fst.flow.propagate = False` when the flow file is active (or keep propagate
  and accept the duplication at INFO-only volume — the clean version is the
  propagate-off variant; the worker decides).

### 3.3 Files & console
| file | level | mode | content |
|---|---|---|---|
| `fst.log` | `INFO` default (all areas) | **`filemode='a'`** (fixes the per-run wipe) | everything WARNING+ always; DEBUG when enabled |
| `fst_flow.log` | `DEBUG` when enabled, else closed | `filemode='a'` | the key-flow stream only (DEBUG4 successor) |
| console | `WARNING` (headless) | stream | visible errors still show (matches today's print behavior) |
| toast | errors only | via existing `surface_config_error` | GUI mode unchanged |

Log location: keep CWD-relative `fst.log`/`fst_flow.log` for now (matches his
current muscle memory); an explicit `-logdir=` start arg is a cheap future
extension, not needed for the first build.

### 3.4 Start-argument control (extends his existing pattern in `apply_start_arguments`)
| arg | effect | replaces |
|---|---|---|
| `-log=LEVEL` (`debug`/`info`/`warning`/`error`, default `info`) | sets `fst` + all area loggers | `-debug` / source-edit toggling of D1-D3 |
| `-flow` | enables `fst.flow` at DEBUG → `fst_flow.log` | `CONSTANTS.DEBUG4 = True` source edit, menu option 0 |
| `-debug_numpad` | unchanged (safety gate for numpad toggles) | — |
| unknown arg → WARNING (already printed today, `:1461`) | | |

Properties worth noting:
- `apply_start_arguments` is re-run on **every focus change / pause-resume**
  (`apply_start_args_by_focus_name`, `fst_keyboard.py:1095,1104` — same channel as
  the other args) → a `-log=`/`-flow` value is re-applied consistently. This
  matches how `-delay` etc. already behave.
- **Runtime toggles survive focus changes** because logger levels are set on the
  loggers themselves, not re-derived from arg snapshots:
  - `alt+num1` → toggle `fst` area loggers between DEBUG/INFO (all of D1-D3 in one)
  - `alt+num4` → toggle `fst.flow` between DEBUG and closed (the old DEBUG4 toggle)
  - menu option 0 → same as `alt+num4` (its text says exactly "Toggle debugging
    output with evaluation results")
  - `alt+num2`/`num3` → collapse onto `alt+num1` (D2/D3 no longer need a separate
    channel — one root debug level covers them)
- The functional CLI-keep-open behavior (`fst_manager.py:1856`: any debug flag →
  don't `cls`) becomes: any area logger at DEBUG → don't `cls`.

### 3.5 How lag diagnosis benefits
1. **Timed segments** — the flow records already carry relative ms
   (`key_event_time - FST_Keyboard.START_TIME`, `fst_keyboard.py:634,906`); the
   successor makes `IN (t) … OUT (t)` a mandatory pair → per-key pipeline latency
   is `OUT.t − IN.t`, readable directly in `fst_flow.log`.
2. **Delay values logged** — the chosen random delay per tap key
   (`fst_manager.py:742`) and macro `execute_key_event` sleeps go to the flow/macro
   loggers, so a slow macro shows its delay budget next to its playback lines.
3. **First-call paths** — the state KeyErrors (`no value yet for vk_code`,
   `fst_manager.py:185,194`) and eval-variable first-inits (`variable … not set`)
   stay as DEBUG records: first-press-of-a-key / first-use-of-a-variable is
   exactly where one-shot costs (dict init, time-diff bootstrap
   `fst_keyboard.py:626-630`) hide.
4. **Async dispatch latency** — mouse rebinds and macro playback are dispatched
   via `asyncio.run_coroutine_threadsafe` (`fst_keyboard.py:815,920`); a DEBUG
   record at dispatch + the done-callback result bounds the thread-hop cost.
5. **Errors documented across runs** — `filemode='a'` + WARNING always-on means a
   crash in run N is still on disk when run N+1 starts; repeat-task crashes stop
   being swallowed (§5).

---

## 4. DEBUG4 successor design (the formatted key-flow log)

**Keep it** — it is the maintainer's workhorse for watching
pressed → replaced → macro output. Convert it from `print` to structured log
records on the `fst.flow` logger, one file: `fst_flow.log`.

**Record vocabulary** (same arrow language, one line per step — his reading
habit is preserved):
```
--> IN   (t_rel_ms)  real|sim   <ke repr>          pipeline entry
--  rebind  <old ke> -> <new ke>                  rebind step
--  toggle  <ke> -> <toggle_ke>                   toggle arrival
--  toggle-suppress <ke>                          toggle key-up suppressed
--  XX SUPPRESSED <ke>                            final suppression
--> OUT  (t_rel_ms)  real|sim   <ke repr>          pipeline exit
```
- `t_rel_ms` = `key_event_time − START_TIME` (already computed,
  `fst_keyboard.py:634,906`).
- Simulated keys keep the `-- |` marker as a **field**, e.g. `IN -- | (123) sim
  ke(0x41, press)` — the visual distinction he uses stays, but it is now a
  greppable token (`grep 'sim ' fst_flow.log`).
- Format string: `%(asctime)s %(levelname)s %(message)s` — absolute asctime for
  cross-run correlation, relative ms in the message for within-run reading.
- **Level: DEBUG on `fst.flow`** (enabled by `-flow` / `alt+num4` / menu 0).
- Eval-area D4 lines (`received for eval`, `evaluated … to:`, variable set/
  cleared, `-- Eval:` actions) go to `fst.eval` instead, so `fst_flow.log`
  stays a pure key stream.
- The IN/OUT pair gives per-key latency for free (§3.5.1) — the one structural
  addition over today's D4.
- Volume estimate: a 10-key/s tap session ≈ 30–60 lines/s ≈ a few MB/hour —
  acceptable in its own file, unacceptable in the main log (hence separate).

**What dies from the D4 stream:** nothing the maintainer reads. What changes:
the two console-only error prints inside the eval block (`fst_manager.py:116,658`)
become always-on ERROR/WARNING records in `fst.log`.

---

## 5. Error-documentation fixes that ride along (small, local)
1. `check_result` (`fst_keyboard.py:908-914`): restore the commented
   `fut.result()` logging — one `try/except` + `logger.error`; today repeat-task
   crashes are invisible.
2. `fst_manager.py:116` (`! Constraint … not valid`) and `:658` (`print(error)`):
   → `logger.warning` / `logger.error` (always on).
3. `fst_keyboard.py:869-870` stale "suppressed" print (suppression commented out
   at `:873`): fix the message or delete — a lying DEBUG line is worse than none.
4. `fst_keyboard.py:391,394` config-load `ERROR:` prints: → `logger.error`
   (they already raise; the log just needs to persist them).
5. Dead nested `clear_all_variables` (`fst_manager.py:454-459`, self-recursive,
   shadowed by the real method at `:781`): delete — kill.

---

## 6. Per-DEBUG-flag disposition (the summary table)
| flag | sites | disposition | reasoning |
|---|---|---|---|
| `DEBUG` | 24 | **kill flag; convert sites** | mixed bag → per-key lines → `fst.flow` DEBUG; the 2 error-class sites (rebind-not-found, missing macro name) → always-on WARNING; lifecycle → INFO; start-arg echo → `fst.config` DEBUG. The `-debug` arg becomes `-log=debug`. |
| `DEBUG2` | 20 | **kill flag; convert sites** | 100 % key-state/timing diagnostics, no errors → `fst.state`/`fst.flow` DEBUG. First-call KeyErrors are valuable → keep as DEBUG records, not deleted. One stale print fixed (§5.3). |
| `DEBUG3` | 22 | **kill flag; split sites** | eval-area lines → `fst.eval` DEBUG; the error-class ones (NameError constraint ignored, repeat alias not found ×3) → **always-on WARNING** — these document config typos that fail silently today. |
| `DEBUG4` | 24 | **keep, convert** → `fst.flow` logger (§4) | the maintainer's workhorse; becomes a structured, file-backed, level-controlled version with the same arrow vocabulary plus IN/OUT latency pairs. |
| `DEBUG_NUMPAD` | 6 | **keep flag; remap targets** | it's a safety gate for numpad capture, not a debug level — its purpose survives. `alt+num1/4` become logger-level toggles; `num2/3` collapse onto `num1`. |

Net: 5 flags → 2 controls (`-log=LEVEL` + `-flow`) + the numpad safety gate;
~96 flag-guarded print sites → ~80 logger calls (kills: 10 commented/dead sites;
~5 promoted to always-on WARNING/ERROR); hot-path cost unchanged (one
`isEnabledFor(DEBUG)` check vs one boolean read — same order).

---

## 7. Bookkeeping discrepancies found (for planner curation)
1. Spec §"Verified facts" counts "4 `logging.{info,warning,error,debug}` calls
   outside tests" — actual live count is **~14** (`free_snap_tap.py` 5,
   `fst_keyboard.py` 6 live + 1 commented, `fst_manager.py` 2, `fst_tasks.py` 1).
   Minor undercount; the "embryonic" characterization still holds.
2. `fst_manager.py` is 1943 lines (spec said ~1900 — fine, not a discrepancy).
3. Spec says the maintainer toggles flags "at `free_snap_tap.py:32-41`" — actually
   `30-41` (DEBUG is at 30-31); trivial.
4. Found, not in spec: the two parallel flag channels (§1.2) make runtime
   numpad toggles ineffective for `self.DEBUG`/`self._arg_manager.DEBUG2` reads.
5. Found: `fst.log` `filemode='w'` wipes per-run evidence (§1.3).
6. Found: `check_result` `pass` swallows repeat-task exceptions (§5.1).
7. Found: stale "suppressed" print where suppression is disabled
   (`fst_keyboard.py:869-873`).
8. Cosmetic: `print("D1: > playing makro:", …)` typo `fst_keyboard.py:777`.

---

## 8. Recommendation: structured logging over the DEBUG constants

**Use structured logging; retire the five flags.** For this maintainer's actual
workflow (toggle something on while gaming, read a formatted stream later,
figure out why a macro felt laggy or a config silently misbehaves), logging
wins on every axis that matters:

1. **Same mental model, one mechanism.** A log *level* is his flag, generalized:
   `alt+num4` keeps toggling the flow stream at runtime, `-flow` turns it on at
   start, and one root level covers D1-D3 instead of three booleans in two
   channels. The three-surface toggle web (source edit / start arg / numpad /
   menu) collapses to (start arg / numpad / menu) — strictly fewer moving
   parts, and the latent two-channel bug (§1.2) disappears because there is one
   channel.
2. **Errors survive.** Today the single biggest loss is invisible: `fst.log` is
   overwritten on every launch, and repeat-task crashes go to `pass`. Append
   mode + always-on WARNING/ERROR fixes both without any workflow change —
   he just reads the file after a bad session instead of hoping the prints
   were still on screen.
3. **Lag diagnosis gets a structure it can't have with print.** Per-key
   IN/OUT latency pairs, delay values, first-call markers, and area-attributed
   records (`fst.flow` vs `fst.eval` vs `fst.state`) mean "where did the 40 ms
   go" is a grep, not a scroll through interleaved console text.
4. **No overwhelming, by construction.** The high-volume stream lives in its own
   file that only opens when `-flow` is set; the main log stays at INFO. With
   print+flags, "not overwhelming" meant "flag off, see nothing" — logging
   gives him *both* (quiet main log **and** available deep trace) in one run.
5. **Cost is honest and low.** Hot-path overhead is one `isEnabledFor` check —
   the same as today's boolean read. The one real cost is the one-time
   mechanical conversion of ~96 sites; that is one FST worker unit, not a
   redesign, and every site maps 1:1 from the table in §2/§6.

The DEBUG constants' only advantage is familiarity, and the design above keeps
the part of that he actually values: the D4 arrow vocabulary and the numpad
toggles. **Recommendation: build the §3/§4 design; the DEBUG flags die in the
same unit.**
