---
paths:
  - "python/**/*.py"
  - "notebooks/**/*.ipynb"
---

# Python Code Conventions

Apply these conventions to all Python scripts (`python/`) and Jupyter notebooks (`notebooks/`).

---

## Reproducibility

- **Environment pinning:** Every project must have `requirements.txt` or `environment.yml` with pinned versions (`numpy==1.26.4`, not `numpy>=1.26`).
- **Random seeds:** Call `random.seed(42)` and `np.random.seed(42)` at the top of any script with stochastic operations. Document the seed in a comment.
- **No hardcoded paths:** Use `pathlib.Path` for all file paths. Define a `PROJECT_ROOT = Path(__file__).parent.parent` at the top of production scripts.
- **Date of analysis:** Log the run date and key package versions in output metadata or log files.

---

## GEE Conventions

- **Initialize first:** `ee.Initialize(project='YOUR_GEE_PROJECT')` must be the first substantive line after imports. Never call EE functions before initializing.
- **Filter order:** Always apply `.filterBounds()` before `.filterDate()` — bounds reduces data volume first.
- **No `.getInfo()` in loops:** `.getInfo()` triggers a round-trip to GEE servers. Inside `.map()` or any loop it is an anti-pattern. Vectorize operations server-side.
- **Scale factor:** Sentinel-2 SR values are in units × 10000. Divide by 10000 before computing spectral indices.
- **Cloud masking:** Apply SCL-based mask (exclude classes 3, 8, 9, 10, 11) before any compositing. Document which SCL classes are masked.
- **Collection ID:** Always comment the full collection ID, e.g., `# COPERNICUS/S2_SR_HARMONIZED, accessed 2025-03-01`.

---

## Error Handling

- **GEE exports:** Wrap `ee.batch.Export.*` calls in `try/except ee.EEException` and log quota errors explicitly.
- **No silent failures:** Do not use bare `except: pass`. Log errors with `logging.error()` at minimum.
- **Validation:** Validate geometry bounds and image collection size (`.size().getInfo()`) before expensive operations.

---

## Figure Quality

All maps and figures in `python/` and `notebooks/` must be publication-ready:

- **Maps:** Include scale bar, north arrow, colorbar with units, coordinate labels (latitude/longitude graticule).
- **Resolution:** Minimum 300 DPI for raster exports (`plt.savefig(..., dpi=300)`).
- **CRS on maps:** Display CRS in figure caption or subtitle (e.g., "UTM Zone 37N / WGS84").
- **Color palettes:** Use colorblind-safe palettes (`viridis`, `RdYlGn`). Never use default matplotlib jet for continuous data.
- **Figure size:** Set `figsize` explicitly; do not rely on defaults.

---

## Script Hygiene (Production: `python/`)

- **Type hints:** All public functions must have type annotations (`def compute_ndvi(image: ee.Image) -> ee.Image:`).
- **Module docstring:** Every script must have a top-level docstring describing purpose, inputs, outputs, and usage.
- **No magic numbers:** Define constants at the top (`SCALE_FACTOR = 10000`, `CRS = "EPSG:32637"`).
- **Logging:** Use the `logging` module (not `print`) for status messages in production scripts.

---

## Notebook Hygiene (`notebooks/`)

- **Restart + Run All:** Before committing, always restart the kernel and run all cells. No out-of-order execution artifacts.
- **Strip outputs (default):** Strip cell outputs before committing unless the output is essential documentation (e.g., a key figure). Use `nbstripout` or manual clearing.
- **Cell size:** Keep individual cells focused (< 30 lines). Split long cells.
- **Markdown narrative:** Every notebook must have markdown cells explaining the purpose, methodology, and interpretation of each code section.

---

## Memory and Scale

- **Large rasters:** Use `ee.batch.Export.image.toAsset()` or `toCloudStorage()` for rasters > ~100 MB. Never `.getInfo()` on large image collections.
- **Collection size check:** Before `.map()` on a large collection, run `.size().getInfo()` and log the count.
- **Intermediate assets:** Store key intermediate products as GEE assets with descriptive IDs:
  `projects/YOUR_PROJECT/assets/kenya_ndvi_monthly_YYYY`
