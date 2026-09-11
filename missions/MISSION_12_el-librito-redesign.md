# Mission 12 — El Librito: Rename + Paper/Accordion-Booklet Visual Redesign

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Two changes to the page currently called "notas de cinta":

1. **Rename** it to "el librito" everywhere it's user-facing (nav label, `<h1>`, page `<title>`/meta if set, any on-page copy that names the section). The route/URL, component file names, and internal code identifiers may stay as-is unless renaming them is trivial and low-risk — don't go on a refactor beyond what's needed for the user-facing name to read "el librito".
2. **Visual redesign** of the page's content area (the bio + the Mission 11 "quiénes somos hoy" / "amigos de audiocassettes" sections) so it reads as a physical cassette J-card insert/booklet: aged paper texture, a repeating accordion-fold pattern running down the whole page at a consistent visual rhythm, ending in a folded "spine" label (matching the little printed tab that shows the artist name through a closed cassette case) with two internal registers — the band name on top, the isologo below — styled with visible fold relief (drop shadows, crease shadows), not flat color blocks.

## Reference artifact (visuals — settled this time)

`reference/el-librito-look-reference.html` (added by this mission) is a mockup Esteban reviewed through many rounds — including comparing against a real cassette J-card photo — and approved. Port its visuals faithfully:

- **Paper texture:** aged-paper background (fine grain, a few soft "foxing" spots, a subtle edge vignette) at the intermediate intensity shown in the reference — not the very subtle level, not the heavily aged level from earlier rounds, the settled middle ground baked into this reference file. A folded top-right corner on the page.
- **Accordion fold rhythm:** a repeating horizontal crease line (with shadow on both sides, like a worn fold) running down the entire content area at a consistent spacing — this is a decorative background effect, independent of where paragraphs happen to break.
- **Dynamic panel spacing (important — read carefully):** the reference computes the fold spacing at runtime with a small client-side script rather than a hardcoded pixel value. This matters because a hardcoded spacing tuned for one viewport width lands in the wrong place at another width and can cut the last paragraph awkwardly, stranding its last line alone past the final fold. The reference's script measures the real rendered height of the content and the position of the last paragraph, then picks a fold spacing such that the fold immediately before the final panel always lands *before* that last paragraph begins — so the whole final paragraph (last line included) always stays inside one panel, at any viewport width. Port this logic (or equivalent logic that achieves the same guarantee), re-run on resize, not just a copied magic-number value.
- **Spine/label:** a single final panel, shortened to roughly two-thirds the height of a regular accordion panel, styled as one applied sticker/label (its own drop shadow, a hard crease shadow where it folds away from the panel above, inset shading suggesting it's physically folded — not flat fill color). Internally divided into two registers: a thinner top register with the band name ("Audiocassettes", in the same script/cursive treatment already used for section headings and the bio signature) and a larger bottom register with the isologo. The isologo has no existing asset — reuse the reference's placeholder concentric-circle treatment (Rule 1 precedent: an abstract shape built from brand tokens, not an invented illustration) unless Esteban supplies a real logo asset before this mission ships, in which case ask before substituting. This spine panel must be the same width as the rest of the booklet (no side margins narrowing it).
- **No front-cover photo.** This was explored and explicitly dropped — do not add a photo placeholder.
- **Content unchanged:** the actual bio paragraphs (Mission 09), the "quiénes somos hoy" lineup in its exact confirmed order, and the "amigos de audiocassettes" paragraph (Mission 11) are copied through verbatim into this new visual shell — no wording changes.

Open `reference/el-librito-look-reference.html` directly in a browser (and resize it, or check it at 320/375/768/1280px) to see the approved look and confirm the dynamic fold behavior before implementing.

## Scope (in)

1. Rename "notas de cinta" → "el librito" in nav and the page's `<h1>` (and page title/meta if the project sets one per-page).
2. Restyle the page's content area per the reference: aged paper background, repeating accordion fold (dynamically spaced per the reference's approach), and the two-register spine/label panel at the end, sized ~2/3 of a regular panel and full page width.
3. Carry over Mission 09's bio copy and Mission 11's "quiénes somos hoy" / "amigos de audiocassettes" content unchanged into the new shell.
4. Use only established brand tokens for all new colors (Brown Tape, Vintage Sky, View Master, and the neutral/black precedent already used elsewhere) — no invented colors (Rule 1).
5. This is a client-side-only visual/script addition (measuring rendered layout and setting a CSS custom property) — it introduces no server, API, or build-time dependency, and does not affect the static-first constraint (Rule 5).

## Scope (out — do not touch in this mission)

- No changes to the route/URL structure unless trivially required for the rename (confirm with Esteban first if it would require a redirect or break an existing link).
- No changes to the View-Master shelf/viewer (`las-cintas`, Missions 08/10), the page-transition effect (Missions 04–07), or the shared sintoniza footer/CTA (must remain byte-for-byte identical — diff it to confirm).
- No changes to other pages, nav structure beyond the label text, or any other component.
- No new npm dependencies. No real photo/logo assets unless Esteban explicitly supplies one before this ships (ask, don't assume).

## Acceptance Criteria

- [ ] The page reads "el librito" in nav and `<h1>` (and title/meta if applicable) instead of "notas de cinta".
- [ ] Aged paper texture (grain, foxing spots, edge vignette, folded corner) matches the reference's intermediate intensity — diffed against the reference's CSS values, not eyeballed.
- [ ] A repeating fold-crease pattern runs the length of the content area, and its spacing is computed at runtime (verify by reading the shipped script, not just visually) rather than a single hardcoded value copied from one viewport's measurement.
- [ ] At 320px, 375px, 768px, and at least one desktop width (e.g. 1280px), the last real paragraph ("amigos de audiocassettes") renders fully inside a single panel — confirm programmatically (e.g. via a quick Playwright check reading the fold position vs. the paragraph's bounding box) at each of those widths, not just visually at one size.
- [ ] The final spine/label panel is the same width as the rest of the booklet (no side insets), is visibly shorter (~2/3) than a regular accordion panel, and shows two internal registers (name on top, isologo below) with visible fold/crease relief — not flat, undifferentiated color blocks.
- [ ] No front-cover photo placeholder was added.
- [ ] The bio paragraphs, "quiénes somos hoy" lineup (exact order), and "amigos de audiocassettes" paragraph all match their previously-shipped text verbatim — diffed word-for-word, not just re-typed from memory.
- [ ] The shared sintoniza footer component is byte-for-byte unchanged (diffed).
- [ ] `las-cintas`, the transition effect, other pages, and nav structure (beyond the one label) are untouched.
- [ ] Renders cleanly with no jank or horizontal overflow at 320px, 375px, and 768px (Rule 9).
- [ ] `npm run build` succeeds with zero errors; no new dependencies.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_11_notas-de-cinta-lineup.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception. The isologo placeholder follows the same "abstract shape from brand tokens, not an invented illustration" precedent already used elsewhere.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted. This mission specifically requires a runtime/programmatic check of the dynamic fold-spacing behavior at multiple widths, not a visual-only check.
3. **Placeholder discipline.** The isologo is an intentional abstract placeholder per Rule 1's established precedent, not an unfinished stand-in — flag it as such in the response file if Esteban may want to swap in a real logo asset later.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. The dynamic fold-spacing script is a small client-side layout calculation, not a server/API/database dependency — this does not violate the constraint.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against the reference and against the previously-shipped bio/lineup/amigos copy and the sintoniza footer — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the reference file) in the project's `missions/`/`reference/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and reference file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

- If renaming the page also means changing its route/URL (e.g. `/notas-de-cinta` → `/el-librito`), ask Esteban to confirm before doing so — this could break any existing external links (social bio links, etc.) and may need a redirect. If the route can stay `/notas-de-cinta` while everything user-facing reads "el librito", prefer that and note the choice in the response file.
- If a real band isologo asset exists or is provided before this ships, ask whether to use it instead of the abstract placeholder — otherwise proceed with the placeholder and flag it as swappable later.
