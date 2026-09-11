## Why

Игроки ожидают тёмную тему и сохранение выбора между сессиями. Сейчас UI всегда светлый: нет переключения, нет preference в профиле. Нужна тема устройства по умолчанию, явный выбор light/dark и синхронизация для зарегистрированных пользователей между устройствами после логина.

## What Changes

- Добавляется переключение светлой/тёмной темы приложения (chrome Quasar) с кнопкой в общей шапке на Login, Lobby и Game.
- Если пользователь ещё ничего не выбирал — UI следует теме устройства (OS).
- Гость (anonymous): preference только на устройстве (локально).
- Зарегистрированный (email / Google): preference в профиле пользователя в БД; подтягивается при логине; смена темы сохраняется на сервере.
- Доска шашек и правила игры не меняются.

## Scope

- **Capability ID:** `ui/theme` (новый домен UI preferences; согласован в explore).
- **Пакеты:** client и server.
- **Client:** страницы Login / Lobby / Game; поведение для guest vs authenticated; общая шапка с переключателем темы.
- **Server:** поле preference темы в user store; тонкий HTTP для сохранения выбора зарегистрированным пользователем (JWT); значение попадает в userdata при login/register/Google.
- **Контракт:** HTTP preference + userdata поле темы; room `checkers` / move / board **без изменений**.

## Out of scope

- Отдельная тёмная палитра клеток/фигур на доске.
- Возврат к «как на устройстве» после явного light/dark в v1 (только light ↔ dark после первого выбора).
- Live-sync темы на уже открытой сессии другого устройства без повторного логина.
- Сохранение темы гостя в БД и кросс-девайс для anonymous.
- PWA `theme-color`, системные уведомления, email-шаблоны.
- Изменение auth flows (Google / email / anonymous) кроме включения поля темы в userdata / профиля.

## Capabilities

### New Capabilities

- `ui/theme`: выбор и применение light/dark (default — тема устройства); persist для guest локально; для зарегистрированных — в профиле БД и при логине.

### Modified Capabilities

- (нет)

## Impact

- Client SPA (Quasar Dark, шапка, preference sync).
- Server: колонка user profile + HTTP endpoint сохранения темы; userdata при логине содержит theme при наличии.
- Docs/skills после реализации: client styles / auth userdata / server DB+routes (вне runtime change apply — через check-changes).
- Room protocol и gameplay не затрагиваются.

## References

- `docs/projects-map.md` — пути client/server.
- `../happy-tourist.github.io/AGENTS.md` — client UI / auth.
- `../happy-tourist-server/AGENTS.md` — server auth / DB.
- Explore: Quasar Dark; guest = local only; registered = DB, apply on login; header toggle; unset → device theme.
