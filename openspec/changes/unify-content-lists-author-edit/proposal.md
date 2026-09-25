## Why

Наборы живут в трёх поверхностях (коллекция / каталог / «На модерации»), а карты — в одном общем списке со своими черновиками. Пользователям неудобно искать статусы и свои материалы; коллекция мешает модели «весь каталог доступен», а после publish авторы не могут дорабатывать контент через очередь. Нужен паритет списков, избранное вместо коллекции, author re-edit через модерацию и exclusive «взять в модерацию» для staff.

## What Changes

- **BREAKING:** убрать раздел коллекции и membership; lobby ведёт в общий список наборов; default-grant в коллекцию убрать.
- Один общий список наборов (как у карт) со статусами и фильтрами; у пользователей убрать nav «На модерации» (остаётся фильтр «на модерации» = своё).
- Избранное (звезда) для наборов; фильтры на картах без избранного.
- После publish автор / staff могут править; автор — только через модерацию; exclusive edit lock; авторы task set правят свой set.
- Staff: «Взять в модерацию» на строке очереди и внутри; без take нельзя модерировать; TTL как у Edit.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: общий список вместо коллекции; избранное; фильтры/статусы; убрать my-moderation nav у non-staff; author/task-set re-edit через очередь; edit lock для авторов; staff moderation take; add-task-set без коллекции (verified).
- `content/maps`: статусы pending/needs_revision в общем списке; фильтры (без избранного); убрать my-moderation nav у non-staff; author re-edit через модерацию после publish; edit lock для автора; staff moderation take.

## Scope

- **Capability ID:** `content/packs`, `content/maps`
- **Пакеты:** client + server (HTTP content API, UI списков/редакторов/staff queue)
- Client: раздел наборов (единый список, фильтры, звезда, live/editor ACL), раздел карт (статусы + фильтры), staff queue take, убрать коллекцию / author my-moderation для пользователей
- Server: list ACL без коллекции; favorites; author working-copy re-submit после publish; edit lock для non-staff editors; moderation take на request; удаление collection/default-grant путей
- Auth: verified registered для create/edit/submit/add-task-set/favorites; гости — только публичный каталог

## Out of scope

- Привязка packs/maps к `tourist-room` / выбор контента при create game / runtime peek из паков (отдельный change)
- Смена ролей staff/admin вне content ACL
- Hard-delete опубликованного пака/карты
- Block/unblock UI (серверные endpoints могут остаться)
- Новые внешние сервисы / npm-зависимости под I/O

## Impact

- Client: навигация lobby «Наборы», страницы списков/коллекции/my-moderation (non-staff), фильтры, звезда, editor/staff ACL
- Server: HTTP `/api/content/*` (packs list, favorites, collection removal, edit-lock, moderation take, maps list statuses); SQLite (favorites; moderation take fields; deprecate/stop using collections + default pack grants)
- Support `change_pack` select из каталога — без зависимости от коллекции
- Согласованные delta specs `content/packs` + `content/maps`

## References

- Explore-сессия 2026-09-25 (D1–D9, D2b–D5c)
- Main specs: `openspec/specs/content/packs/spec.md`, `openspec/specs/content/maps/spec.md`
- Sibling: `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`
- Meta: `docs/projects-map.md`, `.agents/AGENTS.md`
