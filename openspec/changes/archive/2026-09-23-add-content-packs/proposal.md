## Why

После author-moderation UX остаётся баг и дыры в редакторе карточек: удаление/смена текста ответа часто даёт `answers_dirty` на save (слоты чистятся клиентом → D1′ считает это правкой заданий), кнопка «Отправить ответы» остаётся недоступной, confirm говорит «из черновика» даже для каталога, а после cascade нет жёлтой подсветки затронутых заданий/наборов и статусы под заголовком расходятся со списком. Нужен узкий revision: cascade без ложного lock, copy confirm, highlight, слоты на строке задания, sync статусов.

## What Changes

- Save draft после смены **content** / удаления карточки, которая реально сбрасывает слоты: **не** отклонять как «правка заданий при dirty answers»; answers dirty и **submit answers** доступны (≥2 cards).
- Cascade слотов **только** если после delete/content сбросился ≥1 слот; смена **description** карточки — dirty answers без cascade; delete карточки вне слотов — задания не трогать.
- Confirm delete: «из опубликованного набора» только если набор в каталоге (`hasLive`); иначе «из черновика».
- После cascade: жёлтая подсветка **задания** и **набора**, пока есть что править (дыры); слоты на строке задания в списке заданий видны явно.
- Статус под заголовком страниц answers/tasks **дублирует** фазы меток списка (ожидает отправки / на модерации / нужна доработка).
- Mocha + Traceability; краткие AGENTS/skills hints.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: answer-card cascade save without false tasks-lock; description/unused-card no cascade; delete confirm by hasLive; yellow task/set highlight; task-row slot preview; page status sync with list marks

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** server + client (+ meta hints)
- Сохранить dual submit, D1′ (настоящие правки заданий при dirty), reject-stays / my-moderation / three-phase (D37–D41), no block UI
- Server: putDraft cascade vs `tasksChanging` / answers_dirty; SC-PACK-07+ mocha
- Client: confirm copy; cascade UX + yellow; task list slots; subtitle = list marks

## Out of scope

- Менять семантику D1′ для **ручного** edit tasks при dirty answers без open answers
- Staff hub slot preview (author editor only)
- Room↔pack, peek, media, block UI

## Impact

- Server: `putDraft` / slot clear side-effect vs lock; mocha SC-PACK-07/78…
- Client: `ContentPackEditorPage`, `ContentPackTasksPage`, store/i18n
- Meta: skills/AGENTS hints

## References

- Explore 2026-09-23: D1–D3 / Q1–Q4 (cascade lock, confirm hasLive, highlight task+set, description dirty no cascade, slots on task row, status sync)
- Prior sections 1–21 implemented
- Карта путей: `docs/projects-map.md`
