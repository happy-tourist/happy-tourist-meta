# Документация экосистемы Happy Tourist

Канон для AI-агентов и разработчиков. Исходники живут в sibling-репозиториях `../happy-tourist.github.io` (client) и `../happy-tourist-server` (server) — не внутри meta.

## Как читать

0. [projects-map.md](projects-map.md) — карта сервис → sibling-путь; local YAML.
1. OpenSpec: [`openspec/config.yaml`](../openspec/config.yaml) — context, rules, apply/verify files.
2. Пакетный контекст runtime:
   - Client: `{client}/AGENTS.md` → `../happy-tourist.github.io/AGENTS.md`
   - Server: `{server}/AGENTS.md` → `../happy-tourist-server/AGENTS.md`
3. Skills и индекс агентов: [`.agents/AGENTS.md`](../.agents/AGENTS.md) (`.agents/skills/client/`, `.agents/skills/server/`, `openspec-*`).

Пути вида `{ключ}/…` и `happy-tourist-meta/docs/...` резолвить через [projects-map.md](projects-map.md) (+ `projects-map.local.yaml`) и child `project-map.md` (ключ `happy-tourist-meta`).

## Каталог docs (meta)

| Документ | Содержание |
|----------|------------|
| [projects-map.md](projects-map.md) | Карта сервис → sibling-путь (+ local YAML); таблица «Пути для OpenSpec / агентов» |

Общие code/test rules живут в пакетных `AGENTS.md` и skills (отдельного `docs/client/` / `docs/server/` нет).

OpenSpec: [`openspec/config.yaml`](../openspec/config.yaml); changes/specs в [`openspec/`](../openspec/). Domain-capability-map / information-system — опционально позже.

## Sibling runtime

| Ключ projects-map | Путь (от meta) | Документация / контекст |
|-------------------|----------------|-------------------------|
| `client` | `../happy-tourist.github.io/` | `{client}/AGENTS.md` — продукт, стек, UI-домены |
| `server` | `../happy-tourist-server/` | `{server}/AGENTS.md` — продукт, стек, rooms, auth |

OpenSpec-артефакты: [`openspec/`](../openspec/) в **happy-tourist-meta** (не в siblings). Конфиг: [`openspec/config.yaml`](../openspec/config.yaml).

Always-on инструкции meta: [`AGENTS.md`](../AGENTS.md).
