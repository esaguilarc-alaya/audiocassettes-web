# Mission 05 — Rewind Transition Refinement

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Fix the readability problem in Mission 04's page-transition rewind effect. Esteban's own feedback after seeing it live: it "looks like a shake effect" and it's unclear what's supposed to be happening. This mission replaces the transition's keyframes with a clearer, single-direction motion — it does not touch anything else Mission 04 built (the `<ClientRouter />` wiring, the header exemption, reduced-motion handling all stay as-is).

## Diagnosis (from Cowork's own frame-by-frame render of the current effect)

Rendering the current transition frame-by-frame (via CDP screenshots at fixed delays after a real nav click) showed:
- ~0–150ms: the outgoing page blurs, desaturates, **and stretches horizontally** (`scaleX(1.025)`) — a growth with no clear direction, which reads as an ambiguous wobble rather than a deliberate motion.
- ~150–300ms: the incoming page flickers through **four separate** blur/grayscale/brightness/contrast keyframe stops (`rewind-enter`'s 0%/18%/30%/50%/100%) in quick succession — a rapid pulsing of brightness/contrast with no accompanying positional motion, which reads as a flash/glitch/shake rather than a "tape scrubbing forward" motion.
- The whole effect has no consistent direction — nothing visibly moves *from* somewhere *to* somewhere, so there's nothing for the eye to interpret as "rewinding."

## Scope (in)

Replace the `rewind-exit` / `rewind-enter` `@keyframes` and the `::view-transition-old(root)` / `::view-transition-new(root)` rules in `src/styles/global.css` (added in Mission 04) with a **single-direction slide + blur** design:

- **Exit (old page):** translate horizontally in one consistent direction (e.g. `translateX(0)` → `translateX(-4%)`) while blurring in (`blur(0)` → `blur(8–10px)`) and fading out (`opacity 1` → `0`), over roughly 150–180ms, `ease-in`. One motion, one direction, no `scaleX` stretch.
- **Enter (new page):** translate in from the **same direction's opposite edge** (e.g. `translateX(4%)` → `translateX(0)`) while blurring out (`blur(8–10px)` → `blur(0)`) and fading in (`opacity 0` → `1`), over roughly 180–220ms, `ease-out`, starting shortly after the exit begins (a small overlap is fine, same as Mission 04's approach).
- At most **one** brightness/contrast adjustment on the enter (a single subtle dip-then-settle, e.g. `brightness(0.9)` → `brightness(1)`), not the four-stop pulse Mission 04 shipped. If it doesn't clearly read as "settling," it's fine to drop brightness/contrast from the enter animation entirely and rely on blur + translate + opacity alone.
- Keep total duration in the same 300–450ms band as Mission 04.
- The direction (left/right) is Code's call — pick whichever reads more like "advancing/rewinding through a tape" and stays consistent for every navigation (not different directions for different links).
- Everything else from Mission 04 is unchanged: `<ClientRouter />` wiring, the header's `transition:name="site-header"` exemption (still no motion on the header), and the reduced-motion behavior (still comes for free from Astro's own `viewtransitions.css`).

## Scope (out — do not touch in this mission)

- No changes to `BaseLayout.astro`, `Header.astro`, or `Reel.astro` — Mission 04's `astro:page-load` double-init fix and the `transition:name` wiring stay exactly as shipped.
- No changes to reduced-motion handling — it already works correctly per Mission 04's verification and doesn't need touching.
- No new npm dependencies.
- No changes to any other mission's scope (nav, accent colors, hero copy, reel interaction, mobile layout).

## Acceptance Criteria

- [ ] Frame-by-frame rendering of a real navigation (screenshots at fixed delays, same method as this mission's own diagnosis) shows a single, consistent horizontal direction of motion on both the outgoing and incoming page — not a stretch/grow, not a static blur-in-place.
- [ ] The incoming page's filter changes happen in at most one smooth step (or none, if brightness/contrast is dropped), not the previous four-stop pulse — verified by reading the new keyframes and confirming against the rendered frames.
- [ ] Total transition duration is still in the 300–450ms band.
- [ ] `prefers-reduced-motion: reduce` still gives an instant cut — re-verify this wasn't broken by the CSS change (same method as Mission 04's response: set the media feature, screenshot mid-navigation, confirm no blur/motion).
- [ ] The header still doesn't move or blur during the transition (still exempted via `transition:name="site-header"`).
- [ ] Re-run Mission 04's nav-toggle interop check (toggle click doesn't start a transition) and the reel/modal-after-soft-navigation checks — confirm nothing regressed.
- [ ] No new `package.json`/`package-lock.json` entries.
- [ ] Effect renders cleanly with no jank at 320px, 375px, and 768px (Rule 9).
- [ ] `npm run build` succeeds with zero errors.
- [ ] `git diff` for this mission touches only `src/styles/global.css` (the keyframes/rules named above) — nothing else.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_04_rewind-transition.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Anything standing in for real content must be obviously a placeholder — never plausible enough to ship as real by accident.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. Any server/API/database dependency must be explicitly scoped in the mission that introduces it.
6. **No unlicensed third-party assets.** Confirm license terms before pulling in stock photos, icon packs, fonts, or snippets.
7. **Diff before review.** Reference artifacts get diffed against new work, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file itself first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too. The reviewer then pulls the latest repo state and checks the response against acceptance criteria.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None. The exact direction (left vs. right) and exact easing values are Code's judgment call within the constraints above — flag the choice made in the response file rather than blocking on it.
