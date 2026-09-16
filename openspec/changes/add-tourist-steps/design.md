## Context

See proposal.md — Why. Change `add-tourist-steps` уже частично реализован (бюджеты, peek KEEP, end-turn, keep-focus). Этот design дополняет/исправляет: auto-end, соло-экономику, непроходимые дыры как цель. Технических внешних блокеров нет.

## Goals / Non-Goals

**Goals**

- Server-authoritative steps/peeks; Correct removes tile; Incorrect KEEP; removed cells **not landable** (stand ok); multi peek while peeks remain; auto-end only when no move and no (peeks∧on live `*`); solo peeks∞ / finite steps / step-loss → timeExpired; distinct client copy for timer vs steps-loss.
- Client: counters (solo ∞ peeks only); no red targets on holes; dual loss modals; skills/AGENTS sync.

**Non-Goals**

- Магазин / выкуп тайла; вопросы/ловушки; кап «до 10»; публичные чужие счётчики; смена layout/таймеров; ambient peekable; forced auto-move off hole.

## Decisions

1. **Приватность бюджетов** — `Map<sessionId, { steps, peeks, infinitePeeks?, peekedThisTurn? }>` (или `infinite` только для peeks). Публичного Seat schema нет. `client.send('budgets', …)` владельцу. В соло: peeks infinite, steps всегда число.
2. **Снятые тайлы** — sync `removedTaskKeys`; UI-дыра у всех.
3. **Преген наград** — room-private bag 28/14/6; клиенту только при peek open.
4. **Messages** — `move` / `peek` / `peekAnswer` / `endTurn` без смены имён.
5. **Не advance после move** — grant +1/+1 при становлении current в мульти (и первый grant на старте); в соло **не** применять +1/+1.
6. **Solo** — `eligible === 1`: peeks → ∞; steps carry (finite); без end-turn; 5:00 timer; **дополнительно** если `steps === 0` и нет unfinished на живом `*` → `timeExpired` (как timer) + отдельный client reason/copy. Дальше steps только с Correct.
7. **Auto-end (мульти)** — `advanceTurn` только если нет legal move **и** не (`peeks > 0` и `hasLegalPeek` на живом `*`). Наличие peeks без стояния на `*` — auto-end ok.
8. **Incorrect KEEP** — без `markTaskRemoved`; −peek; награда в Map остаётся. **Без** gate `peekedThisTurn` (лимит 1 peek/ход снят) — можно peek пока peeks > 0.
9. **Дыры** — `validateTouristMove` / `hasLegalMove` / client targets: landing на `removedTaskKeys` **reject**. Фишка уже на клетке после Correct **остаётся**. Уход с дыры на `1`/`*`/`7` разрешён. Застревание без соседа — defer (магазин later).
10. **Client анимации** — +N ~2 с; keep-focus после move; targets не включают дыры; глаз только на живом `*`.
11. **Модалки соло** — вход в соло: ∞ только просмотры, шаги конечные; конец по timer vs steps — **разные** i18n строки (оба → тот же server lock time-expired).

## Server

| Точка | Что сделать |
|-------|-------------|
| `MyRoom.ts` | auto-end condition; drop peekedThisTurn gate; solo infinite peeks only; solo step-loss → timeExpired; budgets payload |
| `touristMove.ts` | landing forbid on removed keys; hasLegalMove respects removed; stand-on-removed ok |
| `test/MyRoom.test.ts` | SC-MOVE-38/39/40/45…; SC-BOARD-11/15; solo step-loss |

## Client

| Точка | Что сделать |
|-------|-------------|
| `game` store / GamePage | targets exclude holes; solo counters ∞ peeks only; dual loss modals; solo enter modal copy |
| `i18n` | timer vs steps-loss; соло peeks∞ |

## Risks / Trade-offs

- [Stuck on hole without exit] → Mitigation: out of scope until shop; piece may wait on void.
- [Solo from lobby alone] → Mitigation: start grant +1/+1 before solo conversion; product assumes multi then leftover solo.
- [Board fragmentation by holes] → Accepted; paths via remaining `*` / start / center.

## Migration

BREAKING относительно уже задеплоенного add-tourist-steps: auto-end, solo ∞ steps, walkable holes, 1 peek/turn. Деплоить server+client вместе.

## Open questions

- (нет)

## Tasks

Чеклист — `tasks.md` (блок 7+).
