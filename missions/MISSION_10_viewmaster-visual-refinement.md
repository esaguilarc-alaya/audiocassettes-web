# Mission 10 — Las Cintas: View-Master Disc & Viewer Visual Refinement

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Mission 08 shipped the shelf/viewer interaction flow correctly, but Esteban reviewed it live and flagged that the disc and viewer don't read as an authentic View-Master toy: the disc's photo windows all face the same (upright) direction with an odd radial-line pattern between them, and the lever sits centered below the eyepiece instead of mounted on the viewer's body. This mission replaces only the disc/viewer/lever **visuals** in `ViewMasterShelf.astro` to match an approved reference — the shelf→viewer→full-open→close interaction flow, looping, dots, and left-half-back gesture from Mission 08 are correct and unchanged.

## Reference artifact (visuals — settled this time)

`reference/las-cintas-viewmaster-look-reference.html` (added by this mission) is a mockup Esteban reviewed through several rounds — including comparing directly against a real View-Master disc photo — and approved. Unlike Mission 08's reference (flow only, visuals explicitly not to be ported), **this reference's visuals are the settled target.** Port them faithfully:

- **Disc:** a ring of small rectangular "photo window" swatches (7 per disc) arranged around the disc face, each one rotated by its own position angle around the circle with **no counter-rotation** — this is what makes them progressively tilt around the ring instead of all staying upright, matching a real disc. Small rim notches between them (no radial line pattern across the face — that was explicitly wrong in Mission 08's shipped version and must not reappear). A center hub area showing, as plain (non-arced) text: a small "▲" orientation mark, a small punched hole, the disc's title, and its frame count.
- **Sizing:** the photo windows should read as small relative to the disc (per Esteban's explicit "smaller squares so it doesn't feel so tight" feedback) — proportion them similarly to the reference (window size noticeably smaller than the disc radius, with visible gaps between adjacent windows), scaled to whatever disc diameter the real component already uses, not copied pixel-for-pixel.
- **Viewer:** the classic View-Master silhouette — a top housing with two circular "eyecup" shapes, above a lower body. The lever moves from its current position (centered below the eyepiece, next to the dots) to **mounted on the right side of the viewer body**, angled, and visibly presses/rotates downward on activation (click or tap) before springing back — matching the reference's `.vm-lever-mount`/`.vm-lever` treatment.

Open `reference/las-cintas-viewmaster-look-reference.html` directly in a browser to see the approved look before implementing.

## Scope (in)

1. **Disc restyling** in `ViewMasterShelf.astro`'s `.vm-disk` (or equivalent): replace the current plain-circle-with-hub look with the reference's ring-of-photo-windows-plus-center-text treatment described above. Keep the existing `DISK_COLORS` deterministic cycling (Rule 1) — the photo-window swatches use the disc's assigned color the same way the current hub does now.
2. **Viewer restyling**: replace the current flat `.vm-viewer__body` look with the twin-eyecup-housing-over-body silhouette. Reposition the lever to the right side of the viewer body per the reference, keeping its existing click handler and "advance to next frame" behavior — only its position, shape, and press animation change, not its function.
3. **Colors**: continue using only established brand tokens for the disc/viewer body coloring (as Mission 08 already did — `--view-master` for the viewer body, `--vintage-sky` for the disc face, `--brown-tape` accents). The rim notches, punch hole, and eyecup interiors may use the same plain black neutral Mission 08 already established as precedent for the video modal's embed background (`background: black`) — not a new invented color, per Rule 1.
4. **No mechanics changes.** The lever's click-advances behavior, the eyepiece's left-half-click-goes-back gesture, frame-counter dots, looping (past-last→first, before-first→last), opening a frame full-size, and Escape/✕-closes-to-shelf all stay exactly as Mission 08 shipped them. This mission only restyles the disc, the viewer chrome, and the lever's position/shape/press animation.

## Scope (out — do not touch in this mission)

- No changes to the shelf↔viewer interaction flow, looping, dots logic, or full-view/modal behavior (Mission 08).
- No changes to `LasCintasPreview.astro`, other pages, nav, or the `notas-de-cinta` bio work (Mission 09).
- No changes to the rewind page-transition effect (Missions 04–07).
- No new npm dependencies.
- No sound.

## Acceptance Criteria

- [ ] Each disc shows 7 photo-window swatches arranged in a ring, each rotated by its own position angle with no counter-rotation (verified by inspecting the actual computed `transform` per window, not just eyeballed) — no radial-line pattern anywhere on the disc face.
- [ ] The photo windows are visibly smaller relative to the disc than Mission 08's shipped hub, with clear gaps between adjacent windows.
- [ ] Each disc's center shows, as plain text (not arced/curved): an orientation mark, a small hole, the disc's title, and its frame count.
- [ ] The viewer shows a twin-eyecup housing above a body (not a flat rectangle), and the lever is positioned on the right side of the viewer body, angled, with a visible press/rotate animation on activation.
- [ ] Lever-click-advances, left-half-click-goes-back, frame-counter dots, looping both directions, opening a frame full-size (video → existing modal, photo → existing full-view overlay), and Escape/✕-closes-to-shelf all still work exactly as before — re-verified, not assumed unchanged.
- [ ] `DISK_COLORS` cycling and all other established brand-token usage are unchanged; no invented colors.
- [ ] `LasCintasPreview.astro`, other pages, and Missions 04–09's work are untouched.
- [ ] Renders cleanly with no jank or overflow at 320px, 375px, and 768px (Rule 9), including the disc's photo-window ring and the viewer's eyecups/lever staying legible and on-screen at 320px.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_09_notas-de-cinta-bio.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception. The disc/viewer's neutral (black) accents follow the precedent already recorded in Mission 08's response file.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Not applicable to this mission's changes.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against the reference and re-checked against Mission 08's untouched mechanics, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the reference file) in the project's `missions/`/`reference/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and reference file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None — the reference's visuals are the settled spec this time. If porting the exact rotation math or lever geometry onto the real component's existing disc/viewer dimensions creates a genuine ambiguity not covered above, use your own judgment to match the reference's proportions and record the choice in the response file.
