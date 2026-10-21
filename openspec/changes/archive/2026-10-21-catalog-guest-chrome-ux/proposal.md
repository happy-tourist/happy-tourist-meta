## Why

Каталоги наборов и карт уже показывают гостю identity-фильтры скрытыми, но оставляют одиночную кнопку «Все» и карточки soft-unpublished («СНЯТО») в общем списке — шум без пользы. Параллельно chrome списков (крошки, заголовок, subtitle, create в одном ряду с названием поверх иллюстрации) нужно приблизить к согласованным light/dark макетам, а гостю — дать увидеть CTA «Добавить набор заданий» с той же gate-модалкой, что у создания набора, и сохранить view-only вход в набор заданий.

После первой apply-итерации всплыли регрессии: create на каталогах ушёл от lobby Quasar primary; solid cream/navy на `.pack-card-grid` перекрыл translucent underlay (SC-PACK-285); soft-unpublished **наборы заданий** на live pack detail по-прежнему видны non-staff (SC-PACK-132), хотя каталожные СНЯТО уже только staff. Плюс sticky game HUD остаётся непрозрачным поверх page art.

## What Changes

- У anonymous на списках **Наборы** и **Карты** скрыть весь filter chrome, включая «Все» (список остаётся public catalog).
- Soft-unpublished (СНЯТО) элементы **каталожных списков** наборов и карт показывать **только staff**; non-staff их не видят в списке.
- На live pack detail: CTA «Добавить набор заданий» **видна** anonymous и unverified → при клике login / verify gate (как у создания набора на списке); права create на сервере не расширяются.
- Сохранить/починить: anonymous и unverified могут открыть опубликованный набор заданий для **просмотра** (view-only) — live drill-in (`view=live`) **без** login/verify gate (`ContentPackTasksPage.checkGate` сейчас блокирует load до чтения).
- Подтянуть page chrome списков Наборы/Карты (крошки, title/subtitle, create на уровне title) — **без** wordmark в header и **без** смены chrome карточек карт на «2-4».
- **Follow-up (post-apply):** create на Наборы/Карты вернуть к тому же Quasar primary, что «Создать игру» в лобби (без custom `.catalog-list-create` sizing).
- **Follow-up:** убрать solid page-canvas override на catalog `.pack-card-grid` — снова translucent underlay SC-PACK-285 (`rgba(…, 0.78)`).
- **Follow-up:** soft-unpublished **наборы заданий** на live pack detail — **только staff** (как packs/maps lists); non-staff не видят карточку (пересмотр SC-PACK-132).
- **Follow-up:** sticky bottom `.game-hud` на Game — translucent фон с тем же alpha `0.78` (light/dark), чтобы page art просвечивал.

## Scope

- **Пакет:** client (`happy-tourist.github.io`). Server-контракт create/add-task-set для anonymous без изменений (reject как сейчас).
- **Capability ID:**
  - `content/packs` — guest filter chrome; soft-unpublished visibility на packs list **и** на live task-set list; add-task-set CTA + gate; view-only task-set.
  - `content/maps` — guest filter chrome; soft-unpublished visibility на maps list.
  - `ui/branding` — list title chrome; create = lobby primary parity; catalog grid keeps translucent underlay (no solid cream/navy override).
  - `game/presence` — sticky bottom game HUD translucent underlay (alpha 0.78).

## Out of scope

- Wordmark «СЧАСТЛИВЫЙ ТУРИСТ» рядом с logo (сохраняем текущий SC-BRAND: mark without text label).
- Смена map-card capacity chrome на pill «2-4» (сохраняем строки ИГРОКОВ / ТУРИСТОВ).
- Top presence row (`.presence-row--top`) — без отдельного opaque panel chrome в этом change.
- Server list API shape / soft-unpublish storage rules; OAuth; staff moderation redesign.
- Pixel `prepare-mock` / `verify-mock` как отдельный workflow (Visual Spec в design — для title/header; create sizing tokens **superseded** lobby parity).

## Capabilities

### New Capabilities

- (нет)

### Modified Capabilities

- `content/packs`: anonymous без filter chrome; soft-unpublished packs list только staff; **live soft-unpublished task sets только staff**; add-task-set CTA + gate; view-only task-set.
- `content/maps`: anonymous без filter chrome; soft-unpublished maps list только staff.
- `ui/branding`: title chrome списков; create = lobby Quasar primary; **no** solid catalog grid canvas (SC-PACK-285 translucent).
- `game/presence`: sticky bottom `.game-hud` translucent (alpha 0.78).

## Impact

- Client: packs/maps filters + soft-unpublish omit; pack detail add-task-set gate + live view-only; **omit soft-unpub task-set cards for `!isStaff`**; restore translucent `.pack-card-grid`; rollback create btn; translucent `.game-hud`.
- Meta: delta specs + design; skills sync (в т.ч. SC-PACK-132 revise, SC-PACK-285, game HUD).
- Server: без обязательных изменений (см. `design.md` «Server contract»).

## References

- Explore + prepare-mock (сессия); follow-up explore 2026-10-21 (create/underlay/СНЯТО task-set/game HUD).
- `openspec/specs/content/packs/spec.md`, `content/maps`, `ui/branding`, `game/presence`
- Архив [`openspec/changes/archive/2026-10-21-guest-auto-entry-catalog-ux`](../archive/2026-10-21-guest-auto-entry-catalog-ux/)
- Sibling: `../happy-tourist.github.io/AGENTS.md`; `docs/projects-map.md`
