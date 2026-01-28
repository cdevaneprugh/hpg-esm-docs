# OSBS Implementation

*Applying the validated hillslope methodology to 1m NEON LIDAR data for the Ordway-Swisher Biological Station.*

---

## Introduction

With the pipeline validated against Swenson's published data (see [MERIT Validation](merit-validation.md)), the next step was to apply it to 1m NEON LIDAR data for OSBS. This required adapting the pipeline for a 90x finer resolution, UTM coordinates instead of geographic, and irregular tile coverage with nodata gaps.

---

## NEON LIDAR Data

| Property | Value |
|----------|-------|
| Dataset | NEON DP3.30024.001 (Elevation - LiDAR) |
| Site | OSBS (Ordway-Swisher Biological Station) |
| Collection | 2023-05 |
| Files | 233 DTM tiles |
| Total size | 554 MB |
| Resolution | 1.0 m |
| CRS | EPSG:32617 (UTM Zone 17N) |
| Tile size | 1000 x 1000 pixels (1 km^2) |

---

## DEM Preprocessing

The 233 tiles were stitched into a single mosaic for processing.

| Property | Value |
|----------|-------|
| Mosaic dimensions | 19,000 x 17,000 pixels |
| Geographic extent | 19 x 17 km (323 km^2) |
| Elevation range | 23.2 - 69.2 m |
| Mean elevation | 38.4 m |
| Valid data coverage | 58.5% (tiles form an irregular shape) |

The low total relief (~46 m) confirmed this is a low-relief wetlandscape where 1m LIDAR captures subtle topography that 90m MERIT would miss.

### Tile Reference System

A grid reference system was created for identifying tiles during review and selection:

- **Format:** `R{row}C{col}` (e.g., R5C7)
- **Grid dimensions:** 17 rows x 19 columns (233 of 323 possible positions occupied)
- **Coordinate mapping:** Easting = 394000 + col * 1000; Northing = 3292000 - row * 1000
- **KML export** available for Google Earth overlay

---

## Resolution Challenges

Adapting from 90m MERIT to 1m LIDAR introduced several issues that required explicit handling:

| Issue | Impact at 1m | Adaptation |
|-------|-------------|------------|
| FFT noise | Spectral analysis detects microtopography, not drainage patterns | Added minimum Lc constraint (100m) |
| Memory | 323M pixels exceeds pysheds `resolve_flats` capacity | 4x subsampling for flow routing |
| DTND formula | pysheds `compute_hand()` uses haversine (expects lat/lon, not UTM) | Replaced with `scipy.ndimage.distance_transform_edt` |
| Edge blending | Original 4-pixel FFT window = 360m at 90m but only 4m at 1m | Increased to 50 pixels (50m) |
| Hardcoded thresholds | `smallest_dtnd=1.0m` masks fine-scale drainage at 1m res | Noted; acceptable for initial runs |

---

## Small-Scale Smoke Test

A 4x4 km subset (4000 x 4000 pixels, 100% valid data) was extracted to verify the pipeline before committing to the full mosaic.

### Issues Found and Fixed

**Issue 1: FFT detects noise at 1m resolution.** The raw characteristic length scale was Lc = 6m, picking up high-frequency surface texture rather than drainage-scale topography. A `min_lc_pixels` parameter was added to enforce a minimum of 100m, producing a constrained Lc = 100m (100 pixels).

**Issue 2: pysheds DTND uses haversine formula.** The pysheds `compute_hand()` method calculates DTND using haversine distance, which assumes geographic coordinates in degrees. For UTM coordinates in meters, haversine produced distances of 500+ km for a 4 km domain. The fix replaced DTND computation with `scipy.ndimage.distance_transform_edt`, which computes Euclidean distance directly in the UTM coordinate system.

### Results

| Metric | Value |
|--------|-------|
| Characteristic length (Lc) | 100 m (constrained) |
| Accumulation threshold | 5000 cells |
| Stream coverage | 0.88% |
| HAND median | 1.3 m |
| DTND median | 33 m |
| Aspect distribution | N: 17.7%, E: 34.5%, S: 17.3%, W: 30.5% |

All 16 hillslope elements were successfully computed. Runtime was 85 seconds.

---

## Full Mosaic Processing

### Debugging Journey

Processing the full 17,000 x 19,000 pixel mosaic required four attempts to resolve a fundamental interaction between pysheds flow routing and the irregular NEON tile coverage.

**Attempt 1: Out of memory.** The `resolve_flats` step in pysheds exceeded 64 GB when processing the full DEM at 1m resolution. This step has O(n^2) or worse complexity for large flat regions.

**Attempt 2: max_accumulation = 1.** With 128 GB memory and nodata pixels treated as natural drainage boundaries, flow accumulation peaked at 1 -- every pixel drained immediately to a nodata boundary before accumulating any flow.

**Attempt 3: max_accumulation = 1 (still).** Extracting the largest connected component of valid data and 4x subsampling did not help. Flow accumulation was still 1 despite individual centered subregions (6000x6000, 8000x8000, etc.) producing accumulations in the millions.

**Root cause discovery:** Analysis of the extracted region's edges revealed the problem:

```
top:    0 valid, 4395 nodata
bottom: 0 valid, 4395 nodata
left:   0 valid, 4081 nodata
right:  0 valid, 4081 nodata
```

**All four edges were 100% nodata.** The bounding box of the largest connected component included the irregular boundary of NEON tile coverage, producing all-nodata edges. pysheds flow routing requires flow to exit the domain at boundary cells. With all edges masked, no cell could accumulate flow from others -- every cell only counted itself.

!!! warning "pysheds edge handling"
    Flow routing algorithms (D8, Dinf) require at least some valid data on domain edges for flow to exit. When all boundary cells are nodata, pysheds creates a closed basin where max_accumulation = 1. This is a silent failure -- no error is raised.

**Attempt 4: Success.** After subsampling and connected component extraction, nodata-only rows and columns were trimmed from the edges:

```python
rows_valid = np.where(np.any(valid_mask_sub, axis=1))[0]
cols_valid = np.where(np.any(valid_mask_sub, axis=0))[0]
dem_sub = dem_sub[rows_valid[0]:rows_valid[-1]+1,
                  cols_valid[0]:cols_valid[-1]+1]
```

This produced max_accumulation = 658,954, confirming the fix.

### Processing Pipeline

The final pipeline for the full mosaic:

1. Load mosaic and identify valid data
2. Extract largest connected component (`scipy.ndimage.label`)
3. Subsample by 4x for flow routing (1m -> 4m effective resolution)
4. Trim all-nodata edges
5. Condition DEM, compute flow direction and accumulation
6. Compute HAND and Euclidean DTND
7. Compute slope and aspect from **original** DEM (not pysheds-processed; see below)
8. Bin by aspect (4 bins) and elevation (4 HAND bins)
9. Calculate six geomorphic parameters per bin
10. Output NetCDF in CTSM-compatible format

---

## Slope Calculation Bug

Initial runs produced impossible slope values (up to 783 million m/m). Three approaches were tried before finding the correct solution:

1. **pgrid's `slope_aspect()`** -- assumed geographic coordinates, not UTM
2. **Horn 1981 averaging from `spatial_scale.py`** -- designed for FFT analysis, not slope calculation
3. **`np.gradient()` on pysheds-processed DEM** -- the problem

!!! warning "pysheds DEM conditioning replaces nodata"
    pysheds replaces nodata values with `max_elevation + 1` to prevent flow through gaps. This creates massive false gradients at every nodata boundary. Any slope calculation must use the **original** DEM with nodata as NaN, not the pysheds-processed version.

The fix stored the original DEM with nodata as NaN before pysheds processing:

```python
dem_for_slope = dem_sub.copy().astype(float)
dem_for_slope[~valid_mask] = np.nan
# ... later ...
dzdy, dzdx = np.gradient(dem_for_slope, pixel_size)
slope = np.sqrt(dzdx**2 + dzdy**2)
```

NaN values propagate correctly through gradient calculation, preventing false gradients at boundaries.

---

## Algorithm Differences from Swenson

| Component | Swenson (90m MERIT) | OSBS (1m LIDAR) | Reason |
|-----------|---------------------|-----------------|--------|
| DTND calculation | Haversine (geographic) | Euclidean (EDT) | UTM coordinates are already in meters |
| Processing resolution | Full resolution | 4x subsampled | Memory/compute constraints for `resolve_flats` |
| Edge handling | Not needed | Trim nodata edges | NEON tiles have irregular coverage |
| Lc determination | FFT natural peak | Constrained >= 100m | 1m data picks up noise/microtopography |
| Connected component | Not needed | Extract largest | Scattered tiles create disconnected regions |
| Slope calculation | `slope_aspect()` on geographic DEM | `np.gradient()` on original DEM with NaN | UTM coords; must avoid pysheds nodata fill |

---

## Interior Tile Selection

The full mosaic included edge tiles with urban areas and irregular coverage. Processing was run in two modes for comparison:

- **Full (233 tiles):** All available NEON tiles
- **Interior (150 tiles):** Tiles selected to exclude the irregular outer boundary, producing a more contiguous study region

The interior selection excluded rows 0-3 (except R3C11-13 and R4C5-14), the northwest corner, and sparse southern/eastern edges, retaining 150 tiles with more uniform coverage.

### Key Difference: Lc

The interior mosaic produced an FFT-derived Lc of **166 m** (vs the 100 m minimum constraint for the full mosaic). With less edge noise, the FFT could identify a natural spectral peak. The higher Lc produced a higher accumulation threshold (864 vs 312 cells) and sparser stream network (1.44% vs 2.32%).

---

## NetCDF Output

The pipeline outputs CTSM-compatible NetCDF files with all required hillslope variables.

### Dimensions

| Dimension | Size | Description |
|-----------|------|-------------|
| `lsmlat` | 1 | Single gridcell latitude |
| `lsmlon` | 1 | Single gridcell longitude |
| `nhillslope` | 4 | Aspect bins (N, E, S, W) |
| `nmaxhillcol` | 16 | Total hillslope columns |

### Key Variables

| Variable | Units | Description |
|----------|-------|-------------|
| `nhillcolumns` | -- | Number of hillslope columns (16) |
| `pct_hillslope` | % | Area fraction per aspect |
| `hillslope_elevation` | m | Height above stream (HAND) |
| `hillslope_distance` | m | Distance from stream (DTND) |
| `hillslope_width` | m | Hillslope width at downslope edge |
| `hillslope_area` | m^2 | Hillslope element area |
| `hillslope_slope` | m/m | Topographic slope |
| `hillslope_aspect` | radians | Azimuthal orientation (0-2pi, clockwise from N) |

Key conversions applied: aspect from degrees to radians, longitude from -82deg to 278deg (0-360 convention for CTSM).

---

## Comparison: Full vs Interior

| Metric | Full (233 tiles) | Interior (150 tiles) |
|--------|------------------|----------------------|
| DEM shape | 17k x 19k | 15k x 16k |
| Valid data | 58.5% | 62.5% |
| Characteristic Lc | 100 m (constrained) | 166 m (FFT-derived) |
| Accumulation threshold | 312 cells | 864 cells |
| Stream coverage | 2.32% | 1.44% |
| HAND median | 1.1 m | 1.8 m |
| Aspect distribution | N: 17%, E: 34%, S: 17%, W: 32% | N: 24%, E: 26%, S: 26%, W: 24% |
| Processing time | 6.2 min | 2.9 min |

!!! note "Aspect distribution"
    The full mosaic showed a strong E/W bias (66% combined), likely caused by edge artifacts or non-representative terrain at the boundary. The interior mosaic produced a much more uniform aspect distribution (~25% per direction), consistent with expectations for gently rolling terrain without a dominant orientation.

---

## Technical Lessons

1. **pysheds edge handling is critical.** Flow routing silently fails when all domain edges are nodata. Always verify that at least some boundary cells contain valid data.

2. **Subsampling is necessary for large DEMs.** The `resolve_flats` step in pysheds has poor scaling with large flat regions. 4x subsampling reduced the problem to manageable size while preserving drainage patterns at scales relevant to hillslope parameterization.

3. **Connected component extraction alone is insufficient.** Even after isolating the largest connected region, the bounding box may have all-nodata edges due to irregular tile coverage. Edge trimming is a separate, required step.

4. **Accumulation threshold must scale with resolution.** When subsampling, both Lc (in pixels) and the derived threshold must be adjusted: `threshold = 0.5 * (Lc / subsample)^2`.

5. **Geographic bounds must track all transformations.** Edge trimming, subsampling, and region extraction all modify the geographic extent. Bounds must be updated at each step for correct georeferencing in the output NetCDF.

6. **Use the original DEM for slope calculation.** pysheds DEM conditioning replaces nodata with high values that create false gradients. Store the original DEM with NaN for nodata before processing.
