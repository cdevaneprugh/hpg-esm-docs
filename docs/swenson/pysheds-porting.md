# pysheds Porting

*Porting Swenson's hillslope methods to our own pysheds fork with NumPy 2.0 compatibility.*

---

## Fork Setup

A fork of [pysheds](https://github.com/mdbartos/pysheds) was created at `$BLUE/pysheds_fork` with two branches:

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

Swenson created `pgrid.py` (~4,200 lines) -- a pure-Python Grid class containing all hillslope-specific methods not present in upstream pysheds. This file was the core dependency for the entire hillslope parameter calculation pipeline.

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

- **Copy entire `pgrid.py`** -- not surgical porting of individual methods
- **Pure Python** -- no numba optimization (Swenson's code uses numba optionally)
- **Match API signatures** -- maintain compatibility with `Representative_Hillslopes` scripts

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

Swenson's fork included a pytest test suite (`test_grid.py`, ~40 tests) covering core pysheds functionality but not the hillslope-specific methods. A new test file (`test_hillslope.py`) was created to cover the hillslope additions.

### Test Classes

| Class | Tests | Coverage |
|-------|-------|----------|
| `TestSlopeAspect` | 4 | Slope/aspect calculation on synthetic and real DEMs |
| `TestChannelMask` | 3 | Channel mask creation, ID assignment, bank classification |
| `TestComputeHand` | 2 | Extended HAND with DTND and drainage ID |
| `TestComputeHillslope` | 2 | Hillslope classification (left/right bank, headwater) |
| `TestExtractProfiles` | 2 | River segment extraction with connectivity |
| `TestIntegration` | 1 | Full pipeline workflow on test DEM |

### Results

```
================== 14 passed, 1 skipped, 33 warnings in 3.63s ==================
```

All hillslope-specific methods verified working.

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
