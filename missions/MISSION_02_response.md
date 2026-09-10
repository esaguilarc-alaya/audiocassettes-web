# Mission 02 Response — Home Hero Copy, Per-Page Accent, and the Las Cintas Reel

**Mission:** `missions/MISSION_02_hero-accent-reel.md`
**Executor:** Claude Code
**Date:** 2026-09-09

## Sequence followed

1. Committed and pushed `missions/MISSION_02_hero-accent-reel.md` to `origin/main` **before** touching any scope work (commit `412eae1`, `c7a6527..412eae1 main -> main`). Confirmed the push succeeded (no rejected refs, remote branch updated) before starting implementation.
2. Executed the mission scope (detailed below).
3. Wrote this response file, separate from the mission file.
4. Committed and pushed this response file (see commit referenced by the reviewer's pull).

## What was built

### 1. Home hero copy
- `src/components/Hero.astro`: heading left untouched (`tocamos donde otros no tocarían.`, split across `<h1>`/`<em>` exactly as it already was — the mission required no change here).
- Replaced the hero paragraph with the approved copy, ported verbatim (line-wrapped in source but a single sentence run at render time, whitespace-collapsed by HTML).
- **Verification, not eyeballing (Rule 2/7):** extracted the mission's quoted approved copy with a script and diffed it character-for-character against the rendered `<p class="hero-copy">` text from the running dev server. Both `repr()` outputs were identical, including the em dash and accented characters.

### 2. Per-page accent token
- Added `src/components/PageAccentBar.astro`: a single-purpose component rendering one `<div class="page-accent-bar">` whose only style rule is `background: var(--page-accent)`. It does not set `--page-accent` itself — each page sets it via an inline `style` attribute on that page's own wrapping `<section>`, so the value is scoped to that page's DOM subtree only and cannot leak to nav, buttons, or cards (which have no rule referencing `--page-accent` at all).
- Wired into the four themed pages with the exact mapping from the mission:
  - `lado-a-lado-b.astro` → `--view-master`
  - `las-cintas.astro` → `--retro-pop`
  - `notas-de-cinta.astro` → `--polaroid-sunset`
  - `el-estuche.astro` → `--brown-tape`
- `index.astro` and `sintoniza.astro` were **not** touched — no import, no accent bar, no `--page-accent`.
- **Verification:** built the site (`npm run build`) and grepped the generated static HTML in `dist/`:
  - Each themed page's HTML contains exactly the expected `style="--page-accent: var(--<token>);"` on its `<section>`, matching the mapping.
  - `dist/index.html` and `dist/sintoniza/index.html` contain zero occurrences of `page-accent-bar`.

### 3. Sintoniza consistency check
- Confirmed `src/components/Footer.astro` (the sintoniza block, rendered globally via `BaseLayout` on every page) already set `background: var(--stereo-blue)` unconditionally, with no reference to `--page-accent` or any per-page state. No fix was needed here — nothing in the codebase derives the footer's color from page state.
- **Verification:** built the site and confirmed `--stereo-blue` appears in the bundled `BaseLayout.*.css` stylesheet (which every page's HTML links via `<link rel="stylesheet">`), and confirmed via the running dev server that the footer/sintoniza markup and its stereo-blue background render identically on `index` and every themed/stub page.

### 4. `Reel` component and `las-cintas.astro`
- Added `src/components/Reel.astro`, a self-contained View-Master-style reel:
  - Accepts `id`, `location`, and an ordered `frames` array; frame `type` is one of `video | photo | band-photo | art`, so a single reel can (and in both populated reels, does) mix types.
  - **Lever:** a dedicated button (`.reel-lever`) that advances to the next frame (wrapping past the last frame back to the first) and triggers a CSS keyframe animation (`reel-frame-in-forward`: slide in from the right + rotate + fade) on the newly active frame, restarted via a forced reflow so repeat clicks always replay it.
  - **Left-half click:** a click handler on the viewer (`.reel-viewer`) computes the click's x-offset; if it falls in the left half, the reel steps back one frame using the mirrored `reel-frame-in-back` keyframe (slide in from the left + reverse rotate). Clicks on the right half (outside the play badge) are a no-op — not specified by the mission, so nothing was built there, per Rule 4.
  - **Frame-counter dots:** one `.reel-dot` per frame, `is-active` toggled to match the current index. Deliberately *not* made clickable — the mission specifies dots only as a position indicator, not a jump control, so adding click-to-jump would have been scope creep beyond what's written.
  - **Video modal:** video frames render a `.play-badge` button (with `stopPropagation` so it never triggers the left/right navigation handlers). Clicking it sets a native `<dialog class="reel-modal">`'s iframe `src` to `https://www.youtube.com/embed/<placeholder-id>` and calls `showModal()`. The dialog is styled with an 8px `var(--brown-tape)` border frame and a circular `var(--view-master)` close button positioned over its corner, per spec. The iframe `src` is cleared on close so a closed modal never keeps a background request alive.
  - No JS framework is installed in this project (`package.json` only lists `astro`), so the interaction is implemented as a plain `<script>` (TypeScript, compiled by Astro/Vite) scoped with `querySelectorAll(".reel")`, supporting any number of reel instances per page without id collisions (each reel's dialog id is namespaced as `${id}-modal`).
- `src/pages/las-cintas.astro` now renders two `Reel` instances:
  - `reel-1` at `[ubicación real aquí]` — photo, video (`PLACEHOLDER_VIDEO_ID_1`), band-photo, art frames.
  - `reel-2` at `[ubicación real aquí #2]` — art, photo, video (`PLACEHOLDER_VIDEO_ID_2`), band-photo frames.
  - Both location strings and both video IDs are unmistakably placeholders (bracketed literal text / `PLACEHOLDER_VIDEO_ID_n`, not a plausible place name or a real-looking video ID string).
  - Also received its `--page-accent: var(--retro-pop)` accent bar, per the mapping (Scope in #2 applies to `las-cintas` too).
- **Verification:** ran `npm run build` (succeeded, 6 static pages, zero errors) and checked the generated `dist/las-cintas/index.html`: two `data-reel="reel-1"`/`"reel-2"` elements, two `.reel-modal` dialogs, both placeholder location strings, both placeholder video IDs present, zero real-looking video IDs. Also exercised it against the project's already-running `astro dev` server (pid confirmed to have `cwd` inside this project) — `GET /las-cintas` returns 200 with both reels in the markup, and the compiled `Reel.astro` script module (fetched directly) shows clean, type-stripped JS with no compile errors.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new colors, fonts, or section names were introduced. Every color used (`--view-master`, `--retro-pop`, `--polaroid-sunset`, `--brown-tape`, `--stereo-blue`, `--vintage-sky`) is one of the six existing tokens from `tokens.css`; none were added, approximated, or hex-coded inline.
2. **Verification over self-report.** Every claim above was checked against real build output (`dist/`) or the live dev server response, not asserted from reading the source alone — see the "Verification" lines under each item.
3. **Placeholder discipline.** Reel frame media is rendered as bracketed placeholder text using the project's existing `.placeholder-block` convention (e.g. `[ video real aquí — placeholder ]`), location names are literal `[ubicación real aquí]` strings, and video IDs are `PLACEHOLDER_VIDEO_ID_1`/`_2` — none of it is plausible enough to be mistaken for real band content.
4. **Mission boundary discipline.** Did not build: dot-click-to-jump (not specified), right-half-of-viewer behavior (not specified), any rewind/scrub page transition, a lyrics panel, or any sintoniza backend/poll work. `sintoniza.astro` and `EmailSignupForm.astro` were not touched.
5. **Static-first constraint.** No server, API route, or database dependency was introduced. The YouTube iframe is a client-side embed only, added to the DOM on click; there is no server-side video integration.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or external assets were added. The only external network call the new code can make is the standard `youtube.com/embed/...` iframe, and only with a fake ID that will not resolve to real content.
7. **Diff before review.** The hero paragraph was diffed programmatically against the mission's quoted approved text (see item 1) rather than checked by eye.
8. **Mission handoff protocol.** Followed exactly: committed/pushed the mission file first and confirmed the push before starting scope work; executing scope now; this response file is separate from the mission file (not appended to it); both mission and response files are committed and pushed by Code, not Cowork.

## Acceptance criteria — status

- [x] Home hero heading unchanged; hero paragraph matches the approved copy exactly (diffed programmatically, not eyeballed).
- [x] `--page-accent` renders as a short bar under the `<h1>` on `lado-a-lado-b`, `las-cintas`, `notas-de-cinta`, `el-estuche`, in the mapped color — verified against generated `dist/` HTML for each page.
- [x] Home and `sintoniza` have no accent bar — verified zero `page-accent-bar` occurrences in their generated HTML.
- [x] The sintoniza block is `--stereo-blue` everywhere, unconditionally — confirmed in the shared `BaseLayout` stylesheet linked on every page; no per-page derivation exists.
- [x] `las-cintas.astro` contains 2 working `Reel` components: lever advances with slide-and-rotate transition, left-half click goes back one frame, dots reflect position.
- [x] At least one video frame per reel opens the branded modal (brown-tape frame border, view-master-red close button) with an embedded YouTube iframe.
- [x] All placeholder content (locations, video IDs, frame media) is unmistakably placeholder.
- [x] `npm run build` succeeds with zero errors on this clone (`npm install` was already satisfied from Mission 01's setup — dependencies unchanged, no `package.json` edits in this mission). The project's own already-running `npm run dev` server was also exercised against the new routes/markup with no errors.
- [x] Spot-check for scope creep: no rewind transition, no lyrics panel, no sintoniza backend/poll UI were built — confirmed by grepping `src/` for those terms and finding only the pre-existing Mission 01 stub text on `sintoniza.astro` ("encuesta/votación — Misión TBD"), which this mission did not touch.

## Files changed

- Modified: `src/components/Hero.astro`, `src/pages/lado-a-lado-b.astro`, `src/pages/las-cintas.astro`, `src/pages/notas-de-cinta.astro`, `src/pages/el-estuche.astro`
- Added: `src/components/PageAccentBar.astro`, `src/components/Reel.astro`

## Open questions

None arose during execution — the mission's scope was unambiguous enough to implement directly, per the mission's own "Open Questions" section.
