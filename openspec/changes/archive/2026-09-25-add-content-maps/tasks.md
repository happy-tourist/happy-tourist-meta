## 1. Server — schema, lib, HTTP maps

- [x] 1.1 Прочитать `design.md`, delta `specs/content/maps/spec.md`, аналоги `src/lib/content.ts` / `src/db/schema.ts` / content endpoints в `app.config.ts`; skills `server-locate-change-points`, `work-with-database`, `work-with-routes`, `server-work-with-auth` (verify gate)
- [x] 1.2 Добавить map tables + ensure/migrate (maps, revisions, moderation type/`mapId` per design D2) и проверить boot без ошибок схемы
- [x] 1.3 Реализовать lib: create / draft get-put / submit (min starts = players×tourists) / approve / needs_revision / cancel / thread messages / staff lock+save / soft-unpublish+republish+cascade-cancel / author delete unpublished / list (public in-catalog + author own unpublished) / mail hooks — зеркало packs semantics из design D5/D8
- [x] 1.4 Зарегистрировать thin `createEndpoint` routes `/api/content/maps*` и расширить `my-moderation` + `staff/pending` type badge pack|map; проверить JWT + verify gates на create/edit/submit
- [x] 1.5 Mocha tests SC-MAP-01…03, 05, 09–23, 26–28 (и связанные ACL) в новом/расширенном test file; `npm test` в server — зелёный для map suite

## 2. Client — store, routes, Maps UI

- [x] 2.1 Прочитать design D6–D7, client skills `client-locate-change-points`, `work-with-pages`, `work-with-stores`, `work-with-forms`, `work-with-localization`, `client-work-with-auth`, `work-with-test`; аналог Content* pages + `stores/content.ts`
- [x] 2.2 Pinia `maps` store (HTTP list/create/draft/submit/moderation/staff/unpublish/delete) + error/loading; routes Maps list / editor / staff map branch; lobby nav «Карты»
- [x] 2.3 Maps list UI: approved для всех + own unpublished для автора; мини-превью + автор + N×M; Create map; без collection (SC-MAP-06…08)
- [x] 2.4 Paint editor: 10×10 + palette start/task/finish + players/tourists 1–4 + autosave + submit + thread; verify gate на create (SC-MAP-04…05, 09–13, 29)
- [x] 2.5 Staff queue / request hub: type badge map; approve / needs_revision / cancel; soft-unpublish confirm; staff Edit lock session; author «На модерации» includes maps (SC-MAP-14…18, 21–25)
- [x] 2.6 i18n RU keys для maps; `npm run lint` + `npm run typecheck` в client — без новых ошибок по затронутым файлам

## 3. Client — vitest + server integration check

- [x] 3.1 Vitest: list ACL / editor paint+submit gate / staff queue badge / my-moderation map row / soft-unpublish labeling (SC-MAP-04, 06–08, 14, 17, 21, 24–25, 29–30); `npm test` в client — зелёный
- [x] 3.2 Полный `npm test` server (регрессия packs + новые maps); убедиться tourist room create не требует map id (SC-MAP-28)
