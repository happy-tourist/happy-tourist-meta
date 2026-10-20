## 0. Server — out of scope

- [x] 0.1 **Нет server-задач.** HTTP content persist, room/schema/messages и mocha — без изменений; list-tile secondary `description` резолвится на client из уже отдаваемого `answerCards` payload (`content` + `description`, slot `answerCardId`, `difficulty` 1|2|3). Verify: в этом change нет правок `happy-tourist-server`.

## 1. Client — PackTaskTile portrait + peek-like body

- [x] 1.1 Перед работой: `design.md` Decisions 1–4 / Visual Spec tile; delta SC-PACK-223/226/238/127/278; skills `.agents/skills/client/work-with-styles/pack-cards.md`, `client-work-with-test`; эталон header/slots в `PackTaskComposeDialog` / Game peek CSS (`app.scss` `.peek-diff-*`, `.peek-slot-like--portrait`); текущий `PackTaskTile.vue` + parents, передающие slots
- [x] 1.2 Перестроить `PackTaskTile` по Visual Spec: W 156–180 / min-H ≈206 / radius 12 / `--pack-card-*`; header h=20 dots 6px left + word badge `content.difficultyWord.*` center (**снять** outline digit `q-badge` / primary `content.difficulty.*`); вопрос 13–15px center + scroll; filled `.peek-slot-like--portrait` gap 8 + `overflow-x: auto`; empty не рендерить; при **нуле** filled — не рендерить ряд слотов и **не** показывать `content.slotEmpty` в body; Edit/Delete `--pack-card-action-h` — verify: light/dark; нет 300×200 / vertical splitter / digit badge
- [x] 1.3 Заменить prop `slotLabels: string[]` на filled-only `{ content; description? }[]` (напр. `filledSlots`); в parents (`ContentPackTasksPage` / `ContentPackAddTaskSetPage` / `ContentStaffRequestPage`) фильтровать empty `answerCardId`, резолвить `content` + optional `description` из загруженных `answerCards` (staff `task_set` preview — live answers, как сейчас SC-PACK-134); без нового HTTP; обновить stubs/Vitest SC-PACK-223/226/238/278 (+ SC-PACK-127 где есть description secondary) — verify: `npm test` по затронутым PackTaskTile / page tests в client
- [x] 1.4 Обновить skill/docs carve-out и размеры tile: `.agents/skills/client/work-with-styles/pack-cards.md` (task **300×200** + SC-PACK-238 compact labeled carve-out → portrait pack-card + filled `.peek-slot-like--portrait`); `.agents/skills/client/work-with-stores/content.md` (task 300×200 / carve-out Don’t); `.agents/skills/client/work-with-game-board/peek.md` (Don’t «force peek portrait AR onto PackTaskTile»); комментарии в `app.scss` про SC-PACK-238 — verify: нет устаревшего «list MUST NOT peek AR» / «task 300×200» для `PackTaskTile`

## 2. Client — Compose narrower + larger difficulty dots

- [x] 2.1 Перед работой: `design.md` Decisions 5–6 / Visual Spec compose; delta SC-PACK-260/276/279/280; `PackTaskComposeDialog.vue`; shared `.peek-diff-dot` (6px) vs compose-only scope
- [x] 2.2 Сузить compose card: `min(4×--peek-tile-max-w + 3×8px + pad, 92vw)`; убрать fixed min/max 360/400 — verify: pool ≤4 колонок; game peek width не тронут
- [x] 2.3 Compose-only dots Ø ≈20px (scoped `.pack-task-compose-card`); interactive preview/commit без регрессии; shared 6px base для game peek — verify: Vitest SC-PACK-276/279/280; peek dots unchanged
- [x] 2.4 `npm run lint` + `npm run typecheck` + `npm test` (compose + tile suites) в `happy-tourist.github.io` — verify: все зелёные

## 3. Client — polish (post-apply feedback)

- [x] 3.1 Перед работой: `design.md` Decisions 3/7–9 + Visual Spec polish; delta SC-PACK-223/224/278/260/279/281; эталон outline+icon в `PackTaskSetCardTile` / `ContentPackPage` #actions; эталон Save в `GamePage` `.peek-submit-btn`; текущий `PackTaskTile.vue` + `PackTaskComposeDialog.vue`
- [x] 3.2 `PackTaskTile`: вопрос **24px**; slots `justify-content: center` + воздух ≈8–12px над actions (не flex-grow void, прижимающий слоты к низу); Edit/Delete → `outline` + icon (`edit` / `delete`), slim `--pack-card-action-h`, Delete MAY `color="negative"` — verify: Vitest SC-PACK-223/224/278; light/dark
- [x] 3.3 `PackTaskComposeDialog`: width formula **+ ≈50px** slack; Save → full-width primary unelevated peekSubmit-like chrome (labels `saveTask`/`addTask`); **убрать** Cancel из UI (dismiss = backdrop/Esc; `@hide` / parents ok) — verify: Vitest SC-PACK-260/279/281; нет `cancelEditTask` btn; peek modal не тронут
- [x] 3.4 Skills notes: pack-cards / content — outline+icon actions, 24px question, centered/raised slots; compose Save / no Cancel / +50 width — verify: нет устаревшего «flat text Edit/Delete» / «13–15px question» / «Cancel required» для этих surfaces
- [x] 3.5 `npm run lint` + `npm run typecheck` + `npm test` (tile + compose suites) в `happy-tourist.github.io` — verify: все зелёные

## 4. Client — PackAnswerCardTile portrait pack-card

- [x] 4.1 Перед работой: `design.md` Decisions 11–15 + Visual Spec answer tile; delta SC-PACK-222/224/225/227/282; эталон размера/шрифта `PackTaskSetCardTile`; эталон actions `PackTaskTile`; текущий `PackAnswerCardTile.vue` + pages (`ContentPackEditorPage` / `ContentPackPage` / `ContentStaffRequestPage`)
- [x] 4.2 Перестроить `PackAnswerCardTile`: W **156** / min-H ≈**206** / `--pack-card-*` (один размер с/без desc); column body — с desc: content top + desc bottom; без desc: content center; type content ≈0.78rem / desc ≈0.68rem muted; Edit/Delete outline+icon (`edit`/`delete`), Delete MAY negative; сохранить selectable hover/selected — verify: нет 150×200 / 300×200 / horizontal splitter; light/dark
- [x] 4.3 Vitest SC-PACK-222/225/224/282 (+ page stubs asserting old sizes) — verify: `npm test` по PackAnswerCardTile / related editor suites
- [x] 4.4 Skills: `.agents/skills/client/work-with-styles/pack-cards.md` + content stores/pages — answer list **150/300** + vertical split → portrait pack-card + outline actions — verify: нет устаревшего «answer MUST 150×200/300×200» для list tile
- [x] 4.5 `npm run lint` + `npm run typecheck` + `npm test` (answer + task tile suites) в `happy-tourist.github.io` — verify: все зелёные

## 5. Client — answer 24px + tile Edit «Редактировать»

- [x] 5.1 Перед работой: `design.md` Decisions 7/13/14/16 + Visual Spec; delta SC-PACK-222/224/282/283; текущий `PackAnswerCardTile.vue` + `PackTaskTile.vue` + i18n `content.edit`
- [x] 5.2 `PackAnswerCardTile`: content `font-size` **24px** (вес 600; desc ≈0.68rem muted без изменений layout) — verify: нет content ≈0.78rem как эталон list content; light/dark
- [x] 5.3 `PackTaskTile` + `PackAnswerCardTile`: Edit label → `$t('content.edit')` («Редактировать»), **оставить** `icon="edit"`; Delete labels без смены; compose dialog titles (`editTask`/`editCard`) не трогать — verify: Vitest SC-PACK-224/283; нет `editTask`/`editCard` на tile Edit btn
- [x] 5.4 Skills notes pack-cards / content: answer content **24px** + tile Edit `content.edit` — verify: нет «answer content ≈0.78rem» / «editCard on list tile» как MUST
- [x] 5.5 `npm run lint` + `npm run typecheck` + `npm test` (answer + task tile suites) в `happy-tourist.github.io` — verify: все зелёные
