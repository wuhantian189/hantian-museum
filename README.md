# The Hantian Wu Museum — portfolio site

A static site. The gallery (entrance → concourse → hall → project → case study) is
rendered by the Claude Design canvas runtime (`support.js`), which loads React and
Babel from unpkg at runtime and compiles `index.html`'s template in the browser.

## Run

Double-click `index.html`, or serve the folder:

```sh
python3 -m http.server 8000
# then open http://127.0.0.1:8000/
```

Both work and render identically. Opening the file directly works because the content
loads as a classic script (`<script src="./museum-content.js">`) publishing
`window.MUSEUM_CONTENT`, rather than as an ES module. Chrome treats a `file://` page as
origin `null` and blocks a dynamic `import()` from it as cross-origin, which silently left
every string on the page empty — the gallery rendered, but with no title, no statement and
no wall labels. A classic script has no such restriction.

Any static host works too (GitHub Pages, Netlify, Cloudflare Pages, S3). There is no build
step. The only external dependencies are unpkg (React 18.3.1, ReactDOM, Babel standalone)
and Google Fonts (Source Serif 4, Noto Serif SC), so the page does need a network
connection for its first paint.

## Layout

| Path | What it is |
|---|---|
| `index.html` | The page that is served. A copy of `Museum.dc.html`. |
| `Museum.dc.html` | The design source, kept in sync with the Claude Design project. |
| `museum-content.js` | All content: 9 projects, 3 halls, UI strings — every string bilingual EN/中文. Publishes `window.MUSEUM_CONTENT`. |
| `support.js` | Claude Design canvas runtime (generated — do not edit). |
| `_ds/broadsheet-…/` | Broadsheet design-system tokens (`styles.css`) and print-plate filters. |
| `img/hall-*.jpg` | The four gallery illustrations (entrance, and one per hall). |
| `img/slides/<project>/` | Page images rendered from each project's own source document. |
| `img/web/tv-*` | TaleVision product renders, field research and user-test photos. |
| `video/` | Project films, H.264 1280px wide with poster frames. |

Editing content means editing `museum-content.js`. Every visible string is a
`T("english", "中文")` pair; the EN/中文 control in the top right switches between them.

## Where the material came from

Rendered from the originals in `~/Documents/hantian_website`:

- **Wide project boards** (2692×947) — 慧帐宝, Mirrorcle, Wanderland, OpenCare, Philips
- **Slide decks** (1440×810) — TaleVision (23), HerMoony (25), 生成式设计大模型 (20)
- **Team document** — 小红书群聊旅游 AI, first 9 pages at 150 dpi
- **Videos** — transcoded to 1280px H.264 with `-crf 26` and `+faststart`:
  TaleVision project film (444 MB → 35 MB), Mirrorcle (259 MB → 36 MB),
  TravelShu demo (29 MB → 3.6 MB), school defence (362 MB → 31 MB)

Regenerating a slide set:

```sh
pdftoppm -jpeg -jpegopt quality=84 -r 72 "<source>.pdf" img/slides/<project>/p
# then zero-pad: p-1.jpg → 01.jpg
```

## Two content tracks

Each project in `museum-content.js` carries both:

- `sections[]` — original narrative prose (background → research → solution → build →
  validation → limits), each with its own images
- `parts[]` — sections taken from the project's own source document, each with the
  slide images from that document

**`parts` wins.** `heroOf`/`rawSecs` in the template use `parts` whenever it is present
and pass `images: []`, so a project with `parts` never shows its `sections` prose or
`sections` images. Eight of the nine projects have `parts`; only OpenCare has none, so
OpenCare is the one project whose narrative sections and section images are on screen.

Both tracks are complete and correct, so removing a `parts` block falls back cleanly to
that project's narrative. Videos are attached to `parts` entries, because that is the
track that renders.

## Responsive behaviour

The artboard is 1672×941 — a ratio of 1.777, effectively 16:9. Below 640px the three
gallery scenes keep that ratio exactly. From 640px up:

```css
[data-scene] { width: 100%; aspect-ratio: 1672 / 941; }
@media (min-width: 640px) {
  [data-scene] { height: max(100vh, calc(100vw * 941 / 1672)); aspect-ratio: auto; }
}
```

The box takes the greater of the window height and the aspect height. On a window wider
than 1.777 that keeps the true ratio and the page scrolls a little; on a taller one it
fills the window height and the artwork stretches with it. Either way no page background
is left showing, which is the whole point — a fixed ratio leaves a gap on any window that
is not roughly 16:9, and every MacBook is 16:10.

**Why nothing is cropped.** An earlier version covered the box and centre-cropped the
overflow, which is the usual way to fill without distorting. It does not work here: the
scene carries text positioned at fixed percentages — wall labels, taglines, the hall note
— so a crop eventually clips one of them. A 3% side crop was already cutting the end of
the HerMoony tagline. Stretch is the only transform that keeps every element on screen, so
stretch is what absorbs the mismatch.

Measured across window shapes, all with zero background showing and zero crop:

| window | stretch | scrolls |
|---|---|---|
| 16:10 fullscreen 1680×907 | 1.00 | yes, slightly |
| MacBook 14" fullscreen 1512×839 | 1.00 | yes, slightly |
| maximised + toolbar 1440×637 | 1.00 | yes |
| 4:3 window 1200×757 | 1.12 | no |
| 5:4 monitor 1280×881 | 1.22 | no |
| squarish 1100×857 | 1.38 | no |

## Hall paintings

All four scene images are current with the Claude Design project:

| file | hall | source |
|---|---|---|
| `hall-entrance.jpg` | Entrance facade | unchanged since the first import |
| `hall-research.jpg` | I · Interaction Design Research | repainted; from `img/_src/new image.png` |
| `hall-commercial.jpg` | II · AI+ Commercial Products | unchanged since the first import |
| `hall-room.jpg` | III · Early Works | repainted; from `img/_src/early works.png` |

Halls I and III were repainted to match Hall II's treatment — sage ceiling and floor,
deeper wall inks, and scene content specific to each project (clinicians and a wheelchair
for OpenCare, cats and dogs for Wanderland, shavers on a plinth for the Philips
internship). Painted figures were removed from the walls where the app draws its own. The
hall scene's fallback `background-color` is `#b3bba1` to match the new ceiling.

The two repainted files could not be pulled through the design API — it caps a response at
256 KiB and both exceed it, so they arrived truncated at about 48%. They were supplied
directly instead, and encoded here from the 1672×941 PNG masters at JPEG quality 92. That
encode is pixel-identical to the design's own copy across the band that could be
downloaded, so nothing was lost in the conversion.

`img/_src/` holds those two PNG masters (5.2 MB). Nothing references them — they are kept
only as the originals and can be deleted before deploying.

