---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist tourist server — onMessage handlers (move, rescue, push,
  returnFromFinish, peek, peekAnswer, endTurn, ready, say), private
  budgets/peekOpen/allJailWarning sends, paced trap pipeline / pendingTurnAdvance
  / idle re-eval (SC-MOVE-93), validating client intents before mutating schema
  state or broadcasting ephemeral events, optional error feedback, or aligning
  message contracts with the client. Not for lobby listing (LobbyRoom / HTTP
  fallback).
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server` (authoritative Colyseus room handlers).

Skills path: `happy-tourist-meta/.agents/skills/server/`. Runtime paths below are relative to this server repo root; sibling client is `../happy-tourist.github.io`.

Coordinate with: `work-with-rooms` (lifecycle / registration), `work-with-schema` (synced state), `work-with-game` (rules / budgets), `server-work-with-errors` (reject / feedback style). Client bubbles UI: `.agents/skills/client/work-with-game-board`.

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage(...)` for game actions (`move`, `rescue`, `push`, `returnFromFinish`, `peek`, `peekAnswer`, `endTurn`, `ready`, `say`) | Lobby listing — client uses **LobbyRoom** (`rooms` / `+` / `-`) |
| Private `client.send('budgets'|'peekOpen'|'allJailWarning', …)` to owner | Auth join gate — `onAuth` + JWT (`server-work-with-auth`) |
| Payload shape / type guards for client intents | Express routes / `createEndpoint` |
| Validate → mutate `@colyseus/schema` state (`move` / `rescue` / `push` / `returnFromFinish` / `peekAnswer` / `ready`) or ephemeral broadcast (`say`) | Pure board-game rules (`work-with-game` / `touristMove.ts`); free-form chat |
| Optional per-client error feedback for bad actions | |

**Prefer changing the server to match the client** rather than inventing a parallel protocol. Do not treat legacy draughts `move` `{ from, to }` as product canon.

## Client Contract (today)

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `move` | `{ side: 'N'\|'E'\|'S'\|'W', row: number, col: number }` — via `sendMove` when `phase === 'playing'`, `isMyTurn`, `!isMySeatFinished`, `!isMySeatTimeExpired`. Spends a step; **does not** advance turn; land → paced trap pipeline (grille / catapult) |
| Client → server | `rescue` | `{ side }` — via `sendRescue`; own trapped + adj free own piece + steps≥1; then remaining catapult on cell resolves |
| Client → server | `push` | `{ pusherSide, targetSessionId, targetSide, row, col }` — via `sendPush`; own free pusher + adj free target → far-side cell; −1 step; relocates target only; land → paced trap pipeline |
| Client → server | `returnFromFinish` | `{ side, row, col }` — via `sendReturnFromFinish`; finished side onto legal center-ring cell; land → paced trap pipeline |
| Client → server | `peek` | `{ side }` — via `sendPeek`; own unfinished **non-trapped** piece on present `*`; server replies with private `peekOpen` |
| Client → server | `peekAnswer` | `{ correct: boolean }` — via `sendPeekAnswer`; Correct removes tile; Incorrect KEEP |
| Client → server | `endTurn` | empty — via `sendEndTurn` when `canSendEndTurn` (multi finite only); **reject** while trap pipeline active |
| Client → server | `ready` | empty — via `sendReady` (waiting, ≥2 seated, under maxSeats, not yet ready) |
| Client → server | `say` | `{ presetId: 'hello' \| 'luck' }` — via `sendSay`; whitelist only (no free text; **not** `ready`); finished seats may still say |
| Server → owner | `budgets` | `{ steps, peeks, infinite, peekedThisTurn }` — private; `infinite` = **peeks∞ only** (steps always finite); `peekedThisTurn` legacy (no gate) |
| Server → owner | `peekOpen` | `{ side, row, col, reward: 1\|2\|3 }` — private modal payload |
| Server → owner | `allJailWarning` | `{}` — private; seat just freed from all-jail (informational modal) |
| Server → clients | Schema sync | `phase` / `maxSeats` / `countdownRemaining` / `seats` (+ `ready` / `finishPlace` / piece `finished`/`trapped`) / `currentTurnSessionId` / `removedTaskKeys` / `holdingGrilleKeys` / `revealingCatapultKeys` / `brokenCatapultKeys` (+ `nextFinishPlace`) → `onStateChange` |
| Server → all clients | `say` | `broadcast('say', { sessionId, presetId, at })` — ephemeral; includes readiness preset from successful `ready` |
| Server → client | room error channel | Client sets `game.error` from `room.onError` |
| Lobby | HTTP fallback | `client.http.get('/rooms/tourist')` — **not** a room message |

**`move`:** accept only when `phase === 'playing'` + seated + `finishPlace === 0` + `!timeExpired` + current turn + `steps > 0` + **trap pipeline idle**; legal one-step per `touristMove.ts` (occupancy ignores finished; trapped still occupy; **landing on removed holes rejected**; stand/leave OK; **reject if piece trapped**). Reject (no mutate) otherwise. On accept: update piece `row`/`col`; always spend 1 step + `sendBudgets`; if target is center → `piece.finished = true` and maybe assign `finishPlace`; else paced trap pipeline (one hop + presentation budget; catapult fling/broken or grille); **do not** `advanceTurn` solely for move — then `maybeAutoEndTurn` / solo step-loss / finish-advance (**deferred** via `pendingTurnAdvance` while pipeline active; on idle: pending → `advanceTurn`, else **re-eval** auto-end/solo — SC-MOVE-93).

**`rescue`:** current turn + steps≥1 + own trapped unfinished + Chebyshev-1 free own piece + **pipeline idle** → −1 step; clear `trapped` + holding grille; rescuer coords unchanged; then paced pipeline if a catapult remains. Silent reject otherwise.

**`push`:** current turn + steps≥1 + own unfinished free `pusherSide` + free (non-trapped) target at Chebyshev-1 + **pipeline idle** → dest = far-side cell `target + (target − pusher)`; reject hole/occupied/bad geometry/trapped; −1 step; relocate **target only** (pusher stays); land side-effects like `move` (paced pipeline on target). Does **not** advance turn. Silent reject otherwise.

**`returnFromFinish`:** current turn + steps≥1 + `finishPlace===0` + own finished side + legal ring cell (Chebyshev-1 from center block, excl. center/holes/occupancy) + **pipeline idle** → −1 step; unfinish onto cell → paced pipeline. Silent reject otherwise.

**`peek`:** current turn + peek budget (or solo peeks∞) + **pipeline idle**; **no** one-peek/turn gate; piece on present task cell and **not trapped**; set room `openPeek` + `client.send('peekOpen', …)`. Silent reject otherwise.

**`peekAnswer`:** resolve open peek for that seat; spend peek (finite peeks only); Correct adds reward steps (always, including solo) and pushes `"r,c"` to `removedTaskKeys`; Incorrect KEEP tile + hidden reward; `sendBudgets`; then maybe auto-end / solo step-loss.

**`endTurn`:** multi (≥2 eligible) + current + not finished/expired + **pipeline idle**; force-close open peek as incorrect KEEP if any; `advanceTurn` (next grant: multi +1/+1; solo become-current +1 step only). Solo peeks∞ → reject. While pipeline active → silent reject (board-lock parity).

**`ready`:** accept only in `waiting`, seated + connected, seated count ≥ 2 and `< maxSeats`, seat not already ready. On accept: `seat.ready = true`, broadcast say preset `ready` (bypass live-say cap), maybe start countdown. Silent reject otherwise.

**`say`:** accept only when seated + `connected`; `presetId` ∈ `{ hello, luck }` (**reject** `ready` — use `ready` message). At most **3** live says per `sessionId` within **10s** TTL. Display: `hello` → «Всем привет», `luck` → «Удачи», `ready` → «Готов начать!» (client i18n).

## Server Today

- `src/rooms/MyRoom.ts` — seating + start + budgets/peek/grilles/catapults + paced trap pipeline (`startTrapPipeline` / `pendingTurnAdvance` / idle re-eval SC-MOVE-93) + `onMessage('move'|'rescue'|'push'|'returnFromFinish'|'peek'|'peekAnswer'|'endTurn'|'ready'|'say')`; `handlePush`; auto-end includes `hasLegalPush`; turn actions rejected while pipeline active.
- Pure move/peek/grille/push/catapult helpers: `src/game/touristMove.ts` (`farSideCell`, `validateTouristPush`, `hasLegalPush`, `catapultFlingCandidates`, `pickCatapultFlingDest`).
- Room registered as `tourist` (+ `lobby` for live list); do not reintroduce `my_room`.
- **No** dedicated catapult room message — reveal/broken are schema-only (`revealingCatapultKeys` / `brokenCatapultKeys`).

## Handler Pattern

Register handlers in the room (typically `onCreate`), not in Express:

```ts
this.onMessage('move', (client, message) => {
  // 1. Require phase === 'playing'
  // 2. Shape-check { side, row, col }
  // 3. Resolve seat; reject if finishPlace > 0 / timeExpired / no steps / trapped; require current turn
  // 4. Reject unknown/finished piece; validate via touristMove (occupancy ignores finished)
  // 5. On success: mutate row/col; spend step; if center → finished + maybe finishPlace
  //    else maybe reveal grille + trap / all-jail
  // 6. Do NOT advance solely on move; maybeAutoEndTurn / finish-advance
});

this.onMessage('rescue', (client, message) => {
  // 1. playing + current + steps≥1; shape { side }
  // 2. own trapped + Chebyshev-1 free own piece → −1 step; clear trapped + holding grille
});

this.onMessage('push', (client, message) => {
  // 1. playing + current + steps≥1; shape { pusherSide, targetSessionId, targetSide, row, col }
  // 2. validateTouristPush → −1 step; relocate target only; finish/trap side-effects; no turn advance
});

this.onMessage('returnFromFinish', (client, message) => {
  // 1. playing + current + steps≥1 + finishPlace===0; shape { side, row, col }
  // 2. legal ring cell → −1 step; unfinish onto cell
});

this.onMessage('peek', (client, message) => {
  // 1. playing + current + peek budget (or solo peeks∞); no one-peek gate; not trapped
  // 2. piece on present *; openPeek + client.send('peekOpen', { side, row, col, reward })
});

this.onMessage('peekAnswer', (client, message) => {
  // 1. Shape-check { correct: boolean }; must own openPeek
  // 2. resolveOpenPeek → spend peek (finite); Correct → +steps + remove tile; Incorrect KEEP; maybeAutoEndTurn / solo step-loss
});

this.onMessage('endTurn', (client) => {
  // 1. multi + current + eligible; force-close open peek as incorrect KEEP if any
  // 2. advanceTurn → applyTurnGrant (multi +1/+1; solo become-current +1 step)
});

this.onMessage('ready', (client) => {
  // 1. phase === 'waiting'; seated+connected; ≥2 and < maxSeats; !seat.ready
  // 2. seat.ready = true; broadcast say preset 'ready' (bypass live cap)
  // 3. maybeStartCountdown()
});

this.onMessage('say', (client, message) => {
  // 1. Seat exists + seat.connected
  // 2. presetId ∈ hello|luck (reject 'ready')
  // 3. Prune liveSays; reject if ≥ SAY_MAX_LIVE
  // 4. broadcast('say', { sessionId, presetId, at })
});
```

### Hard rules

1. **Do not trust the client.** Treat payloads as intent only.
2. **Validate, then mutate** schema state for gameplay (`move` / `rescue` / `push` / `returnFromFinish` / peek resolve); for `say`, validate then broadcast only — do **not** put bubbles in schema. Budgets / hidden grilles stay room-private + `client.send` (`budgets` / `peekOpen` / `allJailWarning`).
3. **Authoritative move/peek/grille/push rules** live in `work-with-game` / `touristMove.ts`, not in the message payload.
4. **Do not** add HTTP endpoints for gameplay actions.
5. **No free-form say text** — whitelist `presetId` only.

## Error Feedback (align with `server-work-with-errors`)

| Situation | Preferred approach |
|-----------|-------------------|
| Malformed / illegal action (`move` / `peek` / `say` / …) | **Do not mutate state / do not broadcast.** Silent reject is OK. |
| Optional user-visible feedback | Prefer `room.onError` patterns the client already maps to `game.error`. Do not invent new message types unless the client is updated in the same change. |
| Auth / join failures | Not message-handlers — `onAuth` / join reject. |

## New Message Types

1. Confirm the sibling client will `room.send` / `room.onMessage` the same name and payload.
2. For schema-backed actions: validate → mutate schema — never trust client state. For ephemeral events like `say`: validate → broadcast; keep room-private bookkeeping (`budgets`, `taskRewards`, `openPeek`) out of schema.
3. Keep lobby listing on LobbyRoom / HTTP `GET /rooms/:roomName`.

## Do / Don't

| Do | Don't |
|----|--------|
| Keep `move` `{ side, row, col }` lockstep with client `sendMove` | Reintroduce draughts `move`/`{from,to}` without a product change |
| Keep `rescue` / `push` / `returnFromFinish` + private `allJailWarning` lockstep with store | Advance turn solely because `move` / `push` succeeded |
| Keep `peek` / `peekAnswer` / `endTurn` + private `budgets` / `peekOpen` lockstep with store | Sync hidden grille locations into schema |
| Keep `say` `{ presetId }` + broadcast `{ sessionId, presetId, at }` lockstep with `sendSay` / `onMessage('say')` | Accept free-form text or unknown preset ids |
| Validate then mutate schema (`move` / rescue / push / return / remove tile on Correct only); broadcast-only for `say` | Trust client board hints; sync say bubbles or steps/peeks into schema |
| Treat lobby as LobbyRoom live list | Add a gameplay room message for room listing |
| Follow `server-work-with-errors` | Invent BFF-style error envelopes for room actions |

## Change Checklist

1. Handler registered on the tourist room via `this.onMessage`.
2. Payload matches the client store (`sendMove` / `sendRescue` / `sendPush` / `sendReturnFromFinish` / `sendPeek` / `sendPeekAnswer` / `sendEndTurn` / `sendReady` / `sendSay`).
3. Illegal actions do not mutate schema (and `say` rejects do not broadcast).
4. Move/peek/grille/push rules delegated to `work-with-game` / `touristMove.ts`; start/ready/countdown in room; say stays I/O + whitelist + live-limit in the room handler.
5. Update `test/` / `loadtest/` when the message contract becomes testable.
6. Run `npm test` from the server package root; fix failures before claiming done.

## Related

- Room lifecycle / registration: `.agents/skills/server/work-with-rooms/SKILL.md`
- Schema sync fields: `.agents/skills/server/work-with-schema/SKILL.md`
- Rules engine: `.agents/skills/server/work-with-game/SKILL.md`
- Reject / log / feedback: `.agents/skills/server/server-work-with-errors/SKILL.md`
- Client board: `.agents/skills/client/work-with-game-board/SKILL.md`
- Package overview: `../happy-tourist-server/AGENTS.md`
