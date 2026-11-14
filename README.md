# Musical Reference Chart

A single-file, offline-capable web app for music production reference: pitch/frequency
tables and tempo-division delay times, filtered by song key, with a per-song summary
sheet saved as a .jpg image.

Inspired by the iOS app **Tempo** (by Frozen Ape).

---

## 1. Quick facts

| | |
|---|---|
| **Type** | Static web app — one HTML file, no build step |
| **Entry point** | `chart.html` |
| **Dependencies** | None. No frameworks, no CDN, nothing fetched. The Inter Display font is embedded in the file (§6) |
| **Runs offline** | Yes — open the file directly in any browser (`file://` works) |
| **Browser target** | Modern desktop Chrome/Edge/Firefox. Not optimised for mobile |
| **Lines** | ~690 (plus ~175 KB of embedded font data) |
| **Author** | Magnus (Type 2 Studio) |

**Design intent:** Teenage Engineering-style hardware look, after the K.O. II: a
black display strip header with big white readouts over a light warm-grey body,
dark grey frames, signal orange / red / blue / dark cyan accents. Large, readable
Inter Display at several weights. Clean contrast, no glow. A **dark mode** keeps the
same header and turns the body into a black display (§6). It should feel like a
piece of studio gear, not a web page. (Chosen Nov 2026 from five mockups; this was
"design 1".)

---

## 2. How to run it

Open `chart.html` in a browser. That's it. No server, no install.

For the hosted version, see §9.

---

## 3. What the app does

### Header (fixed, always visible)

| Field | ID | Default | Notes |
|---|---|---|---|
| Song name | `#song` | *(empty → "Untitled")* | Text. Also updates `document.title` live |
| Tempo | `#bpm` | `128` | BPM. Drives all tempo divisions. DAW-style: click-hold-drag up/down changes it live (1 BPM per 4 px, Shift = 0.1 steps, clamped 20–999); a plain click lets you type |
| Sample Rate | `#srsel` + `#sr` | `96000` | Menu of common rates 11025 – 384000 Hz. The last option, **Custom…**, reveals the `#sr` number field for any value. Blank/0 custom → Samples column shows `N/A` |
| Tuning ref | `#a4` | `440` | A4 in Hz. Drives all pitch calculations |
| Key root | `#keyroot` | `""` (All) | `""` or semitone `0`–`11` |
| Key mode | `#keymode` | `maj` | 35 scales/modes, built from `SCALE_LIST` (§4) |

### Side strip (left of the tables)

One vertical strip, shared by all views, top to bottom:

| Button | ID / class | Default | Notes |
|---|---|---|---|
| Root / Dotted / Triplets | `.tog.root` `.tog.dot` `.tog.tri` | on | Show/hide straight, dotted, triplet rows (§3 Tempo table) |
| *(gap)* | `.strip-gap` | | |
| PRINT | `#printBtn` | — | Downloads the summary as a .jpg (§5) |
| Pitch / Tempo | `.vbtn[data-view]` | both on | Independent show/hide for the two tables. Both on = side-by-side view. The last one on can't be turned off |
| Dark / Light | `#themeBtn` | light | Switches the body theme; saved in `localStorage` key `mrc-theme` |

### Layout

The header is one centred, wrapping row (`.hd-row`): logo and inputs. A
`ResizeObserver` keeps `#body` sitting right under it. Its inner content (`.hd-inner`) and the table area
(`#body`) both fill **80% of the window width, with a 10% gutter each side**. There
is no max-width cap. Font sizes and paddings were scaled up ~25% so the page reads
comfortably at 100% browser zoom. The side strip and the visible tables are
**centred as a group** (`#body{justify-content:center}`); tables are `width:auto` and
hug their content, and the pitch table's numbers are right-aligned.

Top left is the Miu logo (inline SVG, `fill="currentColor"`, light grey) with
**REFERENCE CHART** under it in orange.

### Pitch table

All 128 MIDI notes (C-1 to G9), filtered to the selected key if one is set.

| Column | Value |
|---|---|
| Note | Name + space + octave number, e.g. `C# 4`. Both full-size, white (sharps are blue) |
| Frequency | Hz, 3 decimals |
| Cycle length | ms, 3 decimals — one full waveform period |
| Wavelength | metres, 3 decimals — at 343.2 m/s (air, 20 °C) |
| Samples | Samples per cycle at the given sample rate. `N/A` if SR blank |

The MIDI-number and LFO/OSC columns were removed (Nov 2026).

Row highlights: A4 (orange) and C4 (blue) get badges and a tinted background.

### Tempo table

21 divisions: `1/1` … `1/64`, each in straight, dotted (`D`) and triplet (`T`) form.
Three toggle buttons, stacked top-down in a strip to the left of the table — **Root**, **Dotted**, **Triplets** — show/hide
the straight, dotted and triplet rows independently. All three start on.

| Column | Value |
|---|---|
| Division | e.g. `1/8D`. Colour-coded: black (white in dark mode) = straight, cyan = dotted, orange = triplet |
| Length (ms) | 2 decimals. Headed `Length ms` in the Both view |
| Length (s) | 4 decimals *(full Tempo tab only)* |
| LFO rate | Hz, 4 decimals — the rate that completes one cycle per division |

### Views

Set by the Pitch / Tempo toggles in the side strip (`currentTab()`):

- **Pitch + Tempo on** *(default)* — `#pane-both`: compact tempo table and pitch table
  side by side. On load, auto-scrolls the pitch column to A4.
- **Pitch only** — `#pane-pitch`
- **Tempo only** — `#pane-tempo`, with the extra seconds column

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

`DIVS[]` multipliers are relative to a **quarter note** (`1/4` = `1`). So `1/1` = `4`,
`1/8` = `0.5`, dotted = ×1.5, triplet = ×2/3.

Each entry is `{n: name, m: multiplier, t: type}` where `t` is `0` straight,
`1` dotted, `2` triplet. The `t` value drives both the filter and the row colour.

### Scales (`SCALE_LIST`, semitone offsets from root)

35 scales and modes, in the same order and with the same interval sets as Ableton
Live's scale menu. `SCALE_LIST` is the single source: the Key mode `<select>` is
built from it at load, and `SCALES` / `SCALE_NAMES` lookups are derived from it.
To add a scale, add one row there.

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

### Chord tones (`chordTones(mode)`, `[root, 3rd, 5th, 7th]`)

Used only by the summary image. Computed from the scale, not hand-listed:

- **7-note scales** — stack thirds: scale degrees 1, 3, 5, 7.
- **Everything else** — pick from the notes the scale has: 3rd = major 3rd if present,
  else minor 3rd; 5th = perfect 5th, else #5 (when the 3rd is major), else b5, else #5;
  7th = dim 7th when the 5th is b5, else minor 7th, else major 7th.
- `null` = no such tone in the scale; that section is left out of the summary
  (e.g. In-Sen and Iwato have no 3rd, Major Pentatonic has no 7th).

---

## 5. Print summary (.jpg)

Clicking **PRINT** draws the summary onto an off-screen `<canvas>` (at 2×
for sharpness) and downloads it as a JPEG named `<song name> - summary.jpg`. No
popup window, no library: it's plain Canvas 2D + `canvas.toBlob('image/jpeg')` and a
temporary `<a download>` link, so it works offline and from `file://`.

Layout:

- **Title** — song name
- **Meta line** — key, BPM, sample rate, A4 tuning, plus notes if root/dotted/
  triplet rows are hidden
- **Left column** — tempo divisions (Length ms / seconds / LFO Hz), respecting the
  Root/Dotted/Triplets toggles
- **Right column** — depends on whether a key is selected:
  - **Key selected** → chord tones. Four labelled sections (Root / Third / Fifth /
    Seventh), each listing that tone across **octaves 1–5** (Note, Freq, Cycle,
    Wavelength, Samples)
  - **No key** → flat list of all notes C0–C8
- **Footer** — generation timestamp

Canvas can't read the page stylesheet, so colours live in a small `C` palette object
inside `printSummary()` that mirrors the `:root` tokens: a light set and a dark set, picked from the page's
current theme, so dark mode gives a dark image and light mode a light one. It draws in the embedded Inter Display; the
click handler waits on `document.fonts.load()` first so the canvas doesn't fall back. Canvas height is computed
from the row count, so the no-key image is tall (~2600 px logical).

---

## 6. Colour system

Palette taken from a Teenage Engineering K.O. II. CSS custom properties on `:root`:

```
--orange #ff5a1f   signal orange: accent, BPM readout, A4, triplets, wavelength, delay ms
--red    #e0322b   samples
--blue   #3a86d4   frequency, LFO Hz, C4, sharps
--cyan   #0f8a94   dark cyan: cycle length, dotted
--frame  #3b3b3b   dark grey frames (header underline, Both-view divider)
--disp   #0f0f0f   black display (header)
```

The **header** is fixed black-display styling in both themes. Everything below it
reads semantic tokens (`--bg`, `--th-bg`, `--band-a/-b`, `--v-freq`, `--k-freq`, …).
`html[data-theme="dark"]` redefines those tokens for dark mode. Text colours (`--v-*`)
are darker shades of the accents on light grey and brighter shades on black, so they
stay readable. A tiny script in `<head>` applies the saved theme before first paint.

Column header text is near-black (light) / white (dark); each column's colour lives
in its `border-bottom` accent (`--k-*`).

**Font:** Inter Display (SIL OFL), subset to Latin and embedded as base64 woff2 in
`<style id="fonts">` at the top of the file: weights 300, 400, 500, 600, 700, 800,
about 175 KB. It's registered under the private family name `IDM` so a locally
installed Inter can't interfere. Numbers use `font-feature-settings:'tnum'` so
columns line up. To re-make the subset: `pyftsubset InterDisplay-<W>.otf
--unicodes="U+0020-007E,U+00A0-00FF,U+2013,U+2014,U+2018,U+2019,U+201C,U+201D,U+2022,U+2026,U+2190-2193,U+2212,U+00B7,U+266D,U+266F"
--layout-features='tnum,kern,liga,calt,case,zero,ss01,cv11' --flavor=woff2`, then base64 it.

### Octave banding

Pitch rows alternate between two greys by octave (`--band-a` / `--band-b`) and each C
row gets a heavier top rule:

```css
tr.oct0 td,tr.oct2 td { background: var(--band-a) }
tr.oct1 td,tr.oct3 td { background: var(--band-b) }
```

Class assigned in `pitchHTML()` as `'oct' + ((oct + 1 + 40) % 4)`.
The `+ 40` keeps the modulo positive for negative octaves.

> ⚠️ **Gotcha — read before touching this.**
> The banding **must** be applied to `td`, not `tr`. With
> `border-collapse: collapse`, a background on `<tr>` does not paint through the
> cells in most browsers — it renders as nothing. This was a real bug; don't
> "simplify" it back to `tr { background }`.

---

## 7. Code map

Single file, three blocks: `<style>`, markup, `<script>`.

### Key functions

| Function | Purpose |
|---|---|
| `bpm()` `sr()` `a4()` `songName()` | Read + sanitise header inputs. `sr()` reads the `#srsel` menu, or `#sr` when it's on Custom…; `null` when blank |
| `getKeySet()` | Returns a `Set` of pitch classes 0–11, or `null` for "All" |
| `chordTones(mode)` | `[root, 3rd, 5th, 7th]` semitones for the summary (§4) |
| `keyLabel()` | Human string, e.g. `"D# / Eb Dorian"`. Empty string if no key |
| `pitchHTML(a4, sr, keySet)` | Builds all pitch rows |
| `tempoHTML(bpm, compact)` | Builds tempo rows. `compact` drops the seconds column (Both view) |
| `updateSRLabel(sr)` | Updates the "@ 96,000 Hz" subtitle in the Samples header |
| `refresh()` | Master redraw — reads inputs, repaints only the active tab |
| `syncTogs()` | Updates the Root/Dotted/Triplets buttons' `.on` state |
| `currentTab()` / `applyView()` | Which pane to show from the Pitch/Tempo toggles; shows it and redraws |
| `divOn(d)` | `true` if a DIVS entry's type is currently shown (shared by tables and summary) |
| `printSummary()` | Draws the summary on a canvas and downloads it as .jpg |

### State

Module-scope globals: `showRoot`, `showDot`, `showTri`, `showPitch`, `showTempo` (booleans, all `true`).
Everything else is read live from the DOM inputs on each `refresh()`.

### Event wiring

- `.vbtn` click → flips `showPitch`/`showTempo` (never both off), then `applyView()`
- `.tog` → delegated listener on `document` → flips `showRoot`/`showDot`/`showTri`
- `#bpm #sr #a4` → `input` → `refresh()`
- `#srsel` → `change` → shows/hides the custom `#sr` field, then `refresh()`
- `#themeBtn` → toggles `data-theme="dark"` on `<html>` and saves it to `localStorage`
- `#keyroot #keymode` → `change` → `refresh()`
- `#song` → `input` → updates `document.title` only (doesn't trigger redraw)

### Duplicated markup — important

The tempo and pitch tables each have **two tbodies**
(`#tb`/`#tbb`, `#pb`/`#pbb`). Any change to one must be mirrored. This is the main
source of "I fixed it but it's still broken in the other tab" confusion.

---

## 8. Conventions

- **No build step.** Keep it one file. If it ever needs a bundler, that's a
  different project.
- **No external requests.** It must work with no network. Don't add a CDN font
  or library.
- Styles are minified-ish and grouped by section with `/* ── NAME ── */` banners.
- JS uses short helpers `f2` `f3` `f4` for `toFixed`. Every displayed number goes
  through one of them — never print a raw float.
- Font is the embedded Inter Display (§6); nothing is downloaded. Fallbacks: Inter, system-ui, Segoe UI.

---

## 9. Repo / hosting

- **Local working copy:** inside Dropbox (see §10) — Dropbox gives instant file-level
  backup and version history.
- **Git remote:** GitHub, https://github.com/miuglitch/musicalrefchart (public).
- **Hosting:** GitHub Pages, from the repo root. The app is served at
  `/chart.html`, i.e. `https://miuglitch.github.io/musicalrefchart/chart.html`.

The file is named `chart.html`, so the bare Pages URL (no path) does not open the
app. Always link to `/chart.html`. If the filename changes again, update the
hosted link and this section.

---

## 10. Dropbox + Git together

This repo lives inside a Dropbox folder. That is fine, **but `.git` must be
excluded from Dropbox sync**, or Dropbox will race with Git on the object files and
can corrupt the repo.

On Windows, mark it ignored once per clone:

```powershell
Set-Content -Path 'C:\Users\<you>\Dropbox\...\Musical Reference Chart\.git' `
  -Stream com.dropbox.ignored -Value 1
```

Verify:

```powershell
Get-Content -Path '...\.git' -Stream com.dropbox.ignored
# should print: 1
```

If that was never done and the repo starts throwing object errors, the fix is to
delete `.git`, re-clone from GitHub outside Dropbox, then move it in and set the
ignore flag before the first sync.

---

## 11. Ideas not yet built

- Enharmonic display — show flats (Bb, Eb) rather than sharps when the key calls
  for it. Currently sharp-only throughout.
- Selectable octave range for the summary image (currently hardcoded 1–5 for chord
  tones, C0–C8 for the no-key list).
- Save/load song presets to `localStorage`.
- Reverb/pre-delay helper derived from the tempo grid.
- Haas/widening offsets (sub-20 ms) as a derived column.

---

## 12. Picking this up in a new chat

Paste this and attach the folder:

> This is my Musical Reference Chart project. Read `README.md` first — it has the
> full context, the maths, the colour system and the gotchas. The app is the single
> file `chart.html`. I want to: `<describe change>`

Things worth saying up front so the assistant doesn't get them wrong:

1. **One file, no build, no dependencies, must work offline.** Don't suggest React,
   a bundler, or a CDN.
2. **Mirror duplicated markup** — two tbody pairs (§7). The side strip exists once.
3. **Octave banding goes on `td`, never `tr`** (§6).
4. Keep the K.O. II hardware look: black display header, grey body, orange/red/blue/cyan accents, Inter Display, no glow (§6).
