## 1. Auth guard → Lobby, never Login

- [ ] 1.1 Перед работой: `design.md` D1; delta `specs/auth/login` SC-AUTH-11b/12; skill `client-work-with-auth/guards.md`; текущий `authGuard.ts` + `authGuard.test.ts`
- [ ] 1.2 В `authBeforeEach`: staff/admin без сессии → `/lobby` (без anonymous на hop); silent-guest fail / no-session → `/lobby`, если цель уже lobby — остаться без повторного Lobby-only redirect; ни одного return на `/login`
- [ ] 1.3 Обновить `authGuard.test.ts` под SC-AUTH-11b/12; поправить `guards.md` под новое поведение
- [ ] 1.4 В client: `npm test` (authGuard) + по необходимости точечный rerun

## 2. Header: убрать guest login

- [ ] 2.1 Перед работой: `design.md` D2; delta `specs/ui/branding` SC-BRAND-22/24/31; `AppHeaderChrome.test.ts` + разметка header login в `App.vue` (или эквивалент)
- [ ] 2.2 Удалить wide `header-login` и burger `header-burger-login` (и связанные click → login); guest cluster = theme only
- [ ] 2.3 Обновить vitest SC-BRAND-22/24/31 (и смежные) — guest без login control; registered logout без регрессии
- [ ] 2.4 В client: `npm test` (AppHeaderChrome / related)

## 3. Читаемая ошибка Google на Login

- [ ] 3.1 Перед работой: `design.md` D3; delta `specs/auth/login` SC-AUTH-05/16; `stores/auth.ts` `loginWithGoogle`
- [ ] 3.2 Добавить `formatAuthError` (structured `code` → `error_description` / `error`; не `[object Object]`) и использовать в catch `loginWithGoogle`
- [ ] 3.3 Vitest на форматтер / store catch (SC-AUTH-16): объект `{ error, error_description }` → читаемая строка в `auth.error`
- [ ] 3.4 В client: `npm test` (auth store / related) + `npm run lint` + `npm run typecheck`
