## Why

Авторам наборов неудобно вручную вбивать десятки карточек ответов и заданий в редакторе. Нужен простой обмен через CSV: отдельно ответы, отдельно задания — с гейтом «сначала ответы», поверх уже существующего draft/save, без новых серверных bulk-API.

## What Changes

- Импорт и экспорт **ответов** (CSV) на поверхности редактора карточек («Набор карточек»).
- Импорт и экспорт **заданий** (CSV) на поверхностях добавления набора заданий и редактирования заданий вложенного набора.
- Импорт заданий недоступен / блокируется, пока в контексте нет карточек ответов; при ссылках на отсутствующие ответы — отказ всего импорта с перечнем недостающих.
- Импорт только **добавляет** в конец (без replace/merge по id); сложность в CSV с default `1`; разделитель `;` без экранирования.

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: CSV import/export ответов и заданий на editor surfaces; гейт и валидация слотов по тексту ответа; append-only.

## Scope

- **Capability ID:** `content/packs`
- **Пакеты:** **client** (основное); server — только существующие draft / add-task-set save, **без новых HTTP**
- Контракт: UI + локальный parse/serialize → мутация рабочей копии → текущий save/submit; не room messages
- Поверхности: редактор карточек; add-task-set; редактор заданий существующего набора
- Формат: одна строка = один объект; `;` — разделитель
  - Ответ: `ответ;абзац1;абзац2;…`
  - Задание: `вопрос;сложность;слот1;слот2;…`

## Out of scope

- XLSX / JSON / RFC-экранирование CSV
- Server bulk-upload или новые endpoints под CSV
- Импорт на staff-модерации / live-only view
- Изменение правил submit (≥2 cards/tasks) и cascade слотов
- BOM/Excel encoding tweaks (later при столкновении)
- Replace/merge карточек по тексту; запрет дубликатов content

## Impact

- Client: editor / add-task-set / tasks UX, i18n, helper parse/serialize, vitest
- Server: без изменения контракта
- Delta: `content/packs`

## References

- Explore 2026-09-28: CSV answers/tasks; D1–D9 (`;` без escape, append, оба task-экрана, difficulty default 1, block missing answers)
- Main `openspec/specs/content/packs/spec.md` (SC-PACK-04, SC-PACK-34 и родственные)
- Sibling `../happy-tourist.github.io/AGENTS.md`, `docs/projects-map.md`
