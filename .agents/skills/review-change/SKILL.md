---
name: review-change
description: >-
  Orchestrates full meta review of an active OpenSpec change: brief-server-scope
  → brief-client-scope, then align-implementability-back →
  align-implementability-front → align-meta — each in its own subagent, each
  followed by «Добавь и поправь что считаешь нужным». Use when the user asks to
  review-change, brief+align a change, or run постановка digest then
  implementability/meta aligns without pausing. Replaces former root
  brief-change / align-change. Meta artifacts only — does not propose, apply,
  or edit runtime code.
---

# Review Change — brief (server→client) → align (back→front→meta)

Оркестратор **разбора и выравнивания** уже существующего OpenSpec change: сначала digest постановки (server → client), затем реализуемость и качество meta (back → front → meta). Parent **не** пишет brief/align сам: делегирует субагентам и гонит пайплайн до конца.

**Граница:** только meta (`openspec/`, связанные docs/indexes при правках). **Не** запускать `openspec-propose`, `openspec-apply-change`, `implement-change`; не трогать runtime client/server.

Заменяет бывшие корневые оркестраторы `brief-change` и `align-change`. Одиночные package-скиллы (`brief-*-scope`, `align-implementability-*`, `align-meta`) остаются и вызываются напрямую, если нужна одна фаза.

## Когда применять

- Пользователь запускает `review-change` / просит brief+align / «прогони постановку и align» по активному change
- Нужен end-to-end: brief-server → brief-client → align-back → align-front → align-meta (+ правки meta по находкам) без промежуточного review
- Артефакты change уже есть (после propose / update) — этот скилл их **не** создаёт

Не подменять одиночный `brief-server-scope` / `brief-client-scope` / `align-implementability-*` / `align-meta`, если пользователь явно ограничил одной фазой или стороной.

После прерывания — снова `review-change`: читает `.review-change-state.yaml` и резюмирует с сохранённой фазы.

**Legacy state:** если есть `.brief-change-state.yaml` или `.align-change-state.yaml` без `.review-change-state.yaml` — мигрировать в review-state (map phase) и продолжить; старые файлы можно удалить после записи нового.

## Жёсткое правило прерывания

**Прервать можно ТОЛЬКО если есть задачи которые невозможно выполнить без решения человека.**

Не останавливаться ради «лучше спросить», мелкой неоднозначности с default, промежуточного OK, удобства parent-агента.

Останавливаться только когда без выбора человека нельзя: нет/несколько active change без контекста; нет артефактов; секрет; конфликт без default; субагент вернул `HUMAN_BLOCKER`.

При стопе — записать state, кратко: что блокирует; после решения — снова `review-change` (resume). Не продолжать следующие фазы.

## State-файл (для resume)

Путь: `openspec/changes/<name>/.review-change-state.yaml` (от корня meta).

```yaml
change: <name>
phase: brief-server | brief-client | align-back | align-front | align-meta | done
blocker: null | "<short reason>"
updated: "<ISO-8601>"
```

- Старт → `phase: brief-server`
- После brief-server → `phase: brief-client`
- После brief-client → `phase: align-back`
- После align-back → `phase: align-front`
- После align-front → `phase: align-meta`
- После align-meta → `phase: done`, `blocker: null`
- При стопе → текущая `phase` + `blocker`

Не коммитить обязательность этого файла в продуктовый смысл change — оркестраторский маркер.

### Legacy map

| Старый файл / phase | → review phase |
|---------------------|----------------|
| `.brief-change-state.yaml` `brief-server` | `brief-server` |
| `.brief-change-state.yaml` `brief-client` | `brief-client` |
| `.brief-change-state.yaml` `done` | `align-back` (brief уже пройден) |
| `.align-change-state.yaml` `align-back` | `align-back` |
| `.align-change-state.yaml` `align-front` | `align-front` |
| `.align-change-state.yaml` `align-meta` | `align-meta` |
| `.align-change-state.yaml` `done` | `done` |

## Делегируемые скиллы (обязательно прочитать перед фазой)

| Фаза | Skill |
|------|--------|
| 1. Brief server | [`.agents/skills/server/brief-server-scope/SKILL.md`](../server/brief-server-scope/SKILL.md) |
| 2. Brief client | [`.agents/skills/client/brief-client-scope/SKILL.md`](../client/brief-client-scope/SKILL.md) |
| 3. Align back | [`.agents/skills/server/align-implementability-back/SKILL.md`](../server/align-implementability-back/SKILL.md) |
| 4. Align front | [`.agents/skills/client/align-implementability-front/SKILL.md`](../client/align-implementability-front/SKILL.md) |
| 5. Align meta | [`.agents/skills/align-meta/SKILL.md`](../align-meta/SKILL.md) |

## Вход / выбор change

1. Имя от пользователя, иначе из контекста диалога.
2. Иначе `openspec list --json` — автовыбор, если ровно **один** active change.
3. Иначе при 0 или >1 без однозначности — **прервать**.

Объявить: `Using change: <name>`.

Убедиться, что `openspec/changes/<name>/` существует и есть базовые артефакты — иначе `HUMAN_BLOCKER` (сначала propose / continue-change).

Пути runtime — через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML) — субагентам для reuse/evidence, не для правок runtime.

## Workflow

```
Review-Change Progress:
- [ ] 1. Select active change (parent)
- [ ] 2. Brief-server-scope → субагент (+ правка)
- [ ] 3. Brief-client-scope → субагент (+ правка)
- [ ] 4. Align-implementability-back → субагент (+ правка)
- [ ] 5. Align-implementability-front → субагент (+ правка)
- [ ] 6. Align-meta → субагент (+ правка)
- [ ] 7. Short final report
```

### 1. Подготовка (parent)

1. Выбрать change + при resume прочитать state (или legacy → migrate).
2. Записать state текущей фазы, `change: <name>`.
3. Кратко передать субагентам: change name, projects-map paths, что уже сделано.

### 2–3. Brief phases (отдельный субагент каждая)

Для **brief-server**, затем **brief-client**:

1. Прочитать и выполнить соответствующий brief-skill для этого change (client-фазе передать краткий итог server).
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   По отчёту — правки только в **meta** артефакты change; не ждать подтверждения; **не** runtime.
3. Вернуть parent краткий итог или `HUMAN_BLOCKER`. Обновить state → следующая фаза.

Порядок brief **фиксирован**: server → client.

### 4–6. Align phases (отдельный субагент каждая)

Для **align-back**, затем **align-front**, затем **align-meta**:

1. Прочитать и выполнить соответствующий align-skill.
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   Правки — только meta; не ждать подтверждения; не runtime.
3. Вернуть parent краткий итог или `HUMAN_BLOCKER`. Обновить state.

Порядок align **фиксирован**: back → front → meta.

**Override read-only** у brief/align-скиллов: в рамках **этого** оркестратора субагент после отчёта **обязан** применить разумные правки по промпту выше (только meta). Не трогать секреты и не расширять scope за пределы findings. Не запускать propose / apply / commit / archive / implement-change.

### 7. Финальный отчёт (parent)

```markdown
# Review Change — итог

**Change:** <name>
**Brief-server:** OK | HUMAN_BLOCKER | skipped
**Brief-client:** OK | HUMAN_BLOCKER | skipped
**Align-back:** OK | HUMAN_BLOCKER | skipped
**Align-front:** OK | HUMAN_BLOCKER | skipped
**Align-meta:** OK | HUMAN_BLOCKER | skipped
**Stopped:** нет | <причина только human-blocker>
**Resume:** снова `review-change` (если Stopped ≠ нет; читает state)
**Next:** review артефактов (человек) → `/opsx-apply` или `implement-change`
```

## Субагенты — общие правила

- Каждый шаг 2–6 — **новый** субагент; не смешивать фазы в одном.
- Передавать полный контекст: change name, map-пути meta/client/server, краткий итог предыдущих фаз.
- `run_in_background: false` — ждать результат перед следующей фазой.
- Не резюмировать длинно вывод субагента пользователю между фазами — только прогресс одной строкой.
- Порядок фаз **фиксирован**: brief-server → brief-client → align-back → align-front → align-meta.

## Не делать

- Не пропускать фазы «чтобы быстрее».
- Не запускать propose / apply / implement-change / commit / archive в этом скилле.
- Не править runtime client/server.
- Не ослаблять safety (секреты, force push).
