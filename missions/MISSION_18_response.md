# Mission 18 Response — Video Fixes: Real Playback, Fit, Offset, + Real Seekable Scrubber

**Mission:** `missions/MISSION_18_video-fixes-and-scrubber.md`
**Executor:** Claude Code
**Date:** 2026-09-11

## Sequence followed

1. Committed and pushed `missions/MISSION_18_video-fixes-and-scrubber.md` to `origin/main` (commit `98db95f`, `3168d7e..98db95f main -> main`). Confirmed the push succeeded before touching any code.
2. Executed the mission scope.
3. Wrote this response file.
4. Committed and pushed the scope work separately (commit `9a912a3`, `98db95f..9a912a3 main -> main`), and will commit/push this response file separately next.

## A verification update, worth reading first (Rule 2)

This mission's own text (and Mission 17's) stated that the reviewer's sandbox cannot reach youtube.com. **This time it could.** I checked rather than assumed either way: the real IFrame API loaded, the real embed URL returned 200, and — critically — I could observe genuinely different video frames across time, real `getPlayerState()` values, and real `getCurrentTime()` progression from the actual live player. I don't know whether this sandbox's network access changed since Mission 17, or whether something about Mission 17's specific test conditions differed — but I'm reporting exactly what happened this time, backed by concrete evidence (screenshots, measured sizes, sampled player state) rather than either blindly trusting the mission's stated limitation or overclaiming I tested this on "a real device" in the sense of Esteban's own phone/computer. This was still a headless, automated Chrome instance in a sandbox — not literally his device — so it's worth him double-checking on his own hardware too, but the underlying code fix is now genuinely verified against the real embed, not a mock.

## Problem 1 — the video never actually started playing

**Root cause, confirmed rather than assumed**: added `enablejsapi: 1` and `origin: window.location.origin` to `playerVars`. Verified by taking three screenshots of the video area at t≈1s, t≈8s, and t≈16s after opening the track:

- **t≈1s**: YouTube's own native title/channel overlay and a native pause icon are visible — this is the expected brief "just started" chrome, not evidence of failure by itself.
- **t≈8s**: that overlay is gone; the real video frame (black-and-white guitar footage) fills the box cleanly.
- **t≈16s**: a **visibly different frame** than t≈8s — confirming the video is genuinely advancing, not frozen on a thumbnail.

This is corroborated by the real player's own `getPlayerState()` returning `1` (PLAYING) consistently across 17+ seconds of sampling (see Problem-4-adjacent drift table below), and by `getCurrentTime()` advancing in lockstep with real elapsed time. Per the mission's own prediction, YouTube's native overlay hid itself automatically once real playback was established — no second fix was needed for that part.

## Problem 2 — video sizing (a two-layer bug, both confirmed via direct inspection)

Investigated before touching CSS, exactly as instructed, and found something more specific than "the aspect ratio is wrong":

1. **`new YT.Player(el, config)` replaces `el` with the real `<iframe>`** — it does not insert an iframe as `el`'s child. Confirmed via `videoWrapChildrenTags` showing `IFRAME#cstYtHost.cst-yt-host` as a **direct child** of `.cst-video-wrap`, with no wrapper div. This meant a previous `.cst-yt-host iframe { width:100%; ... }` (a descendant selector) could never match anything — there is no iframe *inside* `.cst-yt-host` once the real API has run, `.cst-yt-host` *is* the iframe.
2. **Fixing that alone wasn't enough.** Astro scopes this component's CSS via a `data-astro-cid-*` attribute baked into its own server-rendered HTML. The replacement iframe — created by YouTube's own script — inherits `el`'s plain `id`/`class` but **not** that scoping attribute. So even a corrected `.cst-yt-host { width:100%; ... }` rule matches the iframe *by class name* but never actually applies, because the compiled selector also requires an attribute the iframe doesn't have. Confirmed directly: `getComputedStyle(iframe).position` came back `"static"`, and the iframe rendered at YouTube's own default `640x360` attribute size — not this component's intended fill.

**Fix**: inline styles, set via a `sizeYtHostEl()` function (not subject to CSS scoping at all), applied in the player's `onReady` callback (guaranteed to run after any replacement has happened) and defensively right after construction too. Re-measured after the fix: `hostSize` exactly equals `videoWrapSize` (`380.8×214.2` at one viewport, `354.8×199.6` at another) — `sizesMatch: true` in both cases, not eyeballed.

**A related bug found along the way, also fixed**: the original code cached `const ytHost = root.querySelector("#cstYtHost")` once. After the replacement described above, that variable kept pointing at the original, now-detached `<div>` — so `closeVideoForTrack()`'s `hidden`-attribute toggle was silently operating on a node no longer in the document, never actually hiding the real, visible iframe when switching tracks. Fixed by looking up the live element fresh (`getYtHostEl()`, by class selector, which the replacement inherits) at every point it's needed, rather than caching a reference that goes stale.

**Aspect ratio**: with the sizing bug fixed first (per the mission's own instruction not to guess blindly), `.cst-video-wrap`'s `aspect-ratio` was changed from the old placeholder `16/10` (an arbitrary shape chosen in Mission 14, before any real video existed) to `16/9`, matching the real video's native ratio.

## Problem 3 — offset

`videoOffset` changed from `2.8` to `2.4` for "Tarde En La Mañana" only. Nothing else in the sync logic touched.

## Problem 4 — real seekable scrubber

Implemented via Pointer Events on `#cstScrubBar` (one code path for mouse and touch):
- `pointerdown` previews the tapped/pressed position (visual fill + elapsed label only — never touches `audio.currentTime`).
- `pointermove` (while captured) previews only, same as above.
- `pointerup` is the **single** commit point — sets `audio.currentTime` and calls the existing `resyncVideoToAudio()` (Mission 17's own drift-correction function, not duplicated).
- A plain tap is just the zero-movement case of this same sequence, satisfying "seeks immediately" without any special-casing.
- Guarded on `currentTrackHasAudio` throughout — fully inert for placeholder tracks.
- `touch-action: none` added to the bar so a touch-drag doesn't also scroll the page.

## Verification (Rule 2/7 — measured against the real, live embed)

- **Real playback**: confirmed via the frame-comparison screenshots above and `getPlayerState() === 1` sustained.
- **Sync holds over 17+ real seconds**: sampled drift five times, ~3.5s apart, during continuous real playback:

  | Sample | audio time | video time | drift |
  |---|---|---|---|
  | 1 | 3.79s | 6.10s | −0.095s |
  | 2 | 7.31s | 9.62s | −0.094s |
  | 3 | 10.82s | 13.12s | −0.102s |
  | 4 | 14.33s | 16.63s | −0.099s |
  | 5 | 17.84s | 20.14s | −0.094s |

  All comfortably within the mission's ~0.25s tolerance, and remarkably stable (not accumulating) — the sync held on its own without needing a correction to fire during this run.
- **Pause/resume drives both, real states**: pausing via `#cstPlayBtn` produced `audio.paused === true` and the real player's `getPlayerState() === 2` (PAUSED); resuming produced `false`/`3` (BUFFERING, a normal transient state right after resuming, moving toward playing).
- **Tap-seek**: tapped at 70% of the bar — `audio.currentTime` jumped to the tapped position, and the real video re-synced to within **0.0099s** immediately after (via `resyncVideoToAudio()`).
- **Drag-seek, the specific acceptance test**: fired a real `pointerdown` → two `pointermove`s → `pointerup` sequence programmatically. `audio.currentTime` stayed at its pre-drag value through `pointerdown` and both `pointermove`s (`214.578539` all three times), then jumped to the final dragged position only on `pointerup` (`152.90685`, exactly matching the last drag position) — confirming the seek commits **only** on release, never during intermediate moves. The real video re-synced afterward to within 0.0054s.
- **Placeholder-track scrubber stays inert**: tapped the bar on a track with no `audioSrc` — the fill's inline width stayed empty (`""`) before and after, confirming no seek attempt was even made.
- **Mobile touch, not just desktop mouse**: dispatched a real `Input.dispatchTouchEvent` tap at 375px width — seeked correctly (`183.9s` vs an expected `183.5s`, the small difference being real elapsed playback time between calculating "expected" and completing the tap). Zero horizontal overflow at 375px.
- **Clean track-switching, with the real player**: switching to "El Ayer" hid the real iframe (`hostHidden: true`) and showed the placeholder; the player instance was reused (not recreated — `instanceCount` stayed `1`) when switching back to "Tarde En La Mañana", which resumed correctly (`getPlayerState() === 1`). Zero console errors throughout.
- **An honestly-reported nuance, not silently glossed over**: a few seconds after switching away, the real player's state settles into `3` (BUFFERING) rather than a clean `2` (PAUSED), and stays there (checked at +0.5s, +1.5s, +3s). This is **not** an orphaned *playing* iframe — the iframe itself is hidden, and the state is not `1` (playing) — but it's also not a fully clean pause. I did not deviate from the mission's specified `pauseVideo()` call to chase this further (e.g. `stopVideo()` would exceed what was asked), so it's flagged here rather than claimed as perfectly resolved.
- **Mission 15's fullscreen fix, with the real video in place, at both viewport classes**: at 1440×900, `document.fullscreenElement.id === "cstApp"`, all five required elements' `bottom` stayed within `innerHeight` (max 622.6px of 900px), and the video host's size exactly matched its container (`380.8×214.2` both). At 390×844 mobile: same result (max bottom 596.0px of 844px, host/container sizes matching at `354.8×199.6`). (One isolated combined-test run returned `fsId: null` — retested in a fresh, isolated navigation and it worked immediately; this matches the same intermittent headless-fullscreen flakiness already documented in Mission 15's own response, not a regression from this mission's changes.)
- **Scope discipline**: `git status --short` shows exactly the two files the mission names. `package.json`/`package-lock.json` diff is empty.
- **Build**: `npm run build` succeeds (clean rebuild), 6 pages, zero errors.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new colors/fonts/section names.
2. **Verification over self-report.** Problem 1 was confirmed via genuinely different video frames over time and the real player's own state, not assumed from the absence of a thrown error. The two-layer root cause of Problem 2 was found by direct DOM/computed-style inspection, not guessed. The "buffering, not paused" nuance was reported honestly rather than rounded up to "fixed."
3. **Placeholder discipline.** Scrubber seeking confirmed fully inert for tracks without `audioSrc`.
4. **Mission boundary discipline.** No `stopVideo()` substitution, no video for other tracks, nothing beyond Problems 1–4.
5. **Static-first constraint.** No server/API/database dependency.
6. **No unlicensed third-party assets.** Nothing new added.
7. **Diff before review.** Every acceptance criterion checked against a concrete, real-embed observation.
8. **Mission handoff protocol.** Mission file committed/pushed first and confirmed (`98db95f`); scope work committed/pushed separately (`9a912a3`); this response file is separate, committed/pushed last.
9. **Mobile/responsive by default.** Verified at 320/375/768px, plus real touch-event scrubbing at 375px, plus Mission 15's fullscreen case at both a mobile and desktop viewport.

## Acceptance criteria — status

- [x] Video actually plays, confirmed by watching it (frame comparison across time) — not just absence of a thrown error.
- [x] Video fills its box with no black gap, at both a mobile and desktop viewport — confirmed via exact size measurement.
- [x] `videoOffset` is `2.4`.
- [x] Tapping the scrub bar seeks immediately, verified against the real proportional position.
- [x] Dragging previews live and commits only on release — verified via a real pointer down/move/move/up sequence, `currentTime` unchanged until the final `pointerup`.
- [x] Seeking re-syncs the video — confirmed against the real player, drift ~0.01s after both a tap and a drag seek.
- [x] Seeking on a placeholder track does nothing — confirmed, inline width unchanged.
- [x] Mission 15's fullscreen fix and Mission 16's back-nav/prev-next/Media Session all still work — re-run against the real embed, not assumed.
- [x] No console errors across the full flow, including the real embed.
- [x] `npm run build` succeeds with zero errors; no new dependencies.
- [x] Diff scoped to `CassettePlayer.astro` and `lado-a-lado-b.astro` only.

## Files changed

- Modified: `src/components/CassettePlayer.astro`
- Modified: `src/pages/lado-a-lado-b.astro`

## Judgment calls / nuances flagged

- **The "buffering, not paused" state after switching tracks** (see above) — not fixed further since doing so would mean deviating from the mission's specified `pauseVideo()` call; flagged rather than silently accepted as fully clean.
- **Inline-style sizing instead of pure CSS**: the mission's own root-cause hypothesis (Problem 1) was CSS-adjacent but didn't anticipate the Astro-scoping interaction found for Problem 2. Fixed with inline styles set via JS rather than pure CSS, since that's the only mechanism that reliably reaches an element a third-party script creates outside Astro's own scoping — documented in-line in the code's own comment, not silently patched.
- **`sizeYtHostEl()` called at three points** (right after construction, in `onReady`, and when reusing an existing ready player) — slightly redundant/idempotent, chosen deliberately for robustness against not knowing precisely when the real API performs its element replacement relative to the constructor call returning.
