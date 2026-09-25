## Why

Наборы жили в нескольких поверхностях; коллекция мешала модели «весь каталог доступен», а после publish авторы не могли дорабатывать контент через очередь. Первый проход уже дал общий список, избранное, author re-edit и staff take; follow-up закрыл статусы сетов, шапку, крошки, leave, View/Edit карт, ghost set и fan-out; chrome FIX — крошки в page-container, moderation без «К наборам», narrow burger. Остаётся баг: после **Cancel** never-live **add-task-set** автор открывает Edit и видит пустой draft (только title/description пака), хотя слепок заявки в БД есть и ghost-строка его показывает.

## What Changes

- **BREAKING (уже в change):** убрать коллекцию и default-grant; единый список наборов; избранное; author/task-set re-edit; staff take.
- **Follow-up (уже в change):** статусы сетов; cancel→draft; шапка/бургер/крошки; Game leave; без in_catalog-бейджа; map View/Edit; never-published → Edit; ghost never-live set; fan-out fix; author draft→Edit.
- **FIX (уже в change):** крошки в page zone с offset; moderation без «К наборам»; narrow burger rightmost + Acc/Theme/Logout in menu.
- **FIX:** после Cancel (author|staff) never-live **add-task-set** GET/Edit MUST вернуть **тот же working слепок** (вопросы/слоты), что ушёл на модерацию — не пустой `taskSets: []` с live title/description пака.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: база + ghost/fan-out/crumbs; **cancel add-task-set сохраняет draft payload на GET Edit**.
- `content/maps`: без изменений в этом FIX.
- `ui/branding`: без изменений в этом FIX.
- `game/leave`: без изменений в этом FIX.

## Scope

- **Capability ID:** `content/packs` (этот FIX); maps/branding/leave — prior landed
- **Пакеты:** server (add-task-set GET/put/submit после cancelled); client только если store/UI не подхватывает restored draft
- Auth: staff = moderator|admin (без смены ролей)

## Out of scope

- Привязка packs/maps к `tourist-room` / createGame / runtime peek
- Hard-delete опубликованного пака/карты (кроме каноничного delete unpublished)
- Block/unblock UI; новые npm/SaaS
- Смена SMTP/OAuth/ролей вне content ACL
- Перестройка live pack browsing (карточки)
- Изменение семантики Cancel для pack-level / live set re-edit через `/draft` (уже covered SC-PACK-179 mocha)

## Impact

- Server: `lib/content.ts` `getAddTaskSet` (+ put/submit reuse cancelled revision); mocha SC-PACK-196…
- Client: smoke vitest / store only if needed after GET fix
- Delta: `content/packs`

## References

- Explore 2026-09-25: cancel add-task-set → empty draft (title/description only); ghost from cancelled revision vs GET open-only
- Main specs + sibling AGENTS + `docs/projects-map.md`
