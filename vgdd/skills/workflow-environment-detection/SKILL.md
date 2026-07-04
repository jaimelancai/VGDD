---
name: workflow-environment-detection
description: "Detect what the runtime environment offers and record it, so the rest of the studio plans within real limits instead of stalling. Checks for git + any remote, CI, a display/GPU, connected devices/simulators, the installed game engine, and any engine Editor MCP. Sets the active verification tier and notes the tool-provisioning context. Run once at intake (before pre-production), and again if the environment changes. Other skills read its findings; they don't re-detect."
---

# Environment detection

Run this **once at intake**, before pre-production, and again only if the
environment changes. Its job: find out what this machine can actually do, and
**write the findings into `studio/bible/environment.md`** (the document it owns) so every later skill plans within
real limits rather than discovering them mid-sprint or stalling. You *detect and
record*; you do not install anything (that's `workflow-tool-provisioning`) and you
do not decide the test strategy (that's `director-qa`, which reads what you found).

Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## What to detect

Probe for each, and record present/absent plus any detail:

1. **Git + remote** — is `git` available? Is there a remote configured (or a
   connector for one)? This informs `vcs` handling (local-only vs. push/PR).
   Also note whether **`git-lfs`** is installed — the Unity `.gitattributes`
   template declares LFS filters for binary assets, which must only be applied
   when the tool exists.
2. **CI** — is a CI system present (GitHub Actions config, Jenkins)? Enables
   Tier 3.
3. **Display / GPU** — is there a graphical display? On headless Linux, note that
   builds/Play-mode need a virtual framebuffer (the engine overlay handles the
   mechanics). Gates Tier 1.
4. **Connected devices / simulators** — any Android device, iOS/OS simulator
   attached? Gates Tier 2.
5. **The game engine** — is the engine named in the tech spec installed, and at
   what version? (Resolves `unity_version: auto` etc.) **If the required engine
   is missing, this is a hard blocker** — surface it and route to
   `workflow-tool-provisioning` for guidance (the engine is never auto-installed).
   **Also check the build-target support modules for every platform the tech spec
   names** (e.g. Unity's per-platform Build Support modules, Hub-installed): an
   engine can be present yet unable to *build the shippable target*. A missing
   module is not a blocker for development, but record it prominently and give
   the install guidance now — don't let "the shippable platform can't actually be
   built here" go undiscovered until someone first cuts that build. (Learned on a
   real run: the spec's Windows target was unbuildable for two sprints because
   the Linux host had only Linux Build Support, and nothing checked.)
6. **Engine Editor MCP** — is an engine Editor MCP server (e.g. a Unity Editor
   MCP) connected? If so, note it: it's available for *interactive/QA work only*
   (not for setup/build/test — see the Technical Director).
7. **Other required runtimes** — if the design needs a backend/services, are their
   runtimes (e.g. Node, a DB) present? Note gaps for provisioning.
8. **Project layout** — is there **already an engine project**, and where? Look for
   engine project markers (for Unity: `Assets/`, `ProjectSettings/`,
   `ProjectSettings/ProjectVersion.txt`). Record which layout applies:
   - **at the repo root** (markers alongside `.vgdd/`, `studio/`, `design/`) — the
     common case when someone drops VGDD into a project they already created;
   - **in a `GameProject/` subfolder** — the VGDD scaffold default;
   - **none yet** — a fresh folder; the Technical Director will create the project
     (scaffolding into `GameProject/` by default).
   The studio **adopts an existing project wherever it is** rather than
   restructuring it. Record the resolved project path so every skill uses it.

## Setting the verification tier

From what you found, record the **highest verification tier the environment
supports** (the QA Director reads this and decides the test strategy):

- **Tier 0** — unit/integration: always available once the engine is installed.
- **Tier 1** — build + smoke: requires a display (or a virtual framebuffer).
- **Tier 2** — device playtest: requires a connected device/simulator.
- **Tier 3** — CI: requires a CI system.

Tier 0 is the floor. Record the **ceiling** the environment allows and note what
is *not* reachable, so the studio never implies more verification than it can run.
Also record whether an **Editor MCP** is in the loop for interactive checks.

## Setting the provisioning context

Note which `tool_provisioning` level applies (from the tech spec) and whether this
looks like an **ephemeral/CI/container** context (where level 2 auto-install may
apply) or a **personal machine** (stay conservative). `workflow-tool-provisioning`
uses this when a missing tool is hit.

## Recording the findings

Write a clear **environment report** into `studio/bible/environment.md`: each capability
present/absent, the resolved engine version, the active verification tier ceiling
and what's unreachable, Editor MCP presence, and any missing tools/runtimes with
their impact. This report is the single source other skills read — they do **not**
re-detect.

## Definition of done

Detection is done when `studio/bible/environment.md` contains a complete report:
git/remote, CI, display, devices, engine (+version or a flagged missing-engine
blocker), Editor MCP, required runtimes, the verification-tier ceiling, and the
provisioning context. If a later skill would have to ask "can we build here? can
we test on a device? is the engine even installed?", detection isn't done.
