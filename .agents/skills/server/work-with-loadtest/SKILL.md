---
name: work-with-loadtest
description: >-
  Guides editing and running @colyseus/loadtest scripts for happy-tourist-server
  (loadtest/example.ts, npm run loadtest, room name, JWT for onAuth). Use when
  the user asks about loadtest, multi-client join pressure, loadtest script
  changes, or aligning loadtest with room rename / auth.
---

# Work With Loadtest

Use this skill for **`@colyseus/loadtest`** in `happy-tourist-server`.

Skills path for now: `.agents/skills/server/` in this repo (canonical copy may later live under `happy-tourist-meta`). Runtime paths below are relative to this server repo root.

## Map

| Piece | Path / command | Role |
|-------|----------------|------|
| Script | `loadtest/example.ts` | Per-client bot: `Client` → `joinOrCreate` → listeners → `cli(main)` |
| npm | `npm run loadtest` | `tsx loadtest/example.ts --room tourist --numClients 2` |
| Room under test | `src/app.config.ts` `rooms` | Must match `--room` / script default |
| Auth gate | `src/rooms/MyRoom.ts` `onAuth` | `JWT.verify(token)` — joins fail without a valid token |
| Mirror in tests | `test/MyRoom.test.ts` | Sets `colyseus.sdk.auth.token` before connect |

Default loadtest endpoint (from `@colyseus/loadtest` CLI): `ws://localhost:2567`. Server must already be running (`npm run dev` / `npm start`).

## Current Script Shape

`loadtest/example.ts` today:

1. `main(options)` builds `new Client(options.endpoint)`.
2. `client.joinOrCreate(options.roomName, { /* join options */ })`.
3. Hooks: `room.onMessage(...)`, `room.onStateChange(...)`, `room.onLeave(...)`.
4. `cli(main)` at module bottom — required so `@colyseus/loadtest` spawns N clients.

Do not remove `cli(main)`. Prefer extending `main` (messages, moves, auth) over inventing a second entrypoint unless the user asks.

## Room Name

Registered playable room is **`tourist`** (with `lobby` for live listing). Keep loadtest in sync:

1. `package.json` → `"loadtest": "tsx loadtest/example.ts --room tourist --numClients 2"`.
2. Any hardcoded room string in `loadtest/` must match.
3. Keep loadtest `--room` in sync with `rooms` keys in `src/app.config.ts` and with `test/MyRoom.test.ts`.

Do not reintroduce `my_room`.

## Auth / JWT

`MyRoom.onAuth` requires a JWT. Unauthenticated `joinOrCreate` will fail once the room rejects missing/invalid tokens.

Attach a token the same way tests do — set SDK auth **before** join:

```ts
import { JWT } from "@colyseus/auth";

export async function main(options: Options) {
  const client = new Client(options.endpoint);

  // Realistic load: one token per virtual client (unique id avoids collisions if server keys by user).
  const token = await JWT.sign({
    id: options.clientId + 1,
    username: `loadtest-${options.clientId}`,
  });
  client.auth.token = token;

  const room = await client.joinOrCreate(options.roomName, {
    // join options if needed
  });
  // ...
}
```

Notes:

- `JWT.sign` needs the same `JWT_SECRET` (and related env) as the running server — load `.env.development` / process env the server uses.
- Prefer signing in-process for local loadtest; do not commit real production secrets into the script.
- Alternative: obtain a token via HTTP `/auth/*` (register/login/anonymous) and assign `client.auth.token` — use when testing the full auth HTTP path rather than only room join.

If `onAuth` is relaxed or bypassed in a branch, document that in the change; default product path expects JWT.

## Agent Workflow

1. Confirm room name in `src/app.config.ts` and keep `--room` aligned.
2. Edit `loadtest/example.ts` for the scenario (join only, send `move`, watch state, leave).
3. Ensure JWT is set when `onAuth` is active.
4. Run loadtest from the server package root:

   ```text
   npm run loadtest
   ```

   Fix failures before claiming done.

5. Optional CLI flags (pass through / document when useful):

   | Flag | Meaning |
   |------|---------|
   | `--endpoint` | WS URL (default `ws://localhost:2567`) |
   | `--room` | Room handler name |
   | `--roomId` | Join by id instead of `--room` |
   | `--numClients` | Parallel clients (script default via npm: `2`) |
   | `--delay` | Stagger client starts (ms) |
   | `--output` | Log file path |

   Example override:

   ```bash
   npm run loadtest -- --numClients 10 --endpoint ws://localhost:2567
   ```

   (`npm run loadtest` already embeds `--room` / `--numClients`; extra args after `--` append.)

## What To Change vs Leave Alone

| Do | Don't |
|----|-------|
| Extend `main` listeners / join options / auth | Drop `cli(main)` |
| Sync room name with `app.config` + tests | Invent a different room name only in loadtest |
| Keep scenarios authoritative-client-light (send intents, observe state) | Assert full board rules inside loadtest — that belongs in room + unit/integration tests |
| Run `npm run loadtest` from server package root | Silently assume loadtest passed without running |

## Checklist Before Running

- [ ] Server listening on the endpoint loadtest will use
- [ ] `--room` matches registered room name
- [ ] JWT attached if `onAuth` verifies tokens
- [ ] `numClients` sensible for room `maxClients` (intended tourist: 2 per match — many clients → many rooms via `joinOrCreate`)
