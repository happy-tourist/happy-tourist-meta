## Why

После follow-up §10–12 остались UX-дыры на коллекции и live-просмотре: иконка «убрать» выглядит как прочерк и клик уводит в набор; Edit из списка у опубликованного пака открывает view без полей; на live нет Edit даже когда пак уже в коллекции; кнопка «В коллекцию» не отражает фактическое членство. Нужен узкий polish affordances без смены домена.

## What Changes

- Коллекция: строка → live view; Edit и корзина — отдельные иконки с изоляцией клика; remove = **корзина** (`delete`) + confirm.
- Edit из списка всегда ведёт в editor (не проигрывает гонку `:to` → view).
- Live pack page: **Редактировать**, если пак в коллекции пользователя; скрывать Edit при чужом pending (`answers` или `tasks`); **автор заявки** по-прежнему видит Edit (докидывать/ресабмит).
- Live/API: честный `inCollection` — кнопка «В коллекции» / disabled, а не вечный «В коллекцию».
- Eligibility (login/verify) — при входе в editor, как create.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: collection list click isolation + trash icon; live Edit when in collection (pending-author exception); `inCollection` on live GET / collect button state

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** client + server (+ meta AGENTS/skills при необходимости)
- Сохранить dual submit, D1′ locks, staff flow, author delete unpublished, no block UI
- Server: `getLivePack` (+ при необходимости) флаги `inCollection` и pending authorship для UI
- Client: ContentCollectionPage, ContentPackPage, store/i18n; починить гонку Edit vs row `:to`

## Out of scope

- Room↔pack, peek, media, transfer ownership
- Staff/public hard delete опубликованного; block anytime UI
- Смена порядка approve / partial submit
- Скрывать Edit у **автора** pending (он MAY докидывать)

## Impact

- Client: ContentCollectionPage, ContentPackPage, Pinia content, i18n
- Server: `lib/content.ts` live pack payload (+ mocha)
- Meta: skills/AGENTS hints

## References

- Explore 2026-09-23 (UX affordances): D1 live-in-collection Edit, D2 trash, D3 pending-author keeps Edit; collect button bug
- Prior sections 1–12 implemented
- Карта путей: `docs/projects-map.md`
