## Why

Админ/модератор после правок опубликованных карт и наборов видит ложный «Черновик» и кнопку «На модерацию», хотя их правки должны сразу попадать в live без очереди. У автора кнопка «На модерацию» активна даже без изменений. В live-просмотре набора заданий лишний блок CSV; слоты в редакторе визуально не совпадают с peek при выполнении задания.

## What Changes

- На **опубликованном** контенте staff (admin|moderator), в том числе создатель, всегда идёт через staff-save: без кнопки модерации, изменения сразу в live.
- После staff-save при отсутствии open author request — убрать retained working, чтобы не висел badge «Черновик» (packs + maps).
- При **создании** (never-published) у staff кнопка «На модерацию» **остаётся** (очередь как у автора); draft badge для never-published ок.
- У автора Submit «На модерацию» disabled, пока нет изменений относительно загруженного состояния (и привычные minima / lock).
- В live **просмотре** набора заданий скрыть блок CSV; в staff Edit / editor surfaces CSV оставить.
- Слоты ответов при заполнении и в плитках заданий (`PackTaskTile`) — размеры/chrome едино с peek-слотами в игре.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: staff published path; working clear after staff-save; author dirty Submit; hide CSV on live view; slot chrome parity with peek.
- `content/maps`: staff published path (creator-staff); working clear after staff-save; author dirty Submit.

## Scope

- **Capability ID:** `content/packs`, `content/maps`
- **Пакеты:** client + server (HTTP staff-save / list status semantics)
- Контракт: существующие draft / staff-save / submit endpoints; без новых room messages
- Heal висящих draft — при следующем staff-save (без one-shot SQL migration)

## Out of scope

- One-shot SQL cleanup всех исторических orphan working
- Изменение take/approve/reject очереди модерации для non-staff авторов
- Авто-публикация never-published без Submit для staff
- CSV на cards editor / add-task-set (кроме уже существующих правил)
- Изменение peek gameplay / difficulty / slot count rules
- Soft-unpublish семантика сверх уже существующего staff-save на soft-unpublished live

## Impact

- Client: boot редакторов packs/maps; Submit dirty gate; Tasks live view CSV; slot chrome shared with peek look
- Server: `staffSavePack` / `staffSaveMap` reconcile `workingRevisionId`
- Delta: `content/packs`, `content/maps`

## References

- Explore 2026-09-30: staff draft bugs; D1 create keeps Submit; D2 published staff-save; D3 clear working after staff-save; CSV hide on view; slot sizes like peek
- Plan: staff edit / draft / submit fixes
- Main `openspec/specs/content/packs`, `openspec/specs/content/maps`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
