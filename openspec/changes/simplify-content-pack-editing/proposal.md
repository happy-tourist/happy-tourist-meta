## Why

Модель «личный черновик у каждого в коллекции + dual submit + гонки» даёт конфликты, stale-draft и путаницу с Edit. Нужна простая публикация: автор один раз собирает и сдаёт пак, после approve контент заморожен для non-staff; правки — только у модераторов (сразу в live) или через поддержку.

Follow-up после первой реализации: staff (moderator = admin) всё ещё видит «На модерации», при правке вопросов на опубликованном паке получает ошибку working-copy, а условные подсказки скачут высоту UI. Нужно довести staff Edit и add-task-set placement до заявленной модели.

Второй follow-up: после упрощения убрали staff unpublish/republish, но модераторам нужна кнопка «Снять с публикации» — пак исчезает из публичного каталога; у владельцев в коллекции остаётся серая disabled-строка; войти и править могут только staff; «Опубликовать снова» одной кнопкой без новой модерации.

Третий follow-up (UX модерации/редактора): после удаления ответа жёлтая рамка видна на задании, но не на наборе в editor; в staff hub и add-task-set в списке вопросов нет слотов ответов; автор add-task-set из «На модерации» не видит статус «нужна доработка» и thread с комментариями staff.

Четвёртый follow-up: unpublish пака нужен staff во **всех** списках (вкл. коллекцию); confirm перед unpublish; soft-unpublish **целого task set** (≥1 опубликованный set остаётся); live — сводка наборов + drill-in к вопросам; снятый set серый без входа; answer tiles на add-task-set — скруглённые chips как у staff.

Пятый follow-up (модерация / атрибуция / навигация): на staff hub слоты вопросов показывают «заполнен» вместо текста ответа (особенно add-task-set — карточек нет в revision); на строках наборов нет имени автора; возврат из списка вопросов (live drill-in) подписан названием пака вместо «назад».

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
- **Staff unpublish / republish (restore):** «Снять с публикации» в **каталоге**, **коллекции** и **внутри набора**; confirm перед unpublish; republish one-click (без confirm); публичный каталог без снятого пака; staff видят строку в общем каталоге с пометкой; коллекция — серая некликабельная строка + «Снято с публикации» (trash оставить); non-staff не открывают live/deep-link; `block` не смешивать; **не** в staff moderation queue.
- **Soft-unpublish task set:** staff снимает **целый** набор заданий (не отдельные вопросы); нельзя снять единственный опубликованный set; снятый set — серый + «Снято…», войти нельзя; staff Edit + republish на **строке** и **внутри** страницы набора; confirm перед unpublish set; republish set one-click.
- **Live как editor:** ответы + список наборов (число заданий + разбивка по сложности 1/2/3); вопросы со слотами — только после клика по строке (drill-in / read-only или staff Edit).
- **Cascade yellow на набор:** после cascade пустых слотов жёлтая рамка видна и на **наборе заданий** в cards editor (не только на задании внутри Tasks).
- **Слоты в каждом списке вопросов:** на staff request hub, add-task-set, tasks editor, live drill-in — в строке задания видны слоты ответов; **заполненный слот MUST показывать текст карточки** (не placeholder «заполнен»); для `task_set` резолв как при просмотре live (live cards).
- **Add-task-set amend UX:** тот же блок статуса + moderation thread + reply, что на cards editor; автор из «На модерации» видит needs_revision и сообщения staff.
- **Add-task-set answer tiles:** picker карточек для слотов — скруглённые chips как на Tasks (не прямоугольные `q-btn`).
- **Имя автора на наборе заданий:** везде, где строка/заголовок набора — «Набор заданий {n} от {имя}»; имя = `displayName`, иначе local-part email; видно всем (вкл. публичный live); строки не схлопывать; coauthors рядом.
- **Назад из списка вопросов:** кнопка возврата к паку/карточкам MUST NOT быть названием пака; стабильная подпись («Вернуться» / как раньше «К карточкам»).

## Scope

- **Capability ID:** `content/packs`, `support/tickets`
- **Пакеты:** client + server (HTTP content + support UI/API)
- Коллекция / live / редакторы / staff queue / my-moderation
- Контракт HTTP: рабочая копия до live, единый approve, add-task-set submit, staff lock + live edit, удаление personal drafts API
- Follow-up: client staff Edit session + nav/placement + layout-stable hints (server lock API без смены контракта, кроме при необходимости TTL/heartbeat на обеих страницах)
- Follow-up 2: staff unpublish/republish soft-hide + catalog/collection/live ACL + i18n/tests
- Follow-up 3: cascade set outline CSS; slot chips на всех task-list surfaces; add-task-set thread/status (reuse existing moderation messages API)
- Follow-up 4: pack unpublish in collection + confirm; task-set soft-unpublish/republish (≥1 published); live set summary + drill-in; AddTaskSet rounded answer tiles
- Follow-up 5: staff slot chip content resolution; task-set author display name everywhere; Tasks back label without pack title

## Out of scope

- Привязка наборов к tourist-room / peek-наградам
- UI product **block** пака (кроме косвенно через support-текст); block API остаётся отдельным
- Новые npm-зависимости / внешние сервисы
- Изменение ролей staff/admin вне ACL Edit (admin уже приравнен к moderator через `isStaff`)
- Hard-delete опубликованного / снятого пака staff или автором
- Hard-delete опубликованного / soft-unpublished **task set** (подумаем позже)

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: freeze после publish; одна рабочая копия; единый approve/доработать; add-only task set; staff live edit + lock **session across cards/tasks**; staff без my-moderation nav; add-task-set у секции «Задания»; layout-stable editor hints; удаление personal drafts / stale pull; **staff soft-unpublish / republish** (pack + task set; collection controls; confirm on unpublish); **live set summary + drill-in**; **cascade yellow на task-set row**; **slot chips с текстом карточки** (вкл. staff `task_set` + live cards); **add-task-set moderation thread + needs_revision status**; **AddTaskSet rounded answer tiles**; **task-set author display name**; **Tasks back без названия пака**
- `support/tickets`: тема change-pack + select каталога + ссылка на пак

## Impact

- Client: Content* pages/store, i18n; Support create form (topic + pack select); staff Edit session; catalog/collection nav; live set list + drill-in; unpublish confirm + collection staff controls; task-set soft-hide UI; Editor cascade CSS; StaffRequest/AddTaskSet slot rows + thread UI + chip answer tiles; author attribution labels; Tasks back label
- Server: `lib/content`, schema pack `inCatalog` + **task-set published/inCatalog flag**; content HTTP unpublish/republish (pack + set); **authorDisplayName** (displayName / email local-part) на task sets; staff preview live cards для `task_set`; `lib/support` topics + create payload
- Тесты: mocha SC-PACK* / SC-SUP*; vitest content + support (вкл. SC-PACK-115… + 120… + 126… + 129… + **134…**)
- Specs: delta → sync в main после archive

## References

- Explore-решения D1–D19 + follow-up staff UX / anti-jump + unpublish + moderation UX + follow-up 4–5 (slots content / author name / back label) (сессия openspec-explore)
- Main: `openspec/specs/content/packs/spec.md`, `openspec/specs/support/tickets/spec.md`
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
