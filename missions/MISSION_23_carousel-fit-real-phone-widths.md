# Mission 23 — the default (desktop) carousel size overflows the card's own edges

## Context

Esteban is reviewing this on desktop, not mobile — so the mobile-only
`@media (max-width: 26.25rem)` shrink rules from Missions 20/22 never
apply here at all. The problem is in the **default, un-shrunk carousel
size** itself (the one every viewport wider than 420px CSS width uses,
including every normal desktop browser window).

I reproduced this independently. `.cst-card` has `max-width: 26rem`
(416px) and does not grow past that on desktop. `.cst-carousel` itself
is `18.5rem` (296px) wide — comfortably narrower than the card — but the
carousel is a 3D scene: the permanent base-angle offset
(`BASE_ANGLE = 24`) keeps a slice of the *adjacent* face visibly rotated
into view at all times, and that rotated face's actual rendered pixels
extend well beyond `.cst-carousel`'s own flat, untransformed
`getBoundingClientRect()` — which only reflects the box's static CSS
layout size, not what its 3D children render outside of it.

**Measured directly** (not assumed): at 1440×900, 1280×800, and
1920×1080, the visually rendered right edge of the front-and-adjacent
carousel faces sits about **5–6px past `.cst-card`'s own right edge** —
a real, small but confirmed overflow — while the left side has a real
~28px gap. This asymmetric overflow-on-one-side-only, invisible-to-flat-
bounding-box-checks pattern is exactly why this shipped in Mission 20
without being caught: whatever verification ran there almost certainly
measured `.cst-carousel`'s own flat box, not the rotated face elements'
actual rendered extent.

Separately, the **bottom** gap (`.cst-carousel-controls` to `.cst-card`'s
bottom edge) shrinks a lot on a shorter desktop browser window — at
1440×700 it measured only ~22px, down from ~150px at 900px tall — for
the same reason Mission 22 diagnosed for mobile (the card's height is
partly viewport-height-driven via `min(44rem, 82vh)`, while the
carousel's own size doesn't respond to that). Mission 22's fix only
touched a **mobile-width-scoped** media query, so it does nothing for a
narrower browser window on desktop where a visitor has simply made
their browser shorter (a smaller laptop screen, or a window not
maximized).

## Scope (in)

1. **Fix the default (non-mobile) carousel/face/stage sizing** so the
   rotated faces' actual visual footprint — not just `.cst-carousel`'s
   flat box — stays clearly inside `.cst-card`'s edges with a real,
   visible margin on both sides. Likely candidates: slightly reduce the
   default `.cst-carousel` width and/or the `translateZ()` values on
   `.cst-face-0..3`, and/or add horizontal padding to
   `.cst-carousel-stage`. Use your judgment on the exact values, but
   verify against the actual rendered geometry (see the measurement
   method below), not the flat parent box.
2. **Fix the measurement method**: whenever you verify side clearance,
   measure the ACTUAL rendered bounding box of the currently-visible
   rotated elements (`.cst-face` elements and/or their child `.cst-slot`/
   `.cst-spine` elements that are front-facing or partially visible via
   the base-angle offset) via `getBoundingClientRect()`, not
   `.cst-carousel`'s own untransformed box, which does not reflect 3D
   children rendered outside its own layout bounds.
3. **Guarantee the bottom gap on desktop at realistic browser window
   heights**, not just tall ones. Test at minimum 1440×900, 1440×700,
   and 1280×720 (all plain desktop widths, well above any mobile
   breakpoint) and confirm a comfortably positive gap between
   `.cst-carousel-controls` and `.cst-card`'s bottom edge at each. If the
   existing Mission 22 approach (shrinking the carousel at short
   viewport heights) needs to apply outside the mobile-width media query
   too — i.e. as a height-only condition, not width-AND-height — do
   that, rather than duplicating a separate desktop-only fix.
4. Re-verify Mission 20/21/22's own mobile-width fixes are unaffected by
   whatever change you make here (this mission's changes should be
   additive/corrective to the *default* sizing rules, not a rewrite of
   the mobile-specific media queries).

## Scope (out)

- No changes to carousel structure, data, interaction, colors, or
  Screens 2/3.
- Don't touch the mobile-width (`max-width: 26.25rem`) media query
  rules themselves unless your fix to the default rules requires it for
  consistency — they were not the reported problem this time.

## Acceptance criteria

- At 1440×900, 1280×800, 1920×1080, and 1280×720, the visually rendered
  extent of the carousel's rotated faces (measured on the actual
  rotated elements, not the flat `.cst-carousel` parent box) has a
  clearly positive gap (aim for roughly 12px+) from both the left and
  right edges of `.cst-card`.
- At 1440×900, 1440×700, and 1280×720, there is a comfortably positive
  gap between `.cst-carousel-controls` and `.cst-card`'s bottom edge.
- Mobile-width behavior (320/375px, and the short-height fix from
  Mission 22) is unchanged/still correct.
- Include at least one before/after screenshot at a plain desktop size
  (e.g. 1440×900) in the response file.
- No regression to carousel interaction (turn/drag/snap/click-to-open)
  or Screens 2/3.
- `npm run build` succeeds with no new errors/warnings.

## Standing Engineering Control Rules (apply to every mission)

1. **Brand fidelity** — no new colors needed.
2. **Verification over self-report** — this exact class of bug (visual
   3D overflow invisible to a flat bounding-box check) has now shipped
   twice without being caught. Verify against the actual rendered
   geometry of the rotated elements, and say plainly in the response
   file what you measured and how.
3. **Placeholder discipline** — n/a.
4. **Mission boundary discipline** — default/desktop sizing and
   measurement-method fix only.
5. **Static-first constraint** — CSS-only change.
6. **No unlicensed third-party assets** — none needed.
7. **Diff before review** — self-review your own `git diff` before
   writing the response file.
8. **Mission handoff protocol** — commit and push this mission file
   FIRST, confirm the push succeeded, THEN execute the mission's scope,
   THEN write a separate `missions/MISSION_23_response.md` and
   commit+push that separately.
9. **Mobile/responsive by default** — re-confirm mobile widths are
   unaffected, even though this mission's actual bug was desktop-only.
