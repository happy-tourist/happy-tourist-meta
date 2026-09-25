## Why

Пробный набор карточек (например математика для дошкольников) создаётся staff/admin и публикуется в каталог, но игроки не видят его в своей коллекции, пока не добавят вручную. Нужна разовая выдача опубликованных default-паков всем текущим и каждому новому авторизованному пользователю, без повторного навязывания после осознанного удаления.

## What Changes

- Server выдаёт в коллекцию паки из env-списка id (только опубликованные / in-catalog).
- Одноразовый backfill всем уже существующим пользователям (включая anonymous).
- Одноразовая выдача каждому **новому** user при создании аккаунта (email/password, Google, anonymous).
- После удаления пака из коллекции система **не** возвращает его автоматически.
- Client UI не меняется: коллекция как сейчас.

## Scope

- **Capability ID:** `content/packs`
- **Пакет:** server (`happy-tourist-server`) — HTTP/auth lifecycle + content collection; env для списка pack id
- **Аудитория:** все JWT-пользователи, включая anonymous
- **Условие пака:** только live + in-catalog (не soft-unpublished, не draft)

## Out of scope

- Wiring паков в tourist-room / peek (stub «Правильно/Неправильно» без вопросов пака)
- Client UX изменений (баннеры, onboarding, принудительный add)
- Авто-возврат после remove; ledger opt-out таблица (не нужна при one-shot модели)
- Bulk-import контента / генерация заданий
- Выбор пака по `BOOTSTRAP_ADMIN_IDS` (только явные pack id в env)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: автоматическая одноразовая выдача default-паков в коллекцию (backfill + создание user); только опубликованные; без re-grant после remove

## Impact

- Server: boot backfill, hooks создания user, env `DEFAULT_CONTENT_PACK_IDS`, `.env.example` + AGENTS env table
- Существующие API коллекции (`GET/POST /api/content/collection`) без breaking-контракта; поведение наполнения коллекции меняется для default id
- Client: без обязательных правок (увидит паки в `GET /collection`)
- Ops: после деплоя указать pack id опубликованного пака в env

## References

- Explore-решения D1–D5 (B′): pack id в env; всем; no re-add; только in-catalog; backfill + once on create
- `openspec/specs/content/packs/spec.md` — коллекция, catalog, soft-unpublish
- Sibling: `happy-tourist-meta/docs/projects-map.md`; `../happy-tourist-server/AGENTS.md` (content packs, `BOOTSTRAP_ADMIN_IDS`)
