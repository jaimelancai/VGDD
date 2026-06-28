---
name: meta-using-vgdd
description: "The map of the VGDD game-studio methodology — every skill, when it fires, and the studio loop order. Use this skill at the very start of ANY game-development task in a VGDD project, and whenever you are unsure which VGDD skill applies, before designing or writing any game code. Read it first."
---

# Using VGDD

**This is the map.** It tells you what skills exist, when each fires, and the
order the studio works in. Read it fully before designing or coding. Every rule
here defers to the **autonomy contract** in `/CLAUDE.md`.

> Authoring status: VGDD is being built in phases. Skills marked **[planned]**
> are not written yet — if you reach a step whose skill is missing, follow the
> intent described here, make the best decision, log it in the Studio Bible, and
> continue (per the autonomy contract). Do not stop.

---

## How skills work

A skill is a markdown file describing a job: when it triggers, what it owns, its
checklist, and its definition of done. Before any task, you check whether a
skill applies, and if so you follow it. Skills are mandatory workflows.

Two layers of skills:

- **Directors** (`director-*` skills) — set strategy and guardrails at
  phase boundaries and big decisions. Invoked in pre-production and consulted
  across sprints.
- **Engineers & specialists** (`engineer-*` and `qa-*` skills) — do the
  per-task work, dispatched as subagents during sprints.

Plus **workflow** skills (`workflow-*` skills) that drive the loop itself,
and **meta** skills (`meta-*` skills) like this one.

---

## The studio loop (the order things happen)

### Phase A — Intake
- **`workflow-intake`** **[planned]** — Read `design/game-design.md` and
  `design/tech-spec.md`. Run environment detection. Fill spec gaps with
  documented defaults, logging each as an assumption. Never stall on a thin
  spec.
- **`workflow-environment-detection`** **[planned]** — Detect git remote, CI,
  GPU/display, connected devices, a working Unity, and any connected **engine
  Editor MCP** (e.g. a Unity Editor MCP server). Set the verification tier, note
  whether an Editor MCP is in the loop, and set the tool-provisioning level.
  Record findings in the Studio Bible.

### Phase B — Pre-production (Directors)
Run once at project start; revisited when the GDD changes materially.
- **`director-technical`** **[planned]** — architecture, Unity setup
  (LTS version, render pipeline, input system), build pipeline.
- **`director-game-design`** **[planned]** — pillars, core loop,
  progression, the "minimum shippable" definition.
- **`director-qa`** **[planned]** — test strategy, quality gates,
  definition of done, which verification tier this environment supports.
- **`director-producer`** (authored) — backlog, sprint cadence, tracking-
  tool sync, branching. Owns the loop.

Output: a completed `studio/studio-bible.md` (architecture + plan + assumptions)
and an initial `studio/backlog.md`.

### Phase C — Sprint loop (repeat)
Driven by **`workflow-sprint`** (authored):
1. **Plan** — Producer pulls items, splits anything larger than medium. Atomic
   unit = one feature demoable in a playtest. No XXL tasks in a sprint.
2. **Implement** — dispatch a subagent per task to the right specialist skill
   (gameplay, UI, backend, rendering, multiplayer, networking, tools/build).
3. **Review** — two-stage: spec-compliance, then code quality.
4. **Test** — climb the verification ladder as far as the environment allows
   (see below).
5. **Demo** — build a playable demo, tag it. In `collaborative` mode, stop for
   stakeholder feedback; in `autonomous` mode, continue. (Autonomy contract.)
6. **Update backlog** — feedback and discovered work become backlog items.

### Phase D — Release (a recurring gate, not the end)
- **`workflow-release`** **[planned]** — cut a build per target platform, run
  the release checklist, tag, produce a release (the first one is the beta).
  Crossing to an external store or deploy — or any update that reaches players
  already running the game — hits the escalation line: stop and ask, in every
  autonomy mode. Release is a gate the game passes through repeatedly over its
  life, not a one-way door.

### Phase E — Live-ops & maintenance (after the first release)
**[planned — framework Phase 7; not built yet]**
The studio does not stop at release; that is when live operations begin. The
same sprint loop (Phase C) keeps running, but its **intake changes**: instead of
the original GDD, work is now fed by incoming **bug reports, crash logs, player
feedback, and improvement requests** — triaged into properly-sized backlog
tickets. The tracker (e.g. GitHub Issues) becomes an *input source*, not just an
output board.

Two things differ from pre-release work:
- **Regression safety is first-class.** You are changing a game real people are
  running. Protect existing behavior and save-game compatibility; the QA
  Director's regression checks gate changes, not just new-feature tests.
- **Shipping is always gated.** Developing and verifying a fix can be
  autonomous, but putting any update in players' hands is escalation-gated in
  every mode (autonomy contract).

Until this phase is built, if you reach post-release work: follow this intent —
triage the incoming report into the backlog, run it through the normal sprint
loop, and treat shipping the update as an escalation. Log decisions; do not stop
the studio at "first beta."

---

## The verification ladder (how much you trust a build)

Climb as high as the detected environment allows; the floor is always available.

- **Tier 0 — Unit/Integration** (always): Unity EditMode + PlayMode tests via
  `Unity -runTests -batchmode` (use `xvfb-run` if headless/no GPU). Covers
  frontend and backend logic.
- **Tier 1 — Build + smoke** (needs a display): build a player, launch it, run a
  smoke test confirming the core loop runs.
- **Tier 2 — Device playtest** (needs a connected device/simulator): deploy and
  playtest on an Android device or OS simulator.
- **Tier 3 — CI escalation** (needs CI): push tiers 0–2 onto GitHub Actions or
  Jenkins for gated, repeatable runs.

**Editor MCP (engine-in-the-loop) — a cross-tier capability, when available.**
If an engine Editor MCP is connected (e.g. a Unity Editor MCP server) and a live
Editor is running, you can drive the Editor directly: enter Play Mode, inspect
scene / GameObject state, set up test conditions, trigger gameplay actions, and
read results back. This *enriches* Tiers 0–2 with interactive, human-QA-like
verification — record it under the active tier. It is **not** the floor (it
needs a running Editor plus the MCP server, so it is unavailable on bare CI) and
**not** the sole gate for changes (an interactive session is less deterministic
than batch runs — keep the batch/build tiers as the gating mechanism). Detect
its presence in environment-detection and note it in the Studio Bible. This is
the same MCP mechanism used for tracking, just pointed at the engine.

The QA Director records which tier is active — and whether an Editor MCP is in
the loop — so the stakeholder knows how much was actually verified.

---

## Tool provisioning (touching the host machine)

Governed by **`workflow-tool-provisioning`** **[planned]**. Three levels:

- **Level 0 — detect & guide** (default): if a tool is missing, give the human
  the exact install command; never install it yourself.
- **Level 1 — install w/ confirmation** (opt-in): may install allowlisted
  lightweight tools (Git, Node, CLI utils), announcing the command first.
- **Level 2 — auto-install** (CI/container only): install allowlisted tools
  without per-item confirmation.

**Unity is never auto-installed at any level** — it is large, license-gated, and
Hub-managed. Guide the human to install it. Every install is logged in the
Studio Bible.

---

## Quick reference: where to look

- Autonomy rules → `/CLAUDE.md`
- This map → `.vgdd/skills/meta-using-vgdd/SKILL.md`
- How to author a new skill → `.vgdd/skills/meta-writing-skills/SKILL.md` **[planned]**
- Inputs you read → `design/game-design.md`, `design/tech-spec.md`
- Your outputs → `studio/studio-bible.md`, `studio/backlog.md`, `GameProject/`,
  `builds/`

If a step's skill is **[planned]** and absent: act on the intent above, log your
decision, and keep moving.
