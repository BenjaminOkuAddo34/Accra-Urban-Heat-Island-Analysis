# Accra Urban Heat Island Analysis: Setup Log

Process documentation for Milestone 1 (repository setup) and the first part of Milestone 2 (Earth Engine connection). Last updated: 18 November 2026.

## 1. Project summary

**Goal:** Map Land Surface Temperature (LST) across Accra, Ghana, and examine how vegetation (NDVI) and built-up areas (NDBI) relate to urban heat hotspots, using Landsat 8 and 9 imagery.

**Stack:** Python, Google Earth Engine, Geemap, GeoPandas, Rasterio, Folium, QGIS, Jupyter, Git/GitHub. Docker is planned for later, once the local environment and Earth Engine authentication are stable.


**Shell:** PowerShell on Windows.

## 2. Key decisions

- **QGIS** complements Python: inspection, validation, cartography and publication maps. It is installed as a desktop app, not Dockerized.
- **Docker** is deferred. Local venv first, then Dockerize the Python environment only.
- **Large data stays out of Git:** Landsat imagery, processed GeoTIFFs and NDVI/NDBI rasters are ignored. Code, notebooks, the QGIS project, small boundary files and selected lightweight HTML maps are committed.
- **Credentials are never committed.** Earth Engine's token lives in the user profile, and the project ID lives in `.env`, which is git-ignored.
- **Commit in stages** so the GitHub history shows the development process.

## 3. Milestone 1: Repository setup (complete)

### 3.1 Folder structure

Created with PowerShell:

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
- `requirements.txt`: earthengine-api, geemap, numpy, pandas, scipy, scikit-learn, geopandas, rasterio, rioxarray, shapely, pyproj, xarray, matplotlib, seaborn, folium, mapclassify, jupyterlab, ipykernel, notebook, python-dotenv, tqdm, requests. Exact versions will be pinned once everything is tested.

> Note: because `*.tif` is ignored globally, any small example raster you want to keep needs an exception, for example `!docs/**/*.tif`.

### 3.3 Python environment

```powershell
python -m venv .venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name accra-heat --display-name "Python (Accra Heat Analysis)"
```

**Verification:**

```powershell
python -c "import ee, geemap, geopandas, rasterio, folium, pandas, numpy; print('All core GIS dependencies imported successfully!')"
```

**Result:** all imports succeeded. pip upgraded to 26.3. Jupyter kernel `accra-heat` installed.

### 3.4 README and first commit

The README covers overview, objectives, data sources, five-phase methodology, tech stack, repository structure, deliverables, data management, status and license (MIT). The first commit, "Initial project structure and documentation", was made and pushed to GitHub.

**Result:** `git status` reported the branch up to date with `origin/main`.

## 4. Milestone 2: Earth Engine connection (in progress)

### 4.1 Register a Cloud project

1. Opened `https://code.earthengine.google.com/register`.
2. Chose unpaid (non-commercial) use, for research or education.
3. Created a new Google Cloud project.

**Project ID:** `accra-urban-heat-island`

### 4.2 Authenticate

```powershell
python -c "import ee; ee.Authenticate()"
```

Signed in with the same Google account used for registration and approved access. **Result:** successful. The token is saved in the user profile, not in the repo.

### 4.3 Store the project ID

```powershell
"EE_PROJECT=accra-urban-heat-island" | Out-File -Encoding ascii .env
"EE_PROJECT=your-gcp-project-id" | Out-File -Encoding ascii .env.example
```

`git status` confirmed `.env` is not tracked and only `.env.example` showed as modified.

### 4.4 Test the connection

```powershell
python -c "import ee, os; from dotenv import load_dotenv; load_dotenv(); ee.Initialize(project=os.getenv('EE_PROJECT')); print(ee.Image('USGS/SRTMGL1_003').getInfo()['type'])"
```

**Result:** printed `Image`. Earth Engine is connected from Python.

## 5. Status checklist

| Item | Status |
| --- | --- |
| Folder structure | Done |
| .gitignore, .dockerignore, requirements.txt | Done |
| Virtual environment and dependencies | Done |
| Jupyter kernel (accra-heat) | Done |
| README and first commit pushed to GitHub | Done |
| Earth Engine registration and project ID | Done |
| Earth Engine authentication | Done |
| `.env` created and git-ignored | Done |
| Connection test | Done |
| Commit `.env.example` update | Pending |
| `01_data_exploration.ipynb` | Next |

## 6. Issues and notes

- Earth Engine now requires a Cloud project even for free non-commercial use.
- Accra's rainy seasons (roughly April to July and September to November) produce heavy cloud cover. December to February is likely cleanest, though harmattan dust haze can affect imagery. Scene counts by month will guide the choice of comparison periods.
- The study area will start as a rough bounding box and be replaced with official boundaries (GADM, geoBoundaries or Ghana Statistical Service).

## 7. Next steps

1. Commit the `.env.example` change and push.
2. Create `notebooks/01_data_exploration.ipynb` with the kernel **Python (Accra Heat Analysis)**.
3. Run the connection cell, then count Landsat 8 and 9 scenes over Accra.
4. Source official boundary data.
5. Move on to cloud masking and LST calculation (`02_landsat_preprocessing.ipynb`, `03_lst_calculation.ipynb`).
