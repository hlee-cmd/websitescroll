# Custom Keyboard — Scroll-Linked Frame Sequence

A single-page, scroll-scrubbed image sequence (Apple "AirPods page" technique) built with
vanilla canvas + GSAP ScrollTrigger. 240 frames of an exploded-view keyboard teardown, preloaded
before playback, mapped 1:1 to scroll position.

## Run it locally
No build step. From this folder:
```
python3 -m http.server 8000
```
then open `http://localhost:8000`. (Opening `index.html` directly via `file://` will usually work
too, but some browsers restrict `file://` fetches — a local server is the reliable way to test.)

## Deploy to GitHub Pages
1. Push this whole folder (`index.html` + `frames/`) to a repo.
2. Repo → Settings → Pages → Deploy from branch → `main` / root.
3. Your live URL will be `https://<username>.github.io/<repo>/`.

## How it works
- **Preload**: all 240 `Image()` objects are requested up front. A loading overlay shows live
  percentage. The canvas paints frame 1 the instant it's individually ready, even before the rest
  finish, so there's visual feedback early rather than a blank screen the whole time.
- **Fallback while loading**: the section reserves its full scroll height immediately (so nothing
  jumps once loading finishes), and uses CSS `position: sticky` as a zero-JS stand-in for the pin
  until GSAP takes over — so if a visitor scrolls into the section mid-load, it already behaves
  sensibly instead of janking into place later.
- **Fallback for failed frames**: if any individual frame 404s or errors, playback holds on the
  nearest successfully-loaded frame instead of blanking the canvas or throwing.
- **Scroll → frame mapping**: GSAP timeline scrubs a `frame` value from `0` to `239` across a
  pinned scroll distance (`scrub: 0.6` smooths fast/jerky scroll input so playback doesn't stutter).
- **Sharpness**: canvas backing resolution is set from `devicePixelRatio` (capped at 2.5 to avoid
  memory blowups on very high-DPI phones), redrawn on resize.
- **Text beats**: four short copy blocks fade in/out at fixed points along the same scroll-driven
  timeline, timed to what's actually on screen at that point (assembled → keycaps lifting →
  switches/PCB exposed → full explode).

## Config
Everything you'd need to swap in a different frame set is in one block near the top of the
`<script>` tag in `index.html`:
```js
var FRAME_COUNT = 240;
var FRAME_PATH  = function(i){ ... };   // builds "frames/yourname-001.jpg" etc.
var BG_COLOR    = "#9a9a9a";            // match your frames' own background
```
Also adjust `.sequence-wrap { height: 500vh; }` in the CSS — taller = slower/more deliberate scrub.

## What's actually been tested vs. not
Being straight about this rather than just asserting it works:
- **Verified**: JS syntax, HTML structure, all 240 frame files present and correctly named/served,
  frame-path generation logic, canvas "cover" fit math, image-load/error-handling logic traced
  by hand for edge cases (a broken image no longer gets treated as "ready" by the fallback logic).
- **Not verified in this environment**: I don't have a headless browser available in this sandbox
  (network egress here is locked to a package-registry allowlist, no browser binaries or CDNs
  reachable), so I could not actually watch this scrub in a real browser before handing it to you.
  GSAP/ScrollTrigger CDN links are standard, current versions, but load from cdnjs at runtime —
  please do a real scroll-through on desktop and one mobile device once it's live before you
  consider this done. If the pin/scrub timing feels off, the two easiest knobs are
  `sequence-wrap` height (overall pace) and `scrub` value (lag/smoothing).

## Performance note
This build is 5MB across 240 JPGs, which is light — fine to preload fully up front the way this
does. If you swap in a much larger or higher-res sequence, full-preload will start to feel heavy;
at that point you'd want to preload only a low-res pass first, or lazy-load frames in windows
around the current scroll position, rather than blocking on everything at once.
