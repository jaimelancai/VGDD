---
name: engineer-gameplay-unity
description: "Unity engine overlay for the Gameplay Engineer. Loaded alongside engineer-gameplay when the engine is Unity. Supplies the Unity mechanics: how to structure logic vs. MonoBehaviour code across assembly definitions so it's unit-testable, how to write EditMode/PlayMode tests with the Unity Test Framework, and how they run headlessly in batch mode. Use when implementing gameplay on a Unity project."
---

# Gameplay Engineer — Unity overlay

Unity-specific "how" for the engine-agnostic `engineer-gameplay` role. Load both
together on a Unity project. The base owns the approach; this owns Unity mechanics.

## Logic vs. MonoBehaviour, across assemblies

The base rule "keep logic engine-free and testable" is realized in Unity through
**assembly definitions (`.asmdef`)** — and there's a hard constraint that forces
it: a test assembly **cannot reference the predefined `Assembly-CSharp`**, so any
code you want to unit-test **must** live in its own asmdef assembly.

So follow the structure the Technical Director scaffolded:
- **Logic assembly** (e.g. `Game.Logic`) — pure C# rules, scoring, match
  detection, state machines, win/lose. **No `UnityEngine` dependency** where
  possible (or only lightweight value types). This is what the test assembly
  references and what runs fast in EditMode.
- **Engine assembly** (e.g. `Game.Runtime`) — the MonoBehaviours that wire input
  and rendering to the logic. Thin: they call into `Game.Logic`, they don't hold
  the rules.

Putting rules directly in a MonoBehaviour means they land in `Assembly-CSharp` and
**can't be unit-tested** — exactly the Tier-0 failure to avoid.

## Writing tests (Unity Test Framework)

- **EditMode tests** for pure logic — fast, no play mode. Test asmdef has
  `"includePlatforms": ["Editor"]`, references the logic assembly and
  `nunit.framework.dll`, and uses the NUnit `[Test]` attribute. **Prefer these for
  gameplay logic** — they're the Tier-0 workhorse.
- **PlayMode tests** only when you must exercise runtime/engine behavior — test
  asmdef has `"includePlatforms": []` (empty — not `["Editor"]`, or they won't run
  on a Player), and uses `[UnityTest]` with a coroutine (`yield return null;` to
  advance frames).
- Keep EditMode and PlayMode tests in **separate assemblies** (UTF requires it).
- Add `"defineConstraints": ["UNITY_INCLUDE_TESTS"]` so test assemblies are
  excluded from shipping builds.
- **Use the modern test-asmdef format** — explicit `references` to
  `"UnityEngine.TestRunner"` and `"UnityEditor.TestRunner"` (plus your game
  assemblies). **Do NOT use the deprecated `"optionalUnityReferences": ["TestAssemblies"]`** —
  on Unity 6 it conflicts with explicit TestRunner references and produces
  "duplicate references" errors. (Hit and fixed on a real Unity 6 run.) A working
  EditMode test asmdef:
  ```json
  {
    "name": "Match3.Logic.Tests",
    "references": ["Match3.Logic", "UnityEngine.TestRunner", "UnityEditor.TestRunner"],
    "includePlatforms": ["Editor"],
    "defineConstraints": ["UNITY_INCLUDE_TESTS"],
    "overrideReferences": true,
    "precompiledReferences": ["nunit.framework.dll"]
  }
  ```

Example EditMode test (logic, no engine):
```csharp
using NUnit.Framework;
using Game.Logic;

public class MatchRulesTests
{
    [Test]
    public void ThreeInARow_Clears()
    {
        var board = new Board(/* ... */);
        var result = MatchRules.Resolve(board, swap);
        Assert.IsTrue(result.Cleared.Count >= 3);
    }
}
```

## Running tests (Tier-0, headless)

Tests run via the batch command the Technical Director's pipeline defines:
```
Unity -batchmode -runTests -projectPath <proj> -testPlatform EditMode \
      -testResults <results.xml>
```
(prepend `xvfb-run` on headless Linux; PlayMode via `-testPlatform PlayMode`).
You don't own the invocation — the Tools/Build Engineer and TD do — but write your
tests so they pass under it: deterministic, no reliance on a display for EditMode
logic tests.

## Conventions

Follow the C# conventions in `studio/bible/architecture.md` (Unity defaults:
PascalCase methods/types, camelCase locals, `[SerializeField] private` for
inspector data, one type per file). Keep MonoBehaviours thin; put logic in the
logic assembly.

## Editor MCP

If a Unity Editor MCP is connected, it's available for **interactive checks** while
developing (drive Play Mode, inspect state) — useful for sanity-checking feel. But
your *verification* is the EditMode/PlayMode tests run in batch mode, not the MCP
(reproducibility). Same line the Technical Director and QA Director draw.
