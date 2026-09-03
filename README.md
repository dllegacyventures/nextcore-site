# NextCore Technologies — Company Site

Static company site for **NextCore Technologies Limited Liability Company**
(Delaware, US), deployed via GitHub Pages on nextcore-technologies.com.

## Wording

The business description on this page mirrors, word for word, the description used
in every account application. Do not change one without the other — inconsistent
descriptions across providers are the most common cause of account review.

## Company details are published again

Legal name, registered state, business address, managing member and the EIN were
removed on 2026-09-02 and put back on 2026-09-03, on the home page (`#company`)
and in full on `/imprint/`.

**Know what this costs.** The business address on file is IFZA Properties, Dubai,
while the entity is registered in Delaware. A US registration with a UAE address is
exactly the mismatch that gets flagged in Google Business and Meta verification —
that is why the details came off the page in the first place. Publishing them again
makes the site usable as the "verifiable company presence" banks and PSPs ask for,
and re-exposes the verification problem. Both things are true; the trade-off was
made deliberately.

The **EIN is on `/imprint/` only** — deliberately not in the footer and not in the
JSON-LD, so it does not get syndicated into every search result and scraper feed.
Move it if you want it more visible.

## Pages

- `index.html` — the whole home page: markup, styles and behaviour in one file
- `imprint/index.html` — imprint / Impressum
- `privacy/index.html` — privacy policy / Datenschutzerklärung
- `og.png` — 1200×630 social preview, rendered from a template (see below)
- `robots.txt`, `sitemap.xml`, `favicon.svg`

The two legal pages are **generated**, not hand-edited. Their shared shell lives in
`/tmp/mkdoc.py` at build time — if you need to regenerate them, rebuild that shell
plus `imprint.py` / `privacy.py`, or just edit the produced HTML directly and accept
that the two pages can then drift apart.

### The legal pages are not legal advice

They were written to describe accurately what this site actually does. They have not
been reviewed by a lawyer. Before relying on them, get the imprint checked against
the disclosure rules of whichever jurisdiction you are actually addressing, and have
the privacy policy checked against your real processor list.

Two statements in `privacy/` are claims about the outside world that go stale:
GitHub's certification under the EU–U.S. Data Privacy Framework, and the retention
periods. Re-check both when you touch the file.

## No third-party requests

Opening a page on this site contacts **no** server other than the host:

- **Fonts** (Newsreader, Instrument Sans, IBM Plex Mono) are in `fonts/`, latin and
  latin-ext subsets only, not loaded from Google Fonts.
- **Scripts** (GSAP 3.12.5, ScrollTrigger, Lenis 1.1.18) are in `vendor/`, not
  loaded from cdnjs or jsDelivr.

This is what `privacy/` asserts, so **if you add a CDN link, an embedded video, a
map or an analytics tag, the privacy policy becomes false.** Update it in the same
commit.

To refresh the fonts: fetch the Google Fonts CSS with a modern browser user-agent,
keep the `latin` and `latin-ext` `@font-face` blocks, download each unique woff2
once, and rewrite the `src` URLs to `/fonts/…`. Watch out for variable fonts — several
weights share one file, so name files by URL, not by weight, or you will emit
`@font-face` rules pointing at files that do not exist.

## Languages

English and German. The visitor's language comes from `navigator.languages`; an
unsupported locale falls back to English. A manual choice via the EN/DE switch wins
and is remembered in `localStorage` under `nc-lang` — the same key on all three
pages, so the choice carries across.

- **Home page:** every translatable node carries a `data-i18n` key resolved against
  the `I18N` object at the top of the inline script. `data-i18n-html` marks the four
  values that contain markup (`hero.h1`, `hero.lede`, `faq.4a`, `f.consent`). If you
  add markup to a value, you must add that attribute too, or it will render as
  visible tags.
- **Legal pages:** a different mechanism — parallel `<span data-en>` / `<span data-de>`
  pairs, switched by CSS on `html[lang]`. Both spans are always in the DOM.

The FAQ JSON-LD is generated from the English `faq.*` values, so the structured data
cannot drift from the visible text.

## Enquiry form

`FORM_ENDPOINT` in the inline script is empty. While it is empty, submitting the form
composes a pre-filled `mailto:` draft — the form works today and is never a dead end.
Paste a Formspree/Basin/worker URL there to switch it to a real POST; the code sends
JSON and already handles the failure path.

**When you set an endpoint, update `privacy/` section 5** — it currently states that
no form service is configured and promises to name one before that changes.

Spam handling is a hidden honeypot field (`_fax`). No captcha, no IP evaluation.

## Motion and the backdrop

The atmosphere sits at `z-index:0` behind `.shell` and is deliberately loud. It
is split between CSS and one canvas, and the split is the whole design:

- **CSS owns the colour clouds.** `.para` is a 240vh nebula field holding seven
  radial-gradient auroras — warm at the top, cooling through the middle, warm
  again at the close — that the page travels through as you scroll.
- **`#atmos` (one canvas) owns everything else:** god rays, the conic sweep,
  three inclined orbit rings with travelling nodes, a depth starfield, a
  constellation network that reacts to the pointer, energy pulses running along
  the same 88px lattice the CSS grid draws, and the occasional shooting star.

### Why it is built that way

All of it started as DOM layers. That version cost **92ms per scroll frame**
(≈10fps at 1440px). The fixes, in the order they paid off:

| Change | Frame time @1440 |
|---|---|
| starting point — everything as DOM layers | 92ms |
| dropped `filter:blur(80px)` from seven auroras | 67ms |
| moved rays, sweep and orbits onto the canvas | 24.5ms |
| cached gradients; batched star fills into alpha buckets | 16.8ms* |
| rays and sweep actually drawing again (see the NaN gotcha) | 25.1ms |

\* that row is flattering: a hoisting bug meant the rays and the sweep were
silently not drawn at all. The honest current figures, five runs each:

| Viewport | With backdrop | Backdrop removed |
|---|---|---|
| phone 390 | 8.3ms | 8.3ms |
| desktop 1440 | 25.1ms | 8.3ms |
| 1080p | 41.6ms | 16.7ms |

**Read those as relative, not absolute.** They come from headless Chromium with
software rendering, where a *static* page at 1080p already costs 16.7ms a frame
— the harness is CPU-bound, so anything that moves is charged full price. What
they are good for is comparing two versions of this file, which is exactly how
the table above was produced.

The remaining gap is the seven aurora drift animations. Removing them takes
1080p from 41.6ms to 25.1ms, but the cost does not come from `scale()` (making
the drifts translate-only changes nothing) and it is not fixed by promoting the
clouds individually. It is the software compositor repainting moving layers,
which is the part a real GPU handles. They were kept.

### The rules this leaves behind

- **Never put `filter: blur()` on a full-size backdrop layer.** A radial
  gradient with a soft outer stop looks the same and does not force a blur
  re-composite on every frame.
- **Soft shapes are baked, not blurred per frame.** The rays and the sweep are
  drawn once into offscreen canvases at half resolution, blurred and faded
  there, then blitted each frame with a rotation. One `drawImage` each.
- **Gradients are built on resize, not per frame.** Rebuilding fifteen of them
  every frame was the single most expensive thing in the loop.
- **Stars are batched into six alpha buckets per colour**, so the field is
  twelve paths instead of ~360 fills.
- **`will-change` only on the two parallax wrappers.** Putting it on each of
  the seven clouds promoted seven large layers and made things worse.
- The **horizon** line is `position:fixed`; its opacity is driven from the
  scroll handler via `--hz`, because a fixed rule left at full strength cuts a
  stray line straight through the contact form.
- **`prefers-reduced-motion` pins the backdrop entirely** — `.para`/`.paraM`
  get `transform:none`, and the canvas paints one frame instead of looping.
  Scroll-linked parallax is exactly the motion that setting is asking to stop.

There is no separate hero particle canvas any more; the site-wide network
replaced it, so the pointer reaches into the constellation anywhere on the page.

### Gotcha that cost real time

`setup()` runs near the top of the engine, above the draw helpers. A `var`
declared down with those helpers is hoisted but still `undefined` when `setup()`
fires, which turned every baked dimension into `NaN` — and `if (w < 2) return`
does **not** catch `NaN`, because every comparison against `NaN` is false. Hence
`BAKE` living in the top variable block and the guards being written
`if (!(w >= 2)) return`.

### The head

The figure beside the "perceive / decide / act" lines is Mario's own portrait,
taken apart into a point cloud and put back together in three dimensions, with
the six senses wired to one another: eyes and ears feed a central node, that
node drives the mouth, and the mouth loops back to the ears. Signals travel
along each wire in the direction the sense flows.

It used to be a sculpted head — a profile table of half-widths revolved into
elliptical rings. That was a good likeness of *a* head and a poor likeness of
*this* one, so it is gone. The source is now `head.webp` (with `head.png` as
the fallback the loader falls back to on `onerror`): a 480x732 cut-out made
with Vision's person segmentation, its neck faded out at the bottom.

Five things about it are deliberate and easy to undo by accident:

- **The three constants are measured, not guessed.** `CX0` (the axis it turns
  around), `CY0` (the eye line, model y = 0) and `SCL` (photo pixels per model
  unit) come from Vision face landmarks run over the cut-out, and so do the six
  `NODE` positions. Move the asset and every one of them is wrong. Re-measure;
  do not nudge until it looks right.
- **Depth is a distance transform, not a sphere.** A chamfer distance transform
  of the silhouette — far from an edge = far forward — raised to `pow(dn, 0.85)`
  and scaled by `ZMAX`, plus a luminance term that digs out the sockets and
  lifts the brow. An earlier `sin()` dome put nearly the whole face at full
  depth, so it slid sideways as a slab while the outline stayed put.
- **The turn is small on purpose.** There is no photograph of the back of the
  head, so the silhouette is a mirrored shell and cannot actually turn. Past
  roughly 15 degrees the face starts sliding across a stationary outline and
  the illusion breaks. The angle is `sin(t * 0.62) * MAXA` — about 14 degrees
  each way on a roughly ten-second cycle, enough parallax to sell the volume,
  not enough to show the seam. `MAXA` is the amplitude, the `0.62` the speed;
  they are the two numbers to reach for, and the amplitude is the one with a
  hard ceiling.
- **The rows are staggered, not jittered.** Sampled straight onto the square
  grid, the cloud read as a halftone screen. Scattering each dot freely fixed
  the mesh and smeared the eyes and lips with it. The rows are offset half a
  cell hex-fashion and only lightly jittered on top.
- **The canvas sizes itself.** `size()` sets the backing store from the
  element's real box times the device pixel ratio, capped at 2.6x and at
  1000px a side; `cx`, `cy`, `R` and the dot size all follow from it. The
  fixed 680 in the markup is only a first paint: on a 3x phone it was two
  thirds of the resolution the screen can show, and a point cloud that soft
  stops reading as a face. The grain drops to every 5th pixel below 900px
  wide, so a phone draws a third fewer dots at a higher resolution. A
  debounced `resize` re-runs it, and only rebuilds the cloud if the grain
  actually changed.
- **It draws into an ImageData, not with `arc()`.** At ~18,000 dots a frame,
  an object per dot with a `Math.cos` inside it cost about 12ms. Flat typed
  arrays, hoisted trig and square writes straight into the pixel buffer bring
  the whole draw to about 0.6ms on a desktop and 3.3ms on a 4x-throttled
  iPhone 13. Keep the hot loop allocation-free. `bounds()` measures the region
  the cloud can ever reach, at both ends of the sweep, so the per-frame clear
  and `putImageData` never touch the wide empty margin around the portrait.

Draw order carries the depth: the cloud (mirrored shell first, then the photo)
→ wires behind the face, painted underneath with `destination-over` so they
show through the gaps → wires in front → nodes. The front wires are drawn last
and given a glow on purpose; sandwiched into the cloud they disappeared into
the dots, which defeats the whole illustration.

Under `prefers-reduced-motion` it paints one frame at angle 0.14. If the page
is opened straight from `file://`, `getImageData` throws on the tainted canvas
and it falls back to drawing the photo itself, dimmed, still wired up.

The markets it used to illustrate (US, EU, remote) did not disappear: they moved
into the entity record in the Company section, where the rest of the company
facts already live.

### The header on narrow phones

The brand, the language pair, the call to action and the burger need about
356px on one row; a 360px phone offers 320 inside the gutter. Below 420px the
spacing tightens, and below 380px the language pair leaves the header for the
drawer, where `.drawer-langs` gives it a real touch target instead of a 25px
one.

It is deliberately moved rather than hidden — the site is bilingual and a
German visitor on a small phone still has to be able to switch. That means
there are two `.langs` groups in the document, so the switcher is built and
wired over `querySelectorAll('.langs')` and `apply()` presses the state on
every button in both. Adding a third group anywhere would work the same way;
reverting either to `getElementById('langs')` would silently strip the drawer
one of its behaviour.

### The journey section

Pins `#how` centred and scrubs the track horizontally for exactly
`track.scrollWidth - view.clientWidth` pixels. Any extra end distance turns
directly into blank screen. The card entrance animates on **x**, not y —
`.jview` is `overflow:hidden` during the pin and a y-offset gets clipped.

## Regenerating og.png

Render an HTML template at 1200×630 with Playwright and screenshot it. The template
used is not kept in the repo — rebuild it from the hero (grid + aurora + dots +
horizon, Newsreader headline, mono footer row) or re-shoot the hero directly.

## Testing before you push

- Load at 1440px and at 390px, scroll the whole page, check the console is clean.
- Check the journey pin: card 01 must be the entry state and no card may be clipped.
- Submit the empty form: it must block and mark five fields.
- Switch EN/DE and reload — the choice must survive, on all three pages.
- Cmd-P: the print stylesheet renders a clean white document with every FAQ answer
  open and the form hidden.
- If you touch the backdrop, re-measure frame time while scrolling before you
  push. It is very easy to add one innocent-looking blurred or masked full-size
  layer and lose 30ms a frame.
