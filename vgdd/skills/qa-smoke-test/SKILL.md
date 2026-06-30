---
name: qa-smoke-test
description: "The studio's smoke-test harness — the Tier-1 'is it alive?' check that boots the game and confirms the core loop actually runs without errors, beyond Tier-0 unit logic. A helper the QA Director's strategy invokes and workflow-sprint runs at verification. Reproducible and headless (no editor clicking, no MCP). Engine-agnostic: defers engine mechanics to an overlay such as qa-smoke-test-unity. Use when a sprint or release needs to confirm a game boots and plays, not just that its logic units pass."
---

# Smoke test harness

A **smoke test** answers one question: *does the game actually boot and run the
core loop without falling over?* It is the **Tier-1** rung of the verification
ladder — above Tier-0 unit logic (does the math work?) and below the QA Director's
exploratory pass (does it feel right?). "Smoke" = turn it on and see if smoke comes
out.

This is a **helper**, not a role: the QA Director's strategy decides *when* a smoke
test is required and `workflow-sprint` runs it at the Verify step; this skill is the
*machinery* that performs it. Like everything in the Tier-0/1 gate, it is
**reproducible and headless** — no clicking in an editor, no interactive engine MCP.

**This is the engine-agnostic concept.** The mechanics of *how* to boot a build
headlessly and assert it runs are engine-specific — load your **engine overlay**
for them: Unity → **`qa-smoke-test-unity`**; Godot/Unreal → that overlay
(**[planned]**). This skill says *what* a smoke test must check and the constraints
it runs under; the overlay says *how* on the engine.

Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## What a smoke test checks (and doesn't)

**Checks** — the minimal "alive" facts:
- the game **boots** to its first scene/screen without errors;
- the **core loop runs** — the player can act, the game responds, and a win/lose
  (or equivalent terminal) state can be reached;
- **no error-level logs** appear during boot and a short run.

**Doesn't check** — leave these to the right layer:
- detailed rule correctness → Tier-0 unit tests (engineers' TDD);
- feel, balance, edge exploration → the QA Director's QA pass;
- real-device behavior → Tier-2 device playtest.

A smoke test is shallow and broad on purpose: catch "it doesn't even start" fast,
cheaply, every time.

## How it runs (the constraints any engine overlay must honor)

- **Reproducible and headless.** It runs from the same scripted/batch test command
  the Tools/Build Engineer owns — no editor interaction, no engine MCP (MCP is
  interactive/QA only). It must run identically locally and in CI, where no live
  editor exists.
- **Boot + short run + error watch.** Load the game's entry point, let it
  initialize and run the core loop briefly, and fail if any error/exception-level
  log appears.
- **Machine-readable result.** Emit pass/fail the verification ladder and CI can
  gate on.
- **Cover the scene/level set** the game needs (e.g. every level loads and runs
  clean), ideally parameterized rather than hand-written per level.

The engine overlay implements these against the engine's test runner and runtime.

## Growing from smoke to "fire" tests

When a smoke test catches a real failure, prefer turning that specific failure into
a **faster, more precise test** (a Tier-0 unit case or a targeted runtime test) so
next time it's reported sooner and more clearly — and file the case to the QA
Director's test plan. Over a project's life, smoke failures should trend down as
sharper tests take over. (Defects found still go to the Producer as backlog items;
reproducing cases follow the QA Director's reproduce-first rule.)

## Definition of done (for a smoke check)

- It runs **headlessly via the scripted/batch test command**, no editor, no MCP.
- It asserts the game **boots and the core loop runs without error logs**, and
  returns machine-readable pass/fail.
- It covers the level/scene set the game needs.
- A failure is reported clearly and, where useful, distilled into a sharper test
  and a test-plan case.
