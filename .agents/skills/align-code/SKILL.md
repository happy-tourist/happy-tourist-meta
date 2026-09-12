---
name: align-code
description: >-
  Runs client-align-code + client-verify-code and server-align-code +
  server-verify-code for happy-tourist siblings, then prints a combined Client
  and Server report. Use when the user asks for align-code, full-stack align,
  or to audit both client and server against requirements and skill compliance
  in one pass.
---

# Align Code — client + server

Оркестратор: для **client** и **server** последовательно выполнить **align-code** и **verify-code**, затем выдать **единый отчёт** по обоим пакетам.

Скилл **только анализирует и отчитывается** — не правит runtime-код, не коммитит. Правки — только по явной просьбе после отчёта.

## Когда применять

- Пользователь запускает `align-code` / просит полный align по клиенту и серверу
- Нужен один проход: соответствие постановке (**align**) + compliance skills (**verify**) на обоих siblings

Не подменять собой одиночные `client-align-code` / `server-verify-code`, если пользователь явно ограничил scope одним пакетом.

## Делегируемые скиллы (обязательно прочитать и следовать)

| Пакет | Align | Verify |
|-------|-------|--------|
| Client | [`.agents/skills/client/client-align-code/SKILL.md`](../client/client-align-code/SKILL.md) | [`.agents/skills/client/client-verify-code/SKILL.md`](../client/client-verify-code/SKILL.md) |
| Server | [`.agents/skills/server/server-align-code/SKILL.md`](../server/server-align-code/SKILL.md) | [`.agents/skills/server/server-verify-code/SKILL.md`](../server/server-verify-code/SKILL.md) |

Правила осей, severity, scope diff и формат секций живут **в дочерних скиллах**. Здесь — только оркестрация, пути и сводный отчёт. Не дублировать и не ослаблять дочерние правила.

## Репозитории и пути

Разрешить пути от корня `happy-tourist-meta` через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ `projects-map.local.yaml` если есть):

| Ключ | Репозиторий | Default path | Skills root |
|------|-------------|--------------|-------------|
| `client` | `happy-tourist.github.io` | `../happy-tourist.github.io` | `.agents/skills/client/` |
| `server` | `happy-tourist-server` | `../happy-tourist-server` | `.agents/skills/server/` |

Если путь не найден — спросить пользователя, не угадывать.

Перед работой по пакету — локальный `{client|server}/AGENTS.md`.

## Вход

**Основной источник требований — активный OpenSpec change** в meta (`proposal` / `specs` / `design` / `tasks`). Paste и формулировки из диалога — только уточнения поверх change, не замена.

Принимать (и пробрасывать в оба align):

- имя OpenSpec change (если пользователь назвал)
- опционально: вставленный текст / уточнения из диалога

**Резолв OpenSpec change** (один раз на весь прогон, до Align):

1. Имя, переданное пользователем.
2. Иначе — из контекста диалога, если однозначно.
3. Иначе из meta: `openspec list --json` — автовыбор, если ровно **один** active change.
4. Иначе при нескольких — спросить, какой active change использовать.
5. Иначе при нуле active — спросить имя change или явно подтвердить audit без OpenSpec (`OpenSpec: none`, тогда только paste/диалог/repo).

Объявить: `Using OpenSpec change: <name>` (или `OpenSpec: none`). То же имя передать в client- и server-align. В дочерних align этот change — **primary evidence** для осей A/C (включая краевые случаи из SC-*/design); paste/диалог не перекрывают артефакты change без явного решения пользователя.

Если active change не резолвится и пользователь не дал fallback — спросить **один раз** до шага Align. Один ответ использовать и для client, и для server.

Опционально пользователь может сузить scope: `только client` / `только server` — тогда пропустить другой пакет и отметить это в отчёте.

## Hard Boundary

- Не править, не создавать, не удалять, не форматировать runtime-файлы.
- Не коммитить / не пушить.
- Дочерние align-скиллы остаются read-only (не `npm` для preservation analysis). Verify: агент сам запускает `npm` lint/test/build из корня client/server, исправляет сбои; report-only verify (без «исправить») — всё равно запускать tooling и включать результаты в отчёт.
- Align остаётся read-only; verify в этом оркестраторе — **report-only** (не verify-and-fix), если пользователь явно не попросил «исправить».

## Workflow

```
Align-Code Progress:
- [ ] 1. Resolve paths (projects-map + local)
- [ ] 2. Resolve requirements source (shared)
- [ ] 3. Client: read + run client-align-code
- [ ] 4. Client: read + run client-verify-code
- [ ] 5. Server: read + run server-align-code
- [ ] 6. Server: read + run server-verify-code
- [ ] 7. Combined report (Client + Server)
```

### 1–2. Paths and requirements

Из корня meta: projects-map (+ local). Проверить, что client/server — git work trees.

Собрать общий источник требований. **Primary = активный OpenSpec change** (резолв как в **Вход**), затем `openspec status --change "…" --json` из meta; артефакты change обязательны для Align осей A/C (краевые случаи / SC-*). Paste/диалог — вторичные уточнения, не основной канон.

### 3–4. Client

Рабочий cwd / git / чтение `src/…` — **корень client**. Skills читать из meta `.agents/skills/client/`.

1. Полностью выполнить `client-align-code` (все 4 оси, формат отчёта дочернего скилла).
2. Полностью выполнить `client-verify-code` (ветка vs merge-base + working tree; все Always-include skills).

Сохранить результаты для сводки; **не** публиковать отдельным финальным ответом до шага 7 (допустимы краткие прогресс-апдейты).

### 5–6. Server

То же для server: `server-align-code`, затем `server-verify-code`. Skills — `.agents/skills/server/`.

Client и server можно анализировать последовательно (предпочтительно: сначала client, потом server) или параллельно через субагентов, если так быстрее — но **сводный отчёт один**, в конце.

### 7. Combined report

Язык сводки: **русский**. Секции Align — как в дочерних (русский). Секции Verify — как в дочерних (client verify: English prose; server verify: Russian prose — не переписывать язык дочернего шаблона).

Выдать **один** отчёт:

```markdown
# Align Code — отчёт

Источник требований: активный OpenSpec change <name> (+ paste/диалог только как уточнения | none + fallback)
OpenSpec change: <name | none; passed | active | inferred>
Scope: client + server | client-only | server-only

## Сводка
| Пакет | Постановка | Код проекта | Тесты | Регрессии | Verify Violations | Verify Warnings |
|-------|------------|-------------|-------|-----------|-------------------|-----------------|
| Client | N/10 | N/10 | READY\|BLOCKED | нет\|N | N | N |
| Server | N/10 | N/10 | READY\|BLOCKED | нет\|N | N | N |

## Client

### Align
<полный отчёт client-align-code>

### Verify
<полный отчёт client-verify-code>

## Server

### Align
<полный отчёт server-align-code>

### Verify
<полный отчёт server-verify-code>

## Что делать дальше
- <hard / Violations сначала по обоим пакетам; затем Warnings; Recommendations последними>
- <выполненные npm-команды client/server и результаты, если уместны>
```

Если пакет пропущен (нет пути / user scope) — секция с одной строкой `пропущен: <причина>`, без выдуманных findings.

Пустые hard-секции Align опускать по правилам дочернего скилла. Пустые tier-секции Verify — `- none` / `- нет` по дочернему шаблону.

## Не делать

- Не изобретать параллельные критерии align/verify.
- Не смешивать findings client и server в один немаркированный список.
- Не считать «оба чистые» без реального прогона дочерних скиллов (если в пакете есть branch/working-tree изменения).
- Не запускать `commit` / правки docs skills из этого скилла.
