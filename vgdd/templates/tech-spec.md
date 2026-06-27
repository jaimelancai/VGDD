---
# Only the few settings a non-agent process (a hook, a tracking adapter) must
# read deterministically live here. Everything else is prose below — the
# Technical Director reads it and logs any defaults it applies as assumptions.
# All values are OPTIONAL; the value shown is the default if you leave it.
status: draft                  # draft | ready  — set "ready" to let the studio begin
autonomy_level: collaborative  # collaborative | autonomous
tool_provisioning: 0           # 0 detect-and-guide | 1 install-with-confirm | 2 auto (CI only)
vcs: git-local                 # git-local | github | gitlab
tracker: local                 # local | github | jira | gitlab | trello
---

# Technical Specification

> **How the studio reads this.** This is the *how* — the technology, targets, and
> the controls for how the studio works. The *what* (the design) lives in
> `game-design.md`.
>
> **You can leave all of this blank.** Nothing here is required to start: a sane
> default stack exists for every field. A non-technical stakeholder can set
> `status: ready` and let the Technical Director choose everything, logging each
> choice as an assumption you review at the first demo. A technical user can
> override only the fields they care about.
>
> **Why so little in the header?** The short block at the top (the frontmatter
> between the `---` lines) holds only the handful of settings that a plain
> script or a tracking connector needs to read exactly — where guessing would be
> unsafe. Everything else is ordinary prose below; the studio reads it the same
> way it reads your game design. Each setting names its default inline, so the
> document also documents the studio's defaults.

---

## The five header settings

These are the settings in the frontmatter above. They live there because a
non-agent process may need to read them without interpretation.

**`status`** *(default: `draft`)* — the studio begins only when this is `ready`.
Leave it `draft` while you're still writing; flip to `ready` to start.

**`autonomy_level`** *(default: `collaborative`)* — how much the studio does
between check-ins. The **escalation line** (it always stops before irreversible,
costly actions — force-push, deleting your work, spending money, publishing)
applies in **both** modes; this only changes the sprint-demo gate.
- `collaborative` — a playable demo every sprint, and it **stops for your
  feedback** before continuing. Best if you want to steer.
- `autonomous` — runs straight through to a first beta; demos are still built and
  tagged each sprint but don't block. Stops only at the escalation line. Best if
  you want to hand it the docs and walk away.
Full details: the autonomy contract in `/CLAUDE.md`.

**`tool_provisioning`** *(default: `0`)* — how far the studio may go to set up
missing tools (Git, Node, CLI utilities) on your machine:
- `0` — **detect & guide**: gives you the exact install command and waits; never
  installs anything itself.
- `1` — **install with confirmation**: may install lightweight, allowlisted tools
  after showing you the command.
- `2` — **auto-install**: installs allowlisted tools without asking — for
  disposable CI/containers, **not** a personal machine.
**Unity is never auto-installed at any level.** Every install is logged in the
Studio Bible.

**`vcs`** *(default: `git-local`)* — `git-local` initializes a local git repo
with **no remote** (zero setup, full history and branching locally). Set `github`
or `gitlab` (with the matching connector) to also push to a remote and open PRs.

**`tracker`** *(default: `local`)* — where the backlog lives. `local` keeps
tickets as a committed `studio/backlog.md` file (works offline, no accounts).
Switch to `github`, `jira`, `gitlab`, or `trello` to mirror tickets there via the
matching connector, so you can watch work flow board-to-done in a tool you use.

> If you name a remote `vcs`/`tracker` but its connector isn't available, the
> studio falls back to the local default and logs it — it never stalls waiting
> for a connection.

---

## Targets

Describe these in plain words; the Technical Director reads them.

**Platforms** *(default: Android)* — where the game must run. Android is the
default because it's the easiest to build and test without paid signing. Name any
of: Android, iOS, Windows, macOS, Linux, WebGL. *e.g. "Android phones first,
maybe iOS later."*

**Orientation** *(default: portrait, by genre)* — portrait, landscape, or both.
Left unsaid, the studio picks by genre (casual/match-3 → portrait).

---

## Engine

**Engine** *(default: Unity)* — Unity is the only supported engine today; Godot
and Unreal arrive as overlays in a later phase.

> **Unity must already be installed.** The studio will **not** install Unity for
> you at any provisioning level — it's large, licensed, and Hub-managed. If it's
> missing, the studio stops and guides you to install it.

**Unity version** *(default: latest installed LTS)* — leave it and the studio
uses the newest installed Unity LTS. Pin an exact version (e.g. *2022.3.62f1*)
for reproducibility across machines or CI.

**Render pipeline** *(default: chosen by the studio — URP for 2D/mobile)* —
override to Built-in, URP, or HDRP only if you have a reason.

**Input system** *(default: Unity's current Input System)* — say so if you need
the legacy input manager instead.

---

## Verification

The studio climbs a ladder of test rigor **as high as your environment allows**
and records what it actually ran — you don't need to set anything. Mention a cap
here only if you want to *limit* how far it goes.

- **Tier 0** — unit/integration tests (always available).
- **Tier 1** — build + automated smoke test (needs a display).
- **Tier 2** — device/simulator playtest (needs a connected device).
- **Tier 3** — escalate to CI (GitHub Actions/Jenkins) if present.

If an engine **Editor MCP** (e.g. a Unity Editor MCP server) is connected, the
studio can also drive the live Editor for interactive checks — detected
automatically, recorded in the Studio Bible.

---

### Nothing to fill in?

That's fine — leave the header on its defaults, set `status: ready`, and the
Technical Director configures a sensible Unity-on-Android stack, runs
collaboratively with local git, guides you through any missing tools, and logs
every decision for you to review at the first demo.
