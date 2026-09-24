## Context

См. `proposal.md` — Why. Сейчас `unpublishPack` только ставит `inCatalog: false`; open rows в `content_moderation_requests` не трогает. `getAddTaskSet` / live для non-staff требуют `inCatalog` → ghost «На модерации» и redirect на страницу набора. Mail: `notifyChangeAuthor` уже используется для approve / needs_revision / message / block; ручной `cancelRequest` писем не шлёт — так и оставляем.

## Goals / Non-Goals

**Goals:**

- В `unpublishPack` после успешного перехода in-catalog → soft-unpublished: cancel всех open requests по `packId`, затем один email на каждого distinct `changeAuthorId`
- Client confirm unpublish: предупреждение про отмену заявок
- Тесты server (каскад + mail) и client (confirm copy)

**Non-Goals:**

- Каскад при soft-unpublish task set
- Письма / тред при ручном cancel
- Ссылки в письме; revive при republish
- Смена URL API unpublish

## Decisions

### D1 — Cancel внутри `unpublishPack`, не отдельный endpoint

После `UPDATE … inCatalog: false` (только когда пак реально был in-catalog): выбрать open (`pending` | `needs_revision`) по `packId`, выставить `cancelled`. Early-return «уже unpublished» — без повторного cancel/mail.

Альтернатива: отдельный job — лишняя сложность.

### D2 — Один email на автора события

Собрать unique `changeAuthorId` среди только что отменённых; для каждого с email и не-anonymous — один `notifyChangeAuthor` без `<a href>`. Текст: пак снят с публикации, заявка(и) на модерации отменены. Несколько заявок одного автора → всё равно одно письмо.

### D3 — Без thread message

Отменённая заявка не в my-moderation; system-message не пишем.

### D4 — Client confirm i18n

Обновить copy confirm «Снять с публикации» (каталог / коллекция / live — где уже есть confirm): явно про отмену открытых заявок. Dialog cancel = no-op.

### D5 — Точки врезки

| Пакет | Место |
|-------|--------|
| server | `src/lib/content.ts` — `unpublishPack`; reuse `notifyChangeAuthor` / helpers open-by-pack |
| server | mocha рядом с существующими SC-PACK-120… unpublish tests |
| client | i18n + confirm unpublish (страницы/компоненты, где уже вызывается pack unpublish) |
| client | vitest на наличие confirm / предупреждение |

## Risks / Trade-offs

- [Risk] Письмо уходит, а `inCatalog` update упал после cancel → Mitigation: сначала update pack, затем cancel+notify в том же успешном пути; при ошибке mail — log, как у других notify (не откатывать unpublish)
- [Risk] Два автора с open requests → два письма — ок по product
- [Trade-off] Staged revision отменённой заявки остаётся в БД (как при ручном cancel) — без wipe в этом change

## Migration Plan

1. Deploy server (каскад + mail)
2. Deploy client (confirm copy)
3. Rollback: откат server убирает каскад; уже `cancelled` заявки не оживают

## Open Questions

Нет.
