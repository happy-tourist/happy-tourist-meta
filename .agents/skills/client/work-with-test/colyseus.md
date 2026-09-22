# Work With Colyseus / HTTP Tests

Use with the [core test skill](SKILL.md) when the unit under test calls
Colyseus SDK I/O: `client.auth`, `client.http`, room create/join/`send`, or
lobby subscribe helpers.

Product I/O conventions: meta skill `colyseus-client`. There is **no** axios /
OpenAPI client on this SPA.

## Where I/O lives

| Layer | Pattern |
|-------|---------|
| Pages / components | Call Pinia actions only — mock the **store**, not `client` |
| Stores (`auth` / `theme` / `game` / `support`) | Spy / mock `@/boot/colyseus` `client` |

Do not call `client.*` from page tests “because the store would”. Mock
`use*Store` at the page boundary → [stores.md](stores.md).

## Spying on `client`

Create `vi.spyOn` at module level before the first `describe` when testing a
store. Do not recreate spies in `beforeEach` — `vi.clearAllMocks()` in
`afterEach` resets call history. For one-off scenarios use
`mockRejectedValueOnce` / `mockImplementationOnce`.

```typescript
import { client } from '@/boot/colyseus';

const signInSpy = vi
  .spyOn(client.auth, 'signInWithEmailAndPassword')
  .mockResolvedValue({} as never);

signInSpy.mockRejectedValueOnce(new Error('API Error'));
```

If the boot module constructs a live `Client` at import time and breaks jsdom,
`vi.mock('@/boot/colyseus', () => ({ client: mockClient }))` with a hoisted
plain object that has the methods the store needs:

```typescript
const { mockClient } = vi.hoisted(() => ({
  mockClient: {
    auth: {
      signInWithEmailAndPassword: vi.fn(),
      registerWithEmailAndPassword: vi.fn(),
      // …only what the SUT calls
    },
    http: {
      get: vi.fn(),
      post: vi.fn(),
    },
  },
}));

vi.mock('@/boot/colyseus', () => ({
  client: mockClient,
}));
```

Prefer `vi.spyOn` on the real export when import is safe under the Vitest setup.

## What to verify

For every Colyseus / HTTP call the SUT owns:

- When it is called and with what parameters.
- Success and error scenarios.
- Loading / listing flags while pending and reset after resolve **and** reject.
- Related side effects: `error` string, navigation (pages), room state reset →
  [errors.md](errors.md).

Do not assert WebSocket wire frames or `@colyseus/schema` internals in client
unit tests — that belongs on the server (`server-work-with-test`). Client tests
assert store/page contracts (actions called, flags, mirrored state the store
already holds).

Plan examples:

```text
Verify via each success and error for auth.login:
- On success: client.auth sign-in called with email/password; error null; loading=false
- On error: error message set; loading=false; throw re-thrown for page navigation skip

Verify theme POST /api/theme on toggle for registered user:
- client.http.post called with body { theme }
- On error: theme.error set; Dark preference not left inconsistent with store contract
```

## Room messages

When testing `game` store send helpers (`sendMove`, `sendSay`, …), stub the
active room object the store holds (or inject a fake room with `send: vi.fn()`)
and assert `send` type + payload. Do not require a real `ColyseusTestServer`
inside the client package.

## Checklist

- Page tests mock stores; store tests spy/mock `client`.
- Spies created before the first `describe` (or hoisted mockClient).
- Success and error paths covered; loading resets after both.
- No live `VITE_COLYSEUS_URL` network in unit tests.
- No axios / fetch mocks for Colyseus paths.

## Anti-Patterns

```typescript
// Mocking axios — this app has no axios Colyseus layer
vi.mock('axios', () => ({ default: { get: vi.fn() } }));

// Recreating spies every beforeEach (loses module-level mockResolvedValue defaults)
beforeEach(() => {
  vi.spyOn(client.auth, 'signInWithEmailAndPassword').mockResolvedValue({} as never);
});

// Client unit test booting @colyseus/testing server
import { boot } from '@colyseus/testing';
```
