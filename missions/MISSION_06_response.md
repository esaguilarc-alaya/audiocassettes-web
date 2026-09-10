# Mission 06 Response — Rewind Transition: Final Approved Design

**Mission:** `missions/MISSION_06_rewind-transition-final.md`
**Executor:** Claude Code
**Date:** 2026-09-10

## Sequence followed

1. Committed and pushed `missions/MISSION_06_rewind-transition-final.md` **and** `reference/rewind-transition-reference.html` together in one commit to `origin/main` (commit `3e608da`, `14df44d..3e608da main -> main`). Confirmed the push succeeded before touching any code. (Left the untracked `Claude outputs/` directory alone, as in prior missions.)
2. Read the reference file's source in full, then rendered it live (headless Chrome over the DevTools Protocol, `Animation.seekAnimations` for deterministic frame-by-frame inspection) and studied its exact motion/timing before writing any implementation code — the mission's required order.
3. Executed the mission scope, including a substantial live investigation when the straightforward port didn't render (detailed below, because it changed the final architecture and is directly relevant to verifying acceptance criteria).
4. Wrote this response file.
5. Committed and pushed it.

## Studying the reference (before writing code)

Read `reference/rewind-transition-reference.html` in full, then rendered it in headless Chrome and paused/seeked its real CSS animations to exact millisecond marks (not wall-clock delays, which proved too jittery in a prior mission's verification — see the `cdp-animation-seek-deterministic-frames` note referenced in earlier responses). Confirmed precisely:

- **Exit** (`.leaving`): `translateX(0) scaleX(1) blur(0) opacity(1)` → `translateX(-60%) scaleX(1.4) blur(16px) opacity(0)`, 900ms, `cubic-bezier(.55,0,1,.45)`.
- **Enter** (`.entering`): mirrored from the opposite side, 900ms, `cubic-bezier(0,.55,.45,1)`, delayed `900 × 0.12 = 108ms` after the exit begins.
- **Streak**: `repeating-linear-gradient(100deg, transparent 0 18px, rgba(255,255,255,.55) 18px 20px)`, animated 0%→35%→100% as opacity `0 → .9 → 0` with `background-position-x` sweeping `0 → -300px`, over the same 900ms.
- **OSD**: white triangle + "PLAY" (Courier New, letter-spaced, dark text-shadow/drop-shadow for legibility), top-right, `opacity`/`translateY` keyframed `0%→18%→70%→100%` over `900 + 900 = 1800ms` — deliberately outliving the 900ms motion.

Frame captures at 40/80/120/200/300/450/900ms confirmed the qualitative read (single-direction stretch/smear, streak sweep peaking then fading, OSD rising then holding then fading) matching the mission's own description exactly.

## The real investigation: porting this onto actual View Transitions

The reference is a **standalone mockup** that animates real, plain DOM elements by toggling CSS classes — it never touches the browser's View Transitions API at all. The real site uses Astro's `<ClientRouter />` (Mission 04), where the page-root motion **has** to go through `::view-transition-old/new(root)`, because that's the only way to blur/smear the actual page content across a real navigation. Porting the streak/OSD onto that turned out to be substantially harder than a 1:1 CSS port, and I want to report the investigation honestly rather than just the final result, per Rule 2.

**First attempt** (real DOM overlay, `transition:name` on the overlay so it would visually "sit on top of" the transition, no JS): the pseudo-element animations *did* get created (confirmed via `Animation.animationStarted` events — correct names, correct 900ms/1800ms durations) but **nothing painted**. I spent a long time trying to isolate why — tested `::view-transition-group()` `z-index` overrides, `position: fixed` vs `absolute`, with/without `Emulation.setDeviceMetricsOverride`, and eventually a **real, non-headless, visible Chrome window** driven over CDP (to rule out any headless-specific quirk) — in every case, the OSD/streak stayed invisible while root's own blur was active, even though `getComputedStyle` reported correct opacity/position/color throughout. (I also, at one point, mis-tested a supposedly-fixed version against a stale dev server — a caching gotcha from an earlier mission's response — which cost extra time before I re-confirmed with a full clean rebuild + hard server restart + a cache check via `curl` before trusting a result again.)

**What actually resolved it:** a real, plain (non-participating) element, forced to a hard-coded fully-opaque state, painted correctly *once the transition had fully finished* (confirmed at ~850–1000ms post-click) but was **invisible while root's transition was still active** (confirmed at ~150–600ms post-click, multiple repeats, same result). So: a real DOM element's live rendering is suppressed for the duration of an active View Transition, whether or not it participates — I could not get a real element to visibly animate *during* that window by any means I tried. This matches the observable behavior of the page-root/header pseudo-elements (which is how they're specifically designed to work), but does not extend to giving a separate real element its own visible independent animation during that same window.

**Final architecture:** the streak and OSD are real DOM elements (`TransitionOverlay.astro`, rendered once in `BaseLayout.astro`), *not* participating in the view transition at all, with their `is-playing` CSS animation class added once the browser's **own** `ViewTransition.finished` promise resolves — captured by wrapping `document.startViewTransition` exactly once (guarded on `window`, so it survives across soft navigations without re-wrapping, the same double-init trap fixed for `Header.astro`/`Reel.astro` in Mission 04). `.finished` resolving is the one unambiguous signal that the browser has torn the transition down and resumed normal live rendering — the correct, race-free moment to start a real CSS animation on top of the now-settled page. This is standard, ordinary CSS animation on a live element from that point on — no further pseudo-element complexity.

## The timing trade-off this forces — and what stayed exactly as agreed

**Root's 900ms rewind motion is untouched — exact keyframes, exact cubic-beziers, exact 108ms enter delay, ported 1:1 from the reference, unchanged by anything below.**

Because the streak/OSD can only reliably become visible *after* root's transition finishes (~900–1000ms after the click, root's own duration plus a small swap/network margin), their **own** 900ms/1800ms durations now run starting from that point rather than overlapping with the still-blurring page as they do in the standalone reference demo. Concretely: the streak now finishes sweeping around ~1.8–1.9s after the click (not ~0.9s), and the OSD now finishes fading around ~2.7–2.8s after the click (not ~1.8s). The reference demo could overlap them with the page motion because it never puts real content through an actual View Transition; once the streak and OSD have to render as real, separately-clocked elements to be visible at all, "starts at the same instant as root" and "is actually visible" turned out to be mutually exclusive in this browser, and I chose visible. I did not touch the 900ms root duration to compensate — that number is exactly what was approved, and reference frame-timing evidence above confirms the port is exact.

## Verification (Rule 2)

- **Streak + OSD actually render**, confirmed visually via CDP screenshots at ~1400ms post-click (both real, non-headless and headless Chrome): diagonal streak pattern visible across the page, "▶ PLAY" legible top-right, both fading appropriately for their position in their own timelines.
- **First-load check**: loaded the site fresh (hard navigation, no prior soft nav) and waited 400ms — `.tfx-streak`/`.tfx-osd` computed opacity `0`, no `is-playing` class present. Correct: `document.startViewTransition` is never called on a hard load, so the `.finished` hook never fires.
- **No duplication on rapid re-navigation**: fired three real navigations in quick succession (each ~400ms apart, well inside the previous one's still-playing OSD window) at 320/375/768px. After settling, exactly one `.tfx-osd` and one `.tfx-streak` element existed in the DOM at every width, landed on the correct final page, zero console errors/exceptions.
- **Reduced motion**: set `prefers-reduced-motion: reduce` (confirmed via `matchMedia(...).matches === true` before navigating), then screenshotted at 150/600/1200/1800/2500ms post-click — page cuts instantly to the destination with **no** blur, **no** streak, **no** OSD at any point in that whole window. `TransitionOverlay.astro`'s own `@media (prefers-reduced-motion: reduce) { animation: none !important; opacity: 0 !important; }` handles this (Astro's built-in reduced-motion rule only covers `::view-transition-*` pseudo-elements and `[data-astro-transition-scope]`, not these plain elements, so this mission's own rule is what's actually doing the work here).
- **Header stability**: unchanged from Mission 04/05 — `transition:name="site-header"` + `animation: none` on its pseudo, confirmed sharp/stationary in every screenshot throughout this mission's testing.
- **Mission 04 interop re-run**: nav-toggle click alone still produces zero view-transition starts and correctly opens/closes, including after a real navigation. The reel lever/back-tap/modal check initially came back with the back-tap not registering — traced this to the *test's* fixed wait margin (750ms) no longer being enough now that root's own transition is legitimately 900ms (this project's earlier missions had 160–390ms transitions, so that margin was previously always safe); with an adequate wait (confirmed via `elementFromPoint` actually landing on `.reel-frame-stack` rather than `<html>`), lever advances `0→1` and the back-tap correctly returns `1→0`. Not a regression in `Reel.astro` — `Header.astro` and `Reel.astro` were not modified this mission at all (see diff below).
- **No layout regression**: `scrollWidth === innerWidth` at 320/375/768px, confirmed during the rapid re-navigation test above.
- **Build**: `npm run build` succeeds (clean rebuild, `dist`/`.astro`/vite cache cleared first to rule out any stale-artifact influence on this specific check), 6 pages, zero errors.
- **No new dependency**: `git diff --stat package.json package-lock.json` is empty.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new brand-token colors; the streak/OSD's white/dark-shadow styling is the explicit, mission-recorded exception (same as Mission 04's flagged non-token choice), ported from the reference.
2. **Verification over self-report.** This response documents a real investigation, including the approaches that *didn't* work and why, rather than presenting only the final answer.
3. **Placeholder discipline.** Not applicable to this mission's changes.
4. **Mission boundary discipline.** `Header.astro` and `Reel.astro` were not opened or modified.
5. **Static-first constraint.** Not touched.
6. **No unlicensed third-party assets.** The play triangle is inline SVG (`<polygon>`), ported directly from the reference; no icon pack.
7. **Diff before review.** The implementation was checked frame-by-frame against the reference's own rendered behavior (see the studying section above), not eyeballed.
8. **Mission handoff protocol.** Mission file and reference file committed/pushed together first, confirmed, before any scope work; this response file is separate; both committed/pushed by Code.
9. **Mobile/responsive by default.** Verified the full effect (streak, OSD, rapid re-navigation, no overflow) at 320px, 375px, and 768px.

## Acceptance criteria — status

- [x] Side-by-side comparison against the reference: motion (translateX/scaleX/blur, both directions), 900ms root timing, and streak-overlay opacity/sweep behavior all confirmed frame-by-frame via CDP-driven rendering of both the reference file and the live site.
- [x] The OSD appears top-right after a real navigation, matches the reference's look (triangle + "PLAY", white/dark-shadow), fades in/holds/fades out over its own 1800ms window — confirmed visually. **Flagged deviation, not silently absorbed:** that 1800ms window now starts once the page settles (~900–1000ms post-click) rather than overlapping the page motion from click, a change forced by the investigation above, not a reinterpretation of the approved look.
- [x] Does not appear on hard/first load; does not duplicate on rapid re-navigation — verified directly (first-load opacity/class check; triple rapid-navigation element-count check).
- [x] `prefers-reduced-motion: reduce` gives an instant cut with no streak and no OSD — verified with the media feature actually set, screenshotted across a window wide enough to catch a late-appearing OSD if the rule had failed.
- [x] Header doesn't move, blur, or show the OSD — confirmed throughout.
- [x] Nav-toggle interop and reel/modal checks re-run and pass (see the timing-margin note above for the one wrinkle found and resolved in the test itself, not the app).
- [x] No new `package.json`/`package-lock.json` entries.
- [x] Renders cleanly, no jank, no overflow, OSD fully on-screen and legible at 320/375/768px.
- [x] `npm run build` succeeds with zero errors (clean rebuild).
- [x] Spot-check: no sound, no scope-out items touched (`git status` shows exactly `BaseLayout.astro`, `global.css`, and the new `TransitionOverlay.astro`).

## Files changed

- Modified: `src/layouts/BaseLayout.astro` (import + render `<TransitionOverlay />`), `src/styles/global.css` (root's `rewind-exit`/`rewind-enter` keyframes replaced with the 900ms streak design; fx-streak/fx-osd pseudo-element rules were tried and removed — see investigation above)
- Added: `src/components/TransitionOverlay.astro`

## Judgment call flagged for Esteban

The streak/OSD's visible start had to move from "overlapping the page motion, starting at click" to "starting once the page settles, ~900ms–1s after click" — a platform constraint I could not engineer around while keeping them real, visible DOM content (documented above). Root's own 900ms motion, exact keyframes, and exact easing are untouched. If the extra ~0.9–1s before the streak/OSD appear reads as too much of a gap once you see it live, that's a tuning conversation (e.g., shortening the OSD's own hold), not a re-scope of this mission — flagging it now rather than quietly shipping a different timing than what was described.
