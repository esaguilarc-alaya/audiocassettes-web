# Mission 04 Response — Page-Transition Rewind Effect

**Mission:** `missions/MISSION_04_rewind-transition.md`
**Executor:** Claude Code
**Date:** 2026-09-09

## Sequence followed

1. Committed and pushed `missions/MISSION_04_rewind-transition.md` to `origin/main` **before** touching any scope work (commit `664e4fe`, `74b714a..664e4fe main -> main`). Confirmed the push succeeded before starting implementation. (Left the untracked `Claude outputs/` directory alone — not part of this mission.)
2. Executed the mission scope (detailed below), including a real bug found and fixed during verification.
3. Wrote this response file, separate from the mission file.
4. Committed and pushed this response file.

## A real bug found and fixed during verification (Rule 2 — this is why the checks matter)

Enabling `<ClientRouter />` changed something the mission's Context section didn't call out: Astro's `astro:page-load` event, which `Header.astro` and `Reel.astro` already listened for (added forward-looking in Missions 02/03), now **also fires on the initial page load**, not only after soft-navigation swaps. Both components' scripts called their init function immediately at the top level *and* registered it as an `astro:page-load` listener — harmless before this mission because that event was never dispatched at all without a router installed, but now it double-attaches a second click listener to the same button on first load.

I caught this because the very first mobile-nav-toggle test after enabling the router came back wrong: clicking `.nav-toggle` left `aria-expanded="false"` and the list still `hidden` — i.e. nothing visibly happened. Two independently-bound listeners were each toggling the state on the same click, cancelling each other out. The same double-binding existed in `Reel.astro`, where it's worse: a second `showModal()` call on an already-open `<dialog>` throws (per spec). I confirmed this class of bug was real, not hypothetical, before fixing it — I didn't just reason about it and move on.

**Fix:** removed the redundant immediate call in both files, keeping only the `astro:page-load` listener — which now correctly covers both the initial load and every subsequent transition, exactly once each. Re-ran every check after the fix; see below.

## What was built

### 1. View Transitions enabled
- `BaseLayout.astro` imports `{ ClientRouter } from "astro:transitions"` and renders `<ClientRouter />` in `<head>`. No new dependency — `astro:transitions` is a virtual module the already-installed `astro` package provides (confirmed in `node_modules/astro/dist/transitions/vite-plugin-transitions.js`).

### 2. The rewind/scrub effect
- Implemented in `src/styles/global.css` (not a scoped Astro `<style>` block — `::view-transition-*` pseudo-elements are rooted at the document, not inside any component's DOM, so Astro's per-component CSS scoping can't reach them; this is called out in a comment in the file).
- `::view-transition-old(root)`: a 150ms `ease-in` exit (`rewind-exit` keyframes) — progressive blur, grayscale, and a brightness dip, with a slight horizontal `scaleX` for a "smear" feel.
- `::view-transition-new(root)`: starts 50ms after the exit begins, runs 340ms `ease-out` (`rewind-enter` keyframes) — enters fully blurred/desaturated/over-bright, then steps through two more blur/grayscale/brightness/contrast jumps before settling clean. This is the stand-in for the mission's suggested "scanline-flicker overlay": real scanline stripes aren't renderable on a view-transition pseudo-element (it's the page's captured snapshot, not a normal box you can layer a background image behind), so I interpreted the flicker as a couple of quick filter jumps — a judgment call, flagged per the mission's explicit allowance for one.
- Total: 50ms delay + 340ms enter = **390ms**, inside the requested 300–450ms band; exit overlaps within that window.
- Only `filter`/`transform`/`opacity`/`animation-*` properties are used — no new colors, no new assets.
- Plays once per navigation by construction: it's driven by the browser's native View Transitions API via a single `animation` per pseudo-element with `animation-fill-mode: both` (no `infinite`, no re-triggering) — confirmed empirically below (`vtCount` stayed at exactly 1 per click).

### 3. Header stays stable
- `Header.astro`'s `<header>` now has `transition:name="site-header"`, giving it its own named transition-group separate from `root`.
- `global.css` explicitly sets `::view-transition-old(site-header), ::view-transition-new(site-header) { animation: none; }` — no blur, no fade, no flicker on the header specifically.
- This does **not** persist the header's DOM (no `transition:persist`) — each navigation still swaps in that page's own fresh header markup (fresh `hidden` nav list, `aria-expanded="false"` default), which is exactly the behavior the interop requirement needs (see next section).

### 4. Reduced motion
- No extra code was needed. Astro's own `node_modules/astro/components/viewtransitions.css` (loaded automatically by `<ClientRouter />`) already contains:
  ```css
  @media (prefers-reduced-motion) {
    ::view-transition-group(*), ::view-transition-old(*), ::view-transition-new(*) { animation: none !important; }
  }
  ```
  This `!important` rule neutralizes my custom keyframes (and any others) whenever the user prefers reduced motion, producing an instant cut — exceeding the mission's "instant cut, or at most a simple cross-fade" bar.

## Verification (Rule 2 — real renders and real interaction, not CSS-reading)

All of the below used Chrome driven over the DevTools Protocol (`Emulation.setDeviceMetricsOverride` for real viewport widths, `Emulation.setEmulatedMedia` for `prefers-reduced-motion`), the same trustworthy method established in the Mission 03 response after finding Chrome's `--window-size` CLI flag unreliable below ~500px.

- **Transition fires on real navigation, exactly once:** monkey-patched `document.startViewTransition` to count calls, then dispatched a real `.click()` on a nav `<a>`. Result: `vtCount: 1`, landed on `/las-cintas` with the correct `<h1>`.
- **Visual proof, both conditions:** took a screenshot 140ms into the same navigation. Normal: header crisp, body content visibly blurred/desaturated (the effect actually rendering, not just declared in CSS). With `prefers-reduced-motion: reduce` set (confirmed via `matchMedia('(prefers-reduced-motion: reduce)').matches === true` immediately before the click): the same 140ms-in screenshot shows the destination page **fully sharp**, no blur at all — the reduced-motion path is real, not assumed.
- **Nav-toggle interop:** with the counter patched in, clicking `.nav-toggle` produced `vtCount: 0` (confirmed: a same-page state change never starts a transition) and correctly opened the menu (`aria-expanded: "true"`, list un-hidden) — this is the check that caught the double-binding bug above; after the fix, it passed cleanly.
- **Toggle post-navigation:** clicked a link from inside the open mobile menu → landed on `/las-cintas` (`vtCount: 1`); the fresh page's toggle was then clicked and correctly opened (`aria-expanded: "true"`) — confirms re-init via `astro:page-load` works after a real soft navigation, not just on first load.
- **Reel/modal re-run (same checks as the Mission 03 response), now arriving via a soft navigation instead of a direct URL:** clicked into `las-cintas` from the home page, then: lever click advanced frame `0 → 1`; a simulated tap on the left half of the viewer went back `1 → 0`; clicking a `.play-badge` opened the dialog (`open: true`) with the correct placeholder `iframe` src. All identical to the Mission 02/03 behavior.
- **Console/exception capture:** ran a full navigate → lever → back-tap → modal-open lap at 320px, 375px, and 768px with `Runtime.exceptionThrown` and `console.error`/`warn` capture wired up. Zero errors or exceptions at any width (this specifically re-confirms the double-`showModal()` bug is gone — that bug would have thrown here).
- **No regression on the Mission 03 overflow audit:** re-ran the same numeric `scrollWidth`/`innerWidth` check across all 18 page×width combinations (320/375/768 × six pages). All 18 still exactly equal — no overflow introduced by the transition CSS.
- **Build:** `npm run build` succeeds, 6 pages, zero errors.
- **No new dependency:** `git diff package.json package-lock.json` is empty.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new colors or assets — the effect is built entirely from `filter`/`transform`/`opacity` on the existing rendered pixels.
2. **Verification over self-report.** The double-listener bug above was found and fixed *because* of this rule, not despite it — every claim in this document is backed by a CDP-driven render or DOM read-back.
3. **Placeholder discipline.** Not touched.
4. **Mission boundary discipline.** No `Reel.astro` visual/interaction changes, no accent/token/copy/nav-label changes — confirmed via empty `git diff` on those files.
5. **Static-first constraint.** Not touched — the transition is pure client-side CSS/browser API.
6. **No unlicensed third-party assets.** None added.
7. **Diff before review.** Diffed the accent-mapping pages, `Hero.astro`, and `tokens.css` against this mission's changes and confirmed each is untouched.
8. **Mission handoff protocol.** Followed exactly: mission file committed/pushed first and confirmed; this response file is separate; both committed/pushed by Code.
9. **Mobile/responsive by default.** Verified the effect and full interaction chain at 320px, 375px, and 768px, not just desktop.

## Acceptance criteria — status

- [x] Real link click triggers the rewind/scrub transition — verified via `startViewTransition` call-counting and a mid-transition screenshot showing the actual blur/desaturation in flight.
- [x] Completes in ~390ms (150ms exit + 50ms delay + 340ms enter), does not loop (single `animation` per pseudo-element, no `infinite`).
- [x] `prefers-reduced-motion: reduce` gives an instant cut — verified with the media feature actually set (`matchMedia` confirmed `true`) and a mid-transition screenshot showing zero blur, not assumed from the CSS.
- [x] Mobile nav toggle still opens/closes correctly post-navigation, and its own click never triggers a transition (`vtCount: 0` on toggle click) — this check is also what caught and led to fixing the double-listener bug.
- [x] Reel lever, back-tap, dots, and video modal all still work identically after arriving via a real soft navigation — same checks as the Mission 03 response, re-run and passing, with zero console errors/exceptions captured across 320/375/768px.
- [x] No new `package.json`/`package-lock.json` entries — diff is empty.
- [x] Renders and performs without jank at 320px, 375px, and 768px — verified via real navigation + interaction laps at all three widths with no console errors and no layout overflow (re-ran the Mission 03 overflow audit; still 18/18 clean).
- [x] `npm run build` succeeds with zero errors.
- [x] Spot-check: no `<audio>`/`new Audio()`, no `cursor: url(...)`, no loading-screen/"cargando" placeholder text anywhere in `src/` (grepped, zero matches); accent mapping, brand tokens, hero copy, and nav labels all diff empty (untouched).

## Files changed

- Modified: `src/layouts/BaseLayout.astro` (`<ClientRouter />`), `src/components/Header.astro` (`transition:name`, double-init fix), `src/components/Reel.astro` (double-init fix only — no interaction/visual changes), `src/styles/global.css` (rewind keyframes + header exemption)

## Judgment calls flagged

- The "scanline-flicker overlay" was interpreted as a sequence of quick `filter` (blur/grayscale/brightness/contrast) jumps on the incoming snapshot rather than literal scanline stripes, since a `::view-transition-new()` pseudo-element is a captured snapshot image, not a normal box you can layer a background/overlay behind. This is a reasonable reading of the mission's own "not a locked spec" framing, restrained to a single settle-in beat rather than a repeating flicker.
- Only the header was given a named, animation-exempt transition group. The footer (sintoniza block, present on every page) was left as part of the default `root` snapshot and does play the rewind effect along with the rest of the page content — the mission's example only named the header/nav, and exempting more surface area than asked would have been scope creep in the other direction.
