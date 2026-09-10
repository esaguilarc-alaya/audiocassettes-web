# audiocassettes-web

The official website for the band audiocassettes, built as a static Astro site with a dedicated design system (color tokens, typography, and a shared header/footer/stripe device) and stub pages for every planned section of the site — home, lado a / lado b, las cintas, notas de cinta, el estuche, and sintoniza — established here so future missions can layer real content and features on a consistent foundation without redoing the scaffold.

## Brand fidelity note

**Fredoka and Montserrat are a temporary substitution for the band's actual corporate font, Star Avenue**, pending delivery of the real font file. This is the one approved exception under this project's brand fidelity rule (see `missions/MISSION_01_scaffold.md`) — every other color, font, and section name in this codebase must trace to the brand manual or to an explicit decision recorded in a mission document.

The brand manual (`reference/Audiocassettes_Manual de Identidad.pdf`) and the approved home page mockup (`reference/home-mockup-reference.html`) are both in this repo. Confirmed against them:
- All six color tokens in `src/styles/tokens.css` match the manual's palette page exactly.
- Star Avenue → Fredoka is the correct substitution call (the manual names Star Avenue as the corporate font, Montserrat as complementary).
- The four-color stripe device in `src/components/Stripe.astro` is **brown-tape → polaroid-sunset → view-master → retro-pop** (confirmed from the manual's "elementos complementarios" page and the mockup's own CSS) — an earlier draft of this component had guessed a different four before the manual was available; that guess has been corrected.

The isologo (the "audiocassettes" wordmark with a concentric-circle cassette-reel icon replacing the "o", manual p.7–9) is implemented as an inline SVG in `src/components/Isologo.astro`, used in the header. The ring colors and order (retro-pop → polaroid-sunset → view-master → brown-tape hub) match the manual's palette exactly; the letterforms are Fredoka's rather than Star Avenue's, per the substitution above.

## Project structure

```text
/
├── missions/                     # this project's build history, one file per mission
├── reference/                    # approved source-of-truth artifacts (not built/deployed)
│   ├── home-mockup-reference.html
│   └── Audiocassettes_Manual de Identidad.pdf
├── src/
│   ├── layouts/
│   │   └── BaseLayout.astro      # shared <head>, fonts, Header/Footer
│   ├── components/
│   │   ├── Header.astro
│   │   ├── Footer.astro          # the "sintoniza" block — real site footer, every page
│   │   ├── Isologo.astro         # wordmark + cassette-reel icon (replaces the "o")
│   │   ├── Stripe.astro          # four-color stripe device
│   │   ├── Hero.astro
│   │   ├── MusicTeaser.astro
│   │   ├── LasCintasPreview.astro
│   │   ├── NotasDeCintaTeaser.astro
│   │   └── EmailSignupForm.astro # non-functional UI, no backend
│   ├── pages/
│   │   ├── index.astro           # ported from reference/home-mockup-reference.html
│   │   ├── lado-a-lado-b.astro   # stub
│   │   ├── las-cintas.astro      # stub
│   │   ├── notas-de-cinta.astro  # stub
│   │   ├── el-estuche.astro      # stub
│   │   └── sintoniza.astro       # stub — static only, no backend
│   └── styles/
│       ├── tokens.css            # design tokens (colors, fonts, spacing)
│       └── global.css
└── package.json
```

## Commands

All commands are run from the root of the project, from a terminal:

| Command           | Action                                      |
| :----------------- | :------------------------------------------ |
| `npm install`       | Installs dependencies                        |
| `npm run dev`       | Starts local dev server at `localhost:4321`  |
| `npm run build`     | Build the production site to `./dist/`       |
| `npm run preview`   | Preview the build locally                    |

## Status

This is a foundation-only scaffold (Mission 01). No real photos, video, audio, or copy — placeholder blocks stand in everywhere real band content will go, and no backend, database, or API routes exist yet, including for `sintoniza`. See `missions/MISSION_01_scaffold.md` for full scope and the standing engineering control rules that govern every mission after this one.

## License

MIT — see `LICENSE`.
