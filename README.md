# Musical Reference Chart

A single-file, offline-capable web app for music production reference: pitch/frequency
tables and tempo-division delay times, filtered by song key, with a printable per-song
summary sheet.

Inspired by the iOS app **Tempo** (by Frozen Ape).

---

## 1. Quick facts

| | |
|---|---|
| **Type** | Static web app — one HTML file, no build step |
| **Entry point** | `chart.html` |
| **Dependencies** | None. No frameworks, no CDN, no fonts fetched. Pure HTML/CSS/JS |
| **Runs offline** | Yes — open the file directly in any browser (`file://` works) |
| **Browser target** | Modern desktop Chrome/Edge/Firefox. Not optimised for mobile |
| **Lines** | ~607 |
| **Author** | Magnus (Type 2 Studio) |

**Design intent:** a dark, dense, monospace "instrument panel" aesthetic — information
tool, not a website. Tight rows, tabular numbers, colour-coded columns. It should feel
like a piece of studio gear, not a web page.

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
| Tempo | `#bpm` | `128` | BPM. Drives all tempo divisions |
| Sample Rate | `#sr` | `96000` | Hz. Blank → Samples column shows `N/A` |
| Tuning ref | `#a4` | `440` | A4 in Hz. Drives all pitch calculations |
| Key root | `#keyroot` | `""` (All) | `""` or semitone `0`–`11` |
| Key mode | `#keymode` | `maj` | See scale list below |
| PRINT SUMMARY | `#printBtn` | — | Opens summary in new window (§5) |
| View tabs | `.tbtn` | `both` | `pitch` / `tempo` / `both` |

Header bar spans full width; inner content is capped at `1400px` and centred
(`.hd-inner` + `#body` both use that max-width).

### Pitch table

All 128 MIDI notes (C-1 to G9), filtered to the selected key if one is set.

| Column | Value |
|---|---|
| Note | Name + space + octave number, e.g. `C# 4`. Both full-size, white (sharps are blue) |
| MIDI | 0–127 |
| Frequency | Hz, 3 decimals |
| Cycle length | ms, 3 decimals — one full waveform period |
| Wavelength | metres, 3 decimals — at 343.2 m/s (air, 20 °C) |
| Samples | Samples per cycle at the given sample rate. `N/A` if SR blank |
| LFO / audio rate | Same Hz value, badged `LFO` if < 20 Hz, else `OSC` |

Row highlights: A4 (teal) and C4 (blue) get badges and a brighter background.

### Tempo table

21 divisions: `1/1` … `1/64`, each in straight, dotted (`D`) and triplet (`T`) form.
Two toggle buttons above the table show/hide dotted and triplet rows.

| Column | Value |
|---|---|
| Division | e.g. `1/8D`. Colour-coded: white = straight, teal = dotted, orange = triplet |
| Delay (ms) | 2 decimals |
| Delay (s) | 4 decimals *(full Tempo tab only)* |
| LFO rate | Hz, 4 decimals — the rate that completes one cycle per division |

### Views

- **Pitch** — pitch table only, full width
- **Tempo** — tempo table only, with the seconds column
- **Both** *(default)* — tempo in a narrow fixed left column, pitch filling the rest.
  On load, auto-scrolls the pitch column to A4.

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

### Scales (`SCALES`, semitone offsets from root)

```
maj  [0,2,4,5,7,9,11]   Major
min  [0,2,3,5,7,8,10]   Natural Minor
dor  [0,2,3,5,7,9,10]   Dorian
phr  [0,1,3,5,7,8,10]   Phrygian
lyd  [0,2,4,6,7,9,11]   Lydian
mix  [0,2,4,5,7,9,10]   Mixolydian
pmaj [0,2,4,7,9]        Pentatonic Major
pmin [0,3,5,7,10]       Pentatonic Minor
```

### Chord tones (`CHORD_TONES`, `[root, 3rd, 5th, 7th]`)

Used only by the print summary. `null` = that degree doesn't exist in the scale
(Pentatonic Major has no 7th).

```
maj  [0,4,7,11]    min  [0,3,7,10]    dor  [0,3,7,10]    phr  [0,3,7,10]
lyd  [0,4,7,11]    mix  [0,4,7,10]    pmaj [0,4,7,null]  pmin [0,3,7,10]
```

---

## 5. Print summary

Clicking **PRINT SUMMARY** builds an HTML string and opens it via
`window.open()` + `document.write()` — a standalone window with no dependency on
the parent page. Designed to be screenshotted and saved alongside the project file.

Layout:

- **Title** — song name, with ♪ glyph
- **Meta line** — key, BPM, sample rate, A4 tuning, plus notes if dotted/triplets
  are hidden
- **Left column** — tempo divisions (ms / seconds / LFO Hz), respecting the
  dotted/triplet toggles
- **Right column** — depends on whether a key is selected:
  - **Key selected** → chord tones. Four labelled sections (Root / Third / Fifth /
    Seventh), each listing that tone across **octaves 1–5**, with full columns
    (Note, MIDI, Freq, Cycle, λ, Samples)
  - **No key** → flat list of all notes C0–C8
- **Footer** — generation timestamp

Styles are inlined into the generated document because it has no access to the
parent stylesheet.

> **Popup blockers** will silently stop this. If the button appears to do nothing,
> that's the cause.

---

## 6. Colour system

Defined as CSS custom properties on `:root`.

```
--bg  #06090f   page          --t0  #d6e4ff   primary text
--bg1 #0c1121   header        --t1  #5f78b0   secondary
--bg2 #111829   table head    --t2  #2b3a60   muted
--bg3 #18203a   inputs        --t3  #14203a   near-invisible
--ln  #1c2640   border        --ln2 #273555   border bright
```

Column accent colours — each has a `d` (dark) variant used for header underlines:

```
--c0 #00d4b0  teal      frequency / delay ms
--c1 #9080ff  lavender  cycle length
--c2 #ffaa33  amber     wavelength
--c3 #33ee88  green     samples
--c4 #ff5599  pink      LFO
--c5 #5599ff  blue      sharp note names
--c6 #ff8833  orange    triplet divisions
```

### Octave banding

Pitch rows cycle through a 4-colour tint so octaves are visually separable:

```css
tr.oct0 td { background: rgba(0,212,176,.15) }   /* teal     */
tr.oct1 td { background: rgba(144,128,255,.15) } /* lavender */
tr.oct2 td { background: rgba(255,170,51,.12) }  /* amber    */
tr.oct3 td { background: rgba(255,85,153,.11) }  /* pink     */
```

Class assigned in `pitchHTML()` as `'oct' + ((oct + 1 + 40) % 4)`.
The `+ 40` keeps the modulo positive for negative octaves.

> ⚠️ **Gotcha — read before touching this.**
> The banding **must** be applied to `td`, not `tr`. With
> `border-collapse: collapse`, a background on `<tr>` does not paint through the
> cells in most browsers — it renders as nothing. This was a real bug; don't
> "simplify" it back to `tr { background }`.

Column header labels (`.thm`) are deliberately **all near-white** (`#dce8ff`).
Earlier versions coloured each header to match its column; that was changed —
the colour differentiation now lives only in the `border-bottom` accents.
Subtitles (`.ths`) are `rgba(220,232,255,.45)`.

---

## 7. Code map

Single file, three blocks: `<style>`, markup, `<script>`.

### Key functions

| Function | Purpose |
|---|---|
| `bpm()` `sr()` `a4()` `songName()` | Read + sanitise header inputs. `sr()` returns `null` when blank |
| `getKeySet()` | Returns a `Set` of pitch classes 0–11, or `null` for "All" |
| `keyLabel()` | Human string, e.g. `"D# / Eb Dorian"`. Empty string if no key |
| `pitchHTML(a4, sr, keySet)` | Builds all pitch rows |
| `tempoHTML(bpm, compact)` | Builds tempo rows. `compact` drops the seconds column (Both view) |
| `updateSRLabel(sr)` | Updates the "@ 96,000 Hz" subtitle in the Samples header |
| `refresh()` | Master redraw — reads inputs, repaints only the active tab |
| `syncTogs()` | Keeps the duplicated dotted/triplet buttons in sync across views |
| `printSummary()` | Builds and opens the summary window |

### State

Module-scope globals: `showDot`, `showTri` (booleans, both `true`).
Everything else is read live from the DOM inputs on each `refresh()`.

### Event wiring

- `.tbtn` click → swap `.on` class on button + pane, then `refresh()`
- `.tog` → **delegated** listener on `document` (the toggle buttons are duplicated
  in two views, so delegation keeps both working)
- `#bpm #sr #a4` → `input` → `refresh()`
- `#keyroot #keymode` → `change` → `refresh()`
- `#song` → `input` → updates `document.title` only (doesn't trigger redraw)

### Duplicated markup — important

The dotted/triplet toggle row exists **twice** (once in the Tempo pane, once in the
Both pane's left column), and the tempo and pitch tables each have **two tbodies**
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
- Font stack is a monospace cascade of system fonts; nothing is downloaded.

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
- More scales — harmonic minor, melodic minor, locrian, blues.
- Selectable octave range for the print summary (currently hardcoded 1–5 for chord
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
2. **Mirror duplicated markup** — two toggle rows, two tbody pairs (§7).
3. **Octave banding goes on `td`, never `tr`** (§6).
4. Keep the dense dark instrument-panel aesthetic.
