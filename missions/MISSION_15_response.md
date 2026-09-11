# Mission 15 Response — Fullscreen Player: Fix Clipped Controls + Unresponsive Lyrics Close

**Mission:** `missions/MISSION_15_fullscreen-fix.md`
**Executor:** Claude Code
**Date:** 2026-09-10

## Sequence followed

1. Committed and pushed `missions/MISSION_15_fullscreen-fix.md` to `origin/main` (commit `6a80914`, `eed9692..6a80914 main -> main`). Confirmed the push succeeded before touching any code.
2. Executed the mission scope.
3. Wrote this response file.
4. Committed and pushed the scope work separately (commit `3642f88`, `6a80914..3642f88 main -> main`), and will commit/push this response file separately next.

## What was built

All changes are scoped to `src/components/CassettePlayer.astro` only.

### Fix 1: constrained fullscreen sizing (a genuine two-attempt fix — the first approach didn't work and is documented, not hidden)

**First attempt (verified wrong, not shipped):** I initially wrote `#cstApp:fullscreen { width: min(26rem, 100vw); height: min(44rem, 100vh); margin: auto; }` directly on the fullscreen target, on the theory that author normal-importance beats UA normal-importance regardless of specificity (the same cascade rule behind this project's established `[hidden]` fix from Missions 03/08). Before trusting that, I checked `getComputedStyle(app)` while the element was actually in fullscreen — and it stayed at exactly the UA-forced `width: 1440px; height: 900px; position: fixed;` at a 1440×900 test, meaning the rule had zero effect. Chrome's fullscreen UA stylesheet wins here regardless of specificity.

**Correct fix (shipped):** restructured so the actual `.requestFullscreen()` target (`#cstApp`) is a bare, unstyled wrapper, and moved all of the card's own sizing/background/shadow to a new `.cst-card` child (containing the three `.cst-screen`s, unchanged). `#cstApp:fullscreen` now only sets `display: flex; align-items: center; justify-content: center;` plus the letterbox background — turning the UA-forced full-viewport wrapper into a centering container for the untouched, intrinsically-sized card. This is the same pattern real letterboxed video players use (fullscreen the wrapper, size the content) and cannot be defeated by the UA maximizing the wrapper, because that's now exactly what's wanted. The card's own `height: min(44rem, 82vh)` needed no fullscreen-specific override — `vh` units automatically recompute against whatever the current viewport is, fullscreen or not.

The letterbox background reuses the *exact* `color-mix(in srgb, var(--brown-tape) 15%, black 85%)` expression already used for this same file's video-wrap gradient (Mission 14) — not a new color, per Rule 1. A `::backdrop` rule with the same color is kept as a defense-in-depth fallback.

### Fix 2: lyrics drawer no longer intercepts clicks when closed

Added `pointer-events: none` to `.cst-lyrics-drawer` (its default/closed state) and `pointer-events: auto` only on `.cst-lyrics-drawer.is-open` — the robust fix the mission itself asked for, independent of getting any transform/height calculation exactly right at a given viewport.

## Verification (Rule 2/7 — measured, not eyeballed, with real synthetic clicks)

All checks below used genuine CDP `Input.dispatchMouseEvent` synthetic clicks (a real user-gesture equivalent, not `element.click()`), and were run against a production build served via `npm run preview` (not `npm run dev`) — the dev server's `<astro-dev-toolbar>` overlay was found, during this mission's own testing, to sometimes intercept clicks meant for page elements, which is a dev-only artifact unrelated to real user experience and unrelated to this mission's fix; switching to a preview/production server eliminated that confound entirely.

- **Fullscreen sizing, the acceptance criterion's own explicit test**, at 1440×900: navigated shelf → cassette → player via real clicks, clicked `#cstFsBtn`, confirmed `document.fullscreenElement.id === "cstApp"`, then read `getBoundingClientRect()` on `#cstVideoWrap`, `.cst-track-info`, `.cst-scrub-wrap`, `.cst-controls`, and `#cstLyricsToggle` — every one of their `bottom` values (max: 646.4px) sat well within `window.innerHeight` (900px), with `document.body.scrollHeight` also confirmed unaffected. Before the fix, the same measurement showed `.cst-controls.bottom: 1146.8px` and `#cstLyricsToggle.bottom: 1188.4px` — both well past the 900px fold, exactly reproducing the reported bug.
- **Same check repeated** at 390×844 (max element bottom 618.2px), and at 320px/375px/768px widths at a generous test height (max element bottoms 852.4px/886.8px/896.4px, all within the 1400px test viewport) — all pass, confirming no width-dependent regression per Rule 9.
- **Lyrics toggle — real click, not just presence of a button**: with the drawer closed, `document.elementFromPoint()` at the toggle's own coordinates was checked *before* clicking to confirm the toggle itself (not the drawer) would receive the hit — confirmed `"toggle"` at every one of the 5 tested viewports, both outside and inside fullscreen. A real synthetic click then correctly opened the drawer (`is-open` added) every time.
- **Close button — real click**: with the drawer open, a real synthetic click on `#cstCloseDrawer` correctly removed `is-open` at every one of the 5 viewports, both outside and inside fullscreen — the mission's specific reproduction case (fullscreen + desktop-sized viewport) included.
- **Mission 14's full behavior suite re-run, unregressed**: shelf (4 cassettes, correct colors), opening a specific cassette (not always the first), the lado a/b toggle swapping tracklists, opening a specific track (not a fixed demo track), play/pause state toggle, lyrics content/line count/sample-text note, back navigation at both levels, and the absence of any real audio/video/Media Session — all identical to Mission 14's own verified results.
- **Scope discipline**: `git status --short` shows exactly one modified file, `src/components/CassettePlayer.astro`. No other page or component was opened.
- **No `!important` used anywhere** in the fix — confirmed via `grep`; the correct solution didn't need it (unlike my first, discarded attempt's underlying assumption).
- **Colors trace to existing tokens**: the new letterbox `color-mix(in srgb, var(--brown-tape) 15%, black 85%)` is a `grep`-confirmed exact duplicate of the expression already used for this file's video-wrap gradient — reused, not invented.
- **Build**: `npm run build` succeeds (clean rebuild), 6 pages, zero errors — run both before and after the corrected fix.
- **No new dependency**: `git diff --stat package.json package-lock.json` is empty.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** The letterbox color is a verified exact reuse of an existing `color-mix()` expression already in this file, not a new value.
2. **Verification over self-report.** My own first fix attempt was checked against real `getComputedStyle()` output while actually fullscreen, found not to work, and replaced — documented above rather than silently discarded or shipped unverified.
3. **Placeholder discipline.** N/A — no new sample data in this mission.
4. **Mission boundary discipline.** No changes to the shelf/cassette screens, tracklist, lado a/b logic, real audio/video, Media Session, cassette-box visuals, or lyrics content/sync — confirmed via the full Mission 14 regression re-run.
5. **Static-first constraint.** Both fixes are pure CSS plus the existing (unchanged) Fullscreen API call — no server/API/database dependency.
6. **No unlicensed third-party assets.** Nothing added.
7. **Diff before review.** Checked against every acceptance criterion individually with a corresponding CDP measurement, including the mission's specific reproduction case (fullscreen + 1440×900 + lyrics interaction).
8. **Mission handoff protocol.** Mission file committed/pushed first and confirmed (`6a80914`); scope work committed/pushed separately (`3642f88`); this response file is separate, committed/pushed last.
9. **Mobile/responsive by default.** Re-verified at 320/375/768px plus the mission's own 390×844 and 1440×900 requirements — no regression.

## Acceptance criteria — status

- [x] Real click on `#cstFsBtn` at 1440×900: `document.fullscreenElement` is `#cstApp`, and video/track-info/scrubber/controls/lyrics-toggle are all within `innerHeight`/`innerWidth` with no scrolling — verified via `getBoundingClientRect()`.
- [x] Same check passes at 390×844 and at 320/375/768px widths — no regression from Mission 14's mobile verification.
- [x] With the drawer closed, a real click on `#cstLyricsToggle` succeeds without interception, at both mobile and 1440×900 desktop — verified via `elementFromPoint()` hit-testing plus the actual state change.
- [x] With the drawer open, a real click on `#cstCloseDrawer` succeeds and `is-open` is removed — verified at both mobile and desktop viewports.
- [x] `npm run build` succeeds with zero errors; no new dependencies.
- [x] All colors in the letterbox/fullscreen styling trace to established brand tokens — reused an existing exact expression, no invented hex.
- [x] Diff scoped to `CassettePlayer.astro` only — confirmed via `git status`.

## Files changed

- Modified: `src/components/CassettePlayer.astro`

## Judgment calls flagged

- **Structural change beyond pure CSS**: the mission's root-cause analysis suggested an author `:fullscreen` CSS rule would be sufficient. Testing showed that specific approach doesn't work in this case (verified, not assumed), so I restructured the markup slightly (wrapping the three screens in a new `.cst-card` div, keeping `#cstApp`'s `id` and its role as the fullscreen target unchanged) rather than stopping at a non-working CSS-only fix. This is the minimal structural change needed to make the mission's actual goal (constrained, letterboxed fullscreen sizing) achievable at all — flagged here since it's a bigger diff than "just add a `:fullscreen` rule," but still confined entirely to this one file and doesn't touch any JS logic, IDs used by the script, or any other screen's behavior.
- **Tested against `npm run preview` instead of `npm run dev`**: found the dev-only `<astro-dev-toolbar>` overlay intercepting some test clicks during verification — switched to testing against a production-equivalent server to get a clean, real-world-accurate signal, and confirmed via `grep` that the toolbar is absent from the preview output.
