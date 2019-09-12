
<img src="https://raw.githubusercontent.com/ryali93/rgee/master/man/figures/logo.png" align="right" width = 15%/>

# Google Earth Engine for R

[![Build
Status](https://travis-ci.org/csaybar/rgee.svg?branch=master)](https://travis-ci.org/csaybar/rgee)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/ryali93/rgee?branch=master&svg=true)](https://ci.appveyor.com/project/ryali93/rgee)
[![Codecov test
coverage](https://codecov.io/gh/csaybar/rgee/branch/master/graph/badge.svg)](https://codecov.io/gh/csaybar/rgee?branch=master)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)

[Google Earth Engine](https://earthengine.google.com/) (Gorelick et al.,
2017) is a cloud-based platform that allows users both run geospatial
analysis on Google’s infrastructure and getting access to a
petabyte-scale archive of remote sensing data. The [Google Earth Engine
API’s](https://developers.google.com/earth-engine/) are composed of a
set of modules that enable send queries through a REST API. The `rgee`
package provides full access to them from within R and defines
additional tools for automating processes.

**Earth Engine Python API**:

``` python
import ee
ee.Initialize()
image = ee.Image('CGIAR/SRTM90_V4')
image.bandNames().getInfo()
#> ['elevation']
```

**rgee:**

``` r
library(rgee)
ee_Initialize()
image <- ee$Image('CGIAR/SRTM90_V4')
image$bandNames()$getInfo()
#> [1] "elevation"
```

## Features

  - Complete access to the Earth Engine API from within R.
  - Multi-user support for Initialize
    ([ee\_Initialize](https://csaybar.github.io/rgee/reference/ee_Initialize.html))
  - Installing Python dependencies effortlessly
    ([ee\_install\_\*](https://csaybar.github.io/rgee/reference/ee_check-tools.html)).
  - Managing EE assets and tasks recursively
    ([ee\_manage\_\*](https://csaybar.github.io/rgee/reference/ee_manage-tools.html)).
  - Displaying EE spatial objects in an interactive view on top of
    specific basemaps
    ([ee\_map](https://csaybar.github.io/rgee/reference/ee_map.html)).
  - Generating metadata reports about EE objects
    ([ee\_print](https://csaybar.github.io/rgee/reference/ee_print.html)).
  - Uploading and downloading spatial objects.
    ([ee\_upload](https://csaybar.github.io/rgee/reference/ee_upload.html)
    and
    [ee\_download](https://csaybar.github.io/rgee/reference/ee_download.html))
  - Conversion of EE tables (Feature Collection) to sf and vice-versa
    ([ee\_as\_sf](https://csaybar.github.io/rgee/reference/ee_upload.html),
    [sf\_as\_ee](https://csaybar.github.io/rgee/reference/ee_download.html)
    and
    [ee\_as\_thumbnail](https://csaybar.github.io/rgee/reference/ee_download.html)).
  - Extract values for any EE ImageCollections objects
    ([ee\_extract](https://csaybar.github.io/rgee/reference/ee_upload.html)).
  - Searching among the Earth Engine Data Catalog
    ([ee\_search](https://csaybar.github.io/rgee/reference/ee_search.html)).
  - Displaying Earth Engine documentation
    ([ee\_help](https://csaybar.github.io/rgee/reference/ee_help.html)).

**NOTE: Access to Google Earth Engine is currently only available to
[registered users](https://earthengine.google.com/)**.

## Installation

Install development versions from github with

``` r
remotes::install_git("csaybar/rgee")
```

### Windows

Before install `rgee` be sure that
[Rtools](https://cran.r-project.org/bin/windows/Rtools/) is installed in
the system. Static libraries are automatically downloaded from
[rwinlib](https://github.com/rwinlib/).

### Linux

Please install the follow system libraries.

#### Ubuntu

    sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
    sudo apt-get update
    sudo apt-get install libudunits2-dev libgdal-dev libgeos-dev libproj-dev libv8-3.14-dev libjson-c-dev

### MacOS

Use [Homebrew](https://brew.sh/) to install system libraries:

    brew install pkg-config
    brew install gdal

## Docker image

Cooming soon\!

\#\# How does it
works?

![workflow](https://raw.githubusercontent.com/csaybar/rgee/master/man/figures/rgee.png)

## Credits

The rgee has been inspired by the following third-party Python packages:

  - **[gee\_asset\_manager - Lukasz
    Tracewski](https://github.com/tracek/gee_asset_manager)**  
  - **[geeup - Samapriya Roy](https://github.com/samapriya/geeup)**
  - **[geeadd - Samapriya
    Roy](https://github.com/samapriya/gee_asset_manager_addon)**
  - **[cartoee - Kel Markert](https://github.com/KMarkert/cartoee)**
  - **[geetools - Rodrigo E.
    Principe](https://github.com/gee-community/gee_tools)**
  - **[landsat-extract-gee - Loïc
    Dutrieux](https://github.com/loicdtx/landsat-extract-gee)**

\#\# Getting started

``` r
library(rgee)
library(readr)
library(stars)
library(sf)
library(mapview)

ee_initialize()

# Communal Reserve Amarakaeri - Peru
xmin <- -71.132591318
xmax <- -70.953664315
ymin <- -12.892451233
ymax <- -12.731116372
x_mean <- (xmin + xmax) / 2
y_mean <- (ymin + ymax) / 2

ROI <- c(xmin, ymin, xmin, ymax, xmax, ymax, xmax, ymin, xmin, ymin)
ROI_polygon <- matrix(ROI, ncol = 2, byrow = TRUE) %>%
list() %>%
st_polygon() %>%
st_sfc() %>%
st_set_crs(4326)
ee_geom <- ee_as_eegeom(ROI_polygon)


# Get the mean annual NDVI for 2011
cloudMaskL457 <- function(image) {
qa <- image$select("pixel_qa")
cloud <- qa$bitwiseAnd(32L)$
  And(qa$bitwiseAnd(128L))$
  Or(qa$bitwiseAnd(8L))
mask2 <- image$mask()$reduce(ee_Reducer()$min())
image <- image$updateMask(cloud$Not())$updateMask(mask2)
image$normalizedDifference(list("B4", "B3"))
}

ic_l5 <- ee_ImageCollection("LANDSAT/LT05/C01/T1_SR")$
filterBounds(ee_geom)$
filterDate("2011-01-01", "2011-12-31")$
map(cloudMaskL457)
mean_l5 <- ic_l5$mean()$rename("NDVI")
mean_l5 <- mean_l5$reproject(crs = "EPSG:4326", scale = 500)
mean_l5_Amarakaeri <- mean_l5$clip(ee_geom)
```

``` r
# Download -> ee$Image
task_img <- ee$batch$Export$image$toDrive(
  image = mean_l5_Amarakaeri,
  description = "Amarakaeri_task_1",
  folder = "Amarakaeri",
  fileFormat = "GEOTIFF",
  fileNamePrefix = "NDVI_l5_2011_Amarakaeri"
)
task_img$start()
ee_download_monitoring(task_img, eeTaskList = TRUE) # optional
img <- ee_download_drive(
  task = task_img,
  filename = "amarakaeri.tif",
  overwrite = T
)
plot(img)

# Download -> ee$FeatureCollection
amk_f <- ee$FeatureCollection(list(ee$Feature(ee_geom, list(name = "Amarakaeri"))))
amk_fc <- ee$FeatureCollection(amk_f)
task_vector <- ee$batch$Export$table$toDrive(
  collection = amk_fc,
  description = "Amarakaeri_task_2",
  folder = "Amarakaeri",
  fileFormat = "GEOJSON",
  fileNamePrefix = "geometry_Amarakaeri"
)

task_vector$start()
ee_download_monitoring(task_vector, eeTaskList = TRUE) # optional
amk_geom <- ee_download_drive(
  task = task_vector,
  filename = "amarakaeri.geojson",
  overwrite = TRUE
)

plot(amk_geom$geometry, add = TRUE, border = "red", lwd = 3)
py_help(ee$batch$Export$table$toDrive)
```

## Contact

Please file bug reports and feature requests at
<https://github.com/csaybar/rgee/issues>

In case of Pull Requests, please make sure to submit them to the develop
branch of this repository.
