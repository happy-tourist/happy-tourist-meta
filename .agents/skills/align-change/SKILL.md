---
name: align-change
description: >-
  Orchestrates align passes on an active OpenSpec change: align-implementability-back,
  align-implementability-front, then align-meta — each in its own subagent, each
  followed by the fix prompt «Добавь и поправь что считаешь нужным». Use when the
  user asks to align-change, run implementability + meta aligns on a change, or
  back → front → meta without pausing for review. Meta artifacts only — does not
  propose, apply, or edit runtime code.
---

# Align Change — align-back → align-front → align-meta

Оркестратор выравнивания **уже существующего** OpenSpec change: реализуемость server/client и качество meta. Parent **не** правит артефакты сам: делегирует шаги субагентам и гонит пайплайн до конца.

**Граница:** только meta (`openspec/`, docs/skills indexes при правках align). **Не** запускать `openspec-propose`, `openspec-apply-change`; не трогать runtime client/server.

## Когда применять

- Пользователь запускает `align-change` / просит прогнать align’ы по активному change
- Нужен end-to-end: implementability-back → implementability-front → align-meta без промежуточного review
- Артефакты change уже есть (после propose / update) — этот скилл их **не** создаёт

Не подменять одиночный `align-implementability-back` / `front` / `align-meta`, если пользователь явно ограничил одной фазой.

После прерывания продолжение — снова запустить этот скилл (`align-change`): он читает `.align-change-state.yaml` и резюмирует с сохранённой фазы.

## Жёсткое правило прерывания

**Прервать можно ТОЛЬКО если есть задачи которые невозможно выполнить без решения человека.**

Не останавливаться ради:

- «лучше спросить» / «на всякий случай подтвердить»
- мелкой неоднозначности, если есть разумный default из skills / sibling analogues / dialog
- желания показать промежуточный отчёт и ждать OK
- удобства parent-агента

Останавливаться только когда без выбора человека нельзя продолжить, например: нет/несколько active change без однозначного контекста; артефакты change отсутствуют; секрет/credential; конфликт требований без default; субагент вернул `HUMAN_BLOCKER`.

При таком стопе — записать state (ниже), кратко: что блокирует, какие варианты, что уже сделано; после решения человека — снова `align-change` (resume по state). Не продолжать следующие фазы пайплайна.

## State-файл (для resume)

Путь: `openspec/changes/<name>/.align-change-state.yaml` (от корня meta).

Писать/обновлять **в начале** (после выбора change), **после каждой** завершённой фазы, и **при** `HUMAN_BLOCKER`.

```yaml
change: <name>
phase: align-back | align-front | align-meta | done
blocker: null | "<short reason>"
updated: "<ISO-8601>"
```

- Старт / после выбора change → `phase: align-back`.
- После align-back → `phase: align-front`.
- После align-front → `phase: align-meta`.
- После align-meta → `phase: done`, `blocker: null`.
- При стопе → оставить текущую `phase`, заполнить `blocker`.

Не коммитить обязательность этого файла в продуктовый смысл change — оркестраторский маркер.

## Делегируемые скиллы (обязательно прочитать перед фазой)

| Фаза | Skill |
|------|--------|
| 1. Align back | [`.agents/skills/align-implementability-back/SKILL.md`](../align-implementability-back/SKILL.md) |
| 2. Align front | [`.agents/skills/align-implementability-front/SKILL.md`](../align-implementability-front/SKILL.md) |
| 3. Align meta | [`.agents/skills/align-meta/SKILL.md`](../align-meta/SKILL.md) |

Правила align живут в дочерних скиллах. Здесь — оркестрация, субагенты и override read-only.

## Вход / выбор change

Как в `implement-change` / apply:

1. Имя от пользователя, иначе из контекста диалога.
2. Иначе `openspec list --json` — автовыбор, если ровно **один** active change.
3. Иначе при 0 или >1 без однозначности — **прервать** (нужно решение человека).

Объявить: `Using change: <name>`.

Убедиться, что каталог `openspec/changes/<name>/` существует и есть базовые артефакты (хотя бы proposal/specs/design/tasks по схеме) — иначе `HUMAN_BLOCKER` (сначала propose / continue-change).

Пути runtime — через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML) — субагентам align нужны для evidence, не для правок runtime.

## Workflow

```
Align-Change Progress:
- [ ] 1. Select active change (parent)
- [ ] 2. Align-implementability-back → отдельный субагент (+ правка)
- [ ] 3. Align-implementability-front → отдельный субагент (+ правка)
- [ ] 4. Align-meta → отдельный субагент (+ правка)
- [ ] 5. Short final report
```

### 1. Подготовка (parent)

1. Выбрать change (выше) + при resume прочитать state.
2. Записать state `phase: align-back`, `change: <name>`.
3. Кратко передать субагентам: change name, projects-map paths, что уже сделано.

### 2. Align-implementability-back (отдельный субагент)

Один субагент:

1. Прочитать и выполнить [`align-implementability-back`](../align-implementability-back/SKILL.md) для этого change (передать имя change + paths projects-map).
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   То есть по отчёту hard/warnings/recommendations — внести правки в **meta** артефакты change (proposal/specs/design/tasks и связанные meta-файлы), которые субагент считает нужными; не ждать подтверждения. **Не** править runtime client/server.
3. Вернуть parent краткий итог: verdict + что поправлено, или `HUMAN_BLOCKER`.

### 3. Align-implementability-front (отдельный субагент)

Один субагент:

1. Прочитать и выполнить [`align-implementability-front`](../align-implementability-front/SKILL.md) для этого change.
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   Правки — только meta артефакты change (и связанные meta), не runtime. Не ждать подтверждения.
3. Вернуть parent краткий итог или `HUMAN_BLOCKER`.

### 4. Align-meta (отдельный субагент)

Один субагент:

1. Прочитать и выполнить [`align-meta`](../align-meta/SKILL.md) для этого change / ветки meta.
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   То есть по отчёту — поправить meta артефакты, индексы AGENTS/skills links, broken refs; не ждать подтверждения. Не runtime.
3. Вернуть parent краткий итог или `HUMAN_BLOCKER`.

**Override read-only** у align-скиллов: в рамках **этого** оркестратора субагент после отчёта **обязан** применить разумные правки по промпту выше (только meta). Не трогать секреты и не расширять scope за пределы findings. Не запускать propose / apply / commit / archive.

### 5. Финальный отчёт (parent)

Кратко на русском:

```markdown
# Align Change — итог

**Change:** <name>
**Align-back:** OK | HUMAN_BLOCKER | skipped
**Align-front:** OK | HUMAN_BLOCKER | skipped
**Align-meta:** OK | HUMAN_BLOCKER | skipped
**Stopped:** нет | <причина только human-blocker>
**Resume:** снова `align-change` (если Stopped ≠ нет; читает state)
**Next:** review артефактов → `/openspec-apply-change` или `implement-change`
```

## Субагенты — общие правила

- Каждый шаг 2/3/4 — **новый** субагент; не смешивать фазы в одном.
- Передавать полный контекст: change name, абсолютные/map-пути meta/client/server, краткий итог предыдущих фаз.
- `run_in_background: false` — ждать результат перед следующей фазой.
- Не резюмировать длинно вывод субагента пользователю между фазами — только прогресс одной строкой, затем следующая фаза.
- Порядок фаз **фиксирован**: back → front → meta (не переставлять).

## Не делать

- Не пропускать align-фазы «чтобы быстрее».
- Не запускать propose / apply / implement-change / commit / archive в этом скилле.
- Не править runtime client/server из parent или из align-субагентов align-change.
- Не ослаблять safety (секреты, force push).
