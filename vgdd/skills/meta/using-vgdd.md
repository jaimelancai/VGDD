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

- **Directors** (`.vgdd/skills/directors/`) — set strategy and guardrails at
  phase boundaries and big decisions. Invoked in pre-production and consulted
  across sprints.
- **Engineers & specialists** (`.vgdd/skills/engineering/`, `.vgdd/skills/qa/`)
  — do the per-task work, dispatched as subagents during sprints.

Plus **workflow** skills (`.vgdd/skills/workflow/`) that drive the loop itself,
and **meta** skills (`.vgdd/skills/meta/`) like this one.

---

## The studio loop (the order things happen)

### Phase A — Intake
- **`workflow/intake`** **[planned]** — Read `design/game-design.md` and
  `design/tech-spec.md`. Run environment detection. Fill spec gaps with
  documented defaults, logging each as an assumption. Never stall on a thin
  spec.
- **`workflow/environment-detection`** **[planned]** — Detect git remote, CI,
  GPU/display, connected devices, and installed tools (incl. a working Unity).
  Set the verification tier and the tool-provisioning level. Record findings in
  the Studio Bible.

### Phase B — Pre-production (Directors)
Run once at project start; revisited when the GDD changes materially.
- **`directors/technical-director`** **[planned]** — architecture, Unity setup
  (LTS version, render pipeline, input system), build pipeline.
- **`directors/game-design-director`** **[planned]** — pillars, core loop,
  progression, the "minimum shippable" definition.
- **`directors/qa-director`** **[planned]** — test strategy, quality gates,
  definition of done, which verification tier this environment supports.
- **`directors/producer`** **[planned]** — backlog, sprint cadence, tracking-
  tool sync, branching. Owns the loop.

Output: a completed `studio/studio-bible.md` (architecture + plan + assumptions)
and an initial `studio/backlog.md`.

### Phase C — Sprint loop (repeat)
Driven by **`workflow/sprint`** **[planned]**:
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

### Phase D — Release
- **`workflow/release`** **[planned]** — cut a build per target platform, run
  the release checklist, tag, produce a beta. Crossing to an external store or
  deploy hits the escalation line — stop and ask.

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

The QA Director records which tier is active so the stakeholder knows how much
was actually verified.

---

## Tool provisioning (touching the host machine)

Governed by **`workflow/tool-provisioning`** **[planned]**. Three levels:

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
- This map → `.vgdd/skills/meta/using-vgdd.md`
- How to author a new skill → `.vgdd/skills/meta/writing-skills.md` **[planned]**
- Inputs you read → `design/game-design.md`, `design/tech-spec.md`
- Your outputs → `studio/studio-bible.md`, `studio/backlog.md`, `GameProject/`,
  `builds/`

If a step's skill is **[planned]** and absent: act on the intent above, log your
decision, and keep moving.
