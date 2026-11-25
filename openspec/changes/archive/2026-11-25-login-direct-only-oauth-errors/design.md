## Context

See `proposal.md` — Why. Client today: shared header shows guest login (`header-login` / `header-burger-login`); `authBeforeEach` sends silent-guest fail and staff/admin-without-session to `/login?redirect=…`; Google failures via SDK become `Error` with `message === "[object Object]"` while structured payload sits on `error.code`.

## Goals / Non-Goals

**Goals**

- Убрать все UI-входы на Login из шапки.
- Guard: ноль редиректов на Login; fail / staff-unauth → Lobby один раз без петли.
- LoginPage: читаемый текст structured OAuth error в баннере.
- Обновить vitest под новые SC.

**Non-Goals**

- Server/env/Google secret, COOP/nginx, CORS/`CLIENT_APP_URL`.
- i18n-маппинг `invalid_client`.
- Смена popup OAuth на full-page.

## Decisions

### D1 — Guard targets Lobby, never Login

| Было | Стало |
|------|--------|
| staff/admin + !session → `/login?redirect=` | → `/lobby` (без anonymous на этом hop) |
| silent guest fail / no session → `/login?redirect=` | → `/lobby`, если `to` уже lobby — `return true` / stay (без нового redirect) |

Anonymous на staff-hop по-прежнему запрещён; последующий заход на Lobby может сделать silent guest как обычный `requiresAuth`.

**Alternative rejected:** оставлять `/login` как emergency — противоречит «только прямая ссылка».

### D2 — Header: удалить guest login controls

Убрать wide `header-login` и burger `header-burger-login`. Guest cluster = theme only. Registered logout без изменений.

### D3 — `formatAuthError` в auth store (Google catch)

В `loginWithGoogle` (и при желании общий helper для auth catch): если `Error` и `code` — объект с `error_description` / `error` — брать их; иначе `message`, если не `[object Object]`; иначе `JSON.stringify`. Баннер Login уже показывает `auth.error` строкой.

### D4 — Skills / docs sync on apply

Обновить `client-work-with-auth` / `guards.md` и упоминания header login в branding skills при apply — чтобы канон skills совпал со specs.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Staff deep-link без сессии → Lobby как guest, не Login | Прямой `#/login` для upgrade; server всё равно режет staff HTTP |
| Fail silent guest: `error` в store, на Lobby нет баннера auth | Редкий путь; диагностика через store / прямой Login; не возвращаем на Login |
| Тесты SC-BRAND-22/24/31 и authGuard завязаны на Login | Переписать assertions в том же change |

## Migration Plan

1. Задеплоить client (Pages).
2. Ops отдельно: починить `GOOGLE_CLIENT_SECRET` на VPS (не часть этого change).
3. Rollback: revert client commit / previous Pages deploy.

## Open Questions

(нет — продуктовые решения зафиксированы в explore)
