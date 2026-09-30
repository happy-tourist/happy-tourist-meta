## Why

Карточки наборов заданий на live pack и в cards editor выглядят дёшево и плохо читаются: статус, счётчики и сложность сжаты в мелкий caption, а имя автора и соавторы шумят на каждой плитке. Нужен chrome как на согласованном макете (light/dark): плотная, но аккуратная карточка со статистикой и короткими статусами — без автора/соавторов в UI наборов и лобби. После первой реализации и mock polish (hash / цвета / short actions) каркас ближе к макету, но **раскладка** ещё расходится: слишком толстые кнопки, нет бледных линий между уровнями сложности, колонки icon/dots/labels не выровнены, контент не заполняет высоту карточки.

## What Changes

- Редизайн **карточек набора заданий** (live pack + cards editor) по макету: бейдж статуса сверху, title без автора, строки статистики (заданий / лёгкие / средние / сложные с dots 1–2–3), outline-кнопки снизу с иконками; размер около текущего playing-card, допускается чуть выше.
- Короткие статусы как на макете (`СНЯТО`, `ДОРАБОТАТЬ`, `НА ПРОВЕРКЕ` и т.п. по существующим фазам).
- **Убрать из UI** отображение автора набора заданий **везде** (content labels + lobby create picker + подписи комнат по task-set labels).
- **Соавторы** больше не показываются и не используются как продуктовая фича (display-only уходит; persist пустым).
- **BREAKING** (UX/copy): лейблы наборов больше не содержат «от {имя}»; lobby create/list больше не показывает автора сета.
- **Mock polish (done):** ordinal `Набор заданий #{n}`; total-row с иконкой + «Заданий:»; цветные outline dots; short «Снять» / «Вернуть»; без hover scale.
- **Layout polish (follow-up):** бледные dividers после total, **между** каждыми difficulty rows и **над** actions; колонка lead = ширина 3 dots (icon по центру); labels на одной левой вертикали; counts справа; блоки заполняют высоту карточки (без статуса — пусто только сверху); slim outline actions ~28–32 CSS px (несколько actions — стопкой, как сейчас).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: chrome карточек task-set; короткие статусы; без author/coauthor на лейблах наборов; SC-PACK-135 / coauthor display revise; SC-PACK-228/229 размер/layout для task-set (каталог паков без изменений); mock polish SC-PACK-239…243; layout polish SC-PACK-239/244.
- `lobby/rooms`: create picker и подписи комнат по task sets — без автора набора (только «Набор заданий #{n}» / эквивалент).

## Scope

- **Capability ID:** `content/packs`, `lobby/rooms`
- **Пакеты:** **client** (основное); server — только если нужно согласовать пустые `coauthorLabels` / не опираться на author в lobby metadata display (без новых HTTP)
- Поверхности карточек: **live pack**, **cards editor**
- Автор/соавторы: убрать показ на всех лейблах task set в content + lobby (create + room list metadata)
- Контракт room/create UI: без смены create payload ids; только подписи
- Mock polish: i18n + `PackTaskSetCardTile` chrome + short card action labels
- Layout polish: `PackTaskSetCardTile` CSS/structure + slim action buttons на live/editor; skills `pack-cards.md`

## Out of scope

- Каталог паков (`PackListCardTile` для packs) и maps cards
- Staff hub как card-grid (оставляем текущий text + task tiles)
- Hover scale / enlarge карточки; **новый** hover-polish (border/shadow/tint) сверх существующего clickable border
- Иконки на **бейджах** статуса (asset’ы позже: `task-set-badge-*.svg`) — только имена зафиксированы в design
- Финальный SVG для total-row (`task-set-card-tasks.svg`) — пока Material `description`
- Dots сложности на `PackTaskTile` / других поверхностях
- Drop колонки `coauthor_labels` / миграция SQLite; удаление `authorDisplayName` из API payload
- Автор **карт** (`content/maps`, map picker labels)
- Изменение ACL task-set author / staff soft-unpublish правил
- CSV / playing-card answer/task tiles (150/300×200) кроме косвенного shared chrome
- Смена длинного copy pack/catalog «Снять с публикации» и confirm dialogs
- Смена tint `warning`/`primary` на card actions (нейтральный chrome — отдельное решение)

## Impact

- Client: task-set list chrome; i18n; short card unpublish/republish; lobby create/list copy; layout polish tile + slim btns; тесты SC-PACK-135 / 239…244 / list-cards / lobby
- Server: опционально — всегда пустые coauthor labels при save; metadata/display без требования показывать автора сета клиенту
- Delta: `content/packs`, `lobby/rooms`
- Skills/docs: `work-with-styles/pack-cards.md`, content pages/stores; verify-mock / prepare-mock Anti-miss

## References

- Explore 2026-09-30: redesign task-set cards; D1 author removed in lobby; mock polish; layout polish (pale dividers between all stats rows + above actions; column grid; vertical fill; slim ~28–32px buttons; no scale)
- Макет light/dark (скрины пользователя)
- Main `openspec/specs/content/packs`, `lobby/rooms`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
