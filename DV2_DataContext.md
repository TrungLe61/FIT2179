# FIT2179 DV2 — Data Context & Chart Plan
**Topic:** Australian Residential Building Approvals — Housing Supply vs Demand  
**Due:** 29 May 2026 | **Author:** Trung Le  
**GitHub Pages:** Single scrollable page, full width visible on laptop without horizontal scroll.

---

## Assignment Rules (Claude Code must respect these)

- **Minimum 10 charts** — we have 12 ✅
- **~50% geographic maps** — we have 6 maps out of 12 ✅
- **HD requires custom-built MAP idioms** — maps must use creative/non-standard idioms, not just a basic choropleth copy-paste
- **At least 1 map required** — losing maps entirely caps Idioms criterion at 5% instead of 10%
- **2 data sources** — ABS 8731.0 + ABS 3218.0 ✅
- **No buttons that swap major sections** — single scrollable page only
- **Data < a few MB total** — ~373 KB total ✅
- **HD requires custom-built idioms** — visualisations that cannot be easily created with AI, use derived data or combine idioms
- **Presentation not exploration** — storytelling first, interactivity only where it adds genuine meaning
- **Audience:** Average Australian — no jargon, explain all terms
- **AI use must be acknowledged** in the footer
- **Vega-Lite only** — no other charting libraries unless tutor-approved

---

## File Structure

```
/Users/trungle/Documents/Claude/Projects/FIT2179/
├── index.html
├── css/styles.css
├── js/
│   ├── chart1_national_trend.vg.json       ← non-map, CUSTOM
│   ├── chart2_rolling_total.vg.json         ← non-map
│   ├── chart3_state_choropleth.vg.json      ← MAP (zoomable)
│   ├── chart4_bubble_map.vg.json            ← MAP (symbol)
│   ├── chart5_yoy_choropleth.vg.json        ← MAP (diverging color) CUSTOM
│   ├── chart6_small_multiples_map.vg.json   ← MAP (repeat) CUSTOM
│   ├── chart7_stacked_area.vg.json          ← non-map, CUSTOM (overview+detail)
│   ├── chart8_100pct_bar.vg.json            ← non-map
│   ├── chart9_supply_choropleth.vg.json     ← MAP (per-1000 people)
│   ├── chart10_lga_symbol_map.vg.json       ← MAP (LGA circles) CUSTOM
│   ├── chart11_scatter.vg.json              ← non-map, CUSTOM
│   └── chart12_connected_dot.vg.json        ← non-map, CUSTOM
├── data/
│   ├── approvals_national_monthly.csv
│   ├── approvals_states_monthly.csv
│   ├── approvals_gccsa_monthly.csv
│   ├── population_lga_2024_25.csv           ← needs lat/lon added for top LGAs
│   ├── population_gccsa_2001_2025.csv
│   ├── state_summary.csv
│   └── gccsa_summary.csv                    ← needs lat/lon columns added
└── topojson/
    └── aus-states.topojson
```

---

## Data Files — Full Column Reference

### approvals_national_monthly.csv (513 rows, Jul 1983–Mar 2026)
| Column | Type | Description |
|--------|------|-------------|
| date | YYYY-MM-DD | First day of each month |
| houses | integer | House approvals (original series) |
| non_houses | integer | Apartment/other approvals |
| total | integer | Total (houses + non_houses) |
| total_sa | float | Total seasonally adjusted |

### approvals_states_monthly.csv (4,104 rows)
| Column | Type | Description |
|--------|------|-------------|
| date | YYYY-MM-DD | First day of month |
| state | string | NSW/VIC/QLD/SA/WA/TAS/NT/ACT |
| state_name | string | Full state name |
| approvals | integer | Total dwelling approvals |

### approvals_gccsa_monthly.csv (2,376 rows, Jul 2001–Mar 2026)
| Column | Type | Description |
|--------|------|-------------|
| date | YYYY-MM-DD | First day of month |
| city | string | City abbreviation |
| city_name | string | Full capital city name |
| total | integer | Total approvals |
| houses | integer | House approvals |
| non_houses | integer | Apartment approvals |

### state_summary.csv (8 rows)
| Column | Type | Description |
|--------|------|-------------|
| state | string | Abbreviation: NSW/VIC/QLD/SA/WA/TAS/NT/ACT |
| state_name | string | Full name — **join key to topojson `properties.STATE_NAME`** |
| approvals_fy2425 | integer | Total approvals FY2024-25 |
| approvals_fy2324 | integer | Total approvals FY2023-24 |
| yoy_change_pct | float | Year-on-year % change (positive=growth, negative=decline) |
| pop_2024 | integer | Population 2024 |
| pop_2025 | integer | Population 2025 |
| pop_growth_pct | float | Population growth % |
| approvals_per_1000 | float | Approvals per 1,000 people |

### gccsa_summary.csv (8 rows) — ADD lat/lon columns
| Column | Type | Description |
|--------|------|-------------|
| city | string | City abbreviation |
| city_name | string | Full city name |
| total | integer | Total approvals FY2024-25 |
| houses | integer | House approvals |
| non_houses | integer | Apartment approvals |
| house_pct | float | % of approvals that are houses |
| pop_2025 | integer | Population 2025 |
| pop_growth_pct | float | Population growth % |
| approvals_per_1000 | float | Approvals per 1,000 people |
| lat | float | **ADD THIS** — city latitude |
| lon | float | **ADD THIS** — city longitude |

**Hardcoded lat/lon for gccsa_summary.csv:**
```
Sydney:    lon=151.21, lat=-33.87
Melbourne: lon=144.96, lat=-37.81
Brisbane:  lon=153.02, lat=-27.47
Perth:     lon=115.86, lat=-31.95
Adelaide:  lon=138.60, lat=-34.93
Hobart:    lon=147.33, lat=-42.88
Darwin:    lon=130.84, lat=-12.46
Canberra:  lon=149.13, lat=-35.28
```

### population_lga_2024_25.csv (546 rows) — ADD lat/lon for top LGAs
| Column | Type | Description |
|--------|------|-------------|
| lga_code | string | ABS LGA code |
| lga_name | string | LGA name |
| erp_2024 | integer | Population 2024 |
| erp_2025 | integer | Population 2025 |
| change_pct | float | % population change |
| state_abbr | string | State abbreviation |
| lat | float | **ADD THIS** — approximate LGA centroid latitude |
| lon | float | **ADD THIS** — approximate LGA centroid longitude |

**Approximate lat/lon for top 20 fastest-growing LGAs (add to CSV):**
```
Melton (VIC):                   lon=144.58, lat=-37.68
Wyndham (VIC):                  lon=144.62, lat=-37.90
Sunbury (VIC):                  lon=144.73, lat=-37.58
Casey (VIC):                    lon=145.25, lat=-38.05
Whittlesea (VIC):               lon=145.08, lat=-37.60
Mambourin (VIC):                lon=144.49, lat=-37.96
Cranbourne (VIC):               lon=145.28, lat=-38.10
Rockbank - Mount Cottrell (VIC):lon=144.50, lat=-37.78
Ipswich (QLD):                  lon=152.77, lat=-27.61
Logan (QLD):                    lon=153.02, lat=-27.64
Moreton Bay (QLD):              lon=152.97, lat=-27.11
Gold Coast (QLD):               lon=153.40, lat=-28.00
Armadale (WA):                  lon=116.01, lat=-32.15
Swan (WA):                      lon=116.06, lat=-31.84
Wanneroo (WA):                  lon=115.80, lat=-31.67
Yanchep (WA):                   lon=115.63, lat=-31.55
Mandurah (WA):                  lon=115.72, lat=-32.53
Mawson Lakes (SA):              lon=138.62, lat=-34.83
Playford (SA):                  lon=138.70, lat=-34.68
Hume (ACT/NSW):                 lon=149.07, lat=-35.45
```
Note: If exact LGA names don't match, use the top 20 by change_pct from the actual CSV and assign the closest reasonable coordinates.

### population_gccsa_2001_2025.csv (400 rows)
| Column | Type | Description |
|--------|------|-------------|
| year | integer | Calendar year 2001–2025 |
| gccsa_name | string | e.g. "Greater Sydney", "Greater Melbourne" |
| population | integer | Estimated resident population |

**Capital city filter expression:**
```
indexof(['Greater Sydney','Greater Melbourne','Greater Brisbane','Greater Perth','Greater Adelaide','Australian Capital Territory','Greater Hobart','Darwin'], datum.gccsa_name) >= 0
```

---

## Key Data Facts (use in annotations and narrative text on the page)

| Fact | Value |
|------|-------|
| National total (Mar 2026) | 17,780 dwellings/month |
| GFC annotation date | 2008-09-01 |
| Mining Boom Peak date | 2012-06-01 |
| COVID dip date | 2020-03-01 |
| Rate Hikes Begin date | 2022-05-01 |
| State leader FY2024-25 | Victoria (56,581) |
| WA YoY growth | +31.4% |
| ACT YoY decline | −37.8% |
| Melbourne total FY2024-25 | 46,530 approvals |
| Perth houses % | 79.6% houses |
| Sydney houses % | 38.7% houses |
| Best supply ratio | Melbourne (8.56/1,000) |
| Worst supply ratio | Darwin (2.29/1,000) |
| Fastest growing capital | Perth (+2.43%) |
| National avg approvals/1,000 | ~5.8 |

---

## TopoJSON

**File:** `topojson/aus-states.topojson`  
**Download from:** `https://raw.githubusercontent.com/rowanhogan/australian-states/master/states.min.geojson`

**BEFORE writing any map JSON**, run this to confirm the feature name and property key:
```bash
python3 -c "
import json
d = json.load(open('topojson/aus-states.topojson'))
print('Objects:', list(d['objects'].keys()))
first = list(d['objects'].values())[0]['geometries'][0]
print('Properties:', list(first['properties'].keys()))
print('Sample STATE_NAME:', first['properties'].get('STATE_NAME', 'NOT FOUND'))
"
```

**Expected:** feature name varies by file. The state name property is typically `STATE_NAME`. The choropleth lookup joins `properties.STATE_NAME` (topojson) → `state_name` (state_summary.csv).

**Projection for Australia:** `"mercator"` with static defaults `"center": [134, -26], "scale": 850`. For interactive zoom, replace with `{"expr": "zoom_level"}` and `{"expr": "center_to"}` params.

---

## Chart Plan (12 Charts — 6 Maps + 6 Non-Maps)

### MAP IDIOMS SUMMARY (for HD marking)
| Chart | Map Type | Custom Element |
|-------|----------|----------------|
| Chart 3 | State choropleth | Interactive zoom + state focus dropdown (Week 10) |
| Chart 4 | Capital city symbol map | Multi-layer: base + circles + labels + city filter |
| Chart 5 | State choropleth — diverging | Non-standard diverging color (blue/red) on geographic map |
| Chart 6 | Small multiples map | `repeat` operator with 3 choropleth views (Week 10) |
| Chart 9 | State choropleth — supply | Different metric on same geography, tooltip comparison |
| Chart 10 | LGA proportional symbol map | Circles sized by growth rate, state color coding |

---

### Section 1 — The National Picture

**Chart 1: National Approvals Trend 1983–2026** ★ NON-MAP CUSTOM
- File: `js/chart1_national_trend.vg.json`
- Data: `data/approvals_national_monthly.csv`
- Type: `vconcat` overview+detail (Week 10)
  - Top (height 280): Layered dual lines — houses (blue) + non_houses (orange). x-scale responds to brush: `"scale": {"domain": {"param": "brush"}}`
  - Bottom (height 60): Area navigator with brush param: `{"name": "brush", "select": {"type": "interval", "encodings": ["x"]}}`
  - Annotation layer on top chart: 4 vertical rules + text labels (GFC, Mining Boom, COVID, Rate Hikes)
- Tooltip: date, houses (format ","), non_houses (format ","), total (format ",")
- Title: "Australia's Building Approvals: 40 Years of Booms and Busts (1983–2026)"

**Chart 2: Are We Building Enough?** NON-MAP
- File: `js/chart2_rolling_total.vg.json`
- Data: `data/approvals_national_monthly.csv`
- Type: Bar + rule layer
- Transform: `{"window": [{"op": "sum", "field": "total", "as": "rolling_12m"}], "frame": [-11, 0]}`
- Layer 1: Bar — x=date, y=rolling_12m, color=#1565C0
- Layer 2: Rule at y=240000, color=red, strokeDash=[6,3]
- Layer 3: Text label "National target: 240,000/year" at rule
- Title: "Are We Building Enough? (12-Month Rolling Total Approvals)"

---

### Section 2 — Where Is Australia Building? (All Maps)

**Chart 3: State Building Approvals Map** ✅ MAP — ZOOMABLE CUSTOM
- File: `js/chart3_state_choropleth.vg.json`
- Data: `topojson/aus-states.topojson` + lookup `data/state_summary.csv`
- Width: 800, Height: 450
- Params (Week 10 pattern):
  ```json
  "params": [
    {"name": "zoom_level", "value": 3000,
     "bind": {"input": "range", "min": 1500, "max": 6000, "step": 100, "name": "Zoom: "}},
    {"name": "center_to", "value": [134, -26],
     "bind": {"input": "select",
              "options": [[134,-26],[151,-33],[144,-37],[153,-27],[116,-32],[138,-35]],
              "labels": ["All Australia","New South Wales","Victoria","Queensland","Western Australia","South Australia"],
              "name": "Focus State: "}}
  ],
  "projection": {"type": "mercator", "scale": {"expr": "zoom_level"}, "center": {"expr": "center_to"}}
  ```
- Lookup: `properties.STATE_NAME` → `state_name` → fields: `["approvals_fy2425","state","pop_2025"]`
- Color: `approvals_fy2425`, scheme `"blues"`, legend format ".2s"
- Tooltip: state_name, approvals_fy2425 (format ","), pop_2025 (format ",")
- Title: "Building Approvals by State/Territory (FY 2024–25)"

**Chart 4: Capital City Activity Bubble Map** ✅ MAP — SYMBOL
- File: `js/chart4_bubble_map.vg.json`
- Data: `data/gccsa_summary.csv` (with lat/lon)
- Width: 800, Height: 450
- 3 layers:
  - Layer 1: `topojson/aus-states.topojson` base — fill=#e8e8e8, stroke=white
  - Layer 2: Circles at lat/lon — size=total (range [50,2000]), color=approvals_per_1000 (scheme "orangered"), opacity=0.8
  - Layer 3: Text labels — city_name, dy=-14, fontSize=11
- Param: city_selection dropdown (Show All + 8 cities) — opacity condition on Layer 2+3
- Tooltip: city_name, total (format ","), approvals_per_1000 (.2f), pop_growth_pct (.2f + "%")
- Title: "Capital City Building Activity (FY 2024–25)"

**Chart 5: Where Is Growth Accelerating?** ✅ MAP — DIVERGING COLOR CUSTOM
- File: `js/chart5_yoy_choropleth.vg.json`
- Data: `topojson/aus-states.topojson` + lookup `data/state_summary.csv`
- Width: 800, Height: 450
- Static projection: `{"type": "mercator", "center": [134, -26], "scale": 850}`
- Lookup: `properties.STATE_NAME` → `state_name` → fields: `["yoy_change_pct","state_name","approvals_fy2425","approvals_fy2324"]`
- Color: `yoy_change_pct`, type quantitative, scale with:
  ```json
  "scale": {"domainMid": 0, "scheme": "blueorange", "reverse": true}
  ```
  (blue = growth, orange/red = decline — custom diverging on a map)
- Tooltip: state_name, yoy_change_pct (.1f + "% change"), approvals_fy2425 (format ","), approvals_fy2324 (format ",")
- Title: "Year-on-Year Change in Building Approvals by State (FY23–24 vs FY24–25)"
- **HD justification:** Diverging colour scale on a geographic map — non-standard idiom requiring knowledge of domainMid

**Chart 6: State Housing Metrics at a Glance** ✅ MAP — SMALL MULTIPLES CUSTOM
- File: `js/chart6_small_multiples_map.vg.json`
- Data: `topojson/aus-states.topojson` + lookup `data/state_summary.csv`
- Type: `repeat` operator (Week 10 pattern) — 3 choropleth maps side by side
  ```json
  {
    "repeat": ["approvals_fy2425", "approvals_per_1000", "yoy_change_pct"],
    "columns": 3,
    "spec": {
      "width": 280, "height": 200,
      "projection": {"type": "mercator", "center": [134, -26], "scale": 280},
      "layer": [
        { "data": {"url": "../topojson/aus-states.topojson", ...},
          "transform": [{"lookup": "properties.STATE_NAME", "from": {..., "fields": ["approvals_fy2425","approvals_per_1000","yoy_change_pct","state_name"]}}],
          "mark": {"type": "geoshape"},
          "encoding": {
            "color": {"field": {"repeat": "repeat"}, "type": "quantitative", "scale": {"scheme": "blues"}},
            "tooltip": [{"field": "state_name"}, {"field": {"repeat": "repeat"}, "format": ".2f"}]
          }
        }
      ]
    }
  }
  ```
- Titles for each view: "Total Approvals", "Per 1,000 People", "YoY Change %"
- **HD justification:** `repeat` operator from Week 10 — generates 3 coordinated map views with one spec

---

### Section 3 — What Type of Homes?

**Chart 7: Houses vs Apartments — National Trend** ★ NON-MAP CUSTOM
- File: `js/chart7_stacked_area.vg.json`
- Data: `data/approvals_national_monthly.csv`
- Type: `vconcat` overview+detail stacked area (Week 10)
  - Top (height 260): Stacked area using fold transform, x responds to `time_brush` param
  - Bottom (height 60): Total area navigator with `time_brush` brush param
- Transform: `{"fold": ["houses", "non_houses"], "as": ["type", "approvals"]}`
- Color: type — houses="#1976D2" (Houses), non_houses="#FF7043" (Apartments/Other)
- Title: "Composition of Approvals: Houses vs Apartments (1983–2026)"

**Chart 8: Housing Mix by Capital City** NON-MAP
- File: `js/chart8_100pct_bar.vg.json`
- Data: `data/gccsa_summary.csv`
- Type: Normalized 100% horizontal stacked bar
- Transform: `{"fold": ["houses", "non_houses"], "as": ["type", "count"]}`
- y=city_name (sort by house_pct desc), x=count (stack normalize, format "%")
- Color: houses="#1976D2", non_houses="#FF7043"
- Title: "Housing Mix by Capital City: Houses vs Apartments"

---

### Section 4 — Is Supply Meeting Demand?

**Chart 9: Building Supply Per Person** ✅ MAP — SUPPLY ADEQUACY
- File: `js/chart9_supply_choropleth.vg.json`
- Data: `topojson/aus-states.topojson` + lookup `data/state_summary.csv`
- Width: 800, Height: 450
- Static projection: `{"type": "mercator", "center": [134, -26], "scale": 850}`
- Lookup: `properties.STATE_NAME` → `state_name` → fields: `["approvals_per_1000","pop_growth_pct","state_name","pop_2025"]`
- Color: `approvals_per_1000`, scheme `"greens"`, domain [2, 10]
- Add a calculate transform: `"calculate": "datum.approvals_per_1000 >= 5.8 ? 'Above average' : 'Below average'", "as": "supply_status"`
- Tooltip: state_name, approvals_per_1000 (.2f), pop_growth_pct (.2f + "%"), supply_status
- Title: "Building Supply Rate by State — Approvals per 1,000 People (FY 2024–25)"

**Chart 10: Where Are Australia's Growth Hotspots?** ✅ MAP — LGA SYMBOL CUSTOM
- File: `js/chart10_lga_symbol_map.vg.json`
- Data: `data/population_lga_2024_25.csv` filtered to top 20 by change_pct (with lat/lon added)
- Width: 800, Height: 500
- 2 layers:
  - Layer 1: `topojson/aus-states.topojson` base — fill=#e8e8e8, stroke=white
  - Layer 2: Circles at lat/lon
    - size=change_pct (scale domain [0, 10], range [50, 1200])
    - color=state_abbr (nominal, scheme "category10")
    - opacity=0.75
    - tooltip: lga_name, state_abbr, change_pct (.1f + "%"), erp_2025 (format ",")
- Transform: window rank (sort change_pct desc) → filter rank <= 20
- Static projection: `{"type": "mercator", "center": [134, -26], "scale": 850}`
- Title: "Australia's 20 Fastest-Growing Local Government Areas (FY 2024–25)"
- **HD justification:** Non-standard idiom — sized circles on geographic base using LGA-level data; combines two datasets visually

**Chart 11: Population Growth vs Building Supply** ★ NON-MAP CUSTOM
- File: `js/chart11_scatter.vg.json`
- Data: `data/state_summary.csv` (pre-joined from 2 ABS sources)
- Type: Interactive scatter — circles + text labels + state dropdown (Week 9)
- x=pop_growth_pct (title "Population Growth Rate %"), y=approvals_per_1000 (title "Approvals per 1,000 People")
- Layer 1: circle, size=pop_2025 (threshold scale: [500k,1M,3M,6M] → [50,150,300,500,800]), color=state (nominal), opacity condition on state_highlight param
- Layer 2: text label showing state abbreviation, always visible, dy=-12
- Layer 3: rule at x=1.8 (median pop growth) — gray, dashed
- Layer 4: rule at y=5.8 (national avg supply) — gray, dashed
- Param: `state_highlight` dropdown (Show All + 8 states)
- Tooltip: state_name, pop_growth_pct (.2f), approvals_per_1000 (.2f), pop_2025 (format ",")
- Title: "Which States Are Keeping Up? Population Growth vs Building Supply"
- **HD justification:** Two-source join, quadrant annotation rules, custom size encoding

---

### Section 5 — The Supply-Demand Gap

**Chart 12: Supply vs Demand by Capital City** ★ NON-MAP CUSTOM
- File: `js/chart12_connected_dot.vg.json`
- Data: `data/gccsa_summary.csv`
- Type: Connected dot plot (custom idiom — not standard in Vega-Lite)
- Transform: `{"fold": ["approvals_per_1000", "pop_growth_pct"], "as": ["metric", "value"]}`
- Layer 1: Line mark — `"detail": {"field": "city_name"}`, color=#ccc, connecting the two dots
- Layer 2: Point mark — size=150, color by metric:
  - approvals_per_1000 → #1976D2 (Supply)
  - pop_growth_pct → #E53935 (Demand)
- y=city_name (sort by approvals_per_1000 desc), x=value
- Legend: rename field labels — "approvals_per_1000"="Supply (approvals/1k)", "pop_growth_pct"="Demand (pop growth %)"
- Tooltip: city_name, metric, value (.2f)
- Title: "Supply vs Demand: Building Rate vs Population Growth by Capital City"
- **HD justification:** Custom-built idiom — connected dot plot requires combining line + point layers with fold transform

---

## Rubric Alignment

| Criterion | Weight | Target | Evidence |
|-----------|--------|--------|----------|
| Idioms & Complexity | 10% | **HD** | 6 custom map idioms (diverging choropleth, small multiples, LGA symbol, zoomable, bubble, supply map) + 4 custom non-map idioms |
| Map included | unlocks 10% | ✅ | 6 maps = 50% of all charts |
| Layout, colour, figure-ground | 4% | **HD** | pure.css symmetric grid, consistent blue/orange palette, white 1000px page |
| Typography | 2% | **HD** | Open Sans throughout, config block on all charts, clear hierarchy |
| Storytelling, annotations, grammar | 5% | **HD** | 5 sections with narrative arc, chart annotations, real text (not Lorem Ipsum) |
| Domain/who/what/why/how | 2% | **HD** | ABS sources cited, Australian audience, Munzer framework in Moodle text |
| 2 data sources | — | ✅ | ABS Building Approvals (8731.0) + ABS Regional Population (3218.0) |
| Data size | — | ✅ | ~373 KB total |

---

## Vega-Lite Data Size Check

| File | Size | Status |
|------|------|--------|
| approvals_national_monthly.csv | ~25 KB | ✅ |
| approvals_states_monthly.csv | ~120 KB | ✅ |
| approvals_gccsa_monthly.csv | ~80 KB | ✅ |
| population_lga_2024_25.csv | ~35 KB | ✅ |
| population_gccsa_2001_2025.csv | ~12 KB | ✅ |
| state_summary.csv | <1 KB | ✅ |
| gccsa_summary.csv | <1 KB | ✅ |
| aus-states.topojson | ~100 KB | ✅ |
| **TOTAL** | **~373 KB** | **✅ Well under limit** |
