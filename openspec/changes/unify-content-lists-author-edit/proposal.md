## Why

Наборы жили в нескольких поверхностях; коллекция мешала модели «весь каталог доступен», а после publish авторы не могли дорабатывать контент через очередь. Первый проход уже дал общий список, избранное, author re-edit и staff take. Остались UX-дыры: автор набора заданий не видит статусы модерации **внутри** пака; разделы и staff-модерация спрятаны в лобби/списках; после Cancel правок черновик плохо находится в общем списке; навигация «назад» дублирует нужные крошки; зелёный бейдж «В каталоге» шумит; карта при провале сразу даёт инструменты клеток вместо чистого просмотра.

## What Changes

- **BREAKING (уже в change):** убрать коллекцию и default-grant; единый список наборов; избранное; author/task-set re-edit; staff take.
- **Follow-up:** статусы модерации на строках наборов заданий внутри пака (автор сета + staff; pack creator без чужих).
- **Follow-up:** Cancel (staff или author) → working сохраняется, для автора сущность как **черновик** в общем списке; для остальных — прежний live-слепок; hard-delete автора — удаление.
- **Follow-up:** разделы (Наборы, Карты, Поддержка) и **Модерация** (staff) в общей шапке; бургер на мобилке; убрать секции с лобби и «Модерацию» из списков паков/карт.
- **Follow-up:** крошки под шапкой; убрать «К наборам» / «Вернуться» где крошки закрывают путь.
- **Follow-up / BREAKING UX:** на Game — «выйти из игры» справа; logo на Game тоже leave; session logout на Game нет. Название набора по центру — **out of scope**.
- **Follow-up:** убрать только бейдж статуса **«В каталоге»** (зелёный) в списках packs/maps; pending / needs_revision / draft / unpublished оставить.
- **Follow-up:** never-published пак/карта → сразу Edit; опубликованная карта → сначала View (автор, участники×туристы, без paint tools); Edit снаружи и внутри + exclusive lock; live пак — как сейчас для знакомства с содержимым.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: база + per-set статусы; cancel→draft; крошки; без in_catalog-бейджа; never-published → Edit.
- `content/maps`: база + cancel→draft; крошки; без in_catalog-бейджа; View vs Edit; never-published → Edit.
- `ui/branding`: шапка секций / staff / бургер / крошки.
- `game/leave`: leave справа + logo leave на Game.

## Scope

- **Capability ID:** `content/packs`, `content/maps`, `ui/branding`, `game/leave`
- **Пакеты:** client + server
- Client: статусы сетов; шапка/крошки/leave; без «В каталоге»-бейджа; map View/Edit; never-published → Edit; pack live browsing unchanged
- Server: per-set status; cancel→draft; list status без обязанности отдавать UI-бейдж in_catalog (поле может остаться для ACL)
- Auth: staff = moderator|admin для шапки «Модерация»

## Out of scope

- Привязка packs/maps к `tourist-room` / createGame / runtime peek
- Название набора по центру шапки на Game
- Hard-delete опубликованного пака/карты (кроме каноничного delete unpublished)
- Block/unblock UI; новые npm/SaaS
- Смена SMTP/OAuth/ролей вне content ACL
- Перестройка live pack browsing (карточки/сеты) — остаётся как сейчас

## Impact

- Client: списки (без green in_catalog badge), MapEditor view-only chrome, row→Edit для drafts, header/crumbs/leave
- Server: list/live status fields as needed for draft/pending; map view meta
- Delta: packs, maps, branding, leave

## References

- Explore 2026-09-25 (D1–D9) + Q/G/C + V1–V5 (no in_catalog badge; map View/Edit; never-published→Edit)
- Main specs + sibling AGENTS + `docs/projects-map.md`
