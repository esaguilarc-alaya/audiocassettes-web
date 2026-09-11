# Mission 22 — the carousel's bottom gap still vanishes on short (real-device) viewport heights

## Context

Mission 21 added `margin`/`padding` intended to create breathing room
above and below the Screen 1 carousel. Esteban confirmed the top gap is
now fine, but the bottom gap ("turn-buttons/dots row crammed against the
card's bottom edge") is unchanged on his real device.

I reproduced this independently and found the actual mechanism, which
Mission 21 didn't address:

- `.cst-card`'s height is `min(44rem, 82vh)` — driven by the *viewport's*
  height, not by its own content.
- `.cst-screen` (including `#cstScreenShelf`) is absolutely positioned
  with `inset: 0`, so it's always forced to exactly fill `.cst-card`'s
  height — it does not grow to fit its content.
- The carousel's own dimensions (`.cst-carousel-stage` height, etc.) are
  fixed `rem` values that do NOT scale with viewport height.
- The visible "gap" between the turn-controls and the card's bottom edge
  is therefore not a real, guaranteed margin — it's just whatever slack
  happens to be left over between the (fixed, rem-based) content height
  and the (viewport-height-dependent) card height. Mission 21's added
  margin/padding only shrinks that leftover slack; it doesn't guarantee
  a minimum gap.
- On a tall test viewport (e.g. 320×700) there's tons of slack (180px+),
  so Mission 21's change looked like it worked. On a viewport height
  more representative of a real phone's *usable* browser viewport (once
  the address bar/toolbar chrome is accounted for — e.g. 320×480 to
  390×600), the slack shrinks to nearly nothing (I measured as low as
  ~‑11px, i.e. slightly negative) regardless of Mission 21's fix,
  because that fix only ever redistributes fixed-size slack — it can't
  create room that doesn't exist.

**Confirmed by direct measurement and screenshot**, not assumed: at a
320×480 viewport, `getBoundingClientRect()` on `.cst-carousel-controls`
and `.cst-card` showed the controls' bottom edge at or past the card's
own bottom edge, and a screenshot showed the turn-buttons/dots visually
flush against the card's rounded bottom corner with no visible gap —
this is very likely what Esteban's actual phone is showing, since real
mobile browsers typically report a shorter usable viewport height than
a plain "phone screen size" (device pixel height) would suggest, due to
address-bar/toolbar chrome.

## Scope (in)

1. **Test at realistic short viewport HEIGHTS, not just widths.** In
   addition to the existing 320/375/768px width checks, test at heights
   that represent a real phone's *usable* browser viewport — at minimum
   320×480, 375×550, and 390×600 (narrow width paired with a short
   height) — since that combination is what actually reproduces this
   bug. Do not rely on a tall default headless-browser viewport height,
   which will hide this bug (as it did during Mission 21's own
   verification).
2. **Fix the actual mechanism**: guarantee a minimum visible gap between
   `.cst-carousel-controls` and the bottom of `#cstScreenShelf`/`.cst-card`
   even when the viewport is short, not just when there happens to be
   slack. Prefer a height-responsive approach — e.g. a
   `@media (max-height: ...)` rule (pick a threshold empirically, by
   testing, not guessing) that shrinks `.cst-carousel-stage`/
   `.cst-carousel`/`.cst-carousel-lid` further on short viewports,
   mirroring the pattern the existing `@media (max-width: 26.25rem)`
   rule already uses for narrow widths — rather than relying on
   `overflow-y: auto` scrolling to "solve" it (scrolling to see the turn
   buttons is a worse experience than showing a smaller carousel that
   fits).
3. Whatever threshold/values you choose, verify empirically that they
   produce a clearly positive, visible gap (aim for roughly 12–20px
   minimum, use judgment) between the controls and the card's bottom
   edge at each of the short-height combinations in item 1 — measured
   with `getBoundingClientRect()`, not eyeballed.
4. Re-verify the top gap (Mission 21's fix) is still intact at these
   same short heights — don't fix the bottom at the expense of the top.
5. Keep the existing tall-viewport behavior reasonable — don't make the
   carousel awkwardly tiny on a normal-height viewport just to satisfy
   the short-viewport case; the height-based media query should only
   kick in when the viewport is actually short.

## Scope (out)

- No changes to carousel structure, data, interaction, colors, or
  Screens 2/3 — same boundaries as Missions 20/21.
- Don't just add more `padding-bottom`/`margin` values in rem — that's
  exactly what Mission 21 tried and it doesn't fix a slack-dependent
  gap. The fix needs to either shrink the carousel on short viewports or
  otherwise guarantee real, non-slack-dependent space.

## Acceptance criteria

- At 320×480, 375×550, and 390×600 (paired width×height, not just
  width alone), `getBoundingClientRect()` shows a clearly positive gap
  (not just >0px — comfortably visible, roughly 12px+) between
  `.cst-carousel-controls`'s bottom edge and `.cst-card`'s bottom edge.
  Include a screenshot at at least one of these as evidence in the
  response file.
- The existing 320×(tall)/375×(tall)/768×(tall) width checks from
  Missions 20/21 still pass (no regression).
- The top gap (subtitle → carousel) fixed in Mission 21 is still intact
  at all tested combinations.
- No regression to carousel interaction or Screens 2/3.
- `npm run build` succeeds with no new errors/warnings.

## Standing Engineering Control Rules (apply to every mission)

1. **Brand fidelity** — no new colors needed.
2. **Verification over self-report** — your response file's claims will
   be independently re-verified, including at the specific short
   viewport heights this mission calls out — this is exactly the kind
   of claim ("it fixed the bottom gap") that turned out false in
   Mission 21 when only tested at a tall viewport height, so be
   rigorous here.
3. **Placeholder discipline** — n/a.
4. **Mission boundary discipline** — this is a spacing/layout fix at
   short viewport heights only; do not touch anything else.
5. **Static-first constraint** — CSS-only change.
6. **No unlicensed third-party assets** — none needed.
7. **Diff before review** — self-review your own `git diff` before
   writing the response file.
8. **Mission handoff protocol** — commit and push this mission file
   FIRST, confirm the push succeeded, THEN execute the mission's scope,
   THEN write a separate `missions/MISSION_22_response.md` and
   commit+push that separately.
9. **Mobile/responsive by default** — this mission specifically extends
   that requirement to viewport HEIGHT, not just width; test both
   dimensions together as described above.
