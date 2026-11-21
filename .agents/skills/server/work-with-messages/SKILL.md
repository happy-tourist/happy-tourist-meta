---
name: work-with-messages
description: >-
  Use when adding, changing, reviewing, or debugging Colyseus room messages in
  the happy-tourist tourist server — onMessage handlers (move, rescue,
  returnFromFinish, peekPlace, peekSubmit, ready, say; push/peek/endTurn
  reject), private knowledge sends (`outcome?`), `allJailSkip` broadcast when
  eligible ≥2 (`allJailWarning` retired), open-on-land peek, paced trap
  pipeline / pendingTurnAdvance / idle ensure+absorb / 60s multi timeout
  `peekClose.reason:'timeout'`, validating intents before schema mutate, or
  aligning contracts with the client. Not lobby listing.
---

# Work With Messages

Use this skill for **room WebSocket messages** in `happy-tourist-server`.

Coordinate with: `work-with-rooms`, `work-with-schema`, `work-with-game` (rules / knowledge), `server-work-with-errors`. Client: `.agents/skills/client/colyseus-client/game-messages.md`.

## Scope Boundary

| In scope | Out of scope |
|----------|--------------|
| `this.onMessage(...)` for game actions | Lobby listing (LobbyRoom) |
| Private `client.send('knowledge', …)` (`outcome?`; `allJailWarning` retired) | Auth join (`onAuth`) |
| Payload guards; open-on-land peek session | Express routes |
| Validate → mutate schema or ephemeral `say` | Pure geometry (`touristMove.ts`) |

## Client Contract (knowledge economy)

| Direction | Name | Payload / behavior |
|-----------|------|--------------------|
| Client → server | `move` | `{ pieceId, row, col }` — playing + eligible + `!boardActionUsed` + pipeline idle → set board action; land → paced traps → open-on-land **or** advance |
| Client → server | `rescue` | `{ pieceId }` — free while `!boardActionUsed`; clears grille → empty; does **not** spend board action |
| Client → server | `returnFromFinish` | `{ pieceId, row, col }` — board action like move |
| Client → server | `peekPlace` / `peekSubmit` | peeker only while shared session open |
| Client → server | `ready` / `say` | waiting ready; whitelist `hello`/`luck` |
| Client → server | `push` / `peek` / `endTurn` | **Reject / no-op** (retired) |
| Server → owner | `knowledge` | `{ delivered, pieces: Record<pieceId, number>, outcome?: 'finish'\|'cut' }` |
| Server → owner | `allJailWarning` | **Retired** — server must not send (all-jail wait; SC-MOVE-119) |
| Server → all | `allJailSkip` | `{}` when seat skipped for all-jail and `countEligible() ≥ 2` (SC-MOVE-124/125 / SC-PRESENCE-57) |
| Server → all | `peekClose` | `{ sessionId, correct, row, col, reason?: 'wrong'\|'timeout' }` — multi 60s timeout uses `'timeout'` |
| Server → all | `say` | ephemeral broadcast |
| Schema | — | seats/pieces/`remainingTurns`/peek session/traps/`turnUntil`/holes |

**`move`:** reject if trapped piece, pipeline busy, openPeek held by another path incorrectly, or board action already used. Center → finish knowledge transfer + maybe `finishPlace=1` / cut. Removed holes landable.

**`rescue`:** own trapped + Chebyshev-1 free own piece; silent reject otherwise.

**Open-on-land:** after pipeline idle on Q&A cell → bind-if-needed + sync peek session (owner = landing seat). Client `peek` open rejected. Correct → +difficulty on piece + remove tile; wrong → KEEP + advance.

**`peekClose` reason:** `'wrong'` on wrong submit / leave-force; `'timeout'` on multi turn deadline (SC-MOVE-112/123 / SC-BOARD-70); omit when `correct`.

## Server Today

- `MyRoom.ts` — register move/rescue/return/peek*/push/endTurn/ready/say; private knowledge map; `boardActionUsed`; exclusive trap seed; paced pipeline.
- Helpers: `touristMove.ts` (`pickExclusiveTrapKeys`, move/rescue/return legality, fling).
- Room `tourist`; **no** dedicated catapult message (schema reveal keys only).

## Rules

| Do | Don't |
|----|--------|
| Prefer server match client contract; silent reject bad intents | Revive `budgets` / density create / push success / endTurn advance / peek-open spend / `allJailWarning` |
| Private knowledge only to owner; public `remainingTurns` on Piece | Put carried/delivered on schema; invent draughts `{ from, to }` |
| Hold turn while shared peek open; multi timeout → `'timeout'` + advance; all-jail wait → advance | Permanent leftover `timeExpired`; all-jail teleport |

## Checklist

1. Handler registered; payload typed; authority checks (phase/seat/turn/pipeline/boardAction).
2. Knowledge re-sent after solve/finish/cut/burn.
3. Client store listens `knowledge` (not `budgets`); UI has no push/end-turn/peek-open.
4. Mocha covers reject paths for retired messages + knowledge/open-on-land/cut.
