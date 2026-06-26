# Video Game Driven Development (VGDD) — Project Plan v3

> An open-source, skills-based agentic framework that turns a **Game Design
> Document** + a **Technical Specification** into a playable, deployable Unity
> game, by giving a coding agent (Claude Code first) the skills, roles, and
> workflow of a full game studio.

North stars: [`obra/Superpowers`](https://github.com/obra/Superpowers)
(markdown skills + thin bootstrap + auto-triggering workflow + plugin
distribution) and [`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done)
(framework installed *into the user's project*, game lives beside it).

**What changed from v2:** added a **tool-provisioning policy** (§7) — how the
agent handles missing prerequisites like Git. Default is *detect-and-guide,
never silently install*; install is opt-in and allowlisted; Unity is never
auto-installed. See the changelog at the bottom.

**Locked decisions (from v1→v2):** **Unity-first**, **monorepo/GSD-style**,
**Scrum with a demo every sprint**, **local-git by default**, and a **graduated
verification ladder** instead of assuming CI.

---

## 1. Guiding principles

1. **Markdown-first, script-light.** The product *is* a library of skill and
   role `.md` files plus a thin bootstrap. Scripts/hooks exist only where
   markdown can't reach: git automation, MCP glue, Unity batch-mode invocation,
   eval running.
2. **The game is the deliverable.** Every role and ceremony exists to move a
   *functional, testable build* forward. No studio theater.
3. **Human = Stakeholder + Head of Studio.** Always able to intervene, never
   required to. But the default cadence (Scrum) *expects* a demo + feedback
   each sprint; full unattended run is an explicit mode, not the default.
4. **Engineering & QA first; Art & Audio later.** Early games use
   placeholder/programmer art and stub audio.
5. **Never stall, never overreach.** Thin specs are filled by Director-level
   defaults; missing tooling (no remote, no CI, no GPU, no device) triggers
   graceful degradation, not a halt. Missing *system tools* (e.g. Git) are
   detected and the human is guided to install them — the agent does not
   silently modify the host machine (see §7).
6. **Agile loop, feature-atomic.** The smallest unit of work is *one feature
   playable in a playtest.* No XXL tasks inside an iteration.

---

## 2. The core loop

```
   ┌─────────────────────────────────────────────────────────────┐
   │  INTAKE        Read GDD + Tech Spec. Detect environment       │
   │                (git remote? CI? GPU? device?). Fill spec gaps  │
   │                via defaults or ask the stakeholder.            │
   ├─────────────────────────────────────────────────────────────┤
   │  PRE-PRODUCTION  Directors define architecture, Unity project  │
   │                  setup, test strategy, milestone plan, backlog.│
   │                  Output: Studio Bible + Architecture Doc.       │
   ├─────────────────────────────────────────────────────────────┤
   │  SPRINT LOOP   Per sprint (Scrum):                            │
   │                 plan (split to ≤ medium tasks) →               │
   │                 implement (subagent per feature task) →        │
   │                 review (2-stage) →                             │
   │                 test (verification ladder, see §6) →           │
   │                 build a PLAYABLE DEMO →                        │
   │                 stakeholder demo + feedback → backlog update    │
   ├─────────────────────────────────────────────────────────────┤
   │  RELEASE       Cut build for target platform(s), run release   │
   │                checklist, tag, produce a beta.                 │
   └─────────────────────────────────────────────────────────────┘
```

**Exit condition (default):** first beta meeting the GDD's "minimum shippable"
criteria. The stakeholder can halt earlier, run sprint-by-sprint with demos
(default), or authorize unattended run to first beta.

---

## 3. The studio as a role/skill hierarchy

**Directors (pre-production + cross-sprint governance):**
Technical Director, Game Design Director, QA Director, Producer
(+ Art Director & Audio Director arriving in Phase 5).

**Engineers & specialists (per-task subagents):**
Gameplay, UI, Backend, Rendering, Multiplayer, Networking, Tools/Build,
QA Tester, Automation/Test, Release.

Each role is a skill file declaring: trigger conditions, what it owns, its
checklist, its definition of done, and which skills it may call. Roles are
**engine-agnostic at the top with a Unity overlay underneath**
(`gameplay-engineer` + `gameplay-engineer.unity`), so Godot/Unreal overlays can
be added later without rewriting the role logic.

---

## 4. Repository & game topology (GSD-style monorepo)

The framework is **installed into the user's project**; the generated Unity game
lives **beside it in the same working tree**.

```
my-game/                            # the user's project (git-init'd here)
├── .vgdd/                          # the installed framework
│   ├── skills/ {workflow, directors, engineering, qa, meta}
│   ├── templates/ {gdd, tech-spec, studio-bible, tickets}
│   ├── integrations/               # MCP/tracking adapters
│   ├── evals/reference-games/      # framework self-test briefs
│   └── hooks/ scripts/             # minimal automation (incl. Unity batch)
├── design/
│   ├── game-design.md              # the human's GDD
│   └── tech-spec.md                # the human's tech spec
├── studio/
│   ├── studio-bible.md             # generated pre-production output
│   └── backlog.md                  # local backlog (when no tracker connected)
├── GameProject/                    # the actual Unity project
│   ├── Assets/  Packages/  ProjectSettings/
│   └── Assets/Editor/VGDD/         # static methods Unity batch-mode calls
└── builds/                         # cut player builds per platform
```

The framework's own source repo (what gets published publicly) mirrors `.vgdd/`
plus docs, examples, and the plugin manifest.

---

## 5. Tracking, branching & review (the human's window in)

**Tracking — local-first, remote-optional (Q4).**
- **Default / zero-config:** no remote needed. The Producer skill runs
  `git init` locally and keeps `studio/backlog.md` as committed markdown that
  moves tickets backlog → in-progress → review → done.
- **Upgrade path:** if a GitHub MCP/connector is present, mirror to GitHub
  Issues/Projects. Jira, GitLab, Trello are opt-in adapters behind the same
  Producer interface.

**Branching (default):** `main` (releases) ← `develop` (integration) ←
`sprint/<n>` ← `task/<ticket-id>`. Each feature task is a reviewable diff; sprint
demos are tagged. Works identically with a remote or purely local.

**Review gates:** two-stage per task (spec-compliance, then code quality) +
a **sprint-end demo checkpoint** — the Scrum demo where the stakeholder plays
the build and feeds back. Skippable only in explicit unattended mode.

---

## 6. Verification ladder (Q5 — graceful, environment-aware)

The framework **detects what's available and climbs as high as it can**, never
failing because a higher tier is missing.

| Tier | Needs | What runs | Always on? |
|---|---|---|---|
| **0 — Unit/Integration** | Unity + batch mode (xvfb if headless, no GPU) | EditMode + PlayMode tests via `Unity -runTests -batchmode`, NUnit XML out. Frontend (UI/gameplay) and backend logic both covered. | **Yes — the floor.** |
| **1 — Build + smoke** | Graphical execution available | Build a player via `-executeMethod`, launch it, run an automated smoke test confirming the core loop runs (move / score / win-lose). | When GPU/display present |
| **2 — Device playtest** | Android device or iOS/OS simulator connected | Deploy to device/simulator; automated + human playtest. | When a device is attached |
| **3 — CI escalation** | GitHub Actions or Jenkins detected | Push tiers 0–2 onto CI for gated, repeatable runs. | When CI exists |

Two Unity-specific constraints baked into the relevant skills:
- Headless servers need `xvfb-run` in front of the Unity command (no real GPU).
- `-executeMethod` only calls **parameterless static methods**; pass arguments
  via env vars / `Environment.GetCommandLineArgs()`, not method parameters.

The QA Director records, in the Studio Bible, which tier the current environment
supports — so the human knows exactly how much was actually verified.

---

## 7. Tool-provisioning policy (host-system safety)

Installing system software (Git, Node, a JDK) is categorically different from
writing code inside a project: it's privileged, OS-specific, hard to cleanly
undo, and exactly what a Head-of-Studio stakeholder should approve rather than
discover afterward. A `tool-provisioning` skill governs this with a three-level
policy, set in the tech spec / config:

| Level | Behavior | Default for |
|---|---|---|
| **0 — Detect & guide** | Detect what's missing; stop and give the human the exact OS-specific install command. Never installs. | **Default**, and any detected local workstation |
| **1 — Install w/ confirmation** | May install from a **narrow allowlist** of lightweight, reversible dev tools (Git, Node, common CLI utils) via the platform's standard package manager — announcing the exact command first and logging it. | Opt-in (`allow_tool_install: true`) |
| **2 — Auto-install** | Installs allowlisted tools without per-item confirmation. | Ephemeral/CI/container contexts only |

**Hard rules at every level:**
- **Unity is never auto-installed.** It's large, license-gated,
  version-sensitive, and Hub-managed — always human setup, with clear guidance.
- Every install action is announced with its exact command and written to a
  provisioning log in the Studio Bible.
- The policy is **environment-aware**: permissive on disposable CI/containers,
  conservative (Level 0) on a detected personal machine — even if a higher
  level is configured, the agent surfaces what it's about to do.

This protects the framework's "clone it and run" public-repo trust story:
strangers running VGDD on their own machines should never have the host
modified by surprise.

---

## 8. Sprint mechanics & task sizing (Q3)

- **Atomic unit = one feature demoable in a playtest.** The Producer refuses XXL
  tasks inside a sprint and splits them into medium/small before work starts.
- **User-configurable scope:** the human sets how many / what kind of features
  per iteration; the default just caps maximum task size and protects the
  "one playable feature" floor.
- **Demo every sprint:** from the first playable build onward, each sprint ends
  in a demo. Stakeholder feedback is captured as backlog items and re-enters
  planning — true Scrum, not a one-shot generator.

---

## 9. Quality & evals (framework self-test)

- **Reference-game suite** in `evals/reference-games/`: tiny fully-specified
  briefs (Snake, one-screen platformer, match-3 micro-slice) the framework must
  take spec → playable Unity build. These are regression tests for *the
  framework itself.*
- **Gating:** a framework change is accepted only if the suite still builds and
  passes Tier-0 (and Tier-1 where the dev environment allows).
- **Definition of Done** is encoded per role and checked by the QA Director, not
  left to judgment.

---

## 10. Default behavior when the spec is thin

An `intake` + Director chain produces defaults so the loop never stalls:
missing platform → genre default (mobile-portrait for match-3); missing
tooling/versions → Tech Director default matrix (pinned Unity LTS, render
pipeline, input system); missing scope → Game Design Director defines a
"minimum shippable" slice and defers the rest to the backlog. **Every default
is written into the Studio Bible** for the human to see and override.

---

## 11. Phased delivery

| Phase | Goal | "Done" looks like |
|---|---|---|
| **0. Foundations** | Repo skeleton + methodology + templates + bootstrap | Clone it; agent reads bootstrap, follows the loop on a toy prompt (no real Unity build yet) |
| **1. Engineering spine (Unity)** | End-to-end on Unity, one reference game | System takes a tiny brief → playable Unity build, autonomously, Tier-0 verified |
| **2. QA & evals** | Self-testing & trustworthy | Reference-game suite + verification ladder Tiers 0–1 gate framework changes |
| **3. Tracking & collaboration** | Human-in-the-loop, Scrum demos | Local backlog flow + GitHub adapter; sprint demos & PR review; unattended mode toggle |
| **4. Depth & device** | Backend/multiplayer/networking + Tier-2 device playtest | Match-3 example produces a 100-level-capable slice, testable on an Android device/sim |
| **5. Art & Audio** | Creative support | Art/Audio Directors; placeholder→real asset pipeline; stakeholder-supplied assets integrated |
| **6. Multi-engine + distribution** | Breadth + public-ready | Godot overlay (then Unreal later); plugin marketplace packaging; docs & examples gallery |

**Recommendation unchanged:** lock Phase 0 + Phase 1 tight around *Unity + one
reference game.* Get the loop genuinely working before adding breadth.

---

## 12. Immediate next step

Phase 0 deliverables, which I can draft next for your review:
1. Repo skeleton (`.vgdd/` layout + game topology).
2. `CLAUDE.md` / `AGENTS.md` bootstrap (the "check skills before acting" spine).
3. GDD template and Tech-Spec template (with the thin-spec defaults baked in).
4. First Director skills — **Technical Director**, **Game Design Director**,
   **QA Director**, **Producer** — as readable, testable `.md` files.
5. The core-loop + sprint workflow skill.
6. The **environment-detection + tool-provisioning** skill (Level 0 detect-and-
   guide first; install levels stubbed for later), including the Unity-install
   guidance path.

Small enough to read and test before we commit to Phase 1.

---

## Decision changelog

**v2 → v3**
6. **Tool provisioning:** three-level policy — (0) detect-and-guide [default],
   (1) install allowlisted dev tools with confirmation [opt-in], (2) auto-install
   in CI/container contexts only. **Unity never auto-installed.** Every install
   announced and logged; policy is environment-aware (conservative on detected
   personal machines).

**v1 → v2**

1. **Engine:** Unity-first (medium-tier, industry standard). Godot overlay in
   Phase 6, Unreal later. Verification ladder designed around Unity batch mode.
2. **Game topology:** GSD-style — framework installed into the user's project,
   generated game lives beside it (monorepo working tree).
3. **Autonomy:** Scrum default with a demo + stakeholder feedback every sprint
   from first playable; unattended run-to-beta is an explicit opt-in mode.
   Atomic task = one playtest-demoable feature; no XXL tasks per iteration.
4. **Tracking/VCS:** local `git init` by default, no remote required; GitHub
   (then Jira/GitLab/Trello) as opt-in adapters.
5. **Build/test env:** four-tier verification ladder — unit/integration always
   (Tier 0), build+smoke with a display (1), device/simulator playtest (2), CI
   escalation to GitHub Actions/Jenkins when present (3).
