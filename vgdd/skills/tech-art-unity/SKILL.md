---
name: tech-art-unity
description: "Unity engine overlay for the Technical Artist. Loaded alongside tech-art when the engine is Unity. Supplies the Unity mechanics: sprite/texture import settings, in-project organization under Assets/Art, procedural placeholder generation, and the EditMode load-check pattern that verifies every inventory asset resolves. Use when processing art assets on a Unity project."
---

# Technical Artist — Unity overlay

Unity-specific "how" for the engine-agnostic `tech-art` role. Load both on a
Unity project. The base owns the pipeline/inventory; this owns Unity mechanics.

## Organization & naming (record in the inventory)

- Art lives under **`Assets/Art/`**, grouped by kind:
  `Assets/Art/Sprites/Pieces/`, `Assets/Art/Sprites/Blockers/`,
  `Assets/Art/Sprites/UI/`, `Assets/Art/Fonts/`.
- File names `PascalCase`, matching what code references:
  `PieceRed.png`, `PieceBlue.png`, `Crate.png`, `CrateBroken.png`.
- Source files from `design/assets/` are **copied in** (never referenced
  in place) so the Unity project is self-contained; the inventory records the
  source → project mapping.
- **`.meta` files are version-controlled** (per the TD's Unity git discipline) —
  an asset's import settings live in its `.meta`, so losing it loses the
  configuration.

## Sprite import settings (2D games)

> This overlay currently covers **2D** asset mechanics, validated on a real 2D
> project. **3D asset mechanics are [planned]** — model import (FBX/glTF, scale,
> rigs/animations), materials/shaders (URP Lit, texture maps), mesh
> compression/LODs, prefab-per-model conventions, and primitive-based 3D
> placeholders (`GameObject.CreatePrimitive` + flat-colour materials). They'll be
> written and validated when a 3D game is first tested, extending this file —
> the base `tech-art` role and the inventory/load-check patterns apply unchanged.

For each imported 2D sprite, set deliberately (don't leave defaults unexamined):

- **Texture Type: Sprite (2D and UI)**; Sprite Mode **Single** (Multiple only
  for sheets to slice).
- **Pixels Per Unit** — pick one value project-wide (e.g. 100, or the sprite's
  native size if pieces are designed 1-cell = 1-unit) and record it in the
  inventory; mixed PPUs make board layout math inconsistent.
- **Filter Mode** — Bilinear for smooth casual art; Point for pixel art.
- **Compression** — default (Normal) is fine for a desktop test; note anything
  changed and why.
- For many small sprites of one family (the six pieces), a **Sprite Atlas**
  (`Assets/Art/Atlases/`) reduces draw calls — worthwhile once the set is
  stable; skip while art is churning.

## Placeholders (Unity mechanics)

Generate rough, unmistakably-temporary stand-ins; never imitate final art:

- **Editor script generation** (preferred — reproducible): an Editor method that
  draws solid-colour `Texture2D`s (e.g. 128×128 rounded squares in the six piece
  colours) and saves them as PNGs into `Assets/Art/Sprites/Pieces/`, then sets
  import settings. Lives in the Editor assembly; re-runnable.
- Or **runtime-primitive placeholders** (Unity built-in sprites / `SpriteRenderer`
  with a coloured quad) when even generated files are overkill.
- Tag each as `placeholder` in the inventory with what should replace it.

## The load-check (automated verification)

Every inventory asset gets covered by an **EditMode test** that proves it
resolves — the Tier-0 check for asset work:

```csharp
[Test]
public void All_piece_sprites_load()
{
    foreach (var name in new[] {"PieceRed","PieceBlue","PieceGreen",
                                 "PieceYellow","PiecePurple","PieceOrange"})
    {
        var sprite = AssetDatabase.LoadAssetAtPath<Sprite>(
            $"Assets/Art/Sprites/Pieces/{name}.png");
        Assert.IsNotNull(sprite, $"{name} missing or not imported as Sprite");
    }
}
```

(`AssetDatabase` is Editor-only — this lives in the EditMode test assembly.)
Add a case per inventory group; a renamed or mis-imported asset then fails
Tier 0 instead of appearing as a mystery pink/missing sprite at runtime.

## Swap-in (real assets replacing placeholders)

When the stakeholder drops real files into `design/assets/`: first check that
**Git LFS is active** for binary types (the `.gitattributes` template declares
the filters; `git-lfs` must be installed — if it isn't, route to provisioning
before committing heavy binaries into plain git history). Then copy in, apply the
same import settings, **keep the same file name/path** so no code or prefab
changes are needed (that's the point of the naming convention), flip the
inventory entry from `placeholder` to `provided`, and let the existing
load-check confirm it. The QA pass eyeballs the result in the next demo build.
