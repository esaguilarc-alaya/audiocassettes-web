# Mission 22 response — carousel bottom gap on short real-device viewports

## Summary

Mission 21's fix only ever redistributed *slack* that happened to exist
between the carousel's fixed-`rem` content height and `.cst-card`'s
viewport-height-driven height (`min(44rem, 82vh)`). On a genuinely
short viewport it doesn't create real room, which is exactly what the
mission's own investigation found and what I reproduced independently
before touching any code.

Fixed with a new **height-based** media query, `@media (max-width:
26.25rem) and (max-height: 37.5rem)`, that further shrinks the
carousel (stage/box/translateZ/lid) proportionally on short viewports —
the same pattern Mission 20 already established for narrow *widths* —
rather than adding more margin/padding (which the mission explicitly
rules out as ineffective) or relying on `overflow-y: auto` scrolling to
reach the controls.

## What changed

One new rule block added to `src/components/CassettePlayer.astro`,
directly after the existing `@media (max-width: 26.25rem)` block. No
other rule was touched — Mission 21's margin/padding values are
untouched, and nothing in Screens 2/3, carousel structure/data, or
interaction wiring was touched.

```css
@media (max-width: 26.25rem) and (max-height: 37.5rem) {
  .cst-carousel-stage { height: 10.5rem; }
  .cst-carousel { width: 7.6rem; height: 8.2rem; }
  .cst-face-0, .cst-face-1, .cst-face-2, .cst-face-3 {
    transform: rotateY(var(--cst-face-deg)) translateZ(3.8rem);
  }
  .cst-carousel-lid { width: 4.35rem; height: 1.25rem; top: -0.62rem; }
}
```

Values are a proportional scale-down (×0.724, same ratio the Mission 20
width breakpoint already used relative to the unscaled desktop size)
of the existing mobile-width carousel dimensions, applied only when the
viewport height is also short. The `37.5rem` (600px) height threshold
was picked empirically: it's comfortably above all three of the
mission's required short-height test points (480/550/600px) so the
fix applies at all of them, while every real modern-phone viewport
height I tested (667px+) never crosses it and keeps the Mission 20/21
carousel completely unchanged (verified below).

## Reproducing the bug first (before touching code)

Measured the pre-fix state at the mission's three required combos,
via CDP `getBoundingClientRect()` on a production build
(`npm run preview`):

| Width×Height | `gapControlsToCardBottom` (before) | `needsScroll` |
|---|---|---|
| 320×480 | **-11.16px** | `true` |
| 375×550 | 68.45px | `false` |
| 390×600 | 112.45px | `false` |

This confirms the mission's own finding exactly: at 320×480 the
controls' bottom edge sits *past* the card's own bottom edge (negative
gap), and `#cstScreenShelf` was already scrolling to compensate — the
"crammed against the bottom edge" look Esteban's real device showed.
The other two combos happened to have enough slack from Mission 21's
fix that they weren't broken, but the mission is right that this was
luck, not a guarantee, since it depends entirely on how `82vh`
computes on a given device.

Screenshot at 320×480 (`.cst-card` scrolled into view first, since the
card sits below other page content and CDP's screenshot only captures
the current viewport) confirms the same visually: the turn-buttons/dots
row is cut off at/below the card's visible bottom edge.

## After the fix — measured at all three required combos

| Width×Height | `gapControlsToCardBottom` (after) | `needsScroll` |
|---|---|---|
| 320×480 | **52.84px** | `false` |
| 375×550 | 132.45px | `false` |
| 390×600 | 176.45px | `false` |

All three are comfortably above the 12–20px target the mission asks
for — the worst case (320×480) went from -11.16px to +52.84px, and
scrolling is no longer needed at any of them (the mission's preferred
outcome over relying on scroll). Screenshot at 320×480 (same
scroll-into-view approach) confirms a clean, clearly visible gap below
the controls before the card's rounded bottom edge, replacing the
previous cut-off look.

## Top gap (Mission 21's fix) re-verified at the same short heights

| Width×Height | `gapSubToLid` |
|---|---|
| 320×480 | 59.18px |
| 375×550 | 59.19px |
| 390×600 | 59.19px |

Still clearly present and clearly visible at every short-height combo —
comfortably above the pre-Mission-21 baseline of 52.66px. It reads
slightly smaller here (59.18px) than at tall viewports (68.65px,
unchanged and confirmed below) because the carousel's own lid shrank
along with the rest of the carousel in this mission's new rule (a
smaller lid with a proportionally smaller negative `top` offset pops up
less far above the carousel box) — an expected, minor side effect of
shrinking the whole carousel as a unit, not a separate regression. It's
still an unambiguous, measured improvement over the original
pre-Mission-21 problem at every viewport tested, so Mission 21's top-gap
fix stays intact rather than being sacrificed for the bottom-gap fix.

## No regression at tall viewports (the new rule must not fire there)

| Width×Height | `gapSubToLid` | `gapStageToControls` | `gapControlsToCardBottom` | `stageHeight` |
|---|---|---|---|---|
| 320×667 | 68.65px | 20px | 153.41px | 232px |
| 320×812 | 68.66px | 20px | 238.45px | 232px |
| 375×812 | 68.66px | 20px | 256.45px | 232px |
| 768×1024 | 74.50px | 20px | 152.45px | 368px |

Identical to Mission 21's shipped numbers in every column — the new
`max-height: 37.5rem` (600px) query never triggers at any realistic
phone height (667px+) or on tablet/desktop widths, confirming the
carousel keeps its full Mission 20/21 size whenever there's genuinely
enough room, per the mission's item 5.

## Regression check — carousel interaction and Screens 2/3

Re-ran the established regression scripts from Missions 20/21
unmodified (`m20_regression_check.mjs`, `m20_m19_check.mjs`,
`m20_fullscreen_isolated.mjs`, and the Mission 21 carousel-interaction
script exercising turn/drag/click-to-open) against the current build —
all identical to the previously-shipped baseline: real audio/video
playback and sync, media session metadata, play/pause/next/prev, real
synced lyrics, placeholder-track inert behavior, video click-shield
(click + touch-tap unaffected), scrubber, REC badge and fullscreen
hit-tests, and fullscreen itself (re-verified in isolation at 1440×900
and 390×844, `fsId: "cstApp"`, video wrapper/host sizes matching
exactly). No console errors.

Additionally verified carousel interaction specifically **at the new
short-viewport carousel size** (320×480, where the new media query is
now active), since that's a size tier that had never been
interaction-tested before this mission:

- Turn-right button: visible, correctly sized (41.6×41.6px, matches
  `.cst-turn-btn`'s unscaled 2.6rem — buttons themselves aren't scaled
  by this mission, only the carousel/lid/faces), and clicking it
  changes `getComputedStyle(...).transform` on `#cstCarousel` (a real
  3D rotation, not a no-op).
- Turning back to face 0 via the turn-left button four times still
  produces further transform changes each time (rotation genuinely
  responds).
- Click-to-open a real cassette: hit-tested `elementFromPoint` on the
  "Audiocassettes" spine at this smaller size (confirmed it lands on
  the spine or a descendant), clicked it, and confirmed
  `#cstScreenCassette.classList.contains('is-active')` became `true`.

## Build

`npm run build` completes cleanly — 6 pages built, no new errors or
warnings, both mid-investigation (checked the pre-fix/stashed state
too, to get an honest baseline) and as a final check before committing.

## Self-review

`git diff -- src/components/CassettePlayer.astro` reviewed before
writing this file: the change is a single new, additive media-query
block. No existing rule (including Mission 21's) was modified or
removed, no carousel structure/data/interaction/click-wiring touched,
no Screens 2/3 changes, no new colors, and — per the mission's explicit
warning — no additional `margin`/`padding` rem values were added
anywhere; the fix is entirely a height-conditional dimension shrink.
