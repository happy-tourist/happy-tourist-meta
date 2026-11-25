## Why

Экран Login должен быть доступен только по прямой ссылке (без кнопки в шапке и без принудительных редиректов guard → Login). Параллельно ошибки Google OAuth на Login показывают `[object Object]` вместо читаемого текста провайдера — это мешает диагностике и UX.

## What Changes

- Убрать affordance «Войти» из шапки (wide + burger) для гостя; LoginPage без изменений по набору контролов.
- Guard больше не направляет на Login: вместо прежних fallback/staff-deep-link → Login отправлять на Lobby один раз, без петли Lobby→Lobby при повторном fail.
- На Login при ошибке Google (и аналогичных structured errors) показывать читаемый текст (`error_description` / `error`), а не `[object Object]`.
- Обновить клиентские сценарии/тесты под новое поведение.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `auth/login`: silent-guest fail и staff/admin без сессии → Lobby (не Login); Login только по прямому URL; читаемый текст ошибки Google OAuth на Login.
- `ui/branding`: убрать guest login icon / burger login из шапки (SC-BRAND-22/24/31 и связанные формулировки).

## Scope

- Capability ID: `auth/login`, `ui/branding`.
- Пакет: **client** only (навигация, шапка, auth error surface на Login).
- Контракт room/HTTP/Google provider на server **не** меняется этим change (ops: неверный `GOOGLE_CLIENT_SECRET` — вне кода).

## Out of scope

- Смена OAuth на full-page redirect / nginx COOP / CORS / `CLIENT_APP_URL`.
- Правка server env / ротация Google secret (ops).
- Русские i18n-маппинги кодов вроде `invalid_client` (достаточно сырого `error_description` / `error`).
- Удаление или упрощение контролов на самом LoginPage.
- Изменение logout / registered account cluster в шапке.

## Impact

- Client: shared header, router auth guard, Pinia auth error formatting для Google, vitest (header chrome + authGuard + связанные SC).
- Server: нет.
- **BREAKING** (продуктовое): гость больше не открывает Login из шапки; deep-link staff/admin без сессии и fail silent-guest больше не ведут на Login.

## References

- `openspec/specs/auth/login/spec.md` (SC-AUTH-05, SC-AUTH-11b, SC-AUTH-12, SC-AUTH-13, SC-AUTH-15)
- `openspec/specs/ui/branding/spec.md` (SC-BRAND-22, SC-BRAND-24, SC-BRAND-31)
- Sibling: `../happy-tourist.github.io/AGENTS.md`; skills `client-work-with-auth` (+ `guards.md`), `work-with-pages` / header chrome
