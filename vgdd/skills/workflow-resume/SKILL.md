---
name: workflow-resume
description: "Resume an existing VGDD project in a fresh session — the counterpart to workflow-intake (which is for brand-new projects). Reconstructs 'where are we?' entirely from the durable state on disk (studio/ files + git), then continues from the right point: mid-sprint, between sprints, or between releases. This is what lets a project outlive a single context window — clear context at a safe checkpoint, start a new session, and pick up exactly where you left off. Use when opening an existing VGDD project that already has studio/ state, rather than starting fresh."
---

# Resume

VGDD keeps **all durable state on disk** — the backlog, board, test plan, Studio
Bible, and git (branches, commits, tags). That means a project does **not** depend
on one long-lived session: you can work to a safe checkpoint, **clear the context**,
start a fresh session, and resume — because everything that matters was written
down, not held in memory. This skill is that resume path.

Use it when a VGDD project **already exists** (there's a populated `studio/`) and
you're starting a new session. For a brand-new project, use `workflow-intake`
instead. Everything here defers to the **autonomy contract** in `/CLAUDE.md`.

## How to reconstruct "where are we?" (read, don't assume)

You are a fresh session with an empty context — rebuild the picture from disk:

1. **The Studio Bible** — read `studio/bible/`: `environment.md` (stack, tier,
   resolved project path), `architecture.md` (structure + conventions),
   `design.md` (pillars, core loop, minimum-shippable target), and skim
   `decisions.md` (what's been decided/assumed). This is the project's memory.
2. **The backlog** — read `studio/backlog.md`: which stories are `done`,
   `in-sprint`, `in-review`, `ready`, `backlog`. This tells you exactly how far the
   game has been built and what's next.
3. **The board** — glance at `studio/board.md` (the by-status projection) for a
   quick state overview.
4. **The test plan** — `studio/test-plan.md`: what's verified and what cases exist.
5. **Git** — check the current branch, recent commits, and tags:
   - on a `feature/<story-id>` branch with uncommitted or recent work → a **sprint
     is mid-flight**;
   - on `develop` with the last tag `demo/sprint-N` → **between sprints**;
   - last tag a release (`v*`) on `main` → **between releases / live-ops**.

**Before trusting `environment.md`, check it still describes this machine.** The
recorded environment may be from a different host — projects move (Linux →
Windows, a new machine, an upgraded engine). Compare cheap markers against the
report: current **OS**, whether the recorded **engine binary path exists**, and
whether the recorded display/CI facts plausibly apply. On any mismatch, **re-run
`workflow-environment-detection`** before continuing — the tier ceiling, engine
path, and build strategy (e.g. "Linux build for local smoke") may all change with
the host. On a match, trust the report and don't re-detect.

## Where to resume from

From the reconstructed state, pick up at the right point:

- **Mid-sprint** (a `feature/*` branch with the sprint's stories partly done) —
  read which of the sprint's stories/tasks are complete (backlog status + commits),
  then hand back to **`workflow-sprint`** to continue the *remaining* stories.
  Finished stories are not redone; their tests already prove them.
- **Between sprints** (on `develop`, sprint N tagged, next stories `ready`) —
  confirm the `ready` set still makes sense, then start the next sprint via
  **`workflow-sprint`**. In `collaborative` mode, this is also the natural point to
  fold in any stakeholder feedback from the last demo.
- **Between releases / live-ops** — resume the live-ops loop: **any new
  stakeholder input found on disk (new/changed files under `design/`, e.g. an
  asset delivery) or reported in conversation goes to the Producer for triage
  into the backlog first**, then flows through the normal sprint loop — bugs
  additionally get reproduce-first cases (QA Director). Never implement new
  input directly without a backlog story and a sprint.

Then continue the normal loop. You are not re-planning the project — you're
re-entering a loop that was designed to be re-enterable.

## Safe checkpoints (where it's safe to clear context)

A checkpoint is **safe to clear** when the durable state on disk is fully
consistent — nothing important lives only in context. The guaranteed-safe points:

- **A sprint demo / sprint boundary** — the primary, recommended clear-point. The
  sprint's work is committed, statuses are written via set-status, the Bible is
  updated, and (collaborative mode) the studio has stopped for feedback. Clearing
  here loses nothing.
- **A completed story within a sprint** — the fallback clear-point for an unusually
  large sprint. Each story lands as committed tasks on its feature branch with
  green tests and written status, so a story boundary is also safe.

Prefer clearing at the **sprint boundary** (it matches the scrum rhythm); use the
**story boundary** only when a single sprint is large enough to strain the context
budget. In practice a sprint is a small fraction of a large context window, so the
sprint boundary is usually enough.

> `workflow-sprint` commits, sets status, and updates the Bible at each story and
> at sprint close precisely so these are safe clear-points. Don't clear in the
> *middle* of a task (a half-written test/impl not yet committed) — finish and
> commit the task first.

## Definition of done (for a resume)

Resume is complete when: the Studio Bible, backlog, board, test plan, and git state
have been read; the current position (mid-sprint / between-sprints / live-ops) is
identified; and control has been handed to the right skill (`workflow-sprint` or the
live-ops loop) to continue — **without** re-planning or redoing already-`done` work.
If you find yourself re-deciding architecture or re-running finished stories, you've
misread the durable state — re-read it.
