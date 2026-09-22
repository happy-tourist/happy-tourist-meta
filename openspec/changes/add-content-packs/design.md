## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

В runtime уже есть v1: SQLite content tables, `/api/content/*`, единый `submitPack`, монолитный `ContentPackEditorPage`, staff queue, mail. Этот revision **перестраивает** submit/lock/UI на два потока (`answers` / `tasks`) без новых внешних сервисов.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills. Чеклист — `tasks.md` §4+.

## Goals / Non-Goals

**Goals:**

- Dual moderation requests + раздельные submit/approve/thread.
- Client: answers page + nested task-set pages; autosave; delete confirm; collection-first; staff answers hub.
- Lock: неотправленные изменения answers ⇒ deny create/edit tasks; staff видит pack только при answers pending.
- Сохранить identity gates, collection, block, mail, difficulty persist; tourist room не трогать.

**Non-Goals:**

- Room↔pack, peek runtime, media, hard delete, отдельная top-level staff tasks queue.

## Decisions

### D1: Transport = HTTP + JWT (без изменений)

- `createEndpoint` / `auth.middleware()`; Pinia `content` через `client.http`.
- Нет Colyseus room для packs.

### D2: Data model — добавить `type` на moderation request

Существующие `content_*` таблицы сохраняются. На `content_moderation_requests` (или аналог):

| Поле | Смысл |
|------|--------|
| `type` | `answers` \| `tasks` |
| status / change_author / pack_id / thread | как v1, **по одному pending на (pack, type)** |

- Live snapshot: approve **answers** обновляет live cards (+ pack title/description); approve **tasks** обновляет live task sets/tasks/slots.
- Публичный GET / каталог: pack в каталоге только если есть live answers **и** хотя бы раз был approve answers при наличии live tasks (gate на approve answers).
- Drafts: personal draft per (user, pack) может хранить обе части; submit берёт срез нужного типа.
- Alternative (два отдельных pack entity) — отвергнуто.

### D3: Identity gates (без изменений по ролям)

| Действие | Кто |
|----------|-----|
| Catalog / live view / collection add|remove | JWT (incl. anonymous) для list/add; unauth 401 на mutate collection |
| Create / edit / submit answers или tasks | non-anonymous + `emailVerified` + pack в коллекции |
| Staff | `moderator` \| `admin` |

### D4: Submit validation (раздельная)

**Answers submit:**

- ≥2 answer cards с non-empty content.
- Удаление card / пустые слоты у tasks **не** блокируют answers submit.
- Смена content referenced card → очистка зависимых слотов в drafts (как SC-PACK-07); это влияет на tasks submit, не на answers.
- Answers submit **разрешён без** live/pending tasks (первый цикл: сначала answers, потом tasks).

**Tasks submit:**

- В draft уже есть ≥1 answer card и answers **не dirty** (см. D5).
- ≥2 tasks; каждый ≥1 filled slot; difficulty ∈ {1,2,3}.
- Reminder/badge на answers page **не** используются.

### D5: Lock + race (dirty answers)

- **Dirty answers:** title/description или набор cards отличается от последнего успешно submitted answers snapshot (или «пустой baseline» до первого submit). Любое добавление/изменение/удаление card или правки meta → dirty.
- Пока answers dirty: **никто** не создаёт и не редактирует tasks (server 403/409 + client disable nested editor). Список task set на answers page может быть виден read-only.
- После успешного **submit answers** dirty сбрасывается → create/edit tasks разрешены (даже если answers ещё pending).
- Повторная правка answers снова ставит dirty и снова лочит tasks, пока не будет новый submit answers.
- Pending-автор **answers** может править answers draft и resubmit.
- Пока tasks unlocked и tasks pending: автор tasks может amend/resubmit; race второго submit → 409, личный draft сохраняется.
- Два pending (answers + tasks) **могут** сосуществовать.
- Staff **cancel** answers/tasks снимает pending своего типа; cancel answers не обязан сбрасывать dirty (dirty считается от last successful submit snapshot).

### D6: Moderation thread + mail

- Отдельный thread на каждый request (type-scoped).
- Новый цикл после approve того же type → новый request + thread.
- Mail: approve/reject/staff message/block — как v1; links на SPA answers/moderation routes.

### D7: Co-author labels

- После approve **tasks** вклада — display label на task set. Прав нет.

### D8: Block

- Без изменений смысла: blocked виден везде; edit/submit deny; staff unblock.

### D9: HTTP surface (revision)

Префикс `/api/content/…` (точные пути при apply):

**User:** catalog; live GET; collection; create pack; draft get/put (cards / meta / task sets); `POST …/submit/answers`; `POST …/submit/tasks`; moderation get/post per type или per request id.

**Staff:** list pending **только packs с answers pending** (tasks-only не показывать); get answers hub (preview answers + nested task-set requests/live); approve/reject/cancel/message per request; block/unblock pack.

Ошибки: 401/403/409 + код для banner/i18n.

### D10: Client structure (revision)

```
Lobby --> content-collection --> catalog | create | edit
create --> answers page (title/desc, one card form, cards list,
            task-set list by author/coauthor, submit answers;
            task edit locked while answers dirty)
              \--> task-set page (one question form, slots +/-,
                   answer tiles, questions list, submit tasks)
Staff --> answers-pending queue --> answers hub
              \--> nested task sets (approve tasks first, then answers)
```

- Autosave: debounce PUT draft (разумный default ~500–1000ms).
- Delete: Quasar Dialog confirm.
- Slot UX: min 1 slot; +/−; click tile fills next empty; click slot clears.
- Store Pinia `content` расширить dual submit / pending flags / `answersDirty`.
- Нет badge «отправьте ответы» на answers page.

### D11: Skills / AGENTS

- После кода: обновить hints routes/pages/stores/database/tests под dual flow.

### D12: No new npm deps

- Autoseve/Dialog/routes — существующий стек.

### D13: Staff approve order + catalog gate

- Approve **answers** разрешён только если у pack уже есть **live** tasks (после approve tasks в этом или прошлом цикле).
- Approve **tasks** допускается, пока pack виден staff (т.е. answers уже pending в hub).
- После approve answers → pack в публичном каталоге (если не blocked).
- Повторные циклы: тот же порядок submit/approve.

### D14: Prerequisites

Все explore D* закрыты; новых внешних сервисов нет.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Answers pending без tasks долго | Staff видит hub; approve answers blocked until live tasks; author flow: submit answers → edit tasks |
| Dirty lock непонятен | Client disable + краткий i18n «сначала отправьте ответы» (без badge) |
| Сложнее lock API | `answersDirty` / last-submitted snapshot + D5 matrix в тестах |
| Миграция v1 единых pending | При apply: существующие pending трактовать/мигрировать или cancel; пустой prod ок |

## Migration Plan

- Deploy server (schema type + endpoints) → client.
- Dev/stage: сбросить или cancel старые single-type pending.
- Rollback: feature flag routes не обязателен; можно временно скрыть dual UI.

## Open Questions

- Точный debounce autosave (default 800ms) — не меняет SC.
- Нужен ли явный author cancel своего pending — default нет (только staff cancel), как v1.

Чеклист — `tasks.md`.
