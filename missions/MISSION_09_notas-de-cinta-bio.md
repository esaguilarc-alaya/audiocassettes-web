# Mission 09 — Notas de Cinta: Real Band History + Home Teaser Copy

**Project:** audiocassettes-web
**Owner:** Esteban Aguilar (GitHub account also hosting CIE)
**Executor:** Claude Code
**Reviewer:** Claude (chat) — verifies against this mission before it is considered closed

## Objective

Replace the `notas-de-cinta` placeholder with the real band history Esteban wrote and approved, and update the home-page teaser block that links to it with new approved title/text. Both pieces of copy below are final and confirmed — port them verbatim, no paraphrasing, no trimming, no reordering.

## Scope (in)

1. **`notas-de-cinta.astro` — real bio copy.** Replace the current placeholder (`<p>[ contenido real de esta sección — Misión TBD ]</p>`) with the following approved copy, rendered as the paragraphs shown (one `<p>` per paragraph, in this exact order, matching the line breaks below). The final line is a signature and should be visually set apart from the body paragraphs (e.g. an `<p>`/`<footer>` with lighter styling or an em-dash treatment) — your judgment on exact markup, but the text itself must not change. Keep the page's existing `<h1>` and `PageAccentBar` exactly as they are; this mission only replaces the placeholder paragraph with the copy below.

   > En 1987 Ronald Reagan hacía de las suyas como presidente de Estados Unidos, Óscar Arias celebraba su Nobel de la Paz, mientras miles de costarricenses perdieron sus ahorros de la noche a la mañana con la quiebra masiva de empresas financieras no bancarias... esa parte sí suena muy 2026.
   >
   > Pero bueno, los teléfonos tenían 6 dígitos y pocos hogares contaban con teléfono propio — eso sí cambió.
   >
   > La cosa es que ese año conocí a Gerson entrando a primer grado de escuela. Ya para sexto grado éramos mejores amigos, y a los 13 años decidimos tener nuestra banda de rock — el único problema es que no sabíamos tocar ningún instrumento...
   >
   > Así que bueno, lo resolvimos llevando clases de guitarra los dos a la vez.
   >
   > Pronto empezamos a crear nuestras propias canciones, letras. Mucho de ese material lo tenemos guardado, en cassettes, en cuadernos viejos, y muy a pesar de la juventud todavía encontramos sabiduría ahí.
   >
   > En nuestra época de colegio compartimos con más personas ese interés, y de ahí conocimos gente con la que de una u otra forma hemos compartido por el resto de nuestra vida.
   >
   > Grabamos nuestras primeras canciones en estudio con Iván en la guitarra y nuestro amigo y miembro honorario por siempre, Rolando, en la batería.
   >
   > En los últimos 20 años grabamos dos discos en estudio que todavía no han sido oficialmente lanzados.
   >
   > Tuvimos la oportunidad de conocer gran cantidad de músicos locales, de los conocidos de la escena y de los desconocidos.
   >
   > Compartimos banda con muchos nombres que solo voy a mencionar sin apellido: Roberto, Alonso, Vladimir, Víctor, Erick, Chris, Álvaro... algunos de ellos todavía en nuestra vida. Con todos nos divertimos y compartimos el momento íntimo de crear, y por eso eternamente agradecidos.
   >
   > Hoy volvemos con una agrupación de viejos conocidos como Iván, una dulzura de hombre, y un nuevo amigo que nos devolvió las ganas y la paz: nuestro querido Cali.
   >
   > Y así nos disponemos a empezar esta gira y finalmente compartir lo que hemos creado durante tanto tiempo y con tantas personas lindas. Gracias por estar por acá, nos vemos por ahí.
   >
   > — Esteban

2. **`NotasDeCintaTeaser.astro` (home page) — new title/text.** Replace only the `<h3>` and the body `<p>` (not the `.eyebrow` "notas de cinta", not the button text/link) with:

   - `<h3>`: `amigos haciendo arte desde nuestra casa`
   - body `<p>`: `Caracterizados por amistades largas y pretensiones cortas, conocé la historia de Audiocassettes.` (note the accent on "conocé", matching the voseo already used in this same block's existing button copy — this is a one-word spelling normalization of what Esteban sent, confirmed as part of this mission, not a wording change)

   The `[ espacio para foto de banda ]` placeholder block and the "leer notas" button/link are untouched.

## Scope (out — do not touch in this mission)

- No changes to `notas-de-cinta.astro`'s `<h1>`, `PageAccentBar`, or the page's accent color mapping (Mission 02).
- No changes to any other home-page teaser block (`Hero`, `MusicTeaser`, `LasCintasPreview`).
- No changes to nav, other pages, or any other component.
- No new npm dependencies.

## Acceptance Criteria

- [ ] `notas-de-cinta.astro` renders the full approved bio copy verbatim, in the exact paragraph order given above — diffed word-for-word against this document, not eyeballed.
- [ ] The closing signature line ("— Esteban") is visually distinguished from the body paragraphs.
- [ ] `notas-de-cinta.astro`'s `<h1>` and accent bar are unchanged.
- [ ] `NotasDeCintaTeaser.astro`'s `<h3>` reads exactly "amigos haciendo arte desde nuestra casa" and its body `<p>` reads exactly "Caracterizados por amistades largas y pretensiones cortas, conocé la historia de Audiocassettes." — diffed against this document.
- [ ] `NotasDeCintaTeaser.astro`'s eyebrow text, "leer notas" button label, and its link to `/notas-de-cinta` are unchanged.
- [ ] No other page or component was touched.
- [ ] Renders cleanly with no jank or overflow at 320px, 375px, and 768px (Rule 9), including the longer bio copy on a narrow viewport.
- [ ] `npm run build` succeeds with zero errors; no new dependencies.

## Standing Engineering Control Rules for audiocassettes-web

Rules 1–9 are carried over unchanged from `missions/MISSION_08_las-cintas-viewmaster.md`.

1. **Brand fidelity rule.** Every color, font, and section name must trace to the brand manual or an explicit decision recorded in the project brief or a mission document — never invented or approximated. Star Avenue → Fredoka remains the one flagged exception.
2. **Verification over self-report.** The reviewer checks real files and actual behavior — a response file's claims are checked, not trusted.
3. **Placeholder discipline.** Not applicable — this mission replaces placeholders with real, confirmed content.
4. **Mission boundary discipline.** Missions have explicit in/out scope. Code should not get ahead and build next-mission work early.
5. **Static-first constraint.** The site is static by default. This mission introduces no server/API/database dependency.
6. **No unlicensed third-party assets.** No new fonts, icon packs, or snippets.
7. **Diff before review.** The real implementation is diffed word-for-word against this mission's copy, not just eyeballed for vibes.
8. **Mission handoff protocol.** Cowork authors the mission file and places it in the project's `missions/` directory locally, but does **not** commit or push it. Cowork hands off to Code with a prompt naming the mission file/ID. Code commits and pushes the mission file first, confirms the push, *then* executes the mission's scope, writes a **separate** response file at `missions/MISSION_0X_response.md`, and commits and pushes that response file too.
9. **Mobile/responsive by default.** Every mission ships mobile-friendly and responsive by default, checked at 320px, 375px, and 768px.

## Open Questions for Esteban (Code should ask, not assume)

None — both pieces of copy are final and confirmed. If the exact markup for setting the signature line apart, or minor typographic treatment of the bio paragraphs, needs a judgment call, use your own design judgment consistent with the rest of the site and record the choice in the response file.
