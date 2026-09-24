## Why

Модель «личный черновик у каждого в коллекции + dual submit + гонки» даёт конфликты, stale-draft и путаницу с Edit. Нужна простая публикация: автор один раз собирает и сдаёт пак, после approve контент заморожен для non-staff; правки — только у модераторов (сразу в live) или через поддержку.

Follow-up после первой реализации: staff (moderator = admin) всё ещё видит «На модерации», при правке вопросов на опубликованном паке получает ошибку working-copy, а условные подсказки скачут высоту UI. Нужно довести staff Edit и add-task-set placement до заявленной модели.

Второй follow-up: после упрощения убрали staff unpublish/republish, но модераторам нужна кнопка «Снять с публикации» — пак исчезает из публичного каталога; у владельцев в коллекции остаётся серая disabled-строка; войти и править могут только staff; «Опубликовать снова» одной кнопкой без новой модерации.

Третий follow-up (UX модерации/редактора): после удаления ответа жёлтая рамка видна на задании, но не на наборе в editor; в staff hub и add-task-set в списке вопросов нет слотов ответов; автор add-task-set из «На модерации» не видит статус «нужна доработка» и thread с комментариями staff.

## What Changes

- **BREAKING:** убрать персональные drafts / dual post-publish edit / foreign-pending co-edit; одна рабочая копия до первого апрува.
- После публикации: non-staff (включая creator) не правят карточки и существующие задания; только добавление нового task set (с модерацией этого set).
- Staff: полный Edit любого пака без коллекции, без модерации своих правок, exclusive lock (второй Edit → ошибка); **одна сессия Edit** на cards↔tasks (lock не сбрасывать между страницами); правки вопросов/карточек сразу через staff-save; Edit также для **снятых с публикации** паков.
- Staff/admin **не** видят кнопку/nav «На модерации» (их изменения не идут в очередь автора).
- Add-task-set: кнопка **рядом с разделом «Задания»** на live; **без** кнопки в списке коллекции и **без** шапочной кнопки на live.
- Content-редакторы: условные подсказки (напр. «Выберите хотя бы один ответ в слоте», submit/tasks hints) **не меняют высоту** layout — tooltip на disable-контроле или статичное/зарезервированное место.
- Первая модерация: один submit (карточки + ≥1 task set), одна кнопка «Одобрить» / «Доработать»; hard-reject убрать; автор может Cancel.
- Support: тема «изменить набор карточек» + select из каталога (название/описание) + ссылка на пак в заявке.
- Убрать wipe-drafts / draftStale-rebase из publish-ux.
- **Staff unpublish / republish (restore):** «Снять с публикации» в **каталоге** и **внутри набора**; публичный каталог без снятого пака; staff видят строку в общем каталоге с пометкой; коллекция — серая некликабельная строка + «Снято с публикации» (trash оставить); non-staff не открывают live/deep-link; «Опубликовать снова» одной кнопкой (без новой очереди модерации); `block` не смешивать.
- **Cascade yellow на набор:** после cascade пустых слотов жёлтая рамка видна и на **наборе заданий** в cards editor (не только на задании внутри Tasks).
- **Слоты в каждом списке вопросов:** на staff request hub, add-task-set, tasks editor, live — в строке задания видны слоты ответов (filled/empty).
- **Add-task-set amend UX:** тот же блок статуса + moderation thread + reply, что на cards editor; автор из «На модерации» видит needs_revision и сообщения staff.

## Scope

- **Capability ID:** `content/packs`, `support/tickets`
- **Пакеты:** client + server (HTTP content + support UI/API)
- Коллекция / live / редакторы / staff queue / my-moderation
- Контракт HTTP: рабочая копия до live, единый approve, add-task-set submit, staff lock + live edit, удаление personal drafts API
- Follow-up: client staff Edit session + nav/placement + layout-stable hints (server lock API без смены контракта, кроме при необходимости TTL/heartbeat на обеих страницах)
- Follow-up 2: staff unpublish/republish soft-hide + catalog/collection/live ACL + i18n/tests
- Follow-up 3: cascade set outline CSS; slot chips на всех task-list surfaces; add-task-set thread/status (reuse existing moderation messages API)

## Out of scope

- Привязка наборов к tourist-room / peek-наградам
- UI product **block** пака (кроме косвенно через support-текст); block API остаётся отдельным
- Новые npm-зависимости / внешние сервисы
- Изменение ролей staff/admin вне ACL Edit (admin уже приравнен к moderator через `isStaff`)
- Hard-delete опубликованного / снятого пака staff или автором

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: freeze после publish; одна рабочая копия; единый approve/доработать; add-only task set; staff live edit + lock **session across cards/tasks**; staff без my-moderation nav; add-task-set у секции «Задания»; layout-stable editor hints; удаление personal drafts / stale pull; **staff soft-unpublish / republish**; **cascade yellow на task-set row**; **slot chips во всех списках вопросов**; **add-task-set moderation thread + needs_revision status**
- `support/tickets`: тема change-pack + select каталога + ссылка на пак

## Impact

- Client: Content* pages/store, i18n; Support create form (topic + pack select); staff Edit session; catalog/collection nav; live add-task-set placement; editor hint layout; unpublish/republish controls + collection gray rows; Editor cascade CSS; StaffRequest/AddTaskSet slot rows + AddTaskSet thread UI
- Server: `lib/content`, schema visibility/unpublish flag, content HTTP unpublish/republish; `lib/support` topics + create payload; при необходимости — messages в GET add-task-set (или reuse `/moderation`)
- Тесты: mocha SC-PACK* / SC-SUP*; vitest content + support (вкл. SC-PACK-115… + unpublish SC-PACK-120… + SC-PACK-126…)
- Specs: delta → sync в main после archive

## References

- Explore-решения D1–D16 + follow-up staff UX / anti-jump + unpublish D1–D7 + moderation UX (cascade/slots/thread) (сессия openspec-explore)
- Main: `openspec/specs/content/packs/spec.md`, `openspec/specs/support/tickets/spec.md`
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
