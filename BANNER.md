# REST docs migration banner (v2)

Branch: `banner-2`. Replaces the v1 dual-running banner with the "REST docs"
stage from `docs-migration-banners-v2.html`.

## Heads-up: this repo is build output

`index.html` and `static/js/*.chunk.js` are generated artifacts (see the
`New release <date>` commits). **Both changes below are overwritten by the next
release build.** The banner should be ported into the source project that
produces this bundle; until then, reapply these edits after every release.

## What changed

1. `index.html` — the v1 banner blocks were removed and replaced with:
   - `<style id="op-banner-style">` in `<head>`
   - the `#op-banner` markup plus `<script id="op-banner-script">`, both placed
     immediately before `<div id="root">` so the offset variable is set before
     the React bundles run and ReDoc first paints.

   Every rule is scoped under `#op-banner`. This is not cosmetic: ReDoc injects
   styled-components into `<head>` at runtime, so an unscoped rule loses on
   cascade order. It matters most for `code`, which ReDoc also styles globally,
   and for the generic class names in the design file (`.inner`, `.copy`,
   `.cta`, `.close`) which would otherwise collide with ReDoc's own markup.

2. `static/js/main.ffea48a5.chunk.js` — unchanged from `banner`, and still
   required:

   ```
   scrollYOffset:function(){var e=document.querySelector(".MuiAppBar-root");return e?e.getBoundingClientRect().bottom:164}
   ```

   ReDoc uses `scrollYOffset` for both the sticky sidebar `top` and anchor
   scrolling, and it was hardcoded to `64`. Because this returns the AppBar's
   live bottom edge it needed no edit for v2 even though the banner grew from
   100px to 225px, and it goes back to 64 by itself when the banner is
   dismissed.

## Behaviour

- **Dismissible.** The close button hides the banner and records
  `op-banner-dismissed-v2` in `localStorage`, so it stays hidden on later
  visits. Dismissing restores the page to exactly its pre-banner layout
  (AppBar `top:0`, sidebar and content padding back to 64px). All
  `localStorage` access is wrapped in `try/catch` for private-mode browsers.
- **Height is measured, not assumed.** A `ResizeObserver` keeps
  `--op-banner-h` in sync with the real rendered height, which changes as the
  copy rewraps. The AppBar, the ReDoc sidebar and the content column all offset
  from that variable.
- **No slide-in on load.** The design file animates the banner open when you
  switch stages in the demo. Doing that on a real page load would push the whole
  document down over 0.55s on every navigation, so the banner renders already
  open. The saucer's entrance and float animations are kept, and both are
  disabled under `prefers-reduced-motion`.

## Verified

At 1440x900: banner 225.3px, background `#101623`, shell 1100px, grid columns
674.7/337.3 (2:1), h2 17px/600 white, body 15px `#c9ced9`, `code` 13.5px
`#f08a3c` with ReDoc's global code styling successfully overridden, CTA
`#cf2434` at 14.5px/600 with 5px radius and 13px 22px padding. AppBar bottom
289px, sidebar and content padding 289px. Clicking a sidebar entry lands the
heading 18.8px below the header stack.

## Known issue: height on small viewports

| Viewport | Banner height | Share of viewport |
|---|---|---|
| 1440 x 900 | 225px | 25% |
| 900 x 900 | 317px | 35% |
| 375 x 812 | 511px | **63%** |

At phone widths the banner occupies nearly two thirds of the screen before any
documentation is visible. The design file already contains the fix: the
"REST docs, collapsed" stage (`#stage-1b`), which shows the headline plus a
"Read more" toggle. Applying that below the 860px breakpoint would cut the
mobile height to roughly a quarter of the current figure. That is a design
decision, so it has not been applied.

A related fix *was* applied: once stacked, the headline spans the full width and
ran underneath the absolutely-positioned close button, so it now reserves 34px
of right padding below 860px.

## Unrelated pre-existing quirk

Below roughly 500px the header title "Openprovider documentation" wraps to two
lines and the MUI AppBar grows from 64px to 88px, but ReDoc's content padding
and the `headerSize` theme value are both hardcoded to 64. This predates the
banner and is not affected by it.
