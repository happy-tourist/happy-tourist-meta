## Why

Наборы жили в нескольких поверхностях; коллекция мешала модели «весь каталог доступен», а после publish авторы не могли дорабатывать контент через очередь. Первый проход уже дал общий список, избранное, author re-edit и staff take; follow-up закрыл статусы сетов, шапку, крошки, leave, View/Edit карт, ghost set и fan-out. Остались UX-дыры: крошки **под fixed elevated header**; «К наборам» на модерации при крошках; на мобиле бургер слева от space, а Acc/Theme/Logout дублируют правый кластер и теснят шапку.

## What Changes

- **BREAKING (уже в change):** убрать коллекцию и default-grant; единый список наборов; избранное; author/task-set re-edit; staff take.
- **Follow-up (уже в change):** статусы сетов; cancel→draft; шапка/бургер/крошки; Game leave; без in_catalog-бейджа; map View/Edit; never-published → Edit; ghost never-live set; fan-out fix; author draft→Edit.
- **FIX:** крошки в **page zone с offset шапки** (внутри layout page container), чтобы их не перекрывала fixed `q-header`.
- **FIX:** убрать «К наборам» с chrome модерации при крошках; крошки на staff moderation routes.
- **FIX:** на узком viewport, где уже есть section-меню (authenticated non-Game): бургер **справа**; Acc / Theme / Logout **внутри** бургера (секции + кластер). Game и auth/login — без этого сжатия (Theme на auth как сейчас; Game leave снаружи).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: база + ghost/fan-out; крошки offset; нет «К наборам» на moderation chrome; staff crumbs.
- `content/maps`: база + author draft→Edit; крошки offset.
- `ui/branding`: крошки offset; narrow burger rightmost + Acc/Theme/Logout in menu when section nav applies.
- `game/leave`: без изменений narrow-burger (Game не сжимаем).

## Scope

- **Capability ID:** `content/packs`, `content/maps`, `ui/branding`, `game/leave`
- **Пакеты:** client (этот follow-up); server без изменений в этом FIX
- Client: crumbs in page container; moderation crumbs + drop catalogNav; narrow burger right + fold Acc/Theme/Logout into menu where section burger already exists
- Auth: staff = moderator|admin (без смены ролей)

## Out of scope

- Привязка packs/maps к `tourist-room` / createGame / runtime peek
- Название набора по центру шапки на Game
- Сжатие chrome на Game (Leave/Theme остаются как сейчас)
- Новый бургер на auth/login (Theme остаётся в toolbar)
- Hard-delete опубликованного пака/карты (кроме каноничного delete unpublished)
- Block/unblock UI; новые npm/SaaS
- Смена SMTP/OAuth/ролей вне content ACL
- Перестройка live pack browsing (карточки)

## Impact

- Client: `App.vue` crumbs + narrow burger layout/menu; moderation pages drop `catalogNav`; vitest SC-BRAND-17…19, SC-PACK-193…195
- Server: нет обязательных изменений в этом FIX
- Delta: packs, maps, branding

## References

- Explore 2026-09-25 (D1–D21) + crumbs overlap / moderation «К наборам» / mobile burger Q1–Q2
- Main specs + sibling AGENTS + `docs/projects-map.md`
