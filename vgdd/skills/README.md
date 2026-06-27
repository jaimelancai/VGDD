# VGDD Skills

Each skill is a **folder containing a `SKILL.md`**, discovered automatically and
invocable as `/vgdd:<skill-name>`. The `SKILL.md` frontmatter carries a `name`
(matching the folder) and a deliberately explicit `description` that says when to
fire — Claude tends to under-trigger skills, so descriptions are written to be
"pushy" about their triggers.

## Naming convention: flat folders, grouped by prefix

Claude Code discovers skills **one level deep** under `skills/` — a skill folder
must sit directly here, not inside an intermediate grouping folder. (Nested
grouping like `skills/directors/technical-director/` is silently *not*
discovered; this is a known Claude Code limitation.)

So we keep the conceptual grouping in the **name prefix** instead of in folders:

| Prefix | Layer | Examples |
|---|---|---|
| `meta-` | the skill system itself | `meta-using-vgdd` |
| `workflow-` | the studio loop | `workflow-intake`, `workflow-sprint`, `workflow-release`, `workflow-environment-detection`, `workflow-tool-provisioning` |
| `director-` | strategy & guardrails | `director-technical`, `director-game-design`, `director-qa`, `director-producer` |
| `engineer-` | per-task specialists | `engineer-gameplay`, `engineer-ui`, `engineer-backend`, `engineer-rendering`, `engineer-multiplayer`, `engineer-networking`, `engineer-tools-build` |
| `qa-` | testing & evals | `qa-tester`, `qa-automation`, `qa-smoke-test` |

The prefix preserves grouping in both the skill name and the alphabetical sort,
while keeping every skill discoverable.

## Engine overlays

Engine-specific detail lives in a sibling skill, not a nested folder — e.g.
`engineer-gameplay` (engine-agnostic) + `engineer-gameplay-unity` (Unity
overlay). Godot/Unreal overlays follow the same pattern later.

## Frontmatter: portable core only (for now)

We use only the **Agent Skills standard** fields `name` and `description`, plus
plain Markdown. We deliberately avoid Claude-Code-specific frontmatter
(`context: fork`, `hooks`, dynamic injection) until a skill genuinely needs it,
so skills stay portable to other harnesses (Codex, Gemini) when VGDD goes
multi-harness in a later phase.

## The map

Start with **`meta-using-vgdd`** — it is the index to everything here.
