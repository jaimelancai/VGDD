---
name: workflow-sprint
description: "Run one VGDD development sprint end to end — sprint planning, then per-feature TDD implementation, review, verification, a playable demo, and backlog update. Use this whenever the studio is ready to build the next increment of the game: after pre-production for the first sprint, and repeatedly thereafter until the backlog goal is met. Use it for post-release work too (fixes/improvements run through the same loop)."
---

# Sprint workflow

This skill runs **one sprint**: a single iteration that turns top-priority,
refined backlog items into a **playable, verified increment** of the game, ending
in a demo. Repeat it sprint after sprint.

It is an **orchestrator**. It drives the order of events and hands off to the
skills that own each piece — it does not re-implement them:

- **Prioritization, estimation, sizing, the backlog format** → owned by
  `director-producer` (in *refinement*, which happens before this skill runs).
- **Test strategy and which verification tier applies** → owned by `director-qa`.
- **Writing game code** → owned by the `engineer-*` specialists.
- **When to stop vs. continue** → owned by the **autonomy contract** in
  `/CLAUDE.md`. This skill applies it; it does not restate the rules.

If any of those skills is not yet authored (**[planned]**), follow the intent
described here, make the best decision, log it in `studio/bible/decisions.md`, and continue.

---

## Ceremony order (do not skip steps)

Scrum separates *refining* the backlog from *running* a sprint. This skill is the
sprint. It assumes the top of the backlog has already been **refined** —
prioritized, estimated, and split small enough. It never re-prioritizes mid-sprint.

```
   (refinement happens first — see Producer)
        │
        ▼
   0. Precondition check  → is the top of the backlog "ready"?  if not, refine first
   1. Sprint planning     → pull top-priority ready items into the sprint
   2. Implement (per task)→ TDD: red → green → refactor, by the right engineer
   3. Review (per task)   → two-stage: spec-compliance, then code quality
   4. Verify              → climb the QA Director's verification ladder
   5. Demo                → build a playable demo, tag it; apply autonomy contract
   6. Close the sprint    → update backlog with feedback + discovered work
        │
        ▼  (repeat, or go to workflow-release when a release target is met)
```

---

## 0. Precondition — the backlog must be refined

Before planning, confirm the items you intend to pull are **ready** (refined,
prioritized, estimated, and small enough — no XXL items). The atomic unit is
**one feature demoable in a playtest**; nothing larger may enter a sprint
un-split.

If the top of the backlog is **not** refined (e.g. the very first sprint, or
freshly-added post-release reports), **do not prioritize it here.** Hand back to
`director-producer` to run refinement first, then return. Sprints never start on
an unrefined backlog.

**Who sets priority during refinement** (for your awareness; the Producer owns it):
- **`collaborative`** — the **stakeholder** sets priorities. If they are absent
  when refinement is needed, surface it and wait (per the autonomy contract).
- **`autonomous`** — the **Producer** derives priority from the GDD's value
  signals and task dependencies, and **logs the ordering** in `studio/bible/decisions.md`
  for later review. Build-order dependencies always win over nominal priority
  (you can't test scoring before a board exists).

---

## 1. Sprint planning

1. Read `autonomy_level` and the current state from the Studio Bible and backlog.
2. **First sprint?** The Technical Director has already **scaffolded** a minimal
   running engine project + green test harness during pre-production, at the
   resolved project path (repo root or `GameProject/` — see `director-technical`).
   Do not recreate it. Confirm it exists and Tier-0 tests run green; if it's
   somehow missing, hand back to the Technical Director to scaffold before
   proceeding. The first sprint then builds the **smallest playable fragment of the
   core loop** *on* that scaffold.
3. **Pull** the highest-priority *ready* items from the top of the backlog into
   the sprint, up to a sensible capacity. Respect the one-demoable-feature floor:
   a sprint must produce at least one feature you can show in a playtest. For each
   story pulled, call the Producer's **set-status** to move it
   `ready → in-sprint` (this updates `backlog.md`, regenerates `board.md`, and
   pushes to any tracker in one step — never edit those files directly).
4. **Set the sprint goal** in one sentence ("a swap-and-match board that clears
   3-in-a-row and updates score") and record it + the committed items in the
   Studio Bible. In `collaborative` mode, this goal is what the stakeholder will
   judge at the demo.
5. A sprint is a **time box, not a git branch** (GitFlow has no sprint branch).
   Each committed **story** gets a `feature/<story-id>` branch off `develop` when
   work on it begins (per the Producer's GitFlow scheme).

---

## 2. Implement — one task at a time, tests first (mandatory)

For each committed **story**, work proceeds on its `feature/<story-id>` branch.
Work the story's **tasks in the dependency order the Producer set during
refinement** (top-to-bottom; you don't re-decide the order), applying the right
specialist skill to each (`engineer-gameplay`, `engineer-ui`, `engineer-backend`,
`engineer-rendering`, `engineer-multiplayer`, `engineer-networking`,
`engineer-tools-build`, `tech-art` for art-asset tasks). Each task is started only once its prerequisites exist.

> **Execution mechanism (harness-dependent).** In **Claude Code**, dispatch each
> task to a **subagent** — a fresh worker with its own context — so a large sprint
> doesn't exhaust one context window and tasks stay isolated. On a harness without
> subagents, work the tasks sequentially in context instead. Either way, the
> *work* is the same; the specialist skill describes it, this dispatch is just how
> the worker is spun up.

Tasks are **commits on the story's feature branch** (each TDD'd), not separate
branches — the feature branch lands as one reviewable unit.

**Test-Driven Development is mandatory.** Every task follows red → green →
refactor:

1. **Red** — write the test(s) for the behavior *first*, and watch them fail.
   No production code before a failing test exists.
2. **Green** — write the minimum code to make the test(s) pass.
3. **Refactor** — clean it up with tests staying green.

The QA Director sets *which kinds* of tests are required and the active
verification tier, but **tests-first is not negotiable** and is not deferrable to
"later in the sprint." If you find yourself writing production code without a
failing test, stop and write the test.

> Rationalizations to reject: "this is too simple to test", "I'll add tests after
> it works", "it's just a prototype". Write the test first. A feature with no
> test is not done.

Commit each finished task to the story's `feature/<story-id>` branch. When the
whole story is done, land the feature branch as one reviewable unit (a PR into
`develop` when a remote is configured; a local merge into `develop` otherwise).

---

## 3. Review — two stages per task

When a story's implementation is complete and ready for review, call **set-status**
to move it `in-sprint → in-review`. Then, before a task is considered complete:

1. **Spec-compliance review** — owned by the **Game Design Director** (see
   `director-game-design`): does it do what the story/GDD asked and serve the
   design pillars? Does it match the sprint goal?
2. **Code-quality review** — owned by the **Technical Director** (see
   `director-technical`): does the diff follow the conventions, fit the
   architecture, keep logic/engine separation, and avoid needless complexity and
   debt? Are the tests meaningful, not hollow?

(Behavioral/test verification — test-plan cases + the QA pass — is the QA
Director's, handled in step 4 Verify, not here.)

A task that fails either stage goes back to implementation, not forward.

---

## 4. Verify — climb the ladder as far as the environment allows

Verification has two layers, both required (see `director-qa`):

1. **Automated** — run the verification the QA Director defined, climbing the
   ladder to the highest tier the environment supports. The story's mapped
   **test-plan cases** (`studio/test-plan.md`) must pass at the active tier.

   - **Tier 0** unit/integration (always) — must be green to proceed.
   - **Tier 1** build + smoke (with a display) — the boot-and-runs check via
     `qa-smoke-test`.
   - **Tier 2** device/simulator playtest (with a connected device).
   - **Tier 3** CI escalation (if CI is present).

   If an **engine Editor MCP** is connected, use it for interactive checks (enter
   Play Mode, inspect state, drive the feature) to enrich the tier — but gating
   still rests on the batch/build tiers, which are reproducible.

2. **The QA Director's QA pass** — beyond the engineers' TDD tests, the QA
   Director runs an exploratory + playtest pass on the increment. A blocking
   defect there sends the story back, not forward.

Record which tier actually ran. A sprint increment that only passed Tier 0 is
demoable, but say so plainly — never imply more verification than happened.

---

## 5. Demo — build it, show it, then apply the autonomy contract

1. Build a **playable demo** of the increment and tag it (e.g. `demo/sprint-<n>`).
   The demo is built in **every** mode — it is the proof the sprint produced
   something real.
2. Then apply the autonomy contract (`/CLAUDE.md`):
   - **`collaborative`** — **stop** and present the demo for stakeholder
     feedback. Do not start the next sprint until you have it.
   - **`autonomous`** — record the demo and **continue**; it is not a blocking
     gate. (But remember: if this increment would *ship to live players*, that is
     always escalation-gated regardless of mode.)

---

## 6. Close the sprint

1. For each completed story: merge its `feature/*` branch into `develop`
   (GitFlow: there is no sprint branch to merge — the sprint was a time box), then
   call **set-status** to move the story `in-review → done`. A story is `done`
   only once it passed both review stages, met its verification tier, and merged.
2. Turn stakeholder feedback and anything discovered during the sprint into
   **new backlog items** (they will be refined before a future sprint — not
   slipped into this one).
3. Update the Studio Bible (the relevant docs + `decisions.md`): what shipped, the verification tier reached, open
   assumptions, and the next likely goal.
4. Decide what's next:
   - backlog goal not yet met → run another sprint (back to step 0);
   - a release target is met → go to `workflow-release`;
   - post-release and the backlog is drained → idle until new reports/requests
     arrive, then refine and resume.

> **This is a safe checkpoint to clear context.** At sprint close everything
> durable is on disk — stories committed, statuses set, the Bible updated, the demo
> tagged. If the session is getting large, this is the point to clear and start a
> fresh session; `workflow-resume` will read `studio/` + git and continue from
> here. The **sprint boundary is the recommended clear-point**; a **completed
> story** within a large sprint is also safe. Never clear mid-task (uncommitted
> work) — finish and commit first. (See `workflow-resume`.)

---

## Definition of done (for the sprint)

A sprint is done only when **all** hold:
- at least one feature is **playable in the demo build**;
- every committed task passed both review stages;
- each story's **test-plan cases pass** at the active tier and the QA Director's
  **QA pass** found no blocking defect (see `director-qa`);
- Tier 0 verification is green, and the highest available tier was attempted;
- the demo is built and tagged;
- in `collaborative` mode, the stakeholder has seen the demo;
- every story's status was moved via **set-status** at each transition
  (`ready → in-sprint → in-review → done`), so `backlog.md`, `board.md`, and any
  tracker reflect reality — no story left showing a stale status;
- the Studio Bible is updated (decisions logged; any deepened docs current).

Code written but not demoable-and-verified does **not** count as a finished
sprint. Don't declare victory at "it compiles."
