---
name: engineer-tools-build-unity
description: "Unity engine overlay for the Tools/Build Engineer. Loaded alongside engineer-tools-build when the engine is Unity. Supplies the Unity batch-mode mechanics: the Editor build script using BuildPipeline.BuildPlayer behind a parameterless -executeMethod entry point, the headless test-run command, checking BuildResult, xvfb on headless Linux, and CI wiring. Use when building Unity build/test plumbing."
---

# Tools/Build Engineer — Unity overlay

Unity-specific "how" for the engine-agnostic `engineer-tools-build` role. Load both
on a Unity project. Verified against current Unity command-line build docs.

## The build entry point

Builds run through a **static method in an `Assets/Editor/` folder** invoked by
`-executeMethod`. Remember the constraint (from the Technical Director): **`-executeMethod`
calls a *parameterless* static method** — pass build parameters via environment
variables or `Environment.GetCommandLineArgs()`, never as method arguments.

The method uses `BuildPipeline.BuildPlayer` with `BuildPlayerOptions`, and **checks
the returned `BuildReport`** so the batch build signals pass/fail correctly:

```csharp
using UnityEditor;
using UnityEditor.Build.Reporting;
using UnityEngine;

public static class VgddBuild   // in Assets/Editor/
{
    // Parameterless: read args from env / GetCommandLineArgs()
    public static void BuildPlayer()
    {
        var opts = new BuildPlayerOptions {
            scenes = EditorBuildSettingsScenesEnabled(),
            locationPathName = System.Environment.GetEnvironmentVariable("VGDD_BUILD_PATH"),
            target = ParseTarget(System.Environment.GetEnvironmentVariable("VGDD_BUILD_TARGET")),
            options = BuildOptions.None,
        };
        BuildReport report = BuildPipeline.BuildPlayer(opts);
        if (report.summary.result != BuildResult.Succeeded) {
            Debug.LogError("Build failed: " + report.summary.totalErrors + " errors");
            EditorApplication.Exit(1);   // non-zero exit so CI sees failure
        }
        EditorApplication.Exit(0);
    }
}
```

## The build command

```
Unity -quit -batchmode -projectPath <proj> \
      -buildTarget <Android|StandaloneOSX|StandaloneWindows64|iOS|WebGL|...> \
      -executeMethod VgddBuild.BuildPlayer \
      -logFile <build.log>
```
- `-projectPath` and `-quit` are **required for a build**; `-batchmode`,
  `-logFile`, and `-buildTarget` are recommended. `-nographics` may be added for
  headless builds that don't need the GPU.
- On **headless Linux**, prepend `xvfb-run` (per environment detection) so Unity has
  a virtual display.
- The script's `EditorApplication.Exit(1)` on failure is what gives CI a non-zero
  exit code — don't rely on the log alone.

## The test command (Tier 0 / 1)

Tests use a similar batch shape, plus `-runTests`:
```
Unity -batchmode -runTests -projectPath <proj> \
      -testPlatform <EditMode|PlayMode> \
      -testResults <results.xml> -logFile <test.log>
```
- **Do NOT pass `-quit` with `-runTests`.** They conflict — the test runner exits
  the editor itself when the run finishes, and adding `-quit` makes Unity quit
  before tests complete, producing no results. `-quit` is for **builds**;
  `-runTests` omits it. (Confirmed the hard way on a real Unity 6 run.)
- EditMode for fast logic tests (the Tier-0 workhorse); PlayMode for runtime.
- Emits **NUnit-format XML** to `-testResults` — machine-readable for CI gating.
- **If compilation fails, NO results XML is written** — the runner exits with the
  errors only in the log. Anything waiting on the XML (a watcher, a CI gate)
  must therefore also watch the **log and exit code**, or it deadlocks on a file
  that will never appear. (Hit on a real run: a rename's TDD red phase is a
  *compile error*, not a failing assertion — the XML-only watcher waited
  forever.)
- **Write run artifacts to one ignored directory, one naming scheme.** Send
  `-testResults` and `-logFile` outputs to a dedicated ignored dir (e.g.
  `Artifacts/`) with predictable names (`Artifacts/editmode-results.xml`,
  `Artifacts/editmode.log`), instead of ad-hoc names at the repo root — a real
  run accumulated a dozen differently-named logs/XMLs at root, making "did the
  run finish?" ambiguous and dirtying `git status`.
- **Watcher shells must not self-match.** `pgrep -f "Unity -batchmode"` matches
  any process whose *command line contains the string* — including the watcher
  shell itself (and its sibling watchers), so the wait loop never exits even
  after Unity is done. **Best: capture the PID at launch and wait on that exact PID** — a PID-wait
  cannot self-match or match siblings. If a pattern is unavoidable, use
  self-excluding forms: `pgrep -x Unity` (exact process name) or the bracket
  trick `pgrep -f "[U]nity -batchmode"`. And prefer **one** watcher over several
  concurrent ones — parallel watchers see each other. (All hit on a real run:
  four pattern-watchers deadlocked on each other — including the launcher, so
  the run never even started; the PID-keyed rewrite resolved it.)
- `xvfb-run` on headless Linux as above.

## CI wiring

**Cross-building constraint (verified):** IL2CPP players can only be built from an
editor **on the target platform** — the sole exception is *Linux as a target*,
which can be IL2CPP-cross-built from Windows/macOS via Unity's sysroot/toolchain
packages. Practical consequences: a Linux/macOS host can build a **Windows player
with the Mono backend only** (install the *Windows Build Support (Mono)* module
via Hub; the IL2CPP variant isn't offered there); a **Windows IL2CPP** build
requires a Windows host. Plan release builds accordingly — and remember building
an artifact is not smoking it: booting a Windows `.exe` still needs Windows (or
Wine).

When CI is present (per environment detection), the CI job **invokes these same
commands** — it is not a second build path. Typical gates: run EditMode + PlayMode
tests on every PR into `develop`; run a player build on release branches.
Shipping/deploy steps are **escalation-gated** (autonomy contract). Pin the Unity
version (per the resolved stack) so CI and local builds match.

## Reproducibility line (do not cross)

Never route build or test through a Unity Editor **MCP** — MCP is interactive/QA
only. Build and test are always the scripted batch commands above, because they
must run identically locally and in CI where no live Editor exists. (Same line the
Technical Director and QA Director draw.)
