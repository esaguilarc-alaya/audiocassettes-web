# Mission 06 — Rewind Transition: Final Approved Design

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Replace Mission 05's blur-slide transition with the design Esteban actually approved after reviewing live demos: a 900ms fast-rewind streak (directional stretch/blur, matching "option C" from the demo round) plus a white VHS-style "▶ PLAY" on-screen-display icon that fades in at the top-right corner as the new page settles. Mission 04's `<ClientRouter />` wiring and the header exemption stay as-is — this mission only replaces the motion/visual design and adds the OSD element.

## Reference artifact (source of truth — port this, don't reinterpret)

`reference/rewind-transition-reference.html` (added by this mission, alongside this file) is the exact approved demo Esteban reviewed and confirmed. It is a standalone mockup (not part of the real site's routing) demonstrating:

- The "fast-rewind streak" motion: the outgoing page stretches/smears to one side (`scaleX(1.4)`, `translateX(-60%)`, `blur(16px)`, fading out) over ~900ms with `cubic-bezier(.55,0,1,.45)`; the incoming page mirrors it entering from the opposite side (`translateX(60%)` → `0`, `scaleX(1.4)` → `1`, `blur(16px)` → `0`) with `cubic-bezier(0,.55,.45,1)`, starting after a short delay (~12% of the duration) so the two overlap slightly.
- A diagonal "light streak" overlay (`repeating-linear-gradient(100deg, ...)`) that sweeps across during the transition, peaking in opacity around 35% through and fading out by 100%.
- The white "▶ PLAY" OSD: a triangle + "PLAY" text (monospace, letter-spaced, white with a dark drop-shadow/outline for legibility against any background — see the reference's `.play-osd` styling), positioned top-right, that fades in starting around 18% of its own (slightly longer) lifespan, holds, then fades out. In the reference this OSD lifespan is `var(--dur) + 900ms` (i.e. ~1800ms total) so it lingers a bit after the page/streak motion itself has settled — this is intentional, not a bug: the streak is quick (900ms) but the "PLAY" confirmation should feel like it holds a beat longer, the way a real camcorder OSD does.

Open `reference/rewind-transition-reference.html` directly in a browser and click "play transition" to see the exact approved behavior before implementing.

## Scope (in)

1. **Replace the transition motion.** In `src/styles/global.css`, replace the `rewind-exit`/`rewind-enter` `@keyframes` and their `::view-transition-old(root)`/`::view-transition-new(root)` rules (from Mission 05) with the streak motion described above, timed at 900ms (not Mission 05's 160–220ms). Keep using only existing brand tokens — the streak overlay's white highlight color and the OSD's white/dark-shadow styling are neutral (not brand-token colors) by design, matching the reference exactly.

2. **Add the light-streak overlay.** Since `::view-transition-*` pseudo-elements can't host child elements, implement the sweeping streak as appropriate for how View Transitions actually work in this codebase — e.g. a real overlay element shown for the transition's duration, coordinated via Astro's transition lifecycle events (`astro:before-preparation` / `astro:after-swap`) or another mechanism of your choice, as long as the observable result matches the reference: a diagonal light-streak sweep synced with the page motion, not lingering after the motion settles.

3. **Add the "▶ PLAY" OSD.** A white play-triangle + "PLAY" text, top-right corner, fading in as the new page settles and fading out roughly ~1.8s after the transition starts (per the reference's timing) — implemented as a real DOM overlay (not a `::view-transition` pseudo, since it needs to hold longer than the page motion itself and isn't part of the page's captured snapshot). It must not be part of the page's persisted layout — it should not appear on a hard/first page load, only after an in-site navigation, and must not stack up or duplicate if the user navigates again before a previous OSD has finished fading.

4. **Keep everything else from Mission 04 intact:** `<ClientRouter />` wiring, the header's `transition:name="site-header"` exemption (header still doesn't move/blur/get an OSD), and reduced-motion handling (still an instant cut — the OSD and streak must also not appear under `prefers-reduced-motion: reduce`, since they're motion/attention effects, not just the page-root blur).

## Scope (out — do not touch in this mission)

- No changes to `BaseLayout.astro`, `Header.astro`, or `Reel.astro` beyond what's strictly needed to add/coordinate the OSD overlay (e.g. a small script addition) — no unrelated edits.
- No changes to nav, accent colors, hero copy, or the reel interaction.
- No new npm dependencies — the play-triangle icon should be inline SVG or a CSS shape, matching how other icons in this project are built (see `Isologo.astro`, the Mission 03 hamburger).
- No sound.

## Acceptance Criteria

- [ ] Side-by-side comparison against `reference/rewind-transition-reference.html`: the real site's transition matches its motion, timing (900ms streak), and streak-overlay behavior — confirmed by rendering both and comparing frame-by-frame (same method used to diagnose Mission 04/05), not by reading the CSS.
- [ ] The white "▶ PLAY" OSD appears top-right after a real in-site navigation, matches the reference's look (triangle + "PLAY", white with dark outline/shadow for legibility), and fades in/holds/fades out on a timeline matching the reference (~1.8s total lifespan).
- [ ] The OSD does not appear on a hard/first page load (only on in-site navigations), and does not duplicate/stack if the user navigates again mid-fade — verified by rapid repeated navigation.
- [ ] Under `prefers-reduced-motion: reduce`, navigation is an instant cut with **no** streak and **no** OSD — verified with the media feature actually set, not assumed.
- [ ] The header still doesn't move, blur, or show the OSD during transitions.
- [ ] Re-run the Mission 04 interop checks: nav-toggle click doesn't trigger a transition; the las cintas `Reel`/video modal still work identically after a soft navigation.
- [ ] No new `package.json`/`package-lock.json` entries.
- [ ] Renders cleanly with no jank at 320px, 375px, and 768px (Rule 9) — including the OSD staying fully on-screen and legible at 320px width.
- [ ] `npm run build` succeeds with zero errors.
- [ ] Spot-check: no sound added; no scope-out items touched.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_05_rewind-transition-refinement.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception. The OSD's white/monospace/drop-shadow styling is an explicit decision recorded in this mission (matching the approved reference), not a brand-token color.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Not applicable to this mission's changes.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets — the play triangle is a simple shape (SVG polygon or CSS), not a pulled-in icon asset.
7. **Diff before review.** The real implementation is diffed against `reference/rewind-transition-reference.html`'s behavior, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the reference file) in the project's `missions/`/`reference/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and reference file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None — the reference file is the settled spec. If the exact DOM/CSS mechanism for the streak overlay or OSD needs a judgment call not covered above, make the choice that most faithfully reproduces the reference's observable behavior and flag it in the response file.
