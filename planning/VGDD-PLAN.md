# Video Game Driven Development (VGDD) — Project Plan v1

> An open-source, skills-based agentic framework that turns a **Game Design
> Document** + a **Technical Specification** into a playable, deployable video
> game, by giving a coding agent (Claude Code first) the skills, roles, and
> workflow of a full game studio.

Reference north stars: [`obra/Superpowers`](https://github.com/obra/Superpowers)
(skills-as-markdown, auto-triggering workflow, plugin distribution) and
[`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done). We lean
**Superpowers-style**: markdown skills + a thin bootstrap, minimal scripting,
no heavy custom runtime.

---

## 1. Guiding principles

1. **Markdown-first, script-light.** The product *is* a library of skills and
   role definitions in `.md`. Scripts/hooks exist only where markdown cannot
   (git automation, MCP glue, eval running). This keeps it forkable, auditable,
   and harness-portable like Superpowers.
2. **The game is the deliverable.** Every layer (roles, ceremonies, tracking)
   exists to ship a *functional, testable build*. We resist building studio
   theater that doesn't move a build forward.
3. **Human = Stakeholder + Head of Studio.** Always able to intervene, never
   *required* to. A "no-collaboration" mode must run start → first beta on its
   own.
4. **Engineering & QA first, Art & Audio later.** Phase ordering is deliberate;
   early games use placeholder/programmer art and stub audio.
5. **Default-on sane behavior.** A sparse spec must still produce a sensible
   game via Director-level defaults, not a stall asking for more input.
6. **Agile loop.** info-gathering → planning → design → implementation →
   review → test → release, run as sprints with a backlog, branches, and
   reviewable artifacts.

---

## 2. The core loop (what the system actually does)

```
   ┌─────────────────────────────────────────────────────────────┐
   │  INTAKE        Read GDD + Tech Spec. Fill gaps via defaults    │
   │                or ask the stakeholder (configurable).          │
   ├─────────────────────────────────────────────────────────────┤
   │  PRE-PRODUCTION  Directors define architecture, tech stack,    │
   │  (Phase 0 of a   art/QA strategy, milestone plan, backlog.     │
   │   game)          Produce a Studio Bible + Architecture Doc.     │
   ├─────────────────────────────────────────────────────────────┤
   │  SPRINT LOOP   For each sprint:                                │
   │                 plan → implement (subagent per task) →         │
   │                 review (2-stage) → test (automated + smoke) →  │
   │                 build a playable demo → stakeholder checkpoint  │
   ├─────────────────────────────────────────────────────────────┤
   │  RELEASE       Cut a build for target platform(s), run the     │
   │                release checklist, tag, produce a beta.         │
   └─────────────────────────────────────────────────────────────┘
```

The loop continues sprint-over-sprint until the **exit condition** is met
(default: a playable first beta meeting the GDD's "minimum shippable" criteria),
or the stakeholder halts it.

---

## 3. The studio as a role/skill hierarchy

Two layers, both expressed as markdown so they auto-trigger and compose:

**Directors (pre-production + cross-sprint governance)** — set strategy and
guardrails, invoked at phase boundaries and big decisions:
- Technical Director — architecture, tech stack, engine config, build pipeline
- Game Design Director — pillars, loop, progression, "minimum shippable" def.
- QA Director — test strategy, quality gates, definition of done
- Art Director *(Phase 4)* — visual targets, placeholder-art policy early
- Audio Director *(Phase 4)* — audio targets, stub policy early
- Producer — backlog, sprint cadence, tracking-tool sync, the loop itself

**Engineers & specialists (per-task execution)** — invoked as subagents:
- Gameplay Engineer, UI Engineer, Backend Engineer, Rendering Engineer,
  Multiplayer Engineer, Networking Engineer, Tools/Build Engineer, QA Tester,
  Automation/Test Engineer, Release Engineer.

Each role is a skill file: *when it triggers, what it owns, its checklist, its
definition of done, and which other skills it may call.* Roles are
**engine-agnostic at the top, engine-specialized underneath** (e.g.
`gameplay-engineer` → `gameplay-engineer.unity` overlay).

---

## 4. Repository layout (target end-state)

```
vgdd/
├── README.md                      # what it is, quickstart, philosophy
├── CLAUDE.md / AGENTS.md          # bootstrap: "check skills before acting"
├── .claude-plugin/                # Claude Code plugin manifest (Phase 1)
├── docs/                          # methodology, role catalog, how-to
├── skills/
│   ├── workflow/                  # the loop: intake, sprint, review, release
│   ├── directors/                 # director role skills
│   ├── engineering/               # engineer role skills (+ engine overlays)
│   ├── qa/                        # test strategy, smoke tests, eval hooks
│   └── meta/                      # writing-skills, using-vgdd bootstrap
├── templates/
│   ├── game-design-doc.md         # GDD template the human fills
│   ├── tech-spec.md               # tech spec template
│   ├── studio-bible.md            # generated pre-production output
│   └── backlog/ ticket templates  # epic/story/task formats
├── integrations/                  # MCP/tracking glue (GitHub, Jira, Trello…)
├── evals/                         # test-case games + automated quality checks
│   └── reference-games/           # tiny specs the system must be able to ship
├── hooks/ scripts/                # minimal automation only
└── examples/                      # worked example (the match-3 brief)
```

---

## 5. Tracking, branching & review (the human's window in)

- **Tracking via MCP/connectors.** A `producer` skill drives an integration
  adapter so backlog → in-progress → done is mirrored to GitHub Issues/Projects
  (default), or Jira/GitLab/Trello. Adapter pattern keeps one skill, many
  back-ends. Default fallback = GitHub Issues + a Markdown backlog committed to
  the repo, so it works with zero external setup.
- **Branching strategy (default).** `main` (releases) ← `develop` (integration)
  ← `sprint/<n>` ← `task/<ticket-id>`. Each task lands as a reviewable PR;
  sprint demos are tagged. This gives the human history *and* review gates.
- **Review gates.** Two-stage review per task (spec-compliance, then code
  quality) — adapted from Superpowers — plus a sprint-end stakeholder
  checkpoint that can be skipped in autonomous mode.

---

## 6. Quality & evals (how we trust the system)

This is the part that makes it a *product* and not a demo:
- **Reference-game suite.** A set of tiny, fully-specified briefs (e.g. a
  Snake clone, a one-screen platformer, a match-3 micro-slice) that the system
  must take from spec → playable build. These are the regression tests for the
  *framework itself*.
- **Automated quality checks.** For each reference game: does it build? do unit
  tests pass? does a headless/smoke harness confirm the core loop runs (player
  can move/score/win-lose)? We gate framework changes on these.
- **Definition of Done** is encoded per role and checked by the QA Director
  skill, not left to vibes.

Engine choice matters here: **automated, headless verification of a real game
build is the single hardest technical risk.** See open questions.

---

## 7. Phased delivery (so you can review/test each step)

Each phase ends with something you can run and judge.

| Phase | Goal | Key deliverables | "Done" looks like |
|---|---|---|---|
| **0. Foundations** | Repo skeleton + methodology | Repo layout, `CLAUDE.md` bootstrap, GDD & Tech-Spec templates, the core-loop doc, contribution + skill-writing guide | A human can clone it; agent reads bootstrap and follows the loop on a toy prompt (no real build yet) |
| **1. Engineering spine (one engine)** | End-to-end on a single stack | Director skills (Tech/GameDesign/QA/Producer), core engineer skills, sprint workflow, branching, **one reference game shippable** | System takes a tiny brief → playable build on the chosen engine, autonomously |
| **2. QA & evals** | Trustworthy & self-testing | Eval harness, reference-game suite, automated quality gates, smoke-test skill | CI-style check: framework changes pass the reference-game suite |
| **3. Tracking & collaboration** | Human-in-the-loop tooling | MCP/tracking adapters (GitHub first), checkpointing, review ceremonies, autonomous vs collaborative modes | Stakeholder watches tickets flow board-to-done; can review PRs or let it run |
| **4. Multi-engine + scale** | Breadth | Second engine overlay, multiplayer/networking/backend depth, the match-3 example fully worked | The Candy-Crush-style example brief produces a 100-level-capable slice |
| **5. Art & Audio** | Creative support | Art/Audio Directors, asset-pipeline skills, asset-intake from stakeholder, generative-asset hooks | Placeholder→real asset workflow; stakeholder-supplied assets integrated |
| **6. Polish & distribution** | Public-ready | Plugin marketplace packaging, multi-harness support, docs site, examples gallery | Anyone can install and ship a small game |

**My recommendation:** lock Phase 0 + Phase 1 scope tightly around *one engine
and one reference game*. Breadth is the enemy of a working first beta.

---

## 8. Default behavior when the spec is thin

A `intake` + Director chain produces defaults so the loop never stalls:
- Missing platform → default per genre (e.g. mobile-portrait for match-3).
- Missing engine → Tech Director picks from a documented default matrix.
- Missing scope → Game Design Director defines a "minimum shippable" slice and
  defers the rest to the backlog.
- All defaults are **written into the generated Studio Bible** so the human can
  see and override them.

---

## 9. Open questions (these change the architecture — your call)

1. **First engine.** Your example says Unity. Unity is industry-standard but
   *hard to build/test headlessly in CI* and license-encumbered. A
   code-first engine (e.g. Godot, or an HTML5/JS stack like Phaser) is far
   easier to make the agent build, run, and auto-verify — which Phase 2 leans
   on heavily. **Strong recommendation: prove the whole loop on a
   code-first/web engine in Phases 1–2, then add Unity as an overlay in
   Phase 4.** Do you want to (a) hold to Unity from the start, or (b) start
   code-first and add Unity later?
2. **Where does the *generated game* live?** Same repo as the framework
   (monorepo, simplest to start) or each game scaffolded into its own repo by
   the system? This affects branching and tracking design.
3. **Autonomy default.** When run with no human input, should it stop at
   "first playable" for review, or push all the way to "first beta" unattended?
4. **Default tracking back-end.** GitHub Issues/Projects as the zero-config
   default, with Jira/GitLab/Trello as opt-in adapters — agreed?
5. **Build/test environment.** What can the agent actually run during dev — a
   CI runner, your local machine, a container? This determines how real the
   automated verification in Phase 2 can be.

---

## 10. Suggested immediate next step

Approve scope for **Phase 0** and answer Q1 (engine) + Q2 (repo topology).
With those two locked, I can draft the Phase 0 deliverables in detail: the repo
skeleton, the `CLAUDE.md` bootstrap, the GDD and Tech-Spec templates, and the
first three Director skill files — small enough for you to read and test before
we commit to Phase 1.
