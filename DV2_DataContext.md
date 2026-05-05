# FIT2179 DV2 — Data Context & Chart Plan
**Topic:** Australian Residential Building Approvals — Housing Supply vs Demand  
**Due:** 29 May 2026 | **Author:** Trung Le

---

## Clean CSV Files (ready for Vega-Lite)

| File | Rows | Key Columns | Used For |
|------|------|-------------|----------|
| `approvals_national_monthly.csv` | 513 | date, houses, non_houses, total, total_sa | Charts 1, 4 |
| `approvals_states_monthly.csv` | 4,104 | date, state, state_name, approvals | Charts 2, 8 |
| `approvals_gccsa_monthly.csv` | 2,376 | date, city, city_name, total, houses, non_houses | Charts 3, 5, 9 |
| `population_lga_2024_25.csv` | 546 | lga_code, lga_name, erp_2025, change_pct, area_km2, pop_density, state | Chart 10 |
| `population_gccsa_2001_2025.csv` | 400 | year, gccsa_name, population | Chart 11 |
| `state_summary.csv` | 8 | state, approvals_fy2425, pop_2025, pop_growth_pct, approvals_per_1000, yoy_change_pct | Charts 6, 7 |
| `gccsa_summary.csv` | 8 | city, total, house_pct, pop_2025, pop_growth_pct, approvals_per_1000 | Charts 5, 7 |

---

## Data Sources

### Source 1: ABS Building Approvals (8731.0)
- **Page:** https://www.abs.gov.au/statistics/industry/building-and-construction/building-approvals-australia/latest-release
- **Reference period:** March 2026 (released 4 May 2026)
- **Files used:** Table 06 (National), Table 07 (States), Table 10 (GCCSA)
- **Date range:** July 1983 – March 2026 (Tables 06/07); July 2001 – March 2026 (Table 10)
- **Key facts:** Zero nulls, monthly frequency, original + seasonally adjusted series available

### Source 2: ABS Regional Population (3218.0)
- **Page:** https://www.abs.gov.au/statistics/people/population/regional-population/latest-release
- **Reference period:** 2024–25 financial year (released 31 March 2026)
- **Files used:** 32180DS0002 (LGA), 32180DS0003 Table 4 (GCCSA)
- **Coverage:** 546 LGAs across 7 states/territories; 16 GCCSAs 2001–2025
- **Key facts:** Zero nulls in growth column; LGA growth range –3.0% to +9.3%

---

## Key Findings from Data Profiling

### Building Approvals
- **National total (Mar 2026):** 17,780 dwellings/month
- **Peak periods:** Pre-GFC (~2003), post-COVID stimulus (~2021), current recovery
- **State leader FY2024-25:** Victoria (56,581), then NSW (49,515), QLD (38,229)
- **WA fastest growing:** +31.4% YoY; ACT worst: –37.8% YoY
- **Melbourne dominates capitals:** 46,530 approvals FY2024-25, vs Sydney 33,573
- **Perth house-focused:** 79.6% houses; Sydney apartment-focused: only 38.7% houses

### Population
- **Fastest growing capital:** Perth (+2.43%), Brisbane (+2.10%)
- **Slowest:** Hobart (+0.21%), Sydney (+1.35%)
- **Best supply ratio:** Melbourne (8.56 approvals/1,000 people), Adelaide (7.94)
- **Worst supply ratio:** Darwin (2.29), Hobart (3.47), NT (2.66)
- **Top LGA by growth:** Darwin Waterfront Precinct +9.3%, Melton VIC +5.8%

---

## Chart Plan (12 Charts)

### Section 1 — The National Picture
**Chart 1: National Approvals Trend (1983–2026)**
- Type: Annotated line chart (dual lines: houses + non-houses)
- File: `approvals_national_monthly.csv`
- Fields: x=date, y=houses + y=non_houses (layered), optional SA overlay
- Annotations: GFC (2008–09), Mining boom (2012–13), COVID dip (2020), Rate hikes (2022–23)
- *Complexity:* Custom annotation layer — qualifies as custom-built idiom

**Chart 2: Are We Building Enough? (12-month rolling total)**
- Type: Bar chart with target line
- File: `approvals_national_monthly.csv` (rolling 12-month window via Vega-Lite transform)
- Fields: x=date, y=rolling_total, reference line at ~200,000/year
- *Complexity:* Derived rolling aggregate in Vega-Lite

### Section 2 — Where Is Australia Building?
**Chart 3: State Choropleth Map**
- Type: Geographic choropleth
- File: `state_summary.csv` + ABS/TopoJSON Australian states boundaries
- Fields: state → color=approvals_fy2425
- TopoJSON: aus-states.topojson (from ABS or naturalearth)
- *Requirement:* ✅ Map #1 (state level)

**Chart 4: Capital Cities Bubble Map**
- Type: Proportional symbol map (circles on city locations)
- File: `gccsa_summary.csv` + hardcoded lat/lon for 8 capital cities
- Fields: lat/lon, size=total, color=approvals_per_1000
- *Requirement:* ✅ Map #2 (GCCSA level)

**Chart 5: State Year-on-Year Change**
- Type: Diverging bar chart (positive = growing, negative = declining)
- File: `state_summary.csv`
- Fields: x=yoy_change_pct, y=state, color=positive/negative

### Section 3 — What Type of Homes?
**Chart 6: Houses vs Apartments — National Trend**
- Type: Stacked area chart
- File: `approvals_national_monthly.csv`
- Fields: x=date, y=houses + y=non_houses stacked
- Show shift from houses to apartments over decades

**Chart 7: House vs Apartment Mix by Capital City**
- Type: Stacked/normalized bar chart (100%)
- File: `gccsa_summary.csv`
- Fields: y=city, x=house_pct + non_house_pct (normalized to 100%)
- Perth/Darwin = mostly houses; Sydney/Melbourne = mixed/apartments

### Section 4 — Is Supply Meeting Demand?
**Chart 8: Approvals per 1,000 People (States)**
- Type: Bar chart sorted descending
- File: `state_summary.csv`
- Fields: x=state, y=approvals_per_1000, color=above/below national average

**Chart 9: Population Growth vs Approvals Rate (Scatter)**
- Type: Scatter plot (custom — requires join of two sources)
- File: `state_summary.csv`
- Fields: x=pop_growth_pct, y=approvals_per_1000, size=pop_2025, label=state
- *Complexity:* Two-source join, insight idiom — qualifies as custom-built

**Chart 10: Capital Cities — Supply vs Demand Race**
- Type: Connected dot plot (two dots: Melbourne/Sydney, joined by line)
- File: `gccsa_summary.csv`
- Fields: x=approvals_per_1000 vs x=pop_growth_pct, y=city
- *Complexity:* Custom-built idiom (not standard in Vega-Lite without customisation)

### Section 5 — The Long View
**Chart 11: Capital City Population Growth (2001–2025)**
- Type: Multi-line chart
- File: `population_gccsa_2001_2025.csv`
- Fields: x=year, y=population, color=gccsa_name
- Filter to capital cities only (exclude "Rest of..." rows)

**Chart 12: Top Growing LGAs (Australia-wide)**
- Type: Horizontal bar chart (top 20)
- File: `population_lga_2024_25.csv`
- Fields: y=lga_name, x=change_pct, color=state_abbr
- Sorted by change_pct descending

---

## TopoJSON Map Files Needed

| Map | Source | Description |
|-----|--------|-------------|
| Australian states | ABS / Natural Earth | 8 states/territories |
| GCCSA boundaries | ABS | For bubble map base layer (can use simplified version) |

Recommended: Use `https://raw.githubusercontent.com/rowanhogan/australian-states/master/states.min.geojson`
for states. For GCCSA, use proportional symbol map on blank background with lat/lon.

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
| **TOTAL** | **~273 KB** | **✅ Well under 2 MB limit** |

---

## Rubric Alignment

| Criterion | Target | How We Meet It |
|-----------|--------|----------------|
| Idioms & Complexity (10%) | HD | Charts 1, 9, 10 are custom-built; 12 charts total |
| Map (required) | ✅ | 2 maps: state choropleth + capital city bubble |
| 2 data sources | ✅ | ABS Building Approvals + ABS Regional Population |
| Storytelling | HD | 5 clear sections with narrative arc |
| Interactivity | Good | Tooltips, hover, filter by state/city |
| Data-ink ratio | HD | Clean minimal design, no chartjunk |
