## Context

См. `proposal.md` и delta `specs/content/packs/spec.md`.

Секции 1–18 уже в runtime. Этот revision — **модерация для автора + семантика reject**: очередь не теряет «нужна доработка»; три фазы статуса; авторский список «На модерации». Чеклист — `tasks.md` §19+.

Пакеты: **server** (`../happy-tourist-server`) + **client** (`../happy-tourist.github.io`) + meta AGENTS/skills.

## Goals / Non-Goals

**Goals:**

- Reject → остаётся на модерации; общий UI-статус «нужна доработка» у staff и автора.
- Staff queue: pending **и** rejected (open discussion); cancel снимает; approve снимает.
- Approve из rejected без обязательного resubmit (revision заявки).
- Автор: кнопка «На модерации» → список своих заявок → Edit карточек.
- Три фазы меток: ожидает отправки · на модерации · нужна доработка (task sets + answers/cards).

**Non-Goals:**

- Менять D1′ dirty locks, tasks-only hub layout (D34–D36), collection/live Edit polish.
- Чужие заявки в авторском списке.

## Decisions

### D1–D36: Prior (реализовано)

Dual flow, D1′ locks, tasks-only staff queue/hub (D34–D36), collection/live Edit. **SC-PACK-19 «reject leaves queue» superseded by D37.**

### D37: Reject stays in moderation («нужна доработка»)

```
reject + comment
  |
  +-- status = rejected (DB)
  +-- thread message = comment
  +-- staff queue STILL lists pack (pending OR rejected open)
  +-- author + staff UI label: «нужна доработка»
  |
cancel --> leave both queues; release locks
approve --> leave both queues; live update as today
```

- Discussion continues on the same request thread while rejected.
- Author MAY still amend draft and resubmit (rejected → pending) when locks allow; not required before approve (D39).

### D38: Staff queue includes rejected

```
listPendingPacks (rename conceptually: open moderation queue)
  |
  +-- status IN (pending, rejected) for answers and/or tasks-only
  +-- same dedupe as D34 (answers row prefers when both open)
  +-- row shows type + status label (на модерации | нужна доработка)
```

### D39: Approve from rejected without mandatory resubmit

```
approveRequest
  |
  +-- allow status pending OR rejected
  +-- apply request.revisionId snapshot (not post-reject unsaved draft edits)
  +-- if author edited after reject and wants those edits live --> must resubmit first
```

### D40: Author «На модерации» list

```
packs section (collection / catalog header)
  |
  +-- button «На модерации» (any authenticated change-author with open requests; always visible if useful, empty state OK)
  |
  v
author list: ONLY requests where caller is changeAuthorId
  status IN (pending, rejected) for answers and/or tasks
  one row per pack (answers and/or tasks open)
  click --> ContentPackEditorPage (cards) for that pack
```

- Parallel path: Edit from collection still works.
- Staff queue unchanged for roles; author list is separate route/API.

### D41: Three-phase status marks (answers + task sets)

```
Phase                    Trigger                              Label (RU)
-----------------------  -----------------------------------  ------------------------
ожидает отправки         dirty vs last submit (needsModeration / answersDirty)   Ожидает отправки на модерацию
на модерации             request status = pending             На модерации
нужна доработка          request status = rejected            Нужна доработка
```

- Task-set list: per-set mark when dirty; when tasks pending/rejected, show that phase on sets in scope of the open tasks request (at minimum all sets in the pending revision / list rows).
- Cards/answers: pack-level (or list header) uses the same vocabulary for answers cycle + dirty answers → «ожидает отправки».
- Clear mark after approve when not dirty (as SC-PACK-60).

### D42: Docs

- Skills/AGENTS: author moderation nav; reject stays; approve-from-rejected; three-phase i18n.

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Staff queue noise from old rejected | Only open rejected (not cancelled/approved); cancel clears |
| Approve stale revision after author edits | D39: document; resubmit for new snapshot |
| Dual open statuses on one author row | Show worst/both captions (pending vs needs-revision) briefly |

## Migration Plan

- Server queue + approve-from-rejected + author list → client marks/nav → mocha → docs.
- No DB migration (reuse `rejected` status).

## Open Questions

- Нет (D1–D6 explore closed; approve-rejected = request snapshot).

Чеклист — `tasks.md` §19+.
