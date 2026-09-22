# Work With Plugins Tests

Use with the [core test skill](SKILL.md) when tests need vue-i18n, vue-router,
or Quasar globals/stubs.

## i18n (`$t` / `useI18n`)

Handle i18n in the **global** Vitest setup (pass-through that returns the key,
or a minimal `createI18n` with the locales under `src/i18n`). Do **not** add
per-file `global.mocks.$t = vi.fn()` or `i18n.global.t = vi.fn(...)` unless the
setup is missing and you are fixing setup in the same change.

Assert either:

- the **message key** (if setup returns keys), or
- the **fixture string** from the real locale object you loaded in setup.

Product copy conventions: `work-with-localization`.

## vue-router

When the SUT calls `useRouter().push` / reads `useRoute().query`:

- Prefer mocking `vue-router` composables with `vi.mock('vue-router', …)` and
  hoisted `push` / `replace` / `query` fixtures, **or**
- Pass `global.mocks` / a lightweight plugin if the project setup already does.

```typescript
const { mockPush } = vi.hoisted(() => ({
  mockPush: vi.fn(),
}));

vi.mock('vue-router', async (importOriginal) => {
  const actual = await importOriginal<typeof import('vue-router')>();
  return {
    ...actual,
    useRouter: () => ({ push: mockPush }),
    useRoute: () => ({ query: {}, params: {}, path: '/' }),
  };
});
```

Prefer asserting `mockPush` args over full router installation for unit tests.
Hash-mode details matter for e2e, not for most unit assertions.

## Quasar

- `shallowMount` auto-stubs child Quasar components — usually enough.
- For `mount` + real `q-form` / `q-input`, either install Quasar in setup
  (`app.use(Quasar, { … })` / official Vitest AE) or stub only unrelated heavy
  widgets → [forms.md](forms.md).
- Do not assert Quasar-internal class soup (`.q-btn__content`) as behavior.

## Dark / theme

Theme preference lives in `stores/theme` + Quasar `Dark`. Unit-test via the
store and HTTP spies ([colyseus.md](colyseus.md)); do not require a full Quasar
boot file (`src/boot/theme.ts`) unless the SUT imports it directly.

## Checklist

- `$t` / `useI18n` come from global setup; no ad-hoc per-file i18n mocks.
- Router navigation asserted via mocked `push` / query fixtures.
- Quasar: prefer shallow stubs; real install only when validation mount needs it.

## Anti-Patterns

```typescript
// Per-file $t mock while global setup already provides i18n
const getWrapper = () =>
  shallowMount(Page, {
    global: { mocks: { $t: (k: string) => k } },
  });

// Asserting Quasar internal DOM structure as the contract
expect(wrapper.find('.q-btn__content').text()).toBe('OK');
```
