## Context

See proposal.md — Why. Change `add-tourist-steps` уже реализован по блокам 1–8; этот design дополняет **solo become-current grant**: leftover после leave/finish не должен застревать с steps=0. Технических внешних блокеров нет.

## Goals / Non-Goals

**Goals**

- Server-authoritative steps/peeks; Correct removes tile; Incorrect KEEP; removed cells **not landable** (stand ok); multi peek while peeks remain; auto-end only when no move and no (peeks∧on live `*`); solo peeks∞ / finite steps / step-loss → timeExpired; **become-current into solo → +1 step** (peek skip); distinct client copy for timer vs steps-loss.
- Client: counters (solo ∞ peeks only); no red targets on holes; dual loss modals; skills/AGENTS sync.

**Non-Goals**

- Магазин / выкуп тайла; вопросы/ловушки; кап «до 10»; публичные чужие счётчики; смена layout/таймеров; ambient peekable; forced auto-move off hole; старт партии сразу в одного (lobby alone).

## Decisions

1. **Приватность бюджетов** — `Map<sessionId, { steps, peeks, infinitePeeks?, peekedThisTurn? }>` (или `infinite` только для peeks). Публичного Seat schema нет. `client.send('budgets', …)` владельцу. В соло: peeks infinite, steps всегда число.
2. **Снятые тайлы** — sync `removedTaskKeys`; UI-дыра у всех.
3. **Преген наград** — room-private bag 28/14/6; клиенту только при peek open.
4. **Messages** — `move` / `peek` / `peekAnswer` / `endTurn` без смены имён.
5. **Grant on become-current** — `eligible ≥ 2` → **+1 step +1 peek**. `eligible === 1` и seat **только что стал** current (leave бывшего current / finish → следующий / тот же path `applyTurnGrant`) → **+1 step**, peek **не** инкрементировать (уже ∞ после `syncSoloInfiniteMode`). Пока current **не** менялся (ушёл/финишировал не-current) — grant **не** вызывать повторно; только peeks∞ + 5:00. Пока соло и current тот же — после move **без** новых grant (end-turn нет).
6. **Solo** — `eligible === 1`: peeks → ∞; steps finite; без end-turn; 5:00; `steps === 0` ∧ ¬unfinished на живом `*` → `timeExpired` + отдельный client copy. Дальше steps с Correct (и с разовым +1 на become-current).
7. **Auto-end (мульти)** — `advanceTurn` только если нет legal move **и** не (`peeks > 0` и `hasLegalPeek` на живом `*`). Наличие peeks без стояния на `*` — auto-end ok.
8. **Incorrect KEEP** — без `markTaskRemoved`; −peek; награда в Map остаётся. **Без** gate `peekedThisTurn` — peek пока peeks > 0 (или ∞).
9. **Дыры** — landing на `removedTaskKeys` **reject**; stand ok; уход с дыры разрешён. Магазин later.
10. **Client анимации** — +N ~2 с; keep-focus; targets без дыр; глаз только на живом `*`.
11. **Модалки соло** — вход: ∞ peeks / finite steps; timer vs steps-loss — разные i18n.

## Server

| Точка | Что сделать |
|-------|-------------|
| `MyRoom.ts` `applyTurnGrant` | multi: +1/+1; solo become-current: +1 step only (после `syncSoloInfiniteMode`); не резать grant целиком при `eligible === 1` |
| `test/MyRoom.test.ts` | SC-MOVE-50 leave/finish → solo +1 step; уточнить SC-MOVE-40 (already-current carry) |
| skills `work-with-game` / messages | документировать solo become-current +1 step |

## Client

| Точка | Что сделать |
|-------|-------------|
| (обычно без UX-правок) | +N на +1 step уже есть; peeks∞ без лишнего +1 peek |
| skills при необходимости | одна строка про solo grant |

## Risks / Trade-offs

- [Stuck on hole without exit] → Mitigation: out of scope until shop.
- [Lobby alone] → Out of scope (партия не стартует в одного).
- [Board fragmentation by holes] → Accepted.

## Migration

BREAKING относительно предыдущего деплоя add-tourist-steps (solo grant). Деплоить server (+ skills); client optional.

## Open questions

- (нет)

## Tasks

Чеклист — `tasks.md` (блок 9).
