# Mission 13 Response — El Estuche: Merch Catalog with Per-Item Order-by-Email

**Mission:** `missions/MISSION_13_el-estuche-merch.md`
**Executor:** Claude Code
**Date:** 2026-09-10

## Sequence followed

1. Committed and pushed `missions/MISSION_13_el-estuche-merch.md` and all 6 image assets under `src/assets/merch/` together in one commit to `origin/main` (commit `56b0a51`, `461cac0..56b0a51 main -> main`). Confirmed the push succeeded before touching any code.
2. Executed the mission scope.
3. Wrote this response file.
4. Committed and pushed the scope work separately (commit `bd3c401`, `56b0a51..bd3c401 main -> main`), and will commit/push this response file separately next.

## What was built

All changes are scoped entirely to `src/pages/el-estuche.astro` — no other file was touched.

### Image handling approach (per the mission's "your call, record why")
Used `astro:assets`' `<Image>` component with local imports from `src/assets/merch/`. Checked first whether this needed a new dependency: `sharp` (Astro's default image service) is already present in `package-lock.json` as part of Astro 7.3.2's own dependency tree — confirmed via `node -e "require.resolve('sharp')"` before writing any code — so no new package was added and no `astro.config.mjs` change was needed. `npm run build` shows Astro's "generating optimized images" step actually running and shrinking all 6 WebP files (e.g. the largest, the t-shirt photo, went from 44kB to 28kB), confirming the built-in optimization pipeline is genuinely active, not just present unused.

### Six product cards
Each card (`.merch-card`) shows: the product photo, the Spanish item name, and its own "ordenar" `mailto:` link. Names used exactly as the mission's own examples: "camiseta", "gorra", "pin", "taza — diseño clásico", "taza — diseño cerro", "cordón".

### Responsive image boxes
The six source photos have wildly different native aspect ratios (measured via `sips` before writing any CSS): the t-shirt crop is tall (658×820, ratio ~0.80), the lanyard is a wide banner (850×160, ratio ~5.3), the cap/mugs/pin are all roughly square-ish (250×240 to 471×383). A single cropped `object-fit: cover` box would have badly cropped the shirt and butchered the lanyard. Instead: read each of the six photos directly and confirmed by eye they all share the exact same `--vintage-sky` backdrop as the page itself (a deliberate choice in how the source images were prepared). So each `.merch-card__photo` is a fixed `4/3` aspect-ratio box with `background: var(--vintage-sky)` and `object-fit: contain` on the image inside — any letterboxed space around a non-4:3 photo reads as continuous page background rather than a visible seam, confirmed visually in the screenshots below.

### Order-by-email mechanism
A single `const ORDER_EMAIL = "pedidos@audiocassettes.com";` at the top of the file, with a comment flagging it as a placeholder Esteban will replace once a real inbox exists. A `mailtoFor(item)` helper builds each link from that one constant: `subject` is `Pedido: <item name>`, `body` is a short Spanish message naming the item and inviting the customer to write their preferred color into the reply (no color-picker UI, per the mission's explicit instruction). No prices appear anywhere on the page.

### Layout/chrome
A 3-column grid on desktop, collapsing to 2 columns at the site's existing `50rem` mobile breakpoint and 1 column at `26.25rem`, matching the pattern already established by `LasCintasPreview.astro`'s cinta grid. Card chrome (`border: 2px solid var(--brown-tape)`, `background: rgba(89, 51, 44, 0.06)` — literally `--brown-tape`'s own RGB at low alpha) and the "ordenar" button's visual treatment (`--brown-tape` fill, `--vintage-sky` text, lowercase, font-body bold) are both reused verbatim from `NotasDeCintaTeaser.astro`'s existing `.card`/`.btn-solid` patterns rather than inventing new chrome — no new colors were introduced anywhere.

### Intro copy
Added one short functional line above the grid ("Piezas de merch de audiocassettes, hechas a pedido. Escribinos por correo para elegir el color de la tuya.") describing the actual mechanism (made-to-order, color chosen via email) rather than inventing marketing copy — per the mission's own instruction to keep this short and functional and only ask if something more stylized was wanted.

## Verification (Rule 2/7 — measured, not eyeballed)

- **All 6 items, correct image/name, no price**: a CDP script navigated to the live dev server and read each `.merch-card`'s image `alt`, `src`, name text, and order-link `href` — all 6 items present with the exact Spanish names above. A regex check (`/\$|colones|crc|precio/i`) against the entire grid's text content returned no match at any of the 4 tested widths.
- **`mailto:` links — inspected the actual `href` values, not just presence of a button**: e.g. the "taza — diseño clásico" card's link is exactly `mailto:pedidos@audiocassettes.com?subject=Pedido%3A%20taza%20%E2%80%94%20dise%C3%B1o%20cl%C3%A1sico&body=Hola%2C%0A%0AQuiero%20pedir%3A%20taza%20%E2%80%94%20dise%C3%B1o%20cl%C3%A1sico.%0AMi%20color%20preferido%20es%3A%20%5Bescrib%C3%AD%20aqu%C3%AD%20tu%20color%20preferido%5D%0A%0A%C2%A1Gracias!` — decoding confirms the subject is `Pedido: taza — diseño clásico` and the body invites the customer to write their color preference. All 6 items' `href`s were read and confirmed this way, not just checked for existence.
- **Single placeholder-address constant**: `grep -n "pedidos@audiocassettes" src/pages/el-estuche.astro` returns exactly one line — the `const ORDER_EMAIL = ...` declaration itself. `grep -rn "pedidos@audiocassettes" src/` (codebase-wide) returns only that same one file/line — confirming the address isn't hardcoded anywhere else, even though it naturally appears 6 times in the *rendered* HTML output (once per generated `mailto:` link), which is expected and correct.
- **No form/backend/checkout introduced**: `grep -n "<form" src/pages/el-estuche.astro` returns nothing. A CDP check does find one `<form>` on the live page — traced via `grep -n "<form" src/components/*.astro` to `EmailSignupForm.astro`, the pre-existing sintoniza newsletter signup rendered globally by the shared `Footer.astro` on every page (not introduced by this mission). `git diff --stat package.json package-lock.json` is empty — no new dependency, confirming Astro's already-bundled `sharp` covered the image optimization without any addition.
- **Responsive, no overflow — re-verified, not assumed**: CDP-driven checks at 320px, 375px, 768px, and 1280px all show `document.documentElement.scrollWidth === window.innerWidth` (zero horizontal overflow) and zero console errors/exceptions. Screenshots at 320px (1-column stack), 768px (2-column grid), and ~1100px (3-column grid) all show the photo letterboxing blending seamlessly into the surrounding card background, all 6 items legible with no cropping/distortion of the lanyard or t-shirt.
- **Brand tokens only**: every new color in the page's `<style>` block is either an existing token (`--brown-tape`, `--vintage-sky`) used directly, or the same literal `rgba(89, 51, 44, 0.06)` already established in `NotasDeCintaTeaser.astro` (which is exactly `--brown-tape`'s own RGB at low alpha) — no new hex values.
- **Scope discipline**: `git status --short` shows exactly one modified file, `src/pages/el-estuche.astro`, plus the 6 new asset files and the mission file (already committed separately in the first commit). `Header.astro`, `Footer.astro`, `notas-de-cinta.astro`, `ViewMasterShelf.astro`, `las-cintas.astro`, `global.css`, and the transition system were not opened.
- **Build**: `npm run build` succeeds (clean rebuild after clearing `.astro`/`node_modules/.vite`/`dist`), 6 pages, zero errors, with the image-optimization step visibly running and shrinking all 6 files.

## Standing Engineering Control Rules — how each was respected

1. **Brand fidelity.** All chrome colors reuse existing tokens/precedent (see above); the 6 product photos carry their own brand-manual colors, nothing new was invented for them.
2. **Verification over self-report.** Every `mailto:` `href` was read and decoded, not just clicked-and-assumed; the single-constant claim was grepped codebase-wide, not just visually scanned in one file.
3. **Placeholder discipline.** `ORDER_EMAIL` is flagged in a code comment as an intentional placeholder Esteban will replace once a real inbox exists — not a silently-shipped fake value.
4. **Mission boundary discipline.** No cart, no payment, no color-picker UI, no prices — confirmed absent by direct inspection, not just "not added on purpose."
5. **Static-first constraint.** `mailto:` links are pure client-side anchor hrefs; no server/API/database dependency added.
6. **No unlicensed third-party assets.** Only the 6 supplied brand-manual product photos were used; no new fonts or icon packs.
7. **Diff before review.** Checked `git status`/`git diff --stat` against `package.json`/`package-lock.json` and against the rest of the codebase for the email constant, not eyeballed for vibes.
8. **Mission handoff protocol.** Mission file and image assets committed/pushed together first and confirmed (`56b0a51`); scope work committed/pushed separately (`bd3c401`); this response file is separate, committed/pushed last.
9. **Mobile/responsive by default.** Verified at 320/375/768px plus 1280px desktop, zero overflow, zero console errors at every width.

## Acceptance criteria — status

- [x] All 6 items render with correct image, Spanish name, no price shown anywhere — confirmed via CDP read + regex scan.
- [x] Each item has a working `mailto:` link with the item's name in the subject and a color-preference invitation in the body — confirmed by decoding actual `href` values for all 6 items.
- [x] The placeholder order email address appears in exactly one place in the source (a single constant) — confirmed via codebase-wide grep.
- [x] No form, backend call, or third-party checkout script introduced — confirmed via `grep` (zero `<form>` in this file) and an empty `package.json`/`package-lock.json` diff.
- [x] Images responsive, no horizontal overflow at 320/375/768px — re-verified via CDP measurement, not assumed.
- [x] All new colors/chrome trace to established brand tokens — confirmed (existing tokens + the site's own pre-established `rgba(89,51,44,...)` precedent).
- [x] No other page, nav, or component touched — confirmed via `git status`.
- [x] `npm run build` succeeds with zero errors.

## Files changed

- Modified: `src/pages/el-estuche.astro` (full merch catalog implementation)
- Added (committed in the first, mission-handoff commit): `src/assets/merch/*.webp` (6 files), `missions/MISSION_13_el-estuche-merch.md`

## Judgment calls flagged

- **Intro copy wording**: wrote one short functional sentence describing the actual ordering mechanism (made-to-order, color chosen by email) rather than asking first, since the mission explicitly said to keep it short and only ask if something *more* than a functional label was wanted — this reads as a plain instruction, not invented marketing copy.
- **Image approach**: used `astro:assets`'s `<Image>` component rather than asking, since checking first confirmed `sharp` (the dependency it needs) was already present in the project's existing dependency tree with zero `package.json`/config changes required — the mission's own fallback instruction only asked to check in first if it *wasn't* already available.
- **Fixed 4:3 aspect-ratio + `object-fit: contain` boxes** (not a per-item custom aspect ratio, and not `object-fit: cover`): chosen after directly inspecting all 6 source photos and confirming they share one exact background color, making letterboxing invisible — the simplest option consistent with "images must be responsive" without introducing per-item special-casing.
