# FIT2179 DV2 — Coding Style Guide
**For Claude Code:** This is a companion to `ClaudeCode_Prompt.md` (the main build instructions) and `DV2_DataContext.md` (data specs). Read all three before writing any code.  
All patterns come directly from the FIT3179 course studio code — your tutor will mark based on these exact conventions.

---

## READ THESE FILES FIRST

The actual studio source code is in `studio_reference/`. Read each file before building the corresponding chart type:

| Task | Read this file |
|------|---------------|
| Any HTML page layout | `studio_reference/week9_multiple_charts/index.html` |
| Any CSS styling | `studio_reference/week9_multiple_charts/css/styles.css` |
| Symbol map (circles on map) | `studio_reference/week8_symbol_map/js/symbol_map.vg.json` |
| Symbol map HTML embedding | `studio_reference/week8_symbol_map/index.html` |
| Choropleth map | `studio_reference/week8_choropleth/js/choropleth_map.vg.json` |
| Choropleth HTML embedding | `studio_reference/week8_choropleth/index.html` |
| Interactive chart with params | `studio_reference/week9_scatter/js/scatter_plot_interactive.vg.json` |
| Multiple charts on one page | `studio_reference/week9_multiple_charts/index.html` |
| Responsive chart JSON | `studio_reference/week9_multiple_charts/line-chart-responsive.json` |
| Normalised stacked bars | `studio_reference/week9_multiple_charts/normalised-stacked-bars-responsive.json` |

---

## WEEK 8 STYLE — Maps

### What to copy from `week8_symbol_map/index.html`
- CDN versions: `vega@5.20.2`, `vega-lite@5.1.0`, `vega-embed@6.17.0`
- Single div per chart: `<div id="symbol_map"></div>`
- vegaEmbed call: `vegaEmbed('#symbol_map', 'js/symbol_map.vg.json')`
- Separate CSS file linked: `<link rel="stylesheet" href="css/styles.css">`

### What to copy from `week8_symbol_map/js/symbol_map.vg.json`
- Layer structure for symbol maps:
  - Layer 0: topojson geoshape as base (fill lightgray, stroke white)
  - Layer 1: circles at lat/lon from CSV data
- `"tooltip": {"content": "data"}` for quick full-row tooltips on circles
- Quantitative size encoding with explicit `"scale": {"domain": [min, max]}`
- Quantitative color encoding with named scheme

### What to copy from `week8_choropleth/js/choropleth_map.vg.json`
- Load topojson as main `"data"` source (not in a layer)
- Use `"transform": [{"lookup": ..., "from": {...}}]` to join CSV data to shapes
- `"calculate": "datum.value + 0.1"` trick when you need log scale (avoids log(0))
- Color encoding with `"scale": {"type": "log"}` for skewed distributions
- Tooltip arrays with `{"field": "properties.NAME", "type": "nominal", "title": "..."}`

### What to copy from `week8_choropleth/css/styles.css`
- Minimal CSS — just positioning the chart div with top/bottom/left/right offsets
- No fixed width/height on the div — let Vega handle it

---

## WEEK 9 STYLE — Interactive Charts & Multi-Chart Pages

### What to copy from `week9_multiple_charts/index.html`
This is the **master template** for the assignment HTML page. Copy its exact structure:

```
<head>
  Vega CDN scripts (same versions as week 8)
  Pure.css CDN with integrity hash
  <meta name="viewport" ...>
  Google Fonts (Open Sans)
  Local CSS link
</head>
<body>
  <div class="page">
    [pure-g sections here]
  </div>
  <script>
    var specVis1 = "./chart1.vg.json";
    vegaEmbed('#vis1', specVis1, {"actions": false});
  </script>
</body>
```

**Key differences from week 8 HTML:**
- Uses `{"actions": false}` in ALL vegaEmbed calls (hides the export/edit menu)
- Uses `var specVisN = "./path.json"` variable then passes to vegaEmbed
- Has `<title>` in head
- Has Google Fonts import
- All content inside `<div class="page">`

### What to copy from `week9_multiple_charts/css/styles.css`
This is the **master CSS**. Copy exactly, then extend:

```css
body * { font-family: 'Open Sans', sans-serif; }
h1 { font-weight: 900; }
body { background-color: lightgray; }   /* change to #f0f2f5 for our project */
.page { width: 850px; background-color: white; margin: auto; padding: 50px; padding-top: 35px; }
.vis-container { width: 100%; }
.description h2, h1 { margin-top: 0px; }
.description-right { padding-left: 20px; }
.description-left { padding-right: 20px; }
.pure-g { margin-bottom: 40px; }
.small-font { font-size: 14px; }
```

**For our project:** Change `width: 850px` to `width: 1000px` and `background-color: lightgray` to `background-color: #f0f2f5`.

### What to copy from `week9_multiple_charts/line-chart-responsive.json`
- `"width": "container"` makes chart fill its HTML div — use on all non-map charts
- `"interpolate": "monotone"` on line marks for smooth curves
- `"legend": {"orient": "none", "legendX": 10, "legendY": 5}` to position legend inside chart

### What to copy from `week9_multiple_charts/normalised-stacked-bars-responsive.json`
- `"stack": "normalize"` on y encoding for 100% stacked bars
- `"axis": {"format": "%"}` for percentage axis
- `"labelExpr"` in legend to rename category labels without changing the data

### What to copy from `week9_scatter/js/scatter_plot_interactive.vg.json`
- `"params"` array at top level — before `"transform"` and `"encoding"`
- Range slider param: `{"name": "X", "value": default, "bind": {"input": "range", "min":..., "max":..., "step":..., "name": "Label: "}}`
- Select dropdown param: `{"name": "X", "bind": {"input": "select", "options": [...], "labels": [...], "name": "Label: "}}`
- Filter transform using param: `{"filter": "param_name == null || datum.field == param_name"}`
- `null` as first option = "Show All" — always do this for dropdowns
- Layered text labels: mark type "text" with conditional `"opacity"` — 1 for named items, 0 for others
- Text mark properties: `"align": "right"`, `"baseline": "middle"`, `"dx": -12`, `"fontSize": 11.5`, `"fontStyle": "italic"`

---

## WEEK 10 STYLE — Advanced Interactions (from PDF — no source files)

### Overview+Detail brushing
From the S&P500 example in the PDF. Exact structure:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "data": {"url": "../data/approvals_national_monthly.csv"},
  "vconcat": [
    {
      "width": "container",
      "height": 280,
      "mark": "area",
      "encoding": {
        "x": {
          "field": "date", "type": "temporal",
          "scale": {"domain": {"param": "brush"}},
          "axis": {"title": ""}
        },
        "y": {"field": "total", "type": "quantitative"}
      }
    },
    {
      "width": "container",
      "height": 60,
      "mark": "area",
      "title": "Drag to select a time period",
      "params": [
        {"name": "brush", "select": {"type": "interval", "encodings": ["x"]}}
      ],
      "encoding": {
        "x": {"field": "date", "type": "temporal"},
        "y": {"field": "total", "type": "quantitative", "axis": {"tickCount": 3, "grid": false}}
      }
    }
  ]
}
```

**Key rules from PDF:**
- Brush is defined in the BOTTOM (navigator) view with `"params"`
- Top view RESPONDS by setting `"scale": {"domain": {"param": "brush"}}` on x
- Bottom view height = ~20% of top view height
- Top view x-axis title set to `""` (they share it)
- Both views share the same top-level `"data"`

### Zoomable map with params
From the Victorian house price example in the PDF:

```json
{
  "params": [
    {
      "name": "Year_selection",
      "value": 2018,
      "bind": {"input": "range", "min": 2010, "max": 2020, "step": 1, "name": "Year: "}
    },
    {
      "name": "zoom_level",
      "value": 30000,
      "bind": {"input": "range", "min": 3500, "max": 60000, "step": 100, "name": "Zoom: "}
    },
    {
      "name": "center_to",
      "value": [145, -37.95],
      "bind": {
        "input": "select",
        "options": [[145, -37.95], [144.3, -38.1], [144.9, -36.7], [147.1, -38.1]],
        "labels": ["Melbourne CBD", "Geelong", "Bendigo", "Sale"],
        "name": "Map Centre: "
      }
    }
  ],
  "projection": {
    "type": "equirectangular",
    "center": {"expr": "center_to"},
    "scale": {"expr": "zoom_level"}
  }
}
```

**Key rules from PDF:**
- `"center"` and `"scale"` use `{"expr": "param_name"}` — this is how params bind to projection
- Center param value is an ARRAY `[lon, lat]` — not separate fields
- For Australia maps: use `"mercator"` projection instead of `"equirectangular"`
- Filter data by param value using: `{"filter": "datum.year == Year_selection"}`

### Small multiples with repeat operator
From the Victorian house price small multiples in the PDF:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "repeat": ["field1", "field2", "field3"],
  "columns": 3,
  "spec": {
    "width": 280,
    "height": 200,
    "layer": [
      {
        "data": {"url": "topojson_file.json", "format": {"type": "topojson", "feature": "FEATURE_NAME"}},
        "mark": {"type": "geoshape", "fill": "#ddd", "stroke": "white", "strokeWidth": 1}
      },
      {
        "data": {"url": "../data/state_summary.csv"},
        "transform": [
          {"lookup": "locality_key",
           "from": {"data": {"url": "topojson_file.json", ...}, "key": "properties.KEY"},
           "as": "geo"}
        ],
        "mark": {"type": "geoshape"},
        "encoding": {
          "shape": {"field": "geo", "type": "geojson"},
          "color": {"field": {"repeat": "repeat"}, "type": "quantitative", "scale": {"scheme": "blues"}},
          "tooltip": [
            {"field": "state_name", "type": "nominal"},
            {"field": {"repeat": "repeat"}, "type": "quantitative", "format": ".2f"}
          ]
        }
      }
    ]
  }
}
```

**Key rules from PDF:**
- `{"field": {"repeat": "repeat"}}` is the magic — it substitutes the current repeat value as the field name
- `"columns": N` controls how many views per row
- You can have multiple `layer` entries inside `spec` (unlike `facet` which doesn't support this)
- Add a text label layer with `{"data": {"values": [...]}}` and `{"field": {"repeat": "repeat"}}` to show the metric name on each small map
- For our project: use the **simpler approach** — load topojson as main data and use lookup transform (same as choropleth Pattern C)

### Multiple coordinated views (cross-chart brushing)
From the earthquake coordinated views example in the PDF:

```json
{
  "data": {"url": "shared_data.csv"},
  "vconcat": [
    {
      "layer": [...],
      "transform": [{"filter": {"param": "time_brush"}}]
    },
    {
      "height": 60,
      "params": [
        {"name": "time_brush", "select": {"type": "interval", "encodings": ["x"]}}
      ],
      "encoding": {"x": {"field": "date", "type": "temporal"}, "y": {...}}
    },
    {
      "transform": [
        {"bin": {"step": 0.5, "extent": [5, 7]}, "field": "mag", "as": "magnitude"}
      ],
      "encoding": {
        "x": {"scale": {"domain": {"param": "time_brush"}}}
      }
    }
  ]
}
```

**Key rules from PDF:**
- Define brush in one view with `"params"`, respond in others with `"filter": {"param": "brush_name"}` OR `"scale": {"domain": {"param": "brush_name"}}`
- All views must share the same top-level `"data"` to cross-filter correctly
- Build each view individually first WITHOUT interactions, then add the param connections

---

## TOPOJSON FILES — Map Reference Assets

### Australian states topojson
- **File:** `topojson/aus-states.topojson`
- **Source:** Downloaded from `https://raw.githubusercontent.com/rowanhogan/australian-states/master/states.min.geojson`
- **Feature name:** Check with `python3 -c "import json; d=json.load(open('topojson/aus-states.topojson')); print(list(d['objects'].keys()))"`
- **Property for state name:** `STATE_NAME` (expected — verify before use)
- **Join key:** `properties.STATE_NAME` (topojson) matches `state_name` column in `data/state_summary.csv`

### Course topojson files (referenced in week 8/9 studio code — available via GitHub raw URLs)
These can be used as base layers or reference:
- World countries: `https://raw.githubusercontent.com/FIT3179/Vega-Lite/main/2_symbol_map/js/ne_110m_admin_0_countries.topojson`
- Oceans: `https://raw.githubusercontent.com/FIT3179/Vega-Lite/main/7_others/oceans.topojson`
- Graticules: `https://raw.githubusercontent.com/FIT3179/Vega-Lite/main/2_symbol_map/js/WorldMapWithGraticules.topojson`
- Victoria localities (Week 10 example): `https://raw.githubusercontent.com/FIT3179/Vega-Lite/main/6_advanced_examples/data/VIC_LOCALITY_POLYGON_SHP.json`

---

## EXACT CDN VERSIONS TO USE

Copy these exact script tags — the tutor's examples use these versions:

```html
<script src="https://cdn.jsdelivr.net/npm/vega@5.22.1"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-lite@5.5.0"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-embed@6.21.0"></script>
<link rel="stylesheet" href="https://unpkg.com/purecss@2.0.3/build/pure-min.css"
  integrity="sha384-cg6SkqEOCV1NbJoCu11+bm0NvBRc8IYLRGXkmNrqUBfTjmMYwNKPWBTIKyw9mHNJ" crossorigin="anonymous">
```

---

## PURE.CSS GRID CHEAT SHEET

From `week9_multiple_charts/index.html` — these are the only grid classes used in the course:

| Class | Width |
|-------|-------|
| `pure-u-1-1` | 100% (full width) |
| `pure-u-1-2` | 50% (two equal columns) |
| `pure-u-1-3` | 33.3% (three equal columns) |
| `pure-u-2-3` | 66.7% (two-thirds width) |
| `pure-u-1-4` | 25% |
| `pure-u-3-4` | 75% |

**Always wrap columns inside `<div class="pure-g">`**

Two-column layout (text left, chart right):
```html
<div class="pure-g">
  <div class="pure-u-1-2">
    <div class="description description-left">
      <h2>Section heading</h2>
      <p>Narrative text using real data facts...</p>
    </div>
  </div>
  <div class="pure-u-1-2">
    <div id="chart3" class="vis-container"></div>
  </div>
</div>
```

Full-width chart:
```html
<div class="pure-g">
  <div class="pure-u-1-1">
    <div id="chart1" class="vis-container"></div>
  </div>
</div>
```

---

## COMMON MISTAKES TO AVOID

These are errors from past students — don't repeat them:

1. **Using `"width": "container"` on maps** — maps must have fixed pixel width/height (e.g. `"width": 800, "height": 450`)
2. **Forgetting `{"actions": false}`** in vegaEmbed — always include it
3. **Absolute data paths** — always use relative `"../data/filename.csv"`, never absolute paths
4. **Wrong topojson feature name** — always run the python3 check before writing map JSON
5. **Lookup join mismatch** — `properties.STATE_NAME` in topojson must EXACTLY match `state_name` values in CSV (check case, spacing, and special characters like "Australian Capital Territory")
6. **Missing config block** — every `.vg.json` needs the Open Sans config or chart fonts won't match the page
7. **Lorem Ipsum in HTML** — write real sentences about Australian housing using facts from DV2_DataContext.md
8. **Params before transforms** — `"params"` must come BEFORE `"transform"` in the JSON object
9. **vconcat without shared data** — for cross-chart brushing to work, both views must reference the same top-level `"data"` 
10. **Forgetting `"axis": {"title": ""}` on top view x-axis** in overview+detail charts — without this, both views show duplicate axis labels

---

## DECISION TREE — Which pattern to use?

```
Is it a geographic map?
  YES → Does it show state/territory boundaries filled by a value?
          YES → Choropleth (Pattern C from ClaudeCode_Prompt.md)
                  + Add zoom params? → Add Pattern D
                  + Show 3 metrics? → Use repeat (Pattern G)
                  + YoY or signed value? → Use diverging color (Pattern F)
        Does it show cities/LGAs as circles?
          YES → Symbol map (Pattern E)
  NO  → Is it a time series over 40 years?
          YES → vconcat overview+detail (Pattern H)
        Does it filter by dropdown or slider?
          YES → Add params + filter transform (Pattern I)
        Does it compare two metrics per category on same axis?
          YES → Connected dot plot (Pattern J)
        Otherwise → Standard chart (Pattern A) with layered annotations (Pattern B) where needed
```
