# Musical Reference Chart

A single-file, offline web app for music production reference. It shows pitch and
frequency tables and tempo-division delay times, filtered by song key, and saves a
per-song summary sheet as a .jpg image.

Inspired by the iOS app **Tempo** (by Frozen Ape).

- **Live:** https://miuglitch.github.io/musicalrefchart/chart.html
- **Repo:** https://github.com/miuglitch/musicalrefchart

---

## 1. Quick facts

| | |
|---|---|
| **Type** | Static web app: one HTML file, no build step |
| **Entry point** | `chart.html` |
| **Dependencies** | None. No frameworks, no CDN, nothing fetched. The font is embedded in the file (§6) |
| **Runs offline** | Yes. Open the file straight from disk (`file://` works) |
| **Browser target** | Modern desktop Chrome / Edge / Firefox, plus phones through a separate mobile layout at 700 px wide or less (§3 Mobile layout) |
| **Size** | ~740 lines, ~206 KB (about 175 KB of that is the embedded font) |
| **Author** | Magnus (Type 2 Studio) |

**Design:** Teenage Engineering K.O. II hardware look. A black display header with
big white readouts sits over a light warm-grey body, with dark grey frames and signal
orange, red, blue and dark cyan accents. Text is Inter Display at several weights.
Clean contrast, **no glow**. A dark mode keeps the same header and turns the body into
a black display. It should feel like studio gear, not a web page.

---

## 2. How to run it

Open `chart.html` in a browser. No server, no install. The hosted copy is at the
Live link above (§9).

---

## 3. What the app does

### Header (fixed, always visible)

Top left: the Miu logo (inline SVG, `fill="currentColor"`, light grey) with
**REFERENCE CHART** under it in orange. Then:

| Field | ID | Default | Notes |
|---|---|---|---|
| Song | `#song` | *(empty → "Untitled")* | Also updates `document.title` live |
| Tempo | `#bpm` | `128` | BPM. Drives all tempo divisions. **DAW-style drag:** click, hold and drag up/down to change it live (1 BPM per 4 px, Shift = 0.1 steps, clamped 20–999). A plain click lets you type |
| Sample rate | `#srsel` + `#sr` | `96000` | Menu of common rates: 11025, 16000, 22050, 32000, 44100, 48000, 88200, 96000, 176400, 192000, 352800, 384000. The last option, **Custom…**, shows the `#sr` field for any value. A blank custom value → Samples column shows `N/A` |
| Tuning ref | `#a4` | `440` | A4 in Hz. Drives all pitch calculations |
| Key root | `#keyroot` | All | `""` (All) or semitone `0`–`11` |
| Key mode | `#keymode` | Major | 35 scales/modes, built from `SCALE_LIST` (§4) |

### Side strip (left of the tables)

One vertical strip, shared by every view. All labels are capitals with centred text.
The top edge of ROOT lines up with the underline of the table header rows.

| Button | ID / class | Default | Notes |
|---|---|---|---|
| ROOT / DOTTED / TRIPLETS | `.tog.root` `.tog.dot` `.tog.tri` | on | Show/hide the straight, dotted and triplet tempo rows |
| *(gap)* | `.strip-gap` | | |
| PRINT | `#printBtn` | | Downloads the summary as a .jpg in the current theme (§5) |
| PITCH / TEMPO | `.vbtn[data-view]` | both on | Show/hide each table independently. Both on = side by side. The last one on can't be turned off |
| DARK / LIGHT | `#themeBtn` | light | Switches the theme; remembered in `localStorage` (key `mrc-theme`) |

### Layout

- The header is one centred, wrapping row (`.hd-row`). A `ResizeObserver` keeps
  `#body` sitting right under it.
- Header content (`.hd-inner`) and the table area (`#body`) are **80% of the window
  width, with a 10% gutter each side**. No max-width cap.
- The side strip and the visible tables are **centred as a group**
  (`#body{justify-content:center}`). Tables are `width:auto` and hug their content.
- All table header rows are a fixed 60 px tall, so the tempo and pitch headers,
  their coloured underlines and the first data rows line up.

### Pitch table

All 128 MIDI notes (C-1 to G9), filtered to the selected key if one is set.

| Column | Value |
|---|---|
| Note | Name + octave, e.g. `C# 4` (sharps in blue) |
| Frequency | Hz, 3 decimals |
| Cycle | ms, 3 decimals: one full waveform period |
| Wavelength | metres, 3 decimals, at 343.2 m/s (air, 20 °C) |
| Samples | Samples per cycle at the chosen sample rate. `N/A` if blank |

Numbers are right-aligned so the decimals line up. A4 (orange) and C4 (blue) get a
badge and a tinted row; each C row has a heavier top rule.

### Tempo table

21 divisions: `1/1` … `1/64`, each straight, dotted (`D`) and triplet (`T`).

| Column | Value |
|---|---|
| Division | e.g. `1/8D`. Black (white in dark mode) = straight, cyan = dotted, orange = triplet |
| Length ms | 2 decimals |
| Length seconds | 4 decimals *(Tempo-only view)* |
| LFO Hz | 4 decimals: the rate that completes one cycle per division |

### Views (from the PITCH / TEMPO toggles, `currentTab()`)

- **Both on** *(default)*: `#pane-both`, compact tempo table and pitch table side by
  side. On load the pitch column scrolls to A4.
- **Pitch only**: `#pane-pitch`
- **Tempo only**: `#pane-tempo`, with the extra seconds column

### Mobile layout (window 700 px wide or less)

A separate layout for phones: one `@media (max-width:700px)` block plus a little JS.
Above 700 px nothing changes, and desktop renders exactly as before. Chosen Nov 2026
from four mockups (this was "mobile 1"; all four are in the local `mockups/` folder).

- **Compact header:** a 6-column CSS grid. Logo and song on row 1; Tempo, Sample rate
  and A4 as three readouts on row 2 (the units move into the labels through `.m-only` /
  `.d-only` spans); Key and scale on row 3. The header blocks have ids `#blk-song`
  `#blk-tempo` `#blk-sr` `#blk-a4` `#blk-key` for this. The tempo drag works with a thumb.
- **PITCH / TEMPO tabs:** `.mtabs` under the header (hidden on desktop). One table at a
  time: Pitch uses `#pane-pitch`, Tempo uses `#pane-tempo` (with seconds). Pitch opens
  first, scrolled to A4, and re-centres on A4 (or C4) after a key change.
- **Bottom bar:** `.mbar`, fixed to the bottom. `placeControls()` moves the strip's
  ROOT / DOTTED / TRIPLETS / PRINT / DARK buttons into it at phone width and back into
  the strip above 700 px. They're the same elements, so their listeners keep working.
  The strip's PITCH / TEMPO toggles are hidden on phones (the tabs replace them), and the
  row toggles are dimmed on the Pitch tab because they only affect Tempo.
- **Pitch table:** the Note column is sticky; Wavelength and Samples scroll sideways.
- Crossing 700 px (rotating a tablet, resizing) is live. Going back to desktop restores
  the previous PITCH / TEMPO state.

---

## 4. The maths

```js
SOS = 343.2                              // speed of sound, m/s

frequency   = A4 * 2 ** ((midi - 69) / 12)
cycleMs     = 1000 / frequency
wavelength  = SOS / frequency
samples     = sampleRate / frequency

beatMs      = 60000 / BPM                // one 1/4 note
divisionMs  = multiplier * beatMs        // multiplier from DIVS[]
lfoHz       = 1000 / divisionMs
```

`DIVS[]` multipliers are relative to a **quarter note** (`1/4` = `1`), so `1/1` = `4`,
`1/8` = `0.5`, dotted = ×1.5, triplet = ×2/3. Each entry is
`{n: name, m: multiplier, t: type}` where `t` is `0` straight, `1` dotted, `2` triplet.
`t` drives both the filter and the row colour.

### Scales (`SCALE_LIST`, semitone offsets from root)

35 scales and modes, in the same order and with the same interval sets as Ableton
Live's scale menu. `SCALE_LIST` is the single source: the Key mode menu is built from
it at load, and the `SCALES` / `SCALE_NAMES` lookups come from it. To add a scale, add
one row there.

```
maj  Major              [0,2,4,5,7,9,11]      dor4 Dorian #4          [0,2,3,6,7,9,10]
min  Minor              [0,2,3,5,7,8,10]      phd  Phrygian Dominant  [0,1,4,5,7,8,10]
dor  Dorian             [0,2,3,5,7,9,10]      mmin Melodic Minor      [0,2,3,5,7,9,11]
mix  Mixolydian         [0,2,4,5,7,9,10]      lyda Lydian Augmented   [0,2,4,6,8,9,11]
lyd  Lydian             [0,2,4,6,7,9,11]      lydd Lydian Dominant    [0,2,4,6,7,9,10]
phr  Phrygian           [0,1,3,5,7,8,10]      sloc Super Locrian      [0,1,3,4,6,8,10]
loc  Locrian            [0,1,3,5,6,8,10]      sp8  8-Tone Spanish     [0,1,3,4,5,6,8,10]
wt   Whole Tone         [0,2,4,6,8,10]        bhai Bhairav            [0,1,4,5,7,8,11]
hwd  Half-whole Dim.    [0,1,3,4,6,7,9,10]    hun  Hungarian Minor    [0,2,3,6,7,8,11]
whd  Whole-half Dim.    [0,2,3,5,6,8,9,11]    hira Hirajoshi          [0,2,3,7,8]
mblu Minor Blues        [0,3,5,6,7,10]        insn In-Sen             [0,1,5,7,10]
pmin Minor Pentatonic   [0,3,5,7,10]          iwa  Iwato              [0,1,5,6,10]
pmaj Major Pentatonic   [0,2,4,7,9]           kum  Kumoi              [0,2,3,7,9]
hmin Harmonic Minor     [0,2,3,5,7,8,11]      pels Pelog Selisir      [0,1,3,7,8]
hmaj Harmonic Major     [0,2,4,5,7,8,11]      pelt Pelog Tembung      [0,1,5,7,8]
ms3  Messiaen 3  [0,2,3,4,6,7,8,10,11]        ms4  Messiaen 4  [0,1,2,5,6,7,8,11]
ms5  Messiaen 5  [0,1,5,6,7,11]               ms6  Messiaen 6  [0,2,4,5,6,8,10,11]
ms7  Messiaen 7  [0,1,2,3,5,6,7,8,9,11]
```

### Chord tones (`chordTones(mode)` → `[root, 3rd, 5th, 7th]`)

Used only by the summary image, and computed from the scale:

- **7-note scales:** stack thirds (scale degrees 1, 3, 5, 7).
- **Everything else:** pick from the notes the scale has. 3rd = major 3rd if present,
  else minor 3rd. 5th = perfect 5th, else #5 (when the 3rd is major), else b5, else #5.
  7th = dim 7th when the 5th is b5, else minor 7th, else major 7th.
- `null` = no such tone; that section is left out of the summary (e.g. In-Sen and
  Iwato have no 3rd, Major Pentatonic has no 7th).

---

## 5. PRINT summary (.jpg)

**PRINT** draws the summary on an off-screen `<canvas>` (at 2× for sharpness) and
downloads it as `<song name> - summary.jpg`. Plain Canvas 2D + `canvas.toBlob()` and a
temporary `<a download>` link: no popup, no library, works offline and from `file://`.

- **Theme:** the image follows the current mode, so dark mode gives a dark image and
  light mode a light one. The PRINT button is solid orange in light mode and an orange
  outline in dark mode.
- **Title:** song name. **Meta line:** key, BPM, sample rate, A4, and a note for any
  hidden Root/Dotted/Triplet rows.
- **Left column:** tempo divisions (ms / seconds / LFO Hz), respecting the toggles.
- **Right column:** with a key selected, chord tones: Root / Third / Fifth / Seventh
  sections, each across **octaves 1–5**. With no key, all notes C0–C8.
- **Footer:** generation timestamp.

Canvas can't read the stylesheet, so `printSummary()` holds a light and a dark copy of
the theme colours in its `C` object. It draws in the embedded Inter Display, and the
click handler waits on `document.fonts.load()` first so the canvas never falls back.

---

## 6. Colours and font

Palette from a Teenage Engineering K.O. II, as CSS custom properties on `:root`:

```
--orange #ff5a1f   accent, BPM readout, A4, triplets, wavelength, delay ms, PRINT
--red    #e0322b   samples
--blue   #3a86d4   frequency, LFO Hz, C4, sharps
--cyan   #0f8a94   dark cyan: cycle, dotted
--frame  #3b3b3b   dark grey frames (header underline, divider between tables)
--disp   #0f0f0f   black display (header)
```

- The **header** looks the same in both themes.
- Everything below it uses semantic tokens (`--bg`, `--th-bg`, `--band-a/-b`,
  `--v-*` text colours, `--k-*` header accents …). `html[data-theme="dark"]`
  redefines them for dark mode. Text colours are darker shades of the accents on
  grey and brighter shades on black, so they stay readable.
- A tiny script in `<head>` applies the saved theme before first paint (no flash).
- Column header text is near-black (white in dark mode); each column's colour is its
  `border-bottom` accent.

### Octave banding

Pitch rows alternate two greys by octave, set on `td`:

```css
tr.oct0 td,tr.oct2 td { background: var(--band-a) }
tr.oct1 td,tr.oct3 td { background: var(--band-b) }
```

The class is `'oct' + ((oct + 1 + 40) % 4)`; the `+ 40` keeps it positive for negative
octaves.

> ⚠️ **Banding must be on `td`, never `tr`.** With `border-collapse: collapse`, a `<tr>`
> background doesn't paint through the cells and silently renders as nothing.

### Font

Inter Display (SIL Open Font License), subset to Latin and embedded as base64 woff2 in
`<style id="fonts">` at the top of the file: weights 300, 400, 500, 600, 700, 800. It's
registered as the private family name `IDM` so an installed Inter can't interfere.
Numbers use `font-feature-settings:'tnum'` so columns line up. To rebuild the subset:

```
pyftsubset InterDisplay-<Weight>.otf --flavor=woff2 \
  --unicodes="U+0020-007E,U+00A0-00FF,U+2013,U+2014,U+2018,U+2019,U+201C,U+201D,U+2022,U+2026,U+2190-2193,U+2212,U+00B7,U+266D,U+266F" \
  --layout-features='tnum,kern,liga,calt,case,zero,ss01,cv11'
```

then base64 it into the matching `@font-face`.

---

## 7. Code map

One file, in this order: `<style id="fonts">`, `<style>`, a tiny theme `<script>`,
markup, main `<script>`.

### Key functions

| Function | Purpose |
|---|---|
| `bpm()` `sr()` `a4()` `songName()` | Read header inputs. `sr()` reads `#srsel`, or `#sr` when it's on Custom…; `null` when blank |
| `getKeySet()` | `Set` of pitch classes 0–11, or `null` for All |
| `keyLabel()` | e.g. `"D#/Eb Dorian"`; empty if no key |
| `chordTones(mode)` | `[root, 3rd, 5th, 7th]` semitones for the summary (§4) |
| `pitchHTML(a4, sr, keySet)` | Builds the pitch rows |
| `tempoHTML(bpm, compact)` | Builds the tempo rows. `compact` drops the seconds column |
| `divOn(d)` | Whether a `DIVS` row type is shown (tables and summary) |
| `updateSRLabel(sr)` | The "@ 96,000 Hz" line under the Samples header |
| `refresh()` | Redraws the visible tables from the inputs |
| `currentTab()` / `applyView()` | Picks the pane from the PITCH/TEMPO toggles, shows it, redraws |
| `syncTogs()` | Sets the ROOT/DOTTED/TRIPLETS `.on` state |
| `printSummary()` | Draws the summary canvas and downloads the .jpg |
| `placeControls()` | Phone ⇄ desktop switch (`mq` = `matchMedia('(max-width:700px)')`): moves the buttons between `.strip` and `.mbar` |
| `setMobileView(v)` | Phones only: shows the `'pitch'` or `'tempo'` tab |
| `fitBody()` | Keeps `#body` between the header and, on phones, the bottom bar |

### State

Globals `showRoot`, `showDot`, `showTri`, `showPitch`, `showTempo` (all `true` at
start). Everything else is read from the inputs on each `refresh()`. The theme lives in
`data-theme` on `<html>` and in `localStorage`.

### Event wiring

- `#bpm` `#sr` `#a4` → `input` → `refresh()`. `#bpm` also has pointer handlers for the
  drag (an IIFE after the input listeners).
- `#srsel` → `change` → shows/hides `#sr`, then `refresh()`
- `#keyroot` `#keymode` → `change` → `refresh()`
- `#song` → `input` → updates `document.title` only
- `.tog` → one delegated listener on `document`
- `.vbtn` → flips `showPitch`/`showTempo` (never both off) → `applyView()`
- `#themeBtn` → toggles the theme and saves it
- `#printBtn` → waits for the font → `printSummary()`
- `.mtab` → `setMobileView()`; crossing 700 px (`mq` change) → `placeControls()`

### Duplicated markup

Each table exists in two panes, so there are **two tbodies each**: `#tb`/`#tbb`
(tempo) and `#pb`/`#pbb` (pitch). Change one, change the other. The side strip exists
once.

---

## 8. Conventions

- **One file, no build step, no external requests.** Never add a CDN script, a Google
  Font or a library.
- Every displayed number goes through `f2` / `f3` / `f4`. Never print a raw float.
- Styles are compact and grouped under `/* ── NAME ── */` banners.
- New colours go in as tokens, in both the light and the dark set, and in the
  `printSummary()` palette.

---

## 9. Repo, hosting and local copy

- **GitHub:** https://github.com/miuglitch/musicalrefchart (public). `main` is the
  source of truth.
- **Hosting:** GitHub Pages, deployed by the `.github/workflows/static.yml` Action on
  every push to `main`. The app is at `/chart.html`:
  https://miuglitch.github.io/musicalrefchart/chart.html. The bare Pages URL doesn't
  open the app.
- **How changes land:** Claude works in the cloud on a `claude/…` branch and opens a
  pull request. Magnus merges it on GitHub. Claude also copies the updated files into
  the local folder and reads them back to confirm the copy is right.
- **Local folder:** `D:\Dropbox\Claude\Musical Reference Chart` on Magnus's PC (Dropbox
  gives backup and version history). It holds `chart.html`, `README.md`, `CLAUDE.md`,
  plus local-only `mockups/` (the five Nov 2026 design options and the four mobile ones) and `Claude outputs/`
  (old drafts, git-ignored). There's an old `.git` folder in it that isn't connected
  to GitHub; ignore it.

---

## 10. Ideas not yet built

- Flats instead of sharps when the key calls for it (Bb, Eb …). Currently sharps only.
- Selectable octave range for the summary image (fixed at 1–5 for chord tones and
  C0–C8 for the no-key list).
- Save/load song presets.
- Reverb / pre-delay helper from the tempo grid.
- Haas / widening offsets (under 20 ms) as a derived column.
- DAW-style drag on the other number fields (tuning ref).

---

## 11. Picking this up in a new chat

Attach the folder or repo and say:

> This is my Musical Reference Chart. Read `README.md` and `CLAUDE.md` first. The app
> is the single file `chart.html`. I want to: `<describe change>`
