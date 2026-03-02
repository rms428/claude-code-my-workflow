---
paths:
  - "python/**/*.py"
  - "notebooks/**/*.ipynb"
  - "python/**/*.js"
---

# Google Earth Engine Protocol

Apply these conventions to all GEE-related code (Python API and JavaScript Code Editor scripts).

---

## Authentication and Initialization

```python
# First-time setup (run once per machine)
import ee
ee.Authenticate()

# Every script / notebook
import ee
ee.Initialize(project='YOUR_GEE_PROJECT_ID')  # Replace with actual project ID
```

- Always use `project=` argument in `ee.Initialize()` for project-scoped quota.
- Document the GEE project ID in `.claude/state/personal-memory.md` (gitignored).
- For notebooks, place `ee.Initialize()` in the first code cell after imports.

---

## Collection Selection

```python
# CORRECT: Surface reflectance, harmonized across processing baselines
collection = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
              .filterBounds(study_area)        # bounds FIRST
              .filterDate('2017-01-01', '2025-12-31')
              .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 50)))

# WRONG: TOA reflectance (not corrected for atmosphere)
# collection = ee.ImageCollection('COPERNICUS/S2')  # DON'T use for vegetation analysis
```

**Rules:**
- Always comment the collection ID and the date you accessed/selected it.
- Use `COPERNICUS/S2_SR_HARMONIZED` for surface reflectance vegetation analysis.
- Apply `.filterBounds()` before `.filterDate()` (reduces data volume first).
- Document the cloud cover threshold used (`CLOUDY_PIXEL_PERCENTAGE`).

---

## Cloud Masking

```python
def mask_s2_clouds(image):
    """Mask clouds and shadows using Sentinel-2 Scene Classification Layer (SCL).

    Excluded SCL classes:
        3  = Cloud shadow
        8  = Cloud medium probability
        9  = Cloud high probability
        10 = Thin cirrus
        11 = Snow / Ice (not expected in NE Kenya, included for safety)
    """
    scl = image.select('SCL')
    cloud_mask = scl.neq(3).And(scl.neq(8)).And(scl.neq(9)).And(scl.neq(10)).And(scl.neq(11))
    return image.updateMask(cloud_mask).divide(10000)  # also applies scale factor
```

- Always document which SCL classes are excluded and why.
- Apply mask before any spectral index computation.
- The `.divide(10000)` scale factor application belongs in the mask function or immediately after.

---

## Asset Management

Name GEE assets descriptively following this convention:

```
projects/YOUR_PROJECT_ID/assets/kenya_[product]_[temporal_unit]_[year_or_range]
```

Examples:
- `projects/YOUR_PROJECT/assets/kenya_ndvi_monthly_2017_2024`
- `projects/YOUR_PROJECT/assets/kenya_s2_composite_peak_green_2023`
- `projects/YOUR_PROJECT/assets/garissa_wajir_boundary`

**Rules:**
- Export all intermediate products you'll reuse. Do not rely on ephemeral computations.
- Include the export date in a metadata property: `image.set('export_date', '2025-03-01')`.
- Log asset IDs in session logs or code comments for reproducibility.

---

## Export Conventions

Always set all four parameters explicitly in every export:

```python
task = ee.batch.Export.image.toAsset(
    image=ndvi_composite,
    description='kenya_ndvi_monthly_2023',
    assetId='projects/YOUR_PROJECT/assets/kenya_ndvi_monthly_2023',
    region=study_area.geometry(),
    crs='EPSG:32637',        # UTM Zone 37N for Kenya — ALWAYS for analysis outputs
    scale=10,                # match Sentinel-2 B8 resolution
    maxPixels=1e13,          # set explicitly; default 1e8 is too small for large areas
)
task.start()
```

**CRS rule:** All analysis outputs must use `EPSG:32637` (UTM Zone 37N). Raw GEE data arrives in EPSG:4326 — always reproject for area calculations and spatial joins.

**Export targets:**
- GEE Asset: for intermediate products you'll reuse in GEE
- Google Drive / Cloud Storage: for GeoTIFFs you'll use locally in Python

---

## Quota Awareness

```python
# Check collection size BEFORE mapping over large collections
n_images = collection.size().getInfo()
print(f"Collection contains {n_images} images")

# For very large collections (> 1000 images), consider chunking by year
```

- Use `.size().getInfo()` to check before `.map()` on unknown-size collections.
- Batch exports are asynchronous — always call `.start()` and poll with `ee.batch.Task.list()`.
- If hitting quota limits: (1) check `.filterBounds()` is applied, (2) reduce date range, (3) increase cloud filter threshold as a first pass.

---

## Reproducibility Requirements

- **Pin collection version:** Comment the date you selected the collection and any filtering parameters.
- **Export asset dates:** Always `image.set('export_date', 'YYYY-MM-DD')` and `image.set('collection', 'COPERNICUS/S2_SR_HARMONIZED')`.
- **Log task IDs:** After `.start()`, save the task ID (`task.id`) to a log file for monitoring.
- **No ephemeral computations for key results:** Any result that appears in a paper or slide must be derived from a saved GEE asset or exported GeoTIFF, not recomputed from scratch each time.

---

## JavaScript Code Editor (Supplementary)

If using GEE JavaScript Code Editor for exploration:

- Save scripts to a shared team repository (not just personal scripts).
- Include a header comment with: purpose, author, date, collection IDs used, and export destinations.
- Prototype in JS Code Editor → port to Python API for production use.
