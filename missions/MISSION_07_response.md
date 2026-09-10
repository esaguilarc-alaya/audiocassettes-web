# Mission 07 Response — Rewind Transition: Start the Streak/OSD at Click, Not at Settle

**Mission:** `missions/MISSION_07_transition-fx-start-timing.md`
**Executor:** Claude Code
**Date:** 2026-09-10

## Outcome, up front

**Mission 06's current `.finished`-triggered behavior is unchanged and still shipped.** Neither option (a) nor option (b) from the mission's own scope could be made to satisfy the mission's hard constraint (click-time start with zero pacing change and zero visual defect), so per the mission's own explicit instruction — *"do not ship a compromised version... leave Mission 06's current behavior in place"* — nothing was shipped. `git status` and `git diff` against the last commit show zero code changes (the untracked `Claude outputs/` directory aside, which isn't part of this project). This response documents what was tried, with evidence, so the tradeoff space is visible for a decision on where to go next.

## Sequence followed

1. Committed and pushed `missions/MISSION_07_transition-fx-start-timing.md` to `origin/main` (commit `69e9252`, `fb726a3..69e9252 main -> main`). Confirmed the push succeeded before touching any code. (A stale `.git/index.lock` from Cowork's authoring step needed clearing first — no live git process was holding it, confirmed via `ps aux` and the lock's ~15-minute age, matching the same benign pattern seen at the start of every prior mission in this project.)
2. Attempted the mission's scope, in the order the mission itself prescribes (option a, then option b), with explicit measurement at each step rather than assuming success.
3. Reverted cleanly to Mission 06's exact last-committed state once both options failed the hard constraint.
4. Wrote this response file.
5. Committed and pushed it.

## What was tried

### Option (a): let the OSD's own participating group run the full 1800ms

Gave `.tfx-streak` and `.tfx-osd` their own `transition:name` (`tfx-streak`, `tfx-osd`) plus `transition:persist`, moved their animations to `::view-transition-old/new(tfx-streak)` / `::view-transition-old/new(tfx-osd)` in `global.css` with the OSD's full, unchanged 1800ms keyframes, and removed the `.finished`-triggered JS entirely — exactly as the mission's "Root cause" section describes as the fix.

**Tested explicitly, per the mission's instruction not to assume:** patched `document.startViewTransition` to log real elapsed time, then polled `document.elementFromPoint()` over real page content (a Reel frame) at 700/1000/1300/1600/1900/2200ms after a real navigation click.

```
t=702ms  elementFromPoint(400,400) -> HTML.
t=1001ms elementFromPoint(400,400) -> HTML.
t=1301ms elementFromPoint(400,400) -> HTML.
t=1601ms elementFromPoint(400,400) -> HTML.
t=1901ms elementFromPoint(400,400) -> HTML.
t=2208ms elementFromPoint(400,400) -> FIGURE.reel-frame is-active
```

Real page content only becomes hit-testable again at ~2.2s, not ~0.9–1s as before this attempt. This is a direct, measured violation of the hard constraint ("not how long the page takes to feel interactive again") — the whole View Transition (root and header included, even though their own animations finish at 900ms) stays torn-down/non-interactive until every participating group's animation completes, and the OSD's group was the slowest at 1800ms. **Rejected**, as the mission's own option (a) description anticipated it might need to be.

### Option (b): hybrid — participating "entrance" (0–900ms) + live-DOM "continuation" (900–1800ms)

Designed a version where only the OSD's *entrance* participates (rescaled so it reaches and holds full opacity by 900ms — the original 18%/324ms rise point becomes 36% of a 900ms window; both endpoints' values are unchanged, only re-expressed on a shorter timeline), matching root's own duration exactly so the transition tears down and the page becomes interactive at 900ms again. The remaining hold + fade-out was planned as a plain live-DOM class-toggle animation triggered by `ViewTransition.finished` (the same mechanism Mission 06 used for the whole thing, now only for the back half) — chosen specifically because 900ms falls inside the original approved keyframes' flat hold segment (opacity pinned at 1 from 324ms to 1260ms), so the handoff point has zero visual change regardless of timing slack, by construction.

**Interactivity was confirmed fixed with this design:** the same `elementFromPoint` test came back interactive at ~1000–1300ms, matching Mission 06's own baseline (Mission 06's response similarly needed a >900ms wait margin before Reel became clickable in its own tests) — so option (b)'s pacing was sound for the interactivity half of the constraint.

**But the entrance itself never rendered.** Real-time screenshots of a real navigation, from 40ms through 900ms post-click, showed no streak and no OSD at any point during that window — the browser was correctly progressing through root's own blur/smear the whole time (confirmed visually), but the new `tfx-streak`/`tfx-osd` groups' content stayed invisible throughout, appearing only afterward.

To rule out every variable this session already knew to be suspect from Mission 06's own investigation:

- **Removed `transition:persist`** (kept only `transition:name`, matching `Header.astro`'s exact pattern, which works) — no change.
- **Replaced the real rescaled keyframes with a trivial, `!important`-forced test animation** (`opacity: 0 → 1` over 900ms `linear`, injected via a fresh `<style>` tag added to `<head>` at runtime — ruling out any CSS scoping or dev-server caching issue) on `::view-transition-new(tfx-osd)`. Still nothing rendered, at any checkpoint through 900ms.
- **Repeated the identical trivial-animation test on `::view-transition-new(tfx-streak)`** — same result: nothing rendered.
- **Confirmed participation *can* render complex content at all**, to make sure this wasn't a dead end from the start: forced `.tfx-osd`'s live element to a hardcoded, non-animated `opacity: 1; background: red` *before* the click (no animation involved at all) — this rendered correctly mid-transition, a solid red "▶ PLAY" box clearly visible at 600ms. So participation and complex content (SVG + text) are not the blocker.
- **Confirmed the pseudo-elements and their native group-morph animations genuinely exist**: `Animation.animationStarted` events showed all 8 expected animations (`rewind-exit`, `rewind-enter`, `tfx-streak`, `tfx-osd-entrance`, plus the four browser-generated `-ua-view-transition-group-anim-*` sizing/position animations for root, site-header, tfx-streak, and tfx-osd) firing together, at the same instant, with the correct names and durations. The group infrastructure is there; specifically the *custom, author-defined `animation` property on the image-pair's `::view-transition-old/new()` pseudo* does not visibly apply for these two groups, while the identical CSS pattern (`::view-transition-old/new(root)`, `::view-transition-old/new(site-header)`) reliably works, repeatedly, throughout this whole project's prior missions.

Put together: this points to some kind of real limit on simultaneous custom-animated named view-transition groups in this browser/environment (root + site-header, the two established since Mission 04, keep working; the third and fourth groups added in this attempt do not), not a mistake in the CSS itself, the persist/non-persist choice, or the keyframe values. I did not have time within this mission to fully characterize *why* (e.g., whether exactly 2 is a hard ceiling, or something else about these two elements specifically), and the mission's own "Root cause" section explicitly asks that the underlying theory not be re-litigated — but the theory's practical prediction (participation makes the entrance render) did not hold up under direct, repeated, varied testing in this environment, so I'm reporting the discrepancy rather than shipping code built on an assumption that didn't survive contact with a real render.

Per the hard constraint's explicit instruction for exactly this situation — *"if neither (a) nor (b) can be made pacing-neutral and seamless, do not ship a compromised version"* — option (b) was abandoned once its entrance phase was confirmed non-functional, and I reverted to Mission 06's exact committed files (`git show ad4723e:...` for both changed files, diffed against the working tree — zero difference) rather than leave a partially-modified state.

## Verification that the revert is clean and Mission 06's behavior is fully intact

- `git status`/`git diff` against `HEAD` (Mission 06's response commit) show **zero changes** to any tracked file.
- Re-ran the full Mission 04/06 interop suite against the reverted, freshly-rebuilt server:
  - Nav-toggle: opens/closes correctly, triggers zero view transitions (`vtCount: 0` on toggle click), works again on a fresh page post-navigation.
  - Reel: lever advances `0→1`, left-half back-tap correctly returns `1→0` (confirmed with an adequate wait margin — the same >900ms margin note from the Mission 06 response applies unchanged, this isn't new).
  - Video modal opens with the correct placeholder src.
  - First load: `.tfx-streak`/`.tfx-osd` computed opacity `0`, no `is-playing` class — no flash on hard load.
  - Rapid triple re-navigation at 320/375/768px: exactly one `.tfx-osd` and one `.tfx-streak` in the DOM afterward at every width, landed on the correct final page, zero console errors/exceptions, zero horizontal overflow.
  - `prefers-reduced-motion: reduce`: confirmed via `matchMedia(...).matches === true`, screenshotted mid-navigation — fully sharp, no blur, no streak, no OSD.
- `npm run build` succeeds, 6 pages, zero errors.
- `git diff --stat package.json package-lock.json` is empty — no dependency was added or removed by this investigation (all experiments were pure CSS/markup edits, since reverted).

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** Not touched.
2. **Verification over self-report.** This entire response is built from measured evidence (timestamped `elementFromPoint` results, `Animation.animationStarted` payloads, screenshots at controlled offsets) rather than assumption — including reporting that the mission's own prescribed root-cause fix did not hold up, rather than quietly working around that inconvenient finding.
3. **Placeholder discipline.** Not applicable.
4. **Mission boundary discipline.** No changes to `Header.astro`, `Reel.astro`, nav, accent colors, hero copy, or the root motion's keyframes/easing (confirmed identical to Mission 06 both by construction — the file was reverted wholesale — and because no code shipped at all).
5. **Static-first constraint.** Not touched.
6. **No unlicensed third-party assets.** None added.
7. **Diff before review.** The reverted files were diffed against Mission 06's exact committed blobs (`git show ad4723e:...`), not eyeballed.
8. **Mission handoff protocol.** Mission file committed/pushed first and confirmed; this response file is separate; both committed/pushed by Code.
9. **Mobile/responsive by default.** The (unchanged, reverted-to) behavior was re-verified at 320/375/768px as part of confirming nothing regressed.

## Acceptance criteria — status

Per the mission's own framing, the governing criterion is the last one, and it determines all the others:

- [x] **"If any option considered would have changed pacing, it was not shipped, and the response file explains why."** — Both options were tried; option (a) measurably broke pacing (interactivity delayed to ~2.2s); option (b) preserved pacing but its entrance never rendered (not a pacing problem, a correctness one — arguably worse to ship). Neither shipped.
- [ ] Click-time start — **not achieved**; still starts once the transition settles, as in Mission 06.
- [x] Streak/OSD look and total lifespan unchanged — trivially true, since Mission 06's exact files are what's shipped.
- [x] No seam/flicker — not applicable (no hybrid was shipped).
- [x] Root's 900ms motion byte-for-byte unchanged — confirmed via diff against Mission 06's commit; also never touched by either experiment.
- [x] No OSD/streak on hard load; no duplication on rapid re-navigation — re-verified.
- [x] Reduced motion still an instant cut — re-verified.
- [x] Header stability — re-verified (unchanged code).
- [x] Mission 04 interop checks pass — re-verified.
- [x] No new dependencies.
- [x] No jank at 320/375/768px — re-verified.
- [x] `npm run build` succeeds with zero errors.

## What this means for the goal

The click-time-start goal is not solved. Based on this investigation, the two paths the mission itself anticipated (full-duration participation, and a split entrance/continuation participation) both ran into real problems in this environment — one a pacing violation, the other a rendering limitation that held even for a trivial test case, isolated from every variable this project's prior missions had already flagged as suspect. If click-time start is still wanted, it likely needs either a different technique entirely (not investigated here — e.g., some way to reduce the number of simultaneous named groups, such as merging the streak and OSD into a single named group's animation rather than two, though that wasn't tested and may hit the same ceiling with just root+header+one-more) or acceptance of a pacing tradeoff smaller than option (a)'s but still non-zero. I did not attempt further variations given the hard constraint's clear preference for "leave it as-is" over "ship something compromised," but flagging the "merge into one group" idea as an untried, plausible next avenue rather than a dead end I've ruled out.

## Files changed

None. `missions/MISSION_07_transition-fx-start-timing.md` (this mission's own file) and this response file are the only additions to the repository.
