## Context

См. `proposal.md` (Why / Scope) и delta specs `auth/email-verification`, `auth/password-reset`.

Wave 1 уже в коде (mailer smtp.bz, `emailVerified`, button-only confirm, change-email, cabinet/forgot client, tests). Осталось:

- Confirm/reset письма и HTML шаблоны Colyseus всё ещё **EN** (`buildConfirmEmailContent`, `html/*.html`, subject в `sendEmail` / `onForgotPassword`).
- Colyseus endpoints хардкодят EN query (`Email confirmed successfully!`, `e.message` вроде `jwt expired`) — страница печатает query as-is.
- Client dialog/forgot success без явного «Спам».
- `.env.example` содержит нерезолвящийся `smtp.smtp.bz` (канон кабинета: `connect.smtp.bz`).
- Prod: API `https://api.happy-tourist.ru`; client origin через `CLIENT_APP_URL` (напр. `https://happy-tourist.ru`); From `@happy-tourist.ru`.

Пакеты: **server** + **client** + **meta** skills. Чеклист — в `tasks.md`.

## Goals / Non-Goals

**Goals:**

- Весь human-facing copy auth-email контура (письма, HTML confirm/reset, клиентские post-send тексты) на **русском**; смысл = текущий EN, перевод.
- Везде после отправки письма — напоминание про папку **Спам**.
- Redirect после confirm на `#/lobby` сохранить (D6).
- Skills/AGENTS: новое в этом контуре — RU; `.env.example` → `connect.smtp.bz`.

**Non-Goals:**

- Полный аудит/перевод legacy UI вне auth-email.
- Смена провайдера почты; SPA confirm/reset pages; hard-gate verified.
- i18n language switcher / второй locale catalog.

## Decisions

### D1: Только smtp.bz + nodemailer

- Как Wave 1: `src/lib/mailer.ts`; env `SMTP_BZ_*`, `MAIL_FROM`.
- **Host канон:** `connect.smtp.bz` (порт 2525 / 587 STARTTLS или 465/9465 SSL; в коде `secure` при `port === 465 || port === 9465`).
- Ops: DNS/SPF(+DKIM); секреты только VPS.

### D2: Не вешать `onSendEmailConfirmation`

Без изменений Wave 1.

### D3–D5: `emailVerified`, soft gate, endpoints

Без изменений Wave 1.

### D6: Confirm/reset UI = серверные HTML + redirect lobby

- Сохранить `writeConfirmSuccessHtml` → `CLIENT_APP_URL/#/lobby`.
- Русифицировать `html/address-confirmation.html` (и генерацию в `auth.ts`): заголовок RU; **маппинг** EN success/error query от Colyseus → RU строки (в т.ч. expired/invalid token). Не патчить `node_modules/@colyseus/auth`.
- Русифицировать `html/reset-password-email.html`, `html/reset-password-form.html` (+ success/error отображение).
- Subject/body confirm: `buildConfirmEmailContent` + subject в `app.config` send — RU.
- Forgot: `onForgotPassword` subject RU; тело из RU шаблона (или подмена html перед `sendEmail`).

### D7: Client UX + спам

- `auth.confirmSentDialog` / `auth.forgotSuccess` (i18n): добавить проверку папки «Спам».
- Каталог остаётся `src/i18n/en-US/` (техническое имя); строки — русские.

### D8: Тесты

- Расширить mock-mailer asserts: subject/body содержат кириллицу / ожидаемые RU маркеры; HTML confirm expired → RU outcome где тестируется.
- Client: lint + typecheck после правок i18n.

### D9: Meta канон языка

- Обновить `client-work-with-auth`, `work-with-localization`, `server-work-with-auth`, `work-with-env-deploy`, при необходимости AGENTS (meta/client/server): human-facing auth-email / новое в контуре = русский; не требовать EN copy.

### D10: Перевод = тот же смысл

- Не менять продуктовый flow; только язык. Формулировки — разумный перевод текущего EN.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Colyseus снова шлёт EN в query | Маппинг на HTML-странице + fallback «понятное RU» для неизвестных error |
| VPS `address-confirmation.html` со старым `lobbyUrl` | Reload после `CLIENT_APP_URL`; файл пишется на boot |
| Неверный SMTP host в примере | `.env.example` → `connect.smtp.bz` |

## Migration Plan

1. Задеплоить RU шаблоны + client i18n; reload PM2 (перезапись HTML + env).
2. Ops: убедиться `SMTP_BZ_HOST=connect.smtp.bz`, `CLIENT_APP_URL` = реальный client origin.
3. Rollback: вернуть предыдущий build; login не ломается.

## Open Questions

(нет — D1 reset HTML RU; spam везде при send; `.env.example` host; update текущего change.)
