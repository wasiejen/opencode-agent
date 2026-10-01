# looprun stats — autorun-2026-10-01_03-27
# generated 2026-10-01T06:56:20.900Z by agent/scripts/db/loop_stats.cjs (read-only)

## per session
| session | role | model | tool calls | parts | wall (min) | ctx first -> last | compactions |
|---|---|---|---|---|---|---|---|
| ses_f0af35202ffeX6nthxp8kWxtPv | planner-1 | Qwen3.8-27B-Q3S-235K-slow-HQKV | 67 | 179 | 32.2 | 87%/29K -> 87%/29K | 0 |
| ses_f0ad5d279ffe5cso7Q0CL60RD2 | planner_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 68 | 242 | 70.5 | 63%/85K -> 63%/85K | 0 |
| ses_f0ad0bbfcffeCKb4FmU2gK7c29 | worker_Q3S | Qwen3.8-27B-Q3S-170K | 43 | 183 | 11.4 | CTX=100101 (58%) REM=69899 -> CTX=100101 (58%) REM=69899 | 0 |
| ses_f0abc032dffenb1K8yrfXKOg5V | worker_Q3S | Qwen3.8-27B-Q3S-170K | 98 | 399 | 31.5 | CTX=158746 (93%) REM=11254 -> CTX=158746 (93%) REM=11254 | 0 |
| ses_f0a9543a0ffeOmHASvp72Lj7JI | planner_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 49 | 170 | 23.2 | n/a -> n/a | 0 |
| ses_f0a8d978effem4PJrbIaas9Zfb | worker_Q3S | Qwen3.8-27B-Q3S-170K | 37 | 148 | 3.7 | n/a -> n/a | 0 |
| ses_f0a800624ffe8CVXBLVwWX7y4N | planner-4 | Qwen3.8-27B-Q3S-235K-slow-HQKV | 60 | 195 | 56.1 | 53%/110K -> 53%/110K | 0 |
| ses_f0a79494cffeSBGQwPzwXhXKBj | worker-4 | Qwen3.8-27B-Q3S-170K | 63 | 265 | 25.0 | CTX=101975 (59%) REM=68025 -> CTX=101975 (59%) REM=68025 | 0 |
| ses_f0a5c156effeaF2xWmLo9UoZdk | explorer-4 | Qwen3.8-27B-Q3S-170K | 39 | 148 | 10.6 | CTX=81059 (47%) REM=88941 -> CTX=81059 (47%) REM=88941 | 0 |
| ses_f0a4c9adfffeiYJNFDlqcnbCzX | planner-5 | Qwen3.8-27B-Q3S-235K-slow-HQKV | 77 | 226 | 96.2 | 61%/89K -> 61%/89K | 1 |
| ses_f0a1d6a5affeFgN9YsdlKV7LNr | worker_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 84 | 322 | 33.5 | CTX=149012 (63%) REM=85988 -> CTX=149012 (63%) REM=85988 | 0 |
| ses_f09f48614ffeJs1nvJnOonTFxO | planner_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 52 | 178 | 43.5 | n/a -> n/a | 0 |
| ses_f09e89374ffeFc47RHh5PKNCot | worker | Qwen3.8-27B-Q3S-235K-slow-HQKV | 57 | 213 | 11.1 | 30%/164K -> 30%/164K | 0 |
| ses_f09d877e4ffe7sI5ZnOHKWKBQ6 | explorer_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 26 | 120 | 8.4 | n/a -> n/a | 0 |
| ses_f09cca546ffee8SOxAWdjFOtmC | planner_Q3S_slow | Qwen3.8-27B-Q3S-235K-slow-HQKV | 37 | 106 | 13.7 | n/a -> n/a | 0 |

## looprun
- iterations: 7 (distinct planner sessions)
- max planner-N token: 5
- worker launches: 8
- log lines: START 14 / DONE 14 / RETURN 2 / WARNING 0 / INFO 2 / CORRECT 0
- sessions in DB: 15 of 15
- compactions (ctx.log, scoped to run sessions): 1
