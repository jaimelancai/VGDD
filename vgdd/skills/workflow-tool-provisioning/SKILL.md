---
name: workflow-tool-provisioning
description: "Handle missing tools (Git, Node, CLI utilities, runtimes) safely, without silently modifying the host machine. Applies a three-level policy from the tech spec: detect-and-guide (default), install-with-confirmation, or auto-install (CI/containers only). The game engine is NEVER auto-installed at any level. Trigger whenever any skill finds a required tool missing — at intake or mid-work."
---

# Tool provisioning

Installing system software is categorically different from writing code in a
project: it's privileged, OS-specific, hard to cleanly undo, and exactly what a
head-of-studio stakeholder should approve rather than discover afterward. This
skill governs *how far the studio may go* to set up a missing tool. Trigger it
whenever any skill hits a missing tool (Git, Node, a CLI utility, a backend
runtime) — at intake, or mid-work.

Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## The three-level policy

Read `tool_provisioning` from the tech spec (default `0`):

- **Level 0 — detect & guide** *(default)* — if a tool is missing, give the human
  the **exact OS-specific install command** and wait. Never install it yourself.
  This is the default for any detected **personal machine**.
- **Level 1 — install with confirmation** — may install a **narrow allowlist** of
  lightweight, reversible dev tools (Git, Node, common CLI utilities) via the
  platform's standard package manager, **announcing the exact command first**.
- **Level 2 — auto-install** — install allowlisted tools **without per-item
  confirmation**. Intended for **ephemeral CI/containers only**, never a personal
  machine (use the provisioning context from environment detection to tell which).

## Hard rules at every level

- **The game engine is NEVER auto-installed** — Unity (and Unreal, Godot) are
  large, license-gated, and Hub/launcher-managed. A missing engine is always
  **detect-and-guide**, regardless of level: stop and give the human clear
  install guidance (engine + correct version), then wait. This is a hard blocker
  the studio cannot work around.
- **Only the allowlist is ever auto/confirm-installed** — lightweight, reversible
  dev tools. Anything heavy, licensed, or system-altering beyond that is
  detect-and-guide.
- **Announce and log every install** — the exact command, written to the Studio
  Bible's provisioning log, even at level 2.
- **Be conservative on personal machines** — if environment detection flagged a
  personal (non-ephemeral) machine, surface what you're about to do even when a
  higher level is configured. When in doubt, drop to detect-and-guide.

## What markdown can't do

This governs the studio's *choices*; it can't override the harness. If the runtime
sandbox requires human approval before running any command, that approval is still
required no matter the provisioning level. Don't fight it — guide the human and let
them act.

## Flow

1. A skill reports a missing tool.
2. Is it the **game engine**? → detect-and-guide (hard rule), stop and guide.
3. Else read the level and the provisioning context:
   - Level 0, or a personal machine → give the exact install command, wait.
   - Level 1 → announce the command, install the allowlisted tool on confirmation.
   - Level 2 + ephemeral/CI → install the allowlisted tool, log it.
4. Record the outcome in `studio/bible/provisioning.md` (the log it owns).

## Definition of done

A provisioning request is resolved when the tool is either installed (per the
allowed level, logged) or the human has been given exact guidance and the blocking
item is recorded as waiting on them. Never silently skip a missing tool, and never
install outside the allowlist or the configured level.
