---
name: workflow-release
description: "Cut a release of the game — stabilize a release branch, confirm the verification gates passed, have the player built per platform (by the Tools/Build Engineer), then version and tag it (semver). The first release is the beta. Stops at the ship line: putting a build in players' hands (store submission, live deploy) is always escalation-gated and human-initiated. Engine-agnostic — it orchestrates, calling the Tools/Build Engineer to build and relying on the QA Director for verification; it does not build or test directly. Use when a release target is met, or for a post-release hotfix."
---

# Release

This skill cuts a **release**: it takes integrated work to a versioned, tagged,
built artifact — and **stops** at the line where a build would reach players. It
**orchestrates** the release; it doesn't build or test directly — it calls the
skills that own those.

This is **engine-agnostic**: the build mechanics belong to the Tools/Build Engineer
(and its engine overlay), so release has no engine overlay of its own — it *calls
down* rather than duplicating.

Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## When to run

- A **release target is met** — for the first release, the Game Design Director's
  **minimum-shippable definition** is satisfied (this is the **beta**); later, a
  planned milestone.
- A **post-release hotfix** is needed for a live build (the `hotfix/*` path below).

## What you own vs. defer

- **You own** — the release lifecycle: the `release/*`/`hotfix/*` branch flow,
  versioning + tagging, the release checklist, and enforcing the ship gate.
- **Building the player** → the **Tools/Build Engineer** (its engine overlay runs
  the reproducible build per platform). You invoke it; you don't build directly.
- **Verification** → the **QA Director** / the verification ladder. You confirm the
  gates passed; you don't re-run the test strategy.
- **Branch mechanics** → the **Producer's** GitFlow scheme.

## The release flow

1. **Open a release branch.** Per GitFlow, branch `release/<version>` off
   `develop` (or `hotfix/<version>` off `main` for a live fix). Stabilization
   (final fixes, no new features) happens here.
2. **Confirm the verification gates are green.** Check that the work in this
   release passed its tier as the QA Director defined: Tier-0 unit tests green, and
   the highest available tier attempted (build+smoke via `qa-smoke-test`,
   device/CI where present). Confirm the test plan's relevant cases pass and — for
   a hotfix or post-release — the **regression suite (incl. save compatibility) is
   green**. Do not release on red.
3. **Build the player.** Invoke the **Tools/Build Engineer** to produce a build per
   target platform from the reproducible batch command (never by hand, never via an
   engine MCP). A build that doesn't succeed cleanly blocks the release.
4. **Version and tag.** Use **semantic versioning** `major.minor.patch`:
   - the **first beta is `0.1.0`** (pre-1.0 signals "not the finished product");
   - subsequent releases bump **minor** for features, **patch** for fixes/hotfixes,
     and **major** at a deliberate milestone (e.g. `1.0.0` for the full target).
   Tag the release commit (e.g. `v0.1.0`) and merge per GitFlow: `release/*` →
   `main` **and** back into `develop`; `hotfix/*` → `main` **and** `develop`.
   > The `→ main` merge is the moment `main` becomes "what a release points to."
   > Once the game is live, that merge feeds the ship gate below.
5. **Run the release checklist** — version bumped and tagged; build artifact(s)
   produced and recorded in `builds/`; gates green and the **achieved verification
   tier stated plainly** (don't imply more than ran); release notes summarizing
   what's in this version; the Studio Bible updated.
6. **STOP at the ship line.** See below.

## The ship line — always escalation-gated, human-initiated

Building and tagging a release is something the studio does on its own. **Putting
that build in players' hands is not.** Any action that distributes a build to
people — a store submission, a live deploy, pushing an update to existing players —
is **always escalation-gated, in every autonomy mode**, because mistakes now reach
real users (autonomy contract).

So this skill **stops after build + tag** and hands the decision to the human:
present the tagged, built, verified release and the notes, and let the **human
initiate** the actual ship. The studio does not deploy on its own — not in
`autonomous` mode, not for a hotfix, not ever. Deploy is human-initiated; the
studio can *prepare* everything up to that final step.

## After release: hand to live-ops

A release is **not the end** — it's a handoff into live-ops (see the studio map).
The first beta hands off; the studio keeps looping on bug reports and feedback
through the normal sprint loop, with regression safety first-class and every future
ship equally gated. After tagging, note the release in the Studio Bible and return
to the loop.

## Definition of done (for a release)

- A `release/*` (or `hotfix/*`) branch was stabilized and merged per GitFlow.
- The verification gates are **green** at the achieved tier (regression suite green
  for hotfixes/post-release), and that tier is stated honestly.
- The player is **built** per target platform (via the Tools/Build Engineer) and
  recorded in `builds/`.
- The release is **versioned (semver) and tagged**, with release notes.
- The studio **stopped at the ship line** and handed the deploy decision to the
  human — it did not distribute the build itself.
