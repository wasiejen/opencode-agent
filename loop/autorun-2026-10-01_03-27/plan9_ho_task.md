# Task spec — U0: keep-N surface (`keepMessages`) for context_trim tail

Design source (read first): `proposals/approved/2026-10-01_context-budget-package.md` §U0.

## Goal
A `keepMessages` argument on `context_trim` **tail** mode (mutually exclusive
with `target`), delegating to the existing `tailSetKeep` core export — zero
new validation logic, zero new rejections. The keep form is the post-
compaction shrink lever (the emergency-keep=18 fix case); growing stays
available (no extra gating).

## Verified facts (measured 2026-10-01 by the planner)
- `.opencode/tools/context_trim.ts` is a SINGLE file (no core split).
- `tailSetKeep` export at L686: signature `(dbPath: string, sessionID:
  string, keep: number): Promise<string>`; rejections already exist:
  keep-invalid, keep-exceeds-history, floor 6, no-completed-compaction,
  session-not-found.
- Mode dispatch at L752-753: `report` → `reportWindow`; `tail` → the
  messageID form today. The keep-COUNT boundary form (L671 comment) already
  exists in the core — only the tool surface is missing.
- Smoke: `.opencode/plugin/tests/context_trim.smoke.mjs` = 29/29 (measured
  2026-10-01); it uses fixture DBs only (live opencode.db NEVER written —
  follow the existing fixture pattern).
- Probe: the S33 section covers context_trim (plan39 landed checks 346-351
  when the probe was 352; the probe is now 355 — verify the current S33
  range against the probe header tally before touching it).

## DoD (measured end states)
- Tool surface: `tail <session> keep=N` (keep form) → the `tail=` return
  line identical in shape to the messageID form; the five rejections →
  their plain rejection lines, no DB write.
- Tool DESCRIPTION updated (MODES line + the tail-mode parameter note:
  keep form, mutually exclusive with target, count semantics = compact_
  memory keepMessages — NOT a token budget).
- Smoke: new keep-form pins THROUGH THE TOOL SURFACE (valid + each
  rejection); existing 29 pins stay green (count re-verified at run).
- Standard gate green: probe 355/355 + all 11 smokes.
- Checkpoint: one verified unit → ONE code commit; TODO + handover ride
  the final commit (hashes recorded by the planner, not the worker).

## DO-NOT-touch
- the live opencode.db (fixture DBs only); the fork
  (`C:\Users\Wasiejen\Repos\opencode`); `maintainer/`; other tools/plugins;
  the proposal files; `agent/prompts/**` (U4's domain).
- New TODO entries: file ONLY a genuine discrepancy found (expectation,
  never a promise).

## Worker
worker_Q3S (small surface wiring + pins + docs).
