---
paths:
  - "Slides/**/*.tex"
  - "Quarto/**/*.qmd"
  - "python/**/*.py"
  - "notebooks/**/*.ipynb"
  - "Reports/**/*.tex"
---

# Project Knowledge Base: Kenya Rangeland Monitoring

<!-- Claude reads this before creating/modifying any project content. -->

## Sentinel-2 Band Reference

| Band | Name       | Wavelength | Resolution | Role in Project            |
|------|-----------|------------|------------|----------------------------|
| B2   | Blue       | 490 nm     | 10 m       | EVI background correction  |
| B3   | Green      | 560 nm     | 10 m       | NDWI (with NIR)            |
| B4   | Red        | 665 nm     | 10 m       | NDVI, EVI numerator        |
| B8   | NIR        | 842 nm     | 10 m       | NDVI, EVI (primary NIR)    |
| B8A  | NIR-narrow | 865 nm     | 20 m       | Secondary NIR (if needed)  |
| B11  | SWIR1      | 1610 nm    | 20 m       | BSI, moisture stress       |
| B12  | SWIR2      | 2190 nm    | 20 m       | BSI, bare soil detection   |
| SCL  | Scene Class| --         | 20 m       | Cloud/shadow mask (primary)|

**Collection:** `COPERNICUS/S2_SR_HARMONIZED` (surface reflectance, harmonized)
**Scale factor:** Values are in reflectance × 10000. Divide by 10000 before computing indices.

---

## Vegetation Index Registry

| Index | Full Name             | Formula                                              | Range     | Use Case                          |
|-------|-----------------------|------------------------------------------------------|-----------|-----------------------------------|
| NDVI  | Normalized Difference Vegetation Index | (B8 − B4) / (B8 + B4)            | [−1, 1]   | Primary greenness indicator       |
| EVI   | Enhanced Vegetation Index | 2.5 × (B8 − B4) / (B8 + 6×B4 − 7.5×B2 + 1)    | [−1, 1]   | Reduced soil/atmosphere effects   |
| BSI   | Bare Soil Index       | ((B11 + B4) − (B8 + B2)) / ((B11 + B4) + (B8 + B2)) | [−1, 1] | Bare soil / degradation proxy    |
| NDWI  | Normalized Difference Water Index | (B3 − B8) / (B3 + B8)                | [−1, 1]   | Surface moisture, water bodies    |
| fPAR  | Fraction Absorbed PAR | Derived from NDVI (linear approximation)             | [0, 1]    | Productivity, carbon uptake proxy |

**Citation:** NDVI — Tucker (1979); EVI — Huete et al. (2002); BSI — Rikimaru et al. (2002)

---

## Temporal Aggregation Conventions

| Label                  | Definition                                              | Use Case                                |
|------------------------|---------------------------------------------------------|-----------------------------------------|
| Monthly median         | Median of all valid (cloud-free) pixels in calendar month | Standard time series unit             |
| 6-week rolling         | Rolling median over 42-day window centered on date      | Smooth phenological transitions         |
| Phenological peak      | Maximum NDVI in green-up window (typically Mar–May, Oct–Nov for NE Kenya) | Annual productivity summary |
| Seasonal anomaly       | Departure from long-term monthly mean (2017–2024 baseline) | Drought/intervention detection        |

---

## Study Design Variables

| Variable         | Value / Description                                             |
|------------------|-----------------------------------------------------------------|
| Study area       | Garissa and Wajir counties, northeastern Kenya                  |
| N communities    | 50 communal rangeland management groups                         |
| Treatment        | Mercy Corps rangeland governance intervention                   |
| Intervention date| 2025 (use as DiD post-period cutpoint)                         |
| Pre-period       | 2017–2024 (Sentinel-2 archive baseline)                        |
| Control units    | Matched control wards (propensity-matched or synthetic control) |
| Rainfall covariate | CHIRPS v2.0 monthly precipitation (Funk et al. 2015)         |
| Primary outcome  | NDVI / EVI trend deviation post-intervention                   |
| CRS for analysis | UTM Zone 37N (EPSG:32637)                                      |
| CRS for input    | WGS84 (EPSG:4326) — reproject before area calculations         |

---

## Anti-Patterns (Do Not Do This)

| Anti-Pattern                              | Why It's Wrong                                                  | Correct Approach                              |
|-------------------------------------------|------------------------------------------------------------------|-----------------------------------------------|
| Single-date NDVI analysis                 | Single date is cloud-contaminated, noise-dominated              | Use monthly median composites                 |
| Ignoring bare soil fraction               | NDVI saturates in dense vegetation; BSI needed for semi-arid    | Report NDVI + BSI together                   |
| Not accounting for seasonal phenology     | Treatment effect confounded with wet/dry season signal          | Use seasonal anomalies or seasonally-demeaned NDVI |
| Area calculations in WGS84               | Angular units (degrees) give incorrect areas                    | Reproject to UTM 37N (EPSG:32637) first       |
| `.getInfo()` inside `.map()` loops        | Causes GEE server-side / client-side confusion, extremely slow  | Vectorize; use `.map()` server-side only      |
| Using TOA reflectance for vegetation indices | Atmospheric effects distort index values                     | Always use SR (`S2_SR_HARMONIZED`)            |
| Mixing WGS84/UTM without reprojection     | Silent layer misalignment, incorrect spatial joins              | Document CRS at every step; reproject explicitly |
| No cloud masking before compositing       | Clouds inflate reflectance, suppress NDVI                       | Apply SCL mask (classes 3,8,9,10,11) before composite |

---

## Lecture / Output Progression

| # | Output                   | Core Question                                         | Key Method                        |
|---|--------------------------|-------------------------------------------------------|-----------------------------------|
| 1 | Study Design Paper        | Does communal governance improve rangeland condition? | DiD with satellite-derived NDVI   |
| 2 | Methods Slides            | How do we measure rangeland health from space?        | GEE pipeline, spectral indices    |
| 3 | Results Slides            | What do early trends show?                            | Time series, treatment effect plots|
| 4 | Community Report          | What does this mean for communities?                  | Plain-language maps and summaries |

---

## Key Citations (Short Reference)

| Key | Full Citation |
|-----|--------------|
| Tucker1979 | Tucker, C.J. (1979). Red and photographic infrared linear combinations for monitoring vegetation. *Remote Sensing of Environment*, 8(2), 127–150. |
| Huete2002 | Huete, A., et al. (2002). Overview of the radiometric and biophysical performance of the MODIS vegetation indices. *Remote Sensing of Environment*, 83, 195–213. |
| Gorelick2017 | Gorelick, N., et al. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of Environment*, 202, 18–27. |
| Funk2015 | Funk, C., et al. (2015). The climate hazards infrared precipitation with stations — a new environmental record for monitoring extremes. *Scientific Data*, 2, 150066. |
| ESA_S2 | ESA (2022). Sentinel-2 User Handbook. European Space Agency. |
