# Mission 13 — El Estuche: Merch Catalog with Per-Item Order-by-Email

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Replace the `el-estuche` placeholder with a real merch catalog of 6 items from the band's brand manual. There's no real purchase flow — each item gets its own "ordenar" button that opens the visitor's email client with a pre-filled message, since orders are handled manually and made to order (colors are chosen by the customer via the email, not a UI picker). No prices are shown.

## Assets (added by this mission)

Six product images, already cropped and compressed from the brand manual's merch mockup sheet, placed at `src/assets/merch/`:

- `camiseta-audiocassettes.webp` — raglan t-shirt (cream body, blue sleeves), wordmark on chest
- `gorra-audiocassettes.webp` — trucker cap (cream/brown crown, red brim), wordmark on front
- `pin-audiocassettes.webp` — round pin/button, concentric-ring brand-color design
- `taza-logo-audiocassettes.webp` — enamel mug, plain wordmark + isologo design
- `taza-cerro-audiocassettes.webp` — enamel mug, alternate design (Cerro Chirripó-style mountain graphic in brand colors)
- `cordon-audiocassettes.webp` — lanyard/cordón, striped brand-color pattern with repeated wordmark

Use Astro's built-in image handling (`astro:assets`/`<Image>`) if the project's Astro version supports it for local images under `src/assets/`; otherwise place them under `public/images/merch/` and reference them directly — your call, record which approach and why in the response file. Either way, images must be responsive (no fixed pixel dimensions that overflow at 320px) and should have real `alt` text describing each item in Spanish.

## Scope (in)

1. **Six product cards**, one per item above, each showing: the product image, the item's name (in Spanish — e.g. "camiseta", "gorra", "pin", "taza — diseño clásico", "taza — diseño cerro", "cordón"), and its own "ordenar" button/link.
2. **No prices displayed anywhere on this page.**
3. **Per-item order button = a `mailto:` link**, not a form or any backend integration (Rule 5 — static-first). Use a single placeholder order address defined ONCE as an easy-to-find constant (e.g. `pedidos@audiocassettes.com` — flag clearly in a code comment that this is a placeholder Esteban will replace once he sets up a real inbox). Each item's `mailto:` link should pre-fill:
   - `to`: the placeholder order address
   - `subject`: something like `Pedido: [nombre del item]`
   - `body`: a short pre-filled message in Spanish naming the item and inviting the customer to specify their preferred color in the reply (since items are made to order — there is no color-picker UI to build, just this invitation in the email body text)
4. **Layout**: a simple grid/card layout for the 6 items, consistent with the site's existing card/section patterns and using only established brand tokens for any card chrome, borders, or accents (Rule 1) — the product photos themselves carry their own color, nothing new needs to be invented.
5. Update the page's `<h1>`/intro copy minimally if needed to introduce the catalog (e.g. a one-line intro above the grid) — keep it short, and if you want specific wording beyond a functional label, ask rather than inventing marketing copy.

## Scope (out — do not touch in this mission)

- No shopping cart, checkout, payment integration, or inventory/stock logic of any kind.
- No color-picker UI — color choice is handled entirely via the email body text.
- No prices, anywhere.
- No changes to other pages, nav, the sintoniza footer, el librito, las-cintas, or the transition effect.
- No new npm dependencies beyond what's already available for Astro's built-in image handling.

## Acceptance Criteria

- [ ] All 6 items render with their correct image, Spanish name, and no price shown anywhere.
- [ ] Each item has its own working `mailto:` link with the item's name in the subject and a color-preference invitation in the body — verified by inspecting the actual `href` values, not just that a button exists.
- [ ] The placeholder order email address appears in exactly one place in the source (a single constant), clearly commented as a placeholder to replace — confirm by grepping the codebase for the address.
- [ ] No form, backend call, or third-party checkout script was introduced (Rule 5) — confirm by diffing `package.json`/`package-lock.json` (should be unchanged unless Astro's image support needs a new built-in integration already bundled with Astro).
- [ ] Images are responsive and don't cause horizontal overflow at 320px, 375px, or 768px (Rule 9) — re-verify, don't assume.
- [ ] All new colors/chrome trace to established brand tokens (Rule 1).
- [ ] No other page, nav, or component was touched.
- [ ] `npm run build` succeeds with zero errors.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_12_el-librito-redesign.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** The order email address IS an intentional, clearly-flagged placeholder per this mission's own instructions — not a violation of this rule, but it must be a single easy-to-find constant, not scattered/hardcoded per item.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early (no cart, no payment, no color picker).
5. **Static-first constraint.** The site is static by default. `mailto:` links introduce no server/API/database dependency.
6. **No unlicensed third-party assets.** The 6 product images are Esteban's own brand-manual assets (already cropped/compressed for this mission) — no additional third-party images, icon packs, or fonts.
7. **Diff before review.** The real implementation is diffed against this mission's scope — not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it (plus the image assets) in the project's `missions/`/`src/assets/merch/` directories locally, but does **not** commit or push them. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file and asset files first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

- If specific intro copy is wanted above the grid (beyond a short functional label), ask rather than inventing marketing copy.
- If Astro's image optimization setup isn't already configured in this project, ask whether to add it or just serve the WebP files as static assets from `public/` — don't add new dependencies without checking in first.
