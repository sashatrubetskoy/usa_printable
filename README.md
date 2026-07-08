# usa_printable

A printable US wall map. `usa_printable.pdf` is the hand-designed poster; the
rest of the repo is a pipeline that recreates it as an interactive web editor
with print-exact SVG/PDF export.

## Layout

```
usa_printable/
├── usa_printable.pdf     the poster (design target, hand-made)
├── scripts/              Python pipeline (run from repo root)
│   ├── build_cities.py         merge city/population sources -> data/cities_render.csv
│   ├── compute_metric.py       prominence/isolation score    -> data/cities_scored.csv
│   ├── build_web.py            project + serialize for web   -> web/data.js
│   └── make_reference_layer.py static matplotlib preview     -> out/reference_cities_albers.{pdf,svg}
├── data/
│   ├── ne_50m_*/               Natural Earth 50m layers (8 dirs, downloaded)
│   ├── sources/                raw downloads + provenance (census, GeoNames, Demographia)
│   ├── ne_join_urban_pop.csv   hand-curated Demographia -> Natural Earth name join
│   └── cities_*.csv            generated (pipeline outputs)
├── out/                  generated reference previews
└── web/                  the map editor (open web/index.html directly; file:// works,
                          no server or build step)
```

## Pipeline

All scripts run from the repo root:

```
python scripts/build_cities.py       # data/cities_render.csv
python scripts/compute_metric.py     # data/cities_scored.csv
python scripts/build_web.py          # web/data.js
python scripts/make_reference_layer.py   # out/reference_cities_albers.{pdf,svg} (optional preview)
```

Requires Python 3.12 with geopandas, pandas, matplotlib.

## Web app

Open `web/index.html` in a browser. Tune mode adjusts the label-thinning
parameters; Edit mode pins/drags individual labels. Edits autosave to
localStorage (key `usa_printable_edits_v1`) and can be saved/loaded as JSON.
The SVG/PDF buttons export the current layout at print size (US Letter
landscape, 792x612 pt).

## Data sources (not tracked; re-download to rebuild from scratch)

- **Natural Earth 50m** (https://www.naturalearthdata.com/downloads/50m-cultural-vectors/
  and 50m-physical-vectors): admin_0_countries, admin_0_boundary_lines_land,
  admin_1_states_provinces, admin_1_states_provinces_lines, lakes,
  populated_places, rivers_lake_centerlines_scale_rank, urban_areas —
  unzip each into `data/ne_50m_<layer>/`.
- **Demographia World Urban Areas 2025** (`data/sources/db-worldua.pdf`,
  parsed by hand into `data/sources/urban_agglomerations_demographia2025.csv`
  and joined via `data/ne_join_urban_pop.csv`).
- **US Census sub-county population estimates** (`data/sources/sub-est2024.csv`).
- **GeoNames cities15000** (`data/sources/geonames/cities15000.txt`).

## Git policy

Only deliverables are versioned: the poster PDF and the generated outputs
(`web/data.js`, `data/cities_*.csv`, `out/`). Scripts, the web app source, and
raw downloads are intentionally untracked (see `.gitignore`).

## Python <-> JS sync invariants

- `web/data.js`'s `style` block (written by `scripts/build_web.py`) is the
  single source for dot diameters / font sizes / cap fraction — the JS reads it
  with no fallbacks.
- The city tier rule exists twice: `tier_of()` in `scripts/build_web.py` and
  `tier()` in the web code. Keep them in sync.
- The 8-slot label geometry exists twice: `slot_centers()` in
  `scripts/build_web.py` (an intentional advance-width approximation used only
  for the in-country slot-preference mask) and `slotAnchors()` in the web code
  (exact ink metrics). The slot offsets (r+g, g*0.6, th*0.6, th*0.7) must
  match; if one changes, change the other and rebuild `web/data.js`.
