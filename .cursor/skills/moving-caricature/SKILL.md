---
name: moving-caricature
description: >-
  Overlay transparent caricature PNGs on event scene art with gentle CSS motion
  (bob/sway). Use when adding moving caricatures to wedding event cards, Sangeet/
  Carnival/Phere sections, or when the user asks for animated couple caricatures,
  moving caricature overlays, or reusable caricature motion on illustrated scenes.
---

# Moving Caricature

Composite a **transparent caricature PNG** over an illustrated event scene, then animate it with subtle CSS motion. Prefer this over replacing the whole scene with a full-bleed photo when the asset has a transparent background.

## When to use

- User provides a caricature with transparent background (PNG)
- Event card already has an SVG/scene fallback (Haldi, Sangeet, etc.)
- User asks for “moving caricature”, “animated couple”, or similar

## Asset rules

1. Prefer **transparent PNG** (no solid backdrop).
2. Full figures: head-to-toe visible, generous headroom.
3. Register the filename (no extension) in `CONFIG.ART`, e.g. `'caricature_sangeet'`.
4. Path example: `assets/caricature_sangeet.png`.

## Markup pattern

Inside `.ev-art.scene.art`, keep the scene `fallback` SVG. Add a stage **after** the fallback and **before** `.fx`:

```html
<div class="caricature-stage">
  <img
    class="caricature-fig"
    data-art="assets/caricature_sangeet.png"
    alt="Caricature of the couple"
    width="600"
    height="800"
    decoding="async"
    loading="lazy"
    onload="this.closest('.art').classList.add('has-caricature')"
    onerror="this.closest('.caricature-stage')?.remove()"
  >
</div>
```

Notes:
- Do **not** use `has-img` for this overlay — that hides the SVG scene.
- Use `has-caricature` so the illustrated background stays visible.
- Show sparkle/bokeh `.fx` when `.has-caricature` is present (same as `.has-img`).
- Tall portrait figures: raise the scene `viewBox` height (e.g. `0 0 400 320`) and extend the floor fill so feet sit on the terrace, not in empty space.

## CSS pattern (canonical)

```css
.art>svg.fx{display:none;position:absolute;inset:0;width:100%;height:100%;pointer-events:none;z-index:3}
.art.has-img>svg.fx,.art.has-caricature>svg.fx{display:block}

.caricature-stage{
  position:absolute;inset:0;z-index:2;display:grid;place-items:end center;
  padding:0 6% 2%;pointer-events:none;opacity:0;transition:opacity .6s ease;
}
.art.has-caricature>.caricature-stage{opacity:1}
.caricature-fig{
  display:block;width:min(68%, 300px);height:auto;transform-origin:50% 100%;
  filter:drop-shadow(0 10px 18px rgba(8,6,24,.45));
  animation:cariLive 3.6s ease-in-out infinite alternate;
}
@keyframes cariLive{
  from{transform:translateY(5px) rotate(-1.1deg) scale(1)}
  to{transform:translateY(-7px) rotate(1.3deg) scale(1.02)}
}
```

Motion rules:
- Pivot from **feet** (`transform-origin: 50% 100%`).
- One keyframe combining translate + rotate + slight scale (avoid two animations fighting over `transform`).
- Keep motion soft; this site pauses off-screen scenes via `.scene:not(.live)`.
- Existing `prefers-reduced-motion` block already kills animations — do not add a second path unless needed.

## CONFIG wiring

This site loads `[data-art]` only when the basename is listed in `CONFIG.ART`:

```js
ART: ['Joyful Carnival Celebration Couple', 'caricature_sangeet']
```

If the name is missing, the loader removes the `<img>` and the stage should `onerror`-remove itself.

## Full-bleed / animated GIF variant (no transparency)

When the asset is a full scene photo, illustration, or **animated GIF** (opaque background):

1. Set `data-art` on `.art-img` (not `.caricature-stage`) and add the basename to `CONFIG.ART`.
2. Prefer taller `viewBox` matching the image (~4:3 / `0 0 400 320`).
3. Use `.art-img.top` (`object-position: center top`) so heads are not cropped.
4. Add `.soft-live` for a gentle ken-burns drift on still photos (skip on GIFs).
5. Do **not** stack CSS `cariLive` on a GIF — the file already carries motion.
6. Examples: Sangeet GIF; Baraat `Romantic Varmala Under Floral Mandap.png` + `.soft-live`.

## Baraat critters (elephant + horse over a photo)

When `has-img` hides the SVG fallback, any animals drawn *inside* the SVG disappear. Put them in an HTML overlay so they stay visible and interactive:

```html
<div class="baraat-critters">
  <img class="baraat-creature baraat-elephant" data-art="assets/baraat-elephant.webp" alt="…" tabindex="0" role="button">
  <img class="baraat-creature baraat-horse" data-art="assets/baraat-horse.webp" alt="…" tabindex="0" role="button">
</div>
```

- Register `baraat-elephant` and `baraat-horse` in `CONFIG.ART`.
- Continuous bob via `elephantLive` / `horseLive`; click/Enter triggers `.hop` (`critterHop`).
- Keep matching SVG `<g class="baraat-svg-bob">` images for the no-photo fallback.
- Show falling petals in `.fx` for extra motion over the photo.

## Checklist

- [ ] Asset basename(s) added to `CONFIG.ART`
- [ ] Full-bleed: `.art-img.top` (+ `.soft-live` for stills) **or** transparent overlay: `.caricature-stage`
- [ ] Critters (if any) live in HTML overlay, not only inside SVG fallback
- [ ] `.fx` petals/sparkles enabled under `.has-img` / `.has-caricature`
- [ ] Motion uses single-transform keyframes; reduced-motion respected
