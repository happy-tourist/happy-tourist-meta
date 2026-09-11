## Why

Сейчас вход только через email/password или анонимного гостя. Для шашек в браузере нужен быстрый вход без формы: Google OAuth уже встроен в `@colyseus/auth`, осталось включить провайдер и одну кнопку «вход/регистрация в один клик».

## What Changes

- Добавляется вход через Google: одна кнопка на экране Login совмещает регистрацию и вход.
- Сервер принимает OAuth Google и выдаёт тот же JWT, что и для email/anonymous; gate комнат не меняется.
- Сессия после Google сохраняется при обновлении страницы так же, как после email-входа.
- Email/password и гостевой вход остаются доступными.

## Capabilities

### New Capabilities

- `auth/login`: вход и регистрация через Google (одна кнопка), сохранение сессии, совместимость с существующими режимами email/password и anonymous.

### Modified Capabilities

- (нет — main specs для auth пока отсутствуют)

## Scope

- **Capability ID:** `auth/login`
- **Пакеты:** client (`happy-tourist.github.io`) и server (`happy-tourist-server`)
- **UX:** Login — одна кнопка Google (register+login); Lobby/Game без изменений UX
- **Auth-контракт:** JWT от `@colyseus/auth` после OAuth; HTTP `/auth/provider/google` (+ callback); room `onAuth` по-прежнему через JWT
- **Пользователи:** создание или поиск по email из Google-профиля (дефолтный user store Colyseus); отдельное поле `google_id` не вводится
- Email/password и anonymous остаются

## Out of scope

- Другие OAuth-провайдеры (Discord, GitHub, …)
- Явный UI «привязать Google» к уже вошедшему аккаунту
- Удаление email/password или гостевого входа
- Смена пароля / forgot password / email confirmation
- Правила шашек, lobby listing, board sync
- Кастомная колонка `google_id` в схеме пользователей

## Impact

- Client: на Login появляется Google one-click; auth store получает ещё один способ получить JWT.
- Server: регистрация OAuth-провайдера Google; секреты Google в env; callback на API-хост (не на GitHub Pages).
- Identity: совпадение email с уже существующим email/password-аккаунтом приводит к входу в тот же пользовательский ряд (поведение встроенного store).
- Deploy: нужны Google Cloud OAuth clients + redirect URI на server origin (dev и prod).
- Skills / docs meta: канон auth расширяется Google после реализации.

## References

- Explore: Google one-click через `signInWithProvider` + встроенный OAuth callback `@colyseus/database`
- [Colyseus Auth Module — OAuth](https://docs.colyseus.io/auth/module)
- Sibling: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`
- Meta skills: `.agents/skills/client/client-work-with-auth`, `.agents/skills/server/server-work-with-auth`
