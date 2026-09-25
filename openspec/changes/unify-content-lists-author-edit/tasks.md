## 1. Server — schema, list, drop collection grants

- [x] 1.1 Прочитать `design.md` (D1–D9), delta `specs/content/packs/spec.md` + `specs/content/maps/spec.md`, skills `server-locate-change-points`, `work-with-database`, `work-with-routes`; сверить аналоги lock/TTL в `lib/content.ts` / `lib/contentMaps.ts`
- [x] 1.2 Добавить favorites table + moderation `taken_by`/`taken_at` (ensure/migrate); verify boot без ошибок схемы
- [x] 1.3 Unified packs list (in-catalog + own never-published + staff soft-unpub) с `moderationStatus` / mine / favorite / contribution flags; выключить product-paths коллекции и default-grant (`defaultContentPacks` no-op); verify SC-PACK-148…150, 170 (mocha)
- [x] 1.4 Favorites star/unstar HTTP (registered only, in-catalog); verify SC-PACK-154/155
- [x] 1.5 add-task-set без `inCollection`; verify SC-PACK-164/165
- [x] 1.6 Maps list: `moderationStatus` в summary; verify SC-MAP-31/32

## 2. Server — author re-edit, locks, moderation take

- [x] 2.1 Author / task-set-author working-copy Edit + submit после publish (packs); staff Edit/save reject при open author request; verify SC-PACK-156…159, 111
- [x] 2.2 Edit lock acquire для авторов (тот же TTL); verify SC-PACK-160
- [x] 2.3 Moderation take/release + gate approve/needs_revision/cancel; author resubmit while taken; verify SC-PACK-161…163
- [x] 2.4 Maps: author re-edit через очередь + creator lock + staff block при open request; verify SC-MAP-17, 35…37
- [x] 2.5 Maps moderation take; verify SC-MAP-38/39
- [x] 2.6 `npm test` в `../happy-tourist-server` — зелёный на новых SC

## 3. Client — unified packs list, filters, favorites

- [x] 3.1 Прочитать client skills `client-locate-change-points`, `work-with-pages`, `work-with-stores` (content/maps topics), `work-with-localization`, `work-with-test`; `design.md` D9
- [x] 3.2 Lobby → unified packs list; убрать/redirect коллекцию; убрать non-staff my-moderation nav; verify SC-PACK-41, 166 (vitest)
- [x] 3.3 Статусы + фильтры (all / moderation / drafts / mine / favorites) на списке наборов; verify SC-PACK-148…153
- [x] 3.4 Звезда в списке и на detail; store favorites API; verify SC-PACK-154/155
- [x] 3.5 add-task-set UI без коллекции; verify SC-PACK-108, 164/165

## 4. Client — author edit, staff take, maps parity

- [x] 4.1 Author / task-set-author Edit affordances + lock UX после publish; staff Edit hidden/blocked при open author request; verify SC-PACK-156…160
- [x] 4.2 Staff queue + detail: «Взять в модерацию»; actions только после take; verify SC-PACK-161…163
- [x] 4.3 Maps list: статусы pending/needs_revision + фильтры (без favorites); hide my-moderation nav; verify SC-MAP-31…34, 40
- [x] 4.4 Maps author re-edit + lock + staff take UI; verify SC-MAP-17, 35…39
- [x] 4.5 i18n RU ключи (фильтры, статусы, take, favorites, errors)
- [x] 4.6 Vitest на list/filter/ACL/take/maps; `npm test` + `npm run lint` + `npm run typecheck` в `../happy-tourist.github.io` — зелёные

## 5. Server — per-set status + cancel→draft

- [x] 5.1 Прочитать `design.md` D10–D11/D15, delta SC-PACK-171…179, SC-MAP-41/42; skills `server-locate-change-points`, `work-with-database`, `work-with-routes`
- [x] 5.2 Live/editor pack payload: per-set `moderationStatus` (pending/needs_revision/draft/live) для set author + staff; pack creator без чужих; verify SC-PACK-171…174 (mocha)
- [x] 5.3 Cancel (staff|author) → keep working; author list status `draft` (+ drafts filter); others keep live snapshot; verify SC-PACK-175…179 (mocha)
- [x] 5.4 Maps cancel→draft parity; verify SC-MAP-41/42 (mocha)
- [x] 5.5 `npm test` в `../happy-tourist-server` — зелёный на новых SC блока 5

## 6. Client — set statuses, cancel→draft UX

- [x] 6.1 Прочитать client skills pages/stores/localization/test; design D10–D11; delta SC-PACK-171…179
- [x] 6.2 Live pack task-set rows: статусы для set author + staff; creator без чужих; verify SC-PACK-171…174 (vitest)
- [x] 6.3 List badges/filters после cancel → draft для автора; live для остальных; maps parity; verify SC-PACK-175…179, SC-MAP-41/42 (vitest)

## 7. Client — header, breadcrumbs, Game leave

- [x] 7.1 Прочитать design D12–D14, delta `ui/branding` SC-BRAND-11…16, `game/leave` SC-LEAVE-08…12; skills `work-with-pages`, `work-with-styles`, `work-with-localization`, `work-with-test`
- [x] 7.2 App header: секции справа от logo; Acc/Theme/Logout; staff «Модерация»; бургер на узком viewport; убрать duplicate section toolbar с Lobby; verify SC-BRAND-11…15 (vitest)
- [x] 7.3 Убрать embedded «Модерация» из chrome списков packs/maps; verify SC-PACK-183/184, SC-MAP-43 (vitest)
- [x] 7.4 Breadcrumbs под шапкой (packs + maps); убрать «К наборам» / «Вернуться» где крошки закрывают путь; verify SC-PACK-180…182, SC-MAP-44, SC-BRAND-16 (vitest)
- [x] 7.5 Game: leave справа + logo leave; без session logout и без section nav; verify SC-LEAVE-09…12, 08 (vitest)
- [x] 7.6 i18n RU для header/crumbs/leave; `npm test` + `npm run lint` + `npm run typecheck` в client — зелёные

## 8. Client — no in_catalog badge; map View/Edit; never-published → Edit

- [x] 8.1 Прочитать design D16–D18, delta SC-PACK-148/185/186, SC-MAP-45…49; skills `work-with-pages`, `work-with-stores` (maps topic), `work-with-test`
- [x] 8.2 Убрать бейдж «В каталоге» / in_catalog на списках packs и maps; pending/needs_revision/draft/unpublished оставить; verify SC-PACK-148/185, SC-MAP-45 (vitest)
- [x] 8.3 Never-published pack/map из списка → сразу Edit; verify SC-PACK-186, SC-MAP-49 (vitest)
- [x] 8.4 Published map: row → View без paint tools (автор + players×tourists); Edit с списка и из View + lock; verify SC-MAP-46…48 (vitest)
- [x] 8.5 Pack live browsing без strip-down (как сейчас); `npm test` + lint + typecheck — зелёные
