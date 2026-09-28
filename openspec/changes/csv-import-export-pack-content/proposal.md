## Why

Авторам наборов неудобно вручную вбивать десятки карточек ответов и заданий в редакторе. Нужен простой обмен через CSV: отдельно ответы, отдельно задания — с гейтом «сначала ответы», поверх уже существующего draft/save, без новых серверных bulk-API. Импорт должен идти через модалку с кратким форматом CSV и ошибками внутри неё. Списки ответов и заданий должны выглядеть как колода игральных карт на всех поверхностях просмотра/выбора.

## What Changes

- Импорт и экспорт **ответов** (CSV) на поверхности редактора карточек («Набор карточек»).
- Импорт и экспорт **заданий** (CSV) на поверхностях добавления набора заданий и редактирования заданий вложенного набора.
- Импорт заданий недоступен / блокируется, пока в контексте нет карточек ответов; при ссылках на отсутствующие ответы — отказ всего импорта с перечнем недостающих.
- Импорт только **добавляет** в конец (без replace/merge по id); сложность в CSV с default `1`; разделитель `;` без экранирования.
- **Import UX:** кнопка «Импорт» открывает Quasar-модалку с кратким примером формата CSV и действием выбора файла; ошибки импорта показываются **внутри** модалки; успех закрывает модалку. Export — сразу download; disabled, если нечего экспортировать.
- **Playing-card layout** для карточек ответов и заданий: ряд скруглённых плиток (сколько влезает), разделенных пополам; ответы — content | description (без description — прямоугольник «половина»); задания — question | слоты, difficulty сверху слева; edit/delete (карандаш + корзина) сверху справа; длинное description — scroll в нижней половине. Форма Add остаётся сверху; сетка карт — ниже. Макет **везде**: editor, slot picker, live view, staff moderation.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: CSV import/export ответов и заданий; гейт и валидация слотов; append-only; Quasar import-модалка с форматом и ошибками; playing-card presentation ответов/заданий на всех surfaces.

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** **client** (основное); server — только существующие draft / add-task-set save, **без новых HTTP**
- Контракт: UI + локальный parse/serialize → мутация рабочей копии → текущий save/submit; не room messages
- Поверхности CSV: редактор карточек; add-task-set; редактор заданий существующего набора
- Поверхности card chrome: те же editor surfaces **плюс** slot picker, live pack/tasks view, staff moderation views, где показываются answer cards / tasks
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
- Клик по телу карты → модалка просмотра (later); правки в этом change — только через карандаш

## Impact

- Client: editor / add-task-set / tasks / live / staff UX, shared card components, CSV import modal, i18n, helper parse/serialize, vitest
- Server: без изменения контракта
- Delta: `content/packs`

## References

- Explore 2026-09-28: CSV answers/tasks; D1–D9; follow-up: D6′ import modal + playing-card chrome (C1–C9)
- Main `openspec/specs/content/packs/spec.md` (SC-PACK-04, SC-PACK-34 и родственные)
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
