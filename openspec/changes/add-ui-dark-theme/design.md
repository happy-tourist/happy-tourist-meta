## Context

См. `proposal.md` (Why / Scope). Сейчас в client нет runtime Dark: `quasar.config.ts` → `plugins: []`, `App.vue` — только `q-layout` + `router-view` без шапки. Skill `work-with-styles` фиксирует отсутствие runtime theme override. Server: `colyseus_users` расширен `displayName` / rating / games*; отдельного preferences API нет. `GET /auth/userdata` отдаёт JWT payload (`onParseToken`), не live SELECT — для «подтянуть при логине» достаточно поля в row → JWT на login/register/Google.

Пакеты: **client** (`../happy-tourist.github.io`) + **server** (`../happy-tourist-server`). Room `checkers` / messages / board schema **без изменений**.

## Goals / Non-Goals

**Goals:**

- Quasar Dark для chrome; default `auto` (device), после выбора — `light` | `dark`.
- Общая шапка с toggle на всех экранах.
- Guest → `localStorage`; registered → колонка profile + HTTP save; apply theme из userdata при логине/`onChange`.
- Контраст вторичного текста (`text-grey-7` и аналоги) в dark.

**Non-Goals:**

- Редизайн доски; return-to-auto после явного выбора; live sync без логина; guest→DB; изменение OAuth/email flows кроме theme в userdata.

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

- Тонкий composable или setup-store предпочтений (не раздувать `auth` Colyseus-логикой UI): apply Dark; guest read/write `localStorage` (ключ вида `ht-theme`); для registered — `client.http.post` сохранения.
- Применение серверной темы: в реакции на `auth.onChange` / после login, если `user.theme` ∈ {`light`,`dark`} и `!user.anonymous`; иначе local/auto.
- HTTP I/O предпочтений — через auth/preferences helper в store/composable, не из template.

### D5 — Server: колонка + thin POST

- `src/db/schema.ts`: nullable text `theme` (например `theme: text("theme")`) — без NOT NULL, чтобы `/auth/register|/auth/login` не ломались.
- Endpoint: `POST /api/theme` (или `/api/preferences/theme`) через `createEndpoint` + `auth.middleware()`; body `{ theme: 'light' | 'dark' }`; обновить row по `auth.id`; отклонить anonymous / missing id.
- Login уже кладёт custom columns в JWT через `findByEmail` / anonymous register row — theme попадёт в userdata после save и следующего login; для текущей сессии клиент применяет локально сразу после успешного POST.
- Prod SQLite: добавить колонку на существующий `game.db` (ALTER / совместимый путь GameDatabase) — см. Risks.
- CORS уже разрешает POST; Authorization header уже в allow-list.

### D6 — Доска

- Не менять scoped CSS клеток/фигур в GamePage; класс `.cell.dark` остаётся цветом клетки, не режимом приложения.

### D7 — Docs/skills (после кода)

- Обновить `work-with-styles` (runtime Dark), при необходимости DB/routes/auth skills — через check-changes; не блокер design.

Чеклист реализации: `tasks.md`.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| JWT/`/auth/userdata` не отражает свежий theme до re-login | По продукту ок (apply on login); после POST клиент сразу `Dark.set` локально |
| Prod `game.db` без новой колонки | Явный ALTER/migrate step в tasks; default null |
| Anonymous с row в БД может вызвать PUT по ошибке | Endpoint явно rejects `anonymous === true` |
| Контраст `text-grey-7` в dark | Пройти Login/Lobby/Game chrome в tasks |
| Flash светлой темы до boot | Ранний boot Dark + чтение localStorage до paint по возможности |

## Migration Plan

1. Deploy server schema + endpoint (nullable theme).
2. Deploy client с Dark + header + sync.
3. Rollback: client без Dark безопасен при лишней колонке; endpoint можно оставить; колонку не удалять без нужды.

## Technical prerequisites (из explore)

- Закрыты: DB для registered; guest localStorage; apply on login; device default; header all pages; board unchanged.
- Открытых блокеров нет.
