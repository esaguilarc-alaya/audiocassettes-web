# Mission 03 — Mobile & Responsive Retrofit

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Make everything built in Missions 01 and 02 genuinely usable on a phone — not just "doesn't visually break," but a header/nav that works as a real mobile nav, and a las cintas `Reel` interaction that's comfortable on touch. From this mission forward, mobile/responsive behavior is a standing requirement for every mission (see new Rule 9 below), not a separate cleanup pass — this mission exists specifically to retrofit the two missions that shipped before that rule existed.

## Context (what's already there)

- `<meta name="viewport" content="width=device-width, initial-scale=1">` is already present in `BaseLayout.astro`.
- `Hero.astro`, `Footer.astro`, `LasCintasPreview.astro`, and `NotasDeCintaTeaser.astro` already have a `@media (max-width: 50rem)` breakpoint that stacks their grid layouts to one column. These are a reasonable starting pattern — reuse the same breakpoint value for consistency unless a component's content genuinely needs a different one.
- `Header.astro` has **no media query at all**. Its nav (`wordmark` + 5 nav links) relies on flex-wrap alone, which is not a real mobile nav pattern at narrow widths.
- `Reel.astro` has **no media query**. The reel body is a flex row of a 4:3 viewer plus a fixed 2.5rem-wide lever button; this mission needs to confirm (and fix if not true) that this holds up and stays comfortable to tap down to a 320–375px-wide screen, not just that it doesn't overflow.
- The four stub pages (`lado-a-lado-b`, `notas-de-cinta`, `el-estuche`, `sintoniza`) are simple single-column sections with no responsive treatment yet — likely low-risk, but confirm rather than assume.

## Scope (in)

1. **Header/nav mobile treatment.**
   - Below the existing `50rem` breakpoint, the header must not just wrap awkwardly. Implement a real small-screen nav pattern — a hamburger/disclosure toggle revealing the nav links, or another approach of your choice — as long as: all five nav links remain reachable and readable, the wordmark/isologo stays visible and tappable back to home, and no text or tap target is visually cramped or overlapping at 320px width.
   - Whatever toggle control is added must be keyboard-operable and have an accessible label (e.g. `aria-expanded`, `aria-label="abrir menú"` / "cerrar menú").
   - No new colors, fonts, or icon packs — build any icon (e.g. a hamburger glyph) as inline SVG or text glyph using existing tokens, per Rule 1 and Rule 6.

2. **Las cintas `Reel` touch pass.**
   - Confirm real touch behavior, not just click-event equivalence: the lever must be comfortably tappable (minimum ~44×44px touch target — it already is close to this via `.reel-lever` at 2.5rem×6.5rem, confirm it holds at narrow widths), and the left-half-of-viewer back-tap must remain easy to hit accurately on a touchscreen (not a tiny sliver).
   - Add a `@media (max-width: 50rem)` (or narrower, if you find layout needs it) pass to `Reel.astro` if the current fixed-width layout doesn't hold up cleanly on a real small-screen render — verify this with an actual narrow-viewport render, not by reading the CSS and assuming.
   - The video modal (`.reel-modal`) already uses `90vw` sizing — confirm it renders correctly (no overflow, close button reachable and tappable) at a 375px viewport width.
   - Do not change the interaction model itself (lever = forward, left-half = back, dots = position-only) — this mission is a layout/touch-target pass, not a redesign of Mission 02's interaction.

3. **Full-site narrow-viewport audit.**
   - Render every page (`index`, `lado-a-lado-b`, `las-cintas`, `notas-de-cinta`, `el-estuche`, `sintoniza`) at 320px, 375px, and 768px widths and fix any real problem found: horizontal scroll/overflow, overlapping text, a tap target smaller than ~44px, or text too small to read comfortably. Do not make changes to pages/components with no real problem just to "add more responsiveness" — this is a fix pass, not a rebuild.

## Scope (out — do not build these in this mission)

- No new pages, no new sections, no new copy — this mission touches layout/CSS/interaction-affordance only.
- No changes to the per-page accent color mapping, brand tokens, or nav link labels/order from Mission 02.
- No page-transition rewind/scrub effect, no lyrics-follow-along feature, no sintoniza backend/poll — still not started, still out of scope here.
- No new frameworks or CSS libraries (no Tailwind, no a hamburger-menu npm package) — build the mobile nav toggle with plain HTML/CSS/vanilla JS, consistent with how `Reel.astro`'s interaction was built in Mission 02.

## Acceptance Criteria

- [ ] At 320px viewport width, the header shows a working nav toggle; all five nav links are reachable through it; the wordmark/isologo remains visible and links home; nothing overlaps or is visually cut off.
- [ ] The nav toggle has `aria-expanded` reflecting open/closed state and an accessible label; it is operable via keyboard (Enter/Space), not click-only.
- [ ] At 375px viewport width, both `las-cintas` `Reel` instances: lever is tappable and comfortably sized, left-half-of-viewer back-tap is easy to trigger accurately, frame content and captions are legible without overflow.
- [ ] The video modal renders without horizontal overflow at 375px width, and its close button is reachable and tappable.
- [ ] All six pages render with zero horizontal scroll/overflow at 320px, 375px, and 768px widths — verified by rendering each width, not by inspecting CSS alone.
- [ ] No interaction model changes from Mission 02 (lever/left-half/dots semantics unchanged) — diffed against `Reel.astro`'s Mission 02 state to confirm only layout/touch-target code changed, not the logic.
- [ ] `npm run build` succeeds with zero errors.
- [ ] Spot-check confirms no scope-out items were built: no new pages, no accent-mapping changes, no rewind/lyrics/sintoniza-backend work, no new npm dependencies added to `package.json`.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–8 are carried over unchanged from `missions/MISSION_02_hero-accent-reel.md`; Rule 9 is new as of this mission and applies to every mission from here forward, not just this one.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Anything standing in for real content (photos, video, audio, lyrics, location names, song titles) must be obviously a placeholder — never plausible enough to ship as real by accident.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. Any server/API/database dependency must be explicitly scoped in the mission that introduces it — never an incidental side effect.
6. **No unlicensed third-party assets.** Confirm license terms before pulling in stock photos, icon packs, fonts, or snippets.
7. **Diff before review.** Reference artifacts (mockups, prior accepted files) get diffed against new work, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file itself first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md` (never appended to the mission file), and commits and pushes that response file too. The reviewer then pulls the latest repo state and checks the response against acceptance criteria — the response file is a pointer to what to verify, never proof on its own.
9. **Mobile/responsive by default (new).** Every mission from this one forward must ship mobile-friendly and responsive by default — this is not an opt-in or a separate follow-up mission. Any new page, component, or interaction must be checked at 320px, 375px, and 768px viewport widths as part of that mission's own acceptance criteria, not deferred to a later cleanup pass. If a mission's scope makes a genuine mobile treatment ambiguous (e.g. a touch-specific interaction with no obvious phone equivalent), it should ask rather than assume, per Rule 1/Rule 3's spirit — but the default assumption is that mobile support is in scope unless a mission explicitly says otherwise.

## Open Questions for Esteban (Code should ask, not assume)

None — this mission's scope is fully specified above. If Code finds a genuine ambiguity in how the mobile nav toggle should look/behave beyond the functional requirements stated, it should make a reasonable, brand-consistent choice and flag it clearly in the response file rather than blocking on it.
