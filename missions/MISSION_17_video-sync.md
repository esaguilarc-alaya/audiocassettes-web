# Mission 17 — Real Video: "Tarde En La Mañana" (YouTube, muted + synced to the real audio)

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Replace the video placeholder box with the real YouTube video for "Tarde En La Mañana" — the first (and for now, only) track with real video. The video's own soundtrack starts ~2.8s later than our uploaded audio file, so the video plays **muted**, offset, and kept in sync against the real `<audio>` element (from Mission 16), which remains the single source of truth for sound — exactly the hybrid Esteban chose earlier ("el audio siga sonando, no el video").

**Video:** https://www.youtube.com/watch?v=4gBwAFS-iXs (`videoId: "4gBwAFS-iXs"`)
**Start offset:** 2.8 seconds — when the audio is at `currentTime = 0`, the video should be showing its frame at `2.8s`, not `0s`. Esteban gave this exact number after listening; not a value to second-guess or recompute.

## Data changes (`src/pages/lado-a-lado-b.astro`)

Extend only "Tarde En La Mañana"'s entry with two new fields:

```js
{
  title: "Tarde En La Mañana",
  loc: "Audiocassettes",
  dur: "5:06",
  audioSrc: "/audio/tarde-en-la-manana.mp3",
  videoId: "4gBwAFS-iXs",
  videoOffset: 2.8,
}
```

No other track gets these fields — "El Ayer" and every placeholder track keep showing the existing video placeholder box exactly as Mission 14/15/16 built it.

## Scope (in)

1. **Extend the `Track` type** with optional `videoId?: string` and `videoOffset?: number`.
2. **Load the YouTube IFrame Player API** (`https://www.youtube.com/iframe_api`) via a `<script>` tag, the official/standard mechanism for embedding a controllable YouTube video. This is not a "new dependency" or an "unlicensed third-party asset" in the sense Rules 1/6 are guarding against (fonts, icon packs, copied code snippets) — it's the only supported way to embed and script a YouTube video, same category as loading Google Fonts via a `<link>` tag, which this codebase already does. Note this reasoning in the response file so it's not mistaken for a rule violation later.
3. **Render a real, muted YouTube player** in `.cst-video-wrap` in place of the placeholder scene, **only** for a track that has a `videoId`. Every track without one keeps the exact current placeholder box (svg + "video (placeholder)" label) — guard all new video code on `videoId` being present.
   - `playerVars`: `mute: 1, controls: 0, disablekb: 1, modestbranding: 1, playsinline: 1, rel: 0` (or the closest equivalent set) — this is a background visual, not something the visitor drives directly; the existing transport controls (play/pause, prev/next, scrubber) remain the only controls, same as Mission 16.
4. **Sync the video to the audio, not the other way around:**
   - When the real audio starts playing at a track with `videoOffset`, seek the YouTube player to `videoOffset` (in seconds) and play it.
   - On the audio's `timeupdate`, if `Math.abs((video.getCurrentTime() - videoOffset) - audio.currentTime) > 0.25` seconds, correct by calling `video.seekTo(audio.currentTime + videoOffset, true)` — keeps drift from accumulating without constantly reseeking on every tick.
   - Audio play/pause drives video play/pause (call the video player's `playVideo()`/`pauseVideo()` from the exact same functions Mission 16 already uses to control the audio — don't fork a second control path).
5. **Backgrounding/visibility handling:** on `visibilitychange` going hidden, pause the video player (it's not visible and browsers throttle/kill background iframe playback anyway — no point fighting that, and it saves resources); the audio keeps playing per Mission 16/the established background-audio pattern. On becoming visible again, if the audio is still playing, re-seek the video to the audio's current position + offset and resume it — don't just blindly resume from wherever the video was left, since the audio kept advancing while backgrounded.
6. Prev/next and opening a placeholder track must still behave exactly as Mission 16 shipped — switching away from "Tarde En La Mañana" should tear down/pause its video player cleanly (no orphaned playing iframe, no console errors).
7. Fullscreen (Mission 15) must keep working unchanged with the real video in place.

## Scope (out — do not touch in this mission)

- No video for "El Ayer" or any other track — Esteban only provided one video.
- No user-facing seeking/scrubbing by dragging the scrub bar (still visual/read-only, same as before).
- No cassette-box redesign, no karaoke-synced lyrics.
- No persistent-across-navigation player (that's a separate, already-discussed follow-up).
- No changes to Media Session metadata/behavior beyond what's needed to keep it working exactly as Mission 16 shipped it.

## Acceptance Criteria

- [ ] Opening "Tarde En La Mañana" shows the real (muted) YouTube video instead of the placeholder box; every other track still shows the placeholder box exactly as before.
- [ ] When audio playback starts at `currentTime = 0`, the video is seeked to `2.8s` before playing — verified via `player.getCurrentTime()` immediately after starting.
- [ ] During playback, video and audio stay within ~0.25s of the correct offset over at least 15 continuous seconds of playback (sampled at a few points, not just once at the start) — verified programmatically, not just eyeballed.
- [ ] Pausing via `#cstPlayBtn` pauses both audio and video; resuming resumes both.
- [ ] Switching to "El Ayer" or a placeholder track cleanly stops "Tarde En La Mañana"'s video (no orphaned playing iframe, no console errors) — verified by checking the player's state and the console after switching.
- [ ] Backgrounding the tab (`document.visibilitychange` / `document.hidden = true`) pauses the video while audio keeps playing (per Mission 16); returning to the foreground re-syncs and resumes the video to match wherever the audio has gotten to.
- [ ] Mission 15's fullscreen fix still works with the real video in place — `document.fullscreenElement.id === "cstApp"`, all controls remain visible without scrolling, at both a mobile and a desktop viewport.
- [ ] No console errors introduced by the YouTube player across the full flow (open, pause/resume, prev/next away and back, backgrounding/foregrounding, fullscreen).
- [ ] `npm run build` succeeds with zero errors; no new **npm** dependencies (the YouTube IFrame API is loaded via `<script src>`, same category as the existing Google Fonts `<link>`, not an npm package).
- [ ] Diff scoped to `CassettePlayer.astro` and `lado-a-lado-b.astro` only.

## A verification limit, stated upfront (Rule 2)

The reviewer's sandbox cannot reach youtube.com (network egress there is restricted), so the actual video ↔ audio sync — whether the muted video visually looks right against the real song — can only be confirmed by Esteban watching it on the real site, not by the reviewer's automated tests. The reviewer will verify everything that's independently checkable without loading real YouTube content: the data/guard logic (right track gets the player, others don't), that play/pause/switching/backgrounding call the right functions without errors, and that Mission 15/16 haven't regressed. Code's response file should say plainly that visual sync correctness needs Esteban's own eyes/ears on the live site, not claim it as verified from the sandbox.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_16_real-audio-lado-a.md`.

1. **Brand fidelity rule.** No new colors/fonts/section names in this mission.
2. **Verification over self-report.** See the verification-limit note above — be explicit about what was and wasn't actually confirmed.
3. **Placeholder discipline.** Every track without a `videoId` stays unambiguously in its placeholder state.
4. **Mission boundary discipline.** Don't add video for other tracks, seeking, karaoke sync, or the persistent player — all explicitly out of scope.
5. **Static-first constraint.** No server/API/database dependency — the YouTube IFrame API is a client-side script.
6. **No unlicensed third-party assets.** The YouTube IFrame API load is the standard, official embedding mechanism, not a copied snippet or an extra font/icon pack — see the reasoning under Scope (in), item 2.
7. **Diff before review.** Diffed against this mission's acceptance criteria, not eyeballed for vibes.
8. **Mission handoff protocol.** Cowork places this mission file in `missions/` locally, uncommitted. Code commits and pushes it first, confirms the push, *then* executes scope, writes a **separate** response file at `missions/MISSION_17_response.md`, and commits and pushes that too.
9. **Mobile/responsive by default.** Checked at 320px, 375px, 768px, plus Mission 15's desktop-fullscreen case — no regression.

## Open Questions for Esteban (Code should ask, not assume)

None — video, offset, and sync approach are all settled above.
