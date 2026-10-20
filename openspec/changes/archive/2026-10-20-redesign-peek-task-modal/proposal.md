## Why

Модалка задания в игре (shared peek) визуально отстаёт от карточек наборов и макета: плоские chips, текст «Сложность: N», подписи в пустых слотах, CTA не в ритме pack-actions. Нужен редизайн light/dark с консистентностью точек сложности, status-badge, соотношения сторон слотов/ответов и паритетом compose-диалога задачи. После apply/polish — уточнение шапки (точки слева, статус по центру) и правило peeks: каждый open (включая повтор на flipped) тратит просмотр; при 0 — открыть нельзя.

## What Changes

- Обновить chrome shared peek-модалки: шапка (3 точки сложности + badge со словом), пустые слоты без текста, заполненные как мини-карточки ответа, пул ответов с AR карточки набора, сетка до 4×4 со скроллом, full-width primary «Ответить».
- Синхронно адаптировать peek-like compose задачи (слоты + пул); выбор сложности — интерактивные точки (hover-превью, клик фиксирует) + badge со словом сложности.
- **Polish:** ширина слотов/пула **−50%** (~86→~43px), AR 156∶206; badge = только слово; вопрос в game peek ×1.5 (~24px); заполненный слот — тонкая primary-рамка; пул без used-chrome.
- **Header layout:** dots — **слева сверху у левого края**; word badge — **по центру** модалки/диалога (не одна centered-группа). Compose — то же.
- **Peeks budget (server + client UX):** каждый успешный `peek` open — и **fresh**, и **flipped** — в finite mode **MUST consume 1 peek**; при peeks **0** (и не solo ∞) open **MUST** быть rejected на любой клетке. Неверный ответ по-прежнему KEEP bind; повтор возможен только если peeks ≥ 1 (и снова −1). Solo `infinite` peeks — без лимита (как сегодня). Auto end-turn / legal-peek helpers больше не считают flipped peek «бесплатным действием при 0».
- Shared modal, reuse одной карты в нескольких слотах, spectator read-only — без смены payload’ов сообщений.

## Scope

- **Capability ID:** `game/board` (peek modal UX + peeks spend rules), `game/move` (multi-peek / flipped consume / auto-end vs peeks), `content/packs` (task compose chrome parity).
- **Пакеты:** client (`happy-tourist.github.io`) **и** server (`happy-tourist-server`) для peeks-на-`peek` + auto-end helpers; schema/message **names/payloads** без новых полей; content HTTP difficulty persist — без изменений.
- **Поверхности:** Game (peek modal + eye affordance gating), content task compose, server `MyRoom` peek/budget/auto-end.

## Out of scope

- Смена resolve Correct/Incorrect (KEEP vs remove tile / +difficulty steps) — as-is.
- Playing-card list tiles вне compose/peek.
- Редизайн HUD peeks copy, доски, других game-диалогов (кроме CTA-ритма).
- Увеличение шрифта вопроса в compose `q-input`.
- Новые npm-зависимости и глобальная смена Quasar theme tokens.

## Capabilities

### New Capabilities

_(нет)_

### Modified Capabilities

- `game/board`: peek modal chrome (header layout dots-left / badge-center; polish size/badge/question/filled border) + **MODIFIED** peeks spend (SC-BOARD-45/10 — каждый open тратит; 0 = reject).
- `game/move`: **MODIFIED** multi-peek / flipped consume / auto-end (SC-MOVE-39/94/95 и related requirement text).
- `content/packs`: compose parity (slot/pool/dots+badge layout; header dots-left / badge-center; SC-PACK-237/238 carve-out).

## Impact

- Client UI: Game peek dialog, compose dialog, shared chrome, i18n; eye/peek affordance must respect peeks=0 on flipped cells.
- Server: `MyRoom` peek open consume + reject at 0 for flipped; auto-end / `hasLegalPeek` helpers aligned.
- OpenSpec: delta `game/board`, `game/move`, `content/packs`; Visual Spec + design decisions.
- Тесты: client Vitest (header layout); server mocha SC-BOARD-45/10, SC-MOVE-94/95 (и связанные).

## References

- Макет + Visual Spec overrides в `design.md`.
- Канон точек/карточки: `PackTaskSetCardTile` / `--pack-card-*`.
- Sibling AGENTS + skills `work-with-game-board`, `work-with-game` / messages (peek).
