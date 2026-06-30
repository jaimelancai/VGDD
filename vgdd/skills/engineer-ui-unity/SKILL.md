---
name: engineer-ui-unity
description: "Unity engine overlay for the UI Engineer. Loaded alongside engineer-ui when the engine is Unity. Supplies the Unity UI mechanics: working in whichever UI system the Technical Director chose (UI Toolkit or uGUI), keeping presentation logic in a testable assembly separate from view objects, and the defaults to use when the architecture doesn't specify. Use when implementing UI on a Unity project."
---

# UI Engineer — Unity overlay

Unity-specific "how" for the engine-agnostic `engineer-ui` role. Load both on a
Unity project.

## Which UI system — read it, don't pick it

Unity has two runtime UI systems. **The Technical Director chooses which**, recorded
in `studio/bible/architecture.md` — you work within that choice:

- **UI Toolkit** — Unity's recommended system for screen-overlay/menu/data-heavy UI
  across resolutions. Layout lives in **UXML** files; styling in **USS**; a
  `UIDocument` MonoBehaviour loads the UXML and you resolve controls at runtime by
  **query** (`root.Q<Button>("play")`), not editor-assigned references. It
  separates layout from logic by design.
- **uGUI (Unity UI)** — GameObject/Canvas-based, prefab-driven; strong for animated
  HUDs and where the team wants Scene-view visual editing or mature asset-store
  tooling.

**Default when the architecture doesn't specify:** UI Toolkit for menus and
screen-overlay UI (Unity's current recommendation), noting it for the TD to
confirm. The two can coexist (uGUI HUD + UI Toolkit menus) but don't scatter them
without reason — a unified choice is easier to keep coherent.

## Keep presentation logic testable (the base rule, in Unity)

The base's "presentation logic goes in plain testable code" maps to the same asmdef
discipline the Gameplay Engineer uses:

- **Presentation logic** — score formatting, what-screen-shows-given-state, flow/
  navigation state machines — lives in a plain **logic/presentation assembly** with
  no dependency on the view layer, so it's **EditMode-unit-testable** (it can't sit
  in `Assembly-CSharp`, or the test assembly can't reference it).
- **View objects** — the `UIDocument`+UXML (UI Toolkit) or MonoBehaviour+Canvas
  (uGUI) — stay thin: they bind the presentation logic's output to controls and
  forward control events as **player intents** to gameplay logic.
- A button handler should call into logic/intent, not contain game rules.

Example (UI Toolkit view binding to testable logic):
```csharp
// View (thin): resolve controls, bind, forward intent
var root = GetComponent<UIDocument>().rootVisualElement;
var scoreLabel = root.Q<Label>("score");
scoreLabel.text = ScorePresenter.Format(gameState.Score);   // tested logic
root.Q<Button>("swap").clicked += () => intents.Raise(new SwapIntent(a, b));
```
`ScorePresenter.Format` and flow state are unit-tested in EditMode; the binding is
exercised by the QA pass / PlayMode smoke.

## Input

Use the input system the TD resolved (default: Unity's current Input System). UI
events (clicks/taps) flow through the chosen UI system; translate them into the
intents gameplay logic consumes — don't resolve game rules in the handler.

## Conventions & assets

Follow `studio/bible/architecture.md` conventions. Use **placeholder visuals**
(solid colors, default fonts, simple shapes) this phase — the Art Director arrives
later; never block UI work waiting on final art.

## Editor MCP

If a Unity Editor MCP is connected, use it to **interactively check** layout/flow in
Play Mode while developing. Verification still rests on the EditMode tests
(presentation logic) and PlayMode/build smoke (the QA Director), not the MCP.
