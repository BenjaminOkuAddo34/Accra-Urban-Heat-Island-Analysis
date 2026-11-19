# Accra Urban Heat Island Analysis: Project Log

Process documentation from repository setup through the published maps. Last updated: 19 November 2026.

## 1. Project summary

**Goal:** Map land surface temperature (LST) across Greater Accra, Ghana, and examine how vegetation (NDVI) and built-up or bare surfaces (NDBI) relate to urban heat hotspots, using Landsat 8 and 9 imagery.

**Stack:** Python, Google Earth Engine, Geemap, GeoPandas, Folium, QGIS, Jupyter, Git/GitHub. Docker is planned.

**Shell:** PowerShell on Windows
**Earth Engine project ID:** stored in a local `.env` file (not published)
**Live map:** https://benjaminokuaddo34.github.io/Accra-Urban-Heat-Island-Analysis/maps/interactive/accra_heat_map.html

## 2. Key decisions

- **QGIS complements Python.** Python does the repeatable processing and analysis. QGIS is used for inspection, validation and cartography. It runs as a desktop app and is not Dockerized.
- **Docker is deferred** until the local environment is stable, and will cover only the Python environment.
- **Large data stays out of Git.** Landsat imagery and GeoTIFFs are ignored. Code, notebooks, the QGIS project, small CSV results, static maps and the interactive HTML map are committed.
- **Credentials are never committed.** The Earth Engine token lives in the user profile and the project ID lives in `.env`, which is git-ignored.
- **Dry season only.** The rainy season has too few cloud-free scenes (see section 5).
- **Seasons are compared as anomalies, not raw degrees,** because absolute temperatures shift between seasons (see section 7).
- **Commit in stages** so the GitHub history shows the development process.

## 3. Milestone 1: Repository setup (complete)

### 3.1 Folder structure

```text
Accra-Urban-Heat-Island-Analysis/
├── data/ (raw, interim, processed, external)
├── notebooks/
├── src/ (data, processing, analysis, visualization)
├── maps/ (static, interactive)
├── qgis/styles/
├── reports/figures/
├── docs/
├── docker/Dockerfile
├── .gitignore, .dockerignore, .env.example
├── compose.yaml, requirements.txt
└── README.md, LICENSE
```

### 3.2 Configuration files

- `.gitignore`: ignores virtual environments, `__pycache__`, Jupyter checkpoints, `.env` files (except `.env.example`), credential JSON files, contents of the `data/` folders (with `.gitkeep` exceptions), raster formats (`.tif`, `.tiff`, `.jp2`, `.nc` and similar), logs, QGIS temp files, OS and IDE files.
- `.dockerignore`: keeps `.git`, venvs, secrets and large data out of the Docker build context.
- `.gitkeep` placeholders in each `data/` subfolder so Git tracks the empty directories.
- `requirements.txt`: earthengine-api, geemap, numpy, pandas, scipy, scikit-learn, geopandas, rasterio, rioxarray, shapely, pyproj, xarray, matplotlib, seaborn, folium, mapclassify, jupyterlab, ipykernel, notebook, python-dotenv, tqdm, requests. Exact versions are still to be pinned.

> Because `*.tif` is ignored globally, a small example raster that should be tracked needs an exception, for example `!docs/**/*.tif`.

### 3.3 Python environment

```powershell
python -m venv .venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name accra-heat --display-name "Python (Accra Heat Analysis)"
```

**Result:** all core imports succeeded and the Jupyter kernel `accra-heat` was installed.

### 3.4 README and first commit

The first commit, "Initial project structure and documentation", was pushed to GitHub.

## 4. Milestone 2: Earth Engine connection (complete)

1. Registered for Earth Engine (non-commercial use) and created a Google Cloud project for the analysis.
2. Authenticated with `python -c "import ee; ee.Authenticate()"`. The token is stored in the user profile.
3. Saved `EE_PROJECT=<your-project-id>` in `.env` (git-ignored), with a placeholder in `.env.example`.
4. Tested the connection by reading `USGS/SRTMGL1_003`, which returned `Image`.

## 5. Milestone 3: Data exploration (notebook 01)

### 5.1 Scene availability

A rough study box was used first: `ee.Geometry.Rectangle([-0.35, 5.45, 0.05, 5.80])`.

- In 2023, with a scene cloud limit of 30%, there were 7 Landsat 8 and 7 Landsat 9 scenes.
- 12 of those 14 scenes fell between November and March.
- Across 2019 to 2024 with a 70% cloud limit, June to October had only 0 to 3 scenes per month, and September was almost empty.

**Decision:** use the dry season (December to February).

### 5.2 Cloud flags

The `QA_PIXEL` band flags cloud (bit 3), cloud shadow (bit 4), cirrus (bit 2) and dilated cloud (bit 1). A mask removing all four was written as `mask_clouds()`. The 20 December 2023 Landsat 9 scene (2.67% scene cloud) showed both real clouds and harmattan haze. The mask removes clouds and shadow but not haze.

Scene-level cloud cover describes the whole Landsat tile (about 185 km wide), not just Accra.

## 6. Milestone 4: LST, NDVI and NDBI (notebook 02)

### 6.1 Reusable functions

- `mask_clouds(image)`: removes cloud, cirrus, dilated cloud and shadow pixels.
- `get_landsat(start, end, max_cloud=70)`: merges Landsat 8 and 9 over the study area, filters by date and cloud cover, and masks clouds.
- `add_lst(image)`: converts `ST_B10` to °C using `x * 0.00341802 + 149.0 - 273.15`.
- `add_indices(image)`: scales reflectance (`x * 0.0000275 - 0.2`) and adds NDVI (`SR_B5`, `SR_B4`) and NDBI (`SR_B6`, `SR_B5`).

### 6.2 First LST result (dry season 2022/23)

Combined 11 scenes with a per-pixel median. LST statistics over the rough box: minimum 26.4 °C, 5th percentile 27.4 °C, median 36.2 °C, 95th percentile 41.7 °C, maximum 51.3 °C. The mean (34.5 °C) was below the median because the box included ocean, so water masking was added.

### 6.3 Correlation (dry season 2022/23)

A random sample of land pixels (water removed with JRC Global Surface Water, occurrence above 50%) gave 1,022 usable pixels at 200 m.

| Pair | Correlation |
| --- | --- |
| LST vs NDVI | -0.78 |
| LST vs NDBI | +0.74 |
| NDVI vs NDBI | -0.91 |

NDVI and NDBI overlap heavily, so their separate effects cannot be separated reliably.

## 7. Milestone 5: District ranking

### 7.1 Boundaries

Boundaries come from geoBoundaries ADM2 v6.0.0 (`WM/geoLab/geoBoundaries/600/ADM2`, CC BY 4.0), filtered to Ghana (`GHA`). Ghana has 260 districts in the dataset. 28 touched the rough study box. Two Eastern Region districts (Akwapem South and Nsawam Adoagyiri) were excluded, leaving **26 Greater Accra districts**.

### 7.2 Clipping fix

The first ranking clipped LST to the rough box, so districts that extend beyond it (for example Ga South, with only 21 pixels) had unreliable averages. Removing the clip and letting `reduceRegions` cut to each district fixed this (Ga South rose to 12,797 pixels).

### 7.3 Five dry seasons

| Season | Scenes |
| --- | --- |
| 2019/20 | 6 |
| 2020/21 | 6 |
| 2021/22 | 9 |
| 2022/23 | 11 |
| 2023/24 | 11 |

Raw district temperatures shifted by several °C between seasons for almost every district (for example Accra Metropolis: 40.2, 43.4, 39.2, 41.9, 40.2). Likely causes are few scenes, different dates, weather and haze. Raw temperatures are therefore not compared across seasons.

### 7.4 Anomalies and rank stability

Each season's city average (a simple unweighted mean of the 26 districts) was subtracted from every district.

- **Consistently hot:** Ningo/Prampram, Ashaiman, Ablekuma Central, Accra Metropolis, Ayawaso Central, Ablekuma West.
- **Consistently cool:** Ga North (ranked 26th in all five seasons), Ga West, Ga South, La-Nkwantanang-Madina.
- **Less stable:** Tema Metropolitan (4th to 14th), Accra Metropolis (1st to 10th), Ningo/Prampram (1st to 13th).
- Average anomaly ranged from about +1.9 °C (hottest) to -3.8 °C (coolest).

Tema Metropolitan ranks among the hottest despite a low NDBI (0.03). One possible explanation is port and industrial surfaces. This has not been tested.

## 8. Milestone 6: Interactive map and publishing

### 8.1 Folium map

District shapes and results were joined into one GeoDataFrame and saved as `data/processed/accra_districts_heat.geojson` (not tracked). The standalone map `maps/interactive/accra_heat_map.html` has:

- one layer per dry season and a five-season average, coloured by heat relative to the city average
- a pixel-level LST layer for 2022/23 (`lst_2022_23.png`, placed with `ImageOverlay`)
- hover tooltips with temperature, anomaly and rank
- district name labels, two legends, a light grey basemap and a satellite basemap (Esri)

### 8.2 GitHub Pages

The repository was made public after checking `git ls-files` and the history for secrets. Pages was enabled from the `main` branch, root folder. The map is live at the address in section 1.

## 9. Milestone 7: QGIS cartography

1. Exported a three-band GeoTIFF (LST, NDVI, NDBI) at 30 m in EPSG:32630 (UTM zone 30N) from Earth Engine to Google Drive, then placed it in `data/processed/`.
2. Created `qgis/accra_heat_analysis.qgz` with the raster and the district layer.
3. Built three print layouts from one template (title, map, legend, scale bar, north arrow, credits): **LST** (30 to 48 °C), **NDVI** (0 to 0.6, inverted YlGn) and **NDBI** (-0.3 to 0.3, inverted RdYlGn).
4. Exported to `maps/static/` at 300 dpi: `lst_dry_2022_23.png`, `ndvi_dry_2022_23.png`, `ndbi_dry_2022_23.png`.

The QGIS project stores paths to the GeoTIFF and GeoJSON, which are not in GitHub. Anyone cloning the repository must regenerate them.

## 10. Issues and fixes

| Issue | Cause | Fix |
| --- | --- | --- |
| `geemap.ee_to_df` raised "Cannot specify both axis and index/columns" | Version clash between geemap and a newer pandas | Built the table with `pd.DataFrame([f["properties"] for f in features])` |
| Sampling 5,000 pixels timed out | Heavy computation on Earth Engine | Used `scale=200`, `numPixels=1500`, `tileScale=4` |
| Folium basemap asked for an API key | The CartoDB tile service was refusing keyless requests | Switched to Esri light grey and satellite tiles |
| Legends and layer panel missing after adding the pixel layer | Layer control was created before the pixel layer existed | Rebuilt the whole map in one cell with controls added last |
| Pages showed a "custom domain not properly formatted" error and a 404 | The map URL was typed into the Custom domain box | Cleared the field, reset Pages (branch to None, then back to `main`), waited for a successful build |
| Ningo/Prampram and Tema Metropolitan missing in QGIS | Some shapes were `GeometryCollection` objects mixed in with polygons | Converted all shapes to clean MultiPolygons and re-exported the GeoJSON |
| Legend boundary symbol invisible | A line patch cannot display a polygon outline style | Changed the layer to an Outline: Simple Line symbol |
| NDVI map colours reversed | Colour ramp ran the wrong way | Inverted the ramp so dense vegetation is dark green |

## 11. Limitations to carry into the report

- LST is surface temperature, not air temperature.
- Dry season only, with small scene counts in two seasons.
- Raw temperatures are not comparable across seasons, so no warming trend is claimed.
- Harmattan dust haze is not removed by the cloud mask.
- District averages were calculated at 120 m. Large rural districts and small central districts should be read differently.
- NDBI also responds to bare soil, so high NDBI does not always mean buildings.
- NDVI and NDBI are strongly correlated, and correlation is not proof of cause.
- The correlation uses one season and a sample of 1,022 pixels.

## 12. Status checklist

| Item | Status |
| --- | --- |
| Repository, environment and Earth Engine setup | Done |
| Notebook 01: data exploration | Done |
| Notebook 02: LST, NDVI, NDBI, ranking, maps | Done |
| Five-season district comparison | Done |
| Interactive map live on GitHub Pages | Done |
| Static maps (LST, NDVI, NDBI, 2022/23) | Done |
| README with findings and reproduction steps | Done |
| Final written report | Next |
| Docker environment | Planned |
| Move reusable code from notebooks into `src/` | Planned |
| Static maps for other seasons | Optional |

## 13. Next steps

1. Write `reports/final_report.md` (introduction, data, methods, results, discussion, limitations).
2. Add a Docker setup for the Python environment (`docker/Dockerfile`, `compose.yaml`).
3. Move the reusable functions into `src/` and pin package versions in `requirements.txt`.
4. Optional: investigate Tema Metropolitan's surface types in QGIS and add an urban-versus-rural comparison.
