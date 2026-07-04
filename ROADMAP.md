# Spanish Game — Roadmap & Ideas

Forward-looking notes for the game overhaul work. For architecture, data shapes
and the build/test workflow, see `HANDOVER.md`. Spain Spanish throughout; no em
dashes in content.

Status legend: **SHIPPED** · **PLANNED** · **IN DISCUSSION** · **PINNED**

---

## Game overhaul ideas (July 2026 discussion)

### 1. Audio — PINNED
Goal: hear the Spanish everywhere, plus a new **listening / dictation mode** (it
plays a word or sentence, you type what you hear) — trains comprehension, which
nothing currently does.

Decision: **do NOT use the browser Web Speech API** — it's robotic and
inconsistent for Spanish (confirmed from prior experience). Instead
**pre-generate the clips** once with a good neural voice (ElevenLabs / Google /
Azure / similar), bundle them as small audio files, and just play them. No live
synthesis, no backend needed for playback.

Effort: a one-time batch generation over the vocab/sentence lists, plus a few MB
of clips in `assets/`. Revisit as a batch job.

**Voice recipe (ElevenLabs Voice Design) — confirmed working for Castilian:**
Follow their template and state the dialect in Spanish in the first line. The word
"accent" / "strong" causes seseo/Latin-American drift — avoid it. Push Guidance
Scale high (~45-55%) and use a longer *Spanish* preview text. Working prompt for
the character Lucía:
```
Native Spanish, español europeo de España (sin rasgos de español latinoamericano).
Female, in her early 20s. Studio quality. Persona: chica joven y simpática, una
vecina cercana y servicial. Emotion: warm, friendly, upbeat. Light, bright timbre
with natural expressive intonation and a subtle smile in the voice, relaxed
conversational pace, clean noise-free signal.
```
Diagnostic check: "th" on plaza/zapatería/cruzando/cerca, rolled r in barrio, ñ in
acompaño. Per-character voices could be designed the same way (name/persona swapped).

### 2. The Clock — SHIPPED (game #12)
Analogue clock face (drawn in SVG in-code, no image assets) plus a digital
readout, toggled at the top. Reuses `DATA.times` (24 times). Accent- and
synonym-tolerant, weak-spot aware. Already handles "menos" phrasing.

Possible extensions: more times; a dedicated "y cuarto / y media / menos cuarto"
phrasing focus; visualising numbers similarly.

### 3. El / La with images — SHIPPED
Overhaul the dry El/La game: show a **picture of an object**, choose el or la.
Pairing gender with an image is how it sticks.

Shipped: the game keeps its full 98-noun `articleNouns` pool. `DATA.nounImages`
maps 81 of those nouns to images (73 flat object icons in `assets/Objects/`, 8
rooms reusing the house scenes). When a noun has an image it shows in a white card
with a label; otherwise the word shows as before (layer, don't replace). A label
toggle over the card switches English (default) / Español (italic, harder) / Off
(expert). Labels are code-drawn so icons stay clean. Verified via jsdom (images
render for image nouns, word fallback otherwise, toggle works, all 81 image nouns
covered by the pool).

Decisions made:
- Use **dedicated generated images** (preferred over auto-cropping objects out of
  the existing scenes, which is free but rough for small objects).
- Efficiency trick: generate **object sheets** (a grid of ~9 objects on white, no
  labels), and the assistant slices each sheet into individual images. ~4 sheets
  ≈ 36 objects; the ~92 gendered nouns in `articleNouns` fit in a handful of sheets.

Decisions locked:
- **Style**: flat icons on white (clean at small size). Generate as **individual
  images** (not sheets — slicing is fragile), drop in `assets/Objects/`.
- **Layer, don't replace** (IMPORTANT): the El/La game keeps its full
  `articleNouns` pool (98 nouns); show an image + label when the noun has one,
  else fall back to the word (today's behaviour). Image-only would drop ~60
  non-drawable nouns (día, universidad, familia...). Images enrich a subset; the
  pool never shrinks. Add any icon-object missing from `articleNouns` with its gender.
- **Label under the image**: English by default; toggle to Spanish (harder — tests
  the irregulars like la mano, el mapa); optional no-label expert mode. Labels are
  code-drawn text, so generated icons stay clean/unlabelled.
- General principle: images are an **enhancement layer over text content, never a
  replacement** (same as house/city words still living in vocab).

### 4. Place Match → "Click the place" with characters — SHIPPED (game #13 "Errands")
A named character voices a need in a speech bubble ("¡Necesito unas piñas!") and
the player clicks the right building in a city scene; the click is checked against
the calibrated place hotspot. 10 characters (`DATA.characters`, images in
`assets/Charecters/`, backgrounds flooded to exact site cream so they blend),
26 requests (`DATA.cityRequests`). Uses the smart picker. The old text-based
placeAction game still exists alongside it.

Original plan below (kept for reference):


Overhaul the abstract Place Match. A **character asks for something** ("¡Necesito
unas piñas!") and you **click the right building** in a city scene.

Reuses the 6 city scenes and the already-calibrated **place coordinates**, so
checking the click is free. Characters come from a single generated sheet (below);
the assistant slices them and adds speech bubbles with request text it authors.

---

## Generation prompts

### Character sheet (idea 4) — GENERATED
A full cast has been produced: **43 named, numbered, full-body characters**
("Placa de Personajes"), diverse and in a consistent warm illustrated style on
white. Names are baked onto the sheet (Lucía, Mateo, Elena, Diego, ... Beatriz).

**Approach decided:** don't slice the flat sheet. Instead **regenerate ~8-12
chosen characters individually**, each full-body on a solid **`#fbf8f3`** (site
cream) background so the edges blend into the page (no cutout/transparency), with
a **generic "asking for help" pose** (not tied to any specific request) so one
image is reusable across many requests. Keep the character name in data so
requests can be attributed ("Sofía necesita sellos"). Speech bubble + request text
are added in code, not baked into the image.

Per-character prompt (feed the cast sheet as reference):
```
Regenerate character #N "<Name>" from this cast sheet as a single full-body
figure, same warm illustration style, front-facing, friendly expectant
"asking for help" pose (one hand gesturing), on a solid flat #fbf8f3 cream
background. No text, no speech bubble, no name caption, no scene/props.
```

Original prompt used (for reference):
```
A grid of friendly illustrated full-body characters, evenly spaced on a plain
white background. Diverse ages, genders and appearances. Consistent warm
illustration style, front-facing, friendly expressions.
```

### Object sheet (idea 3) — DRAFT, pending style decision
```
A 3x4 grid of twelve everyday [household/city] objects, one per cell, evenly
spaced on a plain white background with clear gaps. [Flat, clean icon style / OR
realistic 3D render] matching a consistent set. Each object centred and clearly
recognisable. No text, no labels, no numbers.
```
(Assistant supplies the exact object list per sheet once style + scope are chosen.)

---

## Recently shipped (context)

- **12 games**: added The House, The City, The Clock (image/visual games).
- **Smart picker** (`pickSmart`): dedupes repeated words, anti-repeat cooldown so
  nothing recurs back-to-back, and weakness weighting per word = lightweight
  spaced repetition. Used by every selection-based game.
- **Accent traps**: the yellow "accent matters" feedback now warns when dropping
  an accent/ñ spells a different (or rude) word — e.g. año → ano.
- **Synonym support**: alternatives are accepted everywhere (nevera/frigorífico,
  televisión/televisor, salón/sala de estar, piso/apartamento, etc.) and kept out
  of each other's multiple-choice wrong options.
- **Course-word vocab sweep**: all En La Casa house words, plus city
  street/transport vocab (a `cityObjects` topic).
- **All UI in English**; only the learning content is Spanish.

---

## Adding content (stays easy)

Everything is data-driven — games read `DATA.*` arrays and select via `pickSmart`,
so new content is picked up automatically (balanced, anti-repeat, weak-spot aware).
To add:
- **A word**: append `{en, es}` to the right `DATA.vocab.<topic>` list. New topics
  appear as buttons automatically; add a `TYPE_BY_TOPIC` entry for Recognition.
- **A synonym**: one line in `SYNONYM_GROUPS`. An **accent trap**: one line in `ACCENT_TRAPS`.
- **A sentence / verb / dialogue / number / time**: append to the matching `DATA` array.
- **An errand request**: append `{place, es, en}` to `DATA.cityRequests` (place must
  match a city hotspot). **A character**: add to `DATA.characters` and drop the image
  in `assets/Charecters/` (flood its bg to `#fbf8f3`).
- **An image-game hotspot** (house/city object or place): the only step that needs a
  tool — run a calibrator to click the item for x/y/r, then add it to
  `houseScenes` / `cityScenes`.

## Open questions / backlog

- **Numbers game rework** — DONE: "Time & Numbers" is now **Numbers**. Time dropped
  (The Clock covers it); numbers are **randomly generated in context** (plain 0-9999,
  price in €, age, year), spelled by a tested `numberToSpanish()` and checked by a
  tolerant `numAccept()` (un/uno, drops euros/céntimos) — no fixed list, infinite
  practice. **Difficulty modes**: Easy (0-30, plain+age), Medium (0-100, +whole-euro
  prices), Hard (full range 0-9999, prices with céntimos, years). Possible extension:
  quantities with a noun (needs feminine agreement, una/veintiuna, doscientas) and a
  visual counting mode.
- **Errands hit-area / building boundaries** (game #13) — PINNED: a centre+radius
  circle can't match a rectangular building, so widening the radius isn't enough.
  Proper fix needs each building's actual footprint — a bounding box per place, or
  a polygon for irregular shapes — which means a richer calibration pass (capture
  box corners / draw a region, not a point). Then the click-hit test becomes
  point-in-box/polygon and the highlight can outline the building. Bigger job;
  revisit when we want to invest in it.
- Idea 3: flat vs realistic style, and object scope/order.
- Fold the anti-repeat picker into the **Recognition** flashcards; make the
  **Dialogue** game less strictly linear.
- Audio voice/service choice (idea 1).
- Housekeeping: temp preview files and `calibrate.html` can be removed at commit;
  README/HANDOVER counts should say 12 games; a commit is due from Jason's terminal.
