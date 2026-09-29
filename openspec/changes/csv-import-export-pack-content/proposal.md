## Why

Авторам наборов неудобно вручную вбивать десятки карточек ответов и заданий в редакторе. Нужен простой обмен через CSV: отдельно ответы, отдельно задания — с гейтом «сначала ответы», поверх уже существующего draft/save, без новых серверных bulk-API. Импорт должен идти через модалку с кратким форматом CSV и ошибками внутри неё. Списки ответов и заданий должны выглядеть как колода игральных карт на всех поверхностях просмотра/выбора.

Follow-up (explore 2026-09-29): после первой поставки CSV/карт проявились UX-баги (белый фон плиток в dark, ложный «черновик» у staff после правок, потеря клеток при быстрой покраске карты, крошка Lobby без перехода) и нужна унификация размеров карт + card-grid для каталога packs и списков наборов заданий; paint-редактор карты — поле как в игре, палитра под полем.

## What Changes

- Импорт и экспорт **ответов** (CSV) на поверхности редактора карточек («Набор карточек»).
- Импорт и экспорт **заданий** (CSV) на поверхностях добавления набора заданий и редактирования заданий вложенного набора.
- Импорт заданий недоступен / блокируется, пока в контексте нет карточек ответов; при ссылках на отсутствующие ответы — отказ всего импорта с перечнем недостающих.
- Импорт только **добавляет** в конец (без replace/merge по id); сложность в CSV с default `1`; разделитель `;` без экранирования.
- **Import UX:** кнопка «Импорт» открывает Quasar-модалку с кратким примером формата CSV и действием выбора файла; ошибки импорта показываются **внутри** модалки; успех закрывает модалку. Export — сразу download; disabled, если нечего экспортировать.
- **Playing-card layout** для карточек ответов и заданий: ряд скруглённых плиток; ответы — content | description; задания — question | слоты, difficulty сверху слева; edit/delete сверху справа. Фиксированные размеры: ответ **без** description — **100×200**; ответ **с** description — **200×200** с разделителем; задание — **200×200**. Читаемый контраст текста/иконок на светлом и тёмном фоне.
- **Каталог packs** и **списки наборов заданий** (live + editor): card grid **100×200** — статус сверху, звезда слева сверху (где применимо), Edit **только справа**, в теле укороченный title / label набора.
- **Staff:** после прямого staff-save (модератор/admin) не показывать ложный статус «черновик» / needs_moderation на наборах — правки сразу в live.
- **Map editor:** не терять клетки при быстрой покраске (блок paint / anti-stale на время save); поле размера как игровое; палитра тайлов **под** картой с подписями **под** тайлами.
- **Breadcrumbs:** клик по «Лобби» реально переходит в лобби.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: CSV import/export; import modal; playing-card sizes/contrast; catalog + task-set card grids; staff без ложного draft после staff-save.
- `content/maps`: paint save race; editor field size + palette under map with labels under tiles.
- `ui/branding`: breadcrumb Lobby navigates to lobby.

## Scope

- **Capability ID:** `content/packs`, `content/maps`, `ui/branding`
- **Пакеты:** **client** (основное); server — правка сериализации staff/status для draft marks при необходимости; **без новых HTTP** под CSV/карты
- Контракт: UI + локальный parse/serialize → мутация рабочей копии → текущий save/submit; map quiet save; не room messages
- Поверхности CSV: редактор карточек; add-task-set; редактор заданий существующего набора
- Поверхности card chrome: editor, slot picker, live, staff; **плюс** packs catalog list; **плюс** task-set lists (live + editor)
- Формат CSV: одна строка = один объект; `;` — разделитель
  - Ответ: `ответ;абзац1;абзац2;…`
  - Задание: `вопрос;сложность;слот1;слот2;…`

## Out of scope

- XLSX / JSON / RFC-экранирование CSV
- Server bulk-upload или новые endpoints под CSV
- Импорт CSV на staff-модерации / live-only view (только visual chrome карт там)
- Изменение правил submit (≥2 cards/tasks) и cascade слотов
- BOM/Excel encoding tweaks (later при столкновении)
- Replace/merge карточек по тексту; запрет дубликатов content
- Клик по телу карты ответа/задания → модалка просмотра (later); правки — только через карандаш
- Drag-paint на карте; изменение 10×10 / cell types

## Impact

- Client: editor / catalog / task-set lists / add-task-set / tasks / live / staff UX; shared card components; CSV modal; map editor layout + save race; App breadcrumbs; i18n; vitest
- Server: при необходимости — статус task-set / pack для staff после staff-save (без новых routes)
- Delta: `content/packs`, `content/maps`, `ui/branding`

## References

- Explore 2026-09-28: CSV answers/tasks; D1–D9; D6′ import modal + playing-card chrome (C1–C9)
- Explore 2026-09-29: tile contrast/sizes; catalog + task-set cards 100×200; staff draft bug; map paint race + palette under; Lobby crumb
- Main `openspec/specs/content/packs/spec.md`, `content/maps/spec.md`, `ui/branding/spec.md`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
