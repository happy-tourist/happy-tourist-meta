## Context

См. proposal.md — Why. Четыре JPG уже лежат в `src/assets/packs/`. Scope расширен: фон на **весь** client shell, не только pack pages.

**Пакет:** client only (`../happy-tourist.github.io`).

### Server / BE

**Нет работы на server.** Change не вводит и не меняет:

- room name / lifecycle
- schema / synced / private payloads
- room messages
- HTTP routes
- auth / roles / onAuth
- DB tables / fields

Атмосферные JPG и underlay — только client CSS + static assets. Совместимость с `ui/theme` SC-THEME-07 (page background under board holes) — client render; server по-прежнему только помечает removed cells в sync state.

## Goals / Non-Goals

**Goals:**

- Один fixed cover-фон на layout/shell для всех маршрутов.
- Выбор ассета по теме × viewport (`sm` ~600px; Quasar Dark).
- Полупрозрачная подложка на `.pack-card-grid` (packs + maps + lobby).

**Non-Goals:**

- Менять burger breakpoint `md`.
- Server / новые зависимости.
- Отдельные арты per-route.

## Decisions

### D1 — Имена ассетов (без изменений)

```
src/assets/packs/
  pack-bg-desktop-light.jpg
  pack-bg-desktop-dark.jpg
  pack-bg-mobile-light.jpg
  pack-bg-mobile-dark.jpg
```

### D2 — Viewport = Quasar `sm` (~600px)

Mobile ниже 600px; desktop от 600px. Независимо от header burger `md`.

### D3 — Тема из `body.body--dark`

### D4 — Вешать на App `q-layout` (все маршруты)

Класс `app-page-bg` на `q-layout` в `App.vue`. CSS в `app.scss`:

- четыре `url(...)` относительными путями из `src/css/` → `../assets/packs/…` (Vite/Quasar переписывает; при необходимости кавычки в путях)
- `background-size: cover`; `center`; `fixed`; `no-repeat`
- light/dark через `body.body--dark` + `@media (max-width: 599px)` (Quasar sm − 1)
- Под `.app-page-bg` принудительно прозрачная заливка Quasar page canvas, чтобы арт просвечивал: `.q-page`, `.q-page-container` если он непрозрачен, и shell chrome на `--q-page-bg` (сейчас: `.app-breadcrumbs` в `App.vue`)

**Почему на layout:** одна точка врезки на все страницы; без списка классов per-page.

**Альтернативы:** класс на каждой странице — отвергнуто после расширения scope на все экраны.

### D5 — Underlay = background только на `.pack-card-grid`

Подложку красить на **layout**-блоке `.pack-card-grid` (он уже отделён от shared token selector `.pack-card-grid, .peek-chrome`). **Не** красить underlay на `.peek-chrome`.

```scss
.pack-card-grid {
  /* existing: display flex; flex-wrap; gap: 0.75rem */
  background: rgba(255, 255, 255, 0.78);
  border-radius: 8px; /* same order as peek-slot chrome */
  padding: 0.75rem; /* match existing grid gap rhythm */
}
body.body--dark .pack-card-grid {
  background: rgba(18, 18, 18, 0.78);
}
```

Плитки сохраняют `--pack-card-bg` / shared pack-card resting tokens.

### D6 — Game включён

Пользователь просил все страницы; Game получает тот же shell-фон. Заливки клеток доски / непрозрачный HUD — как сейчас. Отверстия доски показывают текущий page background (= атмосферный shell art после D4) — совместимо с `ui/theme` SC-THEME-07; server по-прежнему только помечает removed cells.

### D7 — Тесты

Vitest: source-match `app.scss` / `App.vue` на `app-page-bg`, `background-attachment: fixed`, media/dark urls, underlay rgba на `.pack-card-grid`; ассеты существуют.

**Якоря тестов (reuse):** класс на layout — расширить `src/__tests__/AppHeaderChrome.test.ts` (уже монтирует `App.vue`); CSS/ассеты — source-match по образцу `PackListCardTile.test.ts` / `LobbyCreateWire.test.ts` (`readFileSync` `app.scss`, class на grid). Новые сценарии помечать `SC-BRAND-21` / `SC-PACK-285`.

### D8 — Client anchors (as implemented)

- JPG D1: `src/assets/packs/pack-bg-{desktop|mobile}-{light|dark}.jpg`
- Shell: `App.vue` → `<q-layout class="app-page-bg">`; под `.app-page-bg` прозрачны `.q-page` / `.q-page-container` / `.app-breadcrumbs` (и `--q-page-bg`)
- CSS: `src/css/app.scss` — `.app-page-bg` (D4) + shared tokens на `.pack-card-grid, .peek-chrome`; underlay (D5) только на layout-блоке `.pack-card-grid`
- Хосты `.pack-card-grid`: `LobbyPage`, `ContentCatalogPage`, `MapsListPage`, `ContentPackPage`, `ContentPackEditorPage`, `ContentPackTasksPage`, `ContentPackAddTaskSetPage`, `ContentStaffRequestPage` (+ любые будущие с тем же классом)
- Header / toolbar / opaque board tile fills / HUD — не в scope transparent canvas
- Tests: `AppHeaderChrome.test.ts` — SC-BRAND-21 / SC-PACK-285

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Глюки `fixed` на iOS | Оставить как в постановке; mobile-only scroll fallback только если QA упадёт |
| Непрозрачный Quasar page скрывает арт | Принудительно transparent `.q-page` под `.app-page-bg` |
| Контраст login-карточки | Underlay только на grids; login — Quasar card; подкрутить при apply при необходимости |

## Technical prerequisites

JPG на месте (готово). Блокеров нет.

## Migration Plan

Client-only CSS + класс на App. Rollback: снять класс + CSS-правила.

## Open Questions

None.
