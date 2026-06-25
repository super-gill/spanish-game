# spanishgame

A single-file Spanish learning game (Spain Spanish: vosotros, "th" for c/z,
*patata*, *zumo*). Vanilla HTML/CSS/JS, no build step. Open `index.html` to play.

Progress is saved in the browser via `localStorage` (key `jason-spanish-app-v1`).

## Deploy

Hosted via GitHub Pages at https://super-gill.github.io/spanishgame/ once Pages
is enabled (repo Settings > Pages > Branch: `main`, folder `/root`).

## Editing

`index.html` is organised top-to-bottom: CSS, then a `<body>` skeleton, then
data blocks (vocab, verbs, nouns, sentences, etc.), then a `Games` object with
9 games, then `init()`. See `HANDOVER.md` for a detailed map.
