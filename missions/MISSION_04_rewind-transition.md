# Mission 04 — Page-Transition Rewind Effect

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Build the page-transition rewind/scrub effect — the second of the two approved "heavy theming" signature moments from the original project brief (the first, the las cintas `Reel`, shipped in Mission 02–03). This is a single, restrained, brand-appropriate transition that plays between full in-site page navigations, evoking a tape rewinding/scrubbing rather than a generic fade or slide.

## Context

- Astro 7 (already the project's only dependency — see `package.json`) ships native View Transitions support via `<ClientRouter />` (`astro:transitions`). This requires **no new npm dependency** — it's part of the `astro` package already installed.
- This is the project's second and last approved "heavy" theming moment per the brief's theming-discipline section. Explicitly rejected elsewhere in that same section: sound effects, custom cursors, repeated loading-screen gimmick copy. Those rejections apply here too — this transition must stay a quick, single visual beat, not a gimmick that repeats/lingers.
- Per Rule 9 (mobile/responsive by default, added in Mission 03), this effect must work cleanly on both desktop and the mobile nav shipped in Mission 03 — it must not interfere with the nav toggle's open/close behavior, which is a same-page DOM change, not a page navigation.

## Scope (in)

1. **Enable View Transitions.** Add `<ClientRouter />` (from `astro:transitions`) to `BaseLayout.astro` so every full in-site navigation (clicking a nav link, a hero CTA, any internal `<a>`) becomes a view transition instead of a hard page reload.

2. **Design and implement the rewind/scrub effect**, replacing the default cross-fade with a custom transition:
   - A quick (roughly 300–450ms total), single-beat motion suggesting a tape rewinding or scrubbing — for example: the outgoing page exits with a fast horizontal blur/streak and a brief desaturation flicker; the incoming page enters with a quick scanline-flicker overlay that fades out immediately. (This is Cowork's suggested direction, not a locked spec — see the judgment-call note below.)
   - Implement via `::view-transition-old(root)` / `::view-transition-new(root)` CSS (and named `transition:name` groups where useful, e.g. keeping the header/nav visually stable across the transition rather than blurring it too), using only existing tokens (colors, spacing) — no new colors or assets.
   - The effect plays **once per navigation**, does not loop, and does not repeat mid-page. It is not a loading-screen — the destination page's real content is what appears at the end of the transition, not a placeholder screen.
   - No sound. No custom cursor. No new npm dependency.

3. **Respect `prefers-reduced-motion`.** Users with that preference get an instant cut between pages (or, at most, a simple opacity cross-fade) — never the rewind motion.

4. **Interop with existing interactions.**
   - The Mission 03 mobile nav toggle (open/close) must be unaffected — it's a same-page state change, not a navigation, and must not trigger or interact with the transition.
   - The las cintas `Reel` component and its video modal must be unaffected in their own interaction model; only the transition *between* pages (e.g. navigating away from `las-cintas` to another page) is in scope.

## Scope (out — do not build these in this mission)

- No lyrics-follow-along feature.
- No sintoniza fan-engagement poll/backend — still out of scope, still needs its own scoping conversation.
- No changes to nav labels, per-page accent colors, brand tokens, hero copy, or the `Reel` component's interaction or visual design (the View-Master-authenticity visual polish noted separately is explicitly not part of this mission).
- No transition effects on same-page interactions (nav toggle open/close, video modal open/close, reel frame advance) — this mission is about full page-to-page navigation only.
- No new npm dependencies.

## Acceptance Criteria

- [ ] Navigating between any two of the six pages via a real link click triggers the rewind/scrub transition — verified by actually clicking through and observing/recording it, not by reading the CSS.
- [ ] The transition completes within roughly 300–450ms and does not loop or linger.
- [ ] With `prefers-reduced-motion: reduce` simulated, navigation uses an instant cut or simple cross-fade instead of the rewind motion — verified with that media feature actually set in the test environment, not assumed from the CSS.
- [ ] The Mission 03 mobile nav toggle still opens/closes correctly post-navigation, and its open/close action itself does not trigger the page transition.
- [ ] The las cintas `Reel` lever, back-tap, dots, and video modal all still work identically to Mission 02/03 after this mission — confirmed by re-running the same interaction checks used in the Mission 03 response, not assumed unchanged.
- [ ] No new entries in `package.json`/`package-lock.json` — confirmed via diff.
- [ ] The effect renders correctly and performs smoothly (no visible jank) at 320px, 375px, and 768px widths, per Rule 9.
- [ ] `npm run build` succeeds with zero errors.
- [ ] Spot-check: no sound was added, no custom cursor was added, no loading-screen placeholder appears mid-transition, no scope-out items were touched.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_03_mobile-responsive.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Anything standing in for real content (photos, video, audio, lyrics, location names, song titles) must be obviously a placeholder — never plausible enough to ship as real by accident.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. Any server/API/database dependency must be explicitly scoped in the mission that introduces it — never an incidental side effect.
6. **No unlicensed third-party assets.** Confirm license terms before pulling in stock photos, icon packs, fonts, or snippets.
7. **Diff before review.** Reference artifacts (mockups, prior accepted files) get diffed against new work, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file itself first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md` (never appended to the mission file), and commits and pushes that response file too. The reviewer then pulls the latest repo state and checks the response against acceptance criteria — the response file is a pointer to what to verify, never proof on its own.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px as part of that mission's own acceptance criteria.

## Judgment call flagged for Esteban

The brief approves "a page-transition rewind/scrub effect" as a concept but doesn't specify its exact visual mechanics. This mission's Scope (in) #2 above gives Code a concrete, brand-consistent direction (fast blur/streak + brief scanline flicker) to implement rather than leaving it fully open — but the exact look is Cowork's interpretation, not a locked brand-manual requirement. If what ships doesn't feel right once you see it, that's a quick follow-up tweak, not a re-scope of this mission.

## Open Questions for Esteban (Code should ask, not assume)

None — if Code finds a genuine ambiguity beyond what's specified above, it should make a reasonable, brand-consistent, restrained choice and flag it clearly in the response file.
