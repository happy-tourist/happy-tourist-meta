## Why

После tasks-only staff-очереди остаётся дыра для **автора**: нет раздела «что на модерации», в списке наборов заданий нет фаз статуса после submit, а **reject с комментарием** снимает заявку с модерации у staff, хотя идёт обсуждение («нужна доработка»). Нужен узкий revision: общая семантика статусов, очередь staff+author с rejected, approve из «нужна доработка» без обязательного resubmit.

## What Changes

- Reject + комментарий: заявка **остаётся на модерации**; у staff и автора статус **«нужна доработка»** (не пропадает из очереди).
- Cancel по-прежнему снимает с модерации; approve убирает из очередей.
- Staff MAY **approve** ту же заявку и из «нужна доработка» (случайный reject / без доработки); approve берёт revision заявки, не несохранённый черновик после reject.
- Автор: кнопка **«На модерации»** в разделе наборов → список **своих** открытых заявок (pending + нужна доработка) → клик **сразу в Edit** карточек (параллельно входу через коллекцию).
- Метки/статусы в три фазы для **task sets** и **карточек/answers**: ожидает отправки · на модерации · нужна доработка.
- Mocha + Traceability; краткие AGENTS/skills hints.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: reject keeps request in moderation with shared needs-revision status; staff queue includes rejected; author moderation list; three-phase status marks; approve from rejected without mandatory resubmit

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** server + client (+ meta hints)
- Сохранить dual submit, D1′ locks, tasks-only staff hub (D34–D36), live Edit / collection, no block UI
- Server: `listPendingPacks` (+ rejected open), approve from rejected, author «my moderation» list API
- Client: staff queue labels; author «На модерации» page; editor marks vocabulary (answers + task sets)

## Out of scope

- Seed baselines / изменить D1′ dirty edit locks
- Submit tasks при грязных answers
- Room↔pack, peek, media, transfer ownership
- Block/unblock UI (server retain)

## Impact

- Server: `listPendingPacks`, `approveRequest`/`rejectRequest` semantics, author list endpoint, mocha SC-PACK-19/30/73…
- Client: `ContentStaffPage`, author moderation page + nav, `ContentPackEditorPage` / tasks marks, store/i18n
- Meta: skills/AGENTS hints

## References

- Explore 2026-09-23: D1–D6 (reject stays / author queue / three-phase marks / approve-from-rejected / own requests only / cards+tasks)
- Prior sections 1–18 implemented (incl. tasks-only staff)
- Карта путей: `docs/projects-map.md`
