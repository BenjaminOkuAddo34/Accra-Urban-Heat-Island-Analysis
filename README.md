# Accra Urban Heat Island Analysis

## 1. Project Overview

**Category:** Remote Sensing / Environmental Analysis / GIS

**Study Area:** Accra, Greater Accra Region, Ghana

**Primary Technologies:** Google Earth Engine, Python, Geemap, GeoPandas, Rasterio, QGIS and Folium.

### Objective

This project investigates the spatial distribution of Land Surface Temperature (LST) across Accra, Ghana, to identify urban heat hotspots and examine their relationship with vegetation cover, built-up areas and land use patterns.

Using Landsat 8 and Landsat 9 satellite imagery, the project will explore how urban development and vegetation distribution influence surface temperature patterns over time.

## 2. Project Objectives

- Retrieve and process Landsat 8/9 satellite imagery.
- Calculate Land Surface Temperature (LST).
- Calculate Normalized Difference Vegetation Index (NDVI).
- Calculate Normalized Difference Built-up Index (NDBI).
- Examine relationships between surface temperature, vegetation and built-up areas.
- Identify and rank urban heat hotspots.
- Compare surface temperature patterns across multiple years and seasons.
- Develop interactive maps for exploring spatial temperature patterns.

## 3. Data Sources

| Dataset | Purpose |
|---|---|
| Landsat 8 Collection 2 Level-2 | Surface temperature and spectral analysis |
| Landsat 9 Collection 2 Level-2 | Surface temperature and spectral analysis |
| Administrative boundaries | Study area and neighborhood mapping |
| Supporting geographic datasets | Urban and environmental context |

Satellite imagery will be accessed and processed through Google Earth Engine.

## 4. Methodology

### Phase 1: Data Acquisition
- Define the Accra study area.
- Retrieve Landsat imagery.
- Apply cloud and cloud-shadow masking.
- Filter imagery by date and season.

### Phase 2: Land Surface Temperature Analysis
- Process Landsat surface temperature data.
- Convert temperature measurements to degrees Celsius.
- Generate LST maps.
- Compare temperature distributions across selected periods.

### Phase 3: Vegetation and Built-up Analysis
- Calculate NDVI.
- Calculate NDBI.
- Examine vegetation and built-up spatial patterns.
- Investigate their statistical relationships with LST.

### Phase 4: Urban Heat Hotspot Analysis
- Identify areas with elevated surface temperatures.
- Rank neighborhoods based on their temperature statistics.
- Examine spatial patterns and possible contributing factors.

### Phase 5: Visualization and Reporting
- Produce thematic maps using Python and QGIS.
- Create interactive maps using Folium.
- Develop temporal comparison visualizations.
- Document findings and limitations.

## 5. Technology Stack

- **Google Earth Engine:** Satellite imagery acquisition and cloud-based processing.
- **Python:** Data processing and analysis.
- **Geemap:** Earth Engine integration and visualization.
- **GeoPandas:** Vector data processing.
- **Rasterio:** Raster data processing.
- **QGIS:** Spatial inspection, validation and cartography.
- **Folium / Leaflet:** Interactive mapping.
- **Jupyter Notebook:** Reproducible analysis.
- **Docker:** Reproducible development environment.
- **Git / GitHub:** Version control and project publishing.

## 6. Repository Structure

```text
Accra-Urban-Heat-Island-Analysis/
│
├── data/
│   ├── raw/
│   ├── interim/
│   ├── processed/
│   └── external/
│
├── notebooks/
│
├── src/
│   ├── data/
│   ├── processing/
│   ├── analysis/
│   └── visualization/
│
├── maps/
│   ├── static/
│   └── interactive/
│
├── qgis/
│   └── styles/
│
├── reports/
│   └── figures/
│
├── docs/
│
├── docker/
│
├── .dockerignore
├── .gitignore
├── .env.example
├── compose.yaml
├── requirements.txt
└── README.md
```

## 7. Expected Deliverables

- Land Surface Temperature maps of Accra.
- NDVI vegetation distribution maps.
- NDBI built-up area maps.
- Multi-year temperature comparison.
- Neighborhood-level hotspot ranking.
- Interactive urban heat map.
- Final analytical report.

## 8. Data Management

Large satellite imagery and processed raster datasets will be excluded from Git version control.

The repository will contain source code, notebooks, documentation, selected visualizations and lightweight supporting datasets.


## 9. Project Status

**Current Stage:** Repository Setup

This project is under development. Analytical outputs and findings will be added progressively