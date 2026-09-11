# Mission 16 — Real Audio: "Tarde En La Mañana" + "El Ayer" (Lado A, first cassette)

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Wire the first two real songs into the player: real `<audio>` playback, real transport controls (play/pause, prev/next, live scrubber), the Media Session API (lock-screen/notification-shade controls + metadata), and background/lock-screen audio continuation. This is the follow-up mission Mission 14 explicitly deferred pending real audio files — Esteban has now provided them.

## Assets provided (already placed in the repo by Cowork, uncommitted — commit them as part of this mission)

- `public/audio/tarde-en-la-manana.mp3` — "Tarde En La Mañana", 305.81s (**5:06**), 320kbps.
- `public/audio/el-ayer.mp3` — "El Ayer", 186.07s (**3:06**), 320kbps.

No transcoding/compression needed for this mission.

## Data changes (`src/pages/lado-a-lado-b.astro`)

Esteban's explicit decision: rename this cassette's location tag from "cerro chirripó" to "Audiocassettes" everywhere it appears on this cassette (title and every track's `loc`), and put both real songs in its lado A, replacing the two placeholder tracks there. The `id: "cerro"` key itself is internal/not user-visible — leave it unchanged.

Update the first cassette entry to exactly:

```js
{
  id: "cerro",
  title: "grabaciones — Audiocassettes",
  loc: "las cintas · locación 1",
  a: [
    {
      title: "Tarde En La Mañana",
      loc: "Audiocassettes",
      dur: "5:06",
      audioSrc: "/audio/tarde-en-la-manana.mp3",
    },
    {
      title: "El Ayer",
      loc: "Audiocassettes",
      dur: "3:06",
      audioSrc: "/audio/el-ayer.mp3",
    },
  ],
  b: [{ title: "Niebla", loc: "Audiocassettes", dur: "4:05" }],
}
```

**Judgment call, flagged for Esteban:** "Niebla" (lado B, still a placeholder track with no real audio) had its `loc` renamed from "cerro chirripó" to "Audiocassettes" too, purely for consistency within the same cassette — Esteban only explicitly asked about lado A. If he'd rather leave "Niebla" as "cerro chirripó" until it also gets real audio, that's a one-line revert. Note this explicitly in the response file so it's easy to spot and undo.

The other three cassettes are untouched.

## Real lyrics (lyrics drawer, these two tracks only)

Render these as real `.cst-lyric-line` paragraphs (one per line, blank lines as spacer paragraphs — same markup pattern already used for the placeholder lyrics), **replacing** the "letra de ejemplo, sin sincronización" placeholder note for these two tracks only (that note stays on every other still-placeholder track). No time-sync — still a simple expandable panel, per the standing decision.

**Tarde En La Mañana:**
```
En la mesa te deje las llaves
Los cigarros se quedaron en la acera
Era ya tarde en la mañana
Y olvide decirte que te amaba

Conte los pasos mientras me alejaba
Tal vez por las maletas o por tu mirada
Cada paso se me hacía más largo
Cada recuerdo se pasaba a mi conciencia

Y deje la cama sin tender
Tal vez por la mentira de volver
Y deje mi reflejo en el espejo
No te puedes deshacer de mi

Yo no quiero despertarte
Quiero que te quedes en mi cama
Que me busques en la Ventana
Que me esperes hasta que vuelva a abrir la puerta

Tal vez no nos confiamos las verdades
A veces la vida pasa más a prisa
De lo que tardan en pasar nuestras tragedias
```

**El Ayer:**
```
Fueron quedando vacías las calles de mi ciudad
Donde aún resuenan las fiestas de las que ya no hay mas

Con la mirada perdida me encontraron otra vez
Yo estaba pensando que siempre es mejor el ayer

Yo quiero todo de vuelta me lo quitaron sin darme cuenta
Yo quiero vivirlo todo para asegurarme de que si es mejor el ayer

Se fueron perdiendo cosas canciones de alguna persona o fotos que ya no recuerdo

Yo quiero todo de vuelta me lo quitaron sin darme cuenta
Yo quiero vivirlo todo para asegurarme de que si es mejor…

El ayer que ya nunca volverá, ya lo sé, ya nunca volverá

Yo quiero todo de vuelta me lo quitaron sin darme cuenta
Yo quiero vivirlo todo para asegurarme de que si es mejor…
```

## Scope (in)

1. **Extend the `Track` type** (in `CassettePlayer.astro`) with an optional `audioSrc?: string`. Keep following the established pattern of not shipping a parallel JS data blob for content already rendered server-side — real lyrics for these two tracks are rendered server-side per track (e.g. a lyrics block per track, toggled the same `[hidden]`/data-index way `ViewMasterShelf.astro` and this component already do for cassette panels), not reconstructed from a data attribute string.
2. **One real `<audio>` element**, reused across track opens (not recreated per click). When a track row with an `audioSrc` is clicked: set `audio.src`, call `audio.play()` inside that same click handler (it's a real user gesture — needed for autoplay policies), and swap in that track's real lyrics block.
3. **Real transport controls for tracks with `audioSrc`:**
   - Play/pause button and the tap-overlay reflect and drive real `audio.play()`/`audio.pause()` — icon state driven by the audio element's actual `play`/`pause`/`ended` events, not a synthetic flag.
   - Scrubber fill and the elapsed-time label update live from `timeupdate` against `audio.currentTime`/`audio.duration`. The static `dur` value remains the fallback/initial display before metadata loads.
   - Prev/next buttons switch between "Tarde En La Mañana" and "El Ayer" (the only two real tracks right now) — loads the other track's `audioSrc`/lyrics/title/loc and starts it playing, same as clicking its row would.
4. **Tracks without `audioSrc` are unaffected** — they keep exactly Mission 14/15's visual-only behavior (synthetic play/pause toggle, static duration, placeholder lyrics note, decorative prev/next). Guard all new real-audio code on `audioSrc` being present so opening a placeholder track never throws or tries to play an empty src.
5. **Media Session API**, only while a track with `audioSrc` is loaded:
   - `navigator.mediaSession.metadata = new MediaMetadata({ title, artist: "Audiocassettes", album: <cassette title> })`. No `artwork` array — there's no real cover art yet (same deferral as the cassette-box redesign); omitting it is expected and fine, not a bug.
   - Action handlers for `play`, `pause`, `previoustrack`, `nexttrack` at minimum, wired to the same logic as the on-page buttons (don't duplicate the logic — call the same functions). `seekbackward`/`seekforward` are a nice-to-have, not required.
   - `navigator.mediaSession.playbackState` kept in sync ('playing'/'paused') so lock-screen controls show the right state.
   - `setPositionState({ duration, position, playbackRate })` on `timeupdate` if straightforward — nice-to-have for a synced lock-screen scrubber, not a hard requirement.
6. **Background/lock-screen continuation**: this should fall out naturally from using a real, actually-playing `<audio>` element + Media Session — do not add Wake Lock API, a Web Audio API graph, or any other "keep-alive" mechanism; none of that is needed for audio (unlike video) and would be scope creep.
7. Navigating **within this same page** (e.g. tapping the back arrow from player → cassette screen) must **not** pause or reset the audio — the whole point is that it keeps playing while the user does something else. Only the pause button, the track ending, or switching to a different track should stop/change it.

## Known, explicitly-accepted boundary (not a bug, don't try to solve it here)

Playback does **not** persist across a full page navigation to a *different* page on the site (e.g. tapping to `el-estuche` while a song is playing) — Astro's page swap unmounts this component's DOM, including the `<audio>` element. Making playback survive full page navigation would mean hoisting the player into a persistent, global layout-level component — a meaningfully bigger change than this mission, and not requested. Flag this plainly in the response file as a known limitation, not something quietly worked around.

## Scope (out — do not touch in this mission)

- No cassette-box visual redesign, no real video, no karaoke/time-synced lyrics highlighting (lyrics are a real static block now, but still not synced — no per-line timestamps).
- No changes to the other three cassettes or to lado B of this one beyond the `loc` rename noted above.
- No Wake Lock API, no Web Audio API, no persistent-across-navigation player.
- No new npm dependencies (the Media Session API and `<audio>` element are both plain browser APIs).
- No audio transcoding/compression.

## Acceptance Criteria

- [ ] `public/audio/tarde-en-la-manana.mp3` and `public/audio/el-ayer.mp3` are committed and served; both play back correctly end-to-end.
- [ ] Opening "Tarde En La Mañana" or "El Ayer" from the cassette screen starts real playback immediately (a real `play` event fires, `audio.paused === false`), with the correct title/loc/duration shown.
- [ ] The play/pause button and tap-overlay reflect true playback state — verified by pausing via the button and confirming `audio.paused === true`, then resuming and confirming `audio.paused === false`.
- [ ] The scrubber fill and elapsed-time label visibly advance during playback (verified by sampling `audio.currentTime` and the rendered scrubber width at two points in time, confirming both increased).
- [ ] Prev/next correctly switches between the two real tracks and starts the new one playing.
- [ ] Opening any of the other (still-placeholder) tracks does not throw a console error and behaves exactly as it did after Mission 15 — no real playback attempted.
- [ ] `navigator.mediaSession.metadata.title` matches the currently loaded real track; `playbackState` matches actual play/pause state; the `play`/`pause`/`previoustrack`/`nexttrack` action handlers are registered and call the same logic as the on-page controls (verified by reading the diff, and by invoking a handler programmatically and confirming the same state change the corresponding button click produces).
- [ ] Tapping the back arrow from the player screen while a real track is playing does **not** pause it — confirmed by checking `audio.paused === false` after navigating back to the cassette screen.
- [ ] The lyrics drawer for these two tracks shows the real lyrics text above (verbatim, correct line breaks); every other track still shows the placeholder sample note.
- [ ] The "cerro" cassette's title, `las-cintas`-facing display, and lado A/B track locations read "Audiocassettes" per the data block above (with the "Niebla" judgment call flagged in the response file).
- [ ] No horizontal overflow and all elements remain reachable without scrolling at 320px/375px/768px and at a representative desktop size (Rule 9 + Mission 15's desktop-fullscreen regression check) — re-verify fullscreen still works correctly with real audio playing.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.
- [ ] Diff is scoped to `CassettePlayer.astro`, `lado-a-lado-b.astro`, and the two new audio files — no unrelated changes.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_15_fullscreen-fix.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in a mission document — never invented or approximated.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted. This is exactly why this mission waited for real audio files before being written.
3. **Placeholder discipline.** Every track/cassette still without real audio must remain clearly in its existing placeholder state — no partial, ambiguous real-ification.
4. **Mission boundary discipline.** Don't get ahead into cover-art/cassette-box redesign, karaoke sync, or persistent-across-navigation playback — all explicitly out of scope above.
5. **Static-first constraint.** No server/API/database dependency introduced — the Media Session API and `<audio>` element are both plain client-side browser APIs.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against this mission's acceptance criteria — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it, plus the two audio files, in the project's `missions/`/`public/audio/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and audio files first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_16_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px — this mission additionally must not regress Mission 15's fullscreen fix.

## Open Questions for Esteban (Code should ask, not assume)

None for this mission's scope — the two songs, their lyrics, their lado A placement, and the "Audiocassettes" rename are all settled above. The "Niebla" `loc` rename is a flagged judgment call, not an open question requiring an answer before proceeding.
