# Mission 24 — "liner notes" screen: narrow cassette bar + cover art + credits + tracklist

## Context

Esteban reviewed two rounds of an interactive demo
(`reference/liner-notes-demo-reference.html`, attached to this mission —
the second, approved iteration) and gave final approval with two small
adjustments folded into this mission's spec below:

- Screen 2's current big hero (the large colored box with the two
  spinning-reel icons) is replaced by a **narrow, full-width bar**
  spanning the same width as the tracklist below it — even narrower
  (shorter height) than the demo's own bar. Tapping it opens a new
  **liner notes** screen.
- The liner notes screen shows the cover art as its own full-width
  showcase piece at the top (not a small square squeezed next to the
  credits), followed by the cassette title, then credits sections
  (músicos / músicos adicionales / personal técnico), then both sides'
  tracklists.
- **`c.loc` ("las cintas · locación X") must be removed everywhere it
  currently appears, and must NOT be added to the new hero bar or the
  liner notes screen.** Today it only appears under the big hero's
  title (`.cst-hero-loc`, `{c.loc}` in `CassettePlayer.astro`) — that
  hero is being replaced entirely, so just don't carry `c.loc` forward
  into the new bar or the liner screen's heading. The `Cassette.loc`
  field itself can stay in the data model (other code may still
  reference it later), just stop rendering it.

Screen 3 (the player) is untouched — this mission only changes what's
inside Screen 2 (the cassette panel).

## Scope (in)

1. **Replace the big hero** (`.cst-hero` and its reels/label markup) in
   each cassette panel with a narrow, full-width, tappable bar — same
   horizontal padding as the tracklist below it (`0 1.3rem` on the
   panel), noticeably shorter than the demo reference's own bar (aim
   for something closer to a slim strip — use judgment, but the demo's
   height should read as a ceiling, not a target). Show the cassette
   title and a small "créditos y portada" hint/icon on it (see the
   reference for the general shape — title + hint, no location line).
   Keep the per-cassette color (`DISK_COLORS[i % DISK_COLORS.length]`)
   driving its background, same pattern as today's hero/carousel spines.
2. **New "liner notes" screen**, inserted into the existing 3-screen
   flow as its own `.cst-screen` (e.g. `#cstScreenLiner`), reached by
   tapping the new hero bar, with a back button returning to the
   cassette screen (Screen 2) — not the shelf. Structure, top to bottom:
   - Full-width cover-art showcase (square-ish, same brand-derived
     placeholder treatment as the reference — this is NOT real art yet,
     keep it clearly a placeholder per Rule 3, e.g. a subtle diagonal
     brand-derived pattern with a small "portada — pendiente" label;
     do NOT invent or source a real image).
   - Cassette title (no `c.loc` line beneath it).
   - **Músicos** section.
   - **Músicos adicionales** section.
   - **Personal técnico** section (produced/recorded/mixed/mastered by,
     matching a Wikipedia-style album-credits block).
   - **Lado A** tracklist, then **Lado B** tracklist (plain listing —
     not tappable/playable from here, this screen is informational; the
     existing tracklist on Screen 2 is still what opens the player).
3. **Real credits content** — only for the two cassettes Esteban gave
   data for. Use this verbatim (translating nothing, inventing nothing
   beyond what's explicitly marked as a placeholder below):

   **"cerro" cassette (displayed title "Audiocassettes")**
   - Músicos: Esteban, Gerson
   - Músicos adicionales: none provided — render this section with a
     single clearly-placeholder line (e.g. "— espacio reservado,
     pendiente de confirmar —"), matching the reference's own wording/
     treatment.
   - Personal técnico:
     - Producido, grabado y mezclado por Alberto Ortiz en El Closet
       Studio
     - Masterizado por Manuel Mora en Mandrágora Studio

   **"playa" cassette (displayed title "Reel Real")**
   - Músicos: Esteban, Gerson
   - Músicos adicionales:
     - Sonia Bruno — cello
     - Manuel Mora — batería y teclados
     - Iván Salazar — guitarras (pista(s) por confirmar)
     - Kumary Sawyers — voz (pista(s) por confirmar)
     - Two additional placeholder slots, each rendered like the
       "músicos adicionales" placeholder above (Esteban said two more
       musicians are still missing) — do not guess who they are.
   - Personal técnico:
     - Producido, grabado, mezclado y masterizado por Manuel Mora en
       Mandrágora Studio

   **"ciudad" and "bosque" cassettes** (no real credits given yet): the
   liner notes screen for these two should render with every section
   (músicos, músicos adicionales, personal técnico) shown as a single
   clear placeholder line each (e.g. "créditos — pendientes"), per Rule
   3 — do not invent names, roles, or studios for these.
4. **Tracklist padding to 8–10 songs per record.** Esteban wants room to
   see roughly 8–10 songs per cassette even though only 1–3 real tracks
   exist per side today. On the liner notes screen's Lado A/Lado B
   listings, pad each side out with placeholder rows (numbered,
   clearly marked pending — e.g. "— pendiente —" for title and
   duration) so the total across both sides lands roughly in the 8–10
   range per cassette. Use your judgment on the exact split between
   sides; this is a rough target, not an exact spec. This padding is
   for the liner notes screen's tracklist display ONLY — do NOT add
   placeholder tracks to the real `cassettes` data array/Screen 2's
   actual tracklist (which drives real playback) — those stay exactly
   as they are today.
5. **Data model**: add whatever optional field(s) you need to
   `Cassette`/the per-cassette data in `lado-a-lado-b.astro` to carry
   this credits content (e.g. an optional `credits` object with
   `musicians`, `additionalMusicians`, `technical` arrays) — only
   populate it for `cerro` and `playa` with the real content above;
   leave it absent/undefined for `ciudad` and `bosque`, and have the
   component render the placeholder-everything version whenever it's
   absent (this keeps the "no invented content" rule enforced by the
   data shape itself, not just by what you happen to type into the
   template).
6. Follow the reference's own visual approach (full-bleed art showcase,
   sections below with small uppercase headers in `--polaroid-sunset`,
   etc.) but adapt proportions/spacing as needed to fit this file's
   actual card dimensions — the reference is a proof of concept, not a
   pixel spec.

## Scope (out)

- No real cover art — placeholder only (Esteban confirmed this).
- No changes to Screen 3 (player) or to how the real tracklist/playback
  behaves.
- No changes to the carousel (Screen 1) from Missions 20–23.
- Don't add tap/play behavior to the liner notes screen's own
  tracklist — it's read-only reference info; playing a track still
  happens from Screen 2's existing tracklist.
- Don't invent credits for "ciudad"/"bosque" or fill in any of the
  explicitly-left-blank slots for "playa" (the two missing musicians,
  the track numbers for Iván Salazar/Kumary Sawyers).

## Acceptance criteria

- Screen 2's hero is now a narrow, full-width bar (visibly shorter than
  the demo reference's own bar), no `c.loc` text on it.
- Tapping the bar opens a new liner notes screen; the back button
  returns to Screen 2 (not the shelf).
- Liner notes screen shows, top to bottom: full-width placeholder cover
  art, cassette title (no location line), músicos, músicos adicionales,
  personal técnico, lado A tracklist, lado B tracklist.
- "cerro"/"Audiocassettes" and "playa"/"Reel Real" show the exact real
  credits content specified above, with only the explicitly-called-out
  gaps (músicos adicionales for cerro; the two missing musicians and
  the two unconfirmed track references for playa) shown as clear
  placeholders — not invented.
- "ciudad" and "bosque" show a fully placeholder liner notes screen.
- Each cassette's liner-notes tracklist is padded with placeholder rows
  to roughly 8–10 total songs; the real `cassettes` data/Screen 2's
  actual tracklist is unchanged.
- `c.loc` ("las cintas · locación X") does not render anywhere in the
  app anymore.
- Verified at 320px, 375px, and 768px (Rule 9).
- Full regression on Screen 1 (carousel) and Screen 3 (player,
  real audio/video sync, scrubber, click-shield, fullscreen) — untouched
  by this mission, confirm they still work.
- `npm run build` succeeds with no new errors/warnings.

## Standing Engineering Control Rules (apply to every mission)

1. **Brand fidelity** — only established tokens/derivatives; the
   placeholder cover art must not use any new hex value.
2. **Verification over self-report** — your response file's claims will
   be independently re-verified against the real repo/build/browser
   behavior.
3. **Placeholder discipline** — this mission is full of explicit
   placeholder content; every one of them must read clearly as
   "pending"/"placeholder" to a visitor, never presented as real.
4. **Mission boundary discipline** — Screen 2 additions only; don't
   touch Screens 1/3.
5. **Static-first constraint** — no new server/build dependency.
6. **No unlicensed third-party assets** — no real cover art, no new
   fonts/icons.
7. **Diff before review** — self-review your own `git diff` before
   writing the response file.
8. **Mission handoff protocol** — commit and push this mission file
   (and the reference file) FIRST, confirm the push succeeded, THEN
   execute the mission's scope, THEN write a separate
   `missions/MISSION_24_response.md` and commit+push that separately.
9. **Mobile/responsive by default** — verify at 320px, 375px, 768px.
