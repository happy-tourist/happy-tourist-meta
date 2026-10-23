## Context

See `proposal.md` — Why. Пакет: **client only** (`../happy-tourist.github.io` через `docs/projects-map.md`). Server / room / auth HTTP не затрагиваются.

Сейчас shared header в `src/App.vue` импортирует один `src/assets/brand/logo.png` (маскот). Рядом уже лежат сырые wordmark-файлы `logo-dark-removebg-preview.png` / `logo-light-removebg-preview.png` (прозрачный фон, ~12–15px padding). Wide toolbar: `header-logout`, `header-login`, `header-game-leave` с `:label`. Account: `src/pages/AccountPage.vue` — logout с `:label` без иконки. Theme уже доступен через Quasar Dark (`$q.dark.isActive`) и `useThemeStore`.

Блокеров implementability нет (стек Vue/Quasar, без новых npm-зависимостей для runtime).

## Goals / Non-Goals

**Goals:**

- Theme-aware wordmark в header; tight-crop ассеты с нормальными именами.
- Удалить маскот `logo.png` из brand folder.
- Icon-only: session login/logout в wide toolbar, Game leave справа, Account logout.
- Высота mark 28 CSS px с inset в content band шапки (~40 CSS px min-height).

**Non-Goals:**

- Менять burger menu copy (текстовые пункты остаются).
- Favicon / title / i18n keys (только reuse accessible names).
- Серверный контракт (room / schema / messages / HTTP / auth / DB).

## Server scope

**Нет.** Change не добавляет и не меняет Colyseus room lifecycle, synced schema, `onMessage`, HTTP routes, auth или DB. Delta `game/leave` — только chrome правой кнопки (icon-only + accessible name); confirmation / consented leave / `room.leave` / server `onLeave` остаются as-is из существующей capability. Theme wordmarks и session login/logout — чисто client assets/UI.

## Decisions

### D1 — Ассеты: crop + rename, без runtime crop

- **Решение:** одноразово обрезать оба PNG по bbox непрозрачных (не-near-black) пикселей; сохранить как `src/assets/brand/logo-dark.png` и `logo-light.png`; удалить `logo.png` и `*-removebg-preview.png`.
- **Почему:** имена без «removebg-preview»; меньший payload; стабильный aspect ~2:1.
- **Альтернатива:** CSS `object-fit` без crop — оставляет визуальный «воздух» внутри PNG.

Семантика: `logo-dark` = светлые буквы (для **тёмной** темы UI); `logo-light` = тёмные буквы (для **светлой**).

### D2 — Выбор URL по Quasar Dark

- **Решение:** в `App.vue` два static import; `computed` `brandLogoUrl` = `$q.dark.isActive ? logoDark : logoLight` (тот же сигнал, что у theme toggle icon).
- **Почему:** один источник правды с уже работающим theme chrome; без дублирования Pinia-only path.
- **Альтернатива:** только `useThemeStore` — лишний слой, пока Dark уже синхронизирован.

### D3 — Высота логотипа

- **Решение (измеримый контракт для FE / SC-BRAND-10 / SC-BRAND-30):**
  1. Сохранить `.brand-logo-control` vertical padding **2px** top+bottom — inset вокруг mark внутри шапки.
  2. CSS: `.brand-logo { height: 28px; width: auto; object-fit: contain; display: block }`.
  3. Toolbar / header content **min-height: 40px** (logo 28 + inset). Не раздувать toolbar выше этого без нужды.
  4. При узком viewport / клиппе section links — **не** опускать logo ниже 28px; допустимо сжатие соседних controls, не высоты mark.
- **Почему:** отступ от краёв шапки; vitest assert `height: 28px` + toolbar min-height ≥ 40.
- **Альтернатива:** 60/64 — слишком плотно к краям.

### D4 — Icon-only кнопки

- **Решение:**
  - `App.vue` wide toolbar: убрать `:label` у `header-logout`, `header-login`, `header-game-leave`; оставить Material `icon` + `:aria-label`; выставить `flat round dense` (parity с Acc/Theme справа).
  - Accessible names — **reuse существующих i18n keys** (новых строк не добавлять):
    - `header-login` → `auth.login` («Войти»)
    - `header-logout` → `auth.logout` («Выйти»)
    - `header-game-leave` → `game.leave` («Выход из игры»)
  - Burger menu: текстовые пункты login/logout **as-is** (out of scope).
  - `AccountPage.vue` session logout: `icon="logout"` + `:aria-label="$t('auth.logout')"` без `:label`; сохранить `flat` + `color="negative"`; добавить `data-test-id="account-logout"` для SC-BRAND-32.
- **Почему:** единый chrome; accessible name сохраняется; burger пункты не трогаем.
- **Альтернатива:** скрывать label только на xs — отвергнуто («везде»).

### D5 — Тесты

- Точки: `src/__tests__/AppHeaderChrome.test.ts` (header theme mark / height / icon-only login|logout|leave); **новый** узкий vitest для Account logout (отдельного `AccountPage` test сейчас нет) — SC-BRAND-32, селектор `[data-test-id="account-logout"]`.
- Добавить/обновить: SC-BRAND-29/30/31/32, SC-LEAVE-13; ослабить/убрать ожидания видимого label у toolbar кнопок (`:label` / text рядом с icon).
- **SC-BRAND-29:** после mount переключить `$q.dark` (или theme store → Dark) и проверить, что `img.brand-logo` `src` меняется между light/dark imports (не сравнивать absolute URL строк жёстко — достаточно двух разных `src` или spy на computed).
- **SC-BRAND-30:** assert в CSS/style source или computed: `.brand-logo` height `28px` и toolbar/header content min-height ≥ `40px` (D3).
- **SC-BRAND-31 / SC-LEAVE-13:** у `header-login` / `header-logout` / `header-game-leave` есть icon + aria-label (`auth.login` / `auth.logout` / `game.leave`); нет visible text label (нет `:label` / пустой text content кнопки).
- **SC-BRAND-32:** Account `[data-test-id="account-logout"]` — icon + aria-label `auth.logout`, без `:label` / видимого «Выйти».

Чеклист реализации — `tasks.md`. После UI/tests — синхронизировать client skills / AGENTS blurbs (logo paths, 28px, icon-only session/leave) — блок **5** в `tasks.md`.

## Risks / Trade-offs

- **[Risk]** Широкий wordmark (~2:1) на узком viewport жмёт section links → **Mitigation:** высота mark остаётся **28px** (не ниже); при клиппе сжимать соседние section/cluster controls, не высоту logo (D3).
- **[Risk]** Account icon-only менее discoverable → **Mitigation:** aria-label + знакомая Material `logout`; при необходимости позже tooltip (out of scope).
- **[Risk]** SC-BRAND-25 формулировка «без wordmark» путала маскот vs PNG → **Mitigation:** delta уточняет «без отдельного DOM-label».

## Migration Plan

- Client-only UI/assets; откат = revert commit assets + App/Account.
- Нет DB/API миграций.
