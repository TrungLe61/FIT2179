# FIT2179 DV2 — Claude Code Build Instructions
**Open Claude Code at:** `/Users/trungle/Documents/Claude/Projects/FIT2179/`

## 📚 READ THESE FILES FIRST — BEFORE WRITING ANY CODE

Do all of the following reads before starting any task:

1. **`DV2_DataContext.md`** — column names, data facts, chart specs, lat/lon coordinates, topojson notes
2. **`STYLE_GUIDE.md`** — coding patterns, common mistakes, which studio file to reference per task
3. **`studio_reference/week8_choropleth/js/choropleth_map.vg.json`** — exact choropleth pattern
4. **`studio_reference/week8_symbol_map/js/symbol_map.vg.json`** — exact symbol map pattern
5. **`studio_reference/week9_scatter/js/scatter_plot_interactive.vg.json`** — exact params/filter pattern
6. **`studio_reference/week9_multiple_charts/index.html`** — master HTML template with pure.css grid
7. **`studio_reference/week9_multiple_charts/line-chart-responsive.json`** — responsive width pattern
8. **`studio_reference/week9_multiple_charts/css/styles.css`** — master CSS (copy this as your base)

After reading all 8 files, you will have everything you need. Do not start coding until all 8 are read.

---

## ⚠️ CRITICAL RULES (read before writing a single line of code)

1. **50% of charts must be geographic maps** — 6 out of 12 are maps (Charts 3,4,5,6,9,10)
2. **Maps must use custom idioms** — not a basic copy-paste choropleth. Each map needs something creative: zoomable params, diverging colour, repeat/small multiples, or sized circles
3. **Minimum 10 charts total** — we have 12, do not remove any
4. **Single scrollable page** — no tabs, no buttons that swap sections
5. **All vegaEmbed calls must use `{"actions": false}`**
6. **All data URLs must be relative paths** — `"../data/filename.csv"` not absolute paths
7. **Every `.vg.json` must have the `config` block** with Open Sans font (see Pattern Library)
8. **Write real narrative text** in index.html using data facts from DV2_DataContext.md — not Lorem Ipsum

---

## YOUR FIRST TASKS (do these before any chart code)

```bash
# 1. Check the topojson feature name — run this first before writing any map JSON
python3 -c "
import json
d = json.load(open('topojson/aus-states.topojson'))
print('Objects:', list(d['objects'].keys()))
first = list(d['objects'].values())[0]['geometries'][0]
print('Properties:', list(first['properties'].keys()))
print('Sample name:', first['properties'])
"

# 2. Add lat/lon to gccsa_summary.csv
python3 -c "
import pandas as pd
coords = {
  'Sydney': (151.21, -33.87), 'Melbourne': (144.96, -37.81),
  'Brisbane': (153.02, -27.47), 'Perth': (115.86, -31.95),
  'Adelaide': (138.60, -34.93), 'Hobart': (147.33, -42.88),
  'Darwin': (130.84, -12.46), 'Canberra': (149.13, -35.28)
}
df = pd.read_csv('data/gccsa_summary.csv')
df['lon'] = df['city_name'].map(lambda x: next((v[0] for k,v in coords.items() if k in x), None))
df['lat'] = df['city_name'].map(lambda x: next((v[1] for k,v in coords.items() if k in x), None))
df.to_csv('data/gccsa_summary.csv', index=False)
print(df[['city_name','lon','lat']])
"

# 3. Add lat/lon to top 20 LGAs in population_lga_2024_25.csv
# Use coordinates from DV2_DataContext.md — add lat/lon columns, set NULL for LGAs not in top 20
```

---

## CODING CONVENTIONS

### HTML (`index.html`)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Australia's Housing Supply Crisis | Trung Le</title>
  <script src="https://cdn.jsdelivr.net/npm/vega@5.22.1"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-lite@5.5.0"></script>
  <script src="https://cdn.jsdelivr.net/npm/vega-embed@6.21.0"></script>
  <link rel="stylesheet" href="https://unpkg.com/purecss@2.0.3/build/pure-min.css"
    integrity="sha384-cg6SkqEOCV1NbJoCu11+bm0NvBRc8IYLRGXkmNrqUBfTjmMYwNKPWBTIKyw9mHNJ" crossorigin="anonymous">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;700;900&display=swap" rel="stylesheet">
  <link rel="stylesheet" type="text/css" href="css/styles.css" media="all">
</head>
<body>
  <div class="page">
    <!-- All content goes here in pure-g / pure-u-* grid sections -->
  </div>
  <script type="text/javascript">
    // One vegaEmbed per chart — always with {"actions": false}
    vegaEmbed('#chart1', './js/chart1_national_trend.vg.json', {"actions": false});
    vegaEmbed('#chart2', './js/chart2_rolling_total.vg.json', {"actions": false});
    // ... etc
  </script>
</body>
</html>
```

### CSS (`css/styles.css`)

```css
body * { font-family: 'Open Sans', sans-serif; }
h1 { font-weight: 900; font-size: 2em; margin-bottom: 8px; }
h2 { font-weight: 700; color: #1565C0; margin-top: 0; }
h3 { font-weight: 700; font-size: 1em; color: #333; }
body { background-color: #f0f2f5; }
.page { width: 1000px; background-color: white; margin: auto; padding: 50px; padding-top: 35px; }
.vis-container { width: 100%; }
.description-right { padding-left: 20px; }
.description-left { padding-right: 20px; }
.pure-g { margin-bottom: 40px; }
.small-font { font-size: 13px; color: #666; }
.section-divider { border-top: 2px solid #1565C0; margin: 40px 0 20px 0; padding-top: 20px; }
.section-label { font-size: 0.8em; font-weight: 700; color: #1565C0; text-transform: uppercase; letter-spacing: 1px; }
.stat-highlight { font-size: 1.8em; font-weight: 900; color: #1565C0; }
```

### Every Vega-Lite JSON must include this config block

```json
"config": {
  "title": {"font": "Open Sans", "fontSize": 14, "fontWeight": "bold"},
  "axis": {"labelFont": "Open Sans", "titleFont": "Open Sans", "labelFontSize": 11},
  "legend": {"labelFont": "Open Sans", "titleFont": "Open Sans"},
  "view": {"stroke": "transparent"}
}
```

---

## PATTERN LIBRARY (from FIT3179 studio weeks 8, 9, 10)

### Pattern A — Standard chart (Week 9)
```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "title": "Chart title",
  "width": "container", "height": 300,
  "data": {"url": "../data/file.csv"},
  "mark": "bar",
  "encoding": {"x": {...}, "y": {...}, "color": {...}, "tooltip": [...]},
  "config": { ...Open Sans config... }
}
```

### Pattern B — Layered chart with annotations (Week 8/9)
```json
{
  "layer": [
    {"mark": "line", "encoding": {...}},
    {
      "data": {"values": [
        {"date": "2008-09-01", "label": "GFC"},
        {"date": "2020-03-01", "label": "COVID-19"}
      ]},
      "layer": [
        {"mark": {"type": "rule", "strokeDash": [4,4], "color": "#999"},
         "encoding": {"x": {"field": "date", "type": "temporal"}}},
        {"mark": {"type": "text", "align": "left", "dx": 4, "dy": -10, "fontSize": 10, "fontStyle": "italic"},
         "encoding": {"x": {"field": "date", "type": "temporal"}, "y": {"value": 20}, "text": {"field": "label"}}}
      ]
    }
  ]
}
```

### Pattern C — Choropleth map (Week 8)
```json
{
  "width": 800, "height": 450,
  "projection": {"type": "mercator", "center": [134, -26], "scale": 850},
  "data": {
    "url": "../topojson/aus-states.topojson",
    "format": {"type": "topojson", "feature": "FEATURE_NAME"}
  },
  "transform": [{
    "lookup": "properties.STATE_NAME",
    "from": {
      "data": {"url": "../data/state_summary.csv"},
      "key": "state_name",
      "fields": ["approvals_fy2425", "approvals_per_1000", "yoy_change_pct", "state_name"]
    }
  }],
  "mark": {"type": "geoshape"},
  "encoding": {
    "color": {"field": "approvals_fy2425", "type": "quantitative", "scale": {"scheme": "blues"}, "legend": {"format": ".2s"}},
    "tooltip": [
      {"field": "state_name", "type": "nominal", "title": "State"},
      {"field": "approvals_fy2425", "type": "quantitative", "format": ",", "title": "Total Approvals"},
      {"field": "approvals_per_1000", "type": "quantitative", "format": ".2f", "title": "Per 1,000 People"}
    ]
  }
}
```
⚠️ Replace `FEATURE_NAME` with the actual key from the topojson file (run the python3 check first).

### Pattern D — Zoomable map with params (Week 10)
Add these params + dynamic projection to any choropleth:
```json
{
  "params": [
    {"name": "zoom_level", "value": 3000,
     "bind": {"input": "range", "min": 1500, "max": 6000, "step": 100, "name": "Zoom: "}},
    {"name": "center_to", "value": [134, -26],
     "bind": {
       "input": "select",
       "options": [[134,-26],[151,-33],[144,-37],[153,-27],[116,-32],[138,-35]],
       "labels": ["All Australia","New South Wales","Victoria","Queensland","Western Australia","South Australia"],
       "name": "Focus State: "
     }}
  ],
  "projection": {"type": "mercator", "scale": {"expr": "zoom_level"}, "center": {"expr": "center_to"}}
}
```

### Pattern E — Symbol map (Week 8)
Layer topojson base + circle marks at lat/lon:
```json
{
  "width": 800, "height": 450,
  "projection": {"type": "mercator", "center": [134, -26], "scale": 850},
  "layer": [
    {
      "data": {"url": "../topojson/aus-states.topojson", "format": {"type": "topojson", "feature": "FEATURE_NAME"}},
      "mark": {"type": "geoshape", "fill": "#e8e8e8", "stroke": "white", "strokeWidth": 0.5}
    },
    {
      "data": {"url": "../data/gccsa_summary.csv"},
      "mark": {"type": "circle", "opacity": 0.8},
      "encoding": {
        "longitude": {"field": "lon", "type": "quantitative"},
        "latitude": {"field": "lat", "type": "quantitative"},
        "size": {"field": "total", "type": "quantitative", "scale": {"range": [50, 2000]}, "legend": {"format": ".2s"}},
        "color": {"field": "approvals_per_1000", "type": "quantitative", "scale": {"scheme": "orangered"}},
        "tooltip": [...]
      }
    }
  ]
}
```

### Pattern F — Diverging choropleth (CUSTOM map idiom)
Use `domainMid: 0` and a diverging scheme for YoY change maps:
```json
"color": {
  "field": "yoy_change_pct",
  "type": "quantitative",
  "scale": {"domainMid": 0, "scheme": "blueorange", "reverse": true},
  "legend": {"title": "YoY Change %", "format": ".1f"}
}
```

### Pattern G — Small multiples map with repeat (Week 10)
```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v5.json",
  "repeat": ["approvals_fy2425", "approvals_per_1000", "yoy_change_pct"],
  "columns": 3,
  "spec": {
    "width": 280, "height": 200,
    "data": {"url": "../topojson/aus-states.topojson", "format": {"type": "topojson", "feature": "FEATURE_NAME"}},
    "transform": [{
      "lookup": "properties.STATE_NAME",
      "from": {"data": {"url": "../data/state_summary.csv"}, "key": "state_name",
               "fields": ["approvals_fy2425","approvals_per_1000","yoy_change_pct","state_name"]}
    }],
    "projection": {"type": "mercator", "center": [134, -26], "scale": 280},
    "mark": {"type": "geoshape"},
    "encoding": {
      "color": {"field": {"repeat": "repeat"}, "type": "quantitative", "scale": {"scheme": "blues"}},
      "tooltip": [{"field": "state_name", "type": "nominal"}, {"field": {"repeat": "repeat"}, "type": "quantitative", "format": ".2f"}]
    }
  }
}
```

### Pattern H — Overview+Detail brushing (Week 10)
```json
{
  "vconcat": [
    {
      "height": 280, "width": "container",
      "mark": "area",
      "title": null,
      "encoding": {
        "x": {"field": "date", "type": "temporal", "title": null, "scale": {"domain": {"param": "brush"}}},
        "y": {"field": "total", "type": "quantitative", "title": "Monthly Approvals"}
      }
    },
    {
      "height": 60, "width": "container",
      "mark": "area",
      "title": "Drag to zoom into a time period",
      "params": [{"name": "brush", "select": {"type": "interval", "encodings": ["x"]}}],
      "encoding": {
        "x": {"field": "date", "type": "temporal"},
        "y": {"field": "total", "type": "quantitative", "axis": {"tickCount": 3, "grid": false}, "title": null}
      }
    }
  ]
}
```

### Pattern I — Interactive scatter with hover labels (Week 9)
```json
{
  "params": [
    {"name": "state_highlight",
     "bind": {"input": "select",
              "options": [null,"NSW","VIC","QLD","WA","SA","TAS","NT","ACT"],
              "labels": ["Show All","NSW","VIC","QLD","WA","SA","TAS","NT","ACT"],
              "name": "Highlight: "}}
  ],
  "layer": [
    {
      "mark": "circle",
      "encoding": {
        "opacity": {"condition": {"test": "state_highlight == null || datum.state == state_highlight", "value": 0.85}, "value": 0.1},
        "size": {"field": "pop_2025", "type": "quantitative",
                 "scale": {"type": "threshold", "domain": [500000,1000000,3000000,6000000], "range": [50,150,300,500,800]}}
      }
    },
    {
      "mark": {"type": "text", "dy": -12, "fontSize": 11, "fontWeight": "bold"},
      "encoding": {"text": {"field": "state", "type": "nominal"}, "color": {"value": "#333"}}
    }
  ]
}
```

### Pattern J — Connected dot plot (CUSTOM non-map idiom)
```json
{
  "transform": [{"fold": ["approvals_per_1000", "pop_growth_pct"], "as": ["metric", "value"]}],
  "layer": [
    {
      "mark": {"type": "line", "strokeWidth": 2},
      "encoding": {
        "x": {"field": "value", "type": "quantitative"},
        "y": {"field": "city_name", "type": "nominal"},
        "detail": {"field": "city_name"},
        "color": {"value": "#ccc"}
      }
    },
    {
      "mark": {"type": "point", "filled": true, "size": 120},
      "encoding": {
        "x": {"field": "value", "type": "quantitative", "title": "Rate"},
        "y": {"field": "city_name", "type": "nominal", "sort": "-x", "title": null},
        "color": {
          "field": "metric", "type": "nominal",
          "scale": {
            "domain": ["approvals_per_1000", "pop_growth_pct"],
            "range": ["#1976D2", "#E53935"]
          },
          "legend": {
            "title": null,
            "labelExpr": "datum.label == 'approvals_per_1000' ? 'Supply (approvals per 1,000)' : 'Demand (population growth %)'"
          }
        },
        "tooltip": [
          {"field": "city_name", "type": "nominal", "title": "City"},
          {"field": "metric", "type": "nominal", "title": "Metric"},
          {"field": "value", "type": "quantitative", "format": ".2f", "title": "Value"}
        ]
      }
    }
  ]
}
```

---

## PAGE LAYOUT (index.html structure)

```
[Full-width header]
  h1: "Australia's Housing Supply Crisis"
  p: brief intro — use real facts from DV2_DataContext.md

[SECTION 1 — "The National Picture"]
  [full-width] Chart 1: National trend (vconcat overview+detail)
  [left=text | right=chart] Chart 2: Rolling total bar

[SECTION 2 — "Where Is Australia Building?"]
  [full-width] Chart 3: State choropleth (zoomable — use Patterns C+D)
  [full-width] Chart 4: Capital city bubble map (Pattern E)
  [left=chart | right=chart] Chart 5: YoY diverging choropleth (Pattern F) | [right] Chart 6: Small multiples map (Pattern G)

[SECTION 3 — "What Type of Homes?"]
  [full-width] Chart 7: Stacked area (vconcat — Pattern H)
  [left=text | right=chart] Chart 8: 100% bar housing mix

[SECTION 4 — "Is Supply Meeting Demand?"]
  [full-width] Chart 9: Supply choropleth (Pattern C — greens scheme)
  [full-width] Chart 10: LGA symbol map (Pattern E — circles sized by change_pct)
  [left=chart | right=text] Chart 11: Scatter (Pattern I)

[SECTION 5 — "The Supply-Demand Gap"]
  [full-width] Chart 12: Connected dot plot (Pattern J)

[Footer]
  Data: ABS Building Approvals 8731.0 (March 2026) | ABS Regional Population 3218.0 (FY2024-25)
  Author: Trung Le | FIT2179 Data Visualisation, Monash University, 2026
  AI acknowledgement: "This visualisation was created with assistance from Claude AI (Anthropic)."
```

---

## COLOUR PALETTE

| Use | Colour |
|-----|--------|
| Primary / supply / states | `#1565C0` (dark blue) |
| Demand / pressure / negative | `#E53935` (red) |
| Positive growth | `#2196F3` (blue) |
| Decline | `#F44336` (red) |
| Houses | `#1976D2` |
| Apartments | `#FF7043` |
| Map — supply quantity | scheme `"blues"` |
| Map — supply adequacy | scheme `"greens"` |
| Map — diverging YoY | scheme `"blueorange"`, reverse=true, domainMid=0 |
| Map — bubble intensity | scheme `"orangered"` |

---

## BUILD ORDER

1. Run the topojson python3 check — confirm feature name
2. Run python3 to add lat/lon to gccsa_summary.csv
3. Add lat/lon to top 20 LGAs in population_lga_2024_25.csv
4. Build `css/styles.css`
5. Build **Chart 3** (zoomable choropleth) — test in browser
6. Build **Chart 4** (bubble map) — test in browser
7. Build **Chart 5** (diverging choropleth)
8. Build **Chart 6** (small multiples map with repeat)
9. Build **Chart 9** (supply choropleth — greens)
10. Build **Chart 10** (LGA symbol map)
11. Build **Charts 1, 7** (vconcat overview+detail)
12. Build **Charts 2, 8, 11, 12** (standard charts)
13. Build `index.html` — assemble all 12 charts with real narrative text
14. Test all charts in browser, verify no broken data paths

---

## FINAL CHECKLIST

- [ ] 12 `.vg.json` files in `js/`
- [ ] 6 are geographic maps (Charts 3,4,5,6,9,10)
- [ ] All map JSONs have fixed width/height (NOT "container")
- [ ] All non-map JSONs use `"width": "container"`
- [ ] topojson feature name confirmed before use
- [ ] gccsa_summary.csv has lat/lon columns
- [ ] population_lga_2024_25.csv has lat/lon for top 20 LGAs
- [ ] Every `.vg.json` has the `config` Open Sans block
- [ ] Every vegaEmbed uses `{"actions": false}`
- [ ] All data URLs use `"../data/"` prefix
- [ ] index.html has real sentences in every section (not Lorem Ipsum)
- [ ] Footer has data sources, author name, AI acknowledgement
- [ ] Page is 1000px wide, white background, no horizontal scroll at 1280px
- [ ] Chart 1 and Chart 7 use vconcat overview+detail
- [ ] Chart 3 has zoom slider and Focus State dropdown
- [ ] Chart 5 uses diverging colour with domainMid=0
- [ ] Chart 6 uses repeat operator for 3 small map views
