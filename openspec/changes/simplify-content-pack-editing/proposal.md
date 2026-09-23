## Why

Модель «личный черновик у каждого в коллекции + dual submit + гонки» даёт конфликты, stale-draft и путаницу с Edit. Нужна простая публикация: автор один раз собирает и сдаёт пак, после approve контент заморожен для non-staff; правки — только у модераторов (сразу в live) или через поддержку.

## What Changes

- **BREAKING:** убрать персональные drafts / dual post-publish edit / foreign-pending co-edit; одна рабочая копия до первого апрува.
- После публикации: non-staff не правят карточки и существующие задания; только добавление нового task set (с модерацией этого set).
- Staff: полный Edit любого пака без коллекции, без модерации своих правок, exclusive lock (второй Edit → ошибка).
- Первая модерация: один submit (карточки + ≥1 task set), одна кнопка «Одобрить» / «Доработать»; hard-reject убрать; автор может Cancel.
- Support: тема «изменить набор карточек» + select из каталога (название/описание) + ссылка на пак в заявке.
- Убрать staff unpublish/republish / wipe-drafts / draftStale-rebase из недавно добавленного UX.

## Scope

- **Capability ID:** `content/packs`, `support/tickets`
- **Пакеты:** client + server (HTTP content + support UI/API)
- Коллекция / live / редакторы / staff queue / my-moderation
- Контракт HTTP: рабочая копия до live, единый approve, add-task-set submit, staff lock + live edit, удаление personal drafts API

## Out of scope

- Привязка наборов к tourist-room / peek-наградам
- UI product **block** пака (кроме косвенно через support-текст)
- Новые npm-зависимости / внешние сервисы
- Изменение ролей staff/admin вне ACL Edit

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: freeze после publish; одна рабочая копия; единый approve/доработать; add-only task set; staff live edit + lock; удаление personal drafts / unpublish-republish / stale pull
- `support/tickets`: тема change-pack + select каталога + ссылка на пак

## Impact

- Client: Content* pages/store, i18n; Support create form (topic + pack select)
- Server: `lib/content`, schema drafts/locks, content HTTP; `lib/support` topics + create payload
- Тесты: mocha SC-PACK* / SC-SUP*; vitest content + support
- Specs: delta → sync в main после archive
- Код published-ux (drafts/stale/unpublish) снимается этим change

## References

- Explore-решения D1–D16 (сессия openspec-explore)
- Main: `openspec/specs/content/packs/spec.md`, `openspec/specs/support/tickets/spec.md`
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
