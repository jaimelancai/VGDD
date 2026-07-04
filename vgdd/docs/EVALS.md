# VGDD Evals — testing the framework itself

*Every change to the skill files should be verifiable without a human replaying a
full game project by hand. This document specifies VGDD's own eval system —
Phase C of the roadmap. It applies VGDD's own philosophy to VGDD: a tiered
verification ladder, artifact-auditable outcomes, and findings folded back into
the skills. Informed by the first real test (the match-3: 10 sprints, a release,
a live-ops iteration, 20 findings — see the decisions/findings history).*

---

## Design principles

1. **Mirror the verification ladder.** Cheap checks run always; expensive runs
   are gated. Eval tiers E0–E3 below, like the studio's T0–T3.
2. **Judge outcomes and invariants, never transcripts.** LLM runs are
   nondeterministic: two different-but-correct paths to a green demo must both
   pass. All scoring reads the **durable artifacts** (git history, `studio/`,
   logs, builds) after the run.
3. **Methodology compliance is auditable from disk.** The shared-files
   architecture means every choreography leaves evidence: branch discipline is
   in the git graph, red-first TDD is in log timestamps, triage is in the
   backlog, decisions are in `decisions.md`. The judge is mostly a script.
4. **Perturbations over happy paths.** Nearly every real finding was a broken
   *choreography* (watcher deadlocks, methodology bypass, resume gaps, editor
   lock) — not a wrong game. The highest-value evals inject disturbances and
   check the response.
5. **Token cost is a budget.** A mini-game run consumes a meaningful fraction of
   a context window. Tier frequency reflects cost.

---

## The tiers

### E0 — Static lint (every commit, seconds, no LLM)

A script over the repo. Fails the commit on:

- frontmatter that doesn't parse; `name` ≠ folder name; description over limit;
- cross-references to skills/templates/paths that don't exist;
- **engine mechanics leaking into base skills** — greppable markers
  (`asmdef`, `-batchmode`, `-executeMethod`, `BuildPipeline`, `UnityEngine`,
  `pgrep`… maintained list) appearing outside `-unity`/engine overlays;
- `[planned]` markers pointing at skills that are in fact authored (and vice
  versa);
- templates listed in READMEs that don't exist on disk; empty template folders;
- files with stray heredoc terminators / CRLF in the skeleton (both happened).

### E1 — Mini-game battery (headless autonomous runs; on merge to the main branch)

Tiny GDDs, each sized to complete in **one sprint**, run **fully headless in
`autonomous` mode** in a container. Zero human turns is itself an assertion —
any stop-and-wait that isn't the designed release ship-gate is a failure.

**Battery composition — cover subsystems, not genres:**

| Eval game | What it exercises |
|---|---|
| Clicker / incremental | UI-heavy, near-zero logic; menu + HUD path |
| Snake or Pong | input handling + real-time game loop + lose condition |
| Memory cards | state machine, win condition, simple content data |
| **Sparse-GDD variant** (any of the above, Vision+Loop+Scope only) | the Directors' default-and-log machinery — barely exercised by complete GDDs |
| **Asset-delivery variant** (manifest + files dropped mid-run) | Producer triage of non-bug/non-feature input; tech-art pipeline; no out-of-sprint edits |

Each run: fresh container, pinned Unity (GameCI image + license activation),
pinned VGDD commit under test, `claude -p` headless with permissions bypassed
(isolated environment — the only place that mode belongs).

### E2 — Scenario probes (scripted perturbations; on merge, subset nightly)

Injected disturbances mid-run with checkable outcomes:

- **Kill/resume:** terminate the session mid-sprint → fresh session must resume
  from disk, complete the sprint, and **redo nothing already committed**.
- **Bug report:** file a scripted bug post-demo → a reproducing test must exist
  with a **red result timestamped before the fix's green** (log evidence), and
  persist as a regression case.
- **Input delivery:** drop an assets manifest / design change mid-run → it must
  appear as backlog stories *before* any related code change (git order).
- **Environment shift:** resume with a mutated `environment.md` (wrong engine
  path) → detection must re-run before any batch command.
- **Project lock:** hold a fake `Temp/UnityLockfile` → batch work must be
  deferred/reordered, not crash-looped.

### E3 — Full regression (before tagging a VGDD release)

The match-3 test, re-run end-to-end (or replayed to a defined checkpoint):
intake → pre-production → N sprints → release tag, in autonomous mode, scored
by the same auditor. The canonical baseline lives with this repo.

---

## The judge: a post-run artifact auditor

A script (plus an optional cheap LLM pass for fuzzy items) that reads the
finished project and scores a checklist. Core invariants, all mechanically
checkable — each one traces to a real finding:

**Git shape**
- no non-merge commits on `develop`/`main` during sprints (all work via
  `feature/*` merges); release/hotfix flows per GitFlow; demo/release tags
  present; merged feature branches deleted; `.meta` files tracked; no
  banned artifacts committed (results XML, logs, `Library/`).

**Studio artifacts**
- backlog and board consistent (board = pure projection); every done story
  passed both review stages (recorded); every default/assumption has a
  `decisions.md` entry; single-owner Bible docs modified only by their owners
  (inferable from commit content); test plan grew with the sprint's stories.

**Process evidence**
- TDD red-before-green per story (failing run logged/timestamped before the
  passing one — compile-error red counts for renames);
- verification tier achieved is **stated honestly** in the demo/release notes
  (claimed tier ≤ evidence found);
- demo is stakeholder-launchable (boot scene contains the increment; how-to-run
  present in the report);
- in autonomous runs: zero stop-and-waits except the ship gate; ship gate
  **held** (nothing distributed).

**Build & play**
- the player build exists and exits its boot clean;
- **autoplay check** (see below) exits 0.

### The autoplay requirement (bots, inverted)

External bots driving a built game (input injection + vision) are fragile and
expensive. Instead, **bot-playability is a requirement in every eval GDD**:

> *Acceptance criterion: the build supports `--autoplay` — plays random valid
> moves/actions until a terminal state (win or lose), then exits 0; exits
> non-zero on any unhandled error or if no terminal state is reached within N
> seconds.*

The studio then builds its own player-bot as part of the game (a loop over the
same logic layer it already unit-tests), and CI "plays" the result by launching
the player with the flag and checking the exit code. A vision-model screenshot
judge stays in reserve for visual regressions (the TextMesh/D3D11 class), run
sparingly — it is garnish, not foundation.

---

## Baselines & metrics

Each run emits a metrics JSON, diffed against the stored baseline for that eval:

```json
{
  "eval": "snake-sparse", "vgdd_commit": "…", "result": "pass",
  "sprints_to_demo": 1, "tests_green": "41/41", "build": "succeeded",
  "autoplay_exit": 0, "violations": [], "human_interventions": 0,
  "tokens_total": 210431, "wall_minutes": 38
}
```

- **Hard gates:** violations empty; interventions 0; autoplay 0; build success.
- **Soft signals (trend, not gate):** tokens, wall time, sprints-to-demo —
  regressions flagged for review, not auto-failed (nondeterminism).
- Baselines are updated deliberately (a reviewed commit), never silently.

## Frequency & budget

| Tier | Trigger | Cost |
|---|---|---|
| E0 | every commit | seconds, free |
| E1 | merge to main branch (battery: 3–5 games) | ~1 container-hour + LLM tokens per game |
| E2 | merge (one probe) / nightly (rotation) | similar to E1 per probe |
| E3 | before a VGDD release tag | hours; the big one |

## Practicalities & open items

- **Unity in CI:** GameCI images; license activation in headless env (solved,
  fiddly; document the recipe when built).
- **Claude Code headless:** `claude -p`, bypass permissions **only** in the
  disposable container.
- **Flakiness policy:** one automatic retry on E1/E2 failures; two consecutive
  failures = real. Track flake rate per eval.
- **Findings loop unchanged:** every eval failure that reveals a skill gap gets
  folded into the skills, exactly like live-test findings — the evals exist to
  keep that loop running without a human at the wheel.
- Open: authoring the battery GDDs; the auditor script; whether E1 games also
  run a second engine once Godot overlays exist (they should — same GDDs, per
  the Phase F exit criterion).
