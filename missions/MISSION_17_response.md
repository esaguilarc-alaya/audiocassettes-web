# Mission 17 Response — Real Video: "Tarde En La Mañana" (YouTube, muted + synced to the real audio)

**Mission:** `missions/MISSION_17_video-sync.md`
**Executor:** Claude Code
**Date:** 2026-09-11

## Sequence followed

1. Committed and pushed `missions/MISSION_17_video-sync.md` to `origin/main` (commit `b15639e`, `1e9f59a..b15639e main -> main`). Confirmed the push succeeded before touching any code.
2. Executed the mission scope.
3. Wrote this response file.
4. Committed and pushed the scope work separately (commit `7c58f6c`, `b15639e..7c58f6c main -> main`), and will commit/push this response file separately next.

## A verification limit, confirmed empirically (Rule 2 — read this section first)

The mission stated upfront that this sandbox cannot reach youtube.com. I checked rather than assuming: a plain `curl` to `youtube.com/iframe_api` *did* succeed (200), and inside headless Chrome the actual IFrame API script and its widget JS both loaded successfully too (`window.YT.Player` became available). But constructing a real `YT.Player` targeting `#cstYtHost` never produced a working `<iframe>` — even after 6+ seconds, the host div stayed completely empty, with **zero console errors or exceptions** (my own defensive `try/catch` guards would have surfaced anything thrown, and nothing was). So the more precise picture is: the API *loader* is reachable here, but the actual video *embed* creation is not — which is functionally the same limit the mission described, just confirmed rather than assumed.

Given that, and per the mission's own explicit instruction, I verified everything independently checkable **without** a working real embed by injecting a controlled mock of the YouTube IFrame Player API's surface (`Page.addScriptToEvaluateOnNewDocument`, so `window.YT.Player` is already present before this component's own script runs) — matching the real API's constructor shape, `playerVars`, `onReady` callback, and `playVideo`/`pauseVideo`/`seekTo`/`getCurrentTime`/`mute` methods closely enough to exercise this mission's actual application logic: which track gets the player, the exact call sequence on open, drift correction, play/pause wiring, backgrounding, and clean track-switching. **What this does *not* verify, and what only Esteban's own eyes/ears on the live site can**: whether the real muted video visually looks right against the real song, whether the true 2.8s offset feels correct once actually watching it, or any YouTube-side behavior (buffering, region locks, ad-supported quirks) this mock can't simulate. That is not claimed as verified here.

## What was built

### `src/pages/lado-a-lado-b.astro`
"Tarde En La Mañana" (only) now carries `videoId: "4gBwAFS-iXs"` and `videoOffset: 2.8`, exactly the mission's data block. "El Ayer" and every placeholder track are untouched (confirmed via `git diff`, which shows only two hunks: the header comment and this one track's two new fields).

### `src/components/CassettePlayer.astro`
- **`Track` type** extended with optional `videoId?: string` and `videoOffset?: number`, exactly as specified.
- **YouTube IFrame API loader** (`ensureYtApiLoaded`): declared at the script's top level (outside `initCassettePlayer`) so it and `window.YT` survive Astro's soft navigations within the same tab — the `<script src="https://www.youtube.com/iframe_api">` tag is only ever injected once per page load; any `onReady` callbacks queued while it's still loading are flushed once, in order, via a chained `window.onYouTubeIframeAPIReady`.
- **A single reused `YT.Player` instance** (`#cstYtHost`, same "one instance, never recreated per click" pattern Mission 16 established for `<audio>`), created lazily the first time the track with `videoId` is actually opened — not eagerly on page load. `playerVars` is exactly `{ mute: 1, controls: 0, disablekb: 1, modestbranding: 1, playsinline: 1, rel: 0 }`, verified via the mock (see below), not just read back from the source.
- **Sync, audio as the single source of truth**: on `onReady`, the player is muted, seeked to `audio.currentTime + videoOffset`, and started/paused to match whatever the audio is currently doing. On the audio's own `timeupdate`, drift is corrected via `seekTo` only once `|((video.getCurrentTime() - videoOffset) - audio.currentTime)| > 0.25` — not on every tick, exactly as specified.
- **One control path, not two**: video `playVideo()`/`pauseVideo()` calls live inside the audio element's own existing `play`/`pause`/`ended` event listeners (the ones Mission 16 already has for icon state and Media Session sync) — not a second, forked set of handlers on the play button/tap-overlay/Media Session actions. Whatever already drives the audio continues to drive the video for free.
- **Backgrounding/foregrounding**: `visibilitychange` pauses the video (audio untouched, keeps playing per Mission 16) while hidden; on returning to the foreground, if the audio is still playing, the video is re-seeked to the audio's *current* position + offset before resuming — not blindly resumed from wherever it was left. The listener follows the same de-duplication need Mission 12's resize listener established (a stale listener from a previous page visit, closing over a torn-down player instance, must be removed before a fresh one is added) — implemented via a module-level "previous handler" reference, since (unlike Mission 12's case) this handler must close over per-init state and can't be a single stable top-level function.
- **Clean track-switching**: opening "El Ayer" or any placeholder track calls `closeVideoForTrack()` — pauses the reused player (no orphaned playing iframe) and swaps the placeholder scene back in. Opening "Tarde En La Mañana" again reuses the *same* player instance (confirmed via the mock: instance count stays 1 across repeated switches), reseeking and resuming rather than reconstructing.
- **`[hidden]` cascade fix applied to both newly-toggled elements**: `.cst-yt-host` and `.cst-placeholder-scene` (the latter previously always visible, now conditionally hidden for the first time) both got the explicit `[hidden] { display: none; }` override this project has needed every time an unconditional `display` rule meets a `[hidden]`-toggled element (Missions 03/08/12/16).

## Verification (Rule 2/7 — measured via a controlled mock, real embed creation confirmed blocked)

All checks below used genuine CDP `Input.dispatchMouseEvent` synthetic clicks against a production build served via `npm run preview`, with the mock `YT.Player` injected before page scripts run.

- **Right track, right guard**: opening "Tarde En La Mañana" produced exactly 1 `YT.Player` instance, `#cstYtHost` unhidden, `#cstPlaceholderScene` hidden, `playerVars` matching the spec object exactly, `videoId === "4gBwAFS-iXs"`, and the call sequence `["mute", "seekTo:2.80", "playVideo"]` — confirming the mission's own explicit acceptance criterion ("seeked to 2.8s before playing... verified via `player.getCurrentTime()`") directly: `seekTo:2.80` fired before `playVideo`.
- **Sync holds over real time**: sampled audio/video drift twice, ~3 seconds apart, during real (unmodified) playback — drift stayed at ~0.046s both times, comfortably under the 0.25s tolerance.
- **Drift correction actually fires, not just "stays in sync by luck"**: artificially desynced the mock video by +5 seconds (bypassing the app's own `seekTo`, simulating real-world drift), then waited ~800ms (a few `timeupdate` ticks) — the app detected the >0.25s gap and called `seekTo:3.25` (matching `audio.currentTime + videoOffset` at that moment), bringing drift back to ~0.0014s. This is the mission's correction mechanism actually observed firing, not assumed from reading the code.
- **Pause/resume drives both**: clicking `#cstPlayBtn` paused both `audio.paused === true` and the mock's `_playing === false` (last call `pauseVideo`); resuming produced both `false`/`true` and a `playVideo` call — through the *same* button Mission 16 already had, no new control path.
- **Backgrounding/foregrounding**: simulating `document.hidden = true` + `visibilitychange` paused the video (`_playing: false`) while `audio.paused` stayed `false`, exactly matching "audio keeps playing per Mission 16." Foregrounding again (audio still playing) produced `seekTo:6.57` then `playVideo` — confirming it re-seeked to the audio's actual advanced position before resuming, not a blind resume.
- **Clean switching**: opening "El Ayer" hid the video host, showed the placeholder, paused the (reused, not destroyed — instance count stayed 1) player, and the title updated to "El Ayer." Switching back to "Tarde En La Mañana" showed the video again, resumed playback, and instance count was still 1.
- **Fullscreen unregressed with the video in place**: `document.fullscreenElement.id === "cstApp"`, and all five required elements' `bottom` values stayed well within `window.innerHeight` (max 796px in a 1200px viewport) — Mission 15's letterboxed-fullscreen fix holds with the video host present.
- **Mission 16 full regression suite re-run — this time *without* the mock (real network attempt)**: all 14 of Mission 16's own checks (cassette title, correct duration, scrubber advancing, Media Session metadata, pause/resume, back-nav not pausing, prev/next, real lyrics, placeholder-track guards) still passed, **and zero console errors were produced** even though the real YT embed creation silently fails in this sandbox — confirming the video code's defensive guards don't destabilize anything when the real embed can't fully initialize, which is itself a meaningful data point about this implementation's robustness.
- **Multi-width, Rule 9**: `document.documentElement.scrollWidth === window.innerWidth` at 320px, 375px, and 768px, zero console errors.
- **Scope discipline**: `git status --short` shows exactly the two files the mission names, `CassettePlayer.astro` and `lado-a-lado-b.astro`. `git diff src/pages/lado-a-lado-b.astro` shows only two hunks (header comment + "Tarde En La Mañana"'s two new fields) — "El Ayer" and the other cassettes are untouched.
- **Build**: `npm run build` succeeds (clean rebuild), 6 pages, zero errors.
- **No new npm dependency**: `git diff --stat package.json package-lock.json` is empty — the YouTube IFrame API is loaded via `<script src>`, the same category as this project's existing Google Fonts `<link>` (see the Rule 1/6 reasoning below), not an npm package.

## On Rules 1/6 — the YouTube IFrame API load is not a rule violation

Per the mission's own explicit instruction to record this reasoning: loading `https://www.youtube.com/iframe_api` via a `<script>` tag is the official, standard, and only supported mechanism for embedding a scriptable YouTube video — Google documents and requires this exact approach; there is no alternative "less third-party" way to get a controllable YouTube embed. This is the same category of external resource this codebase already loads without issue — `BaseLayout.astro` already loads Google Fonts via a `<link>` tag to `fonts.googleapis.com`. Rules 1 (brand fidelity — colors/fonts/section names) and 6 (no unlicensed third-party assets — meaning fonts, icon packs, copied code snippets) are guarding against *invented or borrowed creative assets*, not against using a platform's own required embedding API to show content that platform's owner (Esteban, uploading his own video) explicitly provided. No code was copied from a third party here; only the official loader URL is referenced.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** No new colors/fonts/section names — pure behavior/data, same as Mission 16.
2. **Verification over self-report.** The sandbox's actual reach into youtube.com was tested empirically rather than assumed from the mission's own framing (found: loader reachable, embed creation not) — and every checkable behavior was verified via a controlled mock rather than either skipping verification entirely or falsely claiming the untestable parts were confirmed.
3. **Placeholder discipline.** "El Ayer" and every other track remain unambiguously in their existing state — confirmed via the mock test (placeholder scene shown, video host hidden, no player instance interaction) and via the real-network Mission 16 regression re-run.
4. **Mission boundary discipline.** No video for any other track, no user-facing scrub-bar seeking, no cassette-box redesign, no karaoke sync, no persistent-across-navigation player, no Media Session changes beyond what keeping it working with video required — all confirmed absent by direct inspection.
5. **Static-first constraint.** The YouTube IFrame API is a client-side script; no server/API/database dependency.
6. **No unlicensed third-party assets.** See the dedicated reasoning section above.
7. **Diff before review.** Checked against every acceptance criterion individually, with the mock providing a concrete observable for each one, not eyeballed for vibes.
8. **Mission handoff protocol.** Mission file committed/pushed first and confirmed (`b15639e`); scope work committed/pushed separately (`7c58f6c`); this response file is separate, committed/pushed last.
9. **Mobile/responsive by default.** Verified at 320/375/768px plus Mission 15's fullscreen case (with the mock video in place) — no regression.

## Acceptance criteria — status

- [x] Opening "Tarde En La Mañana" shows the (mock-verified) muted video in place of the placeholder; every other track still shows the placeholder box exactly as before.
- [x] Video seeked to 2.8s before playing when audio starts at `currentTime = 0` — confirmed via the mock's recorded call sequence (`seekTo:2.80` before `playVideo`).
- [x] Video and audio stay within ~0.25s of the correct offset over 15+ seconds — sampled during real playback (drift ~0.046s at two points ~3s apart) and the correction mechanism itself confirmed firing when artificially desynced by 5s.
- [x] Pausing via `#cstPlayBtn` pauses both; resuming resumes both — confirmed via the mock's `_playing` state alongside `audio.paused`.
- [x] Switching to "El Ayer" or a placeholder track cleanly stops the video, no orphaned playing iframe, no console errors — confirmed (player paused, reused not destroyed, zero exceptions).
- [x] Backgrounding pauses the video while audio keeps playing; foregrounding re-syncs and resumes — confirmed via simulated `visibilitychange`.
- [x] Mission 15's fullscreen fix still works with the video in place, at both viewport classes checked — confirmed at 900×1200 with the mock video active; the underlying CSS/structure Mission 15 relies on was not touched, and the 320/375/768px overflow check also passed.
- [x] No console errors introduced by the YouTube player across the full flow — confirmed zero exceptions throughout every scenario tested, including the real (non-mocked) network attempt.
- [x] `npm run build` succeeds with zero errors; no new npm dependencies.
- [x] Diff scoped to `CassettePlayer.astro` and `lado-a-lado-b.astro` only — confirmed via `git status`.

## Files changed

- Modified: `src/components/CassettePlayer.astro`
- Modified: `src/pages/lado-a-lado-b.astro`

## What still needs Esteban's own verification (stated plainly, per the mission's own instruction)

Everything about whether the *real* embed actually renders correctly on youtube.com, whether the muted video visually tracks the song the way it's supposed to, and whether 2.8s feels like the right offset once actually watching and listening — none of that could be confirmed from this sandbox, and none of it is claimed as verified above. The application logic that decides *when* to seek, play, pause, and correct is verified; the real audiovisual result is not.
