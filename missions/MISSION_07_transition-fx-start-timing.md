# Mission 07 — Rewind Transition: Start the Streak/OSD at Click, Not at Settle

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Mission 06 shipped the approved 900ms rewind streak + VHS "▶ PLAY" OSD, but because the streak/OSD are triggered off `ViewTransition.finished` (the only way Mission 06 found to render a real DOM element at all, since a live element is invisible for the whole duration of an active View Transition), they now visibly start noticeably after the click — independently measured at roughly 1.1–1.4s post-click, instead of overlapping the page motion from frame one like the approved reference demo. Esteban has confirmed every other part of Mission 06 is exactly right (900ms root motion, streak look, OSD look, the ~1.8s total OSD lifespan/hold) — the **only** thing to fix is moving the streak/OSD's visible start back to the moment of click, with everything else unchanged.

**Hard constraint — read this before touching anything:** Esteban does not want any change to the current pacing, whatsoever — not the 900ms root motion, not the streak's look/duration, not the OSD's fade-in/hold/fade-out timing or its ~1.8s total lifespan, not how long the page takes to feel interactive again. The *only* acceptable change in this entire mission is moving the moment the streak/OSD visibly start from "after `.finished` resolves" to "at click." If the only way to achieve that turns out to alter pacing in any perceptible way — even a side effect like the page feeling unresponsive/non-interactive for longer than today — **do not ship it.** In that case, leave Mission 06's current `.finished`-triggered behavior in place exactly as-is, and report back in the response file that a pacing-neutral fix wasn't achievable, with a clear explanation of what was tried and why it wasn't clean enough. Shipping *something* is not the goal here — shipping the click-time start *only if* it comes with zero pacing change is.

## Root cause (for context, not to be re-litigated — see missions/MISSION_06_response.md for the original investigation)

A plain DOM element outside the View Transition system is invisible while a transition is active because the browser renders the `::view-transition-*` pseudo-element tree in its own top layer, above the live document, for the transition's full duration — it's not a z-index problem, no amount of `z-index`/`position: fixed` fixes it. The fix is to make the streak and OSD **participate** in the View Transition itself (their own `transition:name`, same mechanism already used to exempt the header), which puts them inside that top layer as their own `::view-transition-old/new(...)` pseudo-elements — those start animating at the same instant as the root, at click time, with no dependency on `.finished`.

## Scope (in)

1. **Give the streak and OSD their own named transition groups** (e.g. `transition:name="tfx-streak"` / `transition:name="tfx-osd"` on their container elements in `TransitionOverlay.astro`, following the same pattern as `Header.astro`'s `transition:name="site-header"`). This requires the elements to have stable identity across the outgoing and incoming page — they already render unconditionally in `BaseLayout.astro`, so this should be straightforward, but confirm it in practice.

2. **Re-implement the streak/OSD entrance and timing as `::view-transition-old/new(tfx-streak)` and `::view-transition-old/new(tfx-osd)` keyframe animations**, driven by the View Transition itself rather than a `.finished`-triggered class toggle, so they start at the same moment as the root's `rewind-exit`/`rewind-enter` animations (click time), with **no change to any of the currently-approved timing values**: streak still 900ms, OSD's fade-in/hold/fade-out still totals ~1800ms, same easing curves, same visual look (unchanged from Mission 06 — do not restyle).

3. **Handle the fact that the OSD's own lifespan (~1800ms) outlasts the root's 900ms motion — without changing pacing.** This is the one real engineering judgment call in this mission, and the hard constraint above governs it directly:
   - (a) First choice: keep the OSD's own transition-group animation running the full ~1800ms even though the root's finishes at 900ms (View Transitions allow per-group animation durations to differ, and the overall transition doesn't tear down until every group's animation completes) — **but only if this produces zero perceptible change** to how quickly the page feels interactive/scrollable again. Test this explicitly, don't assume it's fine.
   - (b) If (a) does perceptibly delay interactivity, a hybrid is acceptable *only if it is completely seamless*: the OSD/streak's entrance (the part overlapping the root's 900ms motion) runs as a genuine transition-group animation starting at click, then hands off to Mission 06's existing plain-DOM class-toggle approach for the remaining hold/fade-out once `.finished` resolves at 900ms — with no visible seam (flicker, jump, restart) and no change to the total look/timing.
   - (c) If neither (a) nor (b) can be made pacing-neutral and seamless, do not ship a compromised version. Leave Mission 06's current behavior in place and explain why in the response file, per the hard constraint above.

4. **Keep every other Mission 06 behavior exactly as-is:** no OSD/streak on hard/first load, no duplication on rapid re-navigation, full suppression under `prefers-reduced-motion: reduce`, header stability, and the Mission 04 interop checks (nav-toggle, reel/video-modal after soft navigation).

## Scope (out — do not touch in this mission)

- No changes to the root rewind motion's timing, easing, or look (untouched since Mission 06 — Esteban confirmed this part is correct as shipped).
- No changes to the streak's or OSD's visual design (colors, shapes, text, drop-shadow) — only *when* they start.
- No changes to `Header.astro`, `Reel.astro`, nav, accent colors, hero copy, or the reel interaction.
- No new npm dependencies.
- No sound.

## Acceptance Criteria

- [ ] The streak and OSD visibly begin animating essentially at click time (within roughly one frame, not after a ~900ms-1s delay) — verified with frame-by-frame Playwright inspection timed from the click event, not eyeballed.
- [ ] The streak's total visible duration/look and the OSD's total lifespan (~1800ms), hold, fade-in/fade-out timing, and appearance are unchanged from Mission 06 — confirmed by comparing against `reference/rewind-transition-reference.html` and against Mission 06's shipped behavior, not just the CSS source.
- [ ] No visible seam, flicker, or restart at the point where the entrance and hold/fade-out phases connect (if a hybrid approach per option (b) above is used).
- [ ] The root's 900ms rewind motion and its easing are byte-for-byte unchanged.
- [ ] No OSD/streak on hard/first page load; no duplication on rapid re-navigation.
- [ ] Under `prefers-reduced-motion: reduce`, navigation is still an instant cut with no streak and no OSD.
- [ ] The header still doesn't move, blur, or show the OSD during transitions.
- [ ] Mission 04 interop checks still pass: nav-toggle doesn't trigger a transition; the las cintas `Reel`/video modal still work after a soft navigation.
- [ ] No new `package.json`/`package-lock.json` entries.
- [ ] Renders cleanly with no jank at 320px, 375px, and 768px (Rule 9).
- [ ] `npm run build` succeeds with zero errors.
- [ ] No perceptible change to pacing anywhere: not the 900ms root motion, not the streak/OSD's look or timing, not how long the page takes to feel interactive again. If any option considered would have changed pacing, it was not shipped, and the response file explains why.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_06_rewind-transition-final.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Not applicable to this mission's changes.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed against the reference and against Mission 06's shipped behavior, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None — Esteban has confirmed everything except the streak/OSD's start timing is correct as shipped in Mission 06, and has explicitly said he does not want the current pacing touched at all. If none of options (a)/(b)/(c) from Scope item 3 can hit "click-time start, zero pacing change," don't ask mid-mission — fall back to (c) (leave Mission 06's behavior as-is) and explain the tradeoff space in the response file so Esteban can decide from there.
