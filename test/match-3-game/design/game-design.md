---
title: "Match-3 Puzzle Test Game"
genre: "match-3 puzzle"
elevator_pitch: "A small-scale match-3 puzzle for Windows PC — swap colored pieces to make matches of three or more — built as a first end-to-end test of the VGDD studio."
status: ready
---

# Game Design Document (GDD)

> Adapted to the VGDD template from the source design. The technical details
> (platform, systems, performance) live in the companion `tech-spec.md`.

---

## 1. Vision  **[required to start]**

**Elevator pitch.** A small-scale match-3 puzzle for Windows PC where the player
swaps adjacent colored pieces to make matches of three or more, across a short
run of handcrafted levels — built deliberately as a complete-but-minimal test of
the studio.

**Design pillars.** *(Proposed — not stated in the source; the studio may confirm
or adjust.)*
1. **Faithful classic match-3** — the core swap → match → cascade → refill loop
   works correctly and predictably, the way players expect from the genre.
2. **Readable and mouse-friendly** — clear pieces, large buttons, unambiguous
   board state; designed for mouse input on a landscape PC screen.
3. **Small but complete** — a short game that nonetheless has a real beginning
   (menu), middle (levels), and end (win/lose), fully playable, not a fragment.

**Player fantasy / why it's fun.** The satisfying chain reaction of a planned
swap setting off cascades, and steady progress through escalating level
objectives.

---

## 2. Core gameplay loop  **[required to start]**

1. The player sees a board of colored pieces and the level's objective + remaining
   moves.
2. The player selects a piece, then an adjacent piece, to attempt a swap.
3. The swap is allowed only if it creates at least one match of 3+ same-colored
   pieces; an invalid swap reverts automatically.
4. On a valid swap: matching pieces are removed, pieces above fall to fill gaps,
   new pieces spawn from the top, and the board re-checks for matches —
   **cascading** until stable. The swap consumes **one move**; cascades don't.
5. Objective progress and the move counter update.
6. **Win** when the objective is met (win popup → back to menu); **lose** when
   moves run out (lose popup → restart the level).

---

## 3. Mechanics & rules  **[fill if you can]**

**Core action.** Swap two adjacent pieces (horizontal or vertical). A swap is
valid only if it forms a match of 3+; otherwise it reverts.

**Pieces.** 6 distinct colors (e.g. red, blue, green, yellow, purple, orange),
each visually clear and easy to distinguish.

**Match types (must be detected).**
- Horizontal match of 3+; vertical match of 3+; matches longer than 3.
- **T-shape** and **L-shape** matches — detected and cleared as one combined
  larger match group. (For this version they do **not** create special pieces.)

**Board generation.** The starting board must contain **no existing matches**
(horizontal, vertical, T, or L) and should guarantee at least one valid move.

**Cascades.** After a clear: empty cells form, pieces fall, new pieces spawn on
top, the board re-checks, and cascades continue until stable.

**Dead-board handling.** If no valid move exists, the board reshuffles
automatically — the reshuffle must also have no initial matches and at least one
valid move.

**Blockers — Crate** *(full target; not in the minimum slice)*: occupies one
cell, can't be swapped, blocks falling pieces; damaged by an adjacent match and
destroyed after one adjacent match, freeing the cell. Placed per-level via level
data, appearing from level 6 onward.

**Objectives (per level).** One of: reach a target score; clear a target count of
a specific color; destroy a target count of Crates (or a combination).

**Fail condition.** Run out of moves before meeting the objective.

**Boosters.** **None** — explicitly out of scope (no bombs, line clears, color
bombs, shuffles, extra moves, power-ups).

---

## 4. Progression & difficulty  **[fill if you can]**

**Structure.** 10 handcrafted levels. Levels 1–5 teach core match-3 with no
blockers; levels 6–10 introduce the Crate blocker and combine objectives.

**Difficulty curve.** Board size grows (7×7 → 8×8 → 9×9), move budgets tighten,
objectives shift from pure score → color-clear → blocker-destruction →
combined.

**Level table (full target).**

| Level | Board | Moves | Objective | Blockers |
|---|---:|---:|---|---|
| 1 | 7×7 | 20 | Reach 1,000 points | None |
| 2 | 7×7 | 20 | Reach 1,500 points | None |
| 3 | 8×8 | 22 | Clear 15 red pieces | None |
| 4 | 8×8 | 22 | Clear 20 blue pieces | None |
| 5 | 9×9 | 25 | Reach 3,000 points | None |
| 6 | 9×9 | 25 | Destroy 4 Crates | 4 Crates |
| 7 | 9×9 | 24 | Destroy 6 Crates | 6 Crates |
| 8 | 9×9 | 24 | Clear 25 green + destroy 6 Crates | 6 Crates |
| 9 | 9×9 | 23 | Destroy 8 Crates | 8 Crates |
| 10 | 9×9 | 25 | Reach 5,000 points + destroy 8 Crates | 8 Crates |

**Level data.** Levels are **data-driven**, not hardcoded — each level defines
number, board width/height, moves, objective type + target, initial blocker
positions, available colors, and score target. (See the source's JSON example;
the Technical Director owns the concrete format.)

**Progression/saving.** Minimum: Play always starts at level 1. Optional: save
the latest unlocked level locally and continue from it.

---

## 5. Scope  **[required to start]**

**Minimum shippable slice (first beta target).** Levels **1–5** (no blocker),
proving the whole core loop end to end:
- Main Menu (Play, Options) → level → win/lose → menu flow;
- 6 colored pieces, boards up to 9×9, valid adjacent swaps with revert;
- no-initial-match board generation; horizontal/vertical **and** T/L match
  detection; cascades + refill; move counter;
- score and color-clear objectives; win popup (OK → menu); lose popup (Restart);
- Options screen (placeholders acceptable for audio/fullscreen).

No Crate blocker, no boosters, no saving required in the minimum slice.

**Full target vision.** All **10 levels** including the **Crate blocker**
(levels 6–10) and its objectives, the complete difficulty curve, dead-board
reshuffle, and optionally local save/continue. Still no boosters (out of scope
for this test game entirely).

---

## 6. Player experience & UI/UX  **[fill if you can]**

**Screens / flow.** Main Menu (title, Play, Options, optional Exit) → in-level →
win/lose popups → back to menu; Options → Back → menu.

**In-game HUD.** Current level number, remaining moves, current objective, score
(when the level uses it), optional pause/home button.

**Popups.** Win: "Level Complete!" / "Congratulations!" / OK → menu. Lose:
"Level Failed" / "No moves left!" / Restart → same level.

**Options.** Music volume, SFX volume, fullscreen toggle, Back — placeholders
acceptable in the first version if audio/fullscreen aren't wired yet.

**Controls / input.** Mouse; select a piece then an adjacent piece to swap.
Landscape orientation.

**Feel.** Bright, clean, readable; large clear buttons suited to mouse. Complex
animations are optional/deferred.

---

## 7. Economy & systems  **[fill if you can — write "none" if not applicable]**

**None.** No currencies, lives, monetization, ads, IAP, or leaderboards — all
explicitly out of scope for this test game.

---

## 8. Art & audio direction  **[fill if you can — placeholders are fine]**

> Placeholder-first: early builds use programmer art and stub audio.

**Visual style.** Bright, clean, casual, friendly; high-contrast, easily
distinguishable pieces. Each piece: a clear color + simple shape/icon + a selected
state (disappear animation optional). The Crate: visually distinct (wooden box,
border; cracked/destroyed animation optional).

**Audio.** Optional for this version — button click, swap, match, level
complete/failed are future/placeholder. Can be omitted or represented as Options
placeholders.

**Assets.** Use simple placeholders (colored shapes, default fonts) throughout;
don't block on final art.

---

## 9. Open questions & deferred decisions  **[fill if you can]**

- **Pillars** were derived, not authored — confirm or adjust.
- **Saving/continue** is optional; default is "always start at level 1" unless
  the studio implements local save.
- Deferred to *future* (explicitly out of scope now): boosters, special pieces
  from 4/5/T/L matches, more blocker types, a level map, audio/music, visual
  effects, mobile support, leaderboards, narrative.
