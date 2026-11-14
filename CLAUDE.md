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
2. **Markup is duplicated across views.** The dotted/triplet toggle row exists
   twice, and both tables have two tbodies (`#tb`/`#tbb`, `#pb`/`#pbb`). Change
   one, change the other.
3. **Every displayed number goes through `f2`/`f3`/`f4`.** Never print a raw float.
4. The Both view is the default tab, not Pitch.

## Style

Dark, dense, monospace instrument-panel look. Tight rows, tabular numerals,
colour-coded column accents. It should read as studio gear, not as a website.
Column *header text* is near-white; colour lives in the `border-bottom` accents.

## Workflow

This folder is inside Dropbox and is a Git repo. `.git` is excluded from Dropbox
sync — see README §10. Commit meaningful changes; don't commit on every edit.
