## Context

См. `proposal.md` — Why / Scope. Пакет cross-client+server: lobby/game UX поверх существующих Pinia/`GamePage` паттернов; кабинет поверх Wave 1 `AccountPage` + `@colyseus/auth` + `createEndpoint`.

Текущие щели (explore):

- Lobby: create занят `persistent` modal + `creating`; join — `joining` на кнопке без disable строки / early-return.
- `game.rooms` не очищается при `unsubscribeLobby` / `_resetRoomState` → flash после leave.
- `GamePage`: `isBoardBusy` → `isInteractive`, но `beginMoveAnimation` после `sendMove` и окно до catapult/grille; strip уже под `isInteractive`; say не в busy.
- Top presence: `v-if="topPresenceMarkers.length > 0"` → скачок ~96px+margin.
- Register `{ name }` не попадает в `displayName`; смены имени/пароля в кабинете нет; policy = `min 6`.

## Goals / Non-Goals

**Goals:**

- Закрыть UX 1–4 без новых runtime-сервисов.
- Единый password policy helper (client mirror + server enforce) на register / change / reset.
- Client meter via `@zxcvbn-ts/core` + `language-common` + `language-ru` (advisory).
- Account: имя + смена пароля (email/password) + существующий email-verify; Google — без смены пароля.

**Non-Goals:**

- Модерация имён; hard-gate `emailVerified`; complexity на login; set-password для Google; порог zxcvbn score на Submit.

## Decisions

### D1 — Один change, два домена в tasks

Порядок apply: **server auth/policy/endpoints сначала**, затем client forms/stores, затем lobby/game UX (можно параллелить client UX с server, если контракт имени/пароля уже готов). Чеклист в `tasks.md`.

### D2 — Lobby join busy

`LobbyPage`: early-return если `joining`; `:disable` / non-clickable на `q-item` (или overlay) пока `joining`; кнопка «Войти» может остаться visual loading. Паттерн как skill `work-with-lobby` (bind busy). Опционально общий `entering` на create+join — не обязательно, достаточно симметрии disable.

### D3 — Stale rooms

В `useGameStore`: при старте `subscribeLobby` и/или в `unsubscribeLobby` / `leaveGame` выставлять `rooms = []` и держать `listing` true до первого сообщения `rooms` (предпочтительно: clear + listing until snapshot). Не показывать stale массив после `listing = false`.

### D4 — Board busy без щелей

В `GamePage.vue` (page-local, не Pinia):

- Ставить presentation-busy **до** `sendMove` (intent lock).
- Держать busy непрерывно через intent→move→trap gap→grille **drop**/catapult/fling/finish (pipeline settle bridges into drop).
- После settle в static grille **hold** — unlock, чтобы rescue был возможен при видимой решётке.
- Strip остаётся под `isInteractive`; say / header leave — вне lock.
- Не переносить таймеры анимаций в store.

### D5 — Top presence reserve

Для seated всегда рендерить контейнер `.presence-row--top` с `min-height` (существующие ~96px), даже если оппонентов 0. Без empty-seat аватаров (SC-PRESENCE-05). Spectator без этого «alone»-правила.

### D6 — Password policy

Общий модуль правил (server + зеркало client):

- length ≥ 8
- `/[a-z]/`, `/[A-Z]/`, `/[0-9]/`, `/[^A-Za-z0-9]/` (символ)

Enforce: register path (wrap/validate до/вместо голого Colyseus accept), `POST` change-password, JSON reset. Login form: без complexity UI. Colyseus floor 6 не поднимаем глобально слепо — product reject выше floor.

### D7 — zxcvbn-ts

Client deps: `@zxcvbn-ts/core`, `@zxcvbn-ts/language-common`, `@zxcvbn-ts/language-ru`. UI: цветная шкала + чеклист policy. Score не блокирует Submit.

### D8 — displayName

- Register: сохранить `options.name` → колонка `displayName` (хук/обёртка register или post-create update).
- JWT/userdata: отдавать имя так, чтобы `auth.displayName` предпочитал persisted display name.
- `POST` update displayName (`createEndpoint`, auth required) — trim, min 1; без модерации.
- UI: блок на `AccountPage`.

### D9 — Change password

- `POST` body `{ currentPassword, newPassword }`: `Hash.verify` → `setPasswordHash` / `onResetPassword` → **`bumpTokenVersion`** → client `logout` → Login.
- Скрыть блок если нет password credential (Google) или anonymous.
- Reset SPA: тот же policy + meter; bump на reset — по возможности выровнять с change (если сегодня reset не bump’ает — добавить в этом change для единообразия сессий).

### D10 — Soft-verify

Не добавлять gates; SC-PROFILE-06 фиксирует статус-кво soft.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Colyseus register игнорирует userdata options | Явная запись `displayName` после/внутри register hook; тест SC-AUTH-10 |
| Busy lock слишком длинный (ощущение «зависло») | Lock только на presentation flags; say/leave доступны |
| Dispose race всё ещё даст короткий ghost после свежего snapshot | Clear stale убирает худший flash; snapshot-first listing |
| zxcvbn bundle size | Dynamic import meter helper на формах пароля |
| Имена без модерации видны всем | Зафиксировано out → отдельная волна |

## Migration Plan

- Client: `npm install` новых zxcvbn packages; без миграции данных обязателен backfill имён (только новые register + ручная смена в кабинете).
- Server: новые endpoints; существующие пользователи с пустым `displayName` продолжают видеть email/fallback как сейчас.
- Rollback: revert change; deps удаляются с client.

## Open Questions

Нет — D* из explore закрыты; модерация имён отложена сознательно.
