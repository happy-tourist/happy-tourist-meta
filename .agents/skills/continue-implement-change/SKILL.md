---
name: continue-implement-change
description: >-
  Resumes an interrupted implement-change pipeline for an active OpenSpec
  change from the last phase/block (apply subagents → align → check → commit).
  Use when the user asks to continue-implement-change, resume implement-change,
  or continue after a HUMAN_BLOCKER / interrupted implement.
---

# Continue Implement Change — resume apply → align → check → commit

Продолжение [`implement-change`](../implement-change/SKILL.md) с **прерванного места**. Тот же пайплайн и субагенты; не начинать с нуля, если уже есть прогресс.

## Когда применять

- Пользователь запускает `continue-implement-change` / просит продолжить прерванный `implement-change`
- После `HUMAN_BLOCKER` (или обрыва сессии), когда решение человека уже дано или можно идти дальше

Если пайплайн ещё не стартовал — перенаправить на [`implement-change`](../implement-change/SKILL.md).

## Жёсткое правило прерывания

Как в `implement-change`:

**Прервать можно ТОЛЬКО если есть задачи которые невозможно выполнить без решения человека.**

Если пользователь в этом сообщении **снял** прошлый blocker (дал секрет/выбор/ответ) — считать blocker resolved и продолжать, не спрашивая снова то же самое.

## Общий контракт

1. Прочитать целиком [`implement-change`](../implement-change/SKILL.md) и следовать его фазам 3–7, делегируемым скиллам, правилам субагентов и финальному отчёту.
2. Не дублировать и не ослаблять safety `commit` / override паузы apply.
3. Parent по-прежнему **не** пишет runtime-код сам — только оркестрация + state.

## Вход / выбор change

1. Имя от пользователя, иначе из контекста (в т.ч. прошлый `Using change:` / state-файл).
2. Иначе `openspec list --json` — автовыбор при ровно одном active.
3. Иначе — **прервать**.

Объявить: `Using change: <name>` и `Resuming from: <phase>[ block N]`.

## State-файл

Путь (от корня meta):

`openspec/changes/<name>/.implement-change-state.yaml`

Формат — как в `implement-change` (поля `change`, `phase`, `apply_block`, `blocker`, `updated`, …).

- **Читать** в начале.
- **Обновлять** после каждой завершённой фазы/блока и при новом `HUMAN_BLOCKER` (те же правила записи, что у `implement-change`).
- Если state нет — **вывести** точку продолжения (ниже) и сразу создать/записать state с выведенной фазой.

## Вывод точки продолжения

Определить стартовую фазу в порядке приоритета:

1. **Явно от пользователя** («продолжи с align», «блок 2», …).
2. **State-файл**, если `change` совпадает и `phase` ∈ `apply|align|check|commit`.
3. **Контекст диалога**: последний отчёт implement/continue с `Stopped:` / `HUMAN_BLOCKER` / незакрытой фазой.
4. **Эвристика без state:**
   - есть `- [ ]` в `tasks.md` → `phase: apply`, `apply_block` = номер первого блока с pending;
   - все tasks `- [x]`, но working tree dirty (client/server/meta) и нет свежего успешного commit-отчёта в контексте → `phase: align`;
   - tasks done и tree clean / уже после align+check по контексту, но commit не завершён → `phase: commit`;
   - всё done (tasks complete + state `phase: done` или эквивалент в контексте) → кратко сказать «нечего продолжать», предложить archive / новый `implement-change` только если нужен полный re-run.

При конфликте state vs tasks (например state говорит `align`, но снова появились pending tasks) — **вернуться на `apply`** с первого pending-блока (код важнее устаревшего state).

Объявить выбранную точку до запуска субагентов.

## Workflow

```
Continue-Implement-Change Progress:
- [ ] 1. Select change + read implement-change
- [ ] 2. Load/infer resume point + write state
- [ ] 3. If apply: remaining blocks → subagents (skip completed blocks)
- [ ] 4. If due: align-code subagent (+ правка)
- [ ] 5. If due: check-changes subagent (+ правка)
- [ ] 6. If due: commit subagent
- [ ] 7. Short final report + state → done | blocker
```

### Resume semantics

| Resume `phase` | Что делать |
|----------------|------------|
| `apply` | Как implement §3: только блоки с pending, начиная с `apply_block` (или первого pending). Уже `- [x]` блоки не перезапускать. Затем align → check → commit. |
| `align` | Пропустить apply. Сразу §4 align (+ правка), затем check → commit. |
| `check` | Пропустить apply и align. Сразу §5 check (+ правка), затем commit. |
| `commit` | Пропустить apply/align/check. Сразу §6 commit. |
| `done` | Не перезапускать пайплайн. |

Промпты субагентов — **те же**, что в `implement-change` (включая дословные «Добавь и поправь что считаешь нужным» для align и check).

В prompt apply-субагента дополнительно передать:

- что это **resume** после interrupt;
- текст снятого blocker / ответ человека из текущего сообщения (если был), чтобы не упереться в то же самое.

### После каждой фазы

Обновить state (`phase` = следующая, или `done`, или текущая + `blocker` при стопе) — контракт записи из `implement-change`.

## Финальный отчёт

Как в `implement-change`, плюс строка:

`**Resumed from:** <phase>[ block N]`

## Не делать

- Не запускать полный apply «с блока 1», если resume-точка позже и pending в ранних блоках нет.
- Не пропускать align/check/commit после успешного дожима apply (если resume был в apply).
- Не архивировать change и не создавать PR.
- Не путать с [`openspec-continue-change`](../openspec-continue-change/SKILL.md) (артефакты proposal/specs/design/tasks, не runtime-пайплайн).
