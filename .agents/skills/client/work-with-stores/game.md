# Game store (`stores/game.ts`)

Read with the [core stores skill](SKILL.md). Board UI: `work-with-game-board`; lobby subscribe: `work-with-lobby`; reconnect: `work-with-rooms`. Messages: `colyseus-client/game-messages.md`.

## Ownership

`game` owns lobby listing + tourist room lifecycle, reconnect token, and mirrored
state: seats (pieces by **pieceId**, incl. `trapped` / `remainingTurns`) / `phase` /
`maxSeats` / `grid` / `touristsPerPlayer` / `packTitle` / `flippedCells` /
`answerCards` / peek session / turn / `removedTaskKeys` / grilles/catapults;
private `knowledgeDelivered` / `knowledgePieces` / `knowledgeSynced` /
`allJailWarning`; `peekStatusFlash` from room `peekClose` (`correct` +
`reason?: 'wrong'` → 10s header flash);
`createGame({ mapId, packId, taskSetIds, maxSeats })` (no density keys);
`sendMove` / `sendRescue` / `sendReturnFromFinish` / `sendPeekPlace` /
`sendPeekSubmit` / `sendReady` / `sendSay`. `sendPush` / `sendEndTurn` /
`sendPeek` retired (no-op / always false).

Consumers: `LobbyPage`, `GamePage`, `App.vue` leave. Selection / hints /
board-busy / say picker / modals stay page-local on `GamePage`.

Room constants: `TOURIST_ROOM = 'tourist'`, `LOBBY_ROOM = 'lobby'`.

## Options store pattern (knowledge economy)

```ts
export const useGameStore = defineStore('game', {
  state: () => ({
    rooms: [],
    lobbyRoom: null,
    lobbyWanted: false,
    room: null,
    roomId: null,
    sessionId: null,
    phase: 'waiting',
    maxSeats: 2,
    countdownRemaining: 0,
    started: false, // legacy mirror of phase === 'playing'
    currentTurnSessionId: '',
    turnUntil: 0, // inactive (deadlines retired)
    turnBudgetSeconds: 0,
    seats: [], // pieces include remainingTurns
    removedTaskKeys: [],
    knowledgeDelivered: 0,
    knowledgePieces: {},
    knowledgeSynced: false,
    openPeek: null,
    peekStatusFlash: null,
    status: 'idle',
    error: null,
    listing: false,
  }),
  getters: {
    isInRoom: (state) => Boolean(state.room),
    mySeat: (state) =>
      state.seats.find((s) => s.sessionId === state.sessionId) ?? null,
    isSeated: (state) =>
      Boolean(state.sessionId && state.seats.some((s) => s.sessionId === state.sessionId)),
    isMyTurn: (state) =>
      Boolean(
        state.sessionId &&
        state.currentTurnSessionId &&
        state.sessionId === state.currentTurnSessionId &&
        state.seats.some((s) => s.sessionId === state.sessionId),
      ),
    isPlaying: (state) => state.phase === 'playing',
    canSendReady: (state) => { /* waiting + ≥2 + under maxSeats + own !ready */ },
    canSendEndTurn: () => false, // retired SC-PRESENCE-55
  },
  actions: {
    async subscribeLobby() { /* lobbyWanted + joinOrCreate lobby + quiet resubscribe */ },
    async unsubscribeLobby() {
      /* sync: lobbyWanted=false, rooms=[], lobbyRoom=null;
         then Promise.race(leave.catch(…), LOBBY_LEAVE_TIMEOUT_MS=1500) — SC-LOBBY-37 */
    },
    async createGame(options: {
      mapId: string;
      packId: string;
      taskSetIds: string[];
      maxSeats: number;
    }) {
      return this._enterRoom(() => client.create(TOURIST_ROOM, options));
    },
    sendMove(pieceId, row, col) { /* one board action; no steps budget */ },
    sendRescue(pieceId) { /* free before board action */ },
    sendPush() { return false; },
    sendReturnFromFinish(pieceId, row, col) { /* board action */ },
    sendPeek() { return false; },
    sendPeekPlace(slotIndex, answerCardId) { /* … */ },
    sendPeekSubmit() { /* … */ },
    sendEndTurn() { return false; },
    sendReady() { /* … */ },
    sendSay(presetId) { /* … */ },
  },
});
```

## Attach notes

- `onMessage('knowledge', …)` → update delivered/pieces; set `knowledgeSynced=true` after first payload (gate +N falls).
- Do **not** listen for `budgets`.
- Piece mirror: map `remainingTurns` from schema.
- `finishPlace` 0→1 is a boolean-ish finished/cut gate — GamePage opens knowledge modal, not place UX.
