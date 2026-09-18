## Context

См. `proposal.md` и delta specs `auth/email-verification`, `auth/password-reset`.

Wave 1–2 в коде: mailer smtp.bz, `emailVerified`, cabinet/forgot, RU copy на письмах и серверных HTML Colyseus, `connect.smtp.bz`.

Wave 3 (этот update): перенос confirm/reset UX на клиентский SPA + JSON; forgot not-found; без поддержки старых `api.*` HTML-ссылок.

Prod: client `https://happy-tourist.ru`; API `https://api.happy-tourist.ru`; From `@happy-tourist.ru`.

## Goals / Non-Goals

**Goals:**

- Ссылки в письмах → `{CLIENT_APP_URL}/#/confirm-email?token=` и `#/reset-password?token=`.
- JSON confirm/reset на server (`createEndpoint`); SPA не парсит Colyseus HTML redirects.
- Confirm: auto-call JSON on open → `#/lobby`.
- Reset: SPA form → JSON → `#/login`.
- Forgot: явный not-found; success без «если существует».
- Не поддерживать legacy API HTML как product path (D6 explore).

**Non-Goals:**

- Nginx proxy `/auth/` на apex.
- Hard-gate verified; Resend; password policy.
- Полный перевод legacy UI вне auth-email.

## Decisions

### D1–D5, D7–D10 (Wave 1–2, без изменений смысла)

- D1 smtp.bz + `connect.smtp.bz` (`secure` на 465|9465).
- D2 нет `onSendEmailConfirmation`.
- D3–D5 `emailVerified`, soft gate, send-confirm / change-email.
- D7 spam в client post-send copy.
- D8 tests / D9 skills RU / D10 перевод = тот же смысл для писем.

### D6 (замена Wave 2): Confirm/reset UX = client SPA + JSON

**Было (Wave 2):** серверные `address-confirmation.html` / `reset-password-form.html` + redirect lobby на confirm.

**Стало (Wave 3):**

- Public routes (не `meta.guest`): `ConfirmEmailPage`, `ResetPasswordPage` — без `guest`, иначе залогиненный пользователь уйдёт в lobby до confirm.
- Confirm: on mount читает `token` из query → `POST` JSON → RU success → `router` → lobby; ошибка → RU banner на странице. **Без** второй кнопки «Подтвердить».
- Reset: форма пароля → `POST` JSON → RU success → login.
- Письма: `buildConfirmEmailContent` / forgot HTML `[LINK]` = client hash URLs (`CLIENT_APP_URL`), не `AUTH_BACKEND_URL/auth/...`.
- `AUTH_BACKEND_URL` / `auth.backend_url` — только для Google OAuth и API origin, не для user-facing mail links.
- Colyseus built-in HTML confirm/reset **не** product path; можно перестать писать/обслуживать `writeConfirmSuccessHtml` как UX (удалить или оставить мёртвым — на усмотрение apply, поведение не требуется).
- Альтернатива «прокси `/auth/` на apex» — отклонена (explore D2).

### D11: Forgot unknown email = explicit not-found

- Colyseus уже отдаёт ошибку при отсутствии пользователя; клиент показывает понятный RU (`email_not_found` → «Аккаунт с таким email не найден» или эквивалент).
- Success copy: «Письмо отправлено… и Спам» — **без** «если аккаунт существует».
- Tradeoff: enumерация аккаунтов — принято продуктом.

### D12: JSON endpoint shapes (apply defaults)

- Confirm: unauthenticated `POST` с body `{ token }` → verify JWT `{ email }` → `onEmailConfirmed` / DB `emailVerified=true` → `{ ok: true }` или ошибка с предсказуемым кодом/сообщением (expired/invalid).
- Reset: unauthenticated `POST` `{ token, password }` → verify → `onResetPassword` / Hash → ok или error; reuse token one-time semantics если уже есть в Colyseus presence.
- Можно обернуть существующую логику Colyseus callbacks; не парсить HTML с клиента.
- Prefers `/api/auth/confirm-email` и `/api/auth/reset-password` рядом с send-confirm / change-email (`work-with-routes`).

### D13: Post-success navigation

- Confirm → `#/lobby` (если нет сессии, guard уведёт на login — ок).
- Reset → `#/login` явно.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Старые письма на `api.*` сломаются | Принято (D6 explore); новые письма только client links |
| Token в hash URL | Нормально для SPA; JSON body не логирует token в access log так же, как query на API page |
| Два пути reset (Colyseus HTML + JSON) путают | Не документировать HTML; тесты только на JSON + SPA |
| Enumерация через forgot | Принято (D11) |

## Migration Plan

1. Задеплоить server JSON + link builders, затем client SPA pages.
2. Ops: `CLIENT_APP_URL=https://happy-tourist.ru`; OAuth callback по-прежнему на API.
3. Rollback: предыдущий build (старые HTML снова в письмах только если откатить и server, и уже ушедшие письма).

## Open Questions

(нет — explore D1–D6 закрыты.)
