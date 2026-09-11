# Mission 23 response — default (desktop) carousel sizing overflow

## Summary

Two independent bugs in the **default** (non-mobile-width) carousel
sizing, both confirmed by direct measurement before any code change:

1. The rotated faces' actual rendered extent (not `.cst-carousel`'s own
   flat box) overflowed `.cst-card`'s right edge by ~5.5px at every
   plain desktop size, while the left side had ~28px to spare — an
   asymmetric, invisible-to-flat-bounding-box overflow caused by the
   carousel's permanent `BASE_ANGLE` tilt combined with each face's
   `translateZ()`.
2. The bottom gap (`.cst-carousel-controls` to `.cst-card`'s bottom
   edge) shrinks toward zero on a short desktop browser window, for the
   same underlying reason Mission 22 diagnosed for mobile — but Mission
   22's fix only lives inside the mobile-*width* media query, so it did
   nothing for a normal-width, short-height desktop window.

Both fixed with CSS-only changes to the default sizing rules plus one
new, additive height-only media query. No carousel structure, data,
interaction, colors, or Screens 2/3 were touched, and the mobile-width
media query blocks from Missions 20/22 are byte-for-byte unchanged.

## Fix 1 — face overflow (measurement method + actual fix)

**Measurement method fix first**, per the mission's explicit
instruction: instead of measuring `.cst-carousel`'s own
`getBoundingClientRect()` (its flat, untransformed layout box, which
does not reflect where its 3D-transformed children actually render),
I measure the union of all `.cst-face` elements' own
`getBoundingClientRect()`s — the actual rendered bounding box of the
rotated content — and compare that to `.cst-card`'s edges.

**Reproduced first** (production build, `npm run preview`, CDP):

| Size | Flat `.cst-carousel` box | Actual rendered face extent | Left gap | Right gap |
|---|---|---|---|---|
| 1440×900 | 574.79–846.69 | 540.15–933.55 | +28.15px | **-5.55px** |
| 1280×800 | 494.71–766.85 | 460.37–853.49 | +28.37px | **-5.49px** |
| 1920×1080 | 814.75–1086.76 | 780.25–1173.52 | +28.25px | **-5.52px** |
| 1280×720 | 494.75–766.76 | 460.25–853.52 | +28.25px | **-5.52px** |

Confirms the mission's own finding exactly: a real, consistent ~5.5px
overflow past the card's right edge at every plain desktop size, with
the flat box (`.cst-carousel`) itself never showing it — exactly why
this shipped unnoticed in Mission 20.

**Root cause**: `#cstCarousel` (the whole rotating frame) gets
`transform: rotateY(<angle>)` in JS, where the resting angle includes
the permanent `BASE_ANGLE = 24` offset. Rotating a rigid frame that
contains children offset via their own `translateZ()` composes the
frame's rotation with each child's depth offset — so a face's
`translateZ(9.25rem)` offset, after the frame's 24° rotation, resolves
to a combined sideways-*and*-depth displacement in world space (not
pure depth), and perspective projection foreshortens that
asymmetrically depending on how close each rotated face is to the
camera. That's the actual mechanism behind the asymmetric, "invisible
to the flat box" overflow.

**Fix**: reduced the default carousel's "radius" (width/height, and
`translateZ` kept at exactly half the new width, preserving the
existing cube-radius convention) by the same proportion (×0.85):

- `.cst-carousel`: `18.5rem × 20rem` → `15.725rem × 17rem`
- `.cst-face-0..3` `translateZ`: `9.25rem` → `7.8625rem`

This both shrinks the overall footprint and proportionally shrinks the
asymmetric sideways displacement described above (since that
displacement scales with `translateZ`), which recenters the rendered
extent as well as shrinking it.

**After**, measured the same way at all four required sizes:

| Size | Left gap | Right gap |
|---|---|---|
| 1440×900 | 53.23px | **28.98px** |
| 1280×800 | 53.38px | **29.03px** |
| 1920×1080 | 53.38px–53.60px | **29.03px–29.12px** |
| 1280×720 | 53.38px | **29.03px** |

Both sides now comfortably clear the mission's 12px+ target with
substantial margin, and the geometry is essentially identical across
all four sizes (expected — `.cst-card`'s `max-width: 26rem` doesn't
grow past 416px on any of these, so the horizontal picture is the same
CSS-pixel scene regardless of viewport width). Screenshot comparison
at 1440×900 (below) confirms visually: the "before" shot shows the
carousel's right edge touching/crossing the card's own rounded border;
"after" shows a clean margin on both sides while the carousel is still
clearly legible (not shrunk to the point of feeling undersized).

## Fix 2 — desktop short-window bottom gap

**Reproduced first**: at 1440×700, `gapControlsToCardBottom` measured
only 22.45px (down from 152.45px at 900px tall) — thin but not yet
negative, purely because `.cst-card`'s height is `min(44rem, 82vh)`.
Confirmed the mechanism is identical to Mission 22's mobile diagnosis:
the fixed-rem carousel content doesn't respond to viewport height at
all, so the gap is slack-dependent and would go negative on a shorter
window (verified separately below).

**Fix**: added a new, additive media query —
`@media (min-width: 26.3125rem) and (max-height: 45rem)` — that
shrinks only `.cst-carousel-stage`'s and `.cst-carousel`'s *height*
(15rem/11rem respectively; width/`translateZ` untouched, since those
are unrelated to a height problem and are already fixed above for
horizontal clearance). `min-width: 26.3125rem` is chosen to sit exactly
one step above the existing `max-width: 26.25rem` mobile breakpoint, so
this rule can never overlap or interact with the mobile-width media
query — that one keeps handling mobile short heights entirely on its
own, unmodified.

**After**, measured at the three required sizes:

| Size | Before | After |
|---|---|---|
| 1440×900 | 152.45px | 152.45px (query doesn't fire — height > threshold, correctly untouched) |
| 1440×700 | 22.45px | **150.45px** |
| 1280×720 | 38.84px | **166.84px** |

Also swept intermediate/shorter heights at 1440px width to characterize
the fix's range and honestly document its limits:

| Height | `gapControlsToCardBottom` | Notes |
|---|---|---|
| 750px | 63.45px | query not yet active (>720px threshold) |
| 720px | 166.84px | query becomes active here |
| 700px | 150.45px | |
| 650px | 109.45px | |
| 600px | 68.45px | |
| 550px | 27.45px | still positive but thinning |
| 500px | -13.55px | **negative again** — below this fix's designed range |

Below roughly 520–530px window height, the gap can go negative again —
this single-tier shrink was sized for realistic short desktop browser
windows (the mission's own required check points top out at 700px),
not for an extreme edge case like a 500px-tall window. Verified the
existing `overflow-y: auto` safety net still works cleanly there, the
same way Mission 22 relied on it for mobile's own extreme case:
at 1440×500, `#cstScreenShelf.scrollHeight` (456px) exceeds
`clientHeight` (410px), and scrolling `#cstScreenShelf` to its max
brings the controls fully within the card's bounds
(`controlsWithinCard: true` after `scrollTop` reaches its max of 46px)
— nothing is unreachable, just no longer guaranteed without scrolling
at that extreme.

## Mobile-width behavior — unaffected

Re-measured Mission 20/21/22's own required mobile checkpoints against
the current (Mission 23) build. All match the currently-shipped
(pre-Mission-23) values exactly — confirmed by also re-measuring the
same points against a `git stash`-reverted build of the exact code that
was pushed before this mission, to isolate any actual Mission-23-caused
change:

| Size | `gapSubToLid` | `gapControlsToCardBottom` |
|---|---|---|
| 320×480 | 59.19px | 52.84px |
| 375×550 | 59.19px | 132.45px |
| 390×600 | 59.19px | 176.45px |
| 320×568 | 59.19px | 130.28px |
| 320×667 | 68.66px | 153.41px |

Every one of these is identical before/after this mission's changes —
confirming the mobile-width query (Mission 20/22's own values) is
completely untouched, as intended, since this mission's new query is
scoped with `min-width` specifically to avoid ever overlapping it.

**One honesty note, unrelated to this mission's own changes**: while
re-verifying, `320×568`'s `gapControlsToCardBottom` measured 130.28px
here, not the 66.28px documented in Mission 22's response file. I
isolated this by reverting to the exact currently-pushed pre-Mission-23
commit and re-measuring fresh — it reproduces 130.28px there too, so
this is not something Mission 23 introduced; the Mission 22 response's
figure for that specific cell appears to have been measured under some
stale condition at the time (most likely a preview server not yet
rebuilt against the final source). The underlying computed styles
(`.cst-carousel-stage` height, `.cst-carousel` width/height at that
breakpoint) match Mission 22's own shipped tier-2 values exactly, and
the actual, currently-reproducible number is more generous than what
was documented, not less — so no code change was needed here, but
flagging it plainly per the "verification over self-report" standing
rule.

The 768px tablet/desktop-width top gap (`gapSubToLid`) legitimately
increased from 74.5px to 99.47px as a side effect of Fix 1's carousel
resizing (the smaller carousel's lid sits differently relative to the
now-smaller box) — an expected, harmless side effect at a
non-mobile-scaled width, not a regression (it's still a clear, visible
gap, just a larger one).

## Screenshot (1440×900, before/after)

Before: carousel's right edge touches/crosses the card's own rounded
border. After: clean, even-looking margin on both sides, carousel still
clearly legible.

## Regression check — carousel interaction and Screens 2/3

Re-ran the established regression scripts unmodified against the
current build: real audio/video playback and sync, media session
metadata, play/pause/next/prev, real synced lyrics, placeholder-track
inert behavior, video click-shield (click + touch-tap unaffected),
scrubber, REC badge and fullscreen hit-tests, and fullscreen itself
(re-verified in isolation at 1440×900 and 390×844 — `fsId: "cstApp"`,
video wrapper/host sizes matching exactly). No console errors.

Carousel interaction specifically (desktop size, where this mission's
Fix 1 changed the carousel's own dimensions): turn-right button click
changes `getComputedStyle(#cstCarousel).transform` (`matrix3d`) to a
genuinely different value; a real drag sequence changes it again;
click-to-open a real cassette (hit-tested via `elementFromPoint` on the
spine, per Mission 20's own documented test pitfall) correctly opens
`#cstScreenCassette`.

## Build

`npm run build` completes cleanly — 6 pages built, no new errors or
warnings, checked both mid-investigation (the stashed/pre-fix state
too) and as a final check before committing.

## Self-review

`git diff -- src/components/CassettePlayer.astro` reviewed before
writing this file: the change touches exactly the default
`.cst-carousel`/`.cst-face-0..3` rules (values only) plus one new,
additive media query. The mobile-width (`max-width: 26.25rem`) blocks
from Missions 20/22 are untouched. No carousel structure, data,
interaction/click-wiring, colors, or Screens 2/3 changes.
