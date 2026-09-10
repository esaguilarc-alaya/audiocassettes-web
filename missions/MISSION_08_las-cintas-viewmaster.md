# Mission 08 — Las Cintas: View-Master Disk Shelf & Viewer

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Restructure `las-cintas` from a page of always-open `Reel` components into a two-level interaction: a shelf of View-Master-style disks (one per session/location), and a viewer you zoom into by clicking a disk, where the existing lever/back-tap interaction cycles through that disk's frames as previews, and clicking the current preview opens it full-size. This also resolves the standing backlog item that the Reel's visual presentation should actually look like a real View-Master toy, not an abstract card — the disk and viewer are that visual moment now.

## Reference artifact (flow only — not visuals)

`reference/las-cintas-viewmaster-flow-reference.html` (added by this mission) is a clickable mockup Esteban reviewed and approved, confirming the **interaction flow**: a shelf of disks → click a disk → zoom into a viewer → lever advances / left-half-of-preview goes back → click the current preview → opens full → Escape or an explicit close control returns to the shelf.

Two things to be explicit about:

- **The flow in that reference is settled and should be ported faithfully** — the shelf/viewer/full-open/close sequence, the lever and left-half-back gestures (unchanged from Mission 02/03), the frame-counter dots, and looping (see Scope item 4).
- **The reference's disk/lever/viewer *visuals* are explicit placeholders and are NOT the target.** Esteban was clear the mockup doesn't attempt to look like a real disc or lever, and that's fine — the real design intent (noted in the project backlog since Mission 03) is that the disk and viewer should evoke an actual View-Master toy: a disc with a labeled hub, and a viewer body with an oval eyepiece housing framing the current frame. Use your own design judgment for the visual execution, bounded by Rule 1 below (brand palette only, no invented colors).

Open `reference/las-cintas-viewmaster-flow-reference.html` directly in a browser and click through it to see the approved flow before implementing.

## Scope (in)

1. **Disk shelf.** `las-cintas.astro` shows a shelf/grid of disks, one per existing placeholder session (reuse the placeholder location names and frame content already in place from Mission 02/03 — do not invent new placeholder content). Each disk is labeled with its session's (placeholder) location name and styled to evoke a real View-Master disc (a labeled hub, a disc silhouette — your design judgment on exact execution). Assign each disk an accent color by cycling deterministically through the established palette tokens already used elsewhere in this project (`--view-master`, `--retro-pop`, `--polaroid-sunset`, `--brown-tape`, `--stereo-blue`) in a fixed order — no invented or approximated colors, per Rule 1.

2. **Viewer.** Clicking a disk transitions (a zoom/scale-in feel, per the reference) into a viewer showing that disk's first frame. The viewer's body should read as a real View-Master viewer — an eyepiece housing framing the current frame, a body silhouette — using established brand tokens for its coloring (your judgment on which token(s) fit best; flag the choice in the response file per Rule 1).

3. **Frame interaction inside the viewer — unchanged mechanics.** The existing lever-click-advances and left-half-of-frame-click-goes-back interactions (Mission 02/03) work exactly as before, cycling through that disk's frames, with the existing frame-counter dots reflecting position.

4. **Looping.** Advancing past the last frame returns to frame 1; going back past frame 1 goes to the last frame. No dead ends in either direction.

5. **Opening a frame full.** Clicking the current preview frame opens it full-size:
   - Video frames: the existing Mission 02 branded modal (brown-tape frame border, view-master-red close button, embedded YouTube iframe) — unchanged.
   - Photo frames: a new full-view overlay using the same branded visual treatment (brown-tape border, view-master-red close button) for consistency with the video modal, showing the (placeholder) photo at full size.

6. **Closing the viewer.** Both the Escape key and an explicit close control (a visible "✕"/close button — not just a text link) return from the viewer to the disk shelf, from any frame position.

7. **Home-page teaser link.** `LasCintasPreview.astro` keeps its current visual appearance unchanged, but its link(s) must go to `/las-cintas/` (the disk shelf) rather than any specific individual disk/reel deep-link — confirm and fix if it currently points elsewhere.

8. **No page navigation for the shelf↔viewer interaction.** Opening/closing a disk is in-page state (like the existing lever/modal interactions), not a route change — it must not trigger a URL change or the site's rewind page-transition effect (Missions 04–07). That effect only fires for actual page navigations (e.g. clicking to `/las-cintas/` itself from another page).

## Scope (out — do not touch in this mission)

- No real photos, video, or audio content — placeholders only, per standing Rule 3, and still obviously placeholders (not plausible-enough-to-ship-by-accident).
- No changes to nav, other pages, hero copy, or the Mission 02 per-page `--page-accent` bar mapping (that's a distinct, page-level pattern — this mission's per-disk coloring is separate and scoped only to `las-cintas`'s disks/viewer).
- No changes to the rewind page-transition effect or `TransitionOverlay.astro` (Missions 04–07) — those stay exactly as shipped.
- No new npm dependencies.
- No sound.

## Acceptance Criteria

- [ ] `las-cintas` shows a shelf of View-Master-style disks, one per existing placeholder session, each labeled with its placeholder location name, colored via the established brand palette cycled deterministically per disk (no invented colors).
- [ ] Clicking a disk transitions into a viewer styled to evoke a real View-Master toy (an eyepiece housing framing the frame, a viewer body) — not a plain box — showing that disk's first frame.
- [ ] Lever-forward and left-half-back interactions work exactly as before per disk's frame sequence, with frame-counter dots reflecting position.
- [ ] Looping confirmed both directions: past the last frame goes to frame 1; before frame 1 goes to the last frame.
- [ ] Clicking the current frame's preview opens it full: video → existing branded modal with iframe, unchanged; photo → new branded full-view overlay with the same brown-tape/view-master-red visual treatment.
- [ ] Both Escape and an explicit close control return from the viewer to the shelf, from any frame position, with no dead ends.
- [ ] `LasCintasPreview.astro` links go to `/las-cintas/`; its own appearance is unchanged from before this mission.
- [ ] Opening/closing a disk causes no URL change and does not trigger the rewind page-transition effect — confirmed by checking the URL and by confirming no view-transition/ClientRouter navigation event fires on disk open/close.
- [ ] All placeholder content remains obviously placeholder — no new invented-but-plausible content added.
- [ ] Renders cleanly with no jank or overflow at 320px, 375px, and 768px (Rule 9), including the disk shelf grid and the viewer's eyepiece/lever controls staying fully usable and on-screen at 320px.
- [ ] `npm run build` succeeds with zero errors; no new `package.json`/`package-lock.json` entries.
- [ ] Spot-check: nothing from Missions 04–07's rewind transition, or Mission 02's per-page accent bar mapping, was altered.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_07_transition-fx-start-timing.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception. The per-disk color cycling in this mission uses only the established palette tokens, in a fixed deterministic order, per this mission's own scope.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Anything standing in for real content must be obviously a placeholder — never plausible enough to ship as real by accident.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against the reference's flow (not its visuals) and checked against Mission 02/03's untouched lever/back-tap mechanics, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the reference file) in the project's `missions/`/`reference/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and reference file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None on the interaction flow — it's settled by the reference. On the visual execution of the disk/viewer (exact shapes, exact token choices for the viewer body), use your own design judgment bounded by Rule 1 and the "real View-Master toy" intent described above; record the choices made in the response file rather than asking mid-mission. If something about porting the existing lever/back-tap/modal mechanics into the new shelf/viewer structure creates a genuine ambiguity not covered above, ask rather than guess.
