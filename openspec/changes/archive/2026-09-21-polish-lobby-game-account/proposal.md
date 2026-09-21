## Why

В лобби и на поле остаются щели UX (двойной join, скачок доски, мигание списка комнат, клики во время анимаций). Параллельно кабинет после soft-verify Wave 1 не умеет сменить имя и пароль, а политика пароля ограничена floor Colyseus `min 6`. Нужен один пакет доработок: стабильный lobby/game UX + полноценный кабинет с политикой и шкалой силы.

## What Changes

- Лобби: блокировка повторного входа в комнату на время connect (симметрия с create); очистка stale-списка при выходе/resubscribe, чтобы убрать flash «пусто → комната → пусто».
- Игра: ужесточение lock кликов по полю и полосе туристов на всё presentation-окно (в т.ч. щели move→catapult/grille); say с аватара остаётся доступным; резерв высоты верхнего presence-ряда у seated, чтобы доска не прыгала при подключении соперника.
- Кабинет: смена `displayName` (min 1); смена пароля (текущий + новый) для email/password; после смены — выход и повторный вход; блок смены пароля скрыт для Google/anonymous.
- Регистрация: `name` сохраняется в `displayName`; на register / change / reset — единая password policy (≥8, латиница lower+upper, цифра, символ) и client-only strength meter (`zxcvbn-ts` + RU dict); login без complexity; Colyseus `min 6` остаётся нижним полом API.
- Soft-verify без hard-gate: неподтверждённый email играет как подтверждённый.

## Scope

- **Capability ID:** `lobby/rooms`, `game/board`, `game/presence`, `auth/login`, `auth/password-reset`, `auth/profile` (новый)
- **Пакеты:** client + server + meta (skills/AGENTS при необходимости)
- **UX:** Lobby join busy; Game board/strip lock; Game seated top presence reserve; Account кабинет (имя, пароль, существующий email-verify)
- **Auth / HTTP:** persist register name; POST обновление имени; POST смена пароля + bump token version; policy enforce на register/change/reset
- **Deps (client):** `@zxcvbn-ts/core`, `@zxcvbn-ts/language-common`, `@zxcvbn-ts/language-ru`
- **Язык:** RU human-facing copy в затронутых формах

## Out of scope

- Модерация / denylist имён (отдельная волна)
- Hard-gate по `emailVerified` (create/say/ranked и т.п.)
- Complexity check на **login**
- Set-first-password / смена пароля для Google OAuth
- Новые mail-провайдеры; переписывание SPA confirm/reset links (Wave 1 уже есть)
- Правила игры, seed densities, ranked matchmaking
- Порог zxcvbn score как условие Submit (meter только advisory)

## Capabilities

### New Capabilities

- `auth/profile`: кабинет — обновление отображаемого имени и смена пароля (email/password); Google без смены пароля; после смены пароля — повторный вход

### Modified Capabilities

- `lobby/rooms`: busy/disable на join; очистка stale listing при leave / resubscribe
- `game/board`: board/strip non-interactive на всём presentation pipeline без щелей
- `game/presence`: seated всегда резервирует высоту верхнего presence-ряда (без пустых аватаров)
- `auth/login`: password policy + strength meter на регистрации; persist register name → displayName
- `auth/password-reset`: та же password policy (+ meter) на SPA reset

## Impact

- Client: Lobby join UX; GamePage busy/presence layout; Login/Reset/Account forms; auth store; deps zxcvbn-ts
- Server: register name → DB; HTTP endpoints имени и смены пароля; shared policy validation; bumpTokenVersion на change password
- Soft-verify и email flows Wave 1 без изменения gating
- Skills/AGENTS: auth forms, lobby busy, board-busy, account profile — по факту apply

## References

- Explore 2026-09-21: lobby join/create asymmetry; stale `rooms`; top presence `v-if`; board-busy gaps; Wave 1 AccountPage; D* decisions (policy, zxcvbn-ru, no hard-gate, Google hide pwd, name without moderation)
- Sibling AGENTS; skills `work-with-lobby`, `work-with-game-board`, `client-work-with-auth`, `server-work-with-auth`, `work-with-forms`, `work-with-pages`
- Main specs: `lobby/rooms`, `game/board`, `game/presence`, `auth/login`, `auth/password-reset`, `auth/email-verification` (soft, без hard-gate)
