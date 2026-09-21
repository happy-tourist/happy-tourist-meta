## Context

См. proposal.md — Why. Первая реализация: shared header в `App.vue` с logo слева; на Game — leave/confirm; Account/Support без page «В лобби»; title/favicon продукта. Follow-up: `v-if` button vs bare `img` давал скачок позиции; auth был decorative; высота ~30px мало для прямоугольного `logo.png` (~741×337). Server не затрагивается.

## Goals / Non-Goals

**Goals:**

- Один brand-control слева во всех экранах с shared header — **один и тот же interactive DOM** на всех маршрутах
- На Game — reuse существующего leave/confirm path без нового server контракта
- На auth — клик ведёт в lobby (существующие guards могут вернуть гостя на login)
- Высота логотипа ≥ 60px; актуальный прямоугольный product `logo.png`
- Title + favicon продукта; удаление Quasar scaffold brand junk (уже сделано)

**Non-Goals:**

- Dark-variant логотипа; PNG favicon multi-size; PWA icons
- Правки session logout в Lobby; support nested back-nav
- Изменение правил `needsLeaveConfirm` / `leaveGame`

## Decisions

### D1 — Logo в shared header, не page-local

- **Выбор:** импорт `src/assets/brand/logo.png` в shell (`App.vue`); клик-роутинг по `route.name`.
- **Почему:** leave/status уже в App; единая точка для Game confirm.
- **Альтернатива:** отдельный layout-компонент — лишний слой при одном месте.

### D2 — Поведение клика по route

| Route group | Behavior |
|-------------|----------|
| `login`, `forgot-password`, `confirm-email`, `reset-password` | `router.push({ name: 'lobby' })` (guest без сессии может отскочить `requiresAuth` → login) |
| `lobby` | noop (ignore click; тот же button-control) |
| `game` | existing `onExitClick` / confirm / `onLeave` |
| остальные authenticated | `router.push({ name: 'lobby' })` |

**Стабильный chrome:** всегда один и тот же interactive control (не чередовать bare `img` и `button`) — иначе скачет позиция при смене маршрута.

Accessible name на Game: существующий `game.leave` («Выход из игры»). На non-Game (включая auth и Lobby): `auth.backToLobby` / brand home.

### D3 — Убрать page «В лобби»

- Удалить кнопки с `auth.backToLobby` на Account и Support.
- Ключ i18n оставить для aria логотипа.

### D4 — Title и favicon

- `package.json` `productName` → `Happy Tourist` (подставляется в `index.html` `<title>`).
- `index.html`: оставить один `<link rel="icon" … href="favicon.ico">`; убрать четыре PNG `<link>`.
- Удалить файлы: `public/icons/favicon-*.png`, папку `public/icons/` если пуста; `src/assets/quasar-logo-vertical.svg`; мёртвые scaffold `src/pages/index*`.

### D5 — Размер логотипа

- Высота **≥ 60px** (`height: 60px; width: auto; object-fit: contain`). Toolbar может вырасти выше дефолтных ~50px Quasar — ок для прямоугольного wordmark.
- Product asset: актуальный прямоугольный `src/assets/brand/logo.png` (локально уже заменён пользователем — включить в apply/commit).

### Prerequisites

- Ассеты: `logo.png` (прямоугольный), `favicon.ico` — не генерировать, не затирать scaffold’ом.

## Risks / Trade-offs

- [Узкий Game header] logo ~132px wide при 60px height + status + account + theme → Mitigation: image-only, status уже ужимался; при необходимости статус остаётся центрированным через `q-space`.
- [Auth → lobby → login bounce для гостя] → Mitigation: принятый tradeoff; layout стабилен.
- [Lobby noop выглядит «битым»] → Mitigation: тот же button; курсор pointer ок; клик просто ничего не меняет.
- [Skills устареют] → Mitigation: tasks обновить `work-with-pages` / structure (auth → lobby, 60px, always-button).

## Migration Plan

Только client deploy (GitHub Pages). Rollback: revert commit; ассеты brand остаются безвредны.

## Open Questions

Нет.
