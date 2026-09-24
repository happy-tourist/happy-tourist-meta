## 1. Server — working copy, submit, approve

- [x] 1.1 Прочитать design D1–D2, delta `specs/content/packs` (SC-PACK-100…105, 114), skills `server-locate-change-points`, `work-with-database`, `work-with-routes`; зафиксировать точки schema / `lib/content` / `app.config`
- [x] 1.2 Заменить personal drafts на одну working copy у creator до live; migrate/drop `content_user_drafts`; verify SC-PACK-100/101 mocha
- [x] 1.3 Единый first-submit (cards + ≥1 task set), needs_revision + author Cancel; убрать hard-reject; verify SC-PACK-102/104/105 mocha
- [x] 1.4 Staff single Approve → catalog live (cards + tasks); reject approve без task set; verify SC-PACK-103 mocha
- [x] 1.5 Creator hard-delete unpublished (SC-PACK-114); убрать unpublish/republish/rebase/stale endpoints если есть

## 2. Server — post-publish add-task-set + staff lock

- [x] 2.1 Прочитать design D3, SC-PACK-106…113; skills `work-with-database`, `server-work-with-test`
- [x] 2.2 Add-task-set submit (verified + inCollection, только live card ids); Approve только set; needs_revision/Cancel; verify SC-PACK-108…110 mocha
- [x] 2.3 Staff live save без очереди; Edit без коллекции; exclusive lock + 409 второму; verify SC-PACK-111…113 mocha
- [x] 2.4 Из sibling server: `npm test` (zz-contentPacks) — зелёный для SC-PACK-100…114

## 3. Server + client — support change-pack

- [x] 3.1 Прочитать design D5, delta `specs/support/tickets` (SC-SUP-27…29); skills `work-with-routes` / `work-with-database` (support), `work-with-forms`
- [x] 3.2 Server: topic `change_pack` + `packId` validation (каталог); ссылка в тикете; mocha SC-SUP-27/28; обновить SC-SUP-19
- [x] 3.3 Client Support: тема + select каталога (title/description) + create; vitest SC-SUP-27…29; i18n

## 4. Client — ACL surfaces и staff Edit

- [x] 4.1 Прочитать design D6, SC-PACK-106/107/53; skills `client-locate-change-points`, `work-with-stores` (content), `work-with-pages`, `work-with-localization`
- [x] 4.2 Store/API: убрать drafts/stale/unpublish wrappers; working-copy + unified submit/approve/needs-revision/cancel; add-task-set; staff save + lock
- [x] 4.3 UI: unpublished creator editor; published non-staff add-only (без edit-списков); staff Edit + lock error; my-moderation оставить; i18n
- [x] 4.4 Vitest SC-PACK-100…114 (релевантные UI/store); из client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные

## 5. Client — staff Edit session, placement, stable hints (follow-up)

- [x] 5.1 Прочитать design D3/D6–D8, SC-PACK-115…119; skills `work-with-pages`, `work-with-stores` (content), `work-with-styles` / localization
- [x] 5.2 Staff Edit session: не release lock / не сбрасывать staffEditTarget при cards↔tasks; tasks persist → staff-save; unlock только при выходе из Edit; verify SC-PACK-115 vitest
- [x] 5.3 Скрыть «На модерации» при `auth.isStaff` (catalog + collection); staff hint без moderation copy; verify SC-PACK-116
- [x] 5.4 Live: add-task-set рядом с секцией «Задания»; убрать шапочную кнопку live и row playlist_add в коллекции; verify SC-PACK-117/118 (+ SC-PACK-66)
- [x] 5.5 Editor/Tasks/AddTaskSet: убрать скачущие caption (`questionNeedsSlot`, submit/tasks hints) — tooltip / reserved space; verify SC-PACK-119
- [x] 5.6 Из client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные

## 6. Server + client — staff soft-unpublish / republish

- [x] 6.1 Прочитать design D9, delta SC-PACK-120…125; skills `work-with-database`, `work-with-routes`, `work-with-pages`, `work-with-stores` (content), localization
- [x] 6.2 Server: soft-hide flag (inCatalog) + migrate live→true; `unpublish`/`republish` staff-only; public catalog/getLive/collect filter; staff catalog includes снятые; staff Edit soft-unpublished; mocha SC-PACK-120/122/123/124
- [x] 6.3 Client store/API: unpublish/republish wrappers + catalog/collection flags; i18n «Снято с публикации» / кнопки
- [x] 6.4 UI: staff кнопки в каталоге и внутри набора; коллекция gray disabled + label + trash; non-staff no navigate/deep-link; verify SC-PACK-121/125 vitest
- [x] 6.5 Support change_pack select только in-catalog; из server+client: `npm test` (+ client lint/typecheck) — зелёные для SC-PACK-120…125

## 7. Client — cascade yellow, slots everywhere, add-task-set thread

- [x] 7.1 Прочитать design D10–D12, delta SC-PACK-126…128; skills `work-with-pages`, `work-with-stores` (content), `work-with-styles` / localization / test
- [x] 7.2 Editor: видимый `cascade-gap-outline` на строке набора (CSS паритет с Tasks); verify SC-PACK-126 vitest
- [x] 7.3 Staff hub + AddTaskSet (+ любые другие списки вопросов без chips): слоты ответов на каждой строке; verify SC-PACK-127 vitest
- [x] 7.4 AddTaskSet: статус pending/needs_revision + moderation thread + reply (как Editor); load via existing moderation API; verify SC-PACK-128 vitest
- [x] 7.5 Из client: `npm run lint`, `npm run typecheck`, `npm test` — зелёные для SC-PACK-126…128

## 8. Server + client — pack unpublish everywhere, task-set soft-hide, live drill-in

- [x] 8.1 Прочитать design D9/D13–D16, delta SC-PACK-129…133; skills `work-with-database`, `work-with-routes`, `work-with-pages`, `work-with-stores` (content), localization / test
- [x] 8.2 Server: task-set published/inCatalog flag + migrate live sets → true; staff unpublish/republish set endpoints; reject unpublish when last published set; mocha SC-PACK-131
- [x] 8.3 Client store/API: set unpublish/republish wrappers + pack/set flags; i18n confirm / unpublished set labels
- [x] 8.4 UI pack: staff unpublish/republish + **confirm** in catalog, **collection**, and live; no unpublish in staff queue; verify SC-PACK-129 (+ confirm on existing catalog/live)
- [x] 8.5 UI live: set summary rows (count + difficulty 1/2/3) + drill-in to questions+slots; soft-unpublished set gray no-enter; staff Edit/Republish on row; verify SC-PACK-130/132
- [x] 8.6 UI Editor + Tasks: set unpublish (confirm) / republish on row and inside Tasks; disable unpublish if last published; verify SC-PACK-131/132
- [x] 8.7 AddTaskSet: answer tiles → rounded chips like Tasks; verify SC-PACK-133
- [x] 8.8 Из server+client: `npm test` (+ client lint/typecheck) — зелёные для SC-PACK-129…133

## 9. Server + client — slot text, author name, tasks back (follow-up 5)

- [x] 9.1 Прочитать design D11/D17–D19, delta SC-PACK-134…136; skills `work-with-database`, `work-with-routes`, `work-with-pages`, `work-with-stores` (content), localization / test
- [x] 9.2 Server: `previewPending` для `task_set` отдаёт live answer cards; mocha/vitest path for SC-PACK-134
- [x] 9.3 Server: task-set payload `authorDisplayName` (displayName иначе email local-part); live/draft/preview; mocha SC-PACK-135 (API)
- [x] 9.4 Client: staff hub (+ add-task-set/tasks) `slotLabel` резолвит текст карточки; нет «заполнен» при найденной карте; verify SC-PACK-134
- [x] 9.5 Client: live / editor / staff hub / Tasks — label «Набор заданий {n} от {name}» (+ coauthors); i18n; verify SC-PACK-135
- [x] 9.6 Client Tasks: back label не `pack.title` — «Вернуться» и/или `backToAnswers`; verify SC-PACK-136
- [x] 9.7 Из server+client: `npm test` (+ client lint/typecheck) — зелёные для SC-PACK-134…136
