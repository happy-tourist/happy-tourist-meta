## Context

See `proposal.md` — Why / Scope. Explore closed: одиночная (`maxSeats === 1`) ≠ соло в конце (`maxSeats ≥ 2`, один eligible); countdown 5…1 оставить; end-turn и модал на входе не нужны; +1/+1 когда нет доступных действий (ход, просмотр, rescue/return/push).

Current bug: `maybeStartCountdown` returns early when `seated < 2`, so a full table of one never enters countdown. Current solo path: `eligible === 1` → `infinite: true` + 60s deadline — would wrongly apply to intentional single-player after start is fixed.

Packages: **server** + **client** (room `tourist`).

## Goals / Non-Goals

**Goals:**

- Start: full table includes `maxSeats === 1`.
- Gate solo leftover (∞ + 60s) on `maxSeats ≥ 2`; single-player stays finite, no deadline.
- Reuse `seatHasAvailableActions` for single-player +1/+1 refresh (same predicate as multi auto end-turn).
- Client: numeric HUD, no ring/modal/end-turn for `maxSeats === 1`; +N animation on refresh.

**Non-Goals:**

- Changing solo leftover product rules.
- New schema fields / messages.
- Lobby create UI for choosing maxSeats (already allows 1).

## Decisions

### D1 — Distinguish modes by `maxSeats`, not only eligible count

- **Choice:** Solo leftover = `maxSeats ≥ 2 && countEligible() === 1`. Single-player = `maxSeats === 1`. Wire already syncs `maxSeats`.
- **Why:** Same eligible count, different product; create-time capacity is the intent signal.
- **Alt:** Separate `mode: 'solo'|'single'` create option — rejected (YAGNI; maxSeats=1 already exists).

### D2 — Fix start gate without weakening ready rules

- **Choice:** In `maybeStartCountdown`, allow start when `seated >= maxSeats` even if `seated === 1`; keep `seated < 2` only for the all-ready underfill path (ready still requires ≥2).
- **Why:** Matches «full table» requirement; ready semantics unchanged.
- **Alt:** Special-case `if (maxSeats === 1)` only — equivalent; prefer `full = seated >= maxSeats` without a blanket `seated < 2` return before the full check.

### D3 — Single-player budgets: finite grant + refresh, not ∞

- **Choice:** Prefer helpers `isSinglePlayerRoom()` (`maxSeats === 1`) and `isSoloLeftover()` (`maxSeats ≥ 2 && countEligible() === 1`). `syncSoloInfiniteMode` sets `infinite` only when `isSoloLeftover()`. `applyTurnGrant` for `isSinglePlayerRoom()` always does finite `steps += 1; peeks += 1` (today’s `eligible >= 2` finite branch must also cover maxSeats=1). New branch beside / inside `maybeAutoEndTurn`: when single-player and `!seatHasAvailableActions(current)`, add +1/+1 and `sendBudgets` (do not `advanceTurn`). **Deferral:** while `openPeek` or `trapPipelineActive`, return without granting — **do not** set `pendingTurnAdvance` (today’s idle path clears pending and returns when `countEligible() < 2`, which would silently drop a deferred refresh). Resume after peek close / `onTrapPipelineIdle` F1 re-eval / all-jail→idle / other existing `maybeAutoEndTurn` call sites by running the same refresh branch. **Never** `advanceTurn` / multi `pendingTurnAdvance` turn pass (there is no next seat).
- **Why:** Same «нет действий» predicate as multi; no new action vocabulary; reusing `pendingTurnAdvance` would conflict with idle absorb when eligible is below 2.
- **Alt:** Manual end-turn in single-player — rejected (product). Alt: grant only when both counters are 0 — rejected (can soft-lock with unused peeks and nowhere to peek). Alt: separate `pendingSinglePlayerRefresh` flag — unnecessary if deferral is early-return and idle already calls `maybeAutoEndTurn()`.

### D4 — No turn deadline in single-player

- **Choice:** `resyncTurnDeadline` / `startTurnDeadline`: if `maxSeats === 1`, `clearTurnDeadline()` and return. `onTurnTimeout` MUST NOT mark `timeExpired` when `maxSeats === 1` (defense in depth if a deadline were ever set). Never mark `timeExpired` from wall-clock in this mode. Delta MUST MODIFY main «Authoritative turn deadline while playing» (keep SC-MOVE-25/26/27) so sync does not re-assert a deadline for maxSeats=1; SC-MOVE-101 covers outcome + timeout gate.
- **Why:** Product: no timer; avoids post-sync contradiction with the general deadline SHALL.
- **Alt:** Keep a long soft timer — rejected.

### D5 — Client gates off synced `maxSeats`

- **Choice:** Treat single-player as `state.maxSeats === 1`. Do not open `soloUnlimitedModal` on `budgetsInfinite` when `maxSeats === 1` (should not become infinite; gate the watch even as defense-in-depth). Red ring / `isSoloBudget` / `isSoloBudgetFromSeats`: require solo leftover (`maxSeats ≥ 2` && one eligible), not merely one eligible — pass synced `maxSeats` into the helper (eligible-only is wrong for intentional single-player). Turn countdown chrome: `showTurnRing` (and equivalent) MUST be **false** when `maxSeats === 1` — do not rely only on empty `turnUntil` if a stale deadline were synced. End-turn: leftover already hides via `budgetsInfinite`; **single-player is finite**, so `canSendEndTurn` / end-turn chrome MUST also reject `maxSeats === 1` (do not rely on «sole eligible» alone — that path is not how the client currently gates). Numeric counters when `!budgetsInfinite`; +N fall on finite budget increases (covers refresh). Existing vitest leftover fixtures that seed `maxSeats: 1` with ∞ / red-ring expectations MUST retarget to `maxSeats ≥ 2`.
- **Why:** Spectators see `maxSeats` and seats; private budgets alone cannot distinguish modes for ring accent; finite single-player would otherwise show end-turn; ring/modal gates mirror the same maxSeats signal as budgets.
- **Alt:** Infer from absence of `turnUntil` only — weaker for transient sync; still pair with maxSeats.

## Risks / Trade-offs

- **[Risk] Forgetting `maxSeats ≥ 2` on any solo leftover path leaves single-player in ∞** → Mitigation: centralize `isSoloLeftover()` / `isSinglePlayerRoom()` helpers; mocha SC-MOVE-100 + SC-MOVE-106.
- **[Risk] Auto-+1/+1 loops every tick when still no actions after grant** → Mitigation: grant at most **once per evaluation call**; do not re-grant in the same pass if still `!seatHasAvailableActions` after +1/+1. Next evaluation only after another game event (peek close, `onTrapPipelineIdle` F1, all-jail reset inside the trap pipeline then idle, accepted/rejected action paths that already call the multi auto-end sites) — mirror `maybeAutoEndTurn` call sites; **do not** park work on `pendingTurnAdvance`. Soft-lock until such an event (e.g. all pieces trapped, no free rescuer yet) is accepted; all-jail reset remains the existing escape and MUST re-run the single-player refresh evaluation afterward (via idle → refresh branch).
- **[Risk] Trap deferral via `pendingTurnAdvance` drops refresh** → Mitigation: D3 — early-return deferral only; never set the multi pending flag for maxSeats=1.
- **[Risk] Countdown 5s feels odd alone** → Mitigation: accepted (explicit product choice).

## Migration Plan

1. Server: start gate → solo/single split → deadline skip → auto refresh → mocha.
2. Client: maxSeats gates for modal/ring/HUD → vitest.
3. No DB migration; rollback = revert siblings.

## Technical prerequisites

None open (no new deps / providers).

Implementation checklist: `tasks.md`.
