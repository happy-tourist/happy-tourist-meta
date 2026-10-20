---
name: brief-change
description: >-
  Orchestrates brief passes on an active OpenSpec change: brief-server-scope,
  then brief-client-scope — each in its own subagent, each followed by the fix
  prompt «Добавь и поправь что считаешь нужным». Use when the user asks to
  brief-change, brief client+server scope for a change, or server → client
  digests with meta fixes without pausing for review. Meta artifacts only —
  does not propose, apply, or edit runtime code.
---

# Brief Change — brief-server → brief-client

Оркестратор **разбора постановки** уже существующего OpenSpec change: digest server, затем client. Parent **не** пишет brief сам: делегирует субагентам и гонит пайплайн до конца.

**Граница:** только meta (`openspec/`, связанные docs/indexes при правках). **Не** запускать `openspec-propose`, `openspec-apply-change`, `align-change`, `implement-change`; не трогать runtime client/server.

## Когда применять

- Пользователь запускает `brief-change` / просит brief client+server по активному change
- Нужен end-to-end: brief-server → brief-client (+ правки meta по находкам) без промежуточного review
- Артефакты change уже есть (после propose / update)

Не подменять одиночный `brief-server-scope` / `brief-client-scope`, если пользователь явно ограничил одной стороной.

После прерывания — снова `brief-change`: читает `.brief-change-state.yaml` и резюмирует с сохранённой фазы.

## Жёсткое правило прерывания

**Прервать можно ТОЛЬКО если есть задачи которые невозможно выполнить без решения человека.**

Не останавливаться ради «лучше спросить», мелкой неоднозначности с default, промежуточного OK.

Останавливаться только когда без выбора человека нельзя: нет/несколько active change без контекста; нет артефактов; секрет; конфликт без default; субагент вернул `HUMAN_BLOCKER`.

При стопе — записать state, кратко: что блокирует; после решения — снова `brief-change` (resume).

## State-файл (для resume)

Путь: `openspec/changes/<name>/.brief-change-state.yaml` (от корня meta).

```yaml
change: <name>
phase: brief-server | brief-client | done
blocker: null | "<short reason>"
updated: "<ISO-8601>"
```

- Старт → `phase: brief-server`
- После brief-server → `phase: brief-client`
- После brief-client → `phase: done`, `blocker: null`
- При стопе → текущая `phase` + `blocker`

## Делегируемые скиллы (обязательно прочитать перед фазой)

| Фаза | Skill |
|------|--------|
| 1. Brief server | [`.agents/skills/server/brief-server-scope/SKILL.md`](../server/brief-server-scope/SKILL.md) |
| 2. Brief client | [`.agents/skills/client/brief-client-scope/SKILL.md`](../client/brief-client-scope/SKILL.md) |

## Вход / выбор change

1. Имя от пользователя, иначе из контекста диалога.
2. Иначе `openspec list --json` — автовыбор, если ровно **один** active change.
3. Иначе при 0 или >1 без однозначности — **прервать**.

Объявить: `Using change: <name>`.

Убедиться, что `openspec/changes/<name>/` существует и есть базовые артефакты — иначе `HUMAN_BLOCKER`.

Пути runtime — через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ local YAML) — субагентам для reuse-scan, не для правок runtime.

## Workflow

```
Brief-Change Progress:
- [ ] 1. Select active change (parent)
- [ ] 2. Brief-server-scope → отдельный субагент (+ правка)
- [ ] 3. Brief-client-scope → отдельный субагент (+ правка)
- [ ] 4. Short final report
```

### 1. Подготовка (parent)

1. Выбрать change + при resume прочитать state.
2. Записать state `phase: brief-server`, `change: <name>`.
3. Кратко передать субагентам: change name, projects-map paths, что уже сделано.

### 2. Brief-server-scope (отдельный субагент)

Один субагент:

1. Прочитать и выполнить [`brief-server-scope`](../server/brief-server-scope/SKILL.md) для этого change.
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   По отчёту (расхождения / пробелы / неоднозначности / контракт) — внести правки в **meta** артефакты change (proposal/specs/design/tasks и связанные meta), которые субагент считает нужными; не ждать подтверждения. **Не** править runtime client/server.
3. Вернуть parent краткий итог: что разобрано + что поправлено, или `HUMAN_BLOCKER`.

### 3. Brief-client-scope (отдельный субагент)

Один субагент:

1. Прочитать и выполнить [`brief-client-scope`](../client/brief-client-scope/SKILL.md) для этого change (передать краткий итог server-фазы).
2. Затем **в том же субагенте** выполнить промпт дословно:

   > Добавь и поправь что считаешь нужным

   Правки — только meta артефакты change (и связанные meta), не runtime. Не ждать подтверждения.
3. Вернуть parent краткий итог или `HUMAN_BLOCKER`.

**Override read-only** у brief-скиллов: в рамках **этого** оркестратора субагент после отчёта **обязан** применить разумные правки по промпту выше (только meta). Не трогать секреты и не расширять scope за пределы findings. Не запускать propose / apply / commit / archive / align-change.

### 4. Финальный отчёт (parent)

```markdown
# Brief Change — итог

**Change:** <name>
**Brief-server:** OK | HUMAN_BLOCKER | skipped
**Brief-client:** OK | HUMAN_BLOCKER | skipped
**Stopped:** нет | <причина только human-blocker>
**Resume:** снова `brief-change` (если Stopped ≠ нет; читает state)
**Next:** `align-change` (опционально) → review → `/openspec-apply-change` или `implement-change`
```

## Субагенты — общие правила

- Каждый шаг 2/3 — **новый** субагент; не смешивать фазы в одном.
- Передавать полный контекст: change name, map-пути meta/client/server, краткий итог предыдущей фазы.
- `run_in_background: false` — ждать результат перед следующей фазой.
- Не резюмировать длинно вывод субагента пользователю между фазами — только прогресс одной строкой.
- Порядок фаз **фиксирован**: server → client (контракт сначала, UI вторым).

## Не делать

- Не пропускать фазы «чтобы быстрее».
- Не запускать propose / apply / implement-change / align-change / commit / archive в этом скилле.
- Не править runtime client/server.
- Не ослаблять safety (секреты, force push).
