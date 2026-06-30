---
name: qa-smoke-test
description: "The studio's smoke-test harness — the Tier-1 'is it alive?' check that boots the game and confirms the core loop actually runs without errors, beyond Tier-0 unit logic. A helper the QA Director's strategy invokes and workflow-sprint runs at verification. Reproducible and headless (no editor clicking, no MCP). Use when a sprint or release needs to confirm a built/running game boots and plays, not just that its logic units pass."
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

## How it runs (reproducible, headless)

The reliable, CI-runnable smoke is a **PlayMode test that boots the game in a
headless run and asserts it comes up clean** — run via the same batch test command
the Tools/Build Engineer owns (`-runTests -testPlatform PlayMode`, NUnit XML out,
`xvfb-run` on headless Linux). It runs in PlayMode on the build/CI machine; it does
**not** require driving the shipped player binary by hand.

## Unity specifics

A workable smoke test (adapt to the game):
```csharp
using System.Collections;
using NUnit.Framework;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.TestTools;

public class BootSmokeTests
{
    bool sawError;
    void OnLog(string msg, string stack, LogType type)
    { if (type == LogType.Error || type == LogType.Exception) sawError = true; }

    [UnityTest]
    public IEnumerator Game_Boots_NoErrors()
    {
        sawError = false;
        Application.logMessageReceived += OnLog;
        SceneManager.LoadScene(0);              // the boot scene (build index 0)
        yield return new WaitForSeconds(2f);    // let it initialize
        Application.logMessageReceived -= OnLog;
        Assert.IsFalse(sawError, "Errors/exceptions during boot");
    }
}
```

Key headless rules (Unity, verified):
- **Use `WaitForSeconds`, not `WaitForEndOfFrame`** — in non-graphical (`-nographics`)
  CI runs there's no rendering, so frame-based waits don't tick. Time-based waits
  are portable across headless and on-device.
- Subscribe to **`Application.logMessageReceived`** and fail on `Error`/`Exception`
  — the cheap, universal "something broke" signal.
- Keep waits short but real (a second or two) — enough to catch boot/init crashes
  without making the suite slow.
- For a many-level game, parameterize over scenes (NUnit `[TestFixtureSource]` /
  `[ValueSource]` that enumerates the build's level scenes) so each level gets a
  generic "loads and runs clean" smoke without hand-writing one per level.

## Growing from smoke to "fire" tests

When a smoke test catches a real failure, prefer turning that specific failure into
a **faster, more precise test** (a Tier-0 unit case or a targeted PlayMode test) so
next time it's reported sooner and more clearly — and file the case to the QA
Director's test plan. Over a project's life, smoke failures should trend down as
sharper tests take over. (Defects found still go to the Producer as backlog items;
reproducing cases follow the QA Director's reproduce-first rule.)

## Definition of done (for a smoke check)

- It runs **headlessly via the batch test command**, no editor, no MCP.
- It asserts the game **boots and the core loop runs without error logs**, and
  returns machine-readable pass/fail.
- It uses **time-based waits** (headless-safe), and covers the level/scene set the
  game needs.
- A failure is reported clearly and, where useful, distilled into a sharper test
  and a test-plan case.
