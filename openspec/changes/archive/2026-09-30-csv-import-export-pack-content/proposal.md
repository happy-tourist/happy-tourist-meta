## Why

Авторам наборов неудобно вручную вбивать десятки карточек ответов и заданий в редакторе. Нужен простой обмен через CSV: отдельно ответы, отдельно задания — с гейтом «сначала ответы», поверх уже существующего draft/save, без новых серверных bulk-API. Списки ответов, заданий, паков и карт должны читаться как колода карт с единым chrome.

Follow-up (explore 2026-09-29 вечер): после поставки CSV/модалка/плиток UX уточнён — убрать модалку и tooltip импорта в пользу рамки с подсказкой формата и inline-ошибкой; увеличить шрифт и ширины плиток, вертикальный split, слоты в ряд; action-кнопки вниз словами; описание на карточке каталога; список карт как карточки; редактор карты по центру; миниатюра выбранной карты в Lobby create.

## What Changes

- Импорт и экспорт **ответов** (CSV) на поверхности редактора карточек («Набор карточек»).
- Импорт и экспорт **заданий** (CSV) на поверхностях добавления набора заданий и редактирования заданий вложенного набора.
- Импорт заданий недоступен / блокируется, пока в контексте нет карточек ответов; при ссылках на отсутствующие ответы — отказ всего импорта с перечнем недостающих.
- Импорт только **добавляет** в конец (без replace/merge по id); сложность в CSV с default `1`; разделитель `;` без экранирования.
- **Import UX (revises modal):** Export + Import в общей **рамке**; под кнопками — рекомендации формата CSV; ошибка импорта — **в этой рамке**; **без** tooltip на Import; Import сразу открывает системный file picker (без `q-dialog`). Export — сразу download; disabled, если нечего экспортировать.
- **Playing-card layout:** ряд скруглённых плиток; **вертикальный** разделитель; шрифт контента ~×2; ответ без description — **150×200**; с description — **300×200**; задание — **300×200**; слоты задания — **в ряд** как заполненные чипы; difficulty сверху слева; бейджи статуса сверху; **Edit/Delete** (и прочие action-кнопки карты) — **внизу**, текстом, друг под другом, на всю ширину. Звезда избранного — **слева сверху**.
- **Каталог packs** и **списки наборов заданий**: card grid **150×200**; в каталоге — truncated **description**; status сверху; star TL; action-кнопки снизу словами.
- **Staff:** после прямого staff-save не показывать ложный «черновик» (уже в scope).
- **Maps list:** список карт как **карточки** — сверху миниатюра сетки, ниже ёмкость `players×tourists` (и chrome по тем же правилам кнопок снизу).
- **Map editor:** весь столбец (поле + палитра + селекты) **по центру** страницы как игровое поле; селекты количества игроков/туристов — нормальной ширины (не схлопнутые).
- **Lobby create:** в закрытом/выбранном состоянии селекта карты — **миниатюра** карты (не только текст).
- **Breadcrumbs:** клик по «Лобби» переходит в лобби (уже в scope).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: CSV framed import UX; tile sizes/layout/actions; catalog description + 150×200; task-set cards; staff draft fix.
- `content/maps`: paint save race; centered editor column + usable seat selects; maps list as cards.
- `ui/branding`: breadcrumb Lobby → lobby.
- `lobby/rooms`: create map select shows mini preview when a map is selected.

## Scope

- **Capability ID:** `content/packs`, `content/maps`, `ui/branding`, `lobby/rooms`
- **Пакеты:** **client** (основное); server — staff draft status (уже); **без новых HTTP** под CSV
- Контракт: UI + локальный parse/serialize → мутация рабочей копии → текущий save/submit; map quiet save; не room messages
- Поверхности CSV: редактор карточек; add-task-set; редактор заданий существующего набора
- Поверхности card chrome: editor, slot picker, live, staff; packs catalog; task-set lists; **maps list**
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
- Клик по телу карты ответа/задания → модалка просмотра (later); правки — только через Edit
- Drag-paint на карте; изменение 10×10 / cell types
- Возврат import `q-dialog` / tooltip на Import

## Impact

- Client: editor / catalog / task-set / maps list / lobby create / map editor UX; shared card + CSV panel components; i18n; vitest
- Server: staff draft status (уже в change)
- Delta: `content/packs`, `content/maps`, `ui/branding`, `lobby/rooms`

## References

- Explore 2026-09-28…29: CSV; D6′ modal; playing-card; catalog; staff draft; map paint; Lobby crumb
- Explore 2026-09-29 вечер: D6″ framed CSV; tile 150/300 vertical; actions bottom; maps cards; centered editor; lobby selected mini
- Main `openspec/specs/content/packs|maps`, `ui/branding`, `lobby/rooms`
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
