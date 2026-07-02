# Studio Bible — `studio/bible/`

The Studio Bible is the studio's **shared blackboard**: the durable, human-readable,
git-versioned place the skills coordinate through. Skills are invoked
independently (often as fresh subagents with no memory of each other), so they
communicate by **reading and writing shared documents**, not by talking directly.

It is a **folder of single-owner documents**, not one big file — each document has
a different lifecycle and exactly one writer, so changes stay isolated and skills
load only what they need.

> This folder under `templates/` is the blank set. At runtime the live copy is
> `studio/bible/` in the project (created during intake/pre-production).

## The documents and their owners

| Document | Owner (only writer) | Contents | Lifecycle |
|---|---|---|---|
| `architecture.md` | **Technical Director** | resolved stack, project structure, conventions, integration workflows | stable reference, deepens as systems are built |
| `design.md` | **Game Design Director** | pillars, precise core loop, mechanics, progression, the minimum-shippable definition + full target | stable reference, deepens with the design |
| `environment.md` | **workflow-environment-detection** | capability report (git/remote, CI, display, devices, engine+version, Editor MCP, runtimes), the active verification-tier ceiling, provisioning context | snapshot, rewritten only if the environment changes |
| `provisioning.md` | **workflow-tool-provisioning** | log of tools installed/guided, with exact commands | append-only log |
| `assets.md` | **tech-art** | asset inventory: provided assets (source → engine mapping, licenses), placeholders (= the assets-wanted list), asset conventions | living, updated as assets arrive |
| `decisions.md` | **shared, append-only** | assumptions and decisions logged as the studio resolves ambiguity (each entry notes which skill made it) | append-only journal |

The **backlog** (`studio/backlog.md`), **board** (`studio/board.md`), and **test
plan** (`studio/test-plan.md`) are separate top-level `studio/` artifacts with
their own owners (Producer, Producer, QA Director) — they are **not** inside the
Bible.

## The rules that keep the blackboard from rotting

1. **Single writer per document.** Only the owning skill writes its document;
   everyone else **reads**. This prevents the two-masters/drift problem (the same
   discipline that makes `backlog.md` canonical and `board.md` a pure projection).
2. **`decisions.md` is the one shared-writer file** — and only because it is
   **append-only**. Appends don't conflict the way edits do. Every skill may add
   an assumption/decision entry (tagged with the skill name); no skill edits
   another's entries.
3. **Read only what you need.** A skill loads the document(s) relevant to its task
   (the QA Director reads `environment.md` for the tier; an engineer reads
   `architecture.md`), not the whole folder — this keeps per-invocation context
   small.
4. **Thin then deepen.** `architecture.md` and `design.md` start as thin
   foundations and grow as systems/features are built — same discipline as the
   test plan. Don't front-load detail that doesn't exist yet.

> Token note: the folder is usually *cheaper* per invocation than one big file,
> because skills read only their document. The real growth watch-items —
> `decisions.md` over a long project, and the deepening reference docs — are
> tuning concerns (summarize/archive) to address once real runs show their sizes,
> not a reason to centralize.
