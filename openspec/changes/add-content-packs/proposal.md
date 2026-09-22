## Why

Peek под коричневой клеткой сейчас — заглушка «Правильно / Неправильно». Чтобы наполнить игру учебным контентом, нужен UGC-раздел **наборов карточек**. Базовый CRUD + одна модерация уже в runtime; монолитный редактор и единый submit мешают авторам: ответы и задания живут разным ритмом. Нужно разделить страницы и потоки модерации, сохранив каталог / коллекцию / роли.

## What Changes

- Домен `content/packs`: наборы с карточками-ответами и наборами заданий; сложность 1–3; коллекция; каталог после approve ответов; block; почта staff-событий.
- **BREAKING (относительно v1 runtime):** два типа moderation request — `answers` и `tasks`; submit и модерация раздельно; approve заданий → затем ответов → каталог.
- Клиент: страница ответов (title/description, форма+список, nested список task set) и страница заданий (форма+слоты+тайлы ответов+список вопросов); автосейв; confirm перед delete; после create → answers; вход в раздел → коллекция, каталог оттуда.
- Staff: очередь только когда есть **answers pending**; hub ответов со вложенными наборами заданий; tasks-only pending staff не видит.
- Lock: пока в ответах есть **неотправленные** изменения (новые/изменённые cards или title/description) — создание и редактирование заданий запрещены; после submit answers задания снова доступны. Бейджа на странице ответов нет.
- Удаление ответа не блокирует submit ответов; пустые слоты блокируют только submit заданий.

## Capabilities

### New Capabilities

- (нет — capability уже введена этим change)

### Modified Capabilities

- `content/packs`: split editor UX; dual submit/moderation (`answers` / `tasks`); lock «dirty answers ⇒ no task edit»; staff hub через answers; collection-first nav; autosave; delete confirm

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- **Кто создаёт/правит:** JWT, не anonymous, `emailVerified`; иначе модалка login/verify
- **Кто смотрит каталог / коллекцию:** любой с сессией (вкл. anonymous / unverified)
- **Сущности:** pack (title, description на answers page); answer cards; task sets (подпись автора/соавтора); tasks (question, slots, difficulty 1–3)
- **Жизненный цикл:** раздельные draft → pending → approved|rejected|cancelled по типу; live answers / live tasks мержатся в публичный snapshot; каталог после approve **answers** (и только если уже есть live tasks)
- **Порядок автора:** cards (+ title/desc) → submit answers → создать/править tasks → submit tasks; без cards нельзя открыть tasks
- **Порядок staff:** approve tasks → approve answers → catalog; вход только через answers pending hub
- **Lock:** dirty answers (неотправленные изменения) ⇒ create/edit tasks запрещены; после submit answers — unlock; pending-автор своего типа может amend/resubmit; два pending могут сосуществовать
- **Коллекция / co-author:** как раньше (коллекция gates edit; labels display-only; несколько авторов task set)
- **HTTP + JWT:** REST; Colyseus / peek **не** меняются
- **Навигация:** Lobby → коллекция; из коллекции → каталог / create / edit

## Out of scope

- Привязка tourist-room к pack; join gate по коллекции; peek runtime из pack
- Картинки / медиа; diff-only staff UI; hard delete; передача владения
- Отдельная staff-очередь «только tasks» без answers pending
- Colyseus realtime для каталога/модерации; платежи

## Impact

- Client: заменить монолитный editor на answers + nested task-set pages; dual submit UX; autosave; Dialog confirm; lobby → collection; staff hub; i18n; dirty-answers lock на tasks.
- Server: request `type` answers|tasks; раздельные submit/approve/lock; staff list filter; тесты SC-PACK.
- Meta: точечно AGENTS/skills под dual flow.
- Игра / lobby / board: без контракта peek.

## References

- Explore 2026-09-22 (initial) + explore split editor 2026-09-22: dual types; minima; catalog after answers approve; title on answers; dirty-answers lock (no badge); staff only after answers pending
- Runtime v1: `happy-tourist-server` `/api/content/*`, `happy-tourist.github.io` Content* pages
- Карта путей: `docs/projects-map.md`
