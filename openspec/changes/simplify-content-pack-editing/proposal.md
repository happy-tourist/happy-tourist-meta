## Why

Модель «личный черновик у каждого в коллекции + dual submit + гонки» даёт конфликты, stale-draft и путаницу с Edit. Нужна простая публикация: автор один раз собирает и сдаёт пак, после approve контент заморожен для non-staff; правки — только у модераторов (сразу в live) или через поддержку.

Follow-up после первой реализации: staff (moderator = admin) всё ещё видит «На модерации», при правке вопросов на опубликованном паке получает ошибку working-copy, а условные подсказки скачут высоту UI. Нужно довести staff Edit и add-task-set placement до заявленной модели.

## What Changes

- **BREAKING:** убрать персональные drafts / dual post-publish edit / foreign-pending co-edit; одна рабочая копия до первого апрува.
- После публикации: non-staff не правят карточки и существующие задания; только добавление нового task set (с модерацией этого set).
- Staff: полный Edit любого пака без коллекции, без модерации своих правок, exclusive lock (второй Edit → ошибка); **одна сессия Edit** на cards↔tasks (lock не сбрасывать между страницами); правки вопросов/карточек сразу через staff-save.
- Staff/admin **не** видят кнопку/nav «На модерации» (их изменения не идут в очередь автора).
- Add-task-set: кнопка **рядом с разделом «Задания»** на live; **без** кнопки в списке коллекции и **без** шапочной кнопки на live.
- Content-редакторы: условные подсказки (напр. «Выберите хотя бы один ответ в слоте», submit/tasks hints) **не меняют высоту** layout — tooltip на disable-контроле или статичное/зарезервированное место.
- Первая модерация: один submit (карточки + ≥1 task set), одна кнопка «Одобрить» / «Доработать»; hard-reject убрать; автор может Cancel.
- Support: тема «изменить набор карточек» + select из каталога (название/описание) + ссылка на пак в заявке.
- Убрать staff unpublish/republish / wipe-drafts / draftStale-rebase из недавно добавленного UX.

## Scope

- **Capability ID:** `content/packs`, `support/tickets`
- **Пакеты:** client + server (HTTP content + support UI/API)
- Коллекция / live / редакторы / staff queue / my-moderation
- Контракт HTTP: рабочая копия до live, единый approve, add-task-set submit, staff lock + live edit, удаление personal drafts API
- Follow-up: client staff Edit session + nav/placement + layout-stable hints (server lock API без смены контракта, кроме при необходимости TTL/heartbeat на обеих страницах)

## Out of scope

- Привязка наборов к tourist-room / peek-наградам
- UI product **block** пака (кроме косвенно через support-текст)
- Новые npm-зависимости / внешние сервисы
- Изменение ролей staff/admin вне ACL Edit (admin уже приравнен к moderator через `isStaff`)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: freeze после publish; одна рабочая копия; единый approve/доработать; add-only task set; staff live edit + lock **session across cards/tasks**; staff без my-moderation nav; add-task-set у секции «Задания»; layout-stable editor hints; удаление personal drafts / unpublish-republish / stale pull
- `support/tickets`: тема change-pack + select каталога + ссылка на пак

## Impact

- Client: Content* pages/store, i18n; Support create form (topic + pack select); staff Edit session; catalog/collection nav; live add-task-set placement; editor hint layout
- Server: `lib/content`, schema drafts/locks, content HTTP; `lib/support` topics + create payload
- Тесты: mocha SC-PACK* / SC-SUP*; vitest content + support (вкл. SC-PACK-115… follow-up)
- Specs: delta → sync в main после archive
- Код published-ux (drafts/stale/unpublish) снимается этим change

## References

- Explore-решения D1–D16 + follow-up staff UX / anti-jump (сессия openspec-explore)
- Main: `openspec/specs/content/packs/spec.md`, `openspec/specs/support/tickets/spec.md`
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
