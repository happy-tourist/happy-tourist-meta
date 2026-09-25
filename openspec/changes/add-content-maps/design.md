## Context

См. `proposal.md` — Why. Сейчас UGC-модерация и каталог есть только у `content/packs` (`src/lib/content.ts`, `/api/content/*`, Pinia `content`, страницы Content*). Игровая геометрия — hardcoded `TOURIST_LAYOUT` в server `touristMove.ts` и client `GamePage` (этот change её не трогает).

Паттерны для зеркалирования: working copy до первого approve, unified submit, moderation request + messages, staff exclusive lock + staff-save, soft-unpublish/`in_catalog` + cascade-cancel open requests + mail, author delete unpublished, `my-moderation` + staff pending queue, verify gate на create/edit/submit.

## Goals / Non-Goals

**Goals:**

- Отдельная сущность map + HTTP API рядом с packs, без collection-таблицы.
- Client: раздел «Карты», paint editor, общая staff-очередь с type badge, «На модерации» для map.
- Persist working grid + seat config; валидация min starts = players × tourists на submit.

**Non-Goals:**

- Подключение map id к create-room / sync layout / move rules.
- Переиспользование pack draft endpoints «как есть» без map-specific payload (общий код — helpers/mail/queue shape, не один pack revision JSON).
- Title/description, block UI на client.

## Decisions

### D1 — Capability и API prefix

- **Choice:** `content/maps`, HTTP под `/api/content/maps…` (+ map id в moderation/my-moderation/staff pending).
- **Why:** тот же домен content UGC, что packs; не `game/board` (runtime).
- **Alt:** вложить maps в packs API — отвергнуто (другой payload, нет collection).

### D2 — Хранение

- **Choice:** отдельные SQLite tables через `ensureContentMapTables` (или расширение `ensureContentTables`): `content_maps` (id, createdBy, workingRevisionId, liveRevisionId, inCatalog, editLocked*, timestamps), `content_map_revisions` (grid JSON 10×10 cell enum, players, touristsPerPlayer), moderation requests с `type` включающим `map` (или отдельная таблица map requests с тем же status machine — предпочтительно **единая** `content_moderation_requests` с nullable `packId`/`mapId` + type `map`, либо параллельные columns; при apply выбрать один путь и не дублировать status enum).
- **Why:** packs уже на revision + request + messages; maps проще (один payload).
- **Alt:** одна polymorphic `content_items` — overkill для первого change.

### D3 — Grid encoding

- **Choice:** 10 строк по 10 символов/`cells[100]` с enum `.` | `1` | `*` | `7` (дырка / старт / task / finish) — согласовано с существующим LAYOUT alphabet, без требования позиции центра.
- **Why:** привычный канон; будущий wiring в game проще.
- **Alt:** RGB/цветовые коды — нет.

### D4 — Список без коллекции

- **Choice:** `GET /api/content/maps` возвращает: всем JWT — in-catalog (+ staff soft-unpublished); автору дополнительно own never-published. Нет `content_map_collections`.
- **Why:** D3draft explore; роль «дома черновика» = Maps list для автора.
- **Alt:** только my-moderation для pending — отвергнуто (потеря pre-submit drafts).

### D5 — Shared staff queue / my-moderation

- **Choice:** расширить `GET /api/content/staff/pending` и `GET /api/content/my-moderation` полями type `pack`|`map` (+ map summary). Client staff hub и author list показывают бейдж.
- **Why:** явный product choice «общая очередь».
- **Alt:** отдельный `/api/content/maps/staff` — отвергнуто сейчас.

### D6 — Client structure

- **Choice:** Pinia store `maps` (или расширение `content` с map namespaces — предпочтительно **отдельный `maps` store**, чтобы не раздувать packs store) + pages Maps list / create→editor / staff request map branch; lobby header «Карты». HTTP только через store + `client.http`.
- **Why:** packs store уже большой; maps lifecycle проще и параллелен.
- **Alt:** всё в `content.ts` — допустимо, если locate покажет меньший diff; design default = отдельный store.

### D7 — Editor UX

- **Choice:** palette selection + cell click (toggle same / replace other); top selects players & tourists 1–4; quiet autosave PUT draft; submit button; thread below (как pack editor). Mini preview = downscaled same cell colors (green/brown/yellow/hole).
- **Why:** product explore.
- **Alt:** drag paint — later.

### D8 — Soft-unpublish / mail / lock

- **Choice:** зеркало packs SC-PACK-120…124 / 137…141 semantics for maps (cascade-cancel open map requests; one RU mail/author without links; staff lock + staff-save; author freeze after first approve).
- **Why:** D5 + «как у наборов».
- **Alt:** упростить без cascade — отвергнуто.

### D9 — Prerequisites from explore (closed)

| ID | Resolution |
|----|------------|
| D1 game wire | out of scope |
| D2 seat minima | players×tourists; only start count |
| D2b/c | players 1–4, tourists 1–4 |
| D3draft | author unpublished on Maps page |
| D4 | approved visible to all JWT |
| D5 | soft-unpublish yes |

No open technical blockers for catalog+moderation scope.

## Risks / Trade-offs

- **[Risk] Shared queue schema coupling packs↔maps** → Mitigation: type discriminator + tests for mixed list; don’t break pack rows.
- **[Risk] Large packs-style surface area** → Mitigation: maps omit collection/add-task-set/dual submit; fewer endpoints.
- **[Risk] Future game wiring assumes fixed 48 tasks / center 2×2** → Mitigation: SC-MAP-28 keeps rooms on hardcoded layout; document grid alphabet reuse only.
- **[Trade-off] Separate maps store vs content store** → clearer boundaries vs two HTTP patterns; apply may merge if duplicate thin wrappers hurt.

## Migration Plan

- Boot: `ensure*Tables` create map tables / migrate moderation type enum to include `map`.
- Deploy server then client (new routes 404 until server up).
- Rollback: feature-flag not required; unused tables harmless; hide Maps nav if needed.

## Open Questions

(нет — product D* закрыты в explore)

Чеклист реализации: `tasks.md`.
