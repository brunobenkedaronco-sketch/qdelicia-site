# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

Institutional one-page website for **Q'Delícia Sorvetes** — an ice cream / popsicle
manufacturer in Passo Fundo, RS, Brazil (founded 1994). All content is in
**Brazilian Portuguese (pt-BR)** and must stay that way.

There is **no build system, no package manager, no dependencies, no tests**. The repo
is a flat directory of hand-written static files served as-is. To preview a change,
open the HTML file in a browser (or run `python3 -m http.server` from the repo root —
needed for the `<video>` files and relative image paths to resolve correctly).

## Repository layout

```
index.html            Live/primary layout — full-screen video hero slider
v2.html               Alternative layout: "vitrine de picolés" (popsicle showcase), light theme
v3.html               Alternative layout: cinematic dark theme
README.md             Two-line description
Logo branca.png       White wordmark used in nav/footer of v2/v3 and index footer
img_morango.png       Local product photo (Frutas category / Morango flavor)
img_napolitano_zero.png  Local product photo (Zero Açúcar category)
video_sorvete.mp4     Hero background video 1 (~10s, web-optimized)
video_maxi.mp4        Hero background video 2 (~10s)
video_sundae.mp4      Hero background video 3 (~10s)
```

`index.html` is ~443 KB because the "Nossa história" photo is embedded as a
base64 `data:image/jpeg` URI on a single ~385 KB line. Do not try to read that line;
use `awk`/`sed` with a length filter when scanning the file.

### The three layouts

`v2.html` and `v3.html` are **complete alternative designs of the same site**, not
partial experiments — same copy, same product data, same contact details, different
visual language. They are candidates the owner is comparing. Unless told otherwise:

- Treat `index.html` as the live site.
- A content change (a new flavor, a corrected phone number, a new category) must be
  applied to **all three files**, since each carries its own copy of the data.
- A styling/layout change applies only to the file named.

| | index.html | v2.html | v3.html |
|---|---|---|---|
| Theme | cream `#fdf8f0` / brand blue `#005ca9` | ice-blue `#EDF5FB`, neo-brutalist borders | dark `#060D1B`, glassmorphism |
| Fonts | Terfens W01 Bold Italic (onlinewebfonts `@import`) + DM Sans | Bricolage Grotesque + Instrument Sans | Sora + Inter |
| Hero | 3 fullscreen video slides, arrows + dots + progress bar | static header + "freezer" grid of popsicle cards | 3 video slides, segmented progress bar |
| Categories | hand-written `.cat-card` divs with `onclick` | rendered from the `CATS` array | rendered from the `CATS` array |
| Filters | none | chips: Todos / Picolés / Sorvetes / Açaí & Zero / Pizzas & Salgados | same as v2 |
| Flavor list | inline `.flavor-panel` that expands in place | modal `.overlay` (Esc + backdrop close, focus moved to close button) | same as v2 |
| Custom cursor | yes (`#cur`) | no | no |

## Data model

Every file embeds the same catalog inline in a `<script>` block near the end of `<body>`.

`FLAVORS` — object keyed by category slug, each value an array of
`{name, desc, img}`. 15 categories, **82 flavor entries** total (this count is
displayed in v3's stats strip, so update it there if flavors are added/removed).

Category slugs (identical across all three files):
`maxi`, `frutas`, `picespeciais`, `aoleite`, `kids`, `trad2l`, `especiais`,
`potes1l`, `sundae`, `cones`, `queromais`, `zero`, `acai`, `pizzas`, `salgados`

`index.html` additionally has `CAT_LABELS` (slug → emoji + display name, used as the
panel heading). `v2.html`/`v3.html` instead have `CATS`, an ordered array of
`{k, n, s, t, img, local?}` (key, name, subtitle, filter tag, image, local-file flag)
that drives both the category grid and the filter chips.

### Images: two sources

Most product photos are hosted on Google Drive and referenced by file ID:

```
https://lh3.googleusercontent.com/d/<FILE_ID>        (index.html)
https://lh3.googleusercontent.com/d/<FILE_ID>=w400   (v2/v3, with size suffix)
```

A few are local PNGs, flagged differently in each place:

- In `FLAVORS`, a local image is written as `img:'local:img_morango.png'`.
- In `CATS` (v2/v3), it is `img:'img_morango.png', local:1`.

**Known bug — local flavor images are broken in v2 and v3.** Their `openFlavors()`
concatenates `'https://lh3.googleusercontent.com/d/' + f.img + '=w160'`
unconditionally, so a `local:` value produces a garbage URL. Only `index.html`
strips the prefix (`f.img.startsWith('local:') ? ... : ...`). The `gimg()` helper in
v2/v3 handles the `local:1` flag correctly, but is used for `CATS` only. If you touch
the modal renderer in v2/v3, apply the same `local:` check `index.html` uses.

**Known gap — four referenced local images are not in the repo:**
`img_3chocolates.png`, `img_sundae_zero.png`, `img_picole_coco_zero.png`,
`img_acai_zero.png`. Those flavor cards render broken images in all three layouts.
Don't "fix" this by rewriting the paths to Drive IDs — the files need to be added.

## Business facts (keep consistent everywhere)

- WhatsApp: `https://wa.me/5554991493204` — the only contact channel used, appearing
  in nav, contact cards, and footer of every file.
- Address: R. São Vicente de Paula, 66 · Lucas Araújo · Passo Fundo – RS
  (also embedded in a Google Maps iframe in the contact section).
- Hours: Seg–Sex · 08h às 12h e 13h às 18h
- Founded 1994; "30+ anos" / "mais de 30 anos" phrasing is used throughout.

## Conventions

- **Single-file pages.** CSS lives in one `<style>` in `<head>`, JS in `<script>`
  tags before `</body>`. Do not split into external `.css`/`.js` files or introduce
  a bundler, framework, or npm — the owner edits and previews these files directly.
- **CSS is written dense**, mostly one selector per line with no spaces after colons.
  Match that style rather than reformatting; a reformat produces an unreviewable diff.
- Design tokens are CSS custom properties on `:root` (see the table above). Use the
  existing variables instead of hard-coding colors.
- Section IDs `#topo`/`#sobre`/`#produtos`/`#contato` back the nav anchors — don't
  rename them.
- Scroll reveals use an `IntersectionObserver` plus a marker class: `.r` → `.on` in
  `index.html`, `.r`/`.pop`/`.prato` → `.in` in v2/v3. New sections need the marker
  class or they stay invisible.
- Hero video slides advance off each video's real `duration` with a 1500 ms crossfade
  (`FADE`), falling back to a `loadedmetadata` listener when duration isn't known yet.
  Videos are `muted playsinline` — required for mobile autoplay, don't remove.
- Accessibility: v2/v3 use `aria-*` on the modal, ticker, and filter tablist and have
  visible `:focus-visible` outlines. Preserve those when editing; `index.html` is
  weaker here and improvements are welcome.
- Commit messages in this repo are short, imperative, and in Portuguese
  (e.g. "Crossfade 1.5s para os videos de ~10s"). Follow that.

## Media

The three MP4s total ~6.6 MB and were deliberately re-encoded for web (~10 s each).
If you replace or add video, keep it short and compressed — there is no CDN or
build step to optimize it later.
