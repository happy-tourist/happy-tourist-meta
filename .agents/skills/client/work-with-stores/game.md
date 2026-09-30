# Game store (`stores/game.ts`)

Read with the [core stores skill](SKILL.md). Board UI: `work-with-game-board`; lobby subscribe: `work-with-lobby`; reconnect: `work-with-rooms`.

## Ownership

`game` owns lobby listing + tourist room lifecycle, reconnect token, and mirrored
state: seats (pieces by **pieceId**) / `phase` / `maxSeats` / `grid` /
`touristsPerPlayer` / `packTitle` / `flippedCells` / `answerCards` / peek session /
turn / `removedTaskKeys` / grilles/catapults; private `budgets` + `openPeek` /
`allJailWarning`; `createGame({ mapId, packId, taskSetIds, maxSeats,
grilleDensity?, catapultDensity? })` (`maxSeats` `1…map.players`; server omit →
`min(2, map.players)`); `sendMove` / `sendRescue` / `sendPush` /
`sendReturnFromFinish` / `sendPeek*` / `sendEndTurn` / `sendReady` / `sendSay`.

Consumers: `LobbyPage`, `GamePage`, `App.vue` leave. Selection / hints /
`moveAnimating` / say picker / modals stay page-local on `GamePage`.

Room constants: `TOURIST_ROOM = 'tourist'`, `LOBBY_ROOM = 'lobby'`.

## Options store pattern

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
    turnUntil: 0,
    turnBudgetSeconds: 0,
    seats: [],
    removedTaskKeys: [],
    steps: 0,
    peeks: 0,
    budgetsInfinite: false,
    peekedThisTurn: false,
    openPeek: null,
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
    canSendEndTurn: (state) => { /* playing + isMyTurn + !budgetsInfinite + !finished + !timeExpired */ },
  },
  actions: {
    async subscribeLobby() { /* lobbyWanted + joinOrCreate lobby + quiet resubscribe */ },
    async unsubscribeLobby() { /* lobbyWanted=false; leave lobbyRoom */ },
    async createGame(options: { mapId: string; packId: string; taskSetIds: string[]; maxSeats: number; grilleDensity?: 'few' | 'medium' | 'many'; catapultDensity?: 'few' | 'medium' | 'many' }) {
      return this._enterRoom(() => client.create(TOURIST_ROOM, options));
    },
    async rejoinGame(roomId, options = {}) {
      // localStorage reconnect(token) → clear stale on fail → fallback joinById
    },
    sendMove(pieceId, row, col) {
      if (
        !this.room ||
        this.phase !== 'playing' ||
        !this.isMyTurn ||
        this.isMySeatFinished ||
        this.isMySeatTimeExpired
      ) {
        return false;
      }
      this.room.send('move', { pieceId, row, col });
      return true;
    },
    sendPeek(pieceId) { /* playing + isMyTurn → room.send('peek', { pieceId }) */ },
    sendPeekPlace(slotIndex, answerCardId) { /* room.send('peekPlace', …) */ },
    sendPeekSubmit() { /* room.send('peekSubmit', {}) */ },
    sendEndTurn() {
      if (!this.room || !this.canSendEndTurn) return false;
      this.room.send('endTurn');
      return true;
    },
    sendReady() {
      if (!this.room || !this.canSendReady) return false;
      this.room.send('ready');
      return true;
    },
    sendSay(presetId) {
      // whitelist hello|luck only (block ready); seated + connected; max 3 live
      this.room.send('say', { presetId });
      return true;
    },
  },
});
```

Notes:
- Live lobby listing uses `subscribeLobby` / LobbyRoom messages — not LobbyPage HTTP poll. `refreshRooms` HTTP remains unused fallback. Set `lobby.reconnection.enabled = false`; filter reservation/reconnect noise (see `work-with-lobby`).
- **SC-LOBBY-20:** clear `rooms = []` on subscribe start / unsubscribe / quiet resubscribe; keep `listing` true until the first fresh `rooms` snapshot (clear `listing` in `onMessage('rooms')`, not in a subscribe `finally`).
- Mirror `seats` (incl. connectivity + `ready` + `finishPlace` + `timeExpired` + piece `trapped`) / `phase` / `maxSeats` / `countdownRemaining` / `currentTurnSessionId` / `turnUntil` / `turnBudgetSeconds` / `removedTaskKeys` / `holdingGrilleKeys` / `revealingCatapultKeys` / `brokenCatapultKeys` / `sessionId` in the store; **D13:** `$patch` seats + revealing/broken catapult keys in one tick (avoid split mirror). Listen `budgets` / `peekOpen` / `allJailWarning` privately. Keep tile geometry + presence + selection/hints + peek/rescue/push/return/end-turn chrome + catapult sequential overlays (land→overlay→fling / continuous board-busy) + say/timeout/place/solo/all-jail modals on `GamePage` (not Pinia). Ephemeral `sayEvents` stay in the store (room I/O).
- Persist tourist `reconnectionToken` in `localStorage` (`ht-tourist-reconnect`); clear on consented `leaveGame` / `_leaveTouristRoom` and after failed `reconnect`; keep on unexpected `onLeave`; cross-tab steal OK (see `work-with-rooms`).
- `leaveGame` sets store `consentedLeaving` around room clear (gates GamePage soft-drop), unsubscribes lobby, swallows leave errors (room may already be closed), then clears the flag in `finally`. `GamePage` calls `rejoinGame(roomId)` on mount / soft-fail (reconnect → `joinById`). Leave confirm UX lives in `App.vue` on Game (`work-with-pages`).
