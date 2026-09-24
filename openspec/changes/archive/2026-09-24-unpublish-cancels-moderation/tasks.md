## 1. Server — cascade cancel + email

- [x] 1.1 Прочитать `design.md`, delta `specs/content/packs/spec.md` (SC-PACK-137…141), skills `server-work-with-routes` / `server-work-with-test`; открыть `unpublishPack` и `notifyChangeAuthor` в server `src/lib/content.ts`
- [x] 1.2 В `unpublishPack`: после успешного `inCatalog: false` отменить все open requests по packId; early-return уже unpublished — без повторного cancel/mail; проверить mocha SC-PACK-137/138
- [x] 1.3 После cascade: один RU email без ссылок на каждого distinct change author (SC-PACK-140/141); anonymous skip; убедиться что ручной `cancelRequest` по-прежнему без mail
- [x] 1.4 Добавить/обновить mocha на unpublish+open request+mail; `npm test` в server проходит для затронутых suites

## 2. Client — confirm copy

- [x] 2.1 Найти confirm soft-unpublish пака (catalog/collection/live) + i18n; skills `work-with-localization` / `work-with-test`
- [x] 2.2 Обновить текст confirm: предупреждение об отмене открытых заявок (SC-PACK-139); cancel dialog = no-op; vitest на copy/наличие предупреждения
- [x] 2.3 `npm test` / lint / typecheck в client для затронутых файлов — зелёные
