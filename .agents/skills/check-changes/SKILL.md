---
name: check-changes
description: >-
  Scans unstaged (and untracked) changes in happy-tourist client and server,
  then recommends whether to add, update, delete, or split bloated skills using
  a scored bloat checklist (topic files preferred; score ≥5 → mandatory split
  plan; forbid description-dumps into fat cores), and whether to fix
  projects-map / project-map, docs, or AGENTS.md. Use when the user runs
  check-changes, asks to review unstaged client/server diffs for skill/docs/agent
  drift, wants a «что добавить / что изменить / что удалить / что разбить»
  list, or asks which skills have grown too large to split.
---

# Check Changes — навыки, maps, docs, AGENTS

Проверить **незастейдженные** (и untracked) изменения в **client** и **server**, сопоставить их с каноном skills / projects-map / docs / `AGENTS.md`, и выдать список предложений: **что добавить**, **что изменить**, **что удалить**, **что разбить** (раздувшиеся skills → отдельный skill или под-skills).

Скилл **только анализирует и предлагает** — не правит файлы, не стейджит, не коммитит (для коммита — [`commit`](../commit/SKILL.md)).

## Когда применять

- Пользователь запускает `check-changes` / просит проверить незастейдженные изменения client/server на drift skills/docs/agents
- После локальной разработки, до `commit`, когда нужно понять, не устарели ли навыки и документация агентов
- Явно спрашивают, какие skills раздулись и стоит ли вынести / разбить

Не вызывать «на всякий случай» после каждой мелкой правки без запроса.

## Репозитории и пути

Разрешить пути от корня `happy-tourist-meta` через [`docs/projects-map.md`](../../../docs/projects-map.md) (+ `projects-map.local.yaml` если есть):

| Ключ | Репозиторий | Default path | Что сканировать |
|------|-------------|--------------|-----------------|
| `client` | `happy-tourist.github.io` | `../happy-tourist.github.io` | **да** — unstaged + untracked |
| `server` | `happy-tourist-server` | `../happy-tourist-server` | **да** — unstaged + untracked |
| meta | `happy-tourist-meta` | `.` | **нет** как источник diff; **да** как канон для сверки |

Если путь не найден — спросить пользователя, не угадывать.

### Канон для сверки (meta + siblings)

| Область | Где смотреть |
|---------|--------------|
| Индекс skills | [`.agents/AGENTS.md`](../../AGENTS.md) |
| Client skills | `.agents/skills/client/*/SKILL.md` |
| Server skills | `.agents/skills/server/*/SKILL.md` |
| Workspace skills | `.agents/skills/` (`commit`, `openspec-*`, этот скилл) |
| Projects map | `docs/projects-map.md`, `projects-map.local.yaml.example` |
| Child maps | `{client}/project-map.md`, `{server}/project-map.md` |
| Docs | `docs/README.md` и связанные `docs/**` |
| AGENTS.md | meta корень + `.agents/AGENTS.md` + `{client}/AGENTS.md` + `{server}/AGENTS.md` |

## Scope diff

Только **незастейдженное** working tree client/server:

```bash
# в каждом из client и server:
git status -sb
git status --porcelain
git diff                 # unstaged tracked
git ls-files --others --exclude-standard   # untracked
```

Для untracked — прочитать содержимое новых файлов (или `git diff --no-index /dev/null <file>` где доступно).

**Не включать** в основной анализ:

- `git diff --cached` (staged) — если staged есть, кратко упомянуть: «есть staged, этот скилл их не разбирает; застейджьте/расстейджьте или скажите расширить scope»
- коммиты ветки vs merge-base (это зона `*-verify-code` / `*-align-code`)

Если и client, и server чисты по unstaged/untracked — остановиться: «незастейдженных изменений нет».

## Workflow

```
Check-Changes Progress:
- [ ] 1. Resolve paths
- [ ] 2. Collect unstaged/untracked (client + server)
- [ ] 3. Summarize change themes
- [ ] 4. Map themes → skills / maps / docs / AGENTS
- [ ] 5. Scan bloat candidates (touched + oversized skills)
- [ ] 6. Decide add vs change vs delete vs split vs none
- [ ] 7. Report «Что добавить» / «Что изменить» / «Что удалить» / «Что разбить»
```

### 1. Resolve paths

Из корня meta: projects-map (+ local). Проверить, что client/server — git work trees.

### 2. Collect diffs (параллельно)

В client и server — команды из Scope. Собрать:

| Repo | Unstaged files | Untracked | Themes (1–3 bullets) |
|------|----------------|-----------|----------------------|

Показать пользователю краткую сводку **до** глубокого анализа.

### 3. Summarize change themes

По путям и содержимому diff сгруппировать темы, например:

- новый домен / слой (chat, spectate, ratings UI, новый HTTP endpoint, новая room message)
- смена контракта (payload `move`, auth flow, schema fields, env vars)
- новый каталог / тип файлов, которого нет в structure-skills
- смена путей репозиториев / layout workspace
- правки только стиля / i18n / тестов без нового паттерна

Игнорировать шум: форматирование без смысловых паттернов, lockfile-only, сгенерированные артефакты.

### 4. Map → канон

Для каждой темы проверить покрытие:

| Вопрос | Куда смотреть |
|--------|----------------|
| Есть skill с этим concern? | client/server skill descriptions + `.agents/AGENTS.md` таблицы |
| Skill описывает **устаревший** API/путь/паттерн относительно diff? | соответствующий `SKILL.md` |
| Skill **потерял актуальность** (домен/путь/стек убраны diff’ом)? | skill body + diff + факт наличия кода в client/server |
| Новые пути/ключи сервисов? | `docs/projects-map.md`, child `project-map.md` |
| Новые команды сборки / стек / домены пакета? | `{client\|server\|meta}/AGENTS.md`, `.agents/AGENTS.md` |
| Канон docs устарел или нет ссылки? | `docs/README.md`, профильные `docs/**` |

Читать только релевантные skill/AGENTS/docs (не все подряд), но **не пропускать** индекс `.agents/AGENTS.md` и корневые AGENTS затронутых пакетов. Для кандидатов на удаление — сверить skill с **текущим** деревом client/server (не только с diff): пути/API из skill ещё существуют?

### 5. Scan bloat candidates (обязательный скоринг)

После map (шаг 4) **обязательно** проверить раздутие — не только «есть ли покрытие», но и «не стал ли skill толстым / многотемным / description-dump».

**Анти-паттерн проекта:** дописывать SC-/API-перечисления в `description` и ядро вместо topic-файла. Если diff снова требует такой dump — это сигнал **split**, не «ещё одна строка в description».

#### Кого сканировать

1. Skills, на которые **попали темы diff**.
2. Skills того же слоя/домена, которые diff расширяет (pages/stores/styles/routes/…), даже если файл skill в diff не менялся.
3. Любой client/server `SKILL.md` с `wc -l` ≳ 220 или description ≳ 800 символов — быстрый pass.
4. Уже разбитые skills: проверить, не раздулось ли **ядро** снова и не стал ли topic `*.md` монстром (≳ **250** строк topic → предложить под-topic или сжатие).
5. Не аудитить все `openspec-*` без запроса.

Быстрая оценка (из корня meta):

```bash
wc -l .agents/skills/client/*/SKILL.md .agents/skills/server/*/SKILL.md | sort -n
# description length (optional):
# awk '/^description:/{f=1;next}/^---/{f=0}f' SKILL.md | wc -c
```

Для каждого кандидата прочитать: frontmatter `description`, `##` заголовки, таблицу **Specialized Topics** (есть / нет), соседние `*.md`.

#### Скоринг (обязательно заполнить в уме / в отчёте)

Сложить баллы по таблице. **Исключения process-skills** (`*-align-code`, `*-verify-code`, `server-work-with-test`, длинные checklist workflow): считать только сигналы «ортогональные домены» / «description-dump» / «дубль соседнего skill»; длина сама по себе не даёт баллов.

| Баллы | Сигнал | Как мерить |
|------:|--------|------------|
| +2 | Ядро `SKILL.md` ≳ **220** строк | `wc -l` |
| +3 | Ядро ≳ **350** строк | `wc -l` |
| +2 | `description` ≳ **800** символов | `wc -c` тела description |
| +3 | `description` ≳ **1200** символов **или** 5+ несвязанных concerns / длинный SC-/API enumeration | frontmatter |
| +3 | ≥ **3 ортогональные темы** в одном ядре (разные «когда читать») | `##` / Domain Map / unrelated surfaces |
| +2 | Diff снова дописывает крупный блок (≥ ~30 строк канона) в уже ≥220 skill | diff vs skill size |
| +2 | Нет таблицы Specialized Topics при ядре ≳220 **или** topics есть, но ядро всё ещё держит детали topics | структура папки |
| +2 | Дублирование с соседним skill (тот же concern подробно в двух местах) | сравнение |
| +1 | Topic-файл ≳ **250** строк (сам кандидат на вынос/сжатие) | `wc -l` topic |
| +2 | Индекс `.agents/AGENTS.md` «Когда» — простыня SC-кодов вместо короткого trigger + ссылки на topics | индекс |

**Порог решения:**

| Сумма | Действие |
|------:|----------|
| 0–2 | Не предлагать split («разбивать рано») |
| 3–4 | Предложить split **A** как рекомендацию; если diff только точечный — допустим change + «следить» |
| ≥ **5** | **Обязательно** «Что разбить» с конкретным планом A или B — **нельзя** закрыть только «изменить description» |
| ≥ 5 и diff добавляет канон в тот же skill | В «Что изменить» запретить «дописать в ядро»; писать «вынести в topic X, в ядре — одна строка + ссылка» |

Цель после split **A**: ядро ≲ **220** строк; `description` ≲ **800** символов; детали — в topic; индекс AGENTS — короткий trigger.

#### Эвристики выбора границ topic (A)

Резать по **поверхности чтения агента**, не по SC-номерам:

| Родитель (пример) | Типичные topics |
|-------------------|-----------------|
| `work-with-pages` | `shell.md` (App/header/crumbs), `content-pages.md` (packs/maps UI), `game-page.md` (GamePage HUD wiring) |
| `work-with-styles` | `theme.md` (Dark/HTTP), `board.md` (GamePage CSS), `pack-cards.md` (tiles/grid/cascade) |
| `work-with-stores` | уже `content.md` / `maps.md`; далее `game.md` (options store I/O) |
| `work-with-localization` | короткие namespaces в description; детали ключей — в topic родителя домена (`content.md`), не простыня в description |
| `work-with-game-board` | уже `peek.md` / `focus.md`; новые механики → новый topic, не ядро |

Не плодить topic на один SC-*. Один topic = устойчивая подсистема (shell / packs UI / board CSS / game store).

#### Два варианта разбиения (выбрать явно)

**A. Topic files — по умолчанию**, если темы — грани одного слоя/точки входа. Эталон:

```text
.agents/skills/client/work-with-<domain>/
  SKILL.md          # short description + core + Specialized Topics table
  shell.md          # узкая тема
  …
```

```markdown
## Specialized Topics

Read the matching file in this folder when the change involves that area
(do not load every file at once):

| Topic | File |
|-------|------|
| … | [foo.md](foo.md) |
```

Description: «Core in SKILL.md; topic details in …».

**B. Sibling skill** — только если concern **самостоятельно discoverable** (другой trigger/слой, вызывают без родителя). Иначе A.

В «Что разбить» для каждого кандидата: **score N**, сигналы, **A|B**, имена файлов, что перенести (1–2 строки), целевой размер ядра, правки description + `.agents/AGENTS.md` «Когда».

### 6. Decide: add / change / delete / split / none

Правила решения:

**Добавить новый skill**, если:

- появился устойчивый повторяемый паттерн работы агента (новый домен: chat, spectate, ratings, …);
- ни один существующий skill не покрывает concern даже после расширения description/body;
- паттерн достаточно стабилен, чтобы не быть one-off хаком.

Имя/расположение: `client-*` / `work-with-*` под `.agents/skills/client/` или `server-*` / `work-with-*` под `.agents/skills/server/`; workspace-уровень — рядом с `commit` / `check-changes`.

Если новый concern логично ложится **внутрь** уже большого skill — сначала рассмотреть **split (A)**, а не третий параллельный skill на ту же страницу/слой.

**Изменить существующий skill**, если:

- diff меняет канонический API, пути файлов, сообщения, env, lifecycle, которые skill ещё нужен, но описывает устаревше;
- description skills не триггерится на новую терминологию из diff;
- в skill есть запрет/пример, противоречащий новым изменениям;
- skill частично устарел — достаточно точечной правки, **не** удаления;
- правка **небольшая** и skill ещё не в зоне bloat — точечный update, без split.

**Разбить / вынести (split)**, если скоринг шага 5 ≥ **3** (при ≥ **5** — обязательно, не заменять точечным change description):

- предложить **A (topic files)** или **B (sibling skill)** по правилам выше;
- указать **score** и 2–4 главных сигнала;
- не предлагать и add, и split одного и того же concern без выбора;
- при score ≥ 5 и необходимости нового канона из diff — **запретить** «дописать простыню в ядро/description»; план: topic + короткая ссылка;
- при split ядра — в «Что изменить» кратко («укоротить description / Domain Map после выноса»), детали — в «Что разбить».

**Удалить skill** (предложить удаление каталога `…/SKILL.md` + строки из индексов AGENTS), если:

- diff **убирает** домен / слой / контракт / стек, ради которого skill существовал, и в client/server **не осталось** кода/пути, на который skill опирается;
- skill целиком про чужой стек/продукт (остаток копипаста) и не применим к текущему client/server;
- skill полностью дублирует другой актуальный skill без уникального concern (предложить удалить **дубликат**, оставить канон);
- description/body skill триггерятся на работу, которой в проекте больше нет — агент будет следовать мёртвым правилам.

**Не предлагать удаление**, если:

- skill про будущий/плановый домен, явно ещё в AGENTS/product scope («stub», «planned»);
- устарела только часть — тогда **изменить**, не удалять;
- нет связи с текущим unstaged diff и нет явных признаков мёртвого skill (не устраивать полный purge «на всякий случай»).

При предложении удаления всегда указать: **зачем skill был**, **почему больше не актуален** (ссылка на diff/отсутствие кода), **что почистить в индексах** (`.agents/AGENTS.md`, корневой `AGENTS.md` при упоминании).

**Поправить projects-map / project-map**, если:

- новые ключи путей для OpenSpec/агентов;
- сменился layout siblings / имя репо;
- child `project-map.md` отсутствует или ключ `happy-tourist-meta` неверен;
- ключи map указывают на удалённые пути — поправить или убрать (часто рядом с delete skill).

**Поправить docs**, если:

- канон в `docs/` описывает старый контракт/архитектуру;
- в оглавлении нет нового важного документа, который стоит завести или на который сослаться;
- docs ссылаются на skill/путь, который предлагается удалить.

**Поправить AGENTS.md** (meta корень, `.agents/AGENTS.md`, client, server), если:

- новый skill нужно внести в индекс;
- skill предложен к удалению или split **B** — обновить таблицы/discovery (для **A** обычно достаточно строки «когда» / description, без новых строк индекса на каждый topic.md);
- сменились стек, команды, домены, ссылки на skills/docs;
- always-on инструкции расходятся с фактическим diff-паттерном.

**Ничего не предлагать**, если изменение полностью покрыто актуальными skills/docs/AGENTS и maps, мёртвых skills по diff не видно, и bloat-кандидатов нет (или явно «разбивать рано»). Явно написать: «покрытие достаточное» / «разбиение не требуется».

Не предлагать дубликаты skills «на всякий случай». Не предлагать правки runtime-кода — только agent/docs/map артефакты. Не удалять и не дробить skills самому в этом прогоне — только рекомендовать.

### 7. Report

Формат ответа пользователю (четыре основные секции + краткий контекст):

```markdown
## Сводка изменений
| Repo | Unstaged / untracked | Темы |
|------|----------------------|------|
| client | … | … |
| server | … | … |

## Что добавить
- **`<артефакт>`** (`путь`): зачем (1 предложение, связь с diff)
- … или «ничего»

## Что изменить
- **`<артефакт>`** (`путь`): что именно поправить и почему
- … или «ничего»

## Что удалить
- **`<skill или артефакт>`** (`путь`): почему потерял актуальность (diff / нет кода); что убрать из AGENTS/docs
- … или «ничего»

## Что разбить
- **`<skill>`** (`путь`, ~N строк ядра, desc ~C chars): **score N** (сигналы: …); **A** (topics: `a.md`, `b.md`; цель ядро ≲220) или **B** (sibling `…`); что перенести; правки description / AGENTS «Когда»
- … или «ничего» / «разбивать рано» (score ≤2)

## Вне scope / заметки
- staged ignored; опциональные follow-ups (verify-code, commit) — коротко
```

Каждый пункт — actionable: конкретный файл/skill-name и суть правки/удаления/разбиения. Без длинных цитат diff.

Приоритет в списке: skills (add → change → delete → split) → AGENTS индексы → projects-map → docs.

## Примеры триггеров

- «Запусти check-changes»
- «Проверь незастейдженные изменения client/server — нужны ли новые skills?»
- «Какие skills устарели и можно удалить?»
- «Какие skills раздулись — вынести или разбить на под-скиллы?»
- «Что поправить в AGENTS/docs после моих локальных правок?»

## Не делать

- Не редактировать и не удалять skills/docs/AGENTS/maps в этом скилле без отдельной просьбы применить предложения.
- Не предлагать массовое удаление skills без связи с diff или явной мёртвости относительно текущего client/server.
- Не предлагать split «на всякий случай» без эвристик шага 5; не дробить process-skills (`*-align-code` / `*-verify-code`) только из-за `wc -l`.
- Не анализировать staged / branch commits как основной scope.
- Не сканировать meta working tree как источник «изменений продукта» (кроме сверки канона и bloat scan skills).
- Не запускать npm build/test.
- Не путать с `client-verify-code` / `server-verify-code` (compliance кода) и `commit` (stage/commit/push).
