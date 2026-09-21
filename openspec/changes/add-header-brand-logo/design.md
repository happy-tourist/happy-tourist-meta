## Context

См. proposal.md — Why. Сейчас shared header в client (`App.vue`): слева Material `logout` только на Game (`onExitClick` → confirm / `leaveGame` → lobby); справа account (registered) + theme. Page-level «В лобби» — Account и Support. Title из `package.json` `productName` = `happy-tourist-client`; `index.html` ссылается на scaffold PNG favicons + `favicon.ico`. Ассеты продукта уже положены: `src/assets/brand/logo.png`, `public/favicon.ico`. Server не затрагивается.

## Goals / Non-Goals

**Goals:**

- Один brand-control слева во всех экранах с shared header
- На Game — reuse существующего leave/confirm path без нового server контракта
- Title + favicon продукта; удаление Quasar scaffold brand junk

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
| `login`, `forgot-password`, `confirm-email`, `reset-password` | decorative (`img` / non-button), no navigation |
| `lobby` | noop (ignore click) |
| `game` | existing `onExitClick` / confirm / `onLeave` |
| остальные authenticated | `router.push({ name: 'lobby' })` |

Accessible name на Game: существующий `game.leave` («Выход из игры»). На кликабельных non-Game — aria вроде «В лобби» / brand home (reuse `auth.backToLobby` или короткий brand key).

### D3 — Убрать page «В лобби»

- Удалить кнопки с `auth.backToLobby` на Account и Support.
- Ключ i18n можно оставить, если используется для aria логотипа; иначе убрать мёртвый ключ в том же change.

### D4 — Title и favicon

- `package.json` `productName` → `Happy Tourist` (подставляется в `index.html` `<title>`).
- `index.html`: оставить один `<link rel="icon" … href="favicon.ico">`; убрать четыре PNG `<link>`.
- Удалить файлы: `public/icons/favicon-*.png`, папку `public/icons/` если пуста; `src/assets/quasar-logo-vertical.svg`; опционально мёртвый `src/pages/index/(index).vue` (единственный consumer Quasar logo).

### D5 — Размер логотипа

- Высота ~28–32px в toolbar (визуально вровень с round dense buttons); `object-fit: contain`.

### Prerequisites

- Ассеты уже на месте: `logo.png`, `favicon.ico` — не генерировать, только подключить / не затирать чужим scaffold.

## Risks / Trade-offs

- [Узкий Game header] logo + status + account + theme → Mitigation: logo компактный image-only, status уже ужимался под leave.
- [Клик на Lobby выглядит «битым»] → Mitigation: курсор default / не button-look; или лёгкий press без nav — prefer non-button styling when noop.
- [Skills устареют] (`logout` left on Game) → Mitigation: пункт в tasks обновить `work-with-pages` / structure канон шапки.

## Migration Plan

Только client deploy (GitHub Pages). Rollback: revert commit; ассеты brand остаются безвредны.

## Open Questions

Нет.
