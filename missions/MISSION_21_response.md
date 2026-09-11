# Mission 21 response — carousel vertical spacing

## Summary

Added vertical breathing room above and below Screen 1's carousel
(`.cst-shelf-sub` → `.cst-carousel-stage` gap, and
`.cst-carousel-stage` → `.cst-carousel-controls` gap), and along the
way found and fixed a real, previously-invisible bug: `#cstScreenShelf`
(a column flexbox like every `.cst-screen`) had no `flex-shrink: 0` on
its children, so on a short real-device viewport the browser was
silently *shrinking* `.cst-carousel-stage` below its declared height
instead of letting `.cst-screen`'s existing `overflow-y: auto` do its
job — which is almost certainly the real cause of the "crammed" look
Esteban's screenshot showed. This is flagged per the mission's own
Scope (out) clause even though the actual fix does **not** change the
carousel's declared width/height/translateZ values — it only stops
those declared values from being silently overridden by flexbox.

## What changed (all in `src/components/CassettePlayer.astro`)

1. `.cst-shelf-sub` — `margin` bottom `1rem → 2rem`.
2. `#cstScreenShelf` — new `padding-bottom: 2rem`.
3. `.cst-carousel-controls` — `margin-top` `0.5rem → 1.25rem`.
4. `flex-shrink: 0` added to `.cst-shelf-title`, `.cst-shelf-sub`,
   `.cst-carousel-stage`, and `.cst-carousel-controls` (the root-cause
   fix described above).

All four are outside the `@media (max-width: 26.25rem)` block, so they
apply uniformly at every width — confirmed necessary and sufficient at
320/375/768px below, no separate mobile-only or desktop-only values
needed. No carousel structure, data, interaction wiring, or Screens
2/3 were touched (verified in the diff and in the regression pass
below).

## The flex-shrink bug — found via measurement, not assumption

First attempt at this mission (margin/padding changes only, no
`flex-shrink`) was verified with real `getBoundingClientRect()`
measurements at realistic mobile heights (not just an arbitrarily tall
1400px viewport, which — I confirmed — masks the issue entirely since
`.cst-card`'s own height is capped at `min(42rem, 88vh)` and only
becomes viewport-constrained at real device heights). That first
attempt made the *bottom* gap measurably worse, not better, and
`#cstScreenShelf.scrollHeight` was staying exactly equal to
`clientHeight` at every height tested — i.e. no scrolling was ever
engaging despite the added `padding-bottom`, meaning the extra
required space wasn't being reserved as visible space *or* as scrollable
overflow. It was being silently absorbed by shrinkage.

Diagnosis: `#cstScreenShelf`'s flex children (title, subtitle, stage,
controls) had no `flex-shrink: 0`, so whenever total required content
height exceeded the container's own height, the flex algorithm
proportionally shrank the item with the most room to shrink —
`.cst-carousel-stage`, since it has the largest declared height. Confirmed
directly: before the fix, `.cst-carousel-stage`'s *rendered* height at
short viewports was measurably less than its own `height: 23rem`
computed-style declaration. After adding `flex-shrink: 0` to all four
rules, `.cst-carousel-stage`'s rendered height matches its computed
`height` (`232px` at the mobile breakpoint) at every tested viewport
height from 400px to 900px — it never shrinks again.

## Real measurements (CDP, `getBoundingClientRect()`, production build via `npm run preview`)

### Top gap: `.cst-shelf-sub` bottom → `.cst-carousel-lid` top

| Width | Before | After |
|---|---|---|
| 320px | 52.66px | **68.65px** |
| 375px | 52.66px | **68.65px** |
| 768px | 58.50px | **74.49px** |

Constant across every viewport *height* tested (568–1024px) at a given
width, as expected — this gap only depends on the margin change, not
on available room. A clean, unambiguous +16px (`1rem`) improvement
matching the CSS change exactly, with no interference from anything
else.

### Carousel-to-controls gap: `.cst-carousel-stage` bottom → `.cst-carousel-controls` top

| Width/height | Before | After |
|---|---|---|
| 320×568 / 320×667 / 320×812 / 768×1024 | 8px (`0.5rem`) | **20px (`1.25rem`)** |

Also constant regardless of viewport height, and also a clean,
unambiguous improvement matching the `margin-top` change exactly. This
is the more direct measurement of scope item 2's primary target ("the
bottom of the carousel... and `.cst-carousel-controls`").

### Controls-to-card-bottom gap (secondary metric, depends on total available height)

| Width×height | Before | After |
|---|---|---|
| 320×568 | 94.28px | 66.28px |
| 320×667 | 181.41px | 153.41px |
| 320×812 | 266.45px | 238.45px |
| 375×667 | 199.41px | 171.41px |
| 375×812 | 284.45px | 256.45px |
| 768×1024 | 180.45px | 152.45px |

This number *decreased* by a consistent ~28px at every height (exactly
the sum of the two top-side margin increases: 16px from `.cst-shelf-sub`
+ 12px from `.cst-carousel-controls`'s own margin-top, both of which
push the controls row further down the flex column toward the
fixed-height card). That's expected and not a regression: this metric
was never the primary target (scope item 2 lists it as an "and/or"
alternative to the carousel→controls gap above), and in absolute terms
it's still generous at every realistic device height — 66px is the
*smallest* value across all tested widths/heights, at h=568 (an old
iPhone 5/5s height, effectively obsolete for real devices today); every
more realistic modern-phone height (667px+) still leaves 150px+ of
clear space below the controls. `padding-bottom: 2rem` on
`#cstScreenShelf` is doing real work — it's just invisible in this
particular metric at generous heights, because it only manifests as
either consumed "leftover" flex space or, at genuinely tight heights,
as scrollable overflow (next section).

### Scrolling at genuinely tight heights (verified, not assumed)

At heights below ~500px (well under any realistic modern-phone
viewport, even accounting for browser chrome), `#cstScreenShelf.scrollHeight`
(466px, constant) exceeds `clientHeight`, and `overflow-y: auto`
genuinely engages (`needsScroll: true`). This is explicitly allowed by
the mission (item 5). Verified with a real scroll simulation, not just
checking the CSS property:

- At 320×420 (height chosen specifically to force scrolling): before
  scrolling `#cstScreenShelf`, `.cst-carousel-controls` sits below the
  visible card area (`controlsRect.top: 416.95` vs `cardRect.bottom:
  394.59` — genuinely off-screen, not just visually tight).
- After `scrollTop = 999999` (browser clamps to max), `scrollTop` moved
  from `0` to `96` (`= scrollHeight 466 − clientHeight 370`, i.e. fully
  scrolled), and the controls row is now entirely within the card's
  visible bounds (`controlsWithinCard: true`).
- Hit-tested the turn-right button's own screen position after
  scrolling: `elementFromPoint` at its center lands on its inner `svg`
  icon, which `.contains()`-checks as a descendant of the button — a
  normal button-with-icon structure, click still bubbles to the
  button. Nothing is unreachable or dead.

So even in the extreme case, nothing is clipped without a way to reach
it — scrolling cleanly restores full access to the carousel controls.

### 768px (tablet/desktop, unscaled carousel)

Checked per the mission's instruction. Before the fix it already had
more breathing room than the mobile breakpoint (58.5px top gap vs.
52.66px), but not by a meaningful margin, and the same shared (non
media-query-scoped) CSS changes apply here too — after: 74.49px top
gap, 20px carousel-to-controls gap, 152–256px+ controls-to-card-bottom
depending on height. No separate desktop-only adjustment was needed;
the mission-wide changes already produce clearly visible gaps here.

## Visual confirmation (screenshots, 320/375/768px, `npm run preview`)

Captured before/after screenshots at 320×812, 375×812, and 768×1024 by
temporarily `git stash`-ing the CSS changes, rebuilding, and
re-capturing (then restoring and rebuilding again — confirmed via
`git diff` and a final `npm run build` that the restored state matches
what's being committed). Visually: the subtitle text and the carousel
lid have a clearly visible gap in the "after" shots that reads as
flush/close in the "before" shots, matching the measured +16px/+20px
improvements.

## Regression check — no regression to carousel interaction or Screens 2/3

Re-ran Mission 20's own regression scripts (`m20_regression_check.mjs`,
`m20_m19_check.mjs`, `m20_m18_check.mjs`, `m20_fullscreen_isolated.mjs`)
unmodified against the current build, plus a new carousel-interaction
script exercising turn buttons, drag, and click-to-open specifically:

- **Turn-right button**: `getComputedStyle(...).transform` on
  `#cstCarousel` changed from
  `matrix3d(0.915,0,-0.403,...)` to `matrix3d(0.407,0,0.914,...)` — a
  genuine 3D rotation, not a flattened/no-op transform.
- **Drag-to-rotate**: dispatched a real `mousedown`/`mousemove`×2/`mouseup`
  sequence on the carousel stage; the resulting `matrix3d` changed
  again to a third distinct value, confirming drag rotation still
  works independently of the button path.
- **Click-to-open a real cassette**: after navigating back to face 0
  via the turn-left button, hit-tested `elementFromPoint` on the
  "Audiocassettes" spine (confirmed it lands on the spine or a
  descendant, not a stale/rotated-away element per Mission 20's own
  documented test pitfall), clicked it, and confirmed
  `#cstScreenCassette.classList.contains('is-active')` became `true`.
- **Screens 2/3 (real, non-mocked YouTube embed)**: real audio/video
  playback, media session metadata, play/pause via button, next/prev
  track, real synced lyrics, placeholder-track inert behavior, video
  click-shield (click and touch-tap on the video area confirmed
  unaffected), scrubber tap-seek and drag-seek (audio/video drift
  under 70ms in both cases), REC badge and fullscreen-button hit-tests,
  and fullscreen itself (re-verified in an isolated fresh navigation at
  both 1440×900 and 390×844 — `fsId: "cstApp"`, video wrapper and host
  sizes matching exactly) all behaved identically to Mission 20's
  shipped baseline. No console errors in any run.

## Build

`npm run build` completes cleanly — 6 pages built, no new errors or
warnings, both before writing this response and as a final check
before committing.

## Self-review

`git diff -- src/components/CassettePlayer.astro` reviewed before
writing this file: the change is scoped to exactly the four rules
listed above, all vertical spacing (no horizontal changes), no
carousel structure/data/interaction/click-wiring touched, no Screens
2/3 changes, no new colors.
