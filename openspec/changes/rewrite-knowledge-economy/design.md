## Context

See `proposal.md` — Why / Scope. Explore closed D1–D17 (knowledge economy) plus polish: restore 60s turn timer + timeout toast, grille 15%, all-jail wait (no teleport), strip-only carried knowledge + lightbulb, grille badge z-index, Notify toasts, dual finish/cut copy, finish-travel bugfix.

Packages: **server** + **client** (room `tourist`).

## Goals / Non-Goals

**Goals:**

- Authoritative one-action turn + open-on-land / fling chain; **60s** turn deadline with ring; timeout force-closes open Q&A as «Не успел!» + top toast.
- Private `knowledge` `{ delivered, pieces, outcome? }` + public `Piece.remainingTurns`; strip-only carried TR + lightbulb; avatar delivered + +N.
- Fixed exclusive trap seed **15% grille / 15% catapult** (clamp grille then catapult); remove density create options and push.
- All-jail: stay trapped until remainingTurns expire — no start teleport / no start-warning modal.
- Penultimate cut; knowledge modal with **distinct** finish vs cut copy; reliable finish travel anim (click + catapult).

**Non-Goals:**

- Public cross-seat scoreboard / ranking UI.
- New npm deps beyond enabling Quasar **Notify** (already in Quasar package).
- Map/pack CMS changes.
- Solo leftover ∞ / permanent `timeExpired` from a leftover-only clock (turn timeout advances / closes modal only).

## Decisions

### D1 — Fixed trap ratios, exclusive partition

- **Choice:** At `enterPlaying` / `seedPlayingEconomy`, partition `*` cells with no overlap (pick grille keys first, then catapult from the remainder — never both on one cell; Q&A never under traps). Let `n` = task-cell count. `grilleCount = min(round(n * 0.15), n)`; `catapultCount = min(round(n * 0.15), n - grilleCount)` (**15%/15%**). Remaining cells = Q&A tasks. Drop create options `grilleDensity` / `catapultDensity` and lobby UI; if a client still sends density options, server **ignores** them.
- **Why:** Explore polish — equal trap pressure; clamp order keeps `g + c ≤ n`.
- **Alt:** Keep 12%/15% — rejected after playtest feedback. Density UI — rejected.

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

### D7 — All-jail waits on remainingTurns (no teleport)

- **Choice:** When a seat’s unfinished pieces are all trapped, the server MUST **NOT** clear grilles, MUST **NOT** relocate pieces to start, and MUST **NOT** send `allJailWarning` / show the start-reset modal. Pieces stay trapped with their `remainingTurns`. After the trapping land’s presentation settles, the turn **advances** (trap was the board action). Countdown continues via become-current ticks (D5) until auto-free. Retire client `allJailWarning` modal chrome / i18n for start-reset (or leave inert).
- **Why:** Product polish — «пусть ждут ходы ожидания».
- **Alt:** Keep start teleport — rejected.

### D8 — Finish: dual-copy knowledge modal + penultimate cut

- **Choice:** `finishPlace = 1` gate for full finish and cut (no place UX / no `nextFinishPlace` bump). Client opens **one** modal chrome on local `finishPlace` 0→1. Body uses private `knowledge.delivered` **and** `outcome`:
  - `finish`: product Russian sense — all your tourists reached finish and earned {n};
  - `cut`: all opponents finished; your tourists that made it earned {n}.
  Penultimate cut (`maxSeats ≥ 2`): burn carried→0 without adding to delivered, unfinished pieces `finished=true`, re-send `knowledge` with `outcome: 'cut'`. `maxSeats === 1` skips cut.
- **Why:** Explore D4/D8/D12 + distinct copy request.
- **Alt:** Same single sentence for both — rejected after polish.

### D9 — Client HUD: strip knowledge, focus@3, toast, timer ring

- **Choice:**
  - Avatar: delivered counter + +N on delivered growth; **`mdi-lightbulb-on`** (or same family) beside/in the counter chrome.
  - Carried: **only** personal strip slots, yellow badge **top-right** + lightbulb icon; **remove** board-piece yellow BR badge.
  - Grille: red TR board+strip; z-index above grille overlays.
  - Focus @3 o’clock 36×36; no end-turn; no place badges; no solo leftover/unlimited modals.
  - Outcome flashes: Quasar **Notify** top-center for correct / wrong / timeout («Правильно!» / «Ошибка!» / «Не успел!»); prefer Notify over (or instead of) quiet header `game-status` flash so the result is obvious. Enable `Notify` in `quasar.config.ts` plugins.
  - Lobby: no density UI/payload.
- **Why:** Explore HUD polish.
- **Alt:** Keep board BR knowledge — rejected. Header-only flash — rejected.

### D10 — Finish travel animation reliability

- **Choice:** Ensure click-to-center and catapult-fling-to-center always run the existing `FINISH_TRAVEL_MS` (~550) + fade (~300) path: seed/preserve `lastKnown` / `finishAnimFrom` / `catapultDeferFinishKeys` so pieces never snap-vanish. Fix races where a second finisher or fling hop skips travel. Keep server finish marking; client presentation only unless a sync race is found.
- **Why:** Reported vanish without move anim.
- **Alt:** Accept instant vanish — rejected.

## Risks / Trade-offs

- **[Risk] Timer + open modal stalls less but pressures answers** → Mitigation: accepted; timeout = «Не успел!» KEEP + advance.
- **[Risk] All-jail without teleport soft-locks a seat until ticks** → Mitigation: other seats advance turns so become-current ticks; auto-free at 1; rescue still available before board action when a free piece exists.
- **[Risk] Notify plugin change** → Mitigation: Quasar built-in; no new npm package.
- **[Risk] Dual modal copy needs outcome on private knowledge** → Mitigation: small additive field; default `finish` when omitted for older clients during deploy skew.

## Migration Plan

1. Blocks 1–6 already applied (knowledge economy without timer / with 12% grille / all-jail teleport).
2. Polish apply: server seed 15%, restore deadline, all-jail wait, knowledge.outcome; client strip/toast/ring/z-index/finish anim/i18n → tests.
3. No DB migration; rollback = revert both siblings together.

## Technical prerequisites

None open (Vue/Quasar Notify plugin enable; Colyseus/Drizzle only).

## Key touch points (indicative)

**Server:** `rooms/MyRoom.ts` (seed 15%, turn deadline, all-jail, finish outcome), schema, mocha.

**Client:** `quasar.config.ts` (Notify), `stores/game.ts`, `pages/GamePage.vue`, `App.vue` status flash → Notify, i18n, vitest.

Implementation checklist: `tasks.md` (blocks 1–6 done; block 7+ polish pending).
