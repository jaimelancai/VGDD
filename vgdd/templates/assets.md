---
# Assets Manifest — OPTIONAL third input document (alongside game-design.md and
# tech-spec.md). Omitting this file entirely is a normal, often deliberate
# choice: the studio builds and demos the whole game on generated placeholders,
# so you can validate that the mechanics work and are engaging FIRST, and add
# real art/audio later to complete it. Provide this file (and design/assets/)
# whenever you're ready — at the start, mid-project, or after the MVP proves
# itself — and the studio integrates the assets as normal backlog work.
status: ready
---

# Assets Manifest

> **What this is.** A description of the art and audio source files you are
> providing to the studio, placed in `design/assets/`. The studio's technical
> artist (`tech-art`) processes them into engine-ready assets; anything not
> provided gets a generated placeholder and appears on the demo's "assets
> wanted" list. Describe intent, not just filenames — the studio maps your
> files to the game's needs.

## Art

<!-- One entry per asset or asset group. Say what it is FOR, where it lives,
     and anything special about it. Example entries: -->

- **Piece sprites** — `assets/pieces/` — 6 PNGs, one per colour (red, blue,
  green, yellow, purple, orange), 256×256, transparent background.
- **Crate blocker** — `assets/blockers/crate.png` — plus optional
  `crate_broken.png` for the destruction state.
- **UI buttons** — *(not provided — use placeholders)*.

## Audio

<!-- Same idea. Formats: prefer WAV/OGG for SFX, OGG/MP3 for music. -->

- **Match SFX** — *(not provided yet)*.
- **Music** — *(not provided — omit or placeholder-silence)*.

## Fonts

- *(none provided — default/engine fonts are fine)*.

## Notes & intent

<!-- Anything the studio should know: visual style references, licensing
     constraints on provided files, naming schemes, which assets are final vs.
     drafts likely to be replaced. -->

## Attribution / licensing

<!-- If any provided assets carry license obligations (CC-BY credit lines,
     store-asset license limits), note them here — the studio records them and
     carries required attributions into the game/release notes. -->
