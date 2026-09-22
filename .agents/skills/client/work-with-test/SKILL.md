---
name: work-with-test
description: >-
  Use when planning or writing Vue 3 / Quasar unit tests for the happy-tourist
  client (pages, components, Pinia stores, lib helpers): Vitest +
  @vue/test-utils, q-form rules, auth/theme/game/support stores, Colyseus
  client.auth / client.http / room I/O spies, store error + q-banner, vue-i18n /
  router. Core workflow in SKILL.md; topic details in forms.md, stores.md,
  colyseus.md, errors.md, plugins.md, composables.md, provide-inject.md.
trigger: slash
---

# Work With Test

Use this skill to plan and write Vue 3 unit tests for the **happy-tourist
client** (`happy-tourist.github.io`). Start with a test plan, then implement
tests from that plan.

**Paths:** this skill lives in **happy-tourist-meta** at
`.agents/skills/client/work-with-test/`. Runtime `src/…` paths are relative to
the **happy-tourist.github.io** sibling root (`../happy-tourist.github.io` from
meta). Sibling server tests: `server-work-with-test` (mocha + `@colyseus/testing`)
— do **not** mix stacks.

Stack: **Vitest**, **@vue/test-utils**, **Vue 3**, **Quasar 2**, **Pinia 4**,
**vue-i18n**, **vue-router** (hash), **`@colyseus/sdk`** via `@/boot/colyseus`.
Do not invent Jest, Cypress, Playwright, or mocha for this package.

Core principle: test user-visible behavior and component/store contracts, not
implementation details. Do not test unconditional rendering, static attributes,
or direct prop pass-through unless there is conditional logic, transformation,
user-visible behavior, or an event depending on it.

## Specialized Topics

Read the matching file in this folder when the unit under test involves that
area (do not load every file at once):

| Topic | File |
|-------|------|
| Form validation / Quasar `q-form` + `:rules` | [forms.md](forms.md) |
| Pinia stores (`auth` / `theme` / `game` / `support`) | [stores.md](stores.md) |
| Colyseus `client.auth` / `client.http` / room I/O | [colyseus.md](colyseus.md) |
| Store `error` + `q-banner` / loading flags | [errors.md](errors.md) |
| i18n (`$t` / `useI18n`), router, Quasar stubs | [plugins.md](plugins.md) |
| Lib helpers / pure functions (`passwordPolicy`, …) | [composables.md](composables.md) |
| provide / inject | [provide-inject.md](provide-inject.md) |

## Bootstrap (first tests in the repo)

The client currently has **no** Vitest harness. When adding the **first** test
file, set up the runner in the client package root before writing assertions:

1. Dev deps: `vitest`, `@vue/test-utils`, `jsdom`, `@vitejs/plugin-vue` (and
   types if needed). Prefer Vitest matching the Vite major used by
   `@quasar/app-vite`.
2. Add scripts: `"test": "vitest run"`, `"test:watch": "vitest"`.
3. Add `vitest.config.ts` with `environment: 'jsdom'`, Vue plugin, alias
   `@` → `src`, and `setupFiles: ['./test/setup.ts']` (or
   `src/test-utils/setup.ts`).
4. Global setup: create Pinia once if tests mount with a real store; stub
   `vue-i18n` so `$t` / `useI18n` return the key (or a stable fixture); do
   **not** boot a live Colyseus client — mock `@/boot/colyseus` →
   [colyseus.md](colyseus.md), [plugins.md](plugins.md).
5. Optional small helper `wait` / use `flushPromises` + `nextTick` after mount,
   events, and async work (see Component Mounting).

Do **not** add Jest, mocha, babel-jest, or Cypress for unit tests. Server stays
on mocha; client stays on Vitest.

Agent **runs** `npm test` from the client package root after writing or changing
tests; fix failures before claiming done. Also run `npm run typecheck` /
`lint:check` when the change touches production code.

## Workflow

1. Read the page / component / store / lib under test: props, emits, store
   actions, Colyseus I/O, `q-form` rules, conditional rendering, async flags.
2. Produce a test plan with two sections: **What needs to be mocked** and
   **What to verify**. For pure lib helpers, prefer direct calls →
   [composables.md](composables.md).
3. Place the test file next to the SUT: `Feature/__tests__/Name.test.ts` or
   `src/lib/__tests__/passwordPolicy.test.ts`. Prefer `__tests__` colocated
   with the module.
4. Write tests following the conventions below and specialized topics.
5. Cover planned success, error, loading, event, and conditional scenarios.
6. **Required props check (components):** after writing the file, re-read
   `defineProps` and confirm every required prop is in `getWrapper()` mount
   options.

## Test Plan

### What Needs To Be Mocked

List every external dependency that must be controlled:

- Stores: `useAuthStore` / `useThemeStore` / `useGameStore` / `useSupportStore`
  → [stores.md](stores.md). Prefer mocking the store module when testing a
  **page**; prefer real Pinia + spy on `client` when testing the **store**
  itself → [colyseus.md](colyseus.md).
- Colyseus: `client.auth.*`, `client.http.*`, room `send` / join helpers —
  `vi.spyOn` on the real `@/boot/colyseus` export or a module mock of that
  boot file → [colyseus.md](colyseus.md).
- Router: `useRouter` / `useRoute` when asserting navigation →
  [plugins.md](plugins.md).
- Pure lib used as SUT: do **not** mock it; call it directly.

Do **not** mock Quasar `q-form` validation by faking `$invalid` — drive real
`:rules` / field values → [forms.md](forms.md).

- `useI18n` / `$t` — handle in global Vitest setup; expect keys (or setup
  fixture strings). Do not add per-file `global.mocks.$t` unless the setup is
  missing → [plugins.md](plugins.md).

### What To Verify

Use these categories only when the SUT has relevant behavior:

- Conditional rendering: via `it.each`, element by `data-test-id` exists or not.
- Text / i18n: visible key or translated fixture by condition.
- Child props: only when conditional or transformed — never static pass-through.
- Store / Colyseus calls: when called, with what args, success vs error →
  [colyseus.md](colyseus.md), [stores.md](stores.md).
- Buttons / attributes: conditional `disable` / `loading` by `data-test-id`.
- Events: `wrapper.emitted()`.
- Errors: store `error` set on failure; success path clears / does not leave
  stale error; `q-banner` when mounted → [errors.md](errors.md).
- Loading: flags reset in `finally` after both resolve and reject.

Formulation rules:

- Use “via each” for multiple scenarios.
- Always specify `data-test-id` for HTML elements and the component name for
  Vue children.
- Always name the condition under test.
- For every Colyseus / HTTP call, cover success and error when the SUT branches.
- Do not plan assertions for static, unconditional props, attributes, or
  rendering.

## Do Not Test Static Child Props / Attributes

**Do not assert props or attributes on child components when those values are
fixed in the template and do not depend on state, props, computed logic, or
user actions.**

Typical unnecessary tests:

- Hardcoded Quasar layout attrs (`outlined`, `dense`, `flat`, `color="primary"`).
- Static labels / icons passed unchanged.
- `expect(findComponent(Child).props('foo')).toBe('bar')` when `foo` never
  changes by scenario.

Write child prop/attribute assertions only when the value **changes by
condition** — loading, disabled by invalid form, store flag, transformed
payload, or shown/hidden branch.

```typescript
// Bad: static attrs never change
expect(wrapper.findComponent({ name: 'QBtn' }).props('color')).toBe('primary');

// OK: disable depends on loading / validation
expect(wrapper.find('[data-test-id="submit-btn"]').attributes('disabled')).toBeDefined();
```

Examples:

```text
Verify auth.login call on submit by test-id "login-submit" with (email, password)

Verify via each success and error scenarios:
- On success: navigate to lobby (or redirect query); auth.error is null; loading=false
- On error: auth.error set; success navigation NOT called; loading=false

Verify via each disabled/loading of submit by test-id "login-submit":
- loading when auth.loading=true
- enabled when form valid and not loading
```

## Test File Structure

```typescript
import { shallowMount } from '@vue/test-utils';
import { afterEach, beforeEach, describe, expect, it, vi } from 'vitest';
import { flushPromises } from '@vue/test-utils';
import LoginPage from '@/pages/LoginPage.vue';

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

describe('LoginPage', () => {
  let wrapper: ReturnType<typeof shallowMount> | null = null;

  afterEach(() => {
    wrapper?.unmount();
    wrapper = null;
    vi.clearAllMocks();
  });

  const getWrapper = () =>
    shallowMount(LoginPage, {
      global: {
        stubs: {
          // stub heavy Quasar / router children when needed
        },
      },
    });

  it('calls login on submit with email and password', async () => {
    mockLogin.mockResolvedValue(undefined);
    wrapper = getWrapper();
    await flushPromises();

    // drive fields + submit via data-test-id (add ids in SUT when missing)
    await wrapper!.find('[data-test-id="login-submit"]').trigger('submit');
    await flushPromises();

    expect(mockLogin).toHaveBeenCalled();
  });
});
```

## Component Mounting

- Prefer `shallowMount` for pages/components under test.
- Always create `getWrapper()` with **no parameters**. Never call
  `shallowMount()` directly inside tests. Never pass props as arguments to
  `getWrapper(...)` — put mount options inside the factory, then
  `setProps()` for scenario overrides.
- Form validation that needs real Quasar rules: use `mount` (not
  `shallowMount`) so `q-input` / `q-form` participate → [forms.md](forms.md).
- Place `__tests__` next to the SUT.
- Leave `global.stubs` empty unless a named slot or heavy child requires a
  stub; prefer auto-stubs from `shallowMount`.
- After mount, events, prop changes, and async work: `await flushPromises()`
  and/or `await nextTick()`. Prefer a shared `wait()` helper only if the setup
  already provides one.
- Declare `let wrapper … = null` at `describe` scope; unmount in `afterEach`.
  Never `const wrapper` inside a test with a local unmount.

## Required Props

Required props belong in `getWrapper()` mount options; optional/scenario props
use `setProps()` after mount.

Before finishing any component test file:

1. List every prop without `?` / `default` in `defineProps`.
2. Confirm each appears in `getWrapper()` `props`.
3. Use `vi.fn()` for required callbacks; smallest valid value for data props.
4. Use `setProps()` only for optional overrides.

```typescript
const getWrapper = () =>
  shallowMount(PasswordStrengthMeter, {
    props: {
      password: '', // required
    },
  });

it('updates for a scenario', async () => {
  wrapper = getWrapper();
  await wrapper.setProps({ password: 'Abcd1234!' });
  await flushPromises();
});
```

## Finding Elements

Priority order:

1. `data-test-id` for HTML elements (add in SUT when asserting behavior).
2. `findComponent()` for Vue / Quasar components.
3. Do not find HTML elements by layout/style classes (`.q-pa-md`, flex utilities).

When several instances of the same child exist, give each a `data-test-id`;
do not rely on `findAllComponents(…).at(index)` for behavior assertions.

## Prop Assertions

- Use `toStrictEqual` / `toEqual` for object/array props across stubs/reactive
  boundaries.
- Use `toBe` for primitives, function references, or intentional identity.

## Events

- Verify via `wrapper.emitted()`, not external listener variables.

```typescript
expect(wrapper.emitted('update:modelValue')?.[0]).toEqual(['x']);
```

## Hoisted Mocks

If `vi.mock` needs a variable before initialization, use `vi.hoisted()`.

```typescript
const { mockLogin } = vi.hoisted(() => ({
  mockLogin: vi.fn(),
}));

vi.mock('@/stores/auth', () => ({
  useAuthStore: vi.fn(() => ({ login: mockLogin })),
}));
```

When substituting one export from a module, prefer `vi.spyOn` on the real
module → [composables.md](composables.md), [colyseus.md](colyseus.md).

## Checklist Before Finishing

- Bootstrap exists (or this change adds it) — `npm test` runs from client root.
- Required props audit passed for components.
- All wrappers via `getWrapper()`; `afterEach` unmounts + `vi.clearAllMocks()`
  (never `vi.restoreAllMocks()` unless a specific spy needs full restore and
  the file documents why).
- Prefer `shallowMount`; `mount` only for real `q-form` validation cases.
- Vue components not mocked with `vi.mock`; no handwritten object stubs with
  fake `template`/`props` unless an existing pattern requires it.
- Colyseus / store / errors / forms / plugins topics followed when relevant.
- `flushPromises` / `nextTick` after mount, events, async.
- HTML by `data-test-id`; similar scenarios use `it.each`.
- Events via `wrapper.emitted()`.
- Success and error paths for every Colyseus/HTTP call the SUT owns; loading
  resets after both.
- No static unconditional child prop / attribute assertions.
- Ran `npm test` from the client package root; failures fixed.

## Anti-Patterns

```typescript
// Props to getWrapper from tests
wrapper = getWrapper({ items: mockItems });

// Required prop only via setProps after mount
const getWrapper = () => shallowMount(Comp);
wrapper = getWrapper();
await wrapper.setProps({ requiredCb: vi.fn() });

// Direct shallowMount in a test
wrapper = shallowMount(Comp, { props: { x: 1 } });

// const wrapper inside a test + per-test unmount
const wrapper = getWrapper();
wrapper.unmount();

// Mocking a Vue SFC module; shallowMount already stubs children
vi.mock('@/components/PasswordStrengthMeter.vue', () => ({
  default: { template: '<div />' },
}));

// Handwritten component stub
const QBtnStub = {
  name: 'QBtn',
  props: ['loading'],
  template: '<button />',
};

// External variables for events
let value = '';
wrapper = getWrapper();
// … onUpdate:modelValue: (v) => { value = v }

// Finding HTML by layout/style classes
expect(wrapper.find('.q-pa-md').classes()).toContain('loading');

// Implementation details
expect(wrapper.vm.internalHelper).toHaveBeenCalled();

// Unconditional static prop pass-through
expect(wrapper.getComponent(Child).props('outlined')).toBe(true);

// Jest / mocha APIs on the client
import { describe, it } from 'mocha';
expect(x).toBeTruthy(); // without vitest expect — wrong runner
```

Topic-specific anti-patterns live in the specialized topic files above.
