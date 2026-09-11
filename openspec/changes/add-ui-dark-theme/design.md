## Context

См. `proposal.md` (Why / Scope). Client: Quasar Dark, `App.vue` header toggle, theme store, guest `localStorage`. Server: nullable `users.theme`, `POST /api/theme`. `GET /auth/userdata` отдаёт JWT payload (`onParseToken`), **не** live SELECT — после POST claims устаревают, поэтому F5 и другое уже залогиненное устройство после своего reload показывают старую тему, пока read не идёт из БД.

Пакеты: **client** (`../happy-tourist.github.io`) + **server** (`../happy-tourist-server`). Room `checkers` / messages / board schema **без изменений**.

## Goals / Non-Goals

**Goals:**

- Quasar Dark для chrome; default `auto` (device), после выбора — `light` | `dark`.
- Общая шапка с toggle на всех экранах.
- Guest → `localStorage`; registered → колонка profile + HTTP save; apply при логине и при reload/restore из **профиля БД**.
- Другое устройство с живой сессией → актуальная тема после **его** reload (без re-login).
- Контраст вторичного текста (`text-grey-7` и аналоги) в dark.

**Non-Goals:**

- Редизайн доски; return-to-auto после явного выбора; live-push / polling без reload; guest→DB; изменение OAuth/email flows кроме theme в профиле/userdata.

## Decisions

### D1 — Quasar `Dark` plugin (не вторая дизайн-система)

- Включить `Dark` в `framework.plugins`; стартовое значение через boot/`Dark.set`.
- Альтернатива: CSS-переменные / свой theme engine — отвергнуто (skill запрещает второй theme system; Quasar уже в стеке).

### D2 — Значения preference

- Хранимые: `light` | `dark` | unset/`null`.
- Unset → клиент `Dark.set('auto')`.
- Toggle после первого явного выбора: light ↔ dark (v1 без третьего состояния «снова auto»).
- Альтернатива: всегда три состояния в UI — отложено.

### D3 — Шапка в `App.vue`

- Один `q-header` с кнопкой темы (иконки `dark_mode` / `light_mode`) над `q-page-container`, чтобы Login / Lobby / Game не дублировали chrome.
- Альтернатива: кнопка в каждой page — больше копипасты.

### D4 — Client state / I/O

- Тонкий setup-store предпочтений (не раздувать `auth`): apply Dark; guest read/write `localStorage` (`ht-theme`); registered — `client.http.post` save и **`client.http.get` restore**.
- При session restore / `auth.ready` для registered: читать тему через GET профиля и `apply`; **не** использовать JWT `user.theme` как единственный источник после reload (SC-THEME-08/09).
- Login userdata по-прежнему может сразу показать theme; reload обязан сходиться с GET.
- Guest: без GET; только localStorage.
- После успешного POST: локальный apply + localStorage; патч in-memory `auth.user.theme` — не замена GET на следующем reload.
- HTTP I/O — только из store/composable, не из template.

### D5 — Server: колонка + thin POST

- `src/db/schema.ts`: nullable text `theme` — без NOT NULL.
- Endpoint: `POST /api/theme` через `createEndpoint` + `auth.middleware()`; body `{ theme: 'light' | 'dark' }`; update по `auth.id`; reject anonymous / missing id.
- Login кладёт custom columns в JWT — theme в userdata после следующего login; для текущей сессии клиент применяет локально после POST.
- Prod SQLite: ALTER / auto-migrate колонки — см. Risks.
- CORS: POST (+ GET в D8); Authorization в allow-list.

### D6 — Доска

- Не менять scoped CSS клеток/фигур в GamePage; класс `.cell.dark` остаётся цветом клетки, не режимом приложения.

### D7 — Docs/skills (после кода)

- Обновить `work-with-styles` / stores / routes (runtime Dark, GET restore) — через check-changes; не блокер design.

### D8 — Источник правды при restore: профиль БД через GET

- Thin **`GET /api/theme`**: `auth.middleware()`; registered only; `SELECT theme` по `auth.id`; ответ `{ theme: 'light' | 'dark' | null }`; reject unauth / anonymous.
- Альтернатива: `onParseToken` с SELECT на каждый userdata — сильнее связывает auth с UI-pref; отвергнуто.
- Альтернатива: re-sign JWT после POST — не чинит устройство B со старым JWT до DB read; GET достаточнее.

Чеклист реализации: `tasks.md`.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| JWT/`/auth/userdata` устаревает после POST | GET `/api/theme` при restore; клиент не опирается только на JWT theme |
| Лишний HTTP на каждый reload | Один GET; кэш не обязателен в v1 |
| Flash: boot local → затем GET | Early localStorage apply; GET уточняет registered |
| Prod `game.db` без новой колонки | ALTER/migrate в tasks; default null |
| Anonymous с row в БД | POST/GET reject `anonymous === true` |
| Контраст `text-grey-7` в dark | Пройти Login/Lobby/Game chrome в tasks |

## Migration Plan

1. Deploy server schema + POST (+ затем GET) theme.
2. Deploy client Dark + header + sync; затем restore via GET.
3. Rollback: старый client игнорирует GET; POST/колонка безвредны.

## Technical prerequisites (из explore)

- Закрыты: DB для registered; guest localStorage; apply on login; device default; header; board unchanged; reload sync from profile (не live без reload).
- Открытых блокеров нет.
