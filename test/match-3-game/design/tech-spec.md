---
# Match-3 Puzzle Test Game — Technical Specification
# Only the five keys below are read mechanically; the rest is prose the
# Technical Director interprets. Defaults shown are the studio's; changed here
# where the design calls for it.
status: ready
autonomy_level: collaborative   # demo every sprint, stop for feedback — good for a first test
tool_provisioning: 0            # detect-and-guide; never auto-installs the engine
vcs: git-local                  # local repo, no remote required
tracker: local                  # backlog lives in studio/backlog.md
---

# Technical Specification

> Companion to `game-design.md`. This is the *how*; the design is the *what*.

## The five header settings

- **`status: ready`** — begin.
- **`autonomy_level: collaborative`** — for this first test, run a demo each sprint
  and stop for feedback, so we can watch the loop and steer. (Switchable to
  `autonomous` later.)
- **`tool_provisioning: 0`** — detect-and-guide; the studio guides on any missing
  tool and never auto-installs Unity.
- **`vcs: git-local`** and **`tracker: local`** — everything local; no remote or
  external tracker needed for the test.

---

## Targets

**Platform:** **Windows PC** — a local standalone executable (Unity build target
`StandaloneWindows64`). This is the sole target for the test.

**Orientation:** **Landscape.**

**Input:** **Mouse** — select a piece, then an adjacent piece, to swap. (Use the
resolved input system; mouse-only is sufficient.)

**Distribution:** a local Windows executable for testing — **no store, no deploy**
(and any distribution would be escalation-gated regardless).

---

## Engine

**Engine:** Unity (the supported engine). Version `auto` — latest installed LTS.

> Unity must already be installed; the studio will guide setup if it's missing but
> never auto-installs it.

**Render pipeline:** `auto` (URP is fine for this 2D board game). **Input system:**
`auto` (Unity's current Input System; mouse input).

---

## Core systems (from the design — the Technical Director architects these)

The game needs these systems; the Technical Director owns how they're structured
(logic-in-testable-assemblies, thin MonoBehaviours):

- Main Menu system, Options Menu system
- Level Manager, Level Data Loader (data-driven levels)
- Board Manager, Piece Manager
- Match Detection (horizontal/vertical, matches >3, T-shape, L-shape as one group)
- Cascade + refill system; no-initial-match board generation; dead-board reshuffle
- Objective Manager, Move Counter, Popup Manager

Match/board/cascade logic is **pure testable C#** (Tier-0 unit tests) separated
from the Unity view layer — this is exactly the logic that must not live in
MonoBehaviours.

---

## Performance

- Target **60 FPS**; max board **9×9** (small, so this is comfortable).
- Instant/short response after swaps; no long inter-level loading.

---

## Verification

Leave on `auto` — the studio climbs as high as the environment allows. For this
test on Windows:
- **Tier 0** (always) — unit tests for the match/board/cascade/generation logic.
- **Tier 1** — build the Windows player + smoke test (boots to menu, a level
  loads and the core loop runs without errors).
- **Tier 2/3** — no device needed (desktop target); CI if present.

If a Unity Editor MCP is connected, it may be used for interactive QA — but build
and tests stay on the reproducible batch commands.
