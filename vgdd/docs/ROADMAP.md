# VGDD Roadmap — after the first test

*Status: written after the first end-to-end test (the match-3 project): 8 sprints,
one release candidate, one real regression handled reproduce-first, 17 findings
folded back into the skills. Every skill except `tech-art`, `tech-audio`
[planned], and the external integrations has now executed at least once against a
real Unity project. This roadmap sequences what comes next. Guiding logic:
**exercise what has never run while the project is hot → make changes verifiable →
invite users → expand outward (tools, engines).** Each phase de-risks the next.*

---

## Phase A — Canonical repo & v0.2.0 (housekeeping, do first)

The framework has lived as a synced skeleton while test projects drifted. Before
anything else:

- Create the **canonical VGDD git repository** — the single source of truth the
  test projects sync *from*.
- Tag the current post-first-test state **`v0.2.0`** (v0.1 being the pre-test
  design; v0.2 is the reality-hardened framework with all 17 findings folded in).
- From here, every change to a skill lands as a commit in this repo; test
  projects update from tags, never ad-hoc copies.

**Exit:** the repo exists, tagged, and the match-3 project's `.vgdd/` matches it.

## Phase B — Finish the match-3: full target, real assets, first live-ops

Continue the live test project to its GDD full target, using it to exercise the
authored-but-never-fired machinery:

- **Real assets (was option 4).** Provide `design/assets.md` + `design/assets/`
  (piece sprites, crate art) and run an integration iteration. This is the first
  live run of **`tech-art`**: manifest reading, the inventory document,
  import/config, load-checks, and the swap-in that keeps names/paths stable so
  placeholders are replaced with zero code changes.
- **Full target content:** Crate blocker + Levels 6–10 (the P2 epic), completing
  the design's level table.
- **First live-ops cycle.** With `v0.1.0` tagged, handle at least one bug
  reported *against the release* through the full Phase-7 path: triage →
  reproduce-first failing test → `hotfix/*` → regression suite green → `v0.1.1`.
  This machinery is fully designed and has never run.
- **Audio (stretch):** if audio assets are provided in this phase, write
  **`tech-audio`** (the designed mirror of `tech-art`) against a real need — per
  the write-when-a-design-demands-it rule.

**Exit:** match-3 at full target (10 levels, Crate), real art in, at least one
hotfix released, `tech-art` validated (and `tech-audio` if audio arrived).

## Phase C — Eval harness: VGDD gets its own QA Director (was option 3)

Every skill edit since the first test is currently unverified. Standardize
testing of VGDD itself, layered like the framework's own verification ladder:

- **Smoke eval** — a deliberately tiny game (a clicker / minimal Snake): one
  sprint end-to-end, cheap in time and tokens, run on **every skill change**.
- **Full regression** — the match-3 test re-run (or replayed to a checkpoint),
  run **before tagging a VGDD release**.
- **Scenario checklists** — score the behaviors that matter, not just "it
  finished": intake routed correctly; environment/layout detected; TDD went
  red-first; the three review lenses fired; set-status kept backlog/board
  consistent; resume reconstructed without redoing work; the demo was
  stakeholder-launchable; findings count vs. baseline.
- **Two scenarios the first test never covered:**
  - a **deliberately sparse GDD** — the Directors' default-and-log machinery was
    barely exercised because the match-3 GDD was unusually complete;
  - **`autonomous` mode** — never run. Target: point VGDD at a GDD, come back to
    a tagged `0.1.0` with zero human turns. Also the flagship demo for release.
- New game types (2D/3D, casual/non-casual) join this battery over time; a 3D
  eval is what triggers the 3D sections of `tech-art-unity` (and likely
  `engineer-rendering`).

**Exit:** a documented eval suite in the repo; the smoke eval is the required
gate for merging skill changes. *(Design spec: see `docs/EVALS.md`.)*

## Phase D — Distribution & public release (was option 2)

Make VGDD installable by people who aren't its authors:

- **Installer** — npm package à la GSD (`npx vgdd init` or similar): copies
  `.vgdd/`, places/merges `CLAUDE.md`, handles updates from tagged versions.
  Evaluate the Claude Code **plugin manifest / marketplace** route in parallel.
- **Docs pass** — README, quickstart (the two-documents-in, game-out flow),
  the skills map, template guides, and a write-up of the first test (the
  17-findings story is the launch article).
- **Public repo hygiene** — license check, contribution notes, issue templates.

Gated on Phase C's smoke eval existing: once others install VGDD, every change
must be verifiable without a human manually replaying the match-3.

**Exit:** a stranger can install VGDD into a fresh Unity project and reach a
first sprint using only the docs.

## Phase E — External tool integrations (new)

Validate the designed-but-never-tested `integrations/` layer against real
services. The design holds: `studio/backlog.md` stays the source of truth; the
external tracker is a **one-way outbound projection**; human changes return via
conversation; shipping/deploy remains escalation-gated.

- **GitHub first** — a real remote (`vcs: github`): push, feature-branch PRs as
  the review surface, releases as GitHub Releases. This also gives the VGDD repo
  itself its CI.
- **CI at Tier 3** — GitHub Actions and/or **Jenkins** running the *same* batch
  build/test commands (never a second build path): EditMode+PlayMode on PRs into
  `develop`, player builds on `release/*`. Validates the Tier-3 rung of the
  verification ladder, never yet reachable.
- **Trackers** — **Trello**, then **Jira** (`tracker:` setting): the Producer's
  set-status pushes the board outward; the "anything new on the board?"
  conversational return path is exercised with real human edits on the real
  board.
- Each integration is an adapter skill/doc under `integrations/`, tested against
  the match-3 project or the smoke-eval game.

**Exit:** the match-3 project runs with a GitHub remote + CI green at Tier 3, and
at least one tracker (Trello or Jira) mirroring the backlog outward.

## Phase F — Second engine: Godot, then Unreal (was option 1)

The true test of the base/overlay architecture — deferred last because the split
already isolates it, so waiting costs little:

- **Godot first** — open-source, lightweight, genuinely CLI/headless-friendly:
  the cleanest check that the engine-agnostic bases hold. Write
  `director-technical-godot`, `engineer-*-godot`, `qa-smoke-test-godot`,
  `tech-art-godot` against a real test game (reuse an eval game's GDD), folding
  findings back exactly like the Unity first test.
- **Unreal after** — the hard mode (C++ toolchain, compile times, project
  weight); attempt only once Godot proves the overlay pattern generalizes.
- Expect a Unity-first-test-sized findings list per engine. That's the point.

**Exit:** the same GDD produces a playable game on a second engine with no
changes to any base skill (only overlays added).

---

## Deferred / opportunistic

- **`tech-audio`** — written when a design first needs audio (likely Phase B).
- **3D asset sections** in `tech-art-unity`, `engineer-rendering`,
  `engineer-backend` / `-multiplayer` / `-networking` — each written when an
  eval/test game first demands it.
- **Token tuning** — decisions-log archiving / summarization when the resume
  boot-tax (baseline: ~9%) drifts upward on long projects.
- **Editor MCP** — exercise the interactive-QA path when a Unity Editor MCP is
  actually connected in some test environment.

## Sequencing rationale (one paragraph)

Phase B runs while the match-3 is hot and validates the last unfired skills on a
real project. Phase C makes every subsequent change to VGDD verifiable — the
framework finally applies its own quality philosophy to itself. Only then does
Phase D invite users, because published changes need the eval gate. Phase E layers
real-world tooling on a now-stable core, and Phase F expands engines last because
the overlay architecture already contains that risk. At every phase, findings fold
back into the skills — the first test proved that loop is the engine of the whole
project.
