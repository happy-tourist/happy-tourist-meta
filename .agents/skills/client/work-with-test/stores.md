# Work With Stores Tests

Use with the [core test skill](SKILL.md) when the unit under test depends on a
Pinia store, or when the store itself is the SUT.

Product store conventions: meta skill `work-with-stores`. Domains: `auth`,
`theme`, `game`, `support`, `content` (HTTP packs via `client.http`; dual submit;
my-moderation; three-phase marks; cascadeGap* / `markCascadeGaps` /
`restoreCascadeGapsIfNeeded` / hasLive confirm; D1′/D5′ on open=pending|rejected;
author delete unpublished; staff pending|rejected + `tasksOnly`/`answersActionsAvailable`
/ approve-from-rejected (`not_approvable`) — spy `client.http`, do not hit live
server; cascade helpers: meta `work-with-stores/content.md`).

## Testing a page / component that uses a store

- Declare store method mocks as separate variables outside `vi.mock`.
- Prefer plain state values on a mutable `mockState` object; assign before
  mount.
- Use real Vue `ref(...)` only when the test needs reactive updates after mount.
- Never pass ref-like `{ value: … }` into a mocked store that the page reads as
  plain state — match how the page consumes the store (setup stores expose
  refs that unwrap in templates; prefer returning plain fields from the mock
  unless the SUT reads `.value` in script).

```typescript
const mockLogin = vi.fn();
const mockState = {
  error: null as string | null,
  loading: false,
};

vi.mock('@/stores/auth', () => ({
  useAuthStore: vi.fn(() => ({
    error: mockState.error,
    loading: mockState.loading,
    login: mockLogin,
  })),
}));
```

If `vi.mock` needs a variable before initialization, use `vi.hoisted()` (see
[SKILL.md](SKILL.md) → Hoisted Mocks).

Do **not** mock `pinia` / `storeToRefs` locally unless the global setup already
documents a different pattern — prefer module mock of `use*Store`.

## Testing the store itself

- Use a real Pinia instance: `createPinia()` + `setActivePinia(createPinia())`
  in `beforeEach`.
- Spy on Colyseus I/O (`client.auth`, `client.http`, room helpers) — do not
  hit a live server → [colyseus.md](colyseus.md).
- Assert public actions / returned state / `error` / loading flags →
  [errors.md](errors.md).
- Do not assert private `_` helpers of the options `game` store unless they are
  the only way to observe a contract (prefer public actions).

```typescript
import { createPinia, setActivePinia } from 'pinia';
import { useAuthStore } from '@/stores/auth';
import { client } from '@/boot/colyseus';

beforeEach(() => {
  setActivePinia(createPinia());
});

it('sets error when login fails', async () => {
  vi.spyOn(client.auth, 'signInWithEmailAndPassword').mockRejectedValueOnce(
    new Error('bad credentials'),
  );

  const auth = useAuthStore();
  await expect(auth.login('a@b.c', 'secret')).rejects.toThrow();
  expect(auth.error).toBeTruthy();
  expect(auth.loading).toBe(false);
});
```

(Adapt spy method names to the real `client.auth` / store implementation.)

## Checklist

- Page tests: store method mocks are separate variables outside `vi.mock`.
- Store tests: real Pinia + Colyseus spies; no live network.
- `pinia` is not randomly mocked for page tests when a `use*Store` mock is enough.

## Anti-Patterns

```typescript
// Store mocks buried only inside vi.mock with no outer handles
vi.mock('@/stores/auth', () => ({
  useAuthStore: vi.fn(() => ({ login: vi.fn() })),
}));

// Hitting a real Colyseus URL from a unit test
await useGameStore().createGame({ maxSeats: 2 });
```
