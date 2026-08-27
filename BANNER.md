# Dual-running notice banner

Branch: `banner`. Adds the Figma "Banner / Dual-running" notice to the top of every
documentation page.

## Heads-up: this repo is build output

`index.html` and `static/js/*.chunk.js` are generated artifacts (see the
`New release <date>` commits). **Both changes below are overwritten by the next
release build.** The banner should be ported into the source project that produces
this bundle; until then, reapply these two edits after every release.

## What changed

1. `index.html` — three additions, all marked with `op-banner`:
   - `<style id="op-banner-style">` in `<head>`: banner styles plus the offset
     overrides for the existing chrome.
   - the `#op-banner` markup, injected just before `<div id="root">`.
   - `<script id="op-banner-script">`: keeps the `--op-banner-h` CSS variable in
     sync with the banner's real rendered height (it grows when the copy wraps on
     narrow viewports), so the MUI AppBar, the ReDoc sidebar and the content
     column stay aligned.

2. `static/js/main.ffea48a5.chunk.js` — one literal changed:

   ```
   -  scrollYOffset:64
   +  scrollYOffset:function(){var e=document.querySelector(".MuiAppBar-root");return e?e.getBoundingClientRect().bottom:164}
   ```

   Why this is required: ReDoc uses `scrollYOffset` both for the sticky sidebar's
   `top` and for anchor scrolling. It was hardcoded to 64 (the AppBar height).
   With a 100px banner above it the real offset is 164, so clicking a sidebar
   entry landed the heading ~81px underneath the header. CSS cannot fix this — the
   value is used in ReDoc's scroll maths. ReDoc accepts a function and calls it on
   every scroll, so returning the AppBar's live bottom edge tracks any banner
   height (desktop 100px, mobile ~204px) with no hardcoded constant.

   Side effect: `main.ffea48a5.chunk.js.map` no longer matches the bundle. This
   only affects browser devtools.

## Still to confirm

- The CTA label wording — it was obscured in the Figma export. The date
  (`31-Dec-2026`) and the href (`https://developer.openprovider.com/`) are confirmed.
- Background `#101828`: the banner background was on the parent Figma frame and
  was not included in the exported `Content.svg`.
- Whether the banner should be dismissible. As drawn it is permanent, which costs
  ~25% of a 375x812 mobile viewport.
