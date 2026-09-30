## Why

Карточки наборов заданий на live pack и в cards editor выглядят дёшево и плохо читаются: статус, счётчики и сложность сжаты в мелкий caption, а имя автора и соавторы шумят на каждой плитке. Нужен chrome как на согласованном макете (light/dark): плотная, но аккуратная карточка со статистикой и короткими статусами — без автора/соавторов в UI наборов и лобби. После первой реализации каркас есть, но визуал ещё расходится с макетом (решётка в номере, цвета dots, разделители, короткие action-labels, иконка «Заданий»).

## What Changes

- Редизайн **карточек набора заданий** (live pack + cards editor) по макету: бейдж статуса сверху, title без автора, строки статистики (заданий / лёгкие / средние / сложные с dots 1–2–3), outline-кнопки снизу с иконками; размер около текущего playing-card, допускается чуть выше.
- Короткие статусы как на макете (`СНЯТО`, `ДОРАБОТАТЬ`, `НА ПРОВЕРКЕ` и т.п. по существующим фазам).
- **Убрать из UI** отображение автора набора заданий **везде** (content labels + lobby create picker + подписи комнат по task-set labels).
- **Соавторы** больше не показываются и не используются как продуктовая фича (display-only уходит; persist пустым).
- **BREAKING** (UX/copy): лейблы наборов больше не содержат «от {имя}»; lobby create/list больше не показывает автора сета.
- **Follow-up polish (макет):** ordinal `Набор заданий #{n}` везде; total-row с иконкой + «Заданий:»; цветные difficulty dots (1 green / 2 amber / 3 red, пустые — outline); два горизонтальных разделителя (после total и перед actions); на карточке short actions «Снять» / «Вернуть» (pack/catalog unpublish copy и confirm dialogs без изменений); без нового hover-polish и без scale.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: chrome карточек task-set; короткие статусы; без author/coauthor на лейблах наборов; SC-PACK-135 / coauthor display revise; SC-PACK-228/229 размер/layout для task-set (каталог паков без изменений); mock polish SC-PACK-239…243 (hash, colors, dividers, short card actions).
- `lobby/rooms`: create picker и подписи комнат по task sets — без автора набора (только «Набор заданий #{n}» / эквивалент).

## Scope

- **Capability ID:** `content/packs`, `lobby/rooms`
- **Пакеты:** **client** (основное); server — только если нужно согласовать пустые `coauthorLabels` / не опираться на author в lobby metadata display (без новых HTTP)
- Поверхности карточек: **live pack**, **cards editor**
- Автор/соавторы: убрать показ на всех лейблах task set в content + lobby (create + room list metadata)
- Контракт room/create UI: без смены create payload ids; только подписи
- Mock polish: i18n + `PackTaskSetCardTile` chrome + short card action labels

## Out of scope

- Каталог паков (`PackListCardTile` для packs) и maps cards
- Staff hub как card-grid (оставляем текущий text + task tiles)
- Hover scale / enlarge карточки; **новый** hover-polish (border/shadow/tint) в этом follow-up
- Иконки на **бейджах** статуса (asset’ы позже: `task-set-badge-*.svg`) — только имена зафиксированы в design
- Dots сложности на `PackTaskTile` / других поверхностях
- Drop колонки `coauthor_labels` / миграция SQLite; удаление `authorDisplayName` из API payload
- Автор **карт** (`content/maps`, map picker labels)
- Изменение ACL task-set author / staff soft-unpublish правил
- CSV / playing-card answer/task tiles (150/300×200) кроме косвенного shared chrome
- Смена длинного copy pack/catalog «Снять с публикации» и confirm dialogs

## Impact

- Client: task-set list chrome; i18n short statuses + `#{n}` label; short card unpublish/republish; lobby create/list copy; тесты SC-PACK-135 / 239…243 / list-cards / lobby create
- Server: опционально — всегда пустые coauthor labels при save; metadata/display без требования показывать автора сета клиенту
- Delta: `content/packs`, `lobby/rooms`
- Skills/docs: `work-with-styles/pack-cards.md`, content pages/stores при apply

## References

- Explore 2026-09-30: redesign task-set cards; D1 author removed in lobby too; mock polish follow-up (hash / colors / dividers / Снять / Вернуть)
- Макет light/dark (скрин пользователя)
- Main `openspec/specs/content/packs`, `lobby/rooms`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
