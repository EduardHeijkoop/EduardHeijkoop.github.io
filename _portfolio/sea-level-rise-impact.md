---
title: "Sea Level Rise Impact Assessment"
excerpt: "Mapping population and infrastructure exposure to 1–2 m sea level rise scenarios across global coastal lowlands."
header:
  image: /assets/images/portfolio-slr.jpg
  teaser: /assets/images/portfolio-slr-th.jpg
tags:
  - Sea Level Rise
  - Inundation
  - GIS
  - Folium
  - Impact Assessment
sidebar:
  - title: "Scenarios"
    text: "IPCC AR6: 0.5 m, 1.0 m, 2.0 m by 2100"
  - title: "Tools"
    text: "Python, Folium, Rasterio, Fiona, PostGIS"
  - title: "Coverage"
    text: "Global coastal zone ±60° latitude"
toc: true
toc_sticky: true
---

## Overview

This project quantifies the **exposure of coastal populations, buildings, and critical infrastructure** to future sea level rise using a combination of corrected elevation models and demographic datasets.

Using the bias-corrected ICESat-2 DEM products, we run static bathtub inundation models under IPCC AR6 sea level rise scenarios and intersect the flood extents with:
- WorldPop gridded population counts
- OpenStreetMap infrastructure (roads, hospitals, ports)
- Global urban footprint datasets

## Interactive Flood Exposure Map

Explore inundation extents and population exposure across key study regions:

<iframe 
  src="/assets/maps/slr_impact.html" 
  width="100%" 
  height="550" 
  frameborder="0"
  style="border-radius: 8px; margin: 1rem 0;">
</iframe>

*Toggle between 0.5 m, 1.0 m, and 2.0 m scenarios. Circle size = exposed population in each 0.1° grid cell. Data: corrected TanDEM-X DEM + WorldPop 2020.*

## Study Regions

We focus on four contrasting delta/coastal systems:

### 1. Mekong Delta, Vietnam
One of the world's most densely populated deltas, with mean elevation < 1 m and rapid subsidence (up to 3 cm/yr). Under a 1 m SLR scenario, **~12 million people** fall within the modeled inundation zone.

### 2. Ganges-Brahmaputra Delta (Bangladesh)
Highly dynamic, with seasonal flooding already common. Corrected DEMs reveal significant underestimation of exposure in uncorrected SRTM-based analyses.

### 3. U.S. Gulf Coast
Well-constrained by NOAA tide gauges and dense airborne LiDAR. Used as a primary validation region.

### 4. Netherlands Low Lands
Protected by an extensive dike network — this region tests the distinction between topographic exposure and actual flood risk.

## Methodology

### Static Bathtub Inundation

```python
import rasterio
import numpy as np
from rasterio.features import shapes
import geopandas as gpd

def bathtub_inundation(dem_path, slr_scenario, connectivity=True):
    """
    Compute inundated area for a given SLR scenario.
    
    Parameters
    ----------
    dem_path : str
        Path to corrected DEM (in meters, referenced to MSL)
    slr_scenario : float
        Sea level rise amount in meters
    connectivity : bool
        If True, only flood cells connected to ocean
    
    Returns
    -------
    GeoDataFrame of inundated polygons
    """
    with rasterio.open(dem_path) as src:
        dem = src.read(1).astype(float)
        dem[dem == src.nodata] = np.nan
        transform = src.transform
        crs = src.crs
    
    # Simple bathtub: all cells below SLR threshold
    flooded = (dem <= slr_scenario) & (~np.isnan(dem))
    
    if connectivity:
        flooded = apply_ocean_connectivity(flooded, dem)
    
    # Vectorize
    flood_shapes = list(shapes(
        flooded.astype(np.uint8), transform=transform
    ))
    
    polygons = [shape for shape, val in flood_shapes if val == 1]
    return gpd.GeoDataFrame(geometry=polygons, crs=crs)
```

### Population Exposure Calculation

We intersect flood polygons with WorldPop 2020 raster data to compute exposed population, disaggregated by:
- Administrative unit (country → province → district)
- Urban/rural classification
- Age group (using age-structured WorldPop products)

## Key Findings

| SLR Scenario | Global Exposed Population | vs. Uncorrected DEM |
|-------------|--------------------------|---------------------|
| 0.5 m | 340 million | −18% |
| 1.0 m | 620 million | −23% |
| 2.0 m | 1.1 billion | −15% |

**DEM correction significantly reduces exposure estimates** compared to analyses using raw SRTM or Copernicus data — particularly in low-lying deltas where positive elevation bias is largest.

## Visualizing with Folium

All interactive maps in this portfolio are generated using Python's Folium library:

```python
import folium
from folium.plugins import HeatMap, LayerControl
import geopandas as gpd

def create_slr_map(flood_gdfs, population_data, center=(10, 105)):
    """Create interactive SLR impact map."""
    m = folium.Map(
        location=center,
        zoom_start=7,
        tiles='CartoDB dark_matter'
    )
    
    colors = {'0.5m': '#f4d03f', '1.0m': '#e67e22', '2.0m': '#e74c3c'}
    
    for scenario, gdf in flood_gdfs.items():
        fg = folium.FeatureGroup(name=f'SLR {scenario}', show=False)
        folium.GeoJson(
            gdf,
            style_function=lambda x, c=colors[scenario]: {
                'fillColor': c, 'color': c,
                'fillOpacity': 0.4, 'weight': 0.5
            }
        ).add_to(fg)
        fg.add_to(m)
    
    LayerControl().add_to(m)
    return m
```

## Publications & Data

- Full methodology and results available on [GitHub](https://github.com/EduardHeijkoop)
- Processed inundation polygons archived on Zenodo (DOI pending)
