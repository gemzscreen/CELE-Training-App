# CELE Figure Library

Redrawn board-exam figures, kept separate from the app so they can be reused when a
later transcription turns up the same drawing with different numbers.

The board recycles situations. The eccentric pole, the bolted butt splice, the angle
welded to a gusset, the anchored sheet pile — these come back sitting after sitting with
the geometry unchanged and only the values swapped. This folder exists so that case costs
a label edit rather than a redraw.

```
figures/
├── README.md              this file
├── manifest.json          machine-readable index: archetype, tags, viewBox, label list
├── figures.js             drop-in bundle — window.FIGLIB["psad-2026-03"][6]
├── proof-sheet.html       searchable browser, renders every figure inline
├── psad-2026-03/
│   ├── sit-01.svg … sit-19.svg
│   └── sit-20.html        raster (see "Two exceptions" below)
└── psad-2025-11/
    ├── sit-01.svg … sit-20.svg
    └── sit-17.html        raster (interaction diagram C4-60.7)
```

Figures carried over between sittings record it in `manifest.json` under `reusedFrom`.
Three of the November 2025 figures came straight out of the March 2026 folder.

## Finding the figure you need

Open `proof-sheet.html` and search by archetype, tag, or caption — `weld`, `truss`,
`torsion`, `prestressed`, `sheet-pile`. Every figure carries an archetype description
written in terms of what it *shows*, not which situation it happened to be, so a new
problem can be matched to it without knowing the source exam.

Or grep `manifest.json`:

```bash
jq '.exams[].figures[] | select(.tags | index("fillet-weld"))' manifest.json
```

## Reusing one

1. Copy the `.svg` file into a new exam folder, renamed for its new situation number.
2. Retune the labels. `manifest.json` lists every `<text>` node in each figure with its
   coordinates — that list *is* the set of things a new problem is likely to change
   (dimension letters, member labels, load magnitudes, angles).
3. Reposition loads if the new problem moves them. Load arrows are plain `<line>`
   elements with `marker-end="url(#afN)"`; move the endpoints, leave the marker alone.
4. **Renumber the marker IDs.** Every figure defines its own `afN` / `dfN` / `hfN`
   markers where `N` is the situation number. Two figures sharing an ID in one document
   will collide and one will render with the other's arrowheads. If `sit-06.svg` becomes
   situation 9 in a new exam, rename `af6` → `af9` everywhere in that file.
5. Validate against the source crop before integrating. This step has caught real errors
   every time it has been run — the first redraw pass alone caught four.

## Conventions

Colours are constants shared across the set, so a figure copied into the app inherits the
same visual language:

| Token   | Hex       | Used for                                  |
|---------|-----------|-------------------------------------------|
| `INK`   | `#1B2A41` | primary linework, labels, load arrows     |
| `DIM`   | `#6B7A90` | dimension lines, hatching, secondary text |
| `LINE`  | `#DDE3EA` | figure border                             |
| accent  | `#2E7D8F` | supports (pin, roller, fixed)             |

Every figure is a standalone `<svg>` with its own `viewBox` and a white rounded
background rect. Nothing depends on outside CSS, so a file can be opened on its own,
dropped into the app's `SFIG` map, or pasted into a proof sheet unchanged.

## Two exceptions

**Situation 4** (isometric space frame) is redrawn, but its anchor coordinates were only
resolvable by zooming the source base grid to 4× to separate the dotted construction
lines. They are C = (b, −a, 0) on the +X side and D = (−c, −d, 0) on the −X side —
opposite sides, which is what makes the X-components cancel. Do not "tidy" those anchors.

**Situation 20** (column interaction diagram) is kept as the booklet's own raster with the
watermark stripped, not redrawn. Answers are read off the printed curves, so an
approximated curve family would produce wrong numbers. It is stored as `.html` because it
is an `<img>` element rather than an `<svg>`.

## Revision history

- **sit-06** — rebuilt against `source_6.png`. The earlier version drew the X-axis along
  the cut plane's own diagonal, which left no visible wedge between the axis and the
  plane, so θ had nowhere correct to sit. The projection now matches the source
  (horizontal = `(1, +0.122)`, depth = `(+59, −39)`, vertical true), the X-axis passes
  through the plane's centre, and θ sits in the wedge just right of the Y-axis and just
  below the X-axis, with the curved arrow running from the plane's near edge up to +X.
- **sit-14** — side view rebuilt against `source_14.png`. The connections were drawn as
  round dots reading as bolts; they are now fillet welds at the top and bottom of the
  connected leg. The angle gained a rounded toe and inner fillet (rolled profile), the
  column plate was lengthened to the source's proportion (≈ half the leg length above and
  below), and the load arrow now springs from the member's broken end rather than
  floating clear of it.

- **psad-2025-11 added.** Eighteen figures, three of them reused from `psad-2026-03`:
  situation 4 (guyed pole) and situation 12 (purlin) carried over unchanged, and
  situation 5 reused the March situation-6 inclined-section drawing with **both load
  arrows reversed** — November 2025 loads that bar in tension where March 2026 loaded it
  in compression. All three had their marker IDs renumbered (`af4` → `afv4`, `af6` →
  `afv6`, `af16` → `afv12`) so they cannot collide with their originals now that both
  sittings live in the same document. This is the collision trap described above, met in
  practice on the first reuse.
