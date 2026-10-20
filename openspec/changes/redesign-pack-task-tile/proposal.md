## Why

Карточка задания в списке набора (`PackTaskTile`) всё ещё в старом playing-card layout (широкая плитка, digit-badge, split вопрос|чипы), хотя модалки peek и compose уже на dots + word-badge + portrait filled slots. Нужен паритет list-tile с семьёй карточек набора и лёгкий polish compose: уже диалог и крупнее hit-target точек сложности. После первого apply — follow-up polish: actions как у набора, вопрос как в peek, слоты по центру с воздухом над actions, Save compose как peekSubmit без Cancel, ширина +≈50px. Следом — list-карточки ответов (`PackAnswerCardTile`) всё ещё 150/300×200 с горизонтальным split и flat-кнопками: нужен тот же portrait pack-card размер/шрифт, что у набора, и actions как у заданий.

## What Changes

- Редизайн list-карточки задания: portrait (ширина меньше высоты), chrome как у карточки набора; шапка как в game peek / compose (точки слева, word-badge по центру; без digit outline badge); текст задания **24px** (как game peek); внизу только заполненные answer mini-cards с optional description secondary, **горизонтально по центру**, с воздухом над actions (не прилипать к низу); пустые слоты и ряд при нуле filled — не показывать; при многих слотах — горизонтальный scroll; prop binding filled-only `{ content; description? }` вместо `slotLabels` с empty placeholders.
- Edit/Delete на editable **task** list tile: **outline + icon**, slim `--pack-card-action-h`, как у `PackTaskSetCardTile`; Delete MAY остаться negative/red.
- Compose (редактирование задания): ширина ≈ 4 mini-cards + pad **+ ≈50px** slack; интерактивные точки сложности ≈ высоте status-badge; кнопка Save — full-width primary unelevated как game peekSubmit; **Cancel убрать** (dismiss = backdrop / Esc).
- Снять carve-out SC-PACK-238: list-tile слоты больше не «compact labeled only» — filled slots в peek-подобном chrome.
- Редизайн list-карточки ответа (`PackAnswerCardTile`): **один** portrait размер как набор (**156 × min≈206**), независимо от наличия description; layout **column** — с description: content **сверху**, description **снизу**; без description: content **по центру**; шрифт content ≈ title набора (~0.78rem); actions **outline + icon** как у заданий (Delete MAY red); surface `--pack-card-*`. Снять прежний 150/300×200 + vertical splitter.

## Scope

- **Capability ID:** `content/packs` (list task tile + answer list tile UX + task compose dialog chrome).
- **Пакеты:** client (`happy-tourist.github.io`) only.
- **Поверхности:** список заданий набора (editor / view / add-task-set / staff preview), список answer cards (editor / live / staff), task compose dialog.
- Контракт HTTP/content persist — **без изменений**; **server — out of scope / FE-only**.

## Out of scope

- Game peek modal (ширина, размер точек, layout) — **оставить как сейчас** (кроме того что list task tile **переиспользует** peek question font-size 24px и peekSubmit chrome на compose Save).
- Peek / compose answer **mini-cards** pool (half-size portrait) — без изменений.
- Редизайн карточки набора / catalog pack / maps.
- Смена правил submit (пустые слоты блокируют submit) и cascade clear слотов.
- Явный крестик закрытия в шапке compose (dismiss = backdrop / Esc).
- Answer compose dialog chrome (кроме того что list tile меняется).
- Новые npm-зависимости; смена Quasar theme tokens глобально.
- Server / room / schema / peeks budget.

## Capabilities

### New Capabilities

_(нет)_

### Modified Capabilities

- `content/packs`: `PackTaskTile` + compose polish; **`PackAnswerCardTile` portrait pack-card** (supersedes 150/300 vertical split); MODIFIED SC-PACK-222/223/224/225/226/127/238/260/276/277/278/279/280; ADDED SC-PACK-281/282.

## Impact

- Client UI: task + answer list tiles; compose dialog width + Save chrome + larger interactive dots.
- OpenSpec: delta `content/packs`; Visual Spec в design.
- Тесты: client Vitest (task tile, answer tile, compose); server — без изменений.

## References

- Эталон размера/шрифта ответа: `PackTaskSetCardTile` (156×206, title ~0.78rem).
- Эталон actions: `PackTaskTile` / task-set outline + icon.
- Эталон Save: `GamePage` `.peek-submit-btn`.
- Sibling: `happy-tourist.github.io/AGENTS.md`; meta skills `work-with-styles` / pack-cards.
