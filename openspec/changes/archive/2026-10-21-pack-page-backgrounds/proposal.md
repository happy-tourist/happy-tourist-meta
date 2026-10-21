## Why

Страницы клиента без атмосферного фона. Есть четыре JPG (desktop/mobile × light/dark); их нужно показать на **всех** экранах приложения с фиксированным фоном и полупрозрачной подложкой под сетками карточек (shared pack card wrapping grid), чтобы контент оставался читаемым.

## What Changes

- Единый page-фон на всём client shell (login, lobby, packs, maps, support, account, game и остальные экраны с shared layout).
- Выбор ассета по viewport (mobile / desktop) и теме (light / dark).
- Фон зафиксирован при скролле.
- Полупрозрачная подложка под блоками карточек на shared card grids (packs, maps, lobby rooms).

## Scope

- **Capability IDs:** `ui/branding` (app-wide page background), `content/packs` (product sense подложки на pack card grids; CSS shared с lobby/maps grids)
- **Пакет:** client only (`happy-tourist.github.io`)
- Ассеты: четыре JPG (desktop/mobile × light/dark) уже добавлены; пути — design D1.

## Out of scope

- Server / HTTP / room / schema.
- Отдельные арты по типу страницы.
- Смена breakpoint бургера header (`lt.md`).
- Новый theme-engine или зависимости для медиа.
- Отдельные delta для `content/maps` / `lobby/rooms` — underlay наследует через shared pack card grid chrome (см. `content/packs`).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `ui/branding`: app-wide fixed theme×viewport page background.
- `content/packs`: translucent underlay behind pack-related card grids (shared grid chrome).

## Impact

- Client: `App.vue` (`app-page-bg` на `q-layout`), `src/css/app.scss` (shell bg + `.pack-card-grid` underlay), Vitest (`AppHeaderChrome` + app.scss source-match); Quasar Dark × `sm` viewport; shared pack card wrapping grid на packs/maps/lobby (и прочих хостах `.pack-card-grid`).
- **Server / BE:** нет. Контракт room / schema / messages / HTTP / auth / DB не меняется; новых эндпоинтов и ассетов на server нет (JPG — static client `src/assets/packs/`).

## References

- Client: `happy-tourist.github.io/AGENTS.md`, skills `work-with-styles`, `work-with-pages`.
- Main specs: `openspec/specs/ui/branding/spec.md`, `openspec/specs/content/packs/spec.md`.
- Related: `openspec/specs/ui/theme/spec.md` (SC-THEME-07 — page background under board holes).
