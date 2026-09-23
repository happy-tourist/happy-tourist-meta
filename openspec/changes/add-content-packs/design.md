## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Dual answers/tasks (D1–D14) уже в runtime. Этот revision — **UX + visibility** модерации и редактора поверх того же HTTP/SQLite стека. Чеклист — `tasks.md` §7+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- Статусы и треды очевидны автору на страницах карточек и заданий.
- Staff hub зеркалит author: список task set → nested tasks page; approve на обеих.
- Тихий autosave; submit только когда есть что слать; per-set «нужна модерация».
- Пока answers pending у A — только A доотправляет answers; другие не сабмитят tasks; A может сабмитить tasks.

**Non-Goals:**

- Новый transport / deps; partial tasks submit; смена catalog gate / approve order.

## Decisions

### D1–D14: Dual flow (уже реализовано)

См. предыдущую версию design: type-scoped requests, dirty answers lock, staff answers hub, approve tasks→answers, mail. Ниже — только **добавления/правки**.

### D5′: Pending answers — кто сабмитит (revision)

- Пока pack **answers-pending** под автором A:
  - **Только A** может `submit/answers` (amend/resubmit).
  - **Другие** collection members MUST NOT `submit/tasks` (и UI disabled): tasks опираются на ответы, чужой tasks pending при чужих answers pending запрещён.
  - **A** MAY `submit/tasks` при своём answers pending (чтобы staff мог approve tasks→answers).
- Пока **tasks-pending** под автором T: только T может `submit/tasks` (как раньше `pack_pending_other`).
- Dirty answers lock на create/edit tasks — без изменений.

### D15: Tasks dirty + per-set marks (server)

- Хранить last successful **tasks** submit snapshot (симметрия answers).
- `tasksDirty` pack-level: draft tasks отличаются от snapshot.
- Per task-set flag `needsModeration` (или эквивалент в draft GET): set изменился относительно snapshot и ещё не включён в успешный tasks submit.
- `POST submit/tasks` по-прежнему отправляет **весь** список task sets пака; после успеха snapshot обновляется → все per-set marks сбрасываются.
- Draft GET / put возвращают flags + переживают F5.

### D16: Request status on draft GET

Draft (и при необходимости staff preview) отдаёт для каждого типа:

| Поле | Смысл |
|------|--------|
| status | `none` \| `pending` \| `rejected` \| `approved` (последний завершённый цикл или открытый rejected/pending) |
| requestId | если есть открытый pending/rejected thread |
| changeAuthorId / is*Author | для lock UI |

Клиент показывает подписи: на модерации / отклонено (нужна доработка) / одобрено — на страницах карточек и заданий. Каталог не показывает draft-статусы.

### D17: Threads embedded on editor pages

- Внизу **набора карточек**: тред **answers** (+ reply при pending|rejected).
- Внизу **заданий**: тред **tasks**.
- Отдельный route `/moderation` может остаться deep-link из mail; основной UX — embedded.
- Reject comment MUST быть виден в треде на соответствующей странице (SC-PACK-19).

### D18: Client UX polish

- Убрать caption «Сохранение…»; не менять layout при quiet autosave.
- `:loading` на кнопках add/save/submit (и staff actions).
- Submit answers disabled если `!answersDirty` или <2 cards; submit tasks если `!tasksDirty` или minima не выполнены.
- Добавление/сохранение вопроса disabled без ≥1 filled slot.
- Create task set: `q-tooltip` когда disabled из‑за dirty answers / нет cards («сначала отправьте карточки на модерацию»).
- i18n title страницы: **«Набор карточек»** (не «набор ответов»); title/description pack остаются на этой странице.

### D19: Staff UI = author shape

```
Staff queue --> cards hub (answers preview + task-set LIST with status marks)
                  \--> staff tasks page (preview questions/slots;
                       approve/reject/cancel tasks + tasks thread)
Cards hub: approve/reject/cancel answers + answers thread
```

- Не разворачивать все задания на hub.
- Approve order без изменений (D13).

### D20: Mail links

- Письма ведут на страницу карточек или заданий (hash) с видимым тредом/статусом, не только на голый `/moderation`.

### D11′ / D14: Docs + prerequisites

- Обновить skills/AGENTS hints под статусы, threads, staff nested page, «Набор карточек».
- Explore D*/Q* закрыты; новых внешних сервисов нет.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Чужой не может слать tasks при answers pending | Documented; A still submits tasks; i18n hint |
| Per-set marks vs pack submit | Marks UX-only; submit snapshot clears all |
| Rejected без pending id | D16 status `rejected` + requestId для треда |

## Migration Plan

- Server schema snapshot/flags → client UI → mail link tweak.
- Dev: существующие drafts без tasks snapshot = dirty until first tasks submit или empty baseline.

## Open Questions

- Нет (закрыты explore 2026-09-23).

Чеклист — `tasks.md` §7+.
