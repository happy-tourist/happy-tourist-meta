## Why

На поверхностях редактирования пака и набора заданий формы создания/правки ответов и вопросов занимают место над списком и (на add-task-set) ещё дублируют live-карточки ответов сверху. Пользователю нужен список с кнопкой «добавить» и знакомая модалка задания, близкая к peek на игровом поле. Отдельно: при заполнении слотов задания (compose и peek) нельзя повторно выбрать уже поставленную карточку ответа и она подсвечивается как «использованная» — такого ограничения быть не должно: одна карточка может занимать любое число слотов. Ещё: author-facing черновик должен появляться с первого содержательного изменения, сохранять готовность к отправке после reload и однозначно отличаться от уже отправленного контента; сейчас пустая заготовка видна слишком рано, а Submit после reload ошибочно блокируется.

## What Changes

- Inline-формы ответа и задания уезжают в модалки; над списком — кнопка добавления; Edit на плитке открывает ту же модалку.
- На add-task-set убирается верхний read-only дубль live-карточек; CSV остаётся над списком, а task-dialog использует centered slots и compact answer chips.
- Compose и peek разрешают повторно использовать одну карточку ответа в нескольких слотах без disable или consumed-подсветки.
- Add-task-set тихо сохраняет неполный never-live черновик с первого изменения, восстанавливает его после ухода/reload и показывает только автору; Cancel непубликованного цикла возвращает его в draft.
- Пустая техническая заготовка нового пака не видна в каталоге автора; первое содержательное изменение активирует author-only `draft`.
- Готовность к Submit хранится между сессиями; author edit снимает `pending` в `draft`, оставляет `needs_revision` видимым автору, но исключает изменённую работу из staff moderation до resubmit; never-published pack/task-set можно удалить в любом из этих author states.

## Scope

- **Capability ID:** `content/packs`, `game/board`
- **Пакет:** client (`happy-tourist.github.io`) — UX editor ответов, task-set editor, add-task-set; peek на Game; quiet autosave; persisted Submit readiness; delete never-published work; author-only draft visibility
- **Пакет:** server (`happy-tourist-server`) — persist pre-submit drafts; draft activation for new packs; persisted unsubmitted-change signal; pending→draft on author edit; dirty needs-revision moderation gate; authoritative discard/delete never-published work
- Compose UI / peek chip reuse без смены room peek protocol
- Один never-live add-task-set цикл на автора на пак (как сейчас)

## Out of scope

- Colyseus room/message/schema для peek (уникальность слотов на server и так не требуется)
- Редизайн плиток списка (`PackAnswerCardTile` / `PackTaskTile`)
- Новые поля ответа/задания, изменение approve semantics или CSV-формата
- Несколько параллельных never-live drafts одного автора на одном паке
- Soft-unpublish published set → «черновик» (это unpublished, не add-task-set draft)
- Hard-delete пака или набора заданий после их первой публикации; при author re-edit старая live-версия сохраняется до approve

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: UI compose модалки; chip reuse compose (SC-PACK-256…263); pre-submit never-live draft persist / restore / delete / author list draft (SC-PACK-264…268); **MODIFIED** author draft activation, persisted Submit readiness, pending→draft on edit и delete до первой публикации (SC-PACK-269…275)
- `game/board`: peek answer chips reuse без used-state (SC-BOARD-49) — уже в этом change

## Impact

- Client: compose pages / dialogs, Game peek, pack/task-set autosave + top delete, persisted Submit gate, list draft filter, vitest + skills
- Server: pack draft activation/list visibility; moderation dirty/status transitions; add-task-set put/get/ghost/cancel/discard; mocha
- Main specs: delta `content/packs` (+ уже существующие compose/peek deltas) и `game/board` SC-BOARD-49

## References

- Explore (чат): единый жизненный цикл автора — hidden empty shell → draft → pending / needs_revision → live; readiness после reload; delete до первой публикации
- `openspec/specs/content/packs/spec.md` (SC-PACK-107/108, cancel→draft 175…179/196/197, neverLive 188…190)
- Client/server skills: content-pages, content store, server work-with-routes / structure
