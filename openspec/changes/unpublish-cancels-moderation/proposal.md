## Why

При soft-unpublish пака открытые заявки на модерацию (часто add-task-set) остаются `pending`/`needs_revision`: автор видит призрак в «На модерации», а amend-flow ломается (`pack_not_public` → уход на обычную страницу набора). Нужно каскадно снимать заявки с модерации вместе со снятием пака с публикации.

## What Changes

- Soft-unpublish пака автоматически отменяет все открытые moderation requests по этому паку
- Каждому затронутому change author — одно RU email без ссылок (даже если отменено несколько его заявок)
- Confirm «Снять с публикации» предупреждает об отмене открытых заявок
- Republish не восстанавливает отменённые заявки

## Scope

- **Capability ID:** `content/packs`
- **Server:** HTTP soft-unpublish пака; каскадный cancel open requests; email change authors
- **Client:** текст confirm unpublish (предупреждение про заявки)
- Контракт API unpublish без смены URL; поведение ответа может включать сведения об отменённых заявках при необходимости apply

## Out of scope

- Soft-unpublish отдельного набора заданий (task set) и каскад по нему
- Письма при ручном Cancel автора/staff
- Запись system-message в тред отменённой заявки
- Ссылки в письме; восстановление заявки при republish
- Block/unblock, hard-delete, изменение staff queue UI сверх исчезновения отменённых строк

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: каскадный cancel open requests при staff soft-unpublish пака; email автору(ам); confirm copy; без воскрешения при republish

## Impact

- Server: логика `unpublishPack` + уведомления (существующий mail-канал moderation)
- Client: i18n/confirm soft-unpublish пака
- Авторские «На модерации» и staff queue очищаются от отменённых заявок без отдельного UI

## References

- Main spec: `openspec/specs/content/packs/spec.md` (SC-PACK-120…, email moderation, my-moderation)
- Explore: soft-unpublish оставляет open `task_set` → ghost «На модерации» + redirect с add-task-set
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
