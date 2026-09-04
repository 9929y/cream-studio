# Product Notes

## Register

Product / creative tool.

## User

Yanice and other designers who want to generate visual material quickly in a local browser: portfolio assets, social posts, motion studies, or exportable graphic elements.

## Product purpose

Cream Studio is a single-file HTML workbench for generative visuals. Instead of making seven separate tools, it puts seven generators into one shared shell:

- `DOTS`
- `ARCS`
- `ORBS`
- `FIELD`
- `AURA`
- `SLICE`
- `FLOW`

They share deterministic seeds, named palettes, motion settings, audio response, and export behavior. Switching modes should feel like changing instruments inside the same studio, not opening a different product.

Success criteria:

- Same seed, same composition.
- Slider changes are visible immediately.
- Presets produce finished-looking output on first open.
- The app runs offline by opening `index.html`.
- Export does not trigger surprise downloads.

## Product decisions

### One shell, seven instruments

The important product move is the shared shell. A single inspector, shared palettes, shared motion vocabulary, and shared export path make the seven modes feel like one tool instead of a folder of demos.

### Canvas first

The canvas is the product. Controls are secondary and should stay quiet. The inspector uses restrained glass, compact grouping, and familiar controls so the artwork carries most of the visual weight.

### Determinism over novelty

This is a design tool, not a slot machine. Randomization is useful only when it can be recovered. Seeds make every generated piece reproducible and let users return to a previous direction.

### Offline by default

The tool is useful when it can be opened, shown, and exported without a service account, server, package install, or CDN request. The embedded export logic is part of the product value.

### Export is a deliberate action

Generating an export and downloading it are separate steps. This avoids browser permission loops and gives the user one final chance to choose the output.

## Interaction model

- Presets provide fast entry points.
- Palettes stay global across modes so visual direction survives exploration.
- Structural sliders rebuild cached geometry.
- Visual sliders redraw against cached geometry.
- Audio bands become reusable control signals instead of mode-specific special cases.
- Narrow screens use a drawer so the canvas keeps priority.

## Quality gates

- Run every mode with all presets.
- Confirm seed reproducibility.
- Confirm exported PNG/SVG/GIF opens outside the app.
- Confirm no network requests are required for the core tool.
- Confirm keyboard focus and reduced-motion behavior.
- Confirm the app does not trigger a download until the user explicitly asks.

## Public story

Cream Studio should be presented as a product/design/build artifact:

- Product: one workbench rather than seven disconnected demos.
- Design: a quiet control surface around expressive output.
- Implementation: deterministic generation, shared mode contract, offline export, and embedded GIF encoding.
- QA: reproducibility, performance, accessibility, and export behavior.
