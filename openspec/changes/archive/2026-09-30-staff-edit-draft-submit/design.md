## Context

См. `proposal.md` — Why. Сейчас staff на never-published и на published-as-creator часто остаётся в author-path (`staffMode=false`, putDraft + Submit). `staffSavePack` чистит twin working только если working был близнецом **старого** live (SC-PACK-230); `staffSaveMap` twin/working не чистит. CSV на `ContentPackTasksPage` рендерится и в `viewOnly`. Слоты в форме/плитках — `q-chip`, peek — `.peek-slot` (~72×40).

## Goals / Non-Goals

**Goals:**

- Published + staff → всегда staff-save; Submit скрыт; working cleared после save без open author request.
- Never-published + staff creator → Submit на модерацию сохранить.
- Author Submit → dirty vs load baseline.
- Live view task-set → без CSV; staff Edit / editor → CSV как сейчас.
- Slot chrome редактора и `PackTaskTile` ≈ peek.

**Non-Goals:**

- One-shot SQL migration всех orphan working.
- Авто-publish never-published без Submit для staff.
- Менять peek gameplay или CSV формат.

## Decisions

### D1 — Boot: `hasLive` + `isStaff` → staffMode

- **Choice:** В `ContentPackEditorPage` / `MapEditorPage` (и staff entry с list): если `auth.isStaff && hasLive` → `enterStaffEdit` **до** ветки creator/author. Never-published creator (в т.ч. staff) → текущий author path с Submit.
- **Why:** D2 из explore; maps list уже `?staff=1`, packs boot сейчас пропускает creator-staff в author path.
- **Alt:** Прятать только кнопку Submit у staff — отвергнуто (оставит putDraft / false draft).

### D2 — Staff-save always clears working when no open request

- **Choice:** После успешного staff-save в live: `workingRevisionId = null`; orphan revision удалить, если нет moderation refs. Не только twin-of-old-live.
- **Why:** D3; висящий draft у creator-admin после staff правок.
- **Risk:** Retained cancel-draft автора на том же паке/карте сбрасывается staff-save — принято (staff live authoritative; open request всё ещё блокирует staff-save).
- **Packages:** `content.ts` `staffSavePack`; `contentMaps.ts` `staffSaveMap`.

### D3 — Dirty Submit via pristine snapshot

- **Choice:** После load клонировать baseline; `canSubmit` требует `local !== baseline` (semantic JSON/fingerprint) плюс minima / locks. После successful submit/reload — обновить baseline.
- **Surfaces:** pack editor, tasks editor (author), add-task-set, map editor (non-staff).
- **Alt:** Серверный dirty — YAGNI.

### D4 — CSV: `v-if="!viewOnly"` on tasks page

- **Choice:** `PackTasksCsvControls` только когда не live view-only; staff Edit (`liveViewMode && staffMode` → не viewOnly) оставляет CSV.
- **File:** `ContentPackTasksPage.vue`.

### D5 — Shared peek-like slot chrome

- **Choice:** Вынести/переиспользовать стили уровня `.peek-slot` (min-width/height, padding, dashed/solid) для compose slot row и `PackTaskTile` slot list; не копировать peek interaction из GamePage.
- **Why:** едино с выполнением задания; список и форма одинаково.

## Risks / Trade-offs

- [Staff-save destroys author cancel-draft] → Mitigation: только без open request; задокументировано в proposal Out of scope / Risk.
- [Fingerprint false dirty] → Mitigation: стабильный serialize без volatile ids where possible; refresh baseline after quiet save echo if needed.
- [CSV SC-PACK-217 wording] → ADDED SC-PACK-235 clarifies live view exclusion.

## Migration Plan

- Deploy server+client вместе (working clear + boot).
- Существующие hanging draft: heal на следующем staff-save; без SQL one-shot.

## Technical prerequisites

- Нет внешних сервисов / новых deps.
- Explore D1–D3 закрыты продуктовыми решениями (create keeps Submit; published staff-save; clear working).

## Implementation touchpoints

- Server: `../happy-tourist-server/src/lib/content.ts`, `contentMaps.ts`; mocha `zz-contentPacks.test.ts`, `zz-contentMaps.test.ts`.
- Client: `ContentPackEditorPage.vue`, `MapEditorPage.vue`, `ContentPackTasksPage.vue`, `ContentPackAddTaskSetPage.vue`, `PackTaskTile.vue` (+ shared slot styles); vitest pages/components.
- Чеклист apply — `tasks.md`.
