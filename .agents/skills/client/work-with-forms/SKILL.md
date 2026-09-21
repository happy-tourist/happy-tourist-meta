---
name: work-with-forms
description: >-
  Use when creating, changing, reviewing, or debugging Vue 3 forms in the
  happy-tourist client — Quasar q-form + q-input :rules, LoginPage register /
  login toggle, ForgotPasswordPage / ResetPasswordPage, SupportPage create
  ticket (topic + body), anonymous guest / Google one-click buttons, auth or
  support store submit, or q-banner errors.
---

# Work With Forms

Use this skill for forms in the Vue 3 Quasar **client** (`happy-tourist.github.io`).

This package validates with Quasar `q-form` + `q-input` `:rules` and `<script setup>` refs. Not Vuetify `v-form` / `vue-the-mask`.

## Overview

| Form / surface | Path | Role |
| --- | --- | --- |
| Login | `src/pages/LoginPage.vue` | Email/password register or login → `auth`; guest + Google via separate buttons |
| Forgot | `src/pages/ForgotPasswordPage.vue` | Email → `auth.forgotPassword`; `email_not_found` → RU not-found |
| Reset | `src/pages/ResetPasswordPage.vue` | New password (min 6) → `auth.resetPassword(token, password)`; public route |
| Support create | `src/pages/SupportPage.vue` | Topic `q-select` + body textarea → `support.createTicket`; rate-limit codes → `support.errors.*` |

Shared pieces:

| Concern | Location |
| --- | --- |
| Auth actions / `error` / `loading` | `src/stores/auth.ts` |
| Colyseus client | `src/boot/colyseus.ts` (`client.auth.*`) |
| Route after success | `route.query.redirect` or `/lobby` |

No captcha. No shared validation utils yet — rules are inline on `q-input`.

## Core Pattern (script setup + Quasar)

1. Field state in `ref()`.
2. Wrap controls in `<q-form @submit.prevent="…">`.
3. Attach `:rules` arrays of `(value) => true | errorString`.
4. Quasar runs rules on submit (and blur/change by default); no `$refs.form.validate()` needed for the login flow.
5. Call Pinia `auth` actions from the submit handler; catch and leave `auth.error` for the banner.

```vue
<script setup lang="ts">
import { ref } from 'vue';
import { useAuthStore } from '@/stores/auth';

const auth = useAuthStore();
const email = ref('');
const password = ref('');

async function onSubmit() {
  try {
    await auth.login(email.value, password.value);
    // navigate on success
  } catch {
    // error already in store
  }
}
</script>

<template>
  <q-form class="q-gutter-md" @submit.prevent="onSubmit">
    <q-input
      v-model="email"
      type="email"
      label="Email"
      outlined
      dense
      :rules="[(v) => !!v || 'Введите email']"
    />
    <q-input
      v-model="password"
      type="password"
      label="Пароль"
      outlined
      dense
      :rules="[(v) => (v && v.length >= 6) || 'Минимум 6 символов']"
    />
    <q-banner v-if="auth.error" dense class="bg-negative text-white">
      {{ auth.error }}
    </q-banner>
    <q-btn type="submit" color="primary" :loading="auth.loading" label="Войти" />
  </q-form>
</template>
```

## Rules

Quasar rules: `(val) => true | string`. String = invalid message.

| Field | Rule (LoginPage) |
| --- | --- |
| Email | `!!v` → `'Введите email'` |
| Password | `v && v.length >= 6` → `'Минимум 6 символов'` |
| Display name (register) | Optional — no `:rules` |

Prefer keeping rules next to the input (inline arrays) until a shared helper appears. Do not introduce Vuetify-style `$refs.validate()` or `vue-the-mask` unless product asks.

## Form Catalogue

### LoginPage

- `q-form` → `onSubmit`; primary submit is `q-btn type="submit"`.
- Toggle `isRegister`: same form for register vs login; clear `auth.error` in `toggleMode`.
- Register: optional `displayName` → `auth.register(email, password, displayName ? { name } : {})`.
- Login: `auth.login(email, password)`.
- Anonymous: **outside** the form (`q-card-actions`) → `auth.loginAnonymously(options)`; still uses `:loading="auth.loading"`. Does not go through `q-form` submit / email-password rules.
- Google one-click: **outside** the form (same `q-card-actions`) → `auth.loginWithGoogle()`; label via `$t('login.google')`; same loading/error/redirect pattern as guest.
- API failures: store sets `auth.error`; page shows `q-banner`. Handlers `catch` empty after store throw.
- Success: `router.replace(route.query.redirect || '/lobby')`.
- Password visibility toggle via `#append` `q-icon` — UI only.

## Submit And Errors

| Pattern | Usage |
| --- | --- |
| `@submit.prevent` on `q-form` | Email/password register & login |
| Separate `@click` outside form | Guest / anonymous / Google one-click |
| `auth.error` + `q-banner` | Colyseus / network failures |
| `auth.loading` on buttons | Disable double-submit UX |
| `try/catch` in page | Swallow after store already set `error` |

Keep I/O in `stores/auth` (`register` / `login` / `loginAnonymously` / `loginWithGoogle`). Pages call store actions; do not call `client.auth.*` from the form.

## New Form Checklist

1. `<script setup lang="ts">`; field state in `ref()`.
2. Wrap inputs in `<q-form @submit.prevent="…">`.
3. Put `:rules` on required `q-input`s (email required; password min 6).
4. Show `auth.error` (or the relevant store error) with `q-banner`.
5. Bind `:loading` on submit (and any parallel action buttons).
6. Auth → call Pinia `auth` actions; navigate only after success.
7. Actions that skip field validation (e.g. guest, Google) stay outside `q-form` submit.

## Do

- Match existing Quasar look: `outlined` + `dense` on login inputs.
- Clear store error when switching register ↔ login.
- Pass optional `{ name }` only when display name is non-empty.
- Keep anonymous login and Google one-click as buttons outside the form.

## Don't

- Add captcha, Vuetify `v-form`, or `vue-the-mask`.
- Call `client.auth.*` from the page — use `useAuthStore()`.
- Put guest login behind email/password `:rules`.
- Invent a second error channel when `auth.error` + `q-banner` already covers API failures.
- Lower password min below 6 without a product change.
