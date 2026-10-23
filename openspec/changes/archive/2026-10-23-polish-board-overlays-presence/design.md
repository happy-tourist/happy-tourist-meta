## Context

See `proposal.md` — Why. After `board-tile-skins`, task PNG (~brown) and catapult/grille PNGs share the same tone; overlays still stack above tiles (`z-index` 3 > tile 0) but were hard to read. Catapult presentation budgets are shared client↔server (`CATAPULT_ANIM_MS`, broken holds, `CATAPULT_BROKEN_PRESENTATION_MS` / D14 paced pipeline). **Baseline before this change:** say `top:-32px; right:-8px`, focus `top:32%; right:-8px`, end-turn `top:50%; right:-8px`. **Shipped:** drop-shadow overlays; catapult 1500 / broken 450+450+300; presence clock say 12 / focus 9 / end-turn 3 (D1–D3).

## Goals / Non-Goals

**Goals:**

- Readable catapult/grille via CSS `filter: drop-shadow` (board + strip chrome grille).
- Catapult timings 1000→1500 and broken ×1.5 on client and server.
- Presence clock: say 12, focus 9, skip stays ~3.

**Non-Goals:**

- New/regenerated PNG assets; opaque underlay behind grille; grille drop/rise 1000 ms; MOVE_ANIM_MS; seed/fling rules; finish/ready top-left.

## Decisions

### D1 — Contrast via `drop-shadow` (not underlay)

Apply a light/dark edge `filter: drop-shadow(...)` on `.catapult-overlay`, `.grille-overlay`, and `.my-tourist-chrome-grille` so silhouettes read on brown skins while grille bar openings stay alpha-clear (tourist visible). Rejected: solid underlay under grille (fills openings → hides tourist).

**Starter filter** (one neutral multi-shadow; tweak in apply if too weak/strong on skins):

```css
filter:
  drop-shadow(0 0 1px rgba(0, 0, 0, 0.55))
  drop-shadow(0 0 2px rgba(255, 255, 255, 0.4));
```

Analogue already in client: `.my-tourist-finish-icon` uses a thin white `drop-shadow`. Do **not** add a painted underlay `::before` / opaque bg behind grille.

**Alternatives:** redraw PNG; mix-blend — rejected (art churn / unpredictable on skins).

### D2 — Timing constants (×1.5), mirrored server

| Constant | Was | Becomes |
|----------|-----|---------|
| Successful fade `CATAPULT_ANIM_MS` | 1000 | **1500** |
| Broken hold (each) | 300 | **450** |
| Broken vanish | 200 | **300** |
| Server `CATAPULT_BROKEN_PRESENTATION_MS` | 300+300+200 | **450+450+300** |

Client: `GamePage.vue` TS constants **and** CSS — was `--catapult-anim-ms` fallback `1000ms` and `.catapult-overlay--vanish` hardcoded `200ms`; **shipped** **1500** / **300** via `--catapult-anim-ms` + `--catapult-broken-vanish-ms`. Server (room `tourist`, `MyRoom.ts` only): production `CATAPULT_ANIM_MS = 1500` and `CATAPULT_BROKEN_PRESENTATION_MS = 450 + 450 + 300` (**1200**); `resetTrapPresentationBudgets` restores those same two catapult defaults (`MOVE_ANIM_MS = 250`, `GRILLE_ANIM_MS = 1000` unchanged). `setTrapPresentationBudgetsForTests` API shape unchanged. No messages / schema / HTTP / DB.

### D3 — Presence CSS clock

| Control | Position |
|---------|----------|
| `.say-affordance` | top-center (12): `top: -40px` (raise enough to clear ready/finish TL hit; was −32 on the right), `left: 50%`, `transform: translateX(-50%)`, clear `right` |
| `.focus-affordance` | left-center (9): `top: 50%`, `left: -8px`, `transform: translateY(-50%)`, clear `right` |
| `.end-turn-affordance` | keep right-center (3): `top: 50%`, `right: -8px`, `transform: translateY(-50%)` |

**Ready/finish TL:** ready (`top/left: -4`) and say at 12 share the top edge in waiting. Literal `top: -32px` + centered 36px say **overlaps** the ready box — use **`top: -40px`** (or higher) so boxes do not intersect; prefer centers ≥36px apart when both visible.

Say-picker is already marker-centered (`left: 50%`; `bottom: calc(100% + 28px)`); keep that so the picker stays above the avatar without clip (SC-SAY-15). Hit targets: say ≥~32 CSS px (SC-SAY-15); circular say/focus/end-turn chrome ~36px and no overlap (SC-PRESENCE-33). Vitest: replace vertical-stack geometry (`focusTopPct = 32`) with clock-axis asserts (say center-x ≈ 50%, focus left edge, end-turn right).

### D4 — Skills / docs

Client `work-with-game-board` (+ `focus.md`): timings, drop-shadow note, clock positions. Server `work-with-game`: presentation budget numbers. No protocol/schema change.

### D5 — OpenSpec delta titles

Keep **historic** scenario / requirement headings from main when MODIFIED (SC-BOARD-24 title still `…300+300…`; SC-PRESENCE-30 still `…between say and end-turn`; SC-PRESENCE-33 requirement still `Say affordance sits above the focus control`). Change bodies / THEN only — renaming headings breaks `openspec validate` / archive (same rule as `redesign-pack-task-tile` / `board-tile-skins`).

## Risks / Trade-offs

| Risk | Mitigation |
|------|------------|
| Drop-shadow too weak/strong on light vs dark page | Starter multi-shadow in D1; tweak in apply if needed |
| Say at 12 overlaps finish/ready top-left | Say `top: -40px` (not −32); finish/ready stay TL; verify boxes / ≥36px centers |
| Focus at 9 vs strip on the left | Focus protrudes ~8px into the 12px HUD gap — stays clear of strip; budgets stay right |
| Server/client budget drift | Same numeric table in both; vanish CSS must match TS 300 ms; mocha override helpers |

## Migration Plan

Deploy client + server together so paced hop budget matches longer overlay. Rollback = revert both timing constants + CSS.

## Open Questions

(none — explore locked: 1500, ×1.5 broken, drop-shadow board+strip, clock 12/9/3)
