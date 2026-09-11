# Mission 20 — rotating 3D cassette carousel ("nuestras cintas" shelf)

## Context

Esteban reviewed 10 iterations of a redesign for the Screen 1 shelf
(currently `.cst-grid` / `.cst-box` in `CassettePlayer.astro` — a flat
grid of colored boxes) against a real photo of a cassette-holder rack he
provided, and gave final, explicit approval on the last one. The
approved reference is attached to this mission as
`reference/nuestras-cintas-carousel-reference.html` — a standalone HTML
file (open it directly in a browser) that must be ported faithfully:
same 16-slot layout, same permanent corner-viewing base angle, same
per-cassette "box popping out of its slot" visual, same turn-button +
drag-to-rotate interaction with snap-to-nearest-face.

This mission replaces Screen 1's shelf ONLY. Screens 2 (open
cassette/tracklist) and 3 (player) are explicitly out of scope and must
not be touched — tapping a real cassette must still open Screen 2 exactly
as it does today.

## Scope (in)

1. **Replace the shelf markup.** In `CassettePlayer.astro`, replace the
   current `<div class="cst-grid">...cassettes.map(...)</div>` block
   (Screen 1 only) with the carousel structure from the reference file:
   a `.stage` > `.carousel` > four `.face-0..3` elements, each holding
   four `.slot` > `.spine` (button) entries, plus the `.lid`, turn
   buttons (`#turnLeft`/`#turnRight` — rename IDs with an `cst` prefix
   for consistency with this file's existing naming, e.g.
   `#cstCarouselTurnLeft`/`#cstCarouselTurnRight`), and the face-dots
   indicator.

2. **16 total slots, one data model.** Face 0 holds the 4 real
   cassettes, in the exact order `cassettes` is already passed to this
   component (do not reorder or re-map — the existing `data-cassette-index`
   click → `openCassette(index)` wiring must keep working unchanged).
   Faces 1–3 hold 12 inert placeholder slots. Represent a placeholder
   slot as **markup only, not a new cassette data field** — do NOT add
   fake entries to the `cassettes` array or invent a "placeholder
   cassette" object. Server-render the 12 placeholder `.slot
   .placeholder-slot` blocks directly in the template (a simple
   `Array.from({ length: 12 })` loop is fine), each showing the label
   "próximamente" exactly as the reference does, with no `data-cassette-index`
   and no click handler — clicking one must do nothing (not even a
   visual press state beyond what plain CSS `:hover`/`:active` gives for
   free; don't wire a JS click listener to it at all).

3. **Real cassette display titles**, for the 4 real cassettes only, in
   this exact order/text (these are label overrides for the carousel
   spine only — do NOT rename anything in the `cassettes` data itself,
   `c.title` stays what it is today and keeps driving Screen 2's
   eyebrow/hero exactly as now):
   - "Audiocassettes"
   - "Reel Real"
   - "En vivo - Montserrat, Coronado"
   - "En vivo - Calle Blancos"

   Render these as a small local array in the component (e.g.
   `const CAROUSEL_SPINE_LABELS = [...]`) indexed the same way
   `DISK_COLORS` already is (`labels[i % labels.length]`, though with
   exactly 4 entries for 4 cassettes there's no wraparound today) — used
   only for the `.spine-title` text on the carousel button, nothing else.
   Each real spine keeps using the existing `DISK_COLORS[i % DISK_COLORS.length]`
   token for its `--box-color`-equivalent background (reuse the same
   `color-mix(in srgb, var(--token) 80%, black 10%)` /
   `color-mix(in srgb, var(--token) 55%, black 45%)` pair the reference
   uses for `.spine-face` / `.spine-side`, driven off that per-cassette
   CSS custom property exactly like the current `.cst-box` does with
   `--box-color`).

4. **Interaction, ported from the reference verbatim**: turn-button
   click (`turnLeft`/`turnRight`), drag-to-rotate via Pointer Events
   (this codebase's established pattern — see the Mission 18 scrubber
   and this reference's own use of `pointerdown`/`pointermove`/`pointerup`,
   not separate mouse/touch handlers) with snap-to-nearest-face on
   release, the permanent `BASE_ANGLE = 24` corner-viewing offset, and
   face-dot indicator sync. Keep the reference's constants
   (`FACE_ANGLE = 90`, `BASE_ANGLE = 24`) unless a real-device test shows
   they need adjusting for this site's actual card width — if you do
   change them, say so and why in the response file.

5. **Keep the click-to-open-cassette wiring working.** The real
   cassettes' `.spine` buttons must carry the same `data-cassette-index`
   attribute the current `.cst-box` buttons do, and the existing
   `root.querySelectorAll<HTMLButtonElement>(".cst-box")` click-listener
   block must be updated to select the new real-cassette spine buttons
   instead (e.g. give them an additional class like `.cst-spine` and
   select `.cst-spine[data-cassette-index]`, or reuse `.cst-box` as an
   additional class on the real spines — your call, just don't silently
   drop this wiring or leave it selecting a class that no longer exists).
   Placeholder spines must NOT match this selector.

6. **CRITICAL — the `preserve-3d` chain.** While building this same
   carousel as an interactive demo, I hit a real, easy-to-reproduce CSS
   bug: `transform-style: preserve-3d` was set on the outer `.carousel`
   but NOT on the intermediate `.face`/`.slot` wrappers, which silently
   flattened every `.spine`'s `translateZ()` pop-out onto the `.face`'s
   own 2D plane — the cassette boxes rendered dead flat with zero
   visible error. The reference file already has this fixed (`.face`,
   `.slot`, and `.spine` all carry `transform-style: preserve-3d`) — when
   porting the CSS, copy those three declarations exactly, don't drop
   any of them as "redundant," and if you touch this styling further,
   re-verify the pop-out is still visible (it should look like each
   cassette box visibly sits proud of its slot, more so on hover/focus,
   less on `:active`) rather than trusting that the transform values
   alone are enough.

7. **Namespacing.** Prefix every new class/ID this mission introduces
   with `cst-` (matching this file's existing convention) rather than
   copying the reference's bare class names (`.stage`, `.carousel`,
   `.face`, `.slot`, `.spine`, etc. verbatim) — e.g. `.cst-carousel-stage`,
   `.cst-carousel`, `.cst-face`, `.cst-slot`, `.cst-spine`, and so on.
   Keep the reference's actual CSS values (dimensions, angles, colors,
   transitions) — only the naming needs to change to fit this codebase.

8. **Remove the old shelf** (`.cst-grid`, `.cst-box` and its
   sub-elements/CSS/JS) once the carousel fully replaces it — don't leave
   dead code behind, and don't leave both shelves rendering at once.

9. **Add the reference file to the repo** at
   `reference/nuestras-cintas-carousel-reference.html` (same convention
   as the other approved reference files already in that folder) so it's
   preserved as the design record — it ships alongside this mission file.

## Scope (out)

- Screens 2 and 3 (cassette detail panel, player) — do not touch their
  markup, styles, or behavior. This mission is Screen 1 only.
- No changes to the `cassettes`/`Cassette`/`Track` data model, and no
  changes to `lado-a-lado-b.astro`.
- No changes to any Mission 15–19 behavior (fullscreen, real audio/video
  sync, scrubber, click-shield) — these live entirely in Screens 2/3 and
  should be unaffected by this mission, but call this out explicitly in
  your response file's regression check.
- No "View Master" video-content section — that's a separate, later
  feature Esteban has only mentioned, not requested yet.
- No changes to the carousel's visual design beyond what's needed to
  port it faithfully (namespacing classes, wiring real data) — this is
  an implementation mission, not another design round. If something in
  the reference looks wrong once it's real (e.g. spacing at very narrow
  widths), fix it minimally and flag it in the response file rather than
  redesigning.

## Acceptance criteria

- Screen 1 shows the rotating carousel, not the old grid.
- All 4 real cassettes appear on face 0, in original order, with the
  exact display titles listed in item 3, each still tappable and opening
  the correct Screen 2 cassette (verify by title/tracklist match, not
  just that *a* screen opens).
- Faces 1–3 show 12 total "próximamente" placeholder slots, inert
  (no click behavior, no cassette opens).
- Turn buttons rotate the carousel one face at a time, looping in both
  directions; the face-dot indicator reflects the current face.
- Dragging (mouse and touch/pointer) rotates the carousel proportionally
  and snaps to the nearest face on release.
- The permanent base-angle offset is present — the carousel never rests
  perfectly flat-on; the edge of the adjacent face is always visible.
- Each cassette box visibly "pops out" of its slot (test this
  specifically — take a screenshot or inspect computed styles — don't
  just confirm the CSS properties are present, confirm the 3D effect is
  actually visible, given the `preserve-3d`-chain bug described above).
- Verified at 320px, 375px, and 768px viewport widths (Rule 9) — the
  carousel and its controls must remain usable and not overflow/clip at
  the narrowest width.
- Full regression pass on Screens 2/3: opening a cassette, switching
  lado a/lado b, opening the real-audio tracks ("Tarde En La Mañana" /
  "El Ayer"), fullscreen toggle, scrubber drag, and the Mission 19
  click-shield over the YouTube embed all still work exactly as before —
  this mission must not have touched any of that code, but confirm it
  wasn't accidentally broken (e.g. by a shared CSS selector collision).
- `npm run build` succeeds with no new errors/warnings.

## Standing Engineering Control Rules (apply to every mission)

1. **Brand fidelity** — only the 6 established brand tokens
   (`--brown-tape`, `--vintage-sky`, `--polaroid-sunset`, `--view-master`,
   `--retro-pop`, `--stereo-blue`) and their `color-mix()` derivatives;
   no new hex values invented.
2. **Verification over self-report** — your response file's claims will
   be independently re-verified against the real repo/build/browser
   behavior, not taken on faith.
3. **Placeholder discipline** — clearly mark any remaining
   placeholder/sample content as such; never present placeholder data as
   real.
4. **Mission boundary discipline** — implement exactly this mission's
   Scope (in); do not fix unrelated things you notice, do not touch
   Scope (out) items, even if it looks convenient.
5. **Static-first constraint** — no server/build-time dependency beyond
   what Astro's static output already uses.
6. **No unlicensed third-party assets** — no new fonts/icons/images/libs
   beyond what's already approved in this codebase.
7. **Diff before review** — self-review your own `git diff` before
   writing the response file; don't rely on memory of what you intended
   to change.
8. **Mission handoff protocol** — commit and push this mission file
   (and the reference file) FIRST, confirm the push succeeded, THEN
   execute the mission's scope, THEN write a separate
   `missions/MISSION_20_response.md` and commit+push that separately.
9. **Mobile/responsive by default** — verify at 320px, 375px, and 768px
   viewport widths for anything touching layout.
