# Work With Forms Tests

Use with the [core test skill](SKILL.md) when testing Quasar `q-form` /
`q-input` `:rules` and form field behavior in the happy-tourist client.

Product form conventions: meta skill `work-with-forms` (Login / Forgot / Reset /
Account / Support). This file is **only** how to test them.

## Never fake Quasar validation state

Real `:rules` must run when the assertion is about validity / submit gating.
Do **not** invent a `$invalid` mock or stub `q-form` validate to always pass
unless the test explicitly does **not** care about validation (API-only /
navigation-only with stubbed inputs).

Drive validity by changing real `v-model` values (or emitting
`update:modelValue` on the field stub), then assert UI (`disable`, banner,
submit call).

## How to test validation

Pick **one** pattern.

### A. Field / meter block that does not own submit

Typical: `PasswordStrengthMeter`, a field fragment.

1. `mount` (not `shallowMount`) the fragment when rules / meter must render.
2. Change `password` / field props via `setProps` or real input events.
3. Assert visible strength / helper text / classes by `data-test-id`.

### B. Page that owns `q-form` + submit

Typical: `LoginPage`, `ForgotPasswordPage`, `ResetPasswordPage`, `AccountPage`,
`SupportPage`.

1. Prefer `shallowMount` + mocked store for cases that do **not** need real
   rules (click paths with stubbed children).
2. For submit gating tied to `:rules` / password policy, use `mount(Page)` so
   Quasar inputs register. Stub only heavy unrelated children if needed.
3. Fill fields to valid/invalid; trigger submit on the form or button by
   `data-test-id`.
4. Assert store action called / not called; `loading` / `error` via store mock
   → [stores.md](stores.md), [errors.md](errors.md).

```typescript
const getWrapper = () => shallowMount(LoginPage);

const getValidationWrapper = () =>
  mount(LoginPage, {
    global: {
      stubs: {
        // optional: stub unrelated heavy children only
      },
    },
  });

it('does not call login when email is empty', async () => {
  wrapper = getValidationWrapper();
  await flushPromises();

  await wrapper!.find('[data-test-id="login-submit"]').trigger('submit');
  await flushPromises();

  expect(mockLogin).not.toHaveBeenCalled();
});
```

## Password policy

Shared gate: `src/lib/passwordPolicy.ts`. Prefer **unit tests of the lib**
(direct calls) for complexity rules; page tests assert wiring (submit blocked /
allowed) at the UI boundary → [composables.md](composables.md).

Do not re-implement policy strings in the test; import helpers from the lib.

## Checklist

- Quasar validation is not faked with a hand-built `$invalid`.
- Validation uses `mount` when rules must run; `shallowMount` when they need not.
- Model / field values are driven through real props or `v-model` updates.
- Password policy unit coverage prefers `src/lib/passwordPolicy` tests.

## Anti-Patterns

```typescript
// Faking Quasar form validity — FORBIDDEN for validation assertions
vi.mock('quasar', () => ({
  /* … fake QForm validate always true … */
}));

// Asserting only CSS layout of inputs with no behavior
expect(wrapper.find('.q-field').classes()).toContain('q-field--outlined');
```
