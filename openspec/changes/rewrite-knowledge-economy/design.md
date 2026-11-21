## Context

See `proposal.md` — Why / Scope. Explore closed D1–D17 (knowledge economy) plus polish + playtest follow-up: 60s timer, 15% grille / 20% catapult, all-jail wait/skip + room toast, strip/avatar Material `lightbulb`, Notify, cut copy «которые успели», finish-travel re-fix.

Packages: **server** + **client** (room `tourist`).

## Goals / Non-Goals

**Goals:**

- Authoritative one-action turn + open-on-land / fling chain; **60s** turn deadline with ring; timeout force-closes open Q&A as «Не успел!» + top toast.
- Private `knowledge` `{ delivered, pieces, outcome? }` + public `Piece.remainingTurns`; strip-only carried TR + **visible** lightbulb; avatar delivered + icon + +N.
- Fixed exclusive trap seed **15% grille / 20% catapult** (clamp grille then catapult); remove density create options and push.
- All-jail: stay trapped until remainingTurns expire — no start teleport / no start-warning modal; **skip turn whenever no free unfinished piece remains** if another eligible seat exists; **room-wide Notify** toast; **no** all-jail toast when the game is over (knowledge modal only).
- Penultimate cut; knowledge modal with **distinct** finish vs cut copy (`которые успели`); reliable finish travel anim (click + catapult) — no snap-vanish.

**Non-Goals:**

- Public cross-seat scoreboard / ranking UI.
- New npm deps / enabling `mdi-v7` (use existing `material-icons`).
- Map/pack CMS changes.
- Solo leftover ∞ / permanent `timeExpired` from a leftover-only clock (turn timeout advances / closes modal only).

## Decisions

### D1 — Fixed trap ratios, exclusive partition

- **Choice:** At `enterPlaying` / `seedPlayingEconomy`, partition `*` cells with no overlap (pick grille keys first, then catapult from the remainder — never both on one cell; Q&A never under traps). Let `n` = task-cell count. `grilleCount = min(round(n * 0.15), n)`; `catapultCount = min(round(n * 0.20), n - grilleCount)` (**15% grille / 20% catapult**). Remaining cells = Q&A tasks. Drop create options `grilleDensity` / `catapultDensity` and lobby UI; if a client still sends density options, server **ignores** them.
- **Why:** Playtest — more catapult pressure; clamp order keeps `g + c ≤ n`.
- **Alt:** 15%/15% — superseded. 12%/15% — rejected earlier. Density UI — rejected.

### D2 — Replace budgets with knowledge fields

- **Choice:** Hard-cut private `budgets`. Own-HUD only: room-private Map + private message **`knowledge`** `{ delivered: number, pieces: Record<pieceId, number>, outcome?: 'finish' | 'cut' }` via `client.send`. Correct solve → piece carried `+= difficulty`. Finish piece → `delivered +=` carried; carried `= 0`. On full finish set `outcome: 'finish'`; on penultimate cut set `outcome: 'cut'` (and re-send). Do **not** put carried/delivered on public schema.
- **Why:** Explore D2 + dual modal copy needs a finish|cut discriminator without place UX.
- **Alt:** Heuristic client-side cut detection — rejected (fragile). Public schema counters — rejected.

### D3 — Turn machine + 60s deadline

- **Choice:** Keep end-turn retired and push rejected. Restore synchronized turn deadline: `turnBudgetSeconds = 60` when ≥1 other eligible seat exists (same family as pre-rewrite multi turn clock); set `turnUntil` when a seat becomes current (including after modal-free land advance). **Do not** mark permanent solo-leftover `timeExpired` from this clock. Track room-private `boardActionUsed` (reset on become-current). Rescue while `!boardActionUsed`. On accepted move/return set flag; idle pipeline → open-on-land Q&A (hold turn) or `advanceTurn`. While a shared task modal is open, the turn deadline **keeps ticking**; on timeout the server MUST force-close the session with `peekClose` incorrect + `reason: 'timeout'` (KEEP bind / open digit), then `advanceTurn`. Client local board-action spent mirror unchanged. Presence paints the **turn countdown ring** again (reconnect grace may dual-ring when offline current).
- **Why:** Product polish — «время на ход 1 минуту»; timeout «как было» closes modal + toast.
- **Alt:** Pause timer while modal open — rejected. Timer display-only — rejected.

### D4 — Open-on-land unifies peek entry

- **Choice:** Unchanged from rewrite: land/fling → auto open shared peek; client `peek` reject; wrong KEEP + advance; correct knowledge + empty cell + advance; reopen only by later land.
- **Why:** Explore items 1–2.
- **Alt:** Keep client `peek` — rejected.

### D5 — Grille remainingTurns tick on become-current

- **Choice:** Public `Piece.remainingTurns` (3…1). Tick on become-current; at 1 → clear grille empty + free. UI red TR badge on **board + strip**, painted **above** grille overlay (`z-index` higher than `.grille-overlay` / chrome grille).
- **Why:** Explore D9/D10 + polish visibility under grille art.
- **Alt:** Tick on end of turn — rejected.

### D6 — Trap clear / catapult consume → empty cell

- **Choice:** Unchanged — exclusive traps never hold Q&A; clear/consume → `removedTaskKeys` empty cell. (**Note:** all-jail no longer clears grilles via teleport path — see D7.)
- **Why:** Explore D14.
- **Alt:** Reshuffle task under cell — rejected.

### D7 — All-jail skip whenever nobody can move (+ room toast)

- **Choice:** When a seat’s unfinished pieces are all trapped, the server MUST **NOT** clear grilles, MUST **NOT** relocate pieces to start, and MUST **NOT** show the retired start-reset modal. Pieces stay trapped with `remainingTurns`.
  - **Skip:** Whenever that seat cannot act (all unfinished trapped — after trapping land settle **and** after become-current grille tick if still all trapped) **and** `countEligible() ≥ 2` (another eligible seat exists), the server MUST `advanceTurn` and broadcast a room message (reuse/`allJailSkip` or revived semantic of room notify — **not** the old start-reset modal payload) so **every** connected client shows a top-center Notify toast: «Все туристы в решётке — ход пропущен».
  - **No toast / no fake skip loop:** If `countEligible() < 2` (solo leftover waiting on ticks, or party ended / only finished seats remain), MUST **NOT** fire the all-jail skip toast. End-of-game is the knowledge modal (`finish`/`cut`) only.
  - After tick frees ≥1 piece on become-current, normal turn continues (no skip toast).
- **Why:** Playtest — skip every empty turn; toast for table awareness; end-game stays modal-only.
- **Alt:** Toast only on trapping land — rejected. Toast when game over — rejected. Enable mdi for icons — rejected (see D9).

### D8 — Finish: dual-copy knowledge modal + penultimate cut

- **Choice:** `finishPlace = 1` gate for full finish and cut (no place UX / no `nextFinishPlace` bump). Client opens **one** modal chrome on local `finishPlace` 0→1. Body uses private `knowledge.delivered` **and** `outcome`:
  - `finish`: «Все ваши туристы дошли до финиша и доставили {n} знаний»;
  - `cut`: «Соперники финишировали. Ваши туристы, **которые** успели, доставили {n} знаний» (not «что успели»).
  Penultimate cut (`maxSeats ≥ 2`): burn carried→0 without adding to delivered, unfinished pieces `finished=true`, re-send `knowledge` with `outcome: 'cut'`. `maxSeats === 1` skips cut.
- **Why:** Explore + playtest copy fix.
- **Alt:** Same single sentence for both — rejected. «что успели» — rejected.

### D9 — Client HUD: strip knowledge, focus@3, toast, timer ring, Material lightbulb

- **Choice:**
  - Avatar: delivered counter + +N on delivered growth; **`lightbulb`** Quasar icon name from **material-icons** (already in `quasar.config` extras — do **not** use `mdi-lightbulb-on` without enabling `mdi-v7`).
  - Carried: **only** personal strip slots, yellow badge **top-right** + same `lightbulb` icon; **remove** board-piece yellow BR badge; badge layout MUST NOT leave empty icon-sized gap when icon is present.
  - Grille: red TR board+strip; z-index above grille overlays.
  - Focus @3 o’clock 36×36; no end-turn; no place badges; no solo leftover/unlimited modals.
  - Outcome flashes: Quasar **Notify** top-center for correct / wrong / timeout; all-jail skip toast per D7. Enable `Notify` in `quasar.config.ts` plugins.
  - Lobby: no density UI/payload.
- **Why:** Playtest — mdi names rendered blank; strip looked padded.
- **Alt:** Enable `mdi-v7` — rejected (extra font). Keep board BR knowledge — rejected.

### D10 — Finish travel animation reliability (re-fix)

- **Choice:** Click-to-center and catapult-fling-to-center MUST always run `FINISH_TRAVEL_MS` (~550) + fade (~300): seed/preserve `lastKnown` / `finishAnimFrom` / `boardActionPreCell` / `catapultDeferFinishKeys` so the piece paints one frame at pre-finish cell, then slides to center, then fades. **Must not** snap-vanish even for the first finisher. Prefer a behavioral/vitest path that fails if travel `from` is missing when finishing from a non-center cell (not only SFC source asserts). Keep server finish marking; client presentation unless a sync race is found.
- **Why:** Playtest after block 8 — first tourist still vanishes.
- **Alt:** Accept instant vanish — rejected. Source-only SC-FINISH-35 — insufficient.

## Risks / Trade-offs

- **[Risk] Timer + open modal stalls less but pressures answers** → Mitigation: accepted; timeout = «Не успел!» KEEP + advance.
- **[Risk] All-jail skip+toast spam if become-current loops on solo** → Mitigation: D7 gates toast/skip on `countEligible() ≥ 2`; solo waits on ticks without toast.
- **[Risk] Notify plugin change** → Mitigation: Quasar built-in; already enabled.
- **[Risk] Dual modal copy needs outcome on private knowledge** → Mitigation: small additive field; default `finish` when omitted for older clients during deploy skew.
- **[Risk] Finish travel race with sync flush** → Mitigation: D10 re-fix + stronger test; align `from` seed before finished piece drops from unfinished list.

## Migration Plan

1. Blocks 1–8 already applied (knowledge economy + first polish).
2. Block 9 playtest follow-up: all-jail skip toast, Material lightbulb, cut copy, finish travel re-fix → tests.
3. No DB migration; rollback = revert both siblings together.

## Technical prerequisites

None open (Vue/Quasar Notify + material-icons already in extras; Colyseus/Drizzle only).

## Key touch points (indicative)

**Server:** `rooms/MyRoom.ts` (all-jail become-current skip + broadcast, existing seed/timer/outcome), mocha.

**Client:** `GamePage.vue` / `stores/game.ts` (icon names, finish travel, Notify all-jail), i18n cut string, vitest.

Implementation checklist: `tasks.md` (blocks 1–9 done).
