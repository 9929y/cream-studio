# Cream Studio

Cream Studio is a single-file generative art workbench: seven visual generators in one quiet control surface, sharing the same seed, motion system, audio response, palette model, and export pipeline.

It is designed for fast visual exploration. Open one file, tune a piece, export the result, and move on.

## What it makes

| Mode | Output | Export |
|---|---|---|
| `DOTS` | grid-based dot fields with offset, jitter, falloff, and wave/radial deformation | SVG |
| `ARCS` | concentric arc systems with rotation, spread, and angular jitter | SVG |
| `ORBS` | orbiting spheres with depth, glow, and track lines | SVG |
| `FIELD` | contour terrain built from fBm noise and marching squares | SVG |
| `AURA` | layered radial gradient fields with distribution and blend controls | raster |
| `SLICE` | image-slice compositions with directional offsets and gaps | raster |
| `FLOW` | noise-field streamlines with tapered trails | SVG |

`AURA` and `SLICE` are raster-native effects, so SVG export is disabled for those modes.

## Using it

Open `index.html`.

There is no build step, server, account, or network dependency. The tool runs offline and keeps the current document in the browser between visits.

The interface has two areas:

- Canvas on the left: the artwork stays visually dominant.
- Inspector on the right: presets, palette, fill, mode parameters, motion, audio, export, and settings.

The inspector can be resized on desktop and becomes a bottom drawer on narrow screens.

## Export

Exports are generated as data URLs. Nothing is downloaded until the user explicitly clicks the generated download action.

| Format | Notes |
|---|---|
| PNG | current frame at canvas size |
| SVG | real vector geometry for vector-native modes |
| GIF | one full motion loop, global palette, standard LZW encoding |

The GIF encoder is embedded in the file, so export works offline.

## Implementation notes

- Deterministic layout: structural randomness comes from a seeded LCG, so the same seed recreates the same composition.
- Shared mode contract: each generator declares `groups`, `defaults`, `looks`, `build`, `draw`, optional `toSVG`, and count metadata.
- Responsive controls: structural parameters rebuild cached geometry; visual parameters redraw immediately.
- Audio response: bass, mid, and high bands drive size, rotation/flow, and jitter.
- Accessibility: text contrast is checked across themes, controls are keyboard reachable, focus states are visible, and reduced-motion users get flattened UI transitions.

## Design direction

The control surface uses a restrained glass interface so the canvas can stay loud. The visual system is intentionally neutral: the artwork carries color and motion, while the UI stays quiet, precise, and secondary.

## Lineage

The project was implemented from a generative-art design brief credited to bycoraldesign. The application architecture, seven-mode shell, deterministic control system, offline export behavior, and interaction refinements were built for this implementation.

Keep this note if the source brief or license requires public attribution.
