# Mission 19 — Block Native YouTube Click/Hover Controls on the Embedded Video

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Esteban can hover over the playing video and see YouTube's own native pause affordance, and clicking it actually pauses the video **through YouTube's own click handling**, bypassing this app's controls entirely — that desyncs it from the audio (our code doesn't know it got paused) and produces visibly "sluggish"/janky playback as the periodic drift-correction logic fights against a state it didn't cause. Esteban's own suggested fix is exactly right: remove the ability to interact with the embedded video directly at all — every control must go through this app's own transport buttons/scrubber, never the raw iframe.

## Root cause (confirmed by reading the code, not guessed)

`#cstTapOverlay` (`.cst-tap-overlay`) is the only element sitting on top of `.cst-yt-host`'s iframe, and it gets `display: none` (via its `.is-hidden` class) as soon as playback starts (`tapOverlay?.classList.toggle("is-hidden", playing)`). Once it's `display: none`, there is nothing left covering the iframe — hover and click go directly to YouTube's own player underneath, controls param or not (this is a known real quirk: `controls: 0` hides YouTube's visible control bar, but does not fully block the iframe's own native click-to-toggle/hover behavior).

## Scope (in)

1. Add a dedicated, **always-present** (while a real video is loaded — i.e. `videoId` present), fully transparent click-shield layer positioned over `.cst-yt-host`'s iframe, with `pointer-events: auto`, that:
   - Never disappears during playback (unlike `.cst-tap-overlay`, which is a separate, existing element with its own purpose for placeholder tracks — don't repurpose or remove that one; add a new element for this).
   - Swallows every click/tap on the video area (`preventDefault`/`stopPropagation`, no other action) so nothing reaches the iframe's own native handling.
   - Does **not** block this app's own overlaid controls that must stay clickable: the fullscreen button (`#cstFsBtn`) and the rec badge are positioned separately over the video area and must keep working exactly as before — check actual current stacking/positioning, don't assume, and adjust z-index/pointer-events so the shield sits above the iframe but below (or excludes) those two elements.
   - For tracks without a `videoId` (the placeholder scene), nothing changes — this shield only exists/matters when the real video is present.
2. Confirm the shield doesn't interfere with this app's own JS control of the video (`playVideo()`/`pauseVideo()`/`seekTo()` via the postMessage API) — those aren't real DOM clicks on the iframe, so `pointer-events: none` on the iframe itself (an alternative/complementary approach, whichever Code judges cleaner) would also not block them; either approach (shield element, or `pointer-events: none` directly on the iframe with the shield as a defense-in-depth backstop) is acceptable as long as it's verified to actually work.
3. Re-verify Mission 18's scrubber (tap-to-seek, drag) and all transport controls still work exactly as before — this mission must not accidentally block clicks meant for the app's own controls.

## Scope (out — do not touch in this mission)

- No other changes to video sync/offset/fit — those are Mission 17/18's territory and already settled.
- No changes to the placeholder-track experience.
- No new features.

## Acceptance Criteria

- [ ] With "Tarde En La Mañana" playing, a click/tap anywhere on the video area does **not** pause or otherwise affect the video or audio — verified by dispatching a real click at the video area's coordinates and confirming `audio.paused` and the video's playback state are unchanged afterward.
- [ ] The fullscreen button and rec badge remain clickable/functional over the video area (not accidentally blocked by the new shield).
- [ ] Mission 18's scrubber (tap-to-seek and drag) and the transport controls (play/pause, prev/next) all still work exactly as before — re-run those checks, don't just assume.
- [ ] This app's own programmatic control of the video (play/pause/seek via the postMessage-based JS API) is unaffected — confirmed by exercising play/pause/seek through the app's own buttons/scrubber and confirming the video responds.
- [ ] No console errors introduced.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.
- [ ] Diff scoped to `CassettePlayer.astro` only (unless a genuine reason requires touching `lado-a-lado-b.astro`, which is unlikely for this fix).

## A verification note (Rule 2)

As with Missions 17/18, the reviewer's sandbox cannot reach youtube.com, so the real click-blocking behavior against the actual embedded iframe needs confirming on a real device the way Mission 18's fix was. Everything else (the app's own control paths, fullscreen/badge staying clickable, no regressions) can and should be verified normally.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_18_video-fixes-and-scrubber.md`.

1. **Brand fidelity rule.** No new colors/fonts/section names.
2. **Verification over self-report.** Confirm the click-block against the real embed on a real device, not assumed from reading the code.
3. **Placeholder discipline.** No change to tracks without a `videoId`.
4. **Mission boundary discipline.** Only this one fix — no scope creep into other video/player behavior.
5. **Static-first constraint.** No server/API/database dependency.
6. **No unlicensed third-party assets.** N/A — no new assets.
7. **Diff before review.** Diffed against this mission's acceptance criteria.
8. **Mission handoff protocol.** Cowork places this mission file in `missions/` locally, uncommitted. Code commits and pushes it first, confirms the push, *then* executes scope, writes a **separate** response file at `missions/MISSION_19_response.md`, and commits and pushes that too.
9. **Mobile/responsive by default.** Confirm the click-shield behaves correctly on touch (a tap should be swallowed the same way a click is) at 320/375/768px.

## Open Questions for Esteban (Code should ask, not assume)

None — the fix is fully specified above.
