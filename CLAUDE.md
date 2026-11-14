# Musical Reference Chart — working instructions

**Read `README.md` before making changes.** It has the maths, the colour system,
the code map and the known gotchas.

## Hard constraints

- **One file.** The whole app is `chart.html`. No build step, no bundler, no
  framework. Don't split it up.
- **No network.** It must work opened straight from disk with no connection.
  Never add a CDN script, a Google Font, or any external request.
- **Keep the filename `chart.html`.** GitHub Pages serves it at `/chart.html`
  (`https://miuglitch.github.io/musicalrefchart/chart.html`), and links point there.

## Easy things to get wrong

1. **Octave banding must target `td`, not `tr`.** With `border-collapse: collapse`
   a `<tr>` background doesn't paint through cells. It will silently render as
   nothing.
2. **Table markup is duplicated across views.** Both tables have two tbodies
   (`#tb`/`#tbb`, `#pb`/`#pbb`). Change one, change the other. The side strip (Root,
   Dotted, Triplets, PRINT, Pitch, Tempo, Dark) exists once and serves every view.
3. **Every displayed number goes through `f2`/`f3`/`f4`.** Never print a raw float.
4. Pitch and Tempo are independent toggles; both on (the side-by-side view) is the default.
5. **Layout is 80% of the window width** (10% gutter each side) for both the header
   and `#body`. Don't reintroduce a max-width. In the Both and Pitch views the tables
   are `width:auto` (compact on purpose).
6. **The summary is a canvas-drawn .jpg download**, not a popup. Its colours are a
   copy of the light-theme `:root` tokens inside `printSummary()`; change both together.
7. **Scales live in `SCALE_LIST` only.** The Key mode menu is built from it, so
   don't add `<option>`s by hand. Chord tones are derived by `chordTones()`.
8. **Two themes.** Body colours come from tokens on `:root`, overridden under
   `html[data-theme="dark"]`. Add new colours as tokens in both. The header looks
   the same in both themes.
9. **The font is embedded** (`<style id="fonts">`, base64 Inter Display as `IDM`).
   Don't swap it for a Google Fonts link.

## Style

Teenage Engineering K.O. II look: black display header with big white readouts,
light warm-grey body (or black in dark mode), dark grey frames, orange / red /
blue / dark cyan accents. Inter Display at several weights, tabular numerals, clean
contrast, **no glow**. Column header text is near-black (white in dark mode); colour
lives in the `border-bottom` accents. It should read as studio gear, not a website.

## Workflow

This folder is inside Dropbox and is a Git repo. `.git` is excluded from Dropbox
sync — see README §10. Commit meaningful changes; don't commit on every edit.
