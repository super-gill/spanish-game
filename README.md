# spanishgame

A Spanish learning game (Spain Spanish: vosotros, "th" for c/z, *patata*,
*zumo*). Vanilla HTML/CSS/JS, no build step. Open `index.html` to play. The app
is a single `index.html` plus a set of room images in `assets/` used by the
La Casa game.

Progress is saved in the browser via `localStorage` (key `jason-spanish-app-v1`).

## Deploy

Hosted via GitHub Pages at https://super-gill.github.io/spanishgame/ once Pages
is enabled (repo Settings > Pages > Branch: `main`, folder `/root`).

## Editing

`index.html` is organised top-to-bottom: CSS, then a `<body>` skeleton, then
data blocks (vocab, verbs, nouns, sentences, `houseScenes`, etc.), then a
`Games` object with 10 games, then `init()`. See `HANDOVER.md` for a detailed map.

The 10th game, **La Casa**, is image-based: it shows dollhouse cutaway room
images (`assets/<room>.jpg`) and quizzes either the room name or a highlighted
object. Object positions live in `DATA.houseScenes` as normalized x/y/r
coordinates, produced by clicking each item in a calibration tool.
