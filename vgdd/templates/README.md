# Templates

What the stakeholder fills in (inputs) and reference material the studio copies.

- `game-design.md` — the GDD template (input, required).
- `tech-spec.md` — the Technical Spec template, incl. `autonomy_level` (input;
  may be left on defaults).
- `assets.md` — the asset manifest (input, **optional**): describes art/audio
  source files in `design/assets/`. Omitting it is normal — the studio builds on
  generated placeholders (see `tech-art`).
- `studio-bible/` — the Studio Bible: a folder of single-owner documents the
  skills coordinate through (architecture, design, environment, provisioning,
  assets inventory, and the append-only decisions log). **Reference for structure
  only** — owners create the live docs; cold-start does not pre-copy these.
  See `studio-bible/README.md`.
- `gitignore-unity` + `gitattributes-unity` — Unity project git files, copied to
  the game project root by the Technical Director at scaffold time (`.meta`
  files stay tracked; the gitattributes' LFS section requires `git-lfs`).

(Epic/story/task formats live in `director-producer` — the Producer owns the
backlog's shape and evolves it with use.)
