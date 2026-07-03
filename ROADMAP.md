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
of clips in `assets/`. Revisit as a batch job. Jason is researching voice options.

### 2. The Clock — SHIPPED (game #12)
Analogue clock face (drawn in SVG in-code, no image assets) plus a digital
readout, toggled at the top. Reuses `DATA.times` (24 times). Accent- and
synonym-tolerant, weak-spot aware. Already handles "menos" phrasing.

Possible extensions: more times; a dedicated "y cuarto / y media / menos cuarto"
phrasing focus; visualising numbers similarly.

### 3. El / La with images — IN DISCUSSION
Overhaul the dry El/La game: show a **picture of an object**, choose el or la.
Pairing gender with an image is how it sticks.

Decisions made:
- Use **dedicated generated images** (preferred over auto-cropping objects out of
  the existing scenes, which is free but rough for small objects).
- Efficiency trick: generate **object sheets** (a grid of ~9 objects on white, no
  labels), and the assistant slices each sheet into individual images. ~4 sheets
  ≈ 36 objects; the ~92 gendered nouns in `articleNouns` fit in a handful of sheets.

Open decisions before generating:
- **Style**: flat icons on white vs the realistic house/city render. (Flat reads
  more clearly at small size for a quick el/la call.)
- **Scope / order**: suggested to start with the house + city nouns (most used).

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

## Open questions / backlog

- **Errands hit-area/ring** (game #13): the place hotspots were sized for the
  City naming game, so the green ring and the click tolerance are smaller than
  the buildings. Fix: for Errands, draw a larger ring and widen the click
  hit-area (bump the `* 1.5` tolerance in `onClick`, and/or store a bigger radius
  per place for this game). No re-calibration needed.
- Idea 3: flat vs realistic style, and object scope/order.
- Fold the anti-repeat picker into the **Recognition** flashcards; make the
  **Dialogue** game less strictly linear.
- Audio voice/service choice (idea 1).
- Housekeeping: temp preview files and `calibrate.html` can be removed at commit;
  README/HANDOVER counts should say 12 games; a commit is due from Jason's terminal.
