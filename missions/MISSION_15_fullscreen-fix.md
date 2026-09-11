# Mission 15 — Fullscreen Player: Fix Clipped Controls + Unresponsive Lyrics Close

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Fix two real, reproduced bugs in the Mission 14 player's fullscreen mode (`CassettePlayer.astro`), reported by Esteban testing on desktop Chrome (Mac):

1. After entering fullscreen, the transport controls are not visible/reachable.
2. The lyrics drawer's "✕" close button does not close the drawer.

## Root cause (confirmed by reviewer via Playwright, not guessed)

`#cstApp` is a fixed mobile-card layout (`max-width: 26rem`, `height: min(44rem, 82vh)`). When the browser's native Fullscreen API activates on it, the browser forces it to fill the **entire** screen (`width`/`height`/`position` become `100%`/`fixed` via the UA stylesheet, overriding the author's own sizing). On a desktop viewport (tested at 1440×900) this stretches the card edge-to-edge:

- The vertical stack (video, track info, scrubber, controls, lyrics toggle, lyrics drawer) grows taller than the viewport. The controls end up below the fold (`top: 1066px` in a 900px-tall viewport in the reproduction) — reachable only by scrolling, which isn't an obvious or expected interaction inside "fullscreen," so in practice they read as "missing."
- Separately, and independently reproducible: with the drawer in its **closed** state, its own box still overlaps the `#cstLyricsToggle` button's clickable area at this stretched size — confirmed directly by a real (non-JS-synthesized) Playwright click on the toggle failing with "element intercepts pointer events," pointing at a `<p>` inside the closed `#cstLyricsDrawer`. The same class of overlap is almost certainly why the close button inside the drawer doesn't register clicks either — the drawer's closed/open positioning is not robust to being interecpted or to the container's now-different size.

## Scope (in)

1. **Constrain fullscreen sizing.** When `#cstApp` is the fullscreen element, keep it at its intended card proportions (the existing `max-width: 26rem`, capped height) centered within the fullscreen viewport, with letterboxing (a plain dark background using an existing brand-token `color-mix()`, not a new color) filling the rest — the same visual idea as how video players letterbox instead of stretching content edge-to-edge. Use a `:fullscreen` (and `::backdrop` if useful) CSS rule scoped to `#cstApp` to reassert this — don't rely on the UA default. Confirm with a real click-triggered fullscreen (not `element.click()` in JS) that on a wide/short desktop viewport (test at least 1440×900) all of: video, track info, scrubber, transport controls, and the lyrics toggle are within the visible viewport with zero scrolling required.
2. **Make the lyrics drawer's closed state fully non-interactive.** Regardless of container size, the closed drawer must never intercept clicks meant for `#cstLyricsToggle` or anything below it — add `pointer-events: none` on `.cst-lyrics-drawer` by default and `pointer-events: auto` only on `.cst-lyrics-drawer.is-open`. This is a robust fix that doesn't depend on getting an exact transform/height calculation right for every viewport.
3. **Verify the close button.** With the drawer open, a real click (not JS-triggered) on `#cstCloseDrawer` must close it — confirmed the specific reproduction case (fullscreen active, desktop-sized viewport) as well as the normal non-fullscreen case.
4. Keep using only established brand tokens for the letterbox background (Rule 1).

## Scope (out — do not touch in this mission)

- No changes to the shelf/cassette screens, the tracklist, or the lado a/b toggle logic.
- No real audio/video wiring, no Media Session API — still waiting on Esteban to provide a real audio file (unchanged from Mission 14's deferral).
- No cassette-box visual redesign.
- No changes to lyrics content or karaoke sync.
- No new npm dependencies.

## Acceptance Criteria

- [ ] With a real (non-JS-synthesized) click on `#cstFsBtn` at a desktop viewport (test at minimum 1440×900), `document.fullscreenElement` is `#cstApp`, and the video, track info, scrubber, transport controls, and lyrics toggle are all within `window.innerHeight`/`innerWidth` with no scrolling needed — verified by reading each element's `getBoundingClientRect()` programmatically, not just visually.
- [ ] Same check passes at a mobile viewport (390×844) and at 320px/375px/768px widths per Rule 9 — no regression from Mission 14's mobile verification.
- [ ] With the lyrics drawer closed, a real Playwright click on `#cstLyricsToggle` succeeds without an "intercepts pointer events" error, at both a mobile and the 1440×900 desktop viewport.
- [ ] With the lyrics drawer open, a real Playwright click on `#cstCloseDrawer` succeeds and the drawer's `is-open` class is removed afterward — verified at both a mobile and the desktop viewport.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.
- [ ] All colors used in the letterbox/fullscreen styling trace to established brand tokens (Rule 1) — no invented hex values.
- [ ] Diff is scoped to `CassettePlayer.astro` only (or the minimum set of files needed) — no unrelated changes.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_14_lado-a-lado-b-player.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** N/A to this mission's scope (no new sample data).
4. **Mission boundary discipline.** Missions have explicit in/out scope. Do not touch anything in Scope (out) above.
5. **Static-first constraint.** No server/API/database dependency introduced.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against this mission's acceptance criteria — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_15_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px — this mission additionally must not regress that.

## Open Questions for Esteban (Code should ask, not assume)

None — the bug, its root cause, and the fix approach are settled above. If an exact letterbox shade or spacing needs a judgment call, use existing token `color-mix()` patterns already established in this file and record the choice in the response file.
