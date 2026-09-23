## Why

После UX-polish модерации всплыли дыры: staff после approve tasks остаётся на пустой странице с «набор не опубликован»; ложные метки «нужна модерация»; Edit доступен вне своей коллекции; dirty-answers глушит задания даже у автора pending; нет удаления неопубликованного автором; кнопки block на hub путают. Нужен follow-up polish + узкий author-delete без смены домена.

## What Changes

- Staff: после approve tasks → redirect на answers hub; из списка task set убрать уже промодерированные; не держать пустую nested-страницу / `pack_not_public`; убрать лишнюю кнопку «Открыть задания».
- Починить sticky `needsModeration` после approve (`copyRevision`/snapshot).
- Edit только из «Моей коллекции» (не на live/каталоге).
- Убрать из коллекции — с подтверждением.
- D1′: при dirty answers, если answers **pending** у A — **A MAY** create/edit tasks; остальные по-прежнему нет. Без pending + dirty — lock для всех.
- Убрать кнопки block/unblock с UI (API можно оставить; block anytime / staff delete — позже).
- Автор (`createdBy`) на **неопубликованном** паке: удалить весь пак (wipe + cascade заявок) или один task set (в т.ч. из liveTasks, даже если tasks уже approved).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: staff post-approve navigation; marks fix; collection-only edit; confirm remove; pending-author task edit under dirty answers; hide block UI; author delete unpublished pack / task set

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- Сохранить dual submit, catalog after answers approve, staff answers-only queue, approve order tasks→answers
- Server: snapshot/id coherence после approve; delete unpublished + cascade; D1′ lock; staff preview list filter
- Client: staff redirect/list; edit gate; confirm remove; delete UX; hide block buttons

## Out of scope

- Room↔pack, peek runtime, media, transfer ownership
- Staff/public hard delete опубликованного; block anytime UI; отдельный «block tasks»
- Partial tasks submit; смена порядка approve
- Отдельная staff-очередь «только tasks»

## Impact

- Client: ContentPack* / ContentStaff* / ContentCollection*, Pinia content, i18n
- Server: `lib/content.ts` approve/detach/delete + mocha SC-PACK
- Meta: skills/AGENTS hints

## References

- Explore 2026-09-23 (follow-up): D1′, D2′, D5′=A, D6, D7, cascade
- Prior sections 1–9 implemented
- Карта путей: `docs/projects-map.md`
