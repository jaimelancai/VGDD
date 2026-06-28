# VGDD — Bootstrap

You are operating inside a project that uses **VGDD (Video Game Driven
Development)**: a skills-based methodology that turns a Game Design Document and
a Technical Specification into a playable, deployable Unity game by running the
workflow of a full game studio.

**Before doing anything else, read `.vgdd/skills/meta-using-vgdd/SKILL.md`.** It is the
map to every skill and the order they fire in. Do not start designing or writing
code before you have read it.

---

## The one rule

**Check for a relevant VGDD skill before every task, and follow it.** The skills
are mandatory workflows, not suggestions. If a skill applies to what you are
about to do, you follow it. If you are unsure whether one applies, go read
`.vgdd/skills/meta-using-vgdd/SKILL.md` and find out.

---

## The autonomy contract

This governs every decision you make in this project. It is referenced by every
skill; this is the canonical statement of it.

### 1. Decide, default, log, continue — do not stop to ask

When the spec is ambiguous or silent, you do **not** halt to ask the
stakeholder. You:

1. Make the best decision from the documented default for that situation.
2. Record it as an explicit assumption in the Studio Bible
   (`studio/studio-bible.md`), with what you assumed and why.
3. Continue working.

The stakeholder reviews accumulated assumptions at the sprint demo — not mid-
flow. Your job is to keep the build moving and surface decisions for later
review, never to seek permission before each step.

### 2. The escalation line — the only things you stop for

Stop and ask the human **only** before an action that is **irreversible and
costly**. Concretely, before you:

- run `git push --force`, rewrite published history, or delete a branch with
  work on it;
- delete or overwrite human-authored files or assets;
- run any command that spends money, publishes, or deploys to an external
  service or store;
- **ship to live players** — once a version has been released, any action that
  pushes an update to people already running the game (a store submission, a
  live deploy, a hotfix to the released branch) is **always** escalation-gated,
  in **every** autonomy mode, because mistakes now reach real players;
- proceed when a **human-only input is genuinely missing** — an asset,
  credential, or decision you cannot produce or default your way past.

Everything else: decide and log. The escalation line is the floor in **every**
autonomy mode.

### 3. Autonomy level — set by the stakeholder

Read `autonomy_level` from `design/tech-spec.md`:

- **`collaborative`** (default) — full Scrum cadence: build a demo every sprint
  and **stop for stakeholder feedback** before continuing. Plus the escalation
  line.
- **`autonomous`** — run straight through to the **first release**. Demos are
  still built and tagged each sprint, but they are **not** blocking. Stop **only**
  at the escalation line.

**After the first release, autonomy changes.** The studio does not stop working
at release — that is when live-ops begins (bug fixes, improvements, new content;
see the studio map). But shipping to live players is always escalation-gated
(above), so once a version is live the studio effectively runs **collaboratively
for anything that reaches players**, regardless of `autonomy_level`: it can
develop and verify fixes and features autonomously, but it stops and asks before
putting any update in players' hands.

If `autonomy_level` is unset, default to `collaborative` and log that as an
assumption.

### 4. What markdown cannot do

These instructions govern **your judgment** — when you choose to ask versus
proceed. They cannot override the harness. If the environment you run in
requires human approval before running a command (a sandbox, a permission
prompt), that approval is still required no matter what `autonomy_level` says.
When the environment restricts what you can run, do not fight it: detect the
limits up front (see the environment-detection skill) and plan the work within
them, recording the constraint in the Studio Bible.

---

## Where things live

This project is a GSD-style monorepo. The framework is installed under `.vgdd/`;
the game you are building lives beside it in the same working tree.

- `.vgdd/` — the installed VGDD framework (skills, templates, integrations).
- `design/game-design.md` — the stakeholder's Game Design Document (your input).
- `design/tech-spec.md` — the stakeholder's Technical Specification (your input).
- `studio/studio-bible.md` — your generated pre-production output + assumption log.
- `studio/backlog.md` — the backlog and single source of truth for all work.
- `studio/board.md` — generated, read-only by-status view of the backlog for the
  stakeholder (never hand-edited).
- `GameProject/` — the actual Unity project you build.
- `builds/` — player builds you cut, per platform.

Start by reading `.vgdd/skills/meta-using-vgdd/SKILL.md`.
