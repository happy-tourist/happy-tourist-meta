## Why

После модерации и в публичном просмотре наборы показывают вводящие в заблуждение статусы и счётчики слотов вместо самих слотов; личный черновик может отстать от опубликованного live; staff не может снять пак с публикации или убрать лишний набор заданий из live. Нужно выровнять UX публикации, отображение слотов и staff-управление опубликованным контентом.

## What Changes

- Подписи статусов: опубликованные ответы — «Опубликовано»; одобренные задания без live answers — «Одобрено».
- Live и staff: показывать слоты задания (в т.ч. пустые), не счётчик; cascade — жёлтая рамка без заливки строки.
- При отставании черновика от live — предупреждение и «Подтянуть» с сохранением локальных правок как новых сущностей.
- Staff: снять пак с публикации / вернуть в каталог; снять один набор заданий с live (если set’ов ≥ 2); без ужесточения чужого pending-lock и без UI block.

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (HTTP content API + UI коллекция / live / редакторы / staff preview)
- Коллекция: staff-кнопки снять / вернуть публикацию пака
- Live-просмотр и staff preview заданий: слоты как содержимое
- Редакторы карточек/заданий: статусы под заголовком; баннер+подтянуть; жёлтая рамка cascade; staff — снять task set справа в строке
- Контракт HTTP: unpublish / republish пака; unpublish одного task set из live; индикация stale draft + rebase/pull

## Out of scope

- Ужесточение lock при чужой модерации (оставить как сейчас)
- UI / продуктовое использование **block** пака
- Привязка наборов к tourist-room / peek-наградам
- Hard-delete опубликованного пака авторами
- Изменение правил dual submit answers|tasks, кроме отмены open-заявок при unpublish

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: статусы публикации; отображение слотов; cascade-индикация; stale-draft pull; staff unpublish/republish пака; staff unpublish task set из live

## Impact

- Client: страницы коллекции, live-пака, редакторов карточек/заданий, staff preview заданий; store content; i18n
- Server: content HTTP + `lib/content` (live pointers, last-live снимок, drafts, moderation cancel, task-set live edit)
- Тесты: server mocha SC-PACK*; client vitest по новым/изменённым SC
- Specs: delta `openspec/changes/content-pack-publish-ux/specs/content/packs/spec.md` → sync в main после archive

## References

- Explore-решения D1–D18 (сессия openspec-explore)
- Main spec: `openspec/specs/content/packs/spec.md`
- Sibling AGENTS: `happy-tourist.github.io/AGENTS.md`, `happy-tourist-server/AGENTS.md`
- Skills: `.agents/skills/client/work-with-stores` (content), `.agents/skills/server/work-with-database` / routes (content)
