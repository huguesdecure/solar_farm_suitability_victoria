# Solar Farm Site Suitability Analysis — Victoria, Australia

**Hugues Decure | Portfolio Project #1 | April 2026**

![Solar Farm Suitability Map](outputs/vic_solar_farm_suitability.png)

---

## Overview

This project simulates Stage 1 desktop feasibility screening for large-scale solar development in Victoria, Australia — the first step in a developer's site selection process prior to environmental assessment and planning permit applications.

A Multi-Criteria Decision Analysis (MCDA) was applied using five weighted spatial criteria and three independent hard-exclusion masks to identify land parcels with the highest potential for utility-scale solar farm development. The output is a discrete suitability raster scored 0–5 covering Victoria at 250m resolution, validated against 21 operational utility-scale solar farms drawn from the Clean Energy Regulator registry.

**CRS:** GDA2020 MGA Zone 55 (EPSG:7855) — current Australian standard for Victorian spatial work.
**Resolution:** 250m
**Tools:** QGIS 3.40 LTS, GDAL, SAGA GIS

---

## Methodology

### Weighted Overlay

Each criterion was reclassified to a 1–5 suitability score, rasterised to 250m resolution, and combined via the QGIS Raster Calculator using the weighted overlay formula below. Three independent binary exclusion masks were then applied, forcing any cell flagged by any mask to score 0 in the final output.

```
Suitability = (GHI × 0.35) + (Slope × 0.25) + (Grid distance × 0.20)
            + (Zoning × 0.10) + (Road distance × 0.10)

Final = Suitability × CAPAD_mask × Urban_mask × Conservation_mask
```

### Criteria, Weights and Scoring

| Criterion | Weight | Best score (5) | Excluded (0) | Rationale |
|-----------|--------|----------------|--------------|-----------|
| Solar irradiance (GHI) | 35% | >5.0 kWh/m²/day | N/A | Primary revenue driver |
| Slope (from DEM) | 25% | 0–3° | >10° | Earthworks are the largest construction cost variable |
| Distance to transmission | 20% | <5km | >30km | Grid connection can add $1–5M per project |
| Land zoning | 10% | Farming Zone (FZ) | Residential, urban, parks | Binary suitability |
| Distance to roads | 10% | <2km to sealed road | >20km | Construction logistics |
| **Hard exclusion masks** | — | — | — | — |
| CAPAD protected areas | binary | — | All | Federal conservation legislation |
| Urban planning zones | binary | — | All listed below | Solar incompatible with residential/commercial |
| Conservation zones | binary | — | RCZ, PCRZ, PPRZ, GWAZ | Environmental significance overlays |

### Reclassification Tables

**Slope (degrees)**

| Range | Score |
|---|---|
| 0–3° | 5 |
| 3–5° | 4 |
| 5–7° | 3 |
| 7–10° | 2 |
| 10–90° | 1 |

**Solar Irradiance — GHI (kWh/m²/day)**

| Range | Score |
|---|---|
| 0–3.5 | 1 |
| 3.5–4.0 | 2 |
| 4.0–4.5 | 3 |
| 4.5–5.0 | 4 |
| 5.0–6.0 | 5 |

**Distance to Transmission Lines (metres)**

| Range | Score |
|---|---|
| 0–5,000 | 5 |
| 5,000–10,000 | 4 |
| 10,000–20,000 | 3 |
| 20,000–30,000 | 2 |
| >30,000 | 1 |

**Distance to Roads — major sealed (metres)**

| Range | Score |
|---|---|
| 0–2,000 | 5 |
| 2,000–5,000 | 4 |
| 5,000–10,000 | 3 |
| 10,000–20,000 | 2 |
| >20,000 | 1 |

OSM road data was filtered to motorway, trunk, primary, secondary, and tertiary classes — the network usable by heavy construction vehicles.

**Land Zoning (CASE expression on zone code prefix)**

| Zone | Score | Notes |
|---|---|---|
| FZ — Farming Zone | 5 | Large rural parcels, compatible with utility-scale solar |
| RAZ — Rural Activity Zone | 4 | Mixed agricultural use |
| GWZ — Green Wedge Zone | 3 | Peri-urban rural |
| GWAZ — Green Wedge A Zone | 0 | Conservation overlay (excluded) |
| RCZ, RLZ, PCRZ, PPRZ | 0 | Conservation, rural living, public parks |
| GRZ, NRZ, RGZ, LDRZ, MUZ | 0 | All residential zones |
| C1Z, C2Z, IN1Z, IN2Z, IN3Z | 0 | Commercial and industrial |
| ACZ, CCZ, UGZ, PUZ, RDZ | 0 | Activity centres, capital city, urban growth, public use, road |
| TZ, PZ, DZ, SUZ | 0 | Township, port, docklands, special use |

The conservative zoning treatment reflects real-world feasibility practice: utility-scale solar developers require contiguous parcels with workable lease arrangements, which is structurally incompatible with residential subdivisions, conservation overlays, and public-use land regardless of nominal hectarage.

### Hard Exclusion Masks

Three independent binary exclusion masks were built and multiplied against the weighted suitability raster:

1. **CAPAD mask** — Collaborative Australian Protected Areas Database (DCCEEW). Federal conservation legislation. National parks, nature reserves, conservation parks.
2. **Urban zoning mask** — All residential, commercial, industrial, public-use, and urban-growth planning zones from data.vic.gov.au, derived from the Victorian Planning Provisions under the *Planning and Environment Act 1987*.
3. **Conservation zoning mask** — Rural Conservation Zone, Public Conservation and Resource Zone, Public Park and Recreation Zone, Green Wedge A Zone. These zones carry environmental significance overlays under Victorian planning law and are typically incompatible with utility-scale solar approval.

Using three separate masks rather than a single combined exclusion improves methodology transparency and allows the contribution of each regulatory framework to be audited independently.

---

## Data Sources

| Layer | Source | URL | Format | Accessed |
|-------|--------|-----|--------|----------|
| Solar irradiance (GHI) | Global Solar Atlas | globalsolaratlas.info/download | GeoTIFF | 12 Apr 2026 |
| DEM / Elevation (10m) | ELVIS / Geoscience Australia | elevation.fsdf.org.au | Esri Grid (ADF) | 12 Apr 2026 |
| Transmission lines | AREMI | aremi.v2.terria.io | GeoJSON / SHP | 12 Apr 2026 |
| Protected areas | CAPAD / DCCEEW | dcceew.gov.au/parks-heritage/parks/capad | Shapefile | 12 Apr 2026 |
| Roads | Geofabrik / OSM | download.geofabrik.de/australia-oceania | Shapefile | 12 Apr 2026 |
| Victoria boundary | ABS ASGS Edition 3 | abs.gov.au | Shapefile | 12 Apr 2026 |
| REZ boundaries | VicGrid | energy.vic.gov.au/renewable-energy/renewable-energy-zones | Shapefile / WMS | 12 Apr 2026 |
| Land zoning | data.vic.gov.au | data.vic.gov.au — Vicmap Planning | Shapefile | 13 Apr 2026 |
| Solar farm registry | Clean Energy Regulator | cer.gov.au/markets/reports-and-data | CSV | 13 Apr 2026 |
| Hydrography | data.vic.gov.au | Vicmap Hydro — water polygons | Shapefile | 25 Apr 2026 |

---

## Key Findings

- **North-West Victoria (Mildura → Kerang)** and the Murray River corridor consistently score 4–5 (High to Very High), driven by GHI >5.0 kWh/m²/day, flat terrain, large Farming Zone parcels, and proximity to existing transmission infrastructure.
- **21 operational utility-scale solar farms** (>10 MW) from the CER registry concentrate in the same north-west postcodes (3490, 3494, 3549). The MCDA output independently aligns with real investment decisions made by solar developers — a strong validation signal.
- **5 committed projects and 11 probable projects** from the CER pipeline cluster in high-suitability zones identified by the analysis, including Corop (440 MW), Nowingi (300 MW), and West Mokoan (300 MW).
- **The Central North REZ** (Bendigo to Shepparton), planned for declaration in 2026, overlaps high-suitability zones identified in this analysis — consistent with VicGrid's own REZ planning rationale. Several solar farms are already under development in that corridor.
- **Gippsland and the Central Highlands** score moderate (2–3), reflecting steeper terrain and lower GHI despite reasonable grid access.
- **Melbourne metropolitan area, Geelong, Ballarat, Bendigo, Shepparton** are uniformly excluded by the urban zoning mask, as are Alpine National Park, Wilsons Promontory, the Grampians, and other CAPAD-listed protected areas.

---

## Limitations

This is a Stage 1 desktop feasibility screen. It identifies where solar development is *physically viable* at a broad scale — it does not replace the detailed assessments required before any project proceeds to planning approval or financing.

**Grid capacity not modelled.** Distance to transmission is used as a proxy for connection cost, but does not account for available network capacity. Transmission lines in high-suitability zones — particularly in north-west Victoria — may already be constrained by existing connection queues, currently one of the most significant blockers for new Victorian solar projects.

**Cadastral parcels not modelled.** The analysis operates on planning zones, not land titles. High-suitability cells may be fragmented across multiple landowners, requiring negotiation that spatial data cannot predict. Utility-scale development typically requires 200–500+ contiguous hectares under a workable lease arrangement.

**State forests partially captured.** State forests under the *Forests Act 1958* are excluded indirectly via the PCRZ planning zone score but not via a dedicated VicMap public land tenure layer. A production analysis would integrate VicMap public land tenure as a fourth hard-exclusion layer.

**Flood and bushfire risk not modelled.** Parts of the Murray River corridor identified as highly suitable are subject to periodic flooding affecting insurance, financing terms, and long-term panel degradation. Bushfire-prone areas under the Victorian planning scheme are not separately filtered.

**Logistics constraints not modelled.** Utility-scale construction requires laydown areas, sealed roads capable of handling continuous B-double traffic over a 12–18 month construction period, and proximity to worker accommodation. Remote high-suitability sites may carry construction cost penalties not reflected in the road distance criterion. Water availability for panel washing and construction is also not included — a practical constraint for remote inland sites.

**DEM resolution.** The 10m ELVIS DEM was resampled to 250m to match the GHI raster extent. Slope analysis is therefore indicative at this scale and would require higher-resolution terrain data for site-specific assessment.

**Solar project locations.** Project markers shown on the map are approximate, derived from CER postcode data and public project records rather than surveyed coordinates. Used as a validation reference, not as authoritative siting.

**Currency.** The analysis reflects data accessed in April 2026. Planning zones, REZ boundaries, and transmission infrastructure are subject to ongoing change as Victoria's energy transition accelerates. The Central North REZ has not yet been formally declared and is therefore absent from the REZ boundary layer.

---

## Next Steps

- Integrate flood and bushfire risk layers as additional hard-exclusion criteria
- Apply Python / GeoPandas automation to reproduce the weighted overlay pipeline programmatically — improving reproducibility and enabling sensitivity analysis on weights
- Build an interactive Leaflet / Mapbox web map allowing stakeholders to explore suitability scores and project locations dynamically
- Extend the analysis to include grid capacity constraints from AEMO's ISP transmission augmentation schedule
- Add a cadastral parcel size filter — utility-scale solar requires minimum ~200ha contiguous parcels under workable lease arrangements

---

## Domain Context

### Victorian Renewable Energy Zones (REZs)

VicGrid manages five declared REZs in Victoria. This analysis overlays REZ boundaries to contextualise high-suitability zones within the state's grid planning framework:

| REZ | Primary resource | Notes |
|-----|------------------|-------|
| North West (Kerang to Mildura) | Solar | Highest suitability in this analysis |
| Western Victoria (Wimmera Mallee) | Solar + wind | Strong solar, grid constraints |
| Central Highlands | Wind | Lower solar suitability, steeper terrain |
| Gippsland | Wind + solar | Moderate suitability |
| Gippsland Shoreline | Offshore wind | Not applicable to this analysis |
| Central North (Bendigo to Shepparton) | Solar | Planned 2026 — overlaps high-suitability zones |

### Key Bodies Referenced

| Body | Role |
|------|------|
| AEMO | Manages the national electricity grid; publishes the Integrated System Plan (ISP), 20-year roadmap for transmission and generation |
| ARENA | Funds renewable energy R&D; operates AREMI spatial mapping platform |
| Clean Energy Regulator | Administers the Renewable Energy Target; publishes the solar/wind project registry used for validation |
| VicGrid | Manages Victoria's REZ programme; coordinates grid planning for the energy transition |
| CAPAD / DCCEEW | Collaborative Australian Protected Areas Database — definitive dataset for conservation area exclusions |

### Solar Development Stages

This project simulates **Stage 1**:

1. **Desktop feasibility screening** ← this project
2. Site selection and landowner negotiation
3. Environmental Impact Assessment (EIA)
4. Planning permit (local council + Victorian Government)
5. Grid connection application (AEMO / network operator)
6. Construction and commissioning

Exclusion zones align with CAPAD 2022 protected areas and Victorian planning zones under the *Planning and Environment Act 1987*, consistent with industry-standard desktop feasibility screening methodology.

---

## Repository Structure

```
─ outputs/
│   ├── vic_solar_farm_suitability.png # final map (PNG)
│   └── vic_solar_farm_suitability.pdf # final map (A3 PDF)
├── notes/             # working log — datasets, decisions, processing
└── data/                              # processed rasters (large files excluded via .gitignore)
└──README
```

Raw datasets are not included in the repository — see the Data Sources table above for download links.

---

## Tools

| Tool | Purpose |
|------|---------|
| QGIS 3.40 LTS | Desktop GIS — all analysis, styling, and print layout |
| GDAL | Raster processing, reprojection, clipping |
| SAGA GIS | Slope analysis via QGIS Processing Toolbox |
| Python (QGIS Console) | NoData diagnostics |

**CRS:** GDA2020 MGA Zone 55 (EPSG:7855)
**Analysis resolution:** 250m
**OS:** Windows

---

*Hugues Decure | April 2026 | [github.com/hugues-decure](https://github.com/huguesdecure)*
