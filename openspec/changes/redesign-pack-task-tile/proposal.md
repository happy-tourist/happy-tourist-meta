## Why

Карточка задания в списке набора (`PackTaskTile`) всё ещё в старом playing-card layout (широкая плитка, digit-badge, split вопрос|чипы), хотя модалки peek и compose уже на dots + word-badge + portrait filled slots. Нужен паритет list-tile с семьёй карточек набора и лёгкий polish compose: уже диалог и крупнее hit-target точек сложности. После первого apply — follow-up polish: actions как у набора, вопрос как в peek, слоты по центру с воздухом над actions, Save compose как peekSubmit без Cancel, ширина +≈50px.

## What Changes

- Редизайн list-карточки задания: portrait (ширина меньше высоты), chrome как у карточки набора; шапка как в game peek / compose (точки слева, word-badge по центру; без digit outline badge); текст задания **24px** (как game peek); внизу только заполненные answer mini-cards с optional description secondary, **горизонтально по центру**, с воздухом над actions (не прилипать к низу); пустые слоты и ряд при нуле filled — не показывать; при многих слотах — горизонтальный scroll; prop binding filled-only `{ content; description? }` вместо `slotLabels` с empty placeholders.
- Edit/Delete на editable list tile: **outline + icon**, slim `--pack-card-action-h`, как у `PackTaskSetCardTile`; Delete MAY остаться negative/red.
- Compose (редактирование задания): ширина ≈ 4 mini-cards + pad **+ ≈50px** slack; интерактивные точки сложности ≈ высоте status-badge; кнопка Save — full-width primary unelevated как game peekSubmit; **Cancel убрать** (dismiss = backdrop / Esc).
- Снять carve-out SC-PACK-238: list-tile слоты больше не «compact labeled only» — filled slots в peek-подобном chrome.

## Scope

- **Capability ID:** `content/packs` (list task tile UX + task compose dialog chrome).
- **Пакеты:** client (`happy-tourist.github.io`) only.
- **Поверхности:** список заданий набора (editor / view / add-task-set / staff preview), task compose dialog.
- Контракт HTTP/content persist (`difficulty` 1|2|3, slot `answerCardId`, answer `content`/`description`) — **без изменений** (поля уже есть в content payload / `content_answer_cards`); list-tile secondary description резолвится на client из загруженных `answerCards`; **server — out of scope / FE-only**.

## Out of scope

- Game peek modal (ширина, размер точек, layout) — **оставить как сейчас** (кроме того что list tile **переиспользует** peek question font-size 24px и peekSubmit chrome на compose Save).
- Редизайн карточки набора / catalog pack / answer playing-cards / maps.
- Смена правил submit (пустые слоты блокируют submit) и cascade clear слотов.
- Явный крестик закрытия в шапке compose (dismiss = backdrop / Esc).
- Новые npm-зависимости; смена Quasar theme tokens глобально.
- Server / room / schema / peeks budget.

## Capabilities

### New Capabilities

_(нет)_

### Modified Capabilities

- `content/packs`: layout и chrome `PackTaskTile` (incl. question 24px, centered raised slots, outline+icon actions); compose dialog width (+≈50), Save chrome, no Cancel, difficulty dots; MODIFIED SC-PACK-223/224/226/127/238/260/276/277/278/279/280; ADDED SC-PACK-281.

## Impact

- Client UI: task list tiles на surfaces набора, compose dialog width + Save chrome + larger interactive dots; shared peek CSS tokens MAY получить compose-only modifiers.
- OpenSpec: delta `content/packs`; Visual Spec в design.
- Тесты: client Vitest (tile layout/actions, compose width/dots/Save); server — без изменений.

## References

- Эталон шапки/слотов/question size: game peek + `PackTaskComposeDialog`.
- Эталон actions: `PackTaskSetCardTile` outline + icon (`ContentPackPage` #actions).
- Эталон Save: `GamePage` `.peek-submit-btn`.
- Канон portrait chrome / `--pack-card-*`: skill pack-cards.
- Sibling: `happy-tourist.github.io/AGENTS.md`; meta skills `work-with-styles` / pack-cards, content pages.
