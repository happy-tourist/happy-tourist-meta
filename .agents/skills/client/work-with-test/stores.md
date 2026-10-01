# Work With Stores Tests

Use with the [core test skill](SKILL.md) when the unit under test depends on a
Pinia store, or when the store itself is the SUT.

Product store conventions: meta skill `work-with-stores`. Domains: `auth`,
`theme`, `game`, `support` (`change_pack`+`packId`; pages
`SupportChangePack.test.ts`, store `support.changePack.test.ts`), `content`
(HTTP packs via `client.http`; working copy / `submitPack` / add-task-set /
staff lock+save / soft-unpublish pack+set/`inCatalog` / needs_revision; cascadeGap* / hasLive confirm; simplify ACL
SC-PACK-100…136 — pages `Content*Acl.test.ts` incl. `ContentCatalogAcl` /
`ContentCollectionAcl` / `ContentPackAcl`, `ContentSoftUnpublish.test.ts`
(SC-PACK-120…125 staff unpublish/republish + soft-unpublished gray/non-nav),
`ContentFollowUp4.test.ts` (SC-PACK-129…133 + SC-PACK-139 unpublishConfirm warns open requests / set soft-hide /
live summary+drill-in / AddTaskSet chips), `ContentFollowUp5.test.ts`
(SC-PACK-134…136 slot card text / ordinal `taskSetLabel` no author / Tasks
`content.back`),
`ContentPackTasksHints.test.ts` (SC-PACK-115/119 staff session + stable hints),
`ContentModerationUx.test.ts` (SC-PACK-126 editor cascade CSS; SC-PACK-127 slot
chips on staff hub + add-task-set; SC-PACK-128 add-task-set thread/status/reply;
SC-PACK-194/195 no `content.catalogNav` on pack/staff/my-moderation pages),
pack CSV SC-PACK-210…229/239…248: `src/lib/__tests__/packContentCsv.test.ts`,
`src/components/__tests__/PackTasksCsvControls.test.ts`,
`src/pages/__tests__/ContentPackEditorCsv.test.ts` (framed CSV panel +
in-frame errors — not `content.error`; no `PackCsvImportDialog`), tile smoke
`PackAnswerCardTile` / `PackTaskTile` / `ContentPackPlayingCards` +
sizes/contrast SC-PACK-225…227 + catalog `PackListCardTile` /
`ContentPackListCards` SC-PACK-228/249…251 (~180×260 + set preview + overflow +
outline actions + revise badge) + task-set `PackTaskSetCardTile` /
`src/components/__tests__/PackTaskSetCardTile.test.ts` +
`ContentPackListCards` SC-PACK-229/239…248 (summary chrome / `#{n}` / colored outline
dots / pale dividers between every stats row + above actions / SC-PACK-244
lead/label/count grid + reserved top without badge / slim `dense` ~28–32px
actions / empty `#actions` omitted for viewers; soft muted `--pending`/`--muted`
pills + pale action outline (not Quasar solid fills); SC-PACK-245→247 spacing
retune (`--pack-ts-title-gap: 7px`, `--pack-ts-divider-air: 2–3px`); SC-PACK-246
themed hover no-scale; SC-PACK-248 SVG total+badge CSS masks from
`src/assets/content/task-set-*.svg` + `data-icon` / `pack-task-set-status-icon--*`;
muted soft-unpublish `opacity: 0.72` + `border-style: dashed`; short badges /
outline+icon; card «Снять»/«Вернуть»; no author on `taskSetLabel`; tile file
`PackTaskSetCardTile.test.ts`) + maps list `MapListCardTile` /
`ContentMaps` SC-MAP-55,
staff-edit-draft-submit SC-PACK-231…238 / SC-MAP-62…65:
`src/lib/__tests__/editorDirty.test.ts`,
`src/pages/__tests__/ContentPackStaffBoot.test.ts`,
`ContentPackDirtySubmit.test.ts`, `ContentPackTasksCsvHide.test.ts`
(+ `ContentMaps` staff boot / dirty Submit; `PackTaskTile` peek-slot chrome),
store `content.simplifyAcl.test.ts`; spy `client.http`, do not hit live server;
cascade helpers + `isStaffEditSessionNavigation`: meta `work-with-stores/content.md`;
unify follow-ups: `ContentUnifiedList.test.ts` (`openRequestType` pack→Edit /
task_set→live SC-PACK-191/192; cancel→draft; no published badge),
`ContentAuthorEditTake.test.ts` (author Edit/take + set marks + `neverLive`
ghost `pack-task-set-ghost-*` → add-task-set Edit SC-PACK-188…190),
`maps` (HTTP maps via `client.http`; paint helpers; pages
`ContentMaps.test.ts` SC-MAP-06…08 / 14 / 17 / 21 / 24–25 / 29–30 / 41…56 /
  62…65
  (cancel→draft, view meta, title-row Edit SC-MAP-51, author pending→Edit
  SC-MAP-50, paint race SC-MAP-53, palette under SC-MAP-54, `MapListCardTile`
  SC-MAP-55, centered seats SC-MAP-56, staff published boot SC-MAP-62,
  never-published Submit SC-MAP-63, dirty gate SC-MAP-65, no published badge); store
`maps.paint.test.ts` SC-MAP-04; errors via `mapsErrorI18nKey` → `maps.errors.*`;
meta `work-with-stores/maps.md` — shared staff queue still mocked via `content`
when testing type-badge / my-moderation map rows).
App shell crumbs: `src/__tests__/AppHeaderChrome.test.ts` — breadcrumbs **inside**
`q-page-container` (not elevated `q-header`; SC-BRAND-17…20 / SC-PACK-193 / SC-MAP-52);
narrow burger fold Acc/Theme/Logout (SC-BRAND-19). Moderation pages omit
`content.catalogNav`: `ContentModerationUx` SC-PACK-194/195.
Lobby create/list ordinals (no set author): `LobbyCreateWire` SC-LOBBY-24/26
(`taskSetOption` / `taskSetLabel` «Набор заданий #{n}» / pack-index ordinals when soft sets hide ahead).

## Testing a page / component that uses a store

- Declare store method mocks as separate variables outside `vi.mock`.
- Prefer plain state values on a mutable `mockState` object; assign before
  mount.
- Use real Vue `ref(...)` only when the test needs reactive updates after mount.
- Never pass ref-like `{ value: … }` into a mocked store that the page reads as
  plain state — match how the page consumes the store (setup stores expose
  refs that unwrap in templates; prefer returning plain fields from the mock
  unless the SUT reads `.value` in script).

```typescript
const mockLogin = vi.fn();
const mockState = {
  error: null as string | null,
  loading: false,
};

vi.mock('@/stores/auth', () => ({
  useAuthStore: vi.fn(() => ({
    error: mockState.error,
    loading: mockState.loading,
    login: mockLogin,
  })),
}));
```

If `vi.mock` needs a variable before initialization, use `vi.hoisted()` (see
[SKILL.md](SKILL.md) → Hoisted Mocks).

Do **not** mock `pinia` / `storeToRefs` locally unless the global setup already
documents a different pattern — prefer module mock of `use*Store`.

## Testing the store itself

- Use a real Pinia instance: `createPinia()` + `setActivePinia(createPinia())`
  in `beforeEach`.
- Spy on Colyseus I/O (`client.auth`, `client.http`, room helpers) — do not
  hit a live server → [colyseus.md](colyseus.md).
- Assert public actions / returned state / `error` / loading flags →
  [errors.md](errors.md).
- Do not assert private `_` helpers of the options `game` store unless they are
  the only way to observe a contract (prefer public actions).

```typescript
import { createPinia, setActivePinia } from 'pinia';
import { useAuthStore } from '@/stores/auth';
import { client } from '@/boot/colyseus';

beforeEach(() => {
  setActivePinia(createPinia());
});

it('sets error when login fails', async () => {
  vi.spyOn(client.auth, 'signInWithEmailAndPassword').mockRejectedValueOnce(
    new Error('bad credentials'),
  );

  const auth = useAuthStore();
  await expect(auth.login('a@b.c', 'secret')).rejects.toThrow();
  expect(auth.error).toBeTruthy();
  expect(auth.loading).toBe(false);
});
```

(Adapt spy method names to the real `client.auth` / store implementation.)

## Checklist

- Page tests: store method mocks are separate variables outside `vi.mock`.
- Store tests: real Pinia + Colyseus spies; no live network.
- `pinia` is not randomly mocked for page tests when a `use*Store` mock is enough.

## Anti-Patterns

```typescript
// Store mocks buried only inside vi.mock with no outer handles
vi.mock('@/stores/auth', () => ({
  useAuthStore: vi.fn(() => ({ login: vi.fn() })),
}));

// Hitting a real Colyseus URL from a unit test
await useGameStore().createGame({ maxSeats: 2 });
```
