# Mission 02 — Home Hero Copy, Per-Page Accent, and the Las Cintas Reel

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Port the approved home hero copy verbatim, add the per-page accent color pattern to the four themed section pages, and replace the `las-cintas` stub with the real View-Master-style reel interaction, including the video modal. This is the first mission to add real interaction and real (if placeholder-backed) content on top of the Mission 01 scaffold.

## Scope (in)

1. **Home hero copy.**
   - Heading stays exactly as-is: `tocamos donde otros no tocarían.`
   - Replace the hero paragraph with this approved final copy, ported verbatim — no paraphrasing, no trimming:
     > "Esta es nuestra gira: sin escenarios fijos, sin fechas oficiales. Grabamos donde el país nos lleva — ríos, montañas, calles. Y la entrada es gratis si andás cerca."

2. **Per-page accent token.**
   - Introduce a `--page-accent` CSS custom property, scoped per page (e.g. set on the page's root wrapper or passed as a layout prop — implementer's choice, but it must not leak to other pages).
   - It drives **exactly one** element: a short accent-rule bar directly under that page's `<h1>` title. Nothing else — not nav, not buttons, not cards — may change color based on this token.
   - Mapping (exact, no substitutions):
     - `lado-a-lado-b` → `--view-master`
     - `las-cintas` → `--retro-pop`
     - `notas-de-cinta` → `--polaroid-sunset`
     - `el-estuche` → `--brown-tape`
   - `index` (home) and `sintoniza` do **not** receive this token — no accent bar on either.

3. **Sintoniza consistency check.**
   - Confirm (and fix if it currently isn't) that the sintoniza block — wherever it appears, header or footer, on every page — renders `--stereo-blue` unconditionally, never derived from `--page-accent` or any other per-page state.

4. **Las cintas — real View-Master reel component.**
   - Build a `Reel` component representing one "cinta" (one location/session). A reel holds an ordered sequence of frames; frame types may mix video, photo, old band photo, and band-art within the *same* reel (not one type per reel).
   - Interaction:
     - A lever control that on click advances to the next frame with a slide-and-rotate transition, mimicking a real View-Master's mechanical click-forward feel.
     - Clicking the left half of the lens/viewer goes back one frame.
     - Frame-counter dots show current position within the reel.
   - Video frames show a play badge. Clicking one opens a branded modal — brown-tape frame border, view-master-red close button — embedding a YouTube iframe. Since no real videos exist yet, use obviously fake/placeholder video IDs (never a real, working video ID standing in as if it were band content).
   - Populate `las-cintas.astro` with at least 2 placeholder reels. Location names must be obvious placeholders (e.g. `[ubicación real aquí]`), not invented-but-plausible place names. Each reel needs at least one video frame and one photo frame, all placeholder assets per Standing Rule 3 below.
   - This applies to the `las-cintas` page itself. `LasCintasPreview.astro` (the home-page teaser grid) is unaffected by this mission and can keep linking through as-is.

## Scope (out — do not build these in this mission)

- No real photos, video, or audio content — placeholders only, and they must be unmistakably placeholders.
- No page-transition rewind/scrub effect between pages (separate future mission).
- No lyrics-follow-along feature.
- No sintoniza fan-engagement poll or any backend/serverless/database work — `sintoniza` stays a static stub (email signup UI, social links) exactly as scoped in Mission 01.
- No changes to the brand color tokens, fonts, or nav structure beyond what's specified above.
- No real merch copy on `el-estuche` or real bio copy on `notas-de-cinta` — those pages only gain their accent bar in this mission, nothing else.

## Acceptance Criteria

- [ ] Home hero heading is unchanged (`tocamos donde otros no tocarían.`) and the hero paragraph matches the approved copy above exactly, verbatim — diff it against this document, don't eyeball it.
- [ ] `--page-accent` renders as a short rule/bar under the `<h1>` on `lado-a-lado-b`, `las-cintas`, `notas-de-cinta`, and `el-estuche`, in the correct mapped color — verified by inspecting the actual rendered CSS/DOM, not by visual impression alone.
- [ ] Home and `sintoniza` have no accent bar.
- [ ] The sintoniza block is `--stereo-blue` everywhere it appears, on every page, no exceptions.
- [ ] `las-cintas.astro` contains at least 2 working `Reel` components: lever-click advances forward with the slide-and-rotate transition, left-half-of-lens click goes back one frame, frame-counter dots reflect position.
- [ ] At least one frame across the reels is a video frame that opens the branded modal (brown-tape frame, view-master-red close button) with an embedded YouTube iframe on click.
- [ ] All new placeholder content (location names, "video" content) is obviously placeholder — not plausible enough to be mistaken for real band material.
- [ ] `npm install && npm run dev` and `npm run build` succeed with zero console errors on a clean clone.
- [ ] Spot-check confirms no scope-out items were built early: no rewind transition, no lyrics panel, no sintoniza backend/poll UI.

## Standing Engineering Control Rules for audiocassettes-web

These apply to this mission and every mission on this project. Rules 1–7 are carried over unchanged from `missions/MISSION_01_scaffold.md`; Rule 8 is new as of this mission.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Anything standing in for real content (photos, video, audio, lyrics, location names, song titles) must be obviously a placeholder — never plausible enough to ship as real by accident.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. Any server/API/database dependency must be explicitly scoped in the mission that introduces it — never an incidental side effect.
6. **No unlicensed third-party assets.** Confirm license terms before pulling in stock photos, icon packs, fonts, or snippets.
7. **Diff before review.** Reference artifacts (mockups, prior accepted files) get diffed against new work, not just eyeballed for vibes.
8. **Mission handoff protocol (new).** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork then hands off to Code with a prompt naming the mission file/ID. Code is responsible for: committing and pushing the mission file itself, executing the mission's scope, writing a **separate** response file at `missions/MISSION_0X_response.md` (never appended to the mission file — mission and response stay two distinct files), and committing and pushing that response file too. Cowork/the reviewer then pulls the latest repo state to check the response against the acceptance criteria above — per Rule 2, the response file is a pointer to what to verify, never proof on its own that it was done correctly.

## Open Questions for Esteban (Code should ask, not assume)

None — this mission's scope is fully specified above. If Code hits an ambiguity not covered here, it should ask rather than guess, per Rule 1 and Rule 3.
