# VGDD — Video Game Driven Development

> Give your coding agent a game studio. VGDD is a skills-based methodology that
> turns a **Game Design Document** + a **Technical Specification** into a
> playable, deployable **Unity** game, by running the roles and workflow of a
> full development studio.

VGDD is **markdown-first**: the product is a library of skill files plus a thin
bootstrap that makes the agent trigger them automatically. It is modeled on
[`obra/superpowers`](https://github.com/obra/superpowers) and
[`gsd-build/get-shit-done`](https://github.com/gsd-build/get-shit-done), and
specialized end-to-end for video game development.

> **Status: early, under active construction.** The foundations (this skeleton +
> bootstrap) are in place; the Director and engineer skills are being authored
> phase by phase. See [`docs/ROADMAP.md`](docs/ROADMAP.md).

---

## How it works

You write two documents — a game design and a technical spec — and drop them in
`design/`. The agent then runs the studio loop:

1. **Intake** — reads your docs, detects the environment, fills any gaps with
   sensible defaults (logged for your review).
2. **Pre-production** — Director skills (Technical, Game Design, QA, Producer)
   define the architecture, Unity setup, test strategy, and backlog.
3. **Sprints** — each iteration plans, implements (a subagent per feature),
   reviews, tests, and produces a **playable demo**.
4. **Release** — cuts a build for your target platform(s) and tags a beta.

You are the **Stakeholder and Head of Studio**: always able to step in, never
required to. You set how autonomously it runs.

## Two ways it works for you

- **`collaborative`** (default) — a demo every sprint; it stops for your
  feedback before continuing.
- **`autonomous`** — runs straight to a first beta, stopping only for
  irreversible, costly actions.

Either way, it never silently modifies your machine and never force-pushes or
deletes your work without asking. See the autonomy contract in
[`CLAUDE.md`](CLAUDE.md).

## Requirements

- A coding agent harness (Claude Code first; others later).
- **Unity** installed (an LTS version). VGDD will **not** auto-install Unity —
  it's license-gated and Hub-managed; the agent guides you to install it.
- Git. If missing, the agent guides you to install it (or, opt-in, installs it
  for you). No remote is required — local `git init` is the default.

## Quickstart

> Installation wiring (plugin marketplace, per-harness manifests) lands in a
> later phase. For now this is the intended shape:

1. Install VGDD into your project (it lives under `.vgdd/`).
2. Copy the templates: `design/game-design.md` and `design/tech-spec.md` from
   `.vgdd/templates/`, and fill them in.
3. Start your agent. It reads the bootstrap, follows `using-vgdd`, and begins.

## What's inside

```
.vgdd/
├── skills/            # flat skill folders, grouped by name prefix:
│   ├── meta-*/        #   the skill system (meta-using-vgdd = the map)
│   ├── workflow-*/    #   the loop: intake, sprint, release, env-detection, provisioning
│   ├── director-*/    #   Technical, Game Design, QA, Producer
│   ├── engineer-*/    #   Gameplay, UI, Backend, Rendering, Multiplayer, Networking, Tools
│   └── qa-*/          #   test strategy, smoke tests, eval hooks
├── templates/         # GDD, tech-spec, studio-bible, ticket formats
├── integrations/      # MCP/tracking adapters (GitHub first; Jira/GitLab/Trello)
├── evals/             # reference games the framework must be able to ship
├── hooks/ scripts/    # minimal automation (git, Unity batch-mode, evals)
└── docs/              # methodology, role catalog, roadmap
```

Each skill is a folder with a `SKILL.md`. Claude Code discovers skills one level
deep, so grouping lives in the **name prefix**, not in nested folders — see
[`skills/README.md`](skills/README.md).

## A note on skill naming

Every skill is a folder with a `SKILL.md`, named with a **layer prefix** so the
grouping survives Claude Code's flat one-level discovery: `workflow-` skills are
action verbs (`workflow-intake`, `workflow-sprint`) because actions trigger
reliably; `director-` and `engineer-` skills keep role nouns
(`director-technical`, `engineer-gameplay`) because that's how a studio refers to
them and how the GDD/spec will reference them. The prefix mix is deliberate, not
drift — see [`skills/README.md`](skills/README.md).

## Philosophy

- **The game is the deliverable.** Every role exists to move a testable build
  forward.
- **Markdown-first, script-light.** Forkable, auditable, harness-portable.
- **Never stall, never overreach.** Default under uncertainty; ask only before
  irreversible, costly actions.
- **Engineering & QA first.** Art and audio support arrives in a later phase.

## License

MIT — see [LICENSE](LICENSE).
