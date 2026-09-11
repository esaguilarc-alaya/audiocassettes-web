# Mission 18 — Video Fixes: Real Playback, Fit, Offset, + Real Seekable Scrubber

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Fix three real problems Esteban found testing Mission 17 on his own device (the reviewer's sandbox cannot reach youtube.com, so all of this needs verification against the real, live embed — not a mock), and add real seeking to the scrubber.

## Problem 1 (the important one): the video never actually starts real playback

Esteban's screenshot shows YouTube's own native "unstarted" UI still on screen during what should be active playback: its title/channel overlay bar at the top and its own large red-and-white play button in the center — layered awkwardly over our own topbar text and our own small translucent play icon. This is what YouTube's iframe shows when it has **not actually been told to play** (or the command didn't take effect), not a styling issue.

**Likely root cause (to be confirmed against the real embed, not assumed):** Mission 17's `YT.Player` config is missing `enablejsapi: 1` and/or an `origin` parameter matching the page's own origin. Both are commonly required for the postMessage-based JS API commands (`playVideo()`, `seekTo()`, `mute()`) to actually be honored by the embedded player rather than silently ignored — which would produce exactly this symptom: our code believes it called `playVideo()`, nothing throws, but the real player just sits on its unstarted thumbnail forever.

Investigate and fix on a real device that can actually load youtube.com (Mission 17's own reviewer sandbox couldn't — confirm this mission's fix the same way, for real, not via the same mock approach). Once real playback is confirmed working, re-check whether YouTube's own title/branding overlay still shows during playback — it normally hides automatically once a `controls:0` player is genuinely playing, so this may resolve on its own once the root cause is fixed. If it doesn't, that's a second thing to solve (e.g. `modestbranding`/confirming no `iv_load_policy` annotations are re-triggering it), not something to ignore.

## Problem 2: video sizing/fit

The video area shows a black gap rather than the video filling its box edge-to-edge. Once Problem 1 is fixed and real playback is actually happening, re-check this — investigate whether the `<iframe>` YouTube's own script creates is picking up this component's `width:100%; height:100%` CSS rule at all (a `YT.Player` constructor can set the iframe's `width`/`height` as attributes at creation time; confirm our CSS is what actually wins, not a stale attribute-based size) before changing the container's `aspect-ratio`. If the iframe genuinely is sized correctly and the mismatch is really the container's `16/10` aspect ratio against the video's native `16/9`, change `.cst-video-wrap`'s `aspect-ratio` to `16/9` to match — but confirm which of these it actually is first rather than guessing blindly.

## Problem 3: wrong offset

Esteban watched the real sync and wants **2.4 seconds**, not Mission 17's 2.8. Update `videoOffset` for "Tarde En La Mañana" from `2.8` to `2.4` in `src/pages/lado-a-lado-b.astro`. Nothing else about the sync logic changes.

## Problem 4 (new feature, not a bug): real seekable scrubber

The scrub bar currently only displays progress — Esteban wants to actually use it to navigate: **both** tap-anywhere-to-jump and a draggable handle, standard music-app behavior.

- Tapping anywhere on `.cst-scrub-bar` seeks the audio (`audio.currentTime = (tapX / barWidth) * audio.duration`) to that point immediately.
- The existing `.cst-scrub-fill`'s round end/handle becomes draggable (pointer events: `pointerdown`/`pointermove`/`pointerup`, not the older mouse+touch-separate event pattern) — dragging previews the seek position live (fill/handle follow the pointer) and commits the seek (`audio.currentTime = ...`) on release, not continuously during drag (avoids seeking on every pixel of movement).
- **This must also seek the synced video** for "Tarde En La Mañana" — call the same `seekTo(audio.currentTime + videoOffset, true)` logic Mission 17 already has for drift correction, right after the manual seek, so tapping/dragging the scrubber doesn't leave the video behind.
- For tracks without `audioSrc` (still placeholder), the scrubber remains exactly as decorative/non-functional as it's always been — guard on `audioSrc` being present, same pattern as everything else in Mission 16/17.
- Works with both mouse and touch (pointer events cover both) — verify on a real mobile viewport, not just desktop.

## Scope (in)

Everything under Problems 1–4 above.

## Scope (out — do not touch in this mission)

- No video for "El Ayer" or other tracks.
- No cassette-box redesign, no karaoke-synced lyrics.
- No persistent-across-navigation player.
- No changes to Media Session beyond whatever naturally follows from real seeking (e.g. `setPositionState` should reflect a manual seek too, if it's already wired from Mission 16 — don't add new Media Session scope beyond keeping it consistent).

## Acceptance Criteria

- [ ] On a real device/browser that can reach youtube.com: opening "Tarde En La Mañana" results in the video **actually playing** — YouTube's own unstarted-state UI (title overlay, big native play button) is gone once playback begins, confirmed by watching it, not just by the absence of a thrown error.
- [ ] The video visually fills its box with no stray black gap, on both a mobile and a desktop viewport.
- [ ] `videoOffset` is `2.4` for "Tarde En La Mañana"; behavior otherwise matches Mission 17.
- [ ] Tapping anywhere on the scrub bar for a real track seeks `audio.currentTime` to that position immediately, verified by checking the value changed to match the tap's proportional position.
- [ ] Dragging the scrub handle previews the position live and commits the seek on release — verified via simulated pointer down/move/up, checking `audio.currentTime` only changes at release, not on every intermediate move.
- [ ] Seeking (tap or drag) also re-syncs the video to `audio.currentTime + videoOffset` for "Tarde En La Mañana" — verified against the (real, on a device that can reach YouTube; mocked otherwise) player's position after a seek.
- [ ] Seeking on a placeholder track (no `audioSrc`) does nothing (no error, no visual jump) — same placeholder-discipline guard as everywhere else.
- [ ] Mission 15's fullscreen fix and Mission 16's back-navigation/prev-next/Media Session behavior all still work — re-run those checks, don't just assume no regression.
- [ ] No console errors across: open, play/pause, seek via tap, seek via drag, prev/next, back navigation, fullscreen.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.
- [ ] Diff scoped to `CassettePlayer.astro` and `lado-a-lado-b.astro` only.

## A verification note, same as Mission 17 (Rule 2)

The reviewer's sandbox still cannot reach youtube.com. Whether Problem 1's real fix actually works can only be confirmed on a real device — Code should verify this for real this time (not via a mock, since the mock cannot reveal whether `enablejsapi`/`origin` fixes the actual issue) and say plainly in the response file that this was checked against the live embed, with what was observed. Scrubber seeking, offset, and everything not dependent on the real YouTube embed should be verified normally.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_17_video-sync.md`.

1. **Brand fidelity rule.** No new colors/fonts/section names.
2. **Verification over self-report.** Problem 1 in particular must be confirmed against the real, live YouTube embed — not assumed fixed from reading the code, and not re-verified only via the same mock that couldn't catch this bug the first time.
3. **Placeholder discipline.** Scrubber seeking stays inert for tracks without `audioSrc`.
4. **Mission boundary discipline.** Nothing beyond Problems 1–4 above.
5. **Static-first constraint.** No server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts/icon packs/snippets.
7. **Diff before review.** Diffed against this mission's acceptance criteria.
8. **Mission handoff protocol.** Cowork places this mission file in `missions/` locally, uncommitted. Code commits and pushes it first, confirms the push, *then* executes scope, writes a **separate** response file at `missions/MISSION_18_response.md`, and commits and pushes that too.
9. **Mobile/responsive by default.** Checked at 320px, 375px, 768px — scrubber dragging specifically needs a real mobile/touch check, not just desktop mouse.

## Open Questions for Esteban (Code should ask, not assume)

None — all four problems and the desired scrubber behavior are settled above.
