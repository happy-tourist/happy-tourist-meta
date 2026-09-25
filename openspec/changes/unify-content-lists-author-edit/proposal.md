## Why

Наборы жили в нескольких поверхностях; коллекция мешала модели «весь каталог доступен», а после publish авторы не могли дорабатывать контент через очередь. Первый проход уже дал общий список, избранное, author re-edit и staff take; follow-up закрыл статусы сетов, шапку, крошки, leave, View/Edit карт. Остались UX-дыры: крошки сливаются с синей шапкой; Edit на карте не в title row; новый набор заданий на модерации не виден в списке «Задания», а статус open request красится на чужие live-сеты того же автора.

## What Changes

- **BREAKING (уже в change):** убрать коллекцию и default-grant; единый список наборов; избранное; author/task-set re-edit; staff take.
- **Follow-up (уже в change):** статусы сетов; cancel→draft; шапка/бургер/крошки; Game leave; без in_catalog-бейджа; map View/Edit; never-published → Edit.
- **Follow-up:** крошки **под** elevated header (в зоне страницы), чтобы ссылки не сливались с синей шапкой.
- **Follow-up:** Edit внутри map View — сверху, как Edit у набора карточек.
- **Follow-up / FIX:** open request помечает только сеты, относящиеся к заявке (не fan-out по `changeAuthorId` на все live-сеты автора).
- **Follow-up:** ещё не live add-task-set (pending / needs_revision / draft) — строка в списке «Задания» **только автору сета**, со статусами как у общего списка; клик → Edit (amend).
- **Follow-up:** любой author-facing черновик (draft / pending / needs_revision) у автора → сразу **Edit** (паки, карты); чистый published → View / live. Пак в общем списке при open **только** add-task-set → сначала live (список ответов), затем строка сета → Edit.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: база + per-set статусы без fan-out; ghost never-live set для автора; author draft→Edit; pack list + add-task-set → live first; крошки под header.
- `content/maps`: база + author draft/pending/needs_revision → Edit; Edit сверху в View; крошки под header.
- `ui/branding`: шапка; крошки **ниже** elevated header chrome.
- `game/leave`: leave справа + logo leave на Game (без изменений в этом follow-up).

## Scope

- **Capability ID:** `content/packs`, `content/maps`, `ui/branding`, `game/leave`
- **Пакеты:** client + server
- Client: крошки под header; map Edit top; ghost set row + navigation; author draft→Edit для packs/maps
- Server: per-set status keyed to request revision sets (не authorId fan-out); never-live set rows for set author on live pack payload
- Auth: staff = moderator|admin; staff не видят never-live ghost сеты на live (очередь модерации)

## Out of scope

- Привязка packs/maps к `tourist-room` / createGame / runtime peek
- Название набора по центру шапки на Game
- Hard-delete опубликованного пака/карты (кроме каноничного delete unpublished)
- Block/unblock UI; новые npm/SaaS
- Смена SMTP/OAuth/ролей вне content ACL
- Перестройка live pack browsing (карточки) — остаётся как сейчас, плюс ghost set rows для автора сета

## Impact

- Client: App crumbs placement; MapEditor Edit top; ContentPackPage ghost set + Edit route; list open routing
- Server: `withTaskSetModerationStatuses` / live pack payload merge of author never-live sets from open request revision
- Delta: packs, maps, branding

## References

- Explore 2026-09-25 (D1–D18) + Q/G/C + V1–V5 + crumbs/Edit/badge/ghost Q1–Q3
- Main specs + sibling AGENTS + `docs/projects-map.md`
