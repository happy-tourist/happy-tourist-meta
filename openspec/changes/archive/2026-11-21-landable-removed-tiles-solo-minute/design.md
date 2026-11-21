## Context

See `proposal.md` — Why / Scope. Explore decisions D1–D3 + soft 1–3 closed: removed task cells landable everywhere; map layout holes stay blocked; solo 60s with ∞ steps+peeks and no steps-exhaustion; modal copy agreed.

Current server: `validateTouristMove` / push / return / catapult helpers reject `removedKeys`; `SOLO_BUDGET_SECONDS = 300`; private budget `infinite` means peeks-only (steps still finite); `evaluateSoloStepsExhausted*` marks `timeExpired`. Client mirrors landable checks and solo HUD (∞ peeks only; dual end modals). Target after this change: `infinite` = both budgets unlimited in solo.

Packages: **server** + **client** (room `tourist` contract aligned).

## Goals / Non-Goals

**Goals:**

- One playability rule: removed task key ⊆ landable if `board.isPlayable` and unoccupied.
- Solo budgets: both infinite; constant `SOLO_BUDGET_SECONDS = 60`; delete exhaustion gate.
- Client hints / HUD / i18n / solo end modal match specs.

**Non-Goals:**

- Schema new fields for removed/landable (reuse `removedTaskKeys`).
- Changing multiplayer 60s / finite ±1 grants.
- Restoring tiles / shop.

## Decisions

### D1 — Stop treating `removedKeys` as non-landable in pure rules

- **Choice:** Remove the `removedKeys.has(to)` reject from `validateTouristMove`, `validateTouristPush`, return-target listing, and catapult fling candidate filters in `src/game/touristMove.ts`. Keep rejecting never-playable layout holes via `board.isPlayable`.
- **Why:** Single source of truth; D1 «всё на пустые».
- **Alt:** Client-only hints — rejected (server would still reject).

### D2 — Solo infinite steps via budget flag (not a fake large number)

- **Choice:** **Widen** existing private `budgets.infinite` so that in solo it means **both** peeks and steps are unlimited (no new `stepsInfinite` field). Spend paths (`move` / `rescue` / `push` / `return`) skip step decrement when `infinite`; **`canSpendStep` (and any steps≥1 gate) MUST treat `infinite === true` as steps available** — today HEAD only checks `b.steps > 0` and always decrements steps (peeks-only ∞). Become-current solo skips finite +1 step/+1 peek grant (HEAD today still does `steps += 1` in solo); private `budgets` still sends `{ steps, peeks, infinite, peekedThisTurn }` with `infinite: true` — client shows ∞ for **both** counters when the flag is set. Multiplayer keeps `infinite: false` and finite numeric budgets.
- **Why:** Matches «считать нечего»; avoids soft-lock at 0; reuses the existing wire field (YAGNI vs a second flag).
- **Alt:** Keep numeric steps but never expire — rejected (HUD would still show a number; product wants ∞). Alt: add `stepsInfinite` — rejected (solo always pairs both; one flag is enough).

### D3 — Solo timer 60s; delete exhaustion path

- **Choice:** `SOLO_BUDGET_SECONDS = 60` (HEAD default `300`); remove `maybeSoloStepsExhausted` and its public test hook `evaluateSoloStepsExhaustedForTests` plus all call sites; **rewrite** mocha SC-MOVE-48/97/98 (and client SC-PRESENCE-21/35) to assert **no** time-lock / no steps-exhausted copy under ∞ steps — do **not** delete the scenario IDs. Client drops `stepsExhaustedModal` / `soloEndModalKind` steps branch.
- **Why:** D3 + soft timer choice; keep stable scenario IDs in delta specs.
- **Alt:** Keep exhaustion as safety net — rejected (unreachable under ∞ steps).

### D4 — Client landable + presence

- **Choice:** Game landable helper / legal targets include removed task keys (layout never-playable holes stay out); presence shows ∞ for **both** counters when `budgets.infinite`; move/push/return/rescue affordances treat `budgetsInfinite` as “steps available” (same pattern as peeks — do not gate on numeric `steps === 0` while infinite); `soloUnlimitedModal` i18n → «Вы один. Ходы и просмотры безлимитны. У вас 1 минута.»; timer-only solo end modal (drop steps-exhausted branch / `inferSoloEndModalKind` `'steps'`).
- **HUD / affordance parity with peeks∞:** `stepsBudgetDisplay` MUST use `budgetsInfinite` → ∞ (today only peeks do); skip steps `+N` fall when infinite (same as peeks); every local gate that today uses `game.steps <= 0` — `legalTargets`, return/push/rescue affordances, and the watch that clears `returningPieceId` — MUST treat `budgetsInfinite` as steps available (wire may still carry a numeric `steps` while `infinite: true`).
- **Red ring after 60s solo:** multi and solo both use `turnBudgetSeconds === 60`, so **do not** detect solo via duration (`isSoloBudgetSeconds(300)` / store `isSoloBudget` based on 300 dies). Rewrite solo red-accent detection to **exactly one eligible seat** in synced `seats` (`finishPlace === 0` ∧ `!timeExpired`) — same condition as `game/move` solo; works for seated players and spectators (who never get private `budgets`). End-turn hide already keys off `budgetsInfinite` — keep that.
- **Why:** Specs `game/board` + `game/presence`; avoids UX lie and blue/red collision; avoids soft-lock when numeric steps are 0 under ∞.
- **Alt:** Keep hole hints off while server accepts — rejected (UX lie). Alt: keep a distinct solo `turnBudgetSeconds` (e.g. 59/61) — rejected (product wants 60; eligibility count is enough).

### D5 — Stale OpenSpec titles stay; bodies are canonical

- **Choice:** Delta `### Requirement:` / `#### Scenario [SC-…]:` **titles** keep matching main `openspec/specs/…` for OpenSpec validate (even when wording still says «not landable», «five-minute», «finite steps», «rejected», «only on peeks»). **Bodies and THEN lines** in this change are the product authority for apply.
- **Why:** Renaming MODIFIED headers breaks validate against main; scenario IDs stay stable for tests.
- **Alt:** Rename titles to match new behavior — rejected until a separate main-title cleanup change.

## Risks / Trade-offs

- **[Risk] Solo much easier with landable holes + ∞ steps in 60s** → Mitigation: accepted product trade-off from explore.
- **[Risk] Widening `infinite` from peeks-only to both budgets breaks peeks-only assumptions in tests/comments** → Mitigation: update server/client tests and comments in one pass; grep `infinite` / `budgetsInfinite` / SC-MOVE-48; client HUD uses the same flag for both ∞ counters.
- **[Risk] Fling/push suddenly gains more destinations (removed cells)** → Mitigation: intended; update unit cases that assumed hole reject.

## Migration Plan

1. Server pure rules + room budgets/timer + tests.
2. Client landable + HUD/i18n/modals + vitest.
3. No DB / schema migration; no deploy feature flag required.
4. Rollback: revert siblings; behavior returns to non-landable holes + 300s + finite solo steps.

## Technical prerequisites

None open (no new deps / providers). Explore blockers closed.

Implementation checklist: `tasks.md`.
