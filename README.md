# aurelia-glass-coins
Aurelia — an interactive field of translucent glass currency coins

## Face cross-fade

Each coin carries two extruded glyphs, one per side. They used to be swapped by
toggling `visible`, gated by a ±0.08 hysteresis band on the facing dot product:

```js
function $m(s, t = "front", e = .08) {
  return s > e ? "front" : s < -e ? "back" : t
}
```

Two problems. The swap was a hard cut — one frame the front glyph, the next the
back one. And the dead zone held the *previous* face until the dot product
cleared the far side, so a coin turning away kept drawing its front glyph out to
`acos(0.08)` = **94.6°** — about 5° past edge-on, with the camera already
looking at its back — and only then snapped. That was the flash.

Both glyphs now stay in the scene and cross-fade on a smoothstep of the same dot
product, crossing at exactly 90° where both are foreshortened to nothing:

| Rotation | front | back |
| --- | --- | --- |
| 0° | 1.00 | 0.00 |
| 84° | 0.83 | 0.17 |
| **90°** | **0.50** | **0.50** |
| 96° | 0.17 | 0.83 |
| 180° | 0.00 | 1.00 |

The fade spans 78°–102°. Peak rate of change is 5.9% per degree of rotation,
which at the configured spin is under 2% per frame at 60fps. `front + back`
sums to exactly 1 at every angle, so there is no luminance dip through the
crossover.

The hysteresis is gone. It is the right tool for a value chattering around a
threshold, but these coins spin monotonically — it never chattered, so the dead
zone only ever added lag.

### Implementation notes

`De`, the gold material, is a **single shared instance** across every rim and
both glyphs on all six coins, so per-glyph opacity needs its own clone
(`__abxCloneGlyphMat`). `__abxSyncGlyphMat` copies the control-panel-driven
values back onto the clones each frame, so the sliders and the material dropdown
keep working.

The helper names are deliberately long. This is a minified bundle whose own
identifiers are two characters, and `Gw`/`Gx`/`Gy`/`Gz` are all taken — `Gy` is
a three.js WebGL helper that runs during renderer setup. Colliding with it fails
silently and lethally: our declaration wins the hoist, three.js calls ours with a
`WebGL2RenderingContext`, and the scene dies before `De` is even assigned.

The swap site sits inside a comma expression, so it has to remain a single
expression — hence one `__abxFadeFaces(t, d)` call rather than inline statements.

### Still open

Transparency. The `transparency` slider is already at 0.94 and has almost no
headroom left. What keeps the coins reading solid is two values it never
touches: `ior` at **2.4** (diamond — strong Fresnel, heavy refraction) and
`thickness` at **2.9** (a long Beer–Lambert absorption path). Both are reachable
on the existing sliders; try ~1.5 and ~1.0, then trim `envMapIntensity` and
`clearcoat`. Tune here, hit **Download Preset**, ship the JSON.

