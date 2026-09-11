## Why

Игроки ожидают тёмную тему и сохранение выбора между сессиями. Нужны тема устройства по умолчанию, явный выбор light/dark и синхронизация для зарегистрированных пользователей. После сохранения темы reload (в том числе на другом уже залогиненном устройстве) не должен возвращать устаревшее значение из JWT — нужна актуальная тема из профиля БД без повторного логина.

## What Changes

- Добавляется переключение светлой/тёмной темы приложения (chrome Quasar) с кнопкой в общей шапке на Login, Lobby и Game.
- Если пользователь ещё ничего не выбирал — UI следует теме устройства (OS).
- Гость (anonymous): preference только на устройстве (локально).
- Зарегистрированный (email / Google): preference в профиле пользователя в БД; сохранение на сервере; подтягивание при логине и при reload/restore сессии из профиля (не только из устаревших JWT claims).
- Уже открытая сессия на другом устройстве получает новую тему после **обновления страницы** (без re-login и без live-push).
- Доска шашек и правила игры не меняются.

## Scope

- **Capability ID:** `ui/theme` (новый домен UI preferences; согласован в explore).
- **Пакеты:** client и server.
- **Client:** страницы Login / Lobby / Game; guest vs authenticated; общая шапка с переключателем; restore/reload registered → тема из профиля.
- **Server:** поле preference в user store; thin HTTP сохранения и **чтения** актуальной темы из профиля (JWT для auth, значение theme — из БД при read).
- **Контракт:** HTTP preference (+ userdata при login); room `checkers` / move / board **без изменений**.

## Out of scope

- Отдельная тёмная палитра клеток/фигур на доске.
- Возврат к «как на устройстве» после явного light/dark в v1 (только light ↔ dark после первого выбора).
- Live-sync темы на уже открытой вкладке **без** reload / без возврата к приложению (push / polling в фоне).
- Сохранение темы гостя в БД и кросс-девайс для anonymous.
- PWA `theme-color`, системные уведомления, email-шаблоны.
- Изменение auth flows (Google / email / anonymous) кроме поля темы в профиле / userdata и read профиля.

## Capabilities

### New Capabilities

- `ui/theme`: выбор и применение light/dark (default — тема устройства); persist для guest локально; для зарегистрированных — в профиле БД, при логине и при reload/restore из актуального профиля.

### Modified Capabilities

- (нет)

## Impact

- Client SPA (Quasar Dark, шапка, preference sync, restore via profile read).
- Server: колонка user profile + HTTP save/read темы; login userdata по-прежнему может содержать theme.
- Docs/skills после реализации: client styles / stores / server DB+routes (check-changes).
- Room protocol и gameplay не затрагиваются.

## References

- `docs/projects-map.md` — пути client/server.
- `../happy-tourist.github.io/AGENTS.md` — client UI / auth.
- `../happy-tourist-server/AGENTS.md` — server auth / DB.
- Explore: Quasar Dark; guest = local only; registered = DB; header toggle; unset → device; reload sync from profile (не JWT-only).
