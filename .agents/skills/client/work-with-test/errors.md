# Work With Errors Tests

Use with the [core test skill](SKILL.md) when asserting failure UX: Pinia
`error` strings, `loading` / `listing` flags, and `q-banner` display.

Product conventions: meta skill `client-work-with-errors`. There is **no**
`useNotice` toast pipeline and **no** global dialog error bus — do not invent
Notify assertions.

## What to assert

| Layer | Assert |
|-------|--------|
| Store action failure | `error` set to message (or mapped `support.errors.*` key); flag cleared in `finally` |
| Store action success | `error` null / cleared at start; no stale error left |
| Page | `q-banner` (or `data-test-id` wrapper) visible when store `error` is set; submit/navigation skipped on rethrow |

Configure store mocks so `error` / `loading` are readable after the action →
[stores.md](stores.md). For store SUT tests, drive real reject from Colyseus
spies → [colyseus.md](colyseus.md).

```typescript
it('shows banner when auth.error is set', async () => {
  mockState.error = 'Invalid credentials';
  wrapper = getWrapper();
  await flushPromises();

  expect(wrapper!.find('[data-test-id="auth-error"]').exists()).toBe(true);
});

it('sets error and clears loading on login failure', async () => {
  signInSpy.mockRejectedValueOnce(new Error('nope'));
  const auth = useAuthStore();

  await expect(auth.login('a@b.c', 'x')).rejects.toThrow();
  expect(auth.error).toBe('nope');
  expect(auth.loading).toBe(false);
});
```

Add `data-test-id` on the banner (or its wrapper) in the SUT when asserting
visibility — do not find by `.bg-negative` alone.

## What not to assert

- Quasar `Notify.create` unless production code actually calls it (it should
  not for auth/game/support errors).
- Duplicate page-level `catch` message copies when the store already owns
  `error`.

## Checklist

- Success path: success notification/toasts are **not** required; assert
  `error` cleared / navigation as the product does.
- Error path: `error` set; loading/listing false; success side effects not
  called.
- Banner assertions use `data-test-id`, not layout color classes alone.

## Anti-Patterns

```typescript
// Inventing useNotice / Notify for this client
expect(Notify.create).toHaveBeenCalled();

// Finding the error only by Quasar color utility
expect(wrapper.find('.bg-negative').exists()).toBe(true);
```
