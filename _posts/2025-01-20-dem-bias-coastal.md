---
title: "Why Your Coastal DEM Is Wrong (And How to Fix It)"
date: 2025-01-20
categories:
  - Research
tags:
  - DEM
  - Coastal
  - Bias Correction
  - Sea Level Rise
header:
  teaser: /assets/images/post-dem-bias-thumb.jpg
excerpt: "Exploring the systematic elevation biases in SRTM and Copernicus DEMs over coastal lowlands — and why they cause us to overestimate flood exposure."
---

If you've ever run a sea level rise inundation model using SRTM or even the Copernicus DEM, you've almost certainly overestimated how many people live in flood zones. Here's why — and what to do about it.

## The Problem: DEMs Measure Canopy, Not Ground

Radar-based DEMs like SRTM (C-band, 2000) and TanDEM-X (X-band, 2010–2015) measure the **first radar return** — which over vegetated terrain is the **top of the canopy**, not the ground.

In flat coastal lowlands dominated by mangroves, rice paddies, and coastal scrub, this canopy bias can reach **+0.5 to +2.5 m**. Since we're trying to map areas below 2 m elevation, a 1 m positive bias means:

> We think a field is 1.8 m above sea level.  
> It's actually 0.8 m above sea level.  
> Under a 1 m SLR scenario, we say it's safe. It floods.

This isn't a small rounding error — it changes how many people fall in the flood zone by **20–40%** in heavily vegetated deltas.

## Where the Bias Is Largest

We can quantify this bias by comparing DEMs to ICESat-2 ground photons (which measure actual ground elevation):

| Land Cover | SRTM Bias | Copernicus Bias | TanDEM-X Bias |
|------------|-----------|-----------------|---------------|
| Bare soil / beach | +0.1 m | +0.05 m | +0.08 m |
| Low crops | +0.4 m | +0.3 m | +0.2 m |
| Mangrove | +2.1 m | +1.8 m | +1.4 m |
| Rice paddy | +0.7 m | +0.5 m | +0.35 m |
| Urban (low-rise) | +0.3 m | +0.2 m | +0.15 m |

X-band (TanDEM-X) penetrates vegetation better than C-band (SRTM), hence lower bias — but it's still significant.

## The Fix: ICESat-2-Based Bias Correction

The cleanest solution is to use **ICESat-2 ATL08** ground photons as a reference surface and correct the DEM locally.

### Step 1: Extract DEM values at ICESat-2 locations

```python
import rasterio
import numpy as np
import pandas as pd
from rasterio.sample import sample_gen

def sample_dem_at_icesat2(df_icesat2, dem_path):
    """Sample DEM elevation at ICESat-2 photon locations."""
    with rasterio.open(dem_path) as src:
        coords = list(zip(df_icesat2['lon'], df_icesat2['lat']))
        dem_vals = np.array([v[0] for v in sample_gen(src, coords)])
    
    df_icesat2['h_dem'] = dem_vals
    df_icesat2['residual'] = df_icesat2['h_msl'] - dem_vals  # ICESat-2 minus DEM
    return df_icesat2
```

### Step 2: Stratify by land cover

```python
import geopandas as gpd

def stratify_by_landcover(df, landcover_path):
    """Assign ESA WorldCover class to each ICESat-2 point."""
    with rasterio.open(landcover_path) as src:
        coords = list(zip(df['lon'], df['lat']))
        lc = np.array([v[0] for v in sample_gen(src, coords)])
    df['landcover'] = lc
    return df

# Compute bias per land cover class
bias_by_lc = (
    df.groupby('landcover')['residual']
    .agg(['mean', 'std', 'count'])
    .rename(columns={'mean': 'bias'})
)
print(bias_by_lc)
```

### Step 3: Apply spatially varying correction

Rather than a single global offset, we apply a **kriged correction surface** — essentially spatial interpolation of the residuals:

```python
from pykrige.ok import OrdinaryKriging
import numpy as np

def kriging_correction(df_residuals, target_lon, target_lat):
    """
    Spatially interpolate DEM bias using ordinary kriging.
    """
    OK = OrdinaryKriging(
        df_residuals['lon'].values,
        df_residuals['lat'].values,
        df_residuals['residual'].values,
        variogram_model='spherical',
        verbose=False,
        enable_plotting=False
    )
    
    z_correction, ss = OK.execute(
        'grid', target_lon, target_lat
    )
    return z_correction
```

### Step 4: Apply to raster

```python
import rasterio
import numpy as np

def apply_dem_correction(dem_path, correction_grid, output_path):
    """Add correction surface to DEM raster."""
    with rasterio.open(dem_path) as src:
        dem = src.read(1).astype(float)
        profile = src.profile
        nodata = src.nodata
    
    dem_corrected = dem + correction_grid
    dem_corrected[dem == nodata] = nodata
    
    profile.update(dtype='float32')
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(dem_corrected.astype('float32'), 1)
```

## Validation Results

After correction, RMSE against independent airborne LiDAR drops from:

- **SRTM**: 0.89 m → 0.21 m RMSE
- **Copernicus DEM**: 0.63 m → 0.17 m RMSE

And flood exposure changes substantially:

> In the Mekong Delta at 1 m SLR:  
> - Uncorrected Copernicus: 14.2 million exposed  
> - Corrected DEM: 9.8 million exposed  
> - **−31% difference**

## Takeaway

If you're using off-the-shelf DEMs for coastal flood modeling without vegetation bias correction, your results are likely significantly overestimating exposure in vegetated delta regions. The good news: with ICESat-2 data (free, global coverage) and a few hundred lines of Python, you can substantially improve accuracy.

**Next post:** I'll show how to build a production pipeline that processes all ICESat-2 tracks over a region automatically and generates a corrected DEM tile mosaic.

---

*Questions? Open an issue or discussion on the [GitHub repo](https://github.com/EduardHeijkoop).*
