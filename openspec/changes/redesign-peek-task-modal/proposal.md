## Why

Модалка задания в игре (shared peek) визуально отстаёт от карточек наборов и макета: плоские chips, текст «Сложность: N», подписи в пустых слотах, CTA не в ритме pack-actions. Нужен редизайн light/dark с консистентностью точек сложности, status-badge, соотношения сторон слотов/ответов и паритетом compose-диалога задачи.

## What Changes

- Обновить chrome shared peek-модалки: шапка (3 точки сложности + badge со словом), пустые слоты без текста, заполненные как мини-карточки ответа, пул ответов с AR карточки набора, сетка до 4×4 со скроллом, full-width primary «Ответить».
- Синхронно адаптировать peek-like compose задачи (слоты + пул); выбор сложности — интерактивные точки (hover-превью, клик фиксирует) + badge «Сложность: …».
- Сохранить peek-поведение (shared modal, reuse одной карты в нескольких слотах, spectator read-only, серверный контракт без изменений).

## Scope

- **Capability ID:** `game/board` (peek modal UX), `content/packs` (task compose peek-like parity + difficulty picker chrome).
- **Пакеты:** client only (`happy-tourist.github.io`); server / room messages (`peek` / `peekPlace` / `peekSubmit`) / schema (`peekDifficulty`, `peekPlacements`, `answerCards`, …) / content HTTP difficulty persist — **без изменений** (frozen list in `design.md`). Delta `game/board` may restate SC-BOARD-43/44/47 as baseline under a MODIFIED parent; that is **not** BE scope — apply must not touch `happy-tourist-server` and must not add server tasks/tests.
- **Поверхности:** Game (peek modal), content task compose (creator/staff / add-task-set dialog).

## Out of scope

- Серверная логика peek / шаги за difficulty / deck bind.
- Playing-card list tiles (`PackAnswerCardTile` / task list grids) вне compose/peek.
- Редизайн HUD peeks, доски, других game-диалогов (кроме согласованного CTA-ритма).
- Новые npm-зависимости и смена Quasar theme tokens глобально.

## Capabilities

### New Capabilities

_(нет)_

### Modified Capabilities

- `game/board`: визуальные и UX-требования shared peek modal (header dots+badge, slot/pool chrome, grid/scroll, submit).
- `content/packs`: task compose — тот же slot/pool chrome; difficulty через точки + badge; обновление carve-out пула (не flat chips / не list playing-card tiles); **SC-PACK-237/238** — compose остаётся peek-aligned, `PackTaskTile` list — compact labeled carve-out.

## Impact

- Client UI: Game peek dialog, task compose dialog, shared slot/answer chrome, i18n difficulty word form.
- OpenSpec: delta `game/board`, `content/packs`; Visual Spec в design (макет + overrides консистентности).
- Тесты: client Vitest peek / task compose (layout/chrome assertions where covered).

## References

- Макет light+dark: prepare-mock extract (сессия); filenames `Gemini_Generated_Image_3xs1nh3xs1nh3xs1-*.jpg` — **не** обязательны в change dir; FE-канон = `design.md` Visual Spec + pack tokens.
- Канон точек/карточки набора: `PackTaskSetCardTile` / `--pack-card-*` / `--pack-ts-dot-*` (SC-PACK-229/239…248/255).
- Существующие peek/compose: SC-BOARD-42…49, SC-PACK-237/238/258…263; list tile slots (`PackTaskTile`) — carve-out (delta **MODIFIED** SC-PACK-238), не целевой chrome.
- Sibling: `happy-tourist.github.io/AGENTS.md`; meta skills `work-with-game-board` / `work-with-styles` (pack-cards).
