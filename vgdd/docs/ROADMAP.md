# VGDD Roadmap

VGDD is built in phases so each can be reviewed and tested before the next.
This tracks the **framework build-out**, not the loop the framework runs.

| Phase | Goal | Status |
|---|---|---|
| **0. Foundations** | Repo skeleton, bootstrap, templates, core-loop + all four Directors, intake skills | **done** |
| **1. Engineering spine (Unity)** | End-to-end on Unity, one reference game, Tier-0 verified | planned |
| **2. QA & evals** | Reference-game suite + verification ladder gating changes | planned |
| **3. Tracking & collaboration** | Local backlog + GitHub adapter, Scrum demos, autonomy toggle | planned |
| **4. Depth & device** | Backend/multiplayer/networking + Tier-2 device playtest | planned |
| **5. Art & Audio** | Art/Audio Directors, asset pipeline | planned |
| **6. Multi-engine + distribution** | Godot overlay, plugin packaging, docs/examples gallery | planned |
| **7. Live-ops & maintenance** | Post-release intake (bug/feedback triage from tracker), regression-first QA, versioned releases, live-deploy always escalation-gated | planned |

> The studio is iterative *past* release: the first beta is a handoff into
> live-ops, not a finish line. Phases 0–6 build the studio that ships a game;
> Phase 7 builds the studio that keeps the shipped game alive. The release gate
> and autonomy wording in the foundations are already written to accommodate
> this (release is a recurring gate; shipping to players is always gated).

## Phase 0 deliverables (current)

1. Repo skeleton + game topology — **done (this commit)**
2. Bootstrap: `CLAUDE.md` / `AGENTS.md` with the autonomy contract — **done**
3. `meta-using-vgdd` map skill (folder + SKILL.md) — **done**
4. Input templates: GDD + Tech-Spec — **done**
5. Core-loop + sprint workflow skill (`workflow-sprint`) — **done**
6. Four Director skills (Technical, Game Design, QA, Producer) — **done**
   (Technical Director split into engine-agnostic base + `director-technical-unity`
   overlay)
7. Environment-detection + tool-provisioning skills
   (`workflow-environment-detection`, `workflow-tool-provisioning`) — **done**

**Phase 0 complete.** Authored: bootstrap (`CLAUDE.md`/`AGENTS.md`), the map
(`meta-using-vgdd`), both input templates, `workflow-sprint`, the four Directors
(`director-technical` + `director-technical-unity`, `director-game-design`,
`director-qa`, `director-producer`), and the two intake workflow skills.
Still `[planned]` for Phase 1: `workflow-intake`, `workflow-release`, and the
`engineer-*` / `qa-*` execution skills.

See `VGDD-PLAN-v3.md` for the full design rationale.
