## 1. Server — clear working after staff-save

- [x] 1.1 Прочитать design D2; skills `server-work-with-structure`, `work-with-routes`, `work-with-database`; фрагмент `staffSavePack` / twin clear в `../happy-tourist-server/src/lib/content.ts` — подтвердить точку врезки
- [x] 1.2 В `staffSavePack`: после успешного save в live при отсутствии open author request всегда очищать `workingRevisionId` и удалять orphan revision без moderation refs (не только twin) — verify mocha SC-PACK-233 (+ регрессия SC-PACK-230)
- [x] 1.3 В `staffSaveMap` (`contentMaps.ts`): та же очистка working после staff-save в live без open map request — verify mocha SC-MAP-64
- [x] 1.4 `npm test` в `../happy-tourist-server` (затронутые zz-contentPacks / zz-contentMaps) — зелёный прогон

## 2. Client — staff boot on published

- [x] 2.1 Прочитать design D1; skills `client-work-with-structure`, `work-with-pages` / content-pages, `work-with-stores` content/maps; boot `ContentPackEditorPage` / `MapEditorPage`
- [x] 2.2 Pack editor: `isStaff && hasLive` → `enterStaffEdit` до creator/author ветки; never-published staff creator сохраняет Submit — verify vitest SC-PACK-231/232
- [x] 2.3 Map editor: published + staff → staffMode / staff-save; never-published staff creator → Submit — verify vitest SC-MAP-62/63
- [x] 2.4 `npm test` (затронутые page tests) в `../happy-tourist.github.io`

## 3. Client — dirty Submit

- [x] 3.1 Design D3; pristine snapshot после load на pack editor, tasks editor (author), add-task-set, map editor — `canSubmit` требует dirty + minima/locks — verify vitest SC-PACK-234 / SC-MAP-65
- [x] 3.2 После successful submit/reload обновить baseline; staff surfaces без Submit не ломать — verify vitest


## 4. Client — CSV hide + slot chrome

- [x] 4.1 Design D4: `ContentPackTasksPage` — не рендерить `PackTasksCsvControls` при `viewOnly`; staff Edit оставляет CSV — verify vitest SC-PACK-235/236
- [x] 4.2 Design D5: peek-comparable slot chrome для compose slot row (Tasks + AddTaskSet) и `PackTaskTile` — verify vitest SC-PACK-237/238 (класс/размеры)
- [x] 4.3 `npm test` + `npm run lint` + `npm run typecheck` в client по затронутым файлам

## 5. Cross-check

- [x] 5.1 Server `npm test` полный или целевой suite zz-content* — зелёный
- [x] 5.2 Client `npm test` по content/maps/pack CSV/tile suites — зелёный; регрессия SC-PACK-230 UX не сломана
