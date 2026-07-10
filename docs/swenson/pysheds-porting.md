# pysheds Fork

*Swenson's hillslope methods ported to our own pysheds fork, extended with UTM CRS support.*

---

## Fork Setup

A fork of [pysheds](https://github.com/mdbartos/pysheds) was created at [cdevaneprugh/pysheds](https://github.com/cdevaneprugh/pysheds) with two branches:

| Branch | Purpose |
|--------|---------|
| `master` | Synced with upstream (`mdbartos/pysheds`) |
| `uf-development` | Development branch with Swenson's hillslope methods |

Verification:

```bash
pysheds-env
python -c "from pysheds.pgrid import Grid; print('OK')"
```

---

## What Swenson Added

Swenson created `pgrid.py` (~4,200 lines) — a pure-Python Grid class containing all hillslope-specific methods not present in upstream pysheds. This file was the core dependency for the entire hillslope parameter calculation pipeline.

### Core Hillslope Methods

| Method | Purpose |
|--------|---------|
| `compute_hand()` | Extended HAND + DTND + AZND + drainage ID assignment |
| `compute_hillslope()` | Classify pixels as headwater, left bank, right bank, or channel |
| `slope_aspect()` | Calculate slope and aspect from DEM using Horn (1981) 8-neighbor stencil |
| `extract_profiles()` | Delineate river segments with connectivity information |
| `river_network_length_and_slope()` | Calculate network length and mean slope |
| `create_channel_mask()` | Create channel mask, IDs, and bank classification masks |

### Supporting Methods

| Method | Purpose |
|--------|---------|
| `_2d_geographic_coordinates()` | Generate 2D lon/lat arrays from affine transform |
| `_gradient_horn_1981()` | Horn (1981) gradient calculation for slope and aspect |
| `_translate_dict()` | Map flow direction values to array indices |

### What Upstream pysheds Lacks

| Feature | Status in Upstream |
|---------|-------------------|
| DTND (Distance To Nearest Drainage) | Not implemented |
| AZND (Azimuth To Nearest Drainage) | Not implemented |
| Drainage ID per pixel | Not implemented |
| Hillslope classification (L/R bank, headwater) | Not implemented |
| Slope/Aspect from DEM | Not implemented |
| Channel mask with IDs | Not implemented |
| River network length/slope | Not implemented |

Upstream provides flow direction, flow accumulation, catchment delineation, and basic HAND (height only), but none of the hillslope-specific processing needed for Swenson's methodology.

---

## Implementation Strategy

The approach was to copy the entire `pgrid.py` from Swenson's fork rather than surgically porting individual methods. This preserved internal dependencies and ensured API compatibility with Swenson's `Representative_Hillslopes` processing pipeline.

Key decisions:

- **Copy entire `pgrid.py`** — not surgical porting of individual methods
- **Pure Python** — no numba optimization (Swenson's code uses numba optionally)
- **Match API signatures** — maintain compatibility with `Representative_Hillslopes` scripts

The `grid.py` entry point was updated to fall back to `pgrid` when numba is unavailable, matching Swenson's import pattern.

---

## NumPy 2.0 Compatibility Fixes

Swenson's code was written for NumPy 1.x. Four categories of fixes were required:

| Issue | Occurrences | Fix |
|-------|-------------|-----|
| `np.warnings` deprecated | 12 | Replaced with `warnings` module |
| `np.bool` deprecated | 13 | Replaced with `bool` |
| `np.float` deprecated | 2 | Replaced with `np.float64` |
| Unsigned integer overflow in `_flatten_fdir` | 1 | Added `_signed_mintype()` helper function |

---

## Test Suite

The test suite covers both the original hillslope methods and the UTM CRS additions across four test files:

| File | Tests | Coverage |
|------|-------|----------|
| `test_hillslope.py` | 15 | Hillslope methods on geographic DEM (slope/aspect, channel mask, HAND, hillslope classification, river network, integration) |
| `test_utm.py` | 22 | UTM CRS: slope, aspect, HAND, DTND, AZND, river network, classification on synthetic V-valley DEM |
| `test_grid.py` | ~40 | Core pysheds functionality (pre-existing) |
| `test_dem.py` | ~5 | DEM I/O and conditioning (pre-existing) |

The UTM tests use a synthetic V-valley DEM (1000x1000, 5m pixel spacing, UTM CRS) with analytically known slope, aspect, HAND, and DTND values. The 5m pixel size prevents tests from silently passing at unit pixel spacing.

### Results

```
================== 82 passed, 0 warnings ==================
```

Mutation testing: 30 mutations applied to CRS-dependent code paths, 100% effective score (all mutations caught or functionally equivalent).

---

## API Notes

The `pgrid` API uses an **inplace pattern** by default:

- Methods store results as grid attributes (e.g., `grid.slope`, `grid.aspect`)
- Methods return `None` when `inplace=True` (the default)
- Access results via attributes after calling methods

```python
grid.slope_aspect("dem")
slope = np.array(grid.slope)
aspect = np.array(grid.aspect)
```

This pattern applies to `compute_hand()`, `compute_hillslope()`, `slope_aspect()`, and other methods.

CRS handling is transparent — methods detect the grid's CRS automatically and use the appropriate math (haversine for geographic, Euclidean for projected). The caller does not need to specify or know the CRS type.

---

## UTM CRS Support

### Problem

pysheds assumes geographic (lat/lon) coordinates throughout. On UTM data, haversine formulas produce garbage distances (e.g., treating UTM easting 404000 as 404000 degrees longitude), and the Horn 1981 gradient uses incorrect pixel spacing derived from haversine on meter-valued coordinates.

### Solution

A `_crs_is_geographic()` helper detects the CRS from the grid's pyproj metadata. All CRS-dependent functions branch: the geographic code path uses haversine (unchanged from Swenson's original), the projected code path uses Euclidean distance in the CRS linear units.

### Functions Modified

| Function | Geographic path | Projected path |
|----------|----------------|----------------|
| `compute_hand()` DTND | Haversine distance | Euclidean distance |
| `compute_hand()` AZND | Spherical bearing | Planar arctan2 |
| `_gradient_horn_1981()` | Haversine + cos(lat) spacing | Uniform dx, dy from affine transform |
| `river_network_length_and_slope()` | Haversine segment length | Euclidean segment length |
| `flow_direction()` D8 gradient | Haversine + cos(lat) spacing | Uniform dx, dy from affine transform |

### Validation

Three-level validation confirms correctness of both code paths:

1. **Synthetic V-valley DEM** (1000x1000, 5m pixel, UTM CRS): Analytically known slope, aspect, HAND, and DTND. 22 tests catch CRS math errors directly against known solutions.

2. **MERIT geographic regression**: Confirms UTM additions did not perturb the geographic code path. All 6 hillslope parameters match baseline correlations within 0.01 tolerance.

3. **R6C10 single-tile smoke test**: Real 1m NEON LIDAR with lake, swamp, and upland terrain. 14 validation checks pass (stream coverage, HAND range, DTND range, slope statistics, aspect distribution, etc.).

---

## Additional Improvements

### Deprecation Fixes

| Issue | Fix |
|-------|-----|
| `distutils.version.LooseVersion` removed in Python 3.12 | Replaced with `looseversion` package |
| `np.in1d` deprecated | Replaced with `np.isin` |
| `pd._append()` deprecated | Replaced with `pd.concat()` |
| `np.warnings`, `np.bool`, `np.float` (NumPy 2.0) | See NumPy 2.0 section above |

### Code Cleanup

- `_propagate_uphill()` extracted from three duplicate loops into a single reusable function
- Module-level constants: `_EARTH_RADIUS_M`, `_DEG_TO_RAD` replace scattered magic numbers
- Bare `except:` clauses replaced with specific exception types

### CRS-Neutral Naming

- `_2d_geographic_coordinates()` renamed to `_2d_crs_coordinates()`
- `lon2d`/`lat2d` renamed to `x2d`/`y2d` throughout CRS-dependent functions
