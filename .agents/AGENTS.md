# Индекс агента — happy-tourist-meta

Канонический каталог документации и skills для AI-агентов в метарепозитории Happy Tourist (онлайн-шашки).

Краткие always-on инструкции (Cursor): [`AGENTS.md`](../AGENTS.md) в корне репозитория.

Runtime-код: siblings  
[`../happy-tourist.github.io`](../happy-tourist.github.io) (Vue 3 + Quasar SPA) и  
[`../happy-tourist-server`](../happy-tourist-server) (Colyseus).  
Контекст пакетов:

| Пакет | AGENTS.md |
|-------|-----------|
| Client | [`../happy-tourist.github.io/AGENTS.md`](../happy-tourist.github.io/AGENTS.md) |
| Server | [`../happy-tourist-server/AGENTS.md`](../happy-tourist-server/AGENTS.md) |

---

## Документация

| Документ | Назначение |
|----------|------------|
| [`docs/projects-map.md`](../docs/projects-map.md) | Сервис → sibling-путь; local YAML; Child `project-map.md` (ключ `happy-tourist-meta`) |
| [`docs/README.md`](../docs/README.md) | Оглавление документации |

Локальные оверрайды путей: [`projects-map.local.yaml`](../projects-map.local.yaml) / шаблон [`projects-map.local.yaml.example`](../projects-map.local.yaml.example) в корне meta.

OpenSpec: [`openspec/config.yaml`](../openspec/config.yaml); артефакты в `openspec/changes/` и `openspec/specs/`.

---

## Skills

### OpenSpec / workspace (`.agents/skills/`)

| Skill | Когда |
|-------|--------|
| [`align-code`](skills/align-code/SKILL.md) | Client + server: `*-align-code` + `*-verify-code` → сводный отчёт |
| [`commit`](skills/commit/SKILL.md) | Status → stage → commit → **push** по meta + client + server (main/master) |
| [`check-changes`](skills/check-changes/SKILL.md) | Unstaged client/server → предложения: добавить/изменить skills, maps, docs, AGENTS.md |
| [`openspec-propose`](skills/openspec-propose/SKILL.md) | Новая задача: proposal → specs → design → tasks |
| [`openspec-new-change`](skills/openspec-new-change/SKILL.md) | Создать change и начать артефакты по схеме |
| [`openspec-continue-change`](skills/openspec-continue-change/SKILL.md) | Продолжить создание недостающих артефактов change |
| [`openspec-update-change`](skills/openspec-update-change/SKILL.md) | Правка существующих артефактов change без кода |
| [`openspec-apply-change`](skills/openspec-apply-change/SKILL.md) | Реализация по `tasks.md` |
| [`openspec-verify-change`](skills/openspec-verify-change/SKILL.md) | Проверка реализации против артефактов |
| [`openspec-archive-change`](skills/openspec-archive-change/SKILL.md) | Архивация закрытого change |
| [`openspec-explore`](skills/openspec-explore/SKILL.md) | Explore mode до/во время change |
| [`openspec-sync-specs`](skills/openspec-sync-specs/SKILL.md) | Delta → main specs без archive |

### Discovery (другие корни)

| Корень | Назначение |
|--------|------------|
| [`.agents/skills/client/`](skills/client/) | Client-wide skills (meta); runtime: `../happy-tourist.github.io/` |
| [`.agents/skills/server/`](skills/server/) | Server-wide skills (meta); runtime: `../happy-tourist-server/` |

#### Client-wide skills (`.agents/skills/client/`)

Стек: Vue 3 Composition API / `<script setup>`, Quasar 2, Pinia 4, vue-router 5 (hash), vue-i18n 11, `@colyseus/sdk` 0.18, TypeScript. Runtime: `../happy-tourist.github.io/`.

| Skill | Когда |
|-------|--------|
| [`client-align-code`](skills/client/client-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов |
| [`client-locate-change-points`](skills/client/client-locate-change-points/SKILL.md) | Где править / куда класть новые файлы |
| [`client-verify-code`](skills/client/client-verify-code/SKILL.md) | Проверка кода / compliance skills |
| [`colyseus-client`](skills/client/colyseus-client/SKILL.md) | `client.http` + room messages (не axios/BFF) |
| [`client-work-with-auth`](skills/client/client-work-with-auth/SKILL.md) | Colyseus Auth, `onChange`, route guards |
| [`client-work-with-errors`](skills/client/client-work-with-errors/SKILL.md) | Store `error` + `q-banner`, room `onError` |
| [`client-work-with-structure`](skills/client/client-work-with-structure/SKILL.md) | pages / components / boot / stores placement |
| [`work-with-forms`](skills/client/work-with-forms/SKILL.md) | LoginPage `q-form` / rules |
| [`work-with-pages`](skills/client/work-with-pages/SKILL.md) | Routes + guards login/lobby/game |
| [`work-with-stores`](skills/client/work-with-stores/SKILL.md) | Pinia `auth` / `game` |
| [`work-with-styles`](skills/client/work-with-styles/SKILL.md) | Quasar variables + scoped board CSS |
| [`work-with-localization`](skills/client/work-with-localization/SKILL.md) | vue-i18n boot |
| [`work-with-lobby`](skills/client/work-with-lobby/SKILL.md) | Live LobbyRoom subscribe, leave before enter, create / join / joinOrCreate |
| [`work-with-rooms`](skills/client/work-with-rooms/SKILL.md) | Room lifecycle, `onStateChange` / `onLeave` |
| [`work-with-game-board`](skills/client/work-with-game-board/SKILL.md) | Board, `send('move')`, highlights, `canMove` |
| [`work-with-env-deploy`](skills/client/work-with-env-deploy/SKILL.md) | `VITE_*`, hash router, GitHub Pages |

#### Server-wide skills (`.agents/skills/server/`)

Стек: Colyseus 0.18, `@colyseus/tools`, `@colyseus/auth`, `@colyseus/schema`, Express 5, Drizzle + SQLite, mocha + `@colyseus/testing`. Runtime: `../happy-tourist-server/`.

| Skill | Когда |
|-------|--------|
| [`server-align-code`](skills/server/server-align-code/SKILL.md) | Audit ветки / диффа против требований, skills и аналогов |
| [`server-locate-change-points`](skills/server/server-locate-change-points/SKILL.md) | Где править / куда класть новые файлы |
| [`server-verify-code`](skills/server/server-verify-code/SKILL.md) | Проверка кода на соответствие skills |
| [`server-work-with-auth`](skills/server/server-work-with-auth/SKILL.md) | `@colyseus/auth`, JWT `onAuth`, userdata |
| [`server-work-with-errors`](skills/server/server-work-with-errors/SKILL.md) | Auth/move failures, HTTP health, client-facing errors |
| [`server-work-with-structure`](skills/server/server-work-with-structure/SKILL.md) | rooms / schema / db / app.config placement |
| [`server-work-with-test`](skills/server/server-work-with-test/SKILL.md) | mocha + `@colyseus/testing` |
| [`work-with-rooms`](skills/server/work-with-rooms/SKILL.md) | Room lifecycle, registration, seats |
| [`work-with-schema`](skills/server/work-with-schema/SKILL.md) | `@colyseus/schema` board/turn/status/players |
| [`work-with-messages`](skills/server/work-with-messages/SKILL.md) | Room `move` `{ from, to }` |
| [`work-with-checkers`](skills/server/work-with-checkers/SKILL.md) | Авторитетные правила русских шашек |
| [`work-with-routes`](skills/server/work-with-routes/SKILL.md) | HTTP `createEndpoint` / thin routes |
| [`work-with-middleware`](skills/server/work-with-middleware/SKILL.md) | CORS, `/health`, monitor/playground |
| [`work-with-config`](skills/server/work-with-config/SKILL.md) | env, secrets, `defineServer` |
| [`work-with-database`](skills/server/work-with-database/SKILL.md) | GameDatabase, users schema defaults |
| [`work-with-env-deploy`](skills/server/work-with-env-deploy/SKILL.md) | VPS, PM2, GitHub Actions rsync |
| [`work-with-loadtest`](skills/server/work-with-loadtest/SKILL.md) | `@colyseus/loadtest` scripts |

Runtime-пути в skills (`src/…`) — относительно корня соответствующего sibling-репозитория.

Kilo Code: при появлении канона [`../kilo.template.jsonc`](../kilo.template.jsonc); локально `cp kilo.template.jsonc kilo.jsonc` (ignored). Nested skills — `skills.paths` в рабочем `kilo.jsonc`.

---

## OpenSpec / workflow

- Конфиг: [`openspec/config.yaml`](../openspec/config.yaml) (`projectsMap`, `context`, capability rules).
- Capability ID = продуктовый путь (auth/login, lobby/rooms, game/move, …), не имя change.
- Delta: `openspec/changes/<slug>/specs/<capability-id>/spec.md`
- Main: `openspec/specs/<capability-id>/spec.md`
- Типичный Cursor chat workflow: `/opsx-explore` → `/opsx-propose` → review → `/opsx-apply` → `/opsx-sync` → `/opsx-archive`.
- Артефакты живут в **happy-tourist-meta**, не в siblings. Пути к runtime при apply — через [`docs/projects-map.md`](../docs/projects-map.md) (+ local YAML).
- Always-on: [`AGENTS.md`](../AGENTS.md).
