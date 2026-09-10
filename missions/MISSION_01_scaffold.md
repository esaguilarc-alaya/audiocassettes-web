# Mission 01 — Repo Scaffold & Design Foundation

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Create the audiocassettes-web repository from scratch and scaffold an Astro project with the band's design system in place, a base layout, and stub pages for every planned section. No real content, no backend logic — this mission is foundation only.

## Scope (in)

- Create a new GitHub repo named audiocassettes-web under the same account that hosts CIE. Initialize with a .gitignore appropriate for Astro/Node, a README.md describing the project in one paragraph, and MIT or "all rights reserved" license (ask Esteban which, don't assume).
- Commit this mission document itself into the repo at missions/MISSION_01_scaffold.md, as part of the initial commit. Future missions follow the same pattern (missions/MISSION_02_...md, etc.) so the repo carries its own build history alongside the code.
- Scaffold an Astro project (latest stable) with the following structure:
  ```text
  src/
    layouts/
      BaseLayout.astro
    components/
      Header.astro
      Footer.astro
      Stripe.astro
    pages/
      index.astro           (home — port the approved mockup)
      lado-a-lado-b.astro   (stub)
      las-cintas.astro      (stub)
      notas-de-cinta.astro  (stub)
      el-estuche.astro      (stub)
      sintoniza.astro       (stub — see note below)
    styles/
      tokens.css            (design tokens, see below)
  ```
- Design tokens (tokens.css or equivalent Astro-friendly token file) must define, as CSS custom properties, exactly these values — no invented or approximated colors:
  ```css
  --brown-tape: #59332C;
  --vintage-sky: #FFFDC7;
  --polaroid-sunset: #EF9A49;
  --view-master: #D54852;
  --retro-pop: #4DA495;
  --stereo-blue: #006080;
  ```
- Typography: Fredoka (weights 500/600/700) for display/headings, Montserrat (400/500/700) for body and UI text, loaded via Google Fonts. This is a placeholder for the brand's actual corporate font (Star Avenue) — see Engineering Control Rule 1 below.
- Header must include: the four-color stripe device, the "audiocassettes" wordmark (Fredoka), and nav links to all five sections using their approved names: lado a / lado b, las cintas, notas de cinta, el estuche, sintoniza. All headings and nav labels lowercase, per the brand manual's own convention.
- Port the approved home page mockup (attached separately as home-mockup-reference.html) into index.astro, broken into components where it makes sense (e.g. the "las cintas" preview grid, the music teaser block) rather than as one monolithic page file.
- sintoniza.astro is a stub page only — heading, one paragraph placeholder text noting "poll/voting feature — Mission TBD," and the email signup form UI (non-functional, no backend wiring yet).
- Local dev must run cleanly via npm install && npm run dev with zero console errors.

## Scope (out — do not build these in this mission)

- No real photos, videos, or song audio — use clearly labeled placeholder blocks (e.g. a bordered box with text like [ foto/video real aquí ]), never invented stock content standing in as if it were real band material.
- No View-Master reel interaction, no lyrics panel, no video modal — those are separate future missions and should not be started early.
- No backend, no database, no API routes — including for sintoniza. That page is a static stub only.
- No deployment/hosting setup (Vercel/Netlify/Pages) — hosting target isn't decided yet.

## Acceptance Criteria

- [ ] Repo exists on GitHub under the correct account, with an initial commit containing everything above.
- [ ] missions/MISSION_01_scaffold.md exists in the repo, matching this document.
- [ ] npm install && npm run dev succeeds with no errors on a clean clone.
- [ ] All six color tokens match the hex values above exactly (verified by opening tokens.css directly, not by eyeballing rendered color).
- [ ] Home page visually matches the approved mockup's structure and copy — hero, music teaser, las cintas preview grid, notas de cinta teaser, sintoniza footer.
- [ ] All five nav links route to real (even if stub) pages — no 404s.
- [ ] No placeholder content is written in a way that could be mistaken for real band content (real photos claimed, invented song titles presented as real, etc.).

## Standing Engineering Control Rules for audiocassettes-web

These apply to this mission and every mission after it on this project.

1. **Brand fidelity rule.** Every color, font, and section name used in code must trace directly to the brand manual or to a decision explicitly made in the planning conversation with Claude — never invented, approximated, or "close enough" substituted without flagging it. Star Avenue → Fredoka is the one approved exception, and it must be flagged in the README as a temporary substitution pending the real font file.
2. **Verification over self-report.** Claude (chat) verifies mission completion by reading the actual repo files and running the project locally where possible — never by trusting Code's own summary of what it did. If Code reports a mission complete, that report is a claim to be checked, not a fact to be recorded.
3. **Placeholder discipline.** Anything standing in for real content (photos, video, audio, lyrics, location names, song titles) must be visually and textually obvious as a placeholder. No placeholder should be plausible enough to accidentally ship as real.
4. **Mission boundary discipline.** Each mission has an explicit in-scope and out-of-scope list. Code should not "helpfully" get ahead and start next-mission work (e.g. wiring up the sintoniza backend, building the View-Master interaction) inside an earlier mission. Scope creep here makes review harder, not easier.
5. **Static-first constraint.** The site is static-rendered by default. Any addition of a server, API route, database, or third-party service dependency must be called out explicitly in that mission's scope — it should never appear as an incidental side effect of implementing something else.
6. **No unlicensed third-party assets.** No stock photography, icon packs, fonts, or code snippets get pulled in without confirming license terms permit the intended use (commercial band website). If unsure, stub it and flag it rather than guessing.
7. **Diff before review.** Before Claude reviews a mission's output, any previously approved reference artifact (mockup HTML, an earlier mission's accepted file) should be diffed against the new implementation to catch unintended drift, not just checked for surface-level resemblance.

## Open Question for Esteban (Code should ask, not assume)

License for the repo: MIT (open) or all-rights-reserved (private/closed)?

---

**Resolved during Mission 01 execution (2026-09-09):**

- License: **MIT** initially, changed to **all rights reserved** (Copyright © 2026 Audiocassettes) shortly after — see `LICENSE`.
- Repo created immediately (not staged locally-only first).
- `home-mockup-reference.html` and `Audiocassettes_Manual de Identidad.pdf` were not available at first execution, so `index.astro` was initially built from this document's section descriptions rather than ported. Both reference files were added shortly after (now committed at `reference/`) and the home page was rebuilt to actually port the mockup: exact copy, the reel-icon SVG, the las cintas cards with real place names (Río Celeste, Cerro Chirripó, Puerto Viejo) and the mockup's own `"nombre de canción"` placeholder convention, and the sintoniza footer structure. This acceptance criterion is now met against the real artifact, not a guess.
- Diffing against the manual and mockup also caught a wrong guess: the first draft of the four-color stripe device (`Stripe.astro`) had picked polaroid-sunset/view-master/retro-pop/stereo-blue. The manual (p.21) and the mockup's own CSS both specify **brown-tape → polaroid-sunset → view-master → retro-pop**. Corrected.
- The manual's isologo (concentric-circle cassette-reel icon replacing the "o" in the wordmark, manual p.7–9) was subsequently built as an inline SVG (`src/components/Isologo.astro`) and wired into the header, superseding the plain-text wordmark this mission originally shipped with. Ring colors/order (retro-pop, polaroid-sunset, view-master, brown-tape hub) confirmed against the manual's palette. Screenshotted and visually verified, not just built-and-assumed.
