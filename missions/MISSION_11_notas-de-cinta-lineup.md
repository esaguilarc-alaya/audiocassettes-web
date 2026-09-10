# Mission 11 — Notas de Cinta: Current Lineup + Amigos de Audiocassettes

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Add two new content sections to `notas-de-cinta.astro`, below the existing bio copy shipped in Mission 09: a "quiénes somos hoy" (current lineup) section and an "amigos de audiocassettes" (past bandmates) section. Both are new **page content**, not a replacement of the site-wide "sintoniza" CTA block at the bottom of the page — that block stays identical/static exactly as it is on every other page.

Both pieces of copy below are final and confirmed by Esteban — port them verbatim, no paraphrasing, no reordering the list beyond what's specified.

## Scope (in)

1. **New section: "quiénes somos hoy"** (heading text — casing/treatment your judgment, consistent with the page's existing heading style). A list of the current lineup, in **exactly this order** (Esteban was explicit this must not read as hierarchical / must not lead with him as singer):

   - Gerson — bajo y teclados
   - Cali — batería
   - Esteban — voz y guitarra
   - Iván — guitarra

   Render as a simple list (your judgment on exact markup — a plain `<ul>`, or a small grid of name/role pairs consistent with the site's existing component patterns). No photos, no additional embellishment beyond name + role.

2. **New section: "amigos de audiocassettes"** (heading text — same treatment as above). Body copy, verbatim:

   > A través de los años compartimos banda con: Roberto, Alonso, Vladimir, Víctor, Erick, Chris, Álvaro. Y Rolando nuestro baterista en distintos periodos y nuestro amigo siempre.

   Render as a short paragraph (or paragraph + list of names — your judgment), but the text itself must not change, and the Rolando sentence must stay exactly as given above (do not merge it into a listing of names — it reads as its own sentence, as shown).

3. **Placement.** Both new sections go after the existing bio content (including its "— Esteban" signature) and before the page's shared "sintoniza" closing CTA block. Order between the two new sections is your judgment (recommend "quiénes somos hoy" then "amigos de audiocassettes", but not a hard requirement).

4. **Styling.** Reuse existing typographic/spacing patterns already established on this page (the `.bio` paragraph treatment, section spacing) — do not invent new type scales or colors. Section headings should read as clearly distinct from body paragraphs (e.g. similar treatment to how the page's `<h1>` or any existing sub-heading pattern elsewhere in the site works), using only established brand tokens (Rule 1).

## Scope (out — do not touch in this mission)

- No changes to the site-wide "sintoniza" CTA block on any page — it stays static and identical everywhere.
- No changes to the existing bio paragraphs or signature shipped in Mission 09.
- No changes to `notas-de-cinta.astro`'s `<h1>` or `PageAccentBar`.
- No changes to `NotasDeCintaTeaser.astro` (home page) or any other component/page.
- No changes to the View-Master shelf/viewer (Missions 08/10) or the page-transition effect (Missions 04–07).
- No new npm dependencies.

## Acceptance Criteria

- [ ] "quiénes somos hoy" section lists exactly Gerson (bajo y teclados), Cali (batería), Esteban (voz y guitarra), Iván (guitarra) in that exact order — diffed against this document, not eyeballed.
- [ ] "amigos de audiocassettes" section's body text matches the approved copy above verbatim, including the Rolando sentence exactly as written.
- [ ] Both new sections appear after the existing bio/signature and before the shared sintoniza CTA block.
- [ ] The sintoniza CTA block itself is byte-for-byte unchanged (diff confirms zero changes to that shared component/partial).
- [ ] The existing bio paragraphs, signature, `<h1>`, and accent bar are unchanged.
- [ ] No other page or component was touched.
- [ ] Renders cleanly with no jank or overflow at 320px, 375px, and 768px (Rule 9).
- [ ] `npm run build` succeeds with zero errors; no new dependencies.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_10_viewmaster-visual-refinement.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Not applicable — this mission adds real, confirmed content.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed word-for-word against this mission's copy, and the sintoniza CTA block is diffed to confirm zero changes — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None — both pieces of copy and the lineup order are final and confirmed. If exact heading markup or list styling needs a judgment call, use your own design judgment consistent with the rest of the site and record the choice in the response file.
