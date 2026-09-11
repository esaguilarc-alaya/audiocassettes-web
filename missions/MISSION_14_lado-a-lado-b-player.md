# Mission 14 — Lado A / Lado B: Cassette Shelf + Full Player

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Replace the `lado-a-lado-b` placeholder with the full flow Esteban approved through an interactive demo: a shelf of closed cassette boxes → tap one to open it and see its "lado a" / "lado b" tracklist → tap a track to open a full-screen, Spotify-style player (video up top, transport controls, an expandable lyrics panel, and a fullscreen toggle).

**Scope note (important):** this mission ships the full interaction and visual structure with placeholder content (sample cassettes/tracks/lyrics, clearly marked). It does **not** include background/lock-screen audio continuation (Media Session API) — that needs at least one real audio file to build and test honestly, and is planned as a follow-up mission once Esteban provides one. Building it now against fake/silent audio would mean shipping something never verified against real behavior, which violates Rule 2. Flag this omission clearly in the response file.

## Reference artifact

`reference/lado-a-lado-b-player-flow-reference.html` (added by this mission) is an interactive demo Esteban reviewed and approved — the shelf → cassette → player flow, screen transitions, cassette box visual (colored box, spinning-reel graphic, label), lado a/b toggle, tracklist rows, and the player screen (video placeholder, transport controls, scrubber, fullscreen button, expandable lyrics drawer) are all settled. Open it directly in a browser to see the approved flow before implementing.

**One visual piece is explicitly NOT settled and Esteban already knows it**: the closed cassette box's look (currently an abstract colored rectangle with a spinning-reel graphic) is a placeholder — he wants to swap it for something that reads as an actual cassette case with per-cassette cover art later. Port the reference's cassette box faithfully as the placeholder for this mission; do not try to make it more "realistic" on your own judgment — that visual-authenticity pass is planned as its own follow-up mission (same pattern as the View-Master disc/viewer, Missions 08→10).

## Scope (in)

1. **Shelf screen**: replace the `lado-a-lado-b` placeholder with a grid of cassette boxes per the reference's visual (colored box using `DISK_COLORS`-style deterministic brand-token cycling — reuse that established pattern from `ViewMasterShelf.astro`, don't invent a new one). Use 3-4 sample cassettes with placeholder titles/locations (clearly marked as placeholder data, e.g. a comment block or a `// TODO: replace with real cassette data` note) — Esteban will supply real cassette/track data in a later mission once recordings exist.
2. **Cassette screen**: tapping a cassette opens it to show the hero cassette graphic, a lado a / lado b toggle, and a tracklist for the active side. Tapping a track opens the player.
3. **Player screen**: video area (use the reference's placeholder visual — a bordered "viewfinder" box with placeholder text — since no real video/audio is wired yet), track title/location, a scrubber (visual only, no real playback state needed yet beyond what the reference shows), transport controls (prev/play-pause/next — visual/interactive state toggle only, matching the reference's demo-level behavior), a working fullscreen toggle (real Fullscreen API, not simulated), and an expandable lyrics drawer (simple slide-up panel, not synced/karaoke — per Esteban's explicit decision to do simple lyrics first) with placeholder lyric lines clearly marked as sample text.
4. **Navigation**: a working back arrow on each screen (cassette → shelf, player → cassette), matching the reference.
5. Use only established brand tokens for all colors (Rule 1) — the reference already does this (verify against it, don't just eyeball).
6. This remains a fully static, client-side-only feature (screen switching, fullscreen toggle, lyrics drawer) — no server/API dependency (Rule 5).

## Scope (out — do not touch in this mission)

- **No real audio or video wiring.** No `<audio>`/`<video>` elements playing real files, no Media Session API integration, no background/lock-screen playback. This is Mission 15+ once Esteban provides real audio.
- **No cassette-box visual-authenticity redesign.** Ship the reference's placeholder box as-is; the "make it look like a real cassette with cover art" pass is a separate future mission (Esteban's own explicit deferral).
- **No time-synced/karaoke lyrics.** Simple expandable panel only, per Esteban's decision.
- No changes to `las-cintas`, `el librito`, `el estuche`, the sintoniza footer, the page-transition effect, or any other page/component.
- No new npm dependencies.

## Acceptance Criteria

- [ ] `lado-a-lado-b` shows a shelf of cassette boxes (3-4 sample entries) styled per the reference, using deterministic brand-token color cycling consistent with the `ViewMasterShelf.astro` precedent.
- [ ] Tapping a cassette opens the cassette screen showing its lado a/lado b toggle and the correct tracklist per side, matching the reference's interaction.
- [ ] Tapping a track opens the player screen with that track's placeholder title/location reflected in the UI (not hardcoded to always show the same demo track).
- [ ] The fullscreen button actually toggles real fullscreen on the player (via the Fullscreen API), verified programmatically (checking `document.fullscreenElement`), not just visually.
- [ ] The lyrics drawer slides open/closed on tap, contains placeholder lyric text clearly marked as sample content, and does not attempt any time-sync.
- [ ] Back navigation works correctly at each screen level.
- [ ] No real audio/video elements, no Media Session code, and no cassette-box redesign were introduced — confirm by reading the diff, not just clicking around.
- [ ] All colors trace to established brand tokens (Rule 1) — no invented hex values.
- [ ] Renders cleanly with no jank or horizontal overflow at 320px, 375px, and 768px (Rule 9) — the reference is a phone-frame mockup; the real page should adapt properly to actual mobile viewports, not just the reference's fixed 420px frame.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_13_el-estuche-merch.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted. This is also why background audio is deferred: it can't be honestly verified without a real audio file.
3. **Placeholder discipline.** Sample cassette/track/lyric data must be clearly marked as placeholder (comments, TODOs) so it's obvious what needs real content later — not left ambiguous.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early (no Media Session, no cassette-box redesign, no karaoke sync).
5. **Static-first constraint.** The site is static by default. This mission's fullscreen toggle and screen-switching are client-side only — no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against the reference and re-checked against the acceptance criteria — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the reference file) in the project's `missions/`/`reference/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and reference file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None for this mission's scope — the flow and visuals are settled via the reference, and the deferred pieces (real audio/background playback, cassette-box redesign, karaoke lyrics) are explicitly out of scope, not open questions. If exact placeholder wording or sample data needs a judgment call, use your own judgment and record it in the response file.
