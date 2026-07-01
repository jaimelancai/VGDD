---
name: workflow-intake
description: "The studio's front door — the first skill that runs on a VGDD project. Sets up the project (git, studio files, copying the design templates if missing), reads the game design and tech spec, runs environment detection, then sequences the Directors through pre-production so the studio is ready to sprint. Use at the very start of a project, before any sprint."
---

# Intake

This is the **front door**: the first thing that happens on a VGDD project. Your
job is to get from "a person has (or hasn't yet) written their two documents" to
"the studio is set up, the environment is known, pre-production is done, and the
first sprint can start." You **orchestrate** — you set things up and call the
right skills in the right order; the Directors do the actual pre-production work.

Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## 0. Cold-start setup

Make the project ready to work in, handling the "just cloned it" case:

1. **Git** — if there's no git repo, initialize one (`git init`). No remote is
   required (`vcs: git-local` is the default). If `git` itself is missing, route
   to `workflow-tool-provisioning`.
2. **The input documents** — check for `design/game-design.md` and
   `design/tech-spec.md`. If either is missing, **copy the blank template** from
   `.vgdd/templates/` into `design/`, tell the person to fill it in and set
   `status: ready`, and stop here. Do not invent a game design for them — the GDD
   is the one thing the studio genuinely needs from a human.
3. **Studio scaffolding** — ensure `studio/` and `studio/bible/` **directories**
   exist. Do **not** pre-copy the Bible template files into `studio/bible/` — the
   `.vgdd/templates/studio-bible/` files are **reference for structure only**. Each
   owning skill *creates* its own Bible document when it first writes
   (`architecture.md` by the Technical Director, `environment.md` by
   environment-detection, etc.). Pre-seeding them means the owning skills hit a
   "must read the existing file before overwriting" collision. Leave the directory
   empty; let owners create. (Likewise `studio/backlog.md`, `board.md`,
   `test-plan.md` are created by their owners, not pre-seeded.)

## 1. Check readiness

Read the frontmatter of both documents. Proceed only when **`status: ready`** on
the game design (the tech spec may stay on defaults). If the GDD is still `draft`,
tell the person what's needed and stop — don't start on an unfinished design.

> The GDD's required-to-start sections (Vision, Core loop, Scope) are the floor.
> If `status: ready` but a required section is empty, ask for it; everything else
> the Directors will default and log.

## 2. Read the inputs

- **`design/game-design.md`** — the *what* (design intent).
- **`design/tech-spec.md`** — the *how*: the five frontmatter keys exactly, the
  prose as intent.

Don't resolve gaps yourself — that's each Director's job for their own area. You
just make sure both documents are read and available.

## 3. Detect the environment

Run **`workflow-environment-detection`** before pre-production — the Directors
depend on its report (`studio/bible/environment.md`): the resolved engine/version,
the verification-tier ceiling, devices, CI, Editor MCP, provisioning context. If a
required engine or tool is missing, it routes to `workflow-tool-provisioning`
(engine missing = a hard blocker: guide the human, then wait).

## 4. Sequence pre-production (the Directors)

Run the Directors in dependency order — each builds on the last:

1. **`director-game-design`** — locks pillars + the precise core loop, resolves
   design gaps (logged), and defines the **minimum-shippable target**. *What the
   game is* comes first; everything else builds on it.
2. **`director-technical`** (+ engine overlay) — resolves the stack, writes the
   architecture + conventions, defines integration workflows, and **scaffolds**
   every required system with green Tier-0 harnesses. *How it's built*, on the
   design.
3. **`director-qa`** — writes the **thin test-plan skeleton** from the design, and
   sets the active verification tier from the environment report.
4. **`director-producer`** — decomposes the design + minimum-shippable target into
   the **initial backlog** (Epics → Stories → Tasks), refines and marks the first
   stories `ready`, and sets up the GitFlow branches.

(Order rationale: you can't architect without a design, can't scaffold without the
stack, can't write test cases without knowing the features, and can't build a
backlog until the design and architecture exist to decompose.)

## 5. Hand off to the sprint loop

When pre-production is done — design and architecture written, environment known,
a scaffolded project with green Tier-0 tests, a thin test plan, and a refined
backlog with `ready` stories — hand off to **`workflow-sprint`** to build the
first increment. From here the studio loops sprint by sprint until a release
target is met.

## Definition of done (for intake)

Intake is done when: the project is set up (git, `studio/`, `studio/bible/`); both
input documents exist and the GDD is `ready`; the environment report is written;
all four Directors have completed pre-production (design, architecture + scaffold,
test skeleton, refined backlog); and the first sprint can start without anyone
asking "is there a design, a project to build in, or anything in the backlog?"
If any of that is missing, intake isn't done — except the legitimate stop points:
waiting on the human to fill in the GDD, or on a missing engine/tool.
