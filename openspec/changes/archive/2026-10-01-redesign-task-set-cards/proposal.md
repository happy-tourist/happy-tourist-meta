## Why

Карточки наборов заданий на live pack и в cards editor выглядели дёшево и плохо читались: статус, счётчики и сложность сжаты в мелкий caption, а имя автора и соавторы шумели на каждой плитке. Нужен chrome как на согласованном макете (light/dark): плотная, но аккуратная карточка со статистикой и короткими статусами — без автора/соавторов в UI наборов и лобби. Реализовано: каркас + layout/spacing/hover, **spacing retune** (воздух status→title; tighter title→stats и divider air) и **custom SVG** вместо Material-заглушек.

## What Changes

- Редизайн **карточек набора заданий** (live pack + cards editor) по макету: бейдж статуса сверху, title без автора, строки статистики (заданий / лёгкие / средние / сложные с dots 1–2–3), outline-кнопки снизу с иконками; размер около текущего playing-card, допускается чуть выше.
- Короткие статусы как на макете (`СНЯТО`, `ДОРАБОТАТЬ`, `НА ПРОВЕРКЕ` и т.п. по существующим фазам).
- **Убрать из UI** отображение автора набора заданий **везде** (content labels + lobby create picker + подписи комнат по task-set labels).
- **Соавторы** больше не показываются и не используются как продуктовая фича (display-only уходит; persist пустым).
- **BREAKING** (UX/copy): лейблы наборов больше не содержат «от {имя}»; lobby create/list больше не показывает автора сета.
- **Mock / layout / spacing+hover polish (done):** `#{n}`; total-row; colored outline dots; short «Снять»/«Вернуть»; pale multi-row dividers; column grid; slim actions; fill; themed hover без scale (SC-PACK-239…246).
- **Spacing retune (done):** отступ status→title (~6–8 CSS px); title→stats ~7; divider air ~4–6; grid gutter между карточками **не** трогали (SC-PACK-247).
- **Custom SVG icons (done):** `src/assets/content/task-set-*.svg` на total-row и status badges; total **14px**, badge **~12px**; ink через `currentColor` / mask по Visual Spec (SC-PACK-248).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: chrome карточек task-set; короткие статусы; без author/coauthor на лейблах наборов; SC-PACK-135 / coauthor display revise; SC-PACK-228/229 размер/layout; mock/layout/spacing/hover SC-PACK-239…246; **spacing retune SC-PACK-247**; **custom SVG icons SC-PACK-248**.
- `lobby/rooms`: create picker и подписи комнат по task sets — без автора набора (только «Набор заданий #{n}» / эквивалент).

## Scope

- **Capability ID:** `content/packs`, `lobby/rooms`
- **Пакеты:** **client** (основное); server — только если нужно согласовать пустые `coauthorLabels` / не опираться на author в lobby metadata display (без новых HTTP)
- Поверхности карточек: **live pack**, **cards editor**
- Автор/соавторы: убрать показ на всех лейблах task set в content + lobby (create + room list metadata)
- Контракт room/create UI: без смены create payload ids; только подписи
- Mock / layout / spacing+hover / spacing retune / SVG wire: i18n + `PackTaskSetCardTile` + short card actions + CSS tokens + custom SVG badges/total; assets `src/assets/content/task-set-*.svg`; skills `pack-cards.md` (shipped)

## Out of scope

- Каталог паков (`PackListCardTile` для packs) и maps cards
- Staff hub как card-grid (оставляем текущий text + task tiles)
- **Hover scale / enlarge** карточки (на макете enlarge есть — **не** реализуем; только border + shadow)
- Увеличение **gutter** между карточками в `.pack-card-grid` / соседних списках
- PNG-варианты иконок (берутся SVG из `src/assets/content/`)
- Dots сложности на `PackTaskTile` / других поверхностях
- Drop колонки `coauthor_labels` / миграция SQLite; удаление `authorDisplayName` из API payload
- Автор **карт** (`content/maps`, map picker labels)
- Изменение ACL task-set author / staff soft-unpublish правил
- CSV / playing-card answer/task tiles (150/300×200) кроме косвенного shared chrome
- Смена длинного copy pack/catalog «Снять с публикации» и confirm dialogs
- Custom SVG для **action** кнопок (Edit / Снять / Вернуть) — остаются Quasar Material

## Impact

- Client: task-set list chrome; i18n; short card unpublish/republish; lobby create/list copy; layout + spacing/hover + **spacing retune** + **SVG icons**; тесты SC-PACK-135 / 239…248 / list-cards / lobby
- Server: опционально — всегда пустые coauthor labels при save; metadata/display без требования показывать автора сета клиенту
- Delta: `content/packs`, `lobby/rooms`
- Skills/docs: `work-with-styles/pack-cards.md`, content pages/stores; verify-mock / prepare-mock Anti-miss

## References

- Explore 2026-09-30…2026-10-01: redesign task-set cards; D1 author removed in lobby; mock/layout/spacing+hover; spacing retune (status gap / half title / half divider air); SVG icons placed under `src/assets/content/`
- Макет light/dark (скрины пользователя; prepare-mock measurements)
- Main `openspec/specs/content/packs`, `lobby/rooms`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `../happy-tourist-server/AGENTS.md`, `docs/projects-map.md`
