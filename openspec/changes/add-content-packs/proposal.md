## Why

После dual-flow staff видит в очереди только answers-pending; заявка **только на задания** невидима (SC-PACK-40). Автор уже опубликованного набора не может осмысленно отправить на модерацию только задания: кнопка answers disabled (не dirty), а staff tasks-only не подхватывает. Нужен узкий revision: tasks-only в той же staff-очереди с тем же hub UX (ответы → список task sets), approve только tasks.

## What Changes

- Staff queue: показывать packs с **answers pending** и/или **tasks-only pending** (одна очередь).
- Tasks-only hub: тот же layout — сначала ответы (контекст всего набора, live), затем список наборов заданий; nested tasks для review.
- При tasks-only: **нет** approve/reject answers (не нужны); только approve/reject/cancel **tasks**.
- Answers pending path без ломки: по-прежнему answers hub + nested tasks; approve tasks before answers когда оба нужны.
- Mocha + Traceability; краткие AGENTS/skills hints.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: staff queue includes tasks-only pending; same hub layout with full pack context; tasks-only approve path without answers approve

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** server + client (+ meta hints)
- Сохранить dual submit, D1′ dirty locks (как уже в runtime), live Edit / collection, author delete unpublished, no block UI
- Server: `listPendingPacks` (+ preview hub entry for tasks-only)
- Client: staff list/hub — entry by answers **or** tasks-only request; hide answers approve when no answers pending

## Out of scope

- Seed baselines / false dirty/marks UI (explore отменён — текущее поведение dirty/marks ок)
- Смягчение dirty-without-pending task **edit** lock (оставить как в runtime / SC-PACK-15/36)
- Submit tasks при грязных answers
- Room↔pack, peek, media, transfer ownership

## Impact

- Server: `listPendingPacks`, staff preview/hub payload, mocha SC-PACK-30/40…
- Client: `ContentStaffPage`, `ContentStaffRequestPage` (+ tasks nested), store/i18n
- Meta: skills/AGENTS hints

## References

- Explore 2026-09-23: T1 same hub UX, T2 whole pack, T3 tasks-only approve; prior dirty/marks questions cancelled
- Prior sections 1–15 implemented
- Карта путей: `docs/projects-map.md`
