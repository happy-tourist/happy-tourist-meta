# Content lib placement (`src/lib/content.ts`)

Read with the [core structure skill](SKILL.md). HTTP surface:
`work-with-routes/content.md`.

## Owns

- Unified packs list + favorites + author re-edit + staff take
- Working≠live → list `draft` + per-set `moderationStatus` without authorId fan-out (SC-PACK-171…179, 187)
- Staff omit working≠live `draft` / always clear retained working after live staff-save (SC-PACK-233; extends SC-PACK-230)
- NeverLive ghost for set author (SC-PACK-188…189)
- `retainedNeverLiveAuthorTaskSetCycle` (pending|needs_revision|**draft**|legacy cancelled) shared with `getAddTaskSet` / put draft / submit promote / `discardAddTaskSetDraft` (D9–D11 / SC-PACK-196/197/264…268)
- List `openRequestType` + `listCatalog` lightweight `taskSetsPreview` (COUNT-only; author neverLive SC-PACK-252/253)
- Soft-unpublish + my-moderation/staff queue
- Maps helpers live in `contentMaps.ts` (cancel→draft SC-MAP-41/42; live staff-save clear working SC-MAP-64)
- `defaultContentPacks.ts` — parse/eligible; grant/backfill **no-ops** (SC-PACK-170)

## Tree comment (content.ts)

`content.ts` — ensureContentTables + working copy + submitPack/add-task-set + staff lock/save + soft-unpublish pack+set in_catalog + pack unpublish cascade-cancel open req + mail SC-PACK-137…141 + previewPending live cards + authorDisplayName + needs-revision + cascadeNormalize ≠ answers_dirty + author delete + listMyModeration/listPending + authorWorkingDiffersFromLive list draft SC-PACK-175…179 + withTaskSetModerationStatuses / openStatusByContentSetId SC-PACK-171…174, 187 + staff SC-PACK-230 omit draft / SC-PACK-233 always clear working after live staff-save + retainedNeverLiveAuthorTaskSetCycle / neverLiveAuthorTaskSets SC-PACK-188…189 + getAddTaskSet/putAddTaskSet draft without submit-minima / submit draft→pending / cancel never-live→draft / discardAddTaskSetDraft D9–D11 SC-PACK-196/197/264…268 + list openRequestType + listCatalog lightweight `taskSetsPreview` count-only SC-PACK-252/253
