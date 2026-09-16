---
name: work-with-localization
description: >-
  Patterns for vue-i18n strings in the happy-tourist Quasar client (boot/i18n,
  src/i18n, useI18n / $t). Use when adding or changing translations or
  user-facing copy in pages/components.
---

# Work With Localization

Use this skill for **client** UI strings (`happy-tourist.github.io`).

Stack: **vue-i18n 11**, Composition API (`legacy: false`), default locale **`en-US`**.

There is **no** country config, phone masks, locale switcher, or brand localization in this app.

## Reality check

Boot and message catalog exist; Login / Lobby still mostly **hardcode Russian**. Scaffold keys (`failed` / `success`) are barely used. **Exceptions already on i18n:** `game.say.*` (preset labels / affordance — never put display copy in the wire `presetId`); `game.readyButton` / `game.countdownSoon`; leave UX `game.leave` (accessible name for icon-only Game exit — no visible `:label`) / `game.leaveConfirm` / `game.leaveCancel` / `game.leaveExit`; finish UX `game.finishPlaceModal` / `game.finishPlaceModalOk` / `game.finishStripAria` / `game.finishPlaceBadgeAria`; solo timeout UX `game.timeExpiredModal` / `game.timeExpiredModalOk`. Product copy lives under locale key **`en-US`** (Russian strings) — there is no separate `ru-RU` catalog.

When adding **new** user-facing strings, prefer i18n keys via `$t` / `useI18n`. Do not mass-migrate hardcoded Russian unless the user asks.

## Rules

| Do | Don't |
|----|--------|
| Put user-facing strings in `src/i18n/en-US/` and use `$t` / `useI18n` | Hardcode new shared UI copy in pages when adding fresh strings |
| Keep messages under the `en-US` locale tree (typed schema from that locale) | Invent a second `createI18n` or bypass the Quasar boot instance |
| Use Composition API (`useI18n`) in `<script setup>` | Assume Options API / `this.$t` patterns from Vue 2 apps |
| Mirror key shape if you add another locale later | Add country masks, brand locale maps, or a language switcher unless asked |

## Layout

| Path | Role |
|------|------|
| `src/boot/i18n.ts` | Quasar boot: `createI18n`, `locale: 'en-US'`, `legacy: false`, `app.use(i18n)` |
| `src/i18n/index.ts` | Locale map (`'en-US'` → messages) |
| `src/i18n/en-US/index.ts` | Message catalog (master schema for TypeScript) |
| `quasar.config.ts` → `boot` | Registers `'i18n'` (with `'colyseus'`) |

## Bootstrap

```ts
// src/boot/i18n.ts
const i18n = createI18n<{ message: MessageSchema }, MessageLanguages>({
  locale: 'en-US',
  legacy: false,
  messages,
});
app.use(i18n);
```

`MessageSchema` is derived from `messages['en-US']`. Do not create another i18n instance in pages or stores.

## Adding / changing strings

1. Open `src/i18n/en-US/index.ts`.
2. Add a key (flat or nested). Prefer clear namespaced keys, e.g. `lobby.title`, `game.yourTurn`.
3. In the page/component, use `$t` in the template or `useI18n()` in script.

**Catalog shape:**

```ts
// src/i18n/en-US/index.ts
export default {
  failed: 'Action failed',
  success: 'Action was successful',
  login: {
    google: 'Продолжить с Google', // LoginPage; catalog lives under en-US even when copy is RU
  },
  game: {
    say: {
      affordance: 'Сказать',
      hello: 'Всем привет',
      luck: 'Удачи',
      ready: 'Готов начать!',
    },
    readyButton: 'Готов начать',
    countdownSoon: 'Игра скоро начнётся',
    leave: 'Выход из игры',
    leaveConfirm: 'Вы уверены? Если выйдете, прогресс будет сброшен.',
    leaveCancel: 'Отмена',
    leaveExit: 'Выйти',
    finishPlaceModal: 'Вы {n}-й!',
    finishPlaceModalOk: 'ОК',
    timeExpiredModal: 'Вы не успели довести туристов до финиша вовремя.',
    timeExpiredModalOk: 'ОК',
    finishStripAria: 'Финиш',
    finishPlaceBadgeAria: 'Место {n}',
  },
  // lobby: { title: 'Lobby' },
};
```

**Template:**

```vue
{{ $t('failed') }}
```

**Composition API (`<script setup>`):**

```ts
import { useI18n } from 'vue-i18n';

const { t } = useI18n();
// t('failed')
```

Named placeholders use `{name}` in the message and a params object: `t('key', { name: value })`.

## Checklist

- [ ] New user-facing string lives in `src/i18n/en-US/`
- [ ] UI uses `$t` or `useI18n().t` (not a new hardcoded string for fresh copy)
- [ ] No second `createI18n` / no Options-API-only assumptions
- [ ] No country / phone / brand localization scaffolding unless requested
