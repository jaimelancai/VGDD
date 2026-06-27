---
# This frontmatter is read by the studio. Keep the keys; fill the values.
title: ""                 # working title of the game
genre: ""                 # e.g. "match-3 puzzle", "2D platformer", "endless runner"
elevator_pitch: ""        # one sentence: what is this game?
status: draft             # draft | ready  (set to "ready" when you want the studio to begin)
---

# Game Design Document (GDD)

> **How the studio reads this.** This is the *what* and the *why* of your game —
> the design. The technical *how* lives in `tech-spec.md`. The Game Design
> Director reads this first to set pillars, the core loop, and the
> **minimum shippable** target; the Producer turns it into a backlog.
>
> **You do not need to fill everything in.** Fields marked **[required to start]**
> are the floor — without them the studio can't begin. Everything else is
> **[fill if you can]**: leave it blank and the relevant Director will choose a
> sensible default, **record it as an assumption** in the Studio Bible, and keep
> going. You review those assumptions at the first sprint demo and correct any
> that are wrong. Nothing here is locked; it's a starting point, not a contract.
>
> Examples in *italics* throughout use a Candy-Crush-style match-3 to show what
> good looks like. Delete them and write your own.

---

## 1. Vision  **[required to start]**

**Elevator pitch.** One sentence.
*e.g. "A colourful match-3 puzzle for mobile where players swap candies to clear
objectives across hundreds of handcrafted levels."*

**Design pillars.** 2–4 short phrases that every decision should serve. If a
feature doesn't support a pillar, it's probably out of scope.
*e.g. (1) Instantly readable — anyone can play in 5 seconds. (2) Satisfying
chains — cascades feel great. (3) Bite-sized sessions — one level fits a bus
stop.*

**Player fantasy / why it's fun.** What's the core feeling?
*e.g. The little dopamine hit of setting off a big cascade you planned.*

---

## 2. Core gameplay loop  **[required to start]**

The 10–60 second cycle the player repeats. Describe it as numbered beats. This
is the single most important section — the studio builds the first playable
around exactly this loop.

*e.g.*
1. *Player sees a board of coloured candies and an objective ("clear 20 blue").*
2. *Player swaps two adjacent candies to make a line of 3+.*
3. *Matched candies clear; candies above fall to fill gaps; cascades may chain.*
4. *Objective progress and moves-left update.*
5. *Win when the objective is met; lose when moves run out.*

---

## 3. Mechanics & rules  **[fill if you can]**

The building blocks. For each, a name + one-line rule. Group them however suits
your genre. (Match-3 prompts shown; replace with yours.)

**Core actions.** *e.g. swap adjacent tiles; tap to activate a special.*

**Special pieces / boosters.** *e.g. striped candy (clears a row/column),
wrapped candy (3×3 blast), colour bomb (clears all of one colour).*

**Obstacles / blockers.** *e.g. jelly (clear by matching on top), licorice locks,
chocolate that spreads each turn.*

**Objectives / win conditions.** *e.g. score target, clear-all-jelly, collect N
ingredients, bring items to the bottom.*

**Fail conditions.** *e.g. out of moves; out of time.*

> Leave any group blank and the Game Design Director will propose a minimal set
> fitting your genre and log it for your review.

---

## 4. Progression & difficulty  **[fill if you can]**

**How the game gets harder.** *e.g. more colours, tighter move limits, new
blocker types introduced every ~10 levels.*

**Pacing / teaching.** How are new mechanics introduced? *e.g. each new blocker
debuts in an easy "tutorial" level before it's combined with others.*

**Player progression.** Anything that persists between sessions? *e.g. level map,
stars per level, unlocked boosters. (Write "none" for a pure arcade loop.)*

---

## 5. Scope  **[required to start]**

The studio needs **two numbers**: where to stop the first build, and where you're
ultimately heading.

**Minimum shippable slice.** The smallest version that's genuinely playable and
demonstrates the core loop — your first beta target. Be ruthless; smaller is
better here.
*e.g. "10 handcrafted levels, 4 candy colours, 1 blocker (jelly), 1 booster
(striped), a level-select map, win/lose screens. No meta-progression, no audio
beyond stubs."*

**Full target vision.** The north star the backlog plans toward. The studio will
not build all of this at once — it splits it across sprints.
*e.g. "100 levels, 6 colours, 5 blocker types, 4 boosters, star ratings,
animated map, sound and music, save progress."*

> If you give only one number, the Director treats it as the full target and
> derives a minimum slice from it (logged for your review).

---

## 6. Player experience & UI/UX  **[fill if you can]**

**Key screens / flows.** *e.g. splash → level map → in-level → win/lose →
back to map.*

**On-screen elements during play.** *e.g. board, moves-left counter, objective
display, score, pause button.*

**Controls / input.** *e.g. touch — tap-swap or drag-swap.*

**Feel / juice notes.** Anything that makes it satisfying. *e.g. candies wobble
on hover; cascades escalate a "combo" sound; screen shake on big clears.*

---

## 7. Economy & systems  **[fill if you can — write "none" if not applicable]**

Currencies, lives, monetization, ads, IAP, leaderboards, daily rewards. *e.g.
"5 lives, one refills every 30 min; optional rewarded ad for an extra booster.
No real-money IAP in the first beta."*

---

## 8. Art & audio direction  **[fill if you can — placeholders are fine]**

> Art and Audio are added to the studio in a later phase. Early builds use
> **placeholder/programmer art and stub audio** on purpose. This section just
> records intent so nothing clashes later.

**Visual style.** *e.g. bright, glossy, rounded candy shapes; high-contrast
colours for readability.*

**Mood / references.** *e.g. cheerful, casual; reference: Candy Crush, Toon
Blast.*

**Audio direction.** *e.g. light, bouncy SFX; gentle background loop.*

**Assets you will provide vs. expect generated/placeholder.** *e.g. "I'll supply
final candy sprites later; use coloured circles with letters until then."*

---

## 9. Open questions & deferred decisions  **[fill if you can]**

Anything you're unsure about or deliberately leaving for later. Listing it here
tells the studio *not* to silently default it — it'll surface these at the demo.
*e.g. "Undecided: timed vs. moves-limited levels — try moves first. Deferred:
monetization until after the 100-level milestone."*

---

### Minimum to press start

If you fill only these, the studio can begin: **§1 Vision**, **§2 Core loop**,
and the **minimum shippable slice + full target** in **§5 Scope**. Everything
else will be defaulted and surfaced for your review. Set `status: ready` in the
frontmatter when you want it to start.
