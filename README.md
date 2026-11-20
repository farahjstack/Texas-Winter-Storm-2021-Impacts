# Houston Blackout: Impacts of the February 2021 Texas Winter Storm

## About
This repository contains an analysis of the February 2021 Texas winter storms, estimating how many homes in the Houston metropolitan area lost power using satellite night lights and OpenStreetMap data.

In February 2021, a series of severe winter storms caused a major power crisis across Texas. This project uses remotely sensed nighttime lights from NASA's VIIRS (Visible Infrared Imaging Radiometer Suite) instrument, combined with OpenStreetMap roads and buildings, to estimate how many homes in the Houston metropolitan area lost power. Night light intensities from February 7, 2021 (pre-storm) and February 16, 2021 (post-storm) are compared to identify areas that experienced a blackout, and those areas are spatially joined with residential building footprints.

The goal of this analysis is to quantify the extent of the blackout in Houston and to reflect on the limitations of using night lights as a proxy for power loss.

## Repository Contents
- `texas_blackout.qmd`: Quarto document with R code, maps, and written analysis.
- `texas_blackout.pdf`: Rendered PDF of the Quarto document.
- `.gitignore`: Excludes the locally stored `data/` folder from Git version control.
- `README.md`: Project overview, data information, and references.

## Data
The data are not stored in this repository because of their size. Download the course data bundle, unzip it, and place the folders in a `data/` directory.

### Sources
- NASA Level-1 and Atmospheric Archive & Distribution System Distributed Active Archive Center (LAADS DAAC), VIIRS/NPP Daily Gridded Day Night Band 500m Linear Lat Lon Grid Night (VNP46A1).
- OpenStreetMap contributors, distributed by Geofabrik (roads and buildings).

### Files used
- `VNP46A1/VNP46A1.A2021038.h08v05.001.2021039064328.tif`: tile h08v05, collected 2021-02-07
- `VNP46A1/VNP46A1.A2021038.h08v06.001.2021039064329.tif`: tile h08v06, collected 2021-02-07
- `VNP46A1/VNP46A1.A2021047.h08v05.001.2021048091106.tif`: tile h08v05, collected 2021-02-16
- `VNP46A1/VNP46A1.A2021047.h08v06.001.2021048091105.tif`: tile h08v06, collected 2021-02-16
- `gis_osm_roads_free_1.gpkg`: OpenStreetMap roads for the Houston metropolitan area (motorways only are read in)
- `gis_osm_buildings_a_free_1.gpkg`: OpenStreetMap buildings for the Houston metropolitan area (residential buildings only are read in)

## Reproducing the Analysis
1. Clone or download this GitHub repository.
2. Download the course data and place the files in `data/` (`data/VNP46A1`, `data/gis_osm_roads_free_1.gpkg`, `data/gis_osm_buildings_a_free_1.gpkg`).
3. Install the required R packages.
4. Open `texas_blackout.qmd` in RStudio or another R-compatible environment.
5. Check that the file paths in the data-loading chunk match your local folder names.
6. Render the Quarto document to PDF to generate the maps and written analysis.

## Methods
Spatial data were processed and visualized in R using the following packages:

- terra
- stars
- tmap
- tidyverse
- kableExtra
- gt

**Night lights:** The two tiles covering Houston for each date were combined into one raster per date using `st_mosaic()`. The pre-storm raster was subtracted from the post-storm raster, and any cell with a drop of more than 200 nW cm⁻² sr⁻¹ was classified as a blackout. All other cells were set to `NA`.

**Blackout mask:** The blackout raster was vectorized with `st_as_sf()`, geometries were fixed with `st_make_valid()`, and the mask was cropped to the Houston area (-96.5, 29) to (-94.5, 30.5). It was then reprojected to EPSG:3083 (NAD83 / Texas Centric Albers Equal Area).

**Highway exclusion:** Highways (`fclass = 'motorway'`) were buffered by 200 m, and the buffers were merged with `st_union()`. Areas within this buffer were removed from the blackout mask to avoid falsely flagging reduced traffic as power loss.

**Homes affected:** Residential buildings were read in using a SQL query, reprojected to match the blackout mask, and filtered with `st_filter()` to keep only homes intersecting blackout areas. Approximately **157,408 homes** were estimated to have lost power.

All layers were checked for matching coordinate reference systems, with warnings printed if they do not match.

## Author
Farah Stack

## References
NASA LAADS DAAC. VIIRS/NPP Daily Gridded Day Night Band 500m Linear Lat Lon Grid Night, Version 1 (VNP46A1). https://ladsweb.modaps.eosdis.nasa.gov/

OpenStreetMap contributors. Data distributed by Geofabrik GmbH. https://download.geofabrik.de/

UCSB Master of Environmental Data Science. *EDS 223: Geospatial Analysis and Remote Sensing.* https://eds-223-geospatial.github.io

