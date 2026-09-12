# Mission 24 response — liner notes screen + narrow cassette hero bar

## Summary

- Screen 2's old big hero (`.cst-hero`, the large color box with two
  spinning-reel icons) is replaced with a narrow, full-width, tappable
  bar (`.cst-hero-bar`, 4rem tall — shorter than the demo reference's
  own 4.6rem bar), showing the cassette title and a "créditos y
  portada" hint. No `c.loc` on it.
- A new liner notes screen (`#cstScreenLiner`) is reached by tapping
  that bar: full-width placeholder cover art, cassette title (no
  location line), músicos / músicos adicionales / personal técnico,
  then lado A and lado B tracklists padded to 9 total songs (5+4) with
  clearly marked placeholder rows. Its back button returns to Screen 2,
  not the shelf.
- Real credits content for "cerro" ("Audiocassettes") and "playa"
  ("Reel Real"), verbatim from the mission, with every explicitly-left
  gap (cerro's "músicos adicionales"; playa's two missing musicians and
  two unconfirmed track references) rendered as a clear placeholder,
  not invented. "ciudad"/"bosque" render a fully placeholder liner
  screen, driven by their `credits` field being absent from the data.
- `c.loc` ("las cintas · locación X") no longer renders anywhere in the
  app — confirmed by scanning the built HTML output, not just the
  source.
- Screens 1 (carousel) and 3 (player) are untouched; full regression
  pass below confirms both still work exactly as before.

## Data model change

Added optional `credits?: Credits` to the `Cassette` interface
(`CassettePlayer.astro`), where `Credits` has `musicians: string[]`,
optional `additionalMusicians?: CreditPerson[]` +
`additionalMusiciansPlaceholderCount?: number`, and `technical:
string[]`. Populated only for `cerro`/`playa` in `lado-a-lado-b.astro`;
left `undefined` for `ciudad`/`bosque`. Per the mission's own
instruction, this enforces "no invented content" via the data shape
itself — every section in the liner template branches on `c.credits`
being present/absent (and, for "músicos adicionales" specifically, on
whether `additionalMusicians` itself is present and non-empty), so a
cassette with no `credits` object literally cannot render anything but
the placeholder treatment, regardless of what future edits touch the
template.

Content transcribed verbatim from the mission text — I did not carry
over the reference demo's own invented details (it added an assumed
surname "Aguilar" for Esteban and a "(apellido pendiente)" annotation
for Gerson; the mission's own credits text just says "Esteban, Gerson"
with no surname or pending-note mentioned, so that's exactly what's
rendered — two plain names, nothing invented beyond the mission's own
wording).

The tracklist real `cassettes[i].a`/`.b` arrays are completely
unchanged — verified via `git diff` showing zero modifications to any
existing track entry, only new `credits` keys and comments added.

## Tracklist padding

Used a 5 (lado a) + 4 (lado b) = 9 split for every cassette, matching
the reference demo's own split, applied via a new `linerTracksFor(tracks,
target)` helper that takes the cassette's real tracks first (unmodified
title/dur) and appends "— pendiente —" / "—" rows up to the target
count, each flagged `isPlaceholder` for the `.is-tbd` styling. This is
purely a display-time computation in the liner screen's own markup —
verified `cassettes[i].a`/`.b` (the arrays Screen 2's real tracklist and
playback read from) are never touched by it.

## Contrast fix (not explicitly requested, but silently corrected)

The old hero used a hardcoded `--brown-tape` text color regardless of
its own `--box-color` background. `DISK_COLORS` includes `--brown-tape`
itself as one of its 5 cycling colors — for whichever cassette lands on
that color (currently "bosque", index 3), that combination would render
the hero's own title text invisible against its own matching-brown
background. Since I was replacing this element entirely, I used the
carousel spine's already-established, color-safe pattern instead
(light `--vintage-sky` text on a darkened `color-mix()` of the box
color) rather than carrying that latent bug into the new bar. Flagging
this plainly since it wasn't asked for, though it was essentially free
given the rebuild.

## Verification

**No `c.loc` anywhere**: scanned the built static HTML
(`dist/lado-a-lado-b/index.html`) for the literal string `"las cintas
· locación"` (the exact pattern named in the mission) — zero matches.
Also confirmed the old `.cst-hero-loc`/`.cst-hero-label`/`.cst-hero`
classes no longer appear in the output at all (`grep -c` → 0), and the
new `.cst-hero-bar`/`#cstScreenLiner` markup is present (4 and 5
occurrences respectively, as expected for 4 cassettes).

**Hero bar** (CDP, production build): measured `.cst-hero-bar`'s
`getBoundingClientRect().height` at 64px (4rem) — clearly shorter than
the demo reference's own 4.6rem (73.6px) bar, and its `textContent`
confirmed to exclude "locación" for every cassette.

**Navigation, all 4 cassettes**: for each cassette index (0-3),
clicked its real spine on the carousel (hit-tested via
`elementFromPoint` first, per Mission 20's own documented
rotated-element pitfall — all 4 real spines sit on carousel face 0
simultaneously, not one per face, which an earlier draft of this
verification got wrong before I caught it), then clicked its hero bar,
and confirmed `#cstScreenLiner.classList.contains('is-active')` became
`true` with the correct panel (`document.querySelector('.cst-liner-panel:not([hidden])').dataset.cassetteIndex`)
visible in every case.

**Back button**: confirmed clicking `#cstBackToCassetteFromLiner`
results in `cstScreenCassette` active, `cstScreenShelf` and
`cstScreenLiner` both inactive — returns to Screen 2, not the shelf, as
required.

**Credits content, "cerro" (index 0)**: liner panel content, read
directly from the DOM —
- músicos: `["Esteban", "Gerson"]` ✓
- músicos adicionales: single placeholder line, exact text
  `"— espacio reservado, pendiente de confirmar —"` ✓
- personal técnico: both real lines, verbatim ✓
- lado a: `Tarde En La Mañana` (5:06), `El Ayer` (3:06), then 3
  placeholder rows (5 total) ✓
- lado b: `Niebla` (4:05), then 3 placeholder rows (4 total) ✓

**Credits content, "playa" (index 1)**: liner panel content —
- músicos: `["Esteban", "Gerson"]` ✓
- músicos adicionales: 4 real named entries with roles (Sonia Bruno —
  cello; Manuel Mora — batería y teclados; Iván Salazar — guitarras
  (pista(s) por confirmar); Kumary Sawyers — voz (pista(s) por
  confirmar)), followed by exactly 2 placeholder lines for the still-
  missing musicians ✓
- personal técnico: the one real combined-credit line, verbatim ✓
- lado a: 5 rows total, lado b: 4 rows total ✓

**Credits content, "ciudad"/"bosque" (indices 2-3)**: all three
sections (músicos, músicos adicionales, personal técnico) render the
single placeholder line `"créditos — pendientes"` with the
`cst-credit-tbd` (italic/dimmed) class; tracklists still padded to 5+4
✓ for both.

**Screenshots** (320/375/768px, Screen 2 + liner screen): confirmed
visually — the hero bar reads as a slim strip at every width (title
ellipsis-truncates gracefully at 320/375px when combined with the hint
text, which is normal responsive behavior for a title+hint row, not a
bug — the full title is always visible once the liner screen opens),
and the liner screen shows the full-width diagonal placeholder art,
title, and credits sections in the correct order at every width.

## Regression check — Screens 1 (carousel) and 3 (player)

Re-ran the established regression scripts unmodified against the
current build:
- Real audio/video playback and sync (drift well under 100ms), media
  session metadata, play/pause/next/prev, real synced lyrics,
  placeholder-track inert behavior, video click-shield (click +
  touch-tap unaffected), scrubber tap/drag-seek, REC badge and
  fullscreen hit-tests — all identical to the previously-shipped
  baseline, zero console errors.
- Fullscreen re-verified in isolation at 1440×900 and 390×844
  (`fsId: "cstApp"`, video wrapper/host sizes matching exactly).
- Carousel: turn-right button and a real drag sequence both produce
  genuinely different `getComputedStyle(#cstCarousel).transform`
  (`matrix3d`) values (real 3D rotation, not a no-op); click-to-open a
  real cassette (hit-tested via `elementFromPoint`) correctly opens
  `#cstScreenCassette`.

**One thing this mission's own change required fixing to avoid a
regression**: Screen 2's topbar eyebrow (`#cstCassetteEyebrow`) used to
read its text from the old hero's own `h2`
(`activePanel.querySelector("h2")`). Since that `h2` no longer exists
after removing the old hero, this would have silently gone blank. Fixed
by reading from the new bar's `.cst-hero-bar-title` span instead —
verified via the regression script's own `cassetteTitle` field
(`"grabaciones — Audiocassettes"`), which still populates correctly.

## Build

`npm run build` completes cleanly — 6 pages built, no new errors or
warnings, checked after every substantive edit and as a final check
before committing.

## Self-review

`git diff` reviewed before writing this file across both changed
files:
- `src/pages/lado-a-lado-b.astro`: only adds `credits` (and comments)
  to `cerro`/`playa`, and a comment noting the deliberate absence on
  `ciudad`/`bosque`. No existing field on any cassette/track changed.
- `src/components/CassettePlayer.astro`: new `CreditPerson`/`Credits`
  interfaces and `credits?` field, the `linerTracksFor` helper, the new
  hero bar markup/CSS replacing the old hero (with the old hero's CSS,
  including the now-fully-unused `.cst-reels`/`.cst-reel` base classes,
  removed), the new liner-notes screen markup/CSS, and JS wiring
  (`screenLiner` added to `showScreen`, `openLiner`/`linerPanels`, the
  hero-bar click listener, the new back button, and the eyebrow-title
  selector fix described above). Screens 1 and 3's own markup/CSS/JS
  are untouched — confirmed by the diff showing no changes outside the
  additions described here.
