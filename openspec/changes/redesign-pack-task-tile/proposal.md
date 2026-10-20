## Why

Карточка задания в списке набора (`PackTaskTile`) всё ещё в старом playing-card layout (широкая плитка, digit-badge, split вопрос|чипы), хотя модалки peek и compose уже на dots + word-badge + portrait filled slots. Нужен паритет list-tile с семьёй карточек набора и лёгкий polish compose: уже диалог и крупнее hit-target точек сложности.

## What Changes

- Редизайн list-карточки задания: portrait (ширина меньше высоты), chrome как у карточки набора; шапка как в game peek / compose (точки слева, word-badge по центру; без digit outline badge); текст задания; внизу только заполненные answer mini-cards с optional description secondary; пустые слоты и ряд при нуле filled — не показывать; при многих слотах — горизонтальный scroll; prop binding filled-only `{ content; description? }` вместо `slotLabels` с empty placeholders.
- Compose (редактирование задания): ширина диалога не больше ряда из четырёх answer mini-cards (+ padding); интерактивные точки сложности визуально ≈ высоте status-badge.
- Снять carve-out SC-PACK-238: list-tile слоты больше не «compact labeled only» — filled slots в peek-подобном chrome.

## Scope

- **Capability ID:** `content/packs` (list task tile UX + task compose dialog chrome).
- **Пакеты:** client (`happy-tourist.github.io`) only.
- **Поверхности:** список заданий набора (editor / view / add-task-set / staff preview), task compose dialog.
- Контракт HTTP/content persist (`difficulty` 1|2|3, slot `answerCardId`, answer `content`/`description`) — **без изменений** (поля уже есть в content payload / `content_answer_cards`); list-tile secondary description резолвится на client из загруженных `answerCards`; **server — out of scope / FE-only**.

## Out of scope

- Game peek modal (ширина, размер точек, layout) — **оставить как сейчас**.
- Редизайн карточки набора / catalog pack / answer playing-cards / maps.
- Смена правил submit (пустые слоты блокируют submit) и cascade clear слотов.
- Новые npm-зависимости; смена Quasar theme tokens глобально.
- Server / room / schema / peeks budget.

## Capabilities

### New Capabilities

_(нет)_

### Modified Capabilities

- `content/packs`: layout и chrome `PackTaskTile`; compose dialog max-width и размер difficulty dots; MODIFIED SC-PACK-223/224/226/127/238/260/276/277; ADDED SC-PACK-278/279/280.

## Impact

- Client UI: task list tiles на surfaces набора, compose dialog width + larger interactive dots; shared peek CSS tokens MAY получить compose-only modifiers.
- OpenSpec: delta `content/packs`; Visual Spec в design.
- Тесты: client Vitest (tile layout, compose width/dots); server — без изменений.

## References

- Эталон шапки/слотов: game peek + `PackTaskComposeDialog` (после redesign-peek-task-modal).
- Канон portrait chrome / `--pack-card-*`: `PackTaskSetCardTile`, skill pack-cards.
- Sibling: `happy-tourist.github.io/AGENTS.md`; meta skills `work-with-styles` / pack-cards, content pages.
