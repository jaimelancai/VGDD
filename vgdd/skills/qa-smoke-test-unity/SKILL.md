---
name: qa-smoke-test-unity
description: "Unity engine overlay for the smoke-test harness. Loaded alongside qa-smoke-test when the engine is Unity. Supplies the Unity mechanics: a PlayMode test that loads the boot scene, watches Application.logMessageReceived for errors, uses headless-safe time waits, and runs via the batch test command. Use when implementing a smoke test on a Unity project."
---

# Smoke test — Unity overlay

Unity-specific "how" for the engine-agnostic `qa-smoke-test` helper. Load both on a
Unity project. The base says what a smoke test checks; this implements it with the
Unity Test Framework.

## The pattern: a PlayMode boot test

The reliable, CI-runnable Unity smoke test is a **PlayMode test that boots the game
in a headless run and asserts it comes up clean** — run via the same batch test
command the Tools/Build Engineer owns (`-runTests -testPlatform PlayMode`, NUnit
XML out, `xvfb-run` on headless Linux). It runs in PlayMode on the build/CI machine;
it does **not** require driving the shipped player binary by hand (that's a harder,
separate problem — don't reach for it for the standard Tier-1 check).

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
        yield return new WaitForSeconds(2f);    // let it initialize and run briefly
        Application.logMessageReceived -= OnLog;
        Assert.IsFalse(sawError, "Errors/exceptions during boot");
    }
}
```

## Headless rules (verified — easy to get wrong)

- **Use `WaitForSeconds`, not `WaitForEndOfFrame`.** In non-graphical
  (`-nographics`) CI runs there is no rendering, so frame-based yields don't tick
  and the test hangs or misbehaves. Time-based waits are portable across headless
  and on-device runs.
- **Watch `Application.logMessageReceived`** and fail on `LogType.Error` /
  `LogType.Exception` — the cheap, universal "something broke" signal.
- Keep waits short but real (a second or two): enough to catch boot/init crashes
  without slowing the suite.
- The PlayMode test assembly follows the usual UTF rules (own asmdef,
  `"includePlatforms": []` so it can run on a Player, references the game
  assemblies and `nunit.framework.dll`).

## Many levels: parameterize, don't hand-write

For a game with many scenes/levels, generate a smoke fixture per level rather than
writing one each — NUnit `[TestFixtureSource]` / `[ValueSource]` that enumerates the
build's level scenes, loading each and running the same generic "loads and runs
clean" assertion. Adding a level then automatically adds its smoke coverage.

## Running it

Via the Tools/Build Engineer's batch test command:
```
Unity -batchmode -runTests -projectPath <proj> \
      -testPlatform PlayMode -testResults <results.xml> -logFile <log> [-nographics]
```
(prepend `xvfb-run` on headless Linux per environment detection). The NUnit XML is
the machine-readable pass/fail the verification ladder and CI gate on. Never drive
this through a Unity Editor MCP — MCP is interactive/QA only.
