# spanishgame

A Spanish learning game (Spain Spanish: vosotros, "th" for c/z, *patata*,
*zumo*). Vanilla HTML/CSS/JS, no build step. Open `index.html` to play. The app
is a single `index.html` plus image assets in `assets/` (house rooms, city
scenes, characters) used by the image-based games.

Progress is saved in the browser via `localStorage` (key `jason-spanish-app-v1`).

## Deploy

Hosted via GitHub Pages at https://super-gill.github.io/spanishgame/ once Pages
is enabled (repo Settings > Pages > Branch: `main`, folder `/root`).

## Editing

`index.html` is organised top-to-bottom: CSS, then a `<body>` skeleton, then
data blocks (vocab, verbs, nouns, sentences, `houseScenes`, `cityScenes`, etc.),
then a `Games` object with 13 games, then `init()`. See `HANDOVER.md` and
`ROADMAP.md` for detail.

The newer games are image/visual-based: **The House** and **The City** (name a
highlighted room/place/object), **The Clock** (tell the time on an SVG clock),
and **Errands** (a character voices a need, click the right building). Hotspot
positions live in `DATA.houseScenes` / `DATA.cityScenes` as normalized x/y/r
coordinates. Selection across all games runs through `pickSmart` (dedupe +
anti-repeat + weakness weighting).
