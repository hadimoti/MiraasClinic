# Component adaptation

The production site is intentionally a deployable static RTL HTML/CSS/JavaScript site. It does not contain a React, TypeScript, Tailwind, shadcn, or npm build pipeline, so the supplied React demos were adapted to the current architecture instead of adding an unused framework layer.

Implemented equivalents:

- `Reveal2`: the hero now contains an accessible before/after comparison slider with pointer, touch, keyboard, ARIA values, labels, and generated illustrative dental images.
- `HaloReel`: the portfolio sample photos now use a perspective reel with autoplay, drag/swipe, arrow controls, keyboard controls, and dots.
- `AnimatedBackground`: the fixed mobile bottom navigation uses an animated active background state and safe-area padding.

The hero images in `assets/generated/` are AI-generated illustrative examples. They must not be published as real patient outcomes. Replace them with approved, consented clinical assets if the clinic wants to present actual treatment results.
