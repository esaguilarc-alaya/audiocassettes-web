# audiocassettes-web

The official website for the band audiocassettes, built as a static Astro site with a dedicated design system (color tokens, typography, and a shared header/footer/stripe device) and stub pages for every planned section of the site — home, lado a / lado b, las cintas, notas de cinta, el estuche, and sintoniza — established here so future missions can layer real content and features on a consistent foundation without redoing the scaffold.

## Brand fidelity note

**Fredoka and Montserrat are a temporary substitution for the band's actual corporate font, Star Avenue**, pending delivery of the real font file. This is the one approved exception under this project's brand fidelity rule (see `missions/MISSION_01_scaffold.md`) — every other color, font, and section name in this codebase must trace to the brand manual or to an explicit decision recorded in a mission document.

The brand manual has not yet been added to this repo as of this mission. Once it is, the "four-color stripe device" color selection in `src/components/Stripe.astro` (currently a placeholder pick of four of the six brand tokens) should be checked against it and corrected if it disagrees.

## Project structure

```text
/
├── missions/                    # this project's build history, one file per mission
├── src/
│   ├── layouts/
│   │   └── BaseLayout.astro     # shared <head>, fonts, Header/Footer
│   ├── components/
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── Stripe.astro         # four-color stripe device
│   │   ├── Hero.astro
│   │   ├── MusicTeaser.astro
│   │   ├── LasCintasPreview.astro
│   │   ├── NotasDeCintaTeaser.astro
│   │   ├── SintonizaTeaser.astro
│   │   └── EmailSignupForm.astro # non-functional UI, no backend
│   ├── pages/
│   │   ├── index.astro
│   │   ├── lado-a-lado-b.astro   # stub
│   │   ├── las-cintas.astro      # stub
│   │   ├── notas-de-cinta.astro  # stub
│   │   ├── el-estuche.astro      # stub
│   │   └── sintoniza.astro       # stub — static only, no backend
│   └── styles/
│       ├── tokens.css           # design tokens (colors, fonts, spacing)
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
