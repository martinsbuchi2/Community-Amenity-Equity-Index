# Community Amenity Equity Index — Edmonton
> **Project date:** May 12, 2026  
> **CRS:** EPSG:3776 (NAD83(CSRS) / Alberta 10-TM Forest)  
> **QGIS project file:** `Community_Amenity_Equity_Index.qgz`

---

## 1. Purpose

This project scores each of Edmonton's **400 administrative neighbourhoods** on a
composite **Community Amenity Equity Index (0 – 100)** that captures how well-served
each neighbourhood is by publicly accessible community resources and civic attractions
relative to its area.

The index is designed to answer the question:
> *Which neighbourhoods have the highest density of public amenities, and which are
> most underserved?*

Results support planning and prioritisation decisions around service delivery,
infrastructure investment, and community equity programmes.

---

## 2. Input Data

| Layer | File | Features | Geometry | Description |
|---|---|---|---|---|
| `edmonton_neighbourhoods` | `edmonton_neighbourhoods.gpkg` | 400 | Polygon | City of Edmonton administrative neighbourhood boundaries (OSM) |
| `community_resources` | `community_resources.gpkg` | 502 | Point | Publicly funded community programs and facilities (sports, recreation, social services) |
| `civic_attraction` | `civic_attraction.gpkg` | 56 | Point | Major civic attractions (museums, zoos, heritage sites, science centres) |
| `city_boundary` | `city_boundary.gpkg` | 1 | Polygon | Edmonton city boundary |

---

## 3. Methodology

### 3.1 Point-in-Polygon Counts

Both point layers were counted against the neighbourhood polygon layer using
**QGIS Processing: `native:countpointsinpolygon`**, producing two new integer
fields:

| Field | Description |
|---|---|
| `res_count` | Number of community resources falling within each neighbourhood |
| `civic_count` | Number of civic attractions falling within each neighbourhood |

### 3.2 Area Normalisation

Neighbourhood area was computed in square kilometres using the expression
`$area / 1000000` (CRS units are metres):

| Field | Expression |
|---|---|
| `area_km2` | `$area / 1000000` |
| `res_per_km2` | `res_count / area_km2` |
| `civic_per_km2` | `civic_count / area_km2` |

### 3.3 Min–Max Normalisation

Each density field was rescaled to **[0, 1]** using min–max normalisation to make
community resources and civic attractions comparable despite their very different
absolute ranges:

```
res_norm   = (res_per_km2   - min_res)   / (max_res   - min_res)
civic_norm = (civic_per_km2 - min_civic) / (max_civic - min_civic)
```

Observed ranges:  
- Community resource density: 0.00 – 91.37 resources/km²  
- Civic attraction density:   0.00 – 3.59 attractions/km²

### 3.4 Composite Index

The **Equity Index** is a weighted composite score scaled to 0 – 100:

```
equity_index = round( (0.70 × res_norm + 0.30 × civic_norm) × 100, 2 )
```

**Weighting rationale:**  
Community resources (70%) are weighted higher than civic attractions (30%) because
they represent day-to-day service delivery — recreation programmes, social services,
youth support — while civic attractions are fewer in number and skew towards the
city centre. This weighting reflects a service-equity rather than cultural-amenity
framing.

### 3.5 Classification

The output layer is visualised as a **5-class graduated choropleth** using
**Jenks Natural Breaks** classification on `equity_index`, with a sequential
teal colour ramp (light → dark = low → high index).

| Class | Equity Index Range | Interpretation |
|---|---|---|
| 1 | 0.00 – 3.45 | Very low amenity density |
| 2 | 3.45 – 10.09 | Low amenity density |
| 3 | 10.09 – 19.34 | Moderate amenity density |
| 4 | 19.34 – 38.27 | High amenity density |
| 5 | 38.27 – 70.00 | Very high amenity density |

---

## 4. Output Layer Schema

**File:** `community_amenity_equity_index.gpkg`  
**Layer:** `community_amenity_equity_index`

| Field | Type | Description |
|---|---|---|
| `fid` | Integer | Original feature ID |
| `name` | String | Neighbourhood name |
| `admin_level` | String | OSM admin level (10 = neighbourhood) |
| `res_count` | Integer | Community resources count |
| `civic_count` | Integer | Civic attractions count |
| `area_km2` | Float | Neighbourhood area in km² |
| `res_per_km2` | Float | Community resources per km² |
| `civic_per_km2` | Float | Civic attractions per km² |
| `res_norm` | Float | Min–max normalised resource density [0–1] |
| `civic_norm` | Float | Min–max normalised civic density [0–1] |
| `equity_index` | Float | Composite amenity equity score [0–100] |

---

## 5. Summary Statistics

| Metric | Value |
|---|---|
| Total neighbourhoods | 400 |
| Neighbourhoods with zero index (no amenities) | 304 |
| Mean equity index | 1.70 |
| Max equity index | 70.00 |
| Total community resources | 502 |
| Total civic attractions | 56 |

### Top 5 Neighbourhoods by Equity Index

| Neighbourhood | Resources | Civic | Area km² | Index |
|---|---|---|---|---|
| Cromdale | 33.0 | 0.0 | 0.36 | 70.00 |
| Abbottsfield | 10.0 | 1.0 | 0.42 | 38.27 |
| West Meadowlark Park | 1.0 | 4.0 | 1.11 | 30.69 |
| Downtown | 81.0 | 1.0 | 2.33 | 30.24 |
| Queen Alexandra | 5.0 | 3.0 | 1.24 | 23.35 |

### Bottom 5 Neighbourhoods by Equity Index (Excluding Zeros)

| Neighbourhood | Resources | Civic | Area km² | Index |
|---|---|---|---|---|
| Grandview Heights | 0.0 | 0.0 | 0.57 | 0.00 |
| Sweet Grass | 0.0 | 0.0 | 0.85 | 0.00 |
| Steinhauer | 0.0 | 0.0 | 0.87 | 0.00 |
| Ermineskin | 0.0 | 0.0 | 1.26 | 0.00 |
| Blue Quill Estates | 0.0 | 0.0 | 0.45 | 0.00 |

---

## 6. Project Files

```
Community Amenity Equity Index/
├── Community_Amenity_Equity_Index.qgz          ← QGIS project (open this)
├── community_amenity_equity_index.gpkg          ← Output: index layer (400 polygons)
├── edmonton_neighbourhoods.gpkg                 ← Input: neighbourhood boundaries
├── community_resources.gpkg                     ← Input: 502 community resource points
├── civic_attraction.gpkg                        ← Input: 56 civic attraction points
├── city_boundary.gpkg                           ← Input: Edmonton city boundary
└── README.md                                    ← This file
```

---

## 7. Reproduction Steps

To reproduce this analysis from scratch in QGIS:

1. Load `edmonton_neighbourhoods.gpkg`, `community_resources.gpkg`, and
   `civic_attraction.gpkg` into QGIS.
2. **Processing Toolbox → Count Points in Polygon**  
   Polygons = neighbourhoods, Points = community_resources, Field = `res_count`
3. **Count Points in Polygon** again on the result  
   Points = civic_attraction, Field = `civic_count`
4. **Field Calculator** — add `area_km2`: `$area / 1000000`
5. **Field Calculator** — add `res_per_km2`: `"res_count" / "area_km2"` (guard div/0)
6. **Field Calculator** — add `civic_per_km2`: `"civic_count" / "area_km2"` (guard div/0)
7. **Field Calculator** — add `res_norm` and `civic_norm` using min–max expressions
   (see Section 3.3 for the exact min/max values used)
8. **Field Calculator** — add `equity_index`:
   `round((0.7 * "res_norm" + 0.3 * "civic_norm") * 100, 2)`
9. Export to GeoPackage → apply 5-class Jenks graduated renderer on `equity_index`

---

## 8. Limitations & Next Steps

- **Population weighting:** The index is area-normalised, not population-normalised.
  Adding census population data per neighbourhood would produce a per-capita equity score.
- **Accessibility vs. presence:** The index counts facilities within boundaries only.
  A network-based accessibility analysis (walking/transit isochrones) would better
  capture real accessibility from residential areas.
- **Category weighting:** All community resource categories are treated equally.
  Sub-weighting by category (e.g., health vs. recreation vs. social services) would
  allow sector-specific equity assessments.
- **Temporal dimension:** The dataset reflects a single snapshot. Repeat analysis
  over multiple years would reveal gentrification or disinvestment trends.

---

*Generated with QGIS 3.x via Claude–QGIS MCP integration.*
