## Why

Каталоги наборов и карт уже показывают гостю identity-фильтры скрытыми, но оставляют одиночную кнопку «Все» и карточки soft-unpublished («СНЯТО») в общем списке — шум без пользы. Параллельно chrome списков (крошки, заголовок, subtitle, create в одном ряду с названием поверх иллюстрации) нужно приблизить к согласованным light/dark макетам, а гостю — дать увидеть CTA «Добавить набор заданий» с той же gate-модалкой, что у создания набора, и сохранить view-only вход в набор заданий.

## What Changes

- У anonymous на списках **Наборы** и **Карты** скрыть весь filter chrome, включая «Все» (список остаётся public catalog).
- Soft-unpublished (СНЯТО) элементы **каталожных списков** наборов и карт показывать **только staff**; non-staff их не видят в списке.
- На live pack detail: CTA «Добавить набор заданий» **видна** anonymous и unverified → при клике login / verify gate (как у создания набора на списке); права create на сервере не расширяются.
- Сохранить/починить: anonymous и unverified могут открыть опубликованный набор заданий для **просмотра** (view-only) — live drill-in (`view=live`) **без** login/verify gate (`ContentPackTasksPage.checkGate` сейчас блокирует load до чтения).
- Подтянуть page chrome списков Наборы/Карты под Visual Spec макета (крошки, title/subtitle, create на уровне title, фон illustration) — **без** wordmark в header и **без** смены chrome карточек карт на «2-4».

## Scope

- **Пакет:** client (`happy-tourist.github.io`). Server-контракт create/add-task-set для anonymous без изменений (reject как сейчас).
- **Capability ID:**
  - `content/packs` — guest filter chrome; soft-unpublished visibility на packs list; add-task-set CTA + gate для anonymous/unverified; view-only task-set (live boot без auth gate).
  - `content/maps` — guest filter chrome; soft-unpublished visibility на maps list.
  - `ui/branding` — page chrome / Visual Spec для списков Наборы и Карты (крошки, title block, actions row, header/page tokens по макету).

## Out of scope

- Wordmark «СЧАСТЛИВЫЙ ТУРИСТ» рядом с logo (сохраняем текущий SC-BRAND: mark without text label).
- Смена map-card capacity chrome на pill «2-4» (сохраняем строки ИГРОКОВ / ТУРИСТОВ).
- Soft-unpublished **наборы заданий** на pack detail (muted «СНЯТО» на live/editor) — без изменений; только каталожные списки packs/maps.
- Новые OAuth/провайдеры, server filter query params, staff moderation redesign.
- Pixel `prepare-mock` / `verify-mock` как отдельный workflow после apply (Visual Spec фиксируется в design для сверки).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: anonymous без filter chrome; soft-unpublished packs list только staff; add-task-set CTA + login/verify gate для anonymous/unverified; view-only task-set (без auth gate на live boot).
- `content/maps`: anonymous без filter chrome; soft-unpublished maps list только staff.
- `ui/branding`: измеримый Visual Spec chrome списков Наборы/Карты (light/dark) по макету.

## Impact

- Client: packs/maps list filter + soft-unpublish list filtering; pack detail add-task-set visibility + gate (anonymous/unverified); task-set live view-only without `checkGate` block; shared page chrome / styles / breadcrumbs presentation under Visual Spec.
- Meta: delta specs + design Visual Spec; sync client skills, которые сейчас канонизируют «гость видит Все» (SC-PACK-286 / SC-MAP-69) — `work-with-pages/content-pages.md`, `work-with-stores/content.md`, `work-with-stores/maps.md`, `client-work-with-test/stores.md` (см. `tasks.md` §6).
- Server: без обязательных изменений (см. `design.md` «Server contract») — anonymous create/add-task-set по-прежнему `registered_user_required` via `requireVerifiedEditor`; `listCatalog` / `listMaps` уже не отдают soft-unpublished non-staff; `getLivePack` уже отдаёт in-catalog anonymous (view-only на client); client omit СНЯТО — defense-in-depth.

## References

- Explore + prepare-mock (сессия): guest filters / СНЯТО / add-task-set gate / header chrome; mocks light+dark Maps (`.tmp-prepare-mock/` — локальный extract, не коммитить).
- `openspec/specs/content/packs/spec.md`, `openspec/specs/content/maps/spec.md`, `openspec/specs/ui/branding/spec.md`
- Архив [`openspec/changes/archive/2026-10-21-guest-auto-entry-catalog-ux`](../archive/2026-10-21-guest-auto-entry-catalog-ux/) (SC-PACK-286 / SC-MAP-69 — «Все» остаётся; этот change **разворачивает** guest All-filter).
- Sibling: `../happy-tourist.github.io/AGENTS.md`; client skills `work-with-pages`, `work-with-styles`, `client-work-with-auth`, `client-work-with-test`
- `docs/projects-map.md`
