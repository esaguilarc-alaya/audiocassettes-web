# Mission 05 Response — Rewind Transition Refinement

**Mission:** `missions/MISSION_05_rewind-transition-refinement.md`
**Executor:** Claude Code
**Date:** 2026-09-10

## Sequence followed

1. Committed and pushed `missions/MISSION_05_rewind-transition-refinement.md` to `origin/main` **before** touching any scope work (commit `ab4cc20`, `b0eb359..ab4cc20 main -> main`). Confirmed the push succeeded before starting implementation. (Left the untracked `Claude outputs/` directory alone, as in Mission 04 — not part of this mission.)
2. Executed the mission scope (detailed below).
3. Wrote this response file, separate from the mission file.
4. Committed and pushed this response file.

## What was changed

Replaced the `rewind-exit`/`rewind-enter` `@keyframes` and the `::view-transition-old(root)`/`::view-transition-new(root)` rules in `src/styles/global.css`, per the mission's spec:

- **Exit (old page):** `translateX(0) → translateX(-4%)`, `blur(0) → blur(9px)`, `opacity 1 → 0`, over **160ms**, `ease-in`. No `scaleX`, no grayscale, no brightness/contrast.
- **Enter (new page):** `translateX(4%) → translateX(0)`, `blur(9px) → blur(0)`, `brightness(0.9) → brightness(1)`, `opacity 0 → 1`, over **220ms**, `ease-out`, starting **100ms** after the exit begins (total 320ms — within the 300–450ms band, exit and enter overlapping in the middle as Mission 04's approach did).
- **Direction chosen:** old page exits to the **left**, new page enters from the **right** — read as "advancing forward through the tape" (every navigation moves the same way, regardless of which link was clicked), consistent with the mission's instruction to pick one direction and keep it for every navigation. Flagging this as the judgment call the mission anticipated.
- Only **one** filter adjustment beyond blur on the enter (`brightness`), expressed as a single 0%→100% dip-then-settle — no intermediate keyframe stops, no grayscale, no contrast. This directly replaces Mission 04's four-stop `0%/18%/30%/50%/100%` pulse.
- The header's `::view-transition-old(site-header)`/`::view-transition-new(site-header) { animation: none; }` exemption rule (added in Mission 04) was left completely untouched — not part of this mission's scope, and still needed to keep the header stationary.
- The file's top comment block was updated to describe the new design (still inside `global.css`, not a separate file) and explicitly notes the "looks like a shake effect" feedback that prompted this mission, for anyone reading the file later.

`BaseLayout.astro`, `Header.astro`, and `Reel.astro` were **not** opened or touched — `git diff --stat` for this mission shows exactly one file: `src/styles/global.css`, 24 insertions / 31 deletions, all inside the transition block.

## Verification (Rule 2 — real, deterministic frame inspection, not CSS-reading)

Mission 04's response used wall-clock `setTimeout` delays for its mid-transition screenshots, which turned out to have enough CDP round-trip jitter that repeating it here gave inconsistent frame timing (confirmed by comparing target vs. actual elapsed ms across a few trial runs). For this mission I used a more rigorous method: Chrome DevTools Protocol's `Animation` domain, which can pause the actual running view-transition animations and **seek them to an exact `currentTime` in milliseconds**, then screenshot — eliminating timing jitter entirely. This is the same class of technique the mission's own diagnosis used ("frame-by-frame render... via CDP screenshots"), made deterministic.

- **Single-direction motion, confirmed frame-by-frame** (desktop, 1200px, seeking the real `::view-transition-old/new(root)` animations):
  - `t=40ms` (25% through the 160ms exit): faint blur, slight leftward shift already visible.
  - `t=80ms` (50% through exit): clearly shifted **left** of neutral position (hero-visual box's left edge moved ~9px left; heading text moved ~10px left) with moderate blur and reduced opacity — one direction, one motion, no stretch.
  - `t=120ms`: both exit (near its end, low opacity) and enter (just started, low opacity) overlap — the expected busy midpoint of any cross-fade-style transition, not evidence of a shake; the *before* and *after* frames make the direction unambiguous.
  - `t=200ms` (enter is 45% through its own 220ms): las-cintas content clearly visible, shifted **right** of its final position and blurred, consistent with easing in from `translateX(4%)` toward `0`.
  - `t=320ms`: fully settled, sharp, correct final content — animation complete, matches the declared total duration exactly.
  - Read together: old content moves left and fades/blurs out; new content enters from the right and eases into place. One direction, both halves, exactly as specified — not a stretch (no `scaleX` in the new keyframes at all) and not a static blur-in-place.
- **Mobile width (320px), same seek method:** `t=80ms` shows the identical leftward slide+blur on the exit (header/hamburger stationary and sharp throughout), `t=320ms` shows a fully settled, non-overflowing `las-cintas` page. Confirms the effect isn't desktop-only.
- **At most one filter step on enter:** confirmed both by reading the new `rewind-enter` keyframes (only `0%`/`100%`, only `blur`+`brightness` beyond `translateX`/`opacity` — no grayscale, no contrast, no intermediate stops) and by the rendered frames above, which show a smooth blur/brightness settle rather than the four-stop pulse's rapid flashing.
- **Duration:** 100ms delay + 220ms enter duration = **320ms** total, inside the 300–450ms band; exit is 160ms, both within the mission's suggested sub-ranges.
- **Reduced motion re-verified against the new CSS** (not assumed unbroken): set `prefers-reduced-motion: reduce`, confirmed `matchMedia('(prefers-reduced-motion: reduce)').matches === true` immediately before clicking, then screenshotted 140ms into the navigation — destination page renders **fully sharp**, no blur, no shift, identical to the fully-settled state. Astro's own `viewtransitions.css` (`animation: none !important` under `@media (prefers-reduced-motion)`) still neutralizes the new keyframes exactly as it did the old ones — nothing in this mission's CSS change could touch that rule.
- **Header stays stationary:** visible and confirmed across every seek frame above (40/80/120/200/320ms, both widths) — the wordmark and nav never move or blur, matching the untouched `transition:name="site-header"` exemption.
- **Nav-toggle interop re-run** (same script as the Mission 04 response): clicking `.nav-toggle` alone still produces `vtCount: 0` (no transition started) and correctly opens the menu (`aria-expanded: "true"`); a real link click from the open mobile menu still navigates with `vtCount: 1`; the fresh page's toggle still opens correctly afterward. No regression.
- **Reel/modal-after-soft-navigation re-run** (same script/checks): navigated into `las-cintas` via a real link click, then lever click advanced frame `0 → 1`, a simulated left-half tap went back `1 → 0`, and a `.play-badge` click opened the modal with the correct placeholder `iframe` src. No regression.
- **Console/exception capture at 320/375/768px:** ran a full navigate → lever → play-badge lap at each width with `Runtime.exceptionThrown`/`console.error`/`console.warn` capture wired up. Zero errors or exceptions at any width.
- **No layout regression:** re-ran the full 18-combination `scrollWidth`/`innerWidth` overflow audit (six pages × 320/375/768px) from the Mission 03/04 responses — all 18 still exactly equal, no overflow introduced.
- **Build:** `npm run build` succeeds, 6 pages, zero errors.
- **No new dependency:** `git diff --stat package.json package-lock.json` is empty.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new colors — only `filter`/`transform`/`opacity` on existing rendered pixels, same as Mission 04.
2. **Verification over self-report.** Used a more rigorous method than Mission 04's own verification (deterministic `Animation.seekAnimations` instead of wall-clock delays) specifically because the first attempt at wall-clock timing for this mission produced inconsistent, hard-to-interpret results — switched methods rather than reporting a shaky finding.
3. **Placeholder discipline.** Not touched.
4. **Mission boundary discipline.** `BaseLayout.astro`, `Header.astro`, `Reel.astro` were not opened. The header exemption rule and reduced-motion handling were left byte-for-byte as Mission 04 shipped them.
5. **Static-first constraint.** Not touched.
6. **No unlicensed third-party assets.** None added.
7. **Diff before review.** `git diff --stat` confirmed exactly one file touched before writing this response.
8. **Mission handoff protocol.** Followed exactly: mission file committed/pushed first and confirmed; this response file is separate; both committed/pushed by Code.
9. **Mobile/responsive by default.** Verified the new effect and full interaction chain at 320px, 375px, and 768px, not just desktop.

## Acceptance criteria — status

- [x] Frame-by-frame rendering shows a single, consistent horizontal direction on both halves (left on exit, right-to-center on enter) — verified via deterministic `Animation` domain seeking, not assumption.
- [x] Enter's filter change happens in at most one smooth step (`brightness` only, `0%`→`100%`, no intermediate stops) — confirmed by both reading the keyframes and the rendered frames.
- [x] Total duration 320ms — within the 300–450ms band.
- [x] `prefers-reduced-motion: reduce` still gives an instant cut — re-verified with the media feature actually set and a mid-navigation screenshot, not assumed.
- [x] Header still doesn't move or blur — confirmed across every captured frame at both tested widths.
- [x] Nav-toggle interop and reel/modal-after-soft-navigation checks re-run with identical results to Mission 04 — no regression.
- [x] No new `package.json`/`package-lock.json` entries — diff empty.
- [x] Renders cleanly with no jank at 320px, 375px, 768px — verified via real interaction laps (zero console errors/exceptions) and the full 18-combination overflow audit (all clean).
- [x] `npm run build` succeeds with zero errors.
- [x] `git diff` touches only `src/styles/global.css` — confirmed via `git diff --stat`.

## Files changed

- Modified: `src/styles/global.css` (the `rewind-exit`/`rewind-enter` keyframes, the `::view-transition-old/new(root)` rules, and the section's descriptive comment — nothing else in the file)

## Judgment call flagged

Direction: old page exits left, new page enters from the right, on every navigation regardless of which link was clicked or which page precedes which in the nav order. This reads as "advancing forward" (the natural direction for a left-to-right-reading layout) rather than attempting to compute a "true" back/forward relationship between arbitrary page pairs, which the site's flat nav structure (five peer sections, no hierarchy) doesn't really have anyway.
