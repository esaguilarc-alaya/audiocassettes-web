# Mission 21 — more vertical breathing room around the Screen 1 carousel

## Context

Mission 20 shipped the rotating 3D cassette carousel and Esteban
confirmed it works ("mission done"), but flagged one visual issue from
a real device screenshot (mobile width, matching the `@media (max-width:
26.25rem)` breakpoint `.cst-carousel-stage`/`.cst-carousel` already use):
the carousel is cramped vertically — too little space above it (between
`.cst-shelf-sub` and the carousel's lid) and too little space below it
(between the carousel/turn-controls and the bottom of the visible card).
The lid in particular reads as almost touching the title area above it,
and the turn-buttons/face-dots row sits close to the card's bottom edge.

This is a spacing-only fix. Do not change the carousel's structure,
data, interaction, or the desktop/tablet layout beyond what's needed —
scope this to the vertical rhythm around `.cst-carousel-stage` on Screen
1, primarily (but not necessarily only) at the mobile breakpoint where
the screenshot was taken.

## Scope (in)

1. Increase the vertical gap between `.cst-shelf-sub` and
   `.cst-carousel-stage` (currently effectively 0 — the stage sits
   flush against the subtitle) enough that the carousel's lid
   (`.cst-carousel-lid`, which extends above the carousel box itself via
   its negative `top` offset) has clear breathing room and doesn't read
   as touching the text above it.
2. Increase the vertical gap between the bottom of the carousel
   (including its lid/box, not just its own `height`) and
   `.cst-carousel-controls` (turn buttons + face dots) if needed, and/or
   between `.cst-carousel-controls` and the bottom edge of the visible
   card — enough that the controls don't read as crammed against the
   card's bottom edge.
3. Apply this at the mobile breakpoint (`@media (max-width: 26.25rem)`,
   the one already scaling `.cst-carousel-stage`/`.cst-carousel`/
   `.cst-carousel-lid`) at minimum, since that's what the screenshot
   shows. Check the unscaled (tablet/desktop) layout too — if it has the
   same cramped feel, fix it there as well; if it already looks fine,
   leave it alone rather than adding unnecessary space.
4. A reasonable approach: add `margin-top` to `.cst-carousel-stage` (or
   `margin-bottom` to `.cst-shelf-sub`) for the top gap, and `margin-top`
   to `.cst-carousel-controls` and/or `padding-bottom`/`margin-bottom`
   on the shelf screen's own container for the bottom gap. Use judgment
   on exact values — this is a "make it visibly breathe" fix, not a
   pixel-exact spec — but verify the result with real measurements
   (bounding-box gaps in a headless browser, or screenshots at 320/375/768px
   compared before/after), not just by eyeballing the CSS values.
5. If adding vertical space causes the shelf screen to need to scroll
   where it didn't before, that's acceptable — `.cst-screen` already has
   `overflow-y: auto`. Just confirm scrolling still works cleanly and
   nothing gets clipped/hidden without a way to reach it.

## Scope (out)

- No changes to carousel structure, data, the 16-slot model, labels,
  colors, interaction (turn/drag/snap), or click-to-open wiring.
- No changes to Screens 2/3.
- No horizontal spacing/sizing changes — this is vertical rhythm only.
- Don't redesign the carousel's own dimensions (width/height/translateZ
  values) to "solve" this — solve it with the space around the carousel,
  not the carousel itself, unless you find the carousel's own height is
  the actual cause of the crowding (e.g. `.cst-carousel-stage`'s fixed
  `height` leaving no room), in which case say so plainly in the
  response file before adjusting it minimally.

## Acceptance criteria

- At 320px and 375px (the widths the screenshot represents), there is a
  clearly visible gap between the shelf subtitle text and the top of the
  carousel/lid, and a clearly visible gap between the turn-controls row
  and the bottom edge of the visible card — verified via real
  measurements (e.g. `getBoundingClientRect()` gaps), not assumed from
  the CSS values alone.
- 768px checked too; adjusted only if it shows the same problem.
- No regression to the carousel's own interaction (turn buttons, drag,
  snap, click-to-open a real cassette) or to Screens 2/3.
- `npm run build` succeeds with no new errors/warnings.

## Standing Engineering Control Rules (apply to every mission)

1. **Brand fidelity** — no new colors; this mission shouldn't need any.
2. **Verification over self-report** — your response file's claims will
   be independently re-verified against the real repo/build/browser
   behavior, not taken on faith.
3. **Placeholder discipline** — n/a to this mission, but don't introduce
   any.
4. **Mission boundary discipline** — spacing only; do not touch
   anything in Scope (out).
5. **Static-first constraint** — CSS-only change, no new dependency.
6. **No unlicensed third-party assets** — none needed.
7. **Diff before review** — self-review your own `git diff` before
   writing the response file.
8. **Mission handoff protocol** — commit and push this mission file
   FIRST, confirm the push succeeded, THEN execute the mission's scope,
   THEN write a separate `missions/MISSION_21_response.md` and
   commit+push that separately.
9. **Mobile/responsive by default** — verify at 320px, 375px, and 768px.
