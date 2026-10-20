## Context

See `proposal.md` — Why. Shared peek UI lives inline on the game page; slot chrome is shared with task compose via global peek-slot styles. Pack task-set cards already define difficulty dots (6px, gap 2, green/amber/red) and `--pack-card-*` surface tokens / action height 30px. Prepare-mock extract (light+dark mock) plus explore decisions override mock’s 4 orange dots, landscape labeled slots, and neutral outline CTA.

**Пакет:** client only. Server peek contract unchanged — UI continues to bind to existing synced `peekDifficulty` / `peekQuestion` / `peekPlacements` / `answerCards` and messages `peek` / `peekPlace` / `peekSubmit` (no new fields). Compose difficulty stays `1|2|3` on the existing content task model.

### Frozen server contract (as-is — do not extend in this change)

- **Room:** `tourist` (`MyRoom`) — peek session lifecycle already owned by server.
- **Synced schema (bind UI only):** `peekActive`, `peekSessionId`, `peekPieceId`, `peekRow`/`peekCol`, `peekTaskId`, `peekQuestion`, `peekDifficulty` (`uint8`, 1|2|3 when open), `peekPlacements` (answerCardId or `""`), `answerCards`, `flippedCells`.
- **Messages:** `peek` (open), `peekPlace` (slot fill/clear), `peekSubmit` (ordered check) — existing payloads; no new message names or fields.
- **Resolve (unchanged):** correct ordered slots → `+difficulty` private steps + remove tile; wrong / force-incorrect / leave mid-peek → keep bind + same task (SC-BOARD-43/44/47 baseline).
- **HTTP / DB (compose):** existing `/api/content/*` task `difficulty` `1|2|3` + slot `answerCardId` persist; no new columns/routes.
- **Auth / roles:** no new gates; game peek uses seated turn rules already enforced; compose stays verified editor / staff as today.

## Goals / Non-Goals

**Goals:**

- Peek + compose visual parity with pack task-set language (dots, soft badge, card AR, action height).
- Measurable Visual Spec for verify-mock.
- Keep SC-BOARD-49 / SC-PACK-263 reuse semantics.

**Non-Goals:**

- Server/schema/message changes (including any reimplementation of SC-BOARD-43/44/47 resolve rules restated in the delta for context).
- Redesigning catalog playing-card list tiles.
- Global Quasar palette rewrite.

## Decisions

1. **Dots = PackTaskSetCardTile** — exactly 3; fill count = current difficulty `N` (`i ≤ N` filled). **Resting color:** all three dots use the **single** token for level `N` (`--pack-ts-dot-N`), not a rainbow of 1/2/3 per index — same as `PackTaskSetCardTile` `data-diff` (light `#43a047` / `#f9a825` / `#e53935`; dark `#66bb6a` / `#ffca28` / `#ef5350`). Empty dots = outline in that same color family. Game: read-only (no hover). Compose: hover (or pointer preview) over the K-th dot previews filled `1…K` **all in color K**; click/tap on K commits difficulty `K` (touch without hover still works).
2. **Badge** — soft pill `Сложность: {word}` with sentence-case difficulty word (`лёгкая` / `средняя` / `сложная`) as the **MUST** copy; UPPERCASE (`ЛЁГКАЯ` / …) **MAY** match the mock but is not required. Tint follows difficulty (green/amber/red soft bg + matching fg). New/updated i18n: **word-only** keys for the badge (and compose). Do **not** reuse digit-only `game.peekDifficulty` (`Сложность: {n}`) as the sole badge copy, and do **not** reuse `content.difficulty.{n}` select labels (`1 — лёгкая`) for the badge — those keep a digit prefix.
3. **Empty slots** — dashed border only, no label text; AR = pack task-set **156∶206**; reuse existing dashed rgba borders (light/dark).
4. **Filled slots & pool** — same mini-card chrome as each other: `--pack-card-*` bg/border/shadow/fg, radius 12; AR 156∶206 scaled; pool max **4 cols × 4 visible rows**, `overflow-y: auto`. Not catalog `PackAnswerCardTile` split tiles; not Quasar outline chips as primary chrome.
5. **Submit** — Quasar **primary** `#1976d2`, full width; height `--pack-card-action-h` (30) band 28–36px (MAY scale up slightly vs pack card actions).
6. **Parity** — update compose dialog slot/pool in the same change; shared CSS where possible (`app.scss` peek/pack tokens), with carve-out so `PackTaskTile` list slots are not forced into unlabeled portrait AR.
7. **Reuse** — pool cards never get “used” chrome when placed in slots (SC-BOARD-49 / SC-PACK-263).

**Alternatives rejected:** mock’s 4 orange dots; landscape slots with «СЛОТ N»; neutral outline CTA; keeping flat chips for pool.

## Risks / Trade-offs

- [Tall mini-cards in 4-col grid] → modal height grows → Mitigation: scroll only the pool region; constrain cell width from content width.
- [Compose difficulty UX change vs q-select] → Mitigation: Vitest SC-PACK-276; keyboard: click/tap still works without hover.
- [SC-PACK-260 carve-out text said “chips”] → Mitigation: delta specs redefine pool as mini-cards; list tiles unchanged via **SC-PACK-238** carve-out (compose stays peek-aligned via SC-PACK-237).

## Migration Plan

Client-only UI deploy. No data migration. Rollback = revert client CSS/components.

## Open Questions

_(none blocking — resolved in explore)_

## Implementation touchpoints (client)

- Game peek dialog markup/styles (difficulty header, slots, pool grid, submit).
- Task compose dialog (dots difficulty control, slots, pool).
- Shared CSS: peek-slot empty/filled + new answer mini-card tokens (prefer `--pack-card-*`).
- i18n: difficulty word badge for game + compose.
- Vitest: board peek + task compose coverage for new SC-IDs.

Checklist: `tasks.md`.

## Visual Spec (from mock)

**Source of truth for FE:** this Visual Spec + pack-card / PackTaskSetCardTile canon below.  
**Mock frames:** prepare-mock extract (session); filenames like `Gemini_Generated_Image_3xs1nh3xs1nh3xs1-*.jpg` — **not committed** under this change dir; do not block apply on missing image files.  
**Human overrides** (MUST over literal mock pixels): 3 pack dots (not 4 orange); empty slots unlabeled + AR pack card; pool AR pack card; CTA primary + pack action height; compose interactive dots.

**Compose parity scope:** slot / pool / dots+badge chrome shares these tokens. Compose-only chrome (dialog title, question `q-input`, slot +/−, Cancel/Save) stays Quasar dense defaults — **n/a** for peek modal pixel tokens (not on peek mock).

**CSS carve-out:** new empty-unlabeled + mini-card AR MUST apply to Game peek (`.peek-slot`) and task compose slot row / pool only. **Today** `PackTaskTile` also uses the shared class `.peek-slot-like` (same base as compose in `app.scss`). Apply MUST **not** widen portrait unlabeled AR onto list tiles: either scope the new chrome with modifiers (e.g. compose-only / peek-only) or split list-tile slots onto a compact class / `pack-task-tile__slot` override that keeps labels + compact sizes. Do **not** change the shared base alone and assume PackTaskTile “inherits” the carve-out. Delta revises **SC-PACK-238** so list tiles are no longer required to match peek portrait chrome (compose remains aligned via SC-PACK-237).

### Структура (сверху вниз)

1. Header: dots (3) + badge `Сложность: …` (one row, left-aligned as a group)
2. Question (center)
3. Slots row (centered)
4. Answer pool grid (≤4×4 + scroll)
5. Submit full-width (peeker) / spectator hint

### Copy

| Элемент | Текст | Align |
|---------|-------|-------|
| Badge | `Сложность: лёгкая\|средняя\|сложная` (**MUST** sentence case); UPPER **MAY** match mock | left with dots |
| Question | task question | center |
| Empty slot | _(none)_ | — |
| Pool / filled | answer content (primary line); description secondary if present | center |
| Submit | `Ответить` | center |

### Типографика

| Роль | font-size | weight | line-height | Confidence |
|------|-----------|--------|-------------|------------|
| badge | 11–12px | 600 | 1.2 | med |
| question | 16–18px | 500–600 | 1.35 | med |
| answer body | 14–16px | 600 | 1.2 | approx |
| answer secondary | 10–11px | 400 | 1.2 | approx |
| submit | 0.72–0.85rem | 500–600 | 1.2 | high (pack action) |

### Геометрия / размеры элементов

| Элемент | W | H / min-H | Pad / gap | Notes | Confidence |
|---------|---|-----------|-----------|-------|------------|
| modal | 360–400 (≤92vw) | auto | **16–20 all sides** | radius 16–20 | med |
| header row (dots+badge) | hug | 18–22 | gap dots→badge **6–8** | vertical center | med |
| dots | ≈22 lead | 6×6 | gap **2** between dots | pack canon `--pack-ts-dot-*` | high |
| badge | hug | 18–22 | pad **2–6 × 8–10** | pill radius 999 | med |
| empty/filled/pool tile | AR **156∶206** | derived from col width | filled pad **6–8** | radius **12** | high (canon AR) |
| pool gap | — | — | **8–10** | max 4 cols | approx |
| pool viewport | 100% | ≤4 rows visible | — | overflow-y auto | decision |
| submit | 100% | **28–36** (canon `--pack-card-action-h` 30) | pad **0 0.4–0.8rem** | primary | high |

### Ритм между блоками (peek modal)

| Пара | Gap / margin | Confidence |
|------|--------------|------------|
| header → question | **10–14** | approx (was ~q-mb-sm/md) |
| question → slots | **12–16** | approx |
| slots → pool | **12–16** | approx |
| pool → submit / spectator hint | **12–16** | approx |

Whitespace only — no dividers inside the modal.

### Цвета

#### Canon (consistency — apply MUST)

| Токен | Light | Dark |
|-------|-------|------|
| `dot.1/2/3` | `#43a047` / `#f9a825` / `#e53935` | `#66bb6a` / `#ffca28` / `#ef5350` |
| `answer.bg` | `#ffffff` (`--pack-card-bg`) | `#2a2a2a` |
| `answer.fg` | `rgba(0,0,0,0.87)` | `rgba(255,255,255,0.92)` |
| `answer.muted` | `rgba(0,0,0,0.7)` | `rgba(255,255,255,0.78)` |
| `answer.border` | `rgba(0,0,0,0.14)` | `rgba(255,255,255,0.22)` |
| `answer.border.hover` | `#212121` | `#bdbdbd` |
| `answer.shadow` | `0 1px 3px rgba(0,0,0,0.08)` | `0 1px 3px rgba(0,0,0,0.35)` |
| `answer.shadow.hover` | `0 4px 12px rgba(0,0,0,0.14)` | `0 4px 14px rgba(0,0,0,0.5)` |
| `empty.slot.border` | `rgba(0,0,0,0.28)` 2px dashed | `rgba(255,255,255,0.35)` 2px dashed |
| `empty.slot.bg` | `rgba(255,255,255,0.06)` | same / transparent OK |
| `submit.bg` | primary `#1976d2` | same |
| `submit.fg` | on-primary `#ffffff` | same |
| `question.fg` | `answer.fg` | `answer.fg` |
| `badge.diff.1.bg` | `rgba(67,160,71,0.14)` | `rgba(102,187,106,0.18)` |
| `badge.diff.2.bg` | `rgba(249,168,37,0.16)` | `rgba(255,202,40,0.18)` |
| `badge.diff.3.bg` | `rgba(229,57,53,0.14)` | `rgba(239,83,80,0.18)` |
| `badge.diff.1/2/3.fg` | matching `dot.N` light | matching `dot.N` dark |

Prefer CSS vars `--pack-card-*` / `--pack-ts-dot-*` where they already equal the hex above.

#### Mock samples (reference only)

| Sample | Hex | Note |
|--------|-----|------|
| badge soft bg light | `#e7deda` | prefer difficulty-tinted soft pill |
| mock orange accent | `#ba9572` | **do not** use for dots |

### Icon ink / sizes

| Место | Size | Ink | Asset |
|-------|------|-----|-------|
| difficulty dots | 6×6 | `dot.N` (filled + empty outline same color family for current difficulty) | CSS |

### Missing icons

- none

### Hover / focus

| Target | Default → Hover | Notes |
|--------|-----------------|-------|
| pool / filled mini-card | `answer.border` + `answer.shadow` → `answer.border.hover` + `answer.shadow.hover` | **no scale** |
| empty slot (peeker) | dashed rest → cursor pointer (border/bg unchanged unless pack hover applied) | |
| dots (compose) | resting difficulty → preview fill `1…N` in color N | click commits N |
| dots (game) | read-only | no hover change |
| submit | Quasar primary hover | fixed height 28–36 |

Hover not shown on mock frame — table from pack canon + explore decisions.

### Dividers

- none inside peek modal (whitespace only; see rhythm / «Ритм между блоками» table)
