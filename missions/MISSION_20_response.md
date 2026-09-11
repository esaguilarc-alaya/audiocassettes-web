# Mission 20 Response — Rotating 3D Cassette Carousel ("nuestras cintas" shelf)

**Mission:** `missions/MISSION_20_carousel-shelf.md`
**Reference:** `reference/nuestras-cintas-carousel-reference.html`
**Executor:** Claude Code
**Date:** 2026-09-11

## Sequence followed

1. Committed and pushed `missions/MISSION_20_carousel-shelf.md` and `reference/nuestras-cintas-carousel-reference.html` together in one commit to `origin/main` (commit `081ff62`, `d623eeb..081ff62 main -> main`). Confirmed the push succeeded before touching any code.
2. Read the reference file's full source (markup, CSS, and script) before implementing.
3. Executed the mission scope.
4. Wrote this response file.
5. Committed and pushed the scope work separately (commit `53a9068`, `081ff62..53a9068 main -> main`), and will commit/push this response file separately next.

## What was built

Screen 1's flat `.cst-grid`/`.cst-box` shelf was fully replaced with the 3D carousel from the reference, ported into `src/components/CassettePlayer.astro` only.

### Structure (16 slots, one data model)
- **Face 0**: the 4 real cassettes, in `cassettes`'s existing order, each rendered as `.cst-spine` with `data-cassette-index={i}` — the exact same attribute the old `.cst-box` carried, so the click → `openCassette(index)` semantics are unchanged, just moved onto the new element.
- **Faces 1–3**: 12 markup-only placeholder slots, generated via `[1, 2, 3].map(...)` × `Array.from({ length: 4 })` — no fake entry was added to the `cassettes` array, no "placeholder cassette" object exists anywhere. Placeholder `.cst-spine` buttons carry no `data-cassette-index` and never get a click listener attached at all (confirmed — see verification below, not even a no-op listener).
- **`CAROUSEL_SPINE_LABELS`**: a small local array (`"Audiocassettes"`, `"Reel Real"`, `"En vivo - Montserrat, Coronado"`, `"En vivo - Calle Blancos"`), indexed the same way `DISK_COLORS` already is, used only for the carousel button's visible text. `c.title` itself is completely untouched and still drives Screen 2's eyebrow/hero — verified by opening all 4 real cassettes and matching each one's actual title (`"grabaciones — Audiocassettes"`, `"grabaciones — guanacaste"`, etc.), not just confirming *a* screen opens.
- Each real spine's `--`-token-derived color pair (`color-mix(... 80%, black 10%)` for `.cst-spine-face`, `color-mix(... 55%, black 45%)` for `.cst-spine-side`) is driven off the existing `DISK_COLORS[i % DISK_COLORS.length]` cycling — the same 5-token order `ViewMasterShelf.astro` established, not a new palette.

### Interaction (ported verbatim from the reference's own script)
- Turn buttons (`#cstCarouselTurnLeft`/`#cstCarouselTurnRight`) rotate one face at a time, looping in both directions.
- Drag-to-rotate via Pointer Events: `pointerdown` on the stage, `pointermove`/`pointerup` on `window` (so a drag that moves outside the carousel's own bounds still tracks correctly) — the same pattern Mission 18's scrubber already established for this file. Snap-to-nearest-face on release.
- `BASE_ANGLE = 24` and `FACE_ANGLE = 90` kept exactly as the reference's own constants — no adjustment was needed; verified the permanent corner-viewing offset is present at *every* face, not just face 0 (see verification below).
- Face-dot indicator stays in sync with the current face through both button and drag interactions.

### A real cross-navigation bug avoided, not just noticed
The `pointermove`/`pointerup` listeners live on `window`, which — unlike this component's own DOM — persists across Astro's soft navigations. A naive implementation would accumulate a new pair of these listeners (each closing over a *different* init's now-stale carousel state) every time a visitor navigates away from and back to this page. Deduped using the exact same "track the previous handler in a module-level variable, remove it before adding the new one" pattern Mission 17 already had to establish for its `visibilitychange` listener — documented in-line in the code's own comment, not silently added.

## A real bug found and fixed during this mission's own verification (Problem: narrow-viewport clipping)

The reference's carousel dimensions (`18.5rem` × `20rem`, `23rem`-tall stage) were sized against its own standalone demo page's `30rem`-max-width container — a generous margin (the carousel is ~62% of its container's width there). This site's actual mobile shelf screen, inside `.cst-card`'s own narrower layout, gives the carousel only ~272px of content width at 320px — and the reference's fixed rem values, screenshot-confirmed, visibly clipped text and card edges against `.cst-card`'s own boundary at that width.

**Fixed, not redesigned**: added a scaled-down variant inside the *existing* `@media (max-width: 26.25rem)` rule (the same breakpoint `.cst-card` itself already uses), reducing the carousel to `10.5rem` × `11.3rem` with a proportionally reduced `translateZ`/perspective/lid — a size chosen empirically (not just linearly scaled from the original) so the carousel occupies roughly the same *proportion* of its narrower container as the reference's own carousel does of its wider one, since a straight linear scale-down of every dimension still clipped on a first attempt (confirmed by measurement, then corrected — not assumed correct on the first try). Re-verified via direct `getBoundingClientRect()` comparison against `.cst-card`'s own edges (not just a visual re-check) at 320px, 375px, and 768px — no spine face extends past the card's boundary at any of the three required widths, and the turn buttons remain fully within the card.

## Verification (Rule 2/7 — measured, not eyeballed)

- **Structure**: 4 faces, 16 total slots, 12 of them `.cst-placeholder-slot`, face 0's 4 titles read exactly `"Audiocassettes"`, `"Reel Real"`, `"En vivo - Montserrat, Coronado"`, `"En vivo - Calle Blancos"` in that order. Exactly 4 elements match `.cst-spine[data-cassette-index]`; every placeholder spine confirmed to have no `data-cassette-index` attribute at all.
- **Correct cassette opens, not just *a* screen**: clicked each of the 4 real spines individually — each one's Screen 2 eyebrow/hero title matched that specific cassette's real title (`grabaciones — Audiocassettes`, `— guanacaste`, `— san josé`, `— monteverde`), confirmed via a fresh navigation + click + check cycle per cassette, not a single combined pass that could mask a mismatch.
- **Placeholder slots are genuinely inert**: rotated to face 1 first (so a placeholder is actually front-facing, not testing a 3D-rotated-away element's stale 2D bounding box — an early test-methodology mistake on my own part that I caught and corrected before trusting the result), confirmed via `elementFromPoint` that the click lands on the placeholder spine itself, then clicked it and confirmed the shelf screen stayed active (no cassette screen opened).
- **Turn buttons, looping both directions**: 5 consecutive right-clicks produced face-dot sequence `[1, 2, 3, 0, 1]`; 5 consecutive left-clicks produced `[3, 2, 1, 0, 3]` — both loop correctly in both directions.
- **Permanent base-angle offset present at every face, not just face 0**: read the carousel's actual computed `transform` at rest on all four faces in sequence — `rotateY(24deg)`, `rotateY(-66deg)`, `rotateY(-156deg)`, `rotateY(-246deg)`. None of these is a flat multiple of 90° (0/−90/−180/−270) — the +24° offset is baked into every single face's resting angle, confirming the carousel never rests flat-on anywhere in its rotation.
- **Drag-to-rotate with snap**: dispatched a real `pointerdown` → `pointermove` → `pointermove` → `pointerup` sequence (an ~380px leftward drag) — `is-dragging` class was present mid-drag and correctly removed after release, and the carousel snapped to face 1 (the nearest face for that drag distance).
- **Pop-out effect, actually confirmed visible, not just "properties present"**: this is exactly the check the mission called out by name, given the preserve-3d-chain bug it described. `getComputedStyle()` confirms `transform-style: preserve-3d` on `.cst-face`, `.cst-slot`, *and* `.cst-spine` (all three, matching the mission's explicit list) — and, more importantly, the spine's actual computed `transform` is `matrix3d(1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 8.8, 1)` — a real, non-zero 8.8px Z-translation (exactly `0.55rem` at a 16px root font-size) baked into the rendered matrix. If the preserve-3d chain were broken the way the mission described, this would either collapse to a 2D `matrix()` with no Z-component or resolve to `tz = 0` — it does neither. Screenshots at all four tested viewport widths also visually show each cassette box sitting proud of its slot with its own drop shadow and visible side edge.
- **320px/375px/768px (Rule 9)**: no spine face extends past `.cst-card`'s own edges at any of the three widths (measured directly, not visually estimated); turn buttons remain fully within the card's bounds at all three; zero page-level horizontal overflow (`document.documentElement.scrollWidth === window.innerWidth`) at every width.
- **Full Screens 2/3 regression, re-run against the real, live YouTube embed** (this sandbox can reach it, as established in Missions 18/19) — not assumed unaffected just because Screen 1 was the only intended target:
  - Cassette title, real-audio playback (`audio.paused === false`, correct `5:06` duration), scrubber advancing, Media Session metadata, pause/resume, back-navigation not pausing, prev/next between the two real tracks, real lyrics (23 lines, no placeholder note), placeholder-track guards, zero console errors — all 14 of Mission 16's own checks re-run and passed (one scrubber-advancement sample came back flat on a first run and correct on an immediate re-run — the exact same early-buffering timing flake already documented in Mission 16's own response, not a regression).
  - Mission 19's click-shield: a real click and a real touch tap on the video area, twice each, left `audio.paused` and the real player's `getPlayerState()` completely unchanged; the fullscreen button and rec badge remained independently hit-testable to themselves; the scrubber and transport controls all still worked.
  - Mission 18's drag-seek: a real pointer down/move/move/up sequence showed `audio.currentTime` frozen through every intermediate move and only jumping to the final position on release, with the video re-syncing to within 0.024s afterward.
  - Mission 15's fullscreen fix, at both 1440×900 and 390×844: `document.fullscreenElement.id === "cstApp"`, all required elements within `innerHeight`, and the video host's size exactly matching its container at both viewports. (One isolated run inside a very long combined test returned `fsId: null` — retested immediately in a fresh, isolated navigation and it worked at both viewports; this is the same intermittent headless-fullscreen interaction-fatigue already documented in Missions 15/18's own responses, not a regression from this mission.)
- **Scope discipline**: `git status --short` shows exactly one modified file, `src/components/CassettePlayer.astro`. `lado-a-lado-b.astro` was not touched (no data-model changes were needed — the carousel consumes the same `cassettes` prop the old shelf did). `grep` confirms zero remaining references to `.cst-grid`/`.cst-box`/`.cst-label` anywhere in the file (only a code comment mentions the old class name, explaining the rename) — the old shelf's dead CSS/markup was fully removed, not left alongside the new carousel.
- **Build**: `npm run build` succeeds (clean rebuild), 6 pages, zero errors/warnings.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** Every carousel color is one of the 6 established tokens or a `color-mix()` derivative of one — ported directly from the reference, which already used only those tokens.
2. **Verification over self-report.** The pop-out effect was confirmed via computed-style matrix decomposition, not assumed from the CSS properties being present; the narrow-viewport fix was measured against `.cst-card`'s actual edges, not re-eyeballed; an early test-methodology mistake on my own part (querying a 3D-rotated-away element's stale bounding box) was caught and corrected before trusting the result, not silently reported as a bug or a pass.
3. **Placeholder discipline.** The 12 placeholder slots are markup-only, clearly labeled "próximamente," with no data-model entry and no click listener of any kind attached.
4. **Mission boundary discipline.** Screens 2/3 were not touched at all (confirmed via `git status` and via re-running their full behavior suite); no View-Master-style video section was added; no visual redesign beyond the narrow-viewport fit fix, which was flagged rather than treated as a design opportunity.
5. **Static-first constraint.** Pure client-side DOM/CSS/pointer-event code — no new server/build dependency.
6. **No unlicensed third-party assets.** Nothing new — same fonts/tokens already established.
7. **Diff before review.** `git diff`/`git status` checked before writing this response; every acceptance criterion checked against a concrete measurement.
8. **Mission handoff protocol.** Mission file and reference committed/pushed together first and confirmed (`081ff62`); scope work committed/pushed separately (`53a9068`); this response file is separate, committed/pushed last.
9. **Mobile/responsive by default.** Verified at 320/375/768px, plus Mission 15's fullscreen case at 1440×900 and 390×844.

## Acceptance criteria — status

- [x] Screen 1 shows the rotating carousel, not the old grid.
- [x] All 4 real cassettes appear on face 0, original order, exact display titles from item 3, each tappable and opening the correct Screen 2 cassette (verified by title match per cassette, not just *a* screen).
- [x] Faces 1–3 show 12 total "próximamente" placeholder slots, inert.
- [x] Turn buttons rotate one face at a time, looping both directions; face-dot indicator reflects the current face.
- [x] Dragging (pointer events, mouse and touch) rotates proportionally and snaps to the nearest face on release.
- [x] Permanent base-angle offset present — confirmed at every face's resting angle, not just face 0.
- [x] Pop-out effect confirmed actually visible via computed transform matrix, not just CSS-property presence.
- [x] Verified at 320px, 375px, 768px — a real clipping bug was found and fixed, then re-confirmed clean.
- [x] Full Screens 2/3 regression pass — re-run against the real embed, all green.
- [x] `npm run build` succeeds with no new errors/warnings.

## Files changed

- Modified: `src/components/CassettePlayer.astro`

## Judgment calls flagged

- **Narrow-viewport carousel scaling** (see the dedicated section above): the mission explicitly anticipated this exact situation ("if something in the reference looks wrong once it's real... fix it minimally and flag it"). Chosen dimensions were arrived at empirically (measured, corrected, re-measured) rather than derived from a single formula, since a straightforward proportional scale-down of every dimension still clipped on the first attempt.
- **`.cst-spine` reused for both real and placeholder buttons** (rather than a separate class for each), with `[data-cassette-index]` as the sole distinguishing selector for click-wiring — one of the two options the mission explicitly offered ("give them an additional class like `.cst-spine`... or reuse `.cst-box` as an additional class on the real spines — your call"); chosen because it naturally guarantees placeholder spines can never accidentally match the click-wiring selector, without needing a second class to keep in sync.
- **`BASE_ANGLE`/`FACE_ANGLE` left unchanged** at the reference's own values (24/90) — no real-device testing surfaced a need to adjust them, so per the mission's own instruction they were kept as-is.
