---
name: director-technical-unity
description: "Unity engine overlay for the Technical Director. Loaded alongside director-technical when the resolved engine is Unity. Supplies the Unity-specific mechanics the base role defers: assembly-definition layout for headless testing, the batch-mode build/test invocation and its constraints, render-pipeline/input defaults, the scaffolding recipe, and Unity's C# coding conventions. Use whenever the Technical Director is acting on a Unity project."
---

# Technical Director — Unity overlay

This overlay supplies the **Unity-specific "how"** for the engine-agnostic
`director-technical` role. Load both together when the resolved engine is Unity.
The base skill owns the responsibilities; this owns the Unity mechanics.

## Stack defaults (resolving the tech spec)

- **Unity version** — `auto` → the latest installed Unity **LTS** (read from
  environment detection); else the pinned version (e.g. `2022.3.62f1`).
- **Render pipeline** — `auto` → **URP** for 2D/mobile (the common case); else
  Built-in or HDRP as specified.
- **Input system** — `auto` → Unity's current **Input System** package; `legacy`
  only if the spec requires it.

## Testability layout (how to satisfy the base's testability requirement)

- **Use assembly definitions (`.asmdef`)** to separate testable logic assemblies
  from engine-coupled ones, and to define EditMode and PlayMode test assemblies.
  This is what lets logic be unit-tested without entering Play Mode.
- Pure C# game logic (rules, scoring, match detection, state) goes in a plain
  logic assembly with **no** UnityEngine dependency where possible; MonoBehaviours
  in a thin engine assembly reference it.
- This structure is what makes `Unity -runTests -batchmode` fast and headless.

## Build & test invocation (and its constraints)

- Tests run via **`Unity -runTests -batchmode`** with `-testPlatform EditMode` /
  `PlayMode`, emitting NUnit-format XML to `-testResults`.
- **Never combine `-quit` with `-runTests`.** The test runner exits by itself when
  the run finishes; adding `-quit` makes Unity quit before tests complete and you
  get no results. `-quit` is for builds only. (Learned on a real Unity 6 run.)
- **Headless / no-GPU machines need `xvfb-run`** prepended to the Unity command
  (Linux virtual framebuffer); environment detection reports whether a display
  exists.
- **`-executeMethod` only calls parameterless static methods.** Build and scaffold
  entry points must be parameterless `static` methods in an `Assets/Editor/` folder
  that read arguments from environment variables / `Environment.GetCommandLineArgs()`,
  never from method parameters.
- Builds are produced via such an `-executeMethod` build target per platform.
- **Close the Unity Editor before batch builds/tests.** An open Editor holds the
  project lock (`Temp/UnityLockfile`), and batch-mode operations that write assets
  or run tests will conflict with it. If the Editor is open, do markdown-only work
  (other Directors, backlog) first and run the batch commands once it's closed.
  (Surfaced on a real run — the studio reordered around it, but plan for it.)

## Scaffolding recipe (the Unity "how" for the base's scaffold step)

For the **game client**, the scaffold the base requires means concretely:
- A Unity project at the **project path environment-detection resolved** — an
  existing project at the repo root or in `GameProject/` is **adopted where it is**;
  only a from-nothing project is created (default `GameProject/`). It must open and
  build on the resolved
  version, with URP/Input System configured as resolved.
- `.asmdef` files splitting a logic assembly from the engine assembly, plus
  EditMode/PlayMode test assemblies.
- A test harness wired so **Tier-0 tests run green** via `-runTests -batchmode`,
  including one trivial passing EditMode test proving the pipeline end-to-end.
- A parameterless `static` build entry point under `Assets/Editor/` for
  `-executeMethod`.
- **Unity git discipline, set up at scaffold time** (before the first commit):
  - a Unity-appropriate **`.gitignore`**: ignore `Library/`, `Temp/`, `obj/`,
    `Logs/`, `UserSettings/`, `MemoryCaptures/`, and the `builds/` output dir;
  - **track `.meta` files** — Unity generates one per asset and asset references
    break without them. `Assets/**` including every `*.meta` is version-controlled.
    (Learned on a real run: discovering this mid-sprint forces a corrective
    commit; set it up front.)
  - if the repo may move between OSes, a `.gitattributes` with `* text=auto`
    avoids line-ending churn.
- Committed on `develop` per GitFlow.

Backend/multiplayer skeletons the design requires are scaffolded in their own
runtimes (not Unity) following the base skill and the relevant engineer skills;
the client↔backend seam on the Unity side is a thin client stub in the logic
assembly so it stays unit-testable.

> **Never auto-install Unity** — it is large, licensed, and Hub-managed
> (provisioning rules). If environment detection reports no working Unity, stop
> and guide the human to install it (and the right LTS) before scaffolding.

## Conventions (the concrete set for Unity)

Default to **Microsoft's C# conventions** plus **Unity's C# style guidance**:
- PascalCase for methods, properties, types; camelCase for locals and private
  fields (or `_camelCase` if the tech design prefers); UPPER_CASE for constants.
- One type per file, file name matches the type; namespaces mirror folders.
- Prefer `[SerializeField] private` over public fields for inspector-exposed data.
- Keep MonoBehaviours thin; put logic in plain testable classes.

Record these in the architecture document (overriding only where the tech design
states a different preference) so every engineer follows the same rules.

## Editor MCP

If a Unity Editor MCP server is connected (per environment detection), use it for
**interactive and QA work only** — driving Play Mode, inspecting scene/GameObject
state, exploratory checks during development and the QA Director's pass. Note its
presence in the architecture document so engineers and the QA Director know they
can drive the live Editor.

**Do not route project setup, scaffolding, builds, or test runs through the MCP.**
Those stay on the CLI/batch path (`-runTests -batchmode`, the `-executeMethod`
build target) because they must be reproducible and CI-runnable, and CI has no live
Editor. The MCP is an interactive enhancement on top of that baseline, never a
replacement for it. (Same line the base skill and the QA Director draw.)
