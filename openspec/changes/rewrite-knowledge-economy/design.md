## Context

See `proposal.md` — Why / Scope. Explore closed D1–D17: knowledge economy, exclusive traps 12%/15%, one move|return, free rescue, no push, no turn timer/end-turn, grille timer 3 on become-current, all-jail → advance, penultimate cut burns undelivered, focus at 3 o’clock.

Current economy is woven through private `budgets` `{steps,peeks,infinite}`, peek affordance, 60s deadline, solo leftover ∞, push, and finish **place**. This change replaces that stack.

Packages: **server** + **client** (room `tourist`).

## Goals / Non-Goals

**Goals:**

- Authoritative one-action turn + open-on-land / fling chain on server; client mirrors busy/modal locks.
- Private `knowledge` `{ delivered, pieces }` + public `Piece.remainingTurns`; HUD + transfer anim reuse former +N grant pattern.
- Fixed exclusive trap seed (clamp grille then catapult); remove density create options and push.
- Penultimate full-finish cut; knowledge modal; no solo leftover / turn timer / end-turn.

**Non-Goals:**

- Public cross-seat scoreboard / ranking UI.
- New npm deps or external services.
- Map/pack CMS changes.

## Decisions

### D1 — Fixed trap ratios, exclusive partition

- **Choice:** At `enterPlaying` / `seedPlayingEconomy`, partition `*` cells with no overlap (pick grille keys first, then catapult from the remainder — never both on one cell; Q&A never under traps). Let `n` = task-cell count. `grilleCount = min(round(n * 0.12), n)`; `catapultCount = min(round(n * 0.15), n - grilleCount)` (fixed **15%** catapult — not the old few/medium/many table). Remaining cells = Q&A tasks. Drop create options `grilleDensity` / `catapultDensity` and lobby UI; if a client still sends density options, server **ignores** them and always uses this seed.
- **Why:** Explore D1/D5; removes player tuning; exclusive cells match «задание или решётка или катапульта»; clamp order keeps `g + c ≤ n`.
- **Alt:** Keep density UI — rejected. Absolute N per map — rejected (maps vary).

### D2 — Replace budgets with knowledge fields

- **Choice:** Hard-cut private `budgets` `{steps,peeks,infinite}`. Own-HUD only (same privacy family as old budgets): room-private per-seat store (Map like today’s `budgets`) + private message **`knowledge`** `{ delivered: number, pieces: Record<pieceId, number> }` to the owning client (`client.send`). Correct solve → that piece’s carried knowledge `+= difficulty` and re-send. Finish piece → `delivered += piece.knowledge`; piece carried `= 0`; re-send. Do **not** put carried/delivered on public schema (opponents must not see own counters). Yellow BR badge and avatar counter read from this private payload.
- **Why:** Explore D2; difficulty already on tasks; mirrors existing private-budget channel.
- **Alt:** Public schema `piece.knowledge` / seat field — rejected (product: own counters only). Knowledge only on seat — rejected (tourist yellow circle needs per-piece).

### D3 — Turn machine: rescue* → one move|return → resolve → advance

- **Choice:** Remove end-turn message and turn deadline (`clearTurnDeadline` always in playing; do not mark `timeExpired` from turn/solo clocks; leave `turnUntil`/`turnBudgetSeconds` inactive at 0). Track room-private `boardActionUsed` per seat turn (reset when that seat becomes current). Allow rescue while `!boardActionUsed`. On accepted move/return set flag, run land pipeline; when pipeline goes idle: if the landed piece sits on a Q&A cell (bound or unbound, not removed), **auto-open** the shared task modal and **hold** the turn; else `advanceTurn`. Task modal open holds turn until `peekClose`/submit path, then `advanceTurn`. Reject push handlers (`push` no-op / reject). **Client local mirror (no public schema):** set a local «board-action spent» flag when `sendMove` / `sendReturnFromFinish` returns true (same optimistic gate as today’s post-send travel lock — Colyseus has no per-move ack). While the flag is set, hide/disable move, return, and rescue affordances (peekPlace/peekSubmit stay allowed for the open shared modal). Clear the flag when synchronized `currentTurnSessionId` is no longer the local seat (turn advanced or own turn ended). Optional recover: if after board-pipeline settle the moved piece never left its origin cell and no land modal opened (silent reject), clear the flag so the seat can retry.
- **Why:** Explore D13/D15/D16; open-on-land strategy breaks if push relocates without “your” land intent.
- **Alt:** Keep end-turn — rejected. Keep push free — rejected by product. Sync `boardActionUsed` publicly — rejected (own-turn UX only; local mirror enough).

### D4 — Open-on-land unifies peek entry

- **Choice:** On land (move/return/fling hop) onto Q&A cell → after trap-pipeline idle (no grille/catapult on that exclusive cell), server **auto** bind-if-needed + open shared peek session (reuse `syncPeekSession` / `peekOpen` + `peekPlace` / `peekSubmit` / `peekClose`). Session owner = **landing seat** (reopen by another seat **replaces** `openPeek.sessionId` / place rights). No peeks spend; remove peek affordance from board UX. Client `peek` intent MUST be rejected / no-op (opening is land-driven only). Wrong → KEEP open digit; the solving piece **remains** on the cell and the turn advances — a later reopen requires a **legal land** onto that cell once occupancy allows (standing alone does not reopen). Correct → knowledge on that piece + empty cell (`removedTaskKeys`), then advance.
- **Why:** Explore items 1–2, D7/D11; reuse modal chrome (SC-BOARD-46…).
- **Alt:** Separate new message name — unnecessary if session already shared. Keep client `peek` as open trigger — rejected.

### D5 — Grille remainingTurns tick on become-current

- **Choice:** Public schema field `remainingTurns` (uint, 0 when not trapped) on **`Piece`**. On trap start set to **3**. On `advanceTurn` → new current: for each trapped piece of that seat, if `remainingTurns === 1` clear grille to empty + free piece + set 0; else decrement by 1. UI badge top-right red (`game/board`) — visible to every client. Auto-free same turn may move. Rescue / all-jail clear sets 0 and clears badge.
- **Why:** Explore D9/D10; «было 1 → снова ход → убрали»; SC-BOARD-85 needs public sync.
- **Alt:** Tick on end of own turn — rejected (product: on become-current). Store only on holding-grille key map — rejected (piece chrome badge binds to piece).

### D6 — Trap clear / catapult consume → empty cell

- **Choice:** Exclusive trap cells have no underlying task and MUST never receive a Q&A bind. On grille clear (rescue / auto-free / all-jail / leave) or catapult consume after fling, mark the cell empty via the same `removedTaskKeys` / removed-task-hole path as a correct solve. Update «spent grille leaves task peekable» paths.
- **Why:** Explore D14.
- **Alt:** Reshuffle new task under cell — rejected.

### D7 — All-jail advances turn

- **Choice:** Keep reset-to-start + modal; change SC-MOVE-61 keep-turn → `advanceTurn` after reset (trap was the board action resolve).
- **Why:** Explore D17.
- **Alt:** Keep turn — rejected.

### D8 — Finish: knowledge modal + penultimate cut

- **Choice:** Reuse existing seat gate: on full finish **and** on penultimate cut set `finishPlace = 1` (boolean-ish finished; **never** show place in UI; do not assign monotonic 1/2/3; stop using / incrementing `nextFinishPlace` for UX). **One** knowledge-modal chrome for both full finish and cut: body reports private `delivered` only (burned points excluded for cut); no place number; closing keeps the seat in the room. Client opens that modal on **local** seat `finishPlace` 0→1 (same watch as today’s `celebratedPlace` / `finishPlaceModal`) and binds the body to private `knowledge.delivered` (no new room event; cut seat gets the same 0→1 edge). Hide all place badges / «Вы N-й!» copy (`presence-place-badge`, `finishPlaceModal`). Penultimate cut (`maxSeats ≥ 2`, exactly one other eligible remains): mark last seat `finishPlace = 1`, burn all carried knowledge on that seat’s unfinished pieces to 0 **without** adding to `delivered`, set those pieces `finished = true` (cannot move), re-send private `knowledge`, show the same delivered-only modal. `maxSeats === 1` skips cut.
- **Why:** Explore D4/D8/D12; minimize schema churn by reusing finished-seat gate.
- **Alt:** Ordered places kept alongside knowledge — rejected. New `finished: boolean` field — unnecessary while `finishPlace > 0` already skips turns. Distinct cut-vs-finish dialog layouts — rejected (same delivered-only chrome).

### D9 — Client HUD: knowledge, focus@3, strip badges

- **Choice:** GamePage / presence: remove steps/peeks/end-turn/turn ring/solo modals / place badges; **one** avatar knowledge counter (right of own avatar, former budgets column) reading private `knowledge.delivered` + +N fall only when `delivered` increases (piece-finish transfer — same `.budget-fall` family; correct solve updates yellow BR only, no avatar fall). Focus: reuse the **existing end-turn circumference slot** (right-center / **3 o’clock**, 36×36 illustrated glyph) — move focus there and delete end-turn control + old bottom-right focus placement. Yellow BR carried-knowledge circle on **own** board piece **and** personal strip chrome from private `knowledge.pieces` (hide when 0); red TR grille counter on **all** clients’ board + strip chrome from public `Piece.remainingTurns` (map field on piece sync like `trapped`). Outcome modal: replace `finishPlaceModal` / `celebratedPlace` with **one** knowledge modal opened on local seat `finishPlace` 0→1, body = private `knowledge.delivered` (full finish and cut); no place number / badge. Lobby create: drop grille/catapult density UI and omit those keys from create payload. Remove push + peek-icon affordances (`sendPeek` retired; modal only via land sync). Retire `soloEndModalKind` / solo unlimited UX / steps-exhaustion copy.
- **Why:** Explore HUD + D16; binds FE to existing GamePage slots (end-turn CSS, budgets fall, finishPlace watch) so apply does not invent new chrome families.
- **Alt:** Leave end-turn unused — rejected (confusing). Keep finishPlace badge as “done” mark — rejected (reads as ranking place).

## Risks / Trade-offs

- **[Risk] Large SC surface area (move/board/presence/finish/lobby)** → Mitigation: one change; mocha/vitest rewrite by capability blocks; retire push/solo/timer tests explicitly.
- **[Risk] Holding turn open during shared task modal stalls table** → Mitigation: accepted (product); leave-mid-modal KEEP + advance per close path; no wall-clock turn timer.
- **[Risk] Soft-lock with no legal move and not all-jail** → Mitigation: rare; rescue-before-move; grille auto-free; empty cells landable; if still stuck accept as edge (no end-turn by design).
- **[Risk] Reusing `finishPlace=1` for all finishers loses ordering** → Mitigation: intentional; UI never shows place.
- **[Risk] Client still sends peek/push/endTurn** → Mitigation: server reject; client remove send paths same change.

## Migration Plan

1. Server seed + schema/knowledge + turn machine + retire push/deadline/budgets → mocha.
2. Client lobby + Game HUD/board affordances + modals → vitest.
3. No DB migration; rollback = revert both siblings together (contract break).

## Technical prerequisites

None open (stack Vue/Quasar/Colyseus/Drizzle only; explore D* closed).

## Key touch points (indicative)

**Server:** `rooms/MyRoom.ts` (seed, turn, budgets, all-jail, finish), `game/touristMove.ts` (densities, push/rescue/move), schema seats/pieces, mocha room tests.

**Client:** `stores/game.ts`, `pages/GamePage.vue`, lobby create modal, board affordance components, `lib/soloEndModalKind` retire/adapt, i18n `game.*`, vitest GameBoardWire / presence / lobby.

Implementation checklist: `tasks.md`.
