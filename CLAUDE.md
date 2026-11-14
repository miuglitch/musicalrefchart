# Musical Reference Chart — working instructions

**Read `README.md` before making changes.** It has the features, the maths, the
colours, the code map and the gotchas.

## Working with Magnus

- **No terminal.** Magnus doesn't use PowerShell or a command line, and Remote Control
  doesn't work for him. Never give him commands to run. He uses GitHub in the browser
  (and GitHub Desktop).
- **Keep it simple and direct.** No convoluted workarounds; pick the plain solution.
- **How changes land:** work on a `claude/…` branch, open a pull request, and let
  Magnus merge it on GitHub. Don't push to `main`.
- **Also update his local copy** in `D:\Dropbox\Claude\Musical Reference Chart`
  (`chart.html`, `README.md`, `CLAUDE.md`), then **read the file back from his PC and
  compare it** before saying it's done. A copy can silently land as an older version.
- Show results with screenshots (light and dark mode when the look changes).

## Hard constraints

- **One file.** The whole app is `chart.html`. No build step, no bundler, no
  framework. Don't split it up.
- **No network.** It must work opened straight from disk. Never add a CDN script, a
  Google Font or any external request.
- **Keep the filename `chart.html`.** GitHub Pages serves it at
  https://miuglitch.github.io/musicalrefchart/chart.html (deployed by an Action on
  every push to `main`), and links point there.

## Easy things to get wrong

1. **Octave banding goes on `td`, not `tr`.** With `border-collapse: collapse` a
   `<tr>` background silently renders as nothing.
2. **Each table has two tbodies** (`#tb`/`#tbb`, `#pb`/`#pbb`), one per pane. Change
   one, change the other. The side strip exists once and serves every view.
3. **Every displayed number goes through `f2`/`f3`/`f4`.** Never print a raw float.
4. **PITCH and TEMPO are independent toggles;** both on (side by side) is the default.
   There is no Both button.
5. **Layout is 80% of the window width** (10% gutters) for the header and `#body`;
   strip and tables are centred as a group; tables are `width:auto`. No max-width.
   Header rows are a fixed 60 px so both tables line up; the strip's `margin-top`
   depends on that.
6. **Two themes.** Body colours are tokens on `:root`, overridden under
   `html[data-theme="dark"]`. Add new colours to both. The header is the same in both.
7. **PRINT draws a .jpg on a canvas** in the current theme. Its colours are copies
   inside `printSummary()`; keep them in step with the tokens.
8. **Scales live only in `SCALE_LIST`.** The Key menu is built from it; don't add
   `<option>`s by hand. Chord tones come from `chordTones()`.
9. **The font is embedded** (`<style id="fonts">`, Inter Display as `IDM`). Don't
   swap it for a link.
10. **Phones get a separate layout** at 700 px wide or less: one `@media` block plus
   `placeControls()` (README §3 Mobile layout). Desktop must stay unchanged. On phones
   the strip buttons are *moved* into the bottom bar `.mbar`, so never assume they're
   inside `.strip`. Check UI changes at 390 px wide as well as on desktop.

## Style

Teenage Engineering K.O. II look: black display header with big white readouts,
light warm-grey body (black in dark mode), dark grey frames, orange / red / blue /
dark cyan accents. Inter Display at several weights, tabular numerals, clean contrast,
**no glow**. Side-strip buttons are capitals with centred text. Column header text is
near-black (white in dark mode); colour lives in the `border-bottom` accents. It
should read as studio gear, not a website.
