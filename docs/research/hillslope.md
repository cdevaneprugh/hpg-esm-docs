# Hillslope Hydrology

*CTSM's hillslope hydrology mode enables representation of subgrid topography with lateral water flow between landscape positions.*

---

## Overview

Traditional ESM land models use a **1-D (vertical) hydrology** approach - each gridcell is treated as a flat slab with no internal terrain structure. This misses critical landscape processes:

- Drier hills, wetter valleys
- Ridge-to-valley groundwater convergence
- Delayed water delivery (temporal carryover)
- Aspect-dependent radiation differences

CTSM's **hillslope hydrology** mode addresses these limitations by dividing each gridcell into multiple columns at different topographic positions connected by lateral water flow.

---

## Hillslope Structure

### Column Organization

Each gridcell contains 17 columns in hillslope mode:

```
Gridcell
├── Stream column (1)
├── North-facing hillslope (4 columns)
│   ├── Outlet (lowest, near stream)
│   ├── Lower
│   ├── Upper
│   └── Ridge (highest)
├── East-facing hillslope (4 columns)
├── South-facing hillslope (4 columns)
└── West-facing hillslope (4 columns)
```

### Column Indices

```
Column 1:  Stream
Column 2-5:  North aspect (Outlet → Ridge)
Column 6-9:  East aspect (Outlet → Ridge)
Column 10-13: South aspect (Outlet → Ridge)
Column 14-17: West aspect (Outlet → Ridge)
```

---

## Six Geomorphic Parameters

Each hillslope element is defined by six parameters (Swenson & Lawrence 2025):

| Parameter | Symbol | Description |
|-----------|--------|-------------|
| **Area** | A | Horizontally projected surface area |
| **Height** | h | Mean height above stream channel (HAND) |
| **Distance** | d | Mean distance from channel (DTND) |
| **Width** | w | Width at downslope interface |
| **Slope** | α | Mean topographic slope |
| **Aspect** | β | Azimuthal orientation (from North) |

### HAND and DTND

Two key metrics organize the hillslope:

- **HAND** (Height Above Nearest Drainage): Elevation relative to nearest stream pixel
- **DTND** (Distance To Nearest Drainage): Horizontal distance to nearest stream pixel

These define the hillslope profile (height vs distance from stream).

---

## Physical Processes

### Three Hillslope-Enabled Processes

1. **Lateral subsurface flow**: Water moves between columns based on hydraulic gradient using Darcy's Law
2. **Aspect-dependent insolation**: Solar radiation varies by slope and aspect
3. **Elevation downscaling**: Temperature and precipitation adjusted by elevation

### Why Darcy's Law Matters

The paper emphasizes using **Darcy's Law** instead of kinematic wave for lateral flow:

- Kinematic wave uses constant terrain slope
- Darcy's Law captures negative feedbacks:
    - High water table → accelerated drainage
    - Low water table → decelerated drainage
- Preserves deep soil water storage
- Captures delayed hill-to-valley transfer

### Two-Way Surface-Groundwater Exchange

Rivers can both gain and lose water:

- **Gaining**: Higher water table feeds stream
- **Losing**: Stream water infiltrates to groundwater
- **Dynamic connectivity**: Channel network expands/contracts with conditions

---

## Current Data

### Global Hillslope Dataset (Swenson 2025)

The default global dataset was created from MERIT DEM (~90m resolution):

- Resolution: ~1° globally
- Source: MERIT DEM
- **Limitation**: Too coarse for low-relief wetlands

The paper explicitly notes:

> "The MERIT DEM... may not be fine enough to capture topographic variations in areas of **very low topographic relief, such as wetlands**."

### Custom Data for Low-Relief Sites

For sites like OSBS, custom hillslope parameters from high-resolution LIDAR provide:

- 90x finer resolution than global dataset
- Capture of wetland basin morphology
- Identification of actual stream/drainage networks
- Resolution of TAI transition zones

**Workflow for custom data:**

1. Obtain 1m LIDAR DEM (e.g., from NEON)
2. Apply Laplacian spectral analysis to find characteristic length scale
3. Use pysheds for catchment delineation
4. Calculate HAND/DTND at high resolution
5. Discretize and average to create hillslope parameters
6. Format for CTSM surface dataset

**Tool:** [Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes)

---

## Relevance to Wetlandscapes

### Low-Relief Wetlands

In humid, low-relief regions (like OSBS), lateral drainage matters for different reasons than mountains:

> "In humid and low-relief regions where water is in excess, lateral drainage is also important but for different reasons. Here regional drainage is impeded, resulting in waterlogged soils and oxygen stress for plants."

Slightly elevated mounds improve local drainage and alleviate waterlogging, supporting upland vegetation.

### TAI Dynamics

The Terrestrial-Aquatic Interface (TAI) is the dynamic boundary between wet and dry areas:

- TAI position shifts with water level changes
- TAI dynamics drive carbon flux "hot spots" and "hot moments"
- Hillslope mode can represent these transitions via water table position in different columns

---

## Key Namelist Parameters

### Enabling Hillslope Mode

```fortran
use_hillslope = .true.
use_hillslope_routing = .true.
```

### Spillheight

The spillheight parameter controls surface water redistribution when water table exceeds a threshold:

```fortran
hillslope_pft_spill_height_to_bedrock_frac = 0.5
```

---

## Analysis Scripts

Analysis scripts are available in [hpg-esm-tools](https://github.com/cdevaneprugh/hpg-esm-tools):

```
scripts/hillslope.analysis/
├── bin_temporal.sh                    # Temporal binning (N-year averages)
├── plot_timeseries_*.py               # Time series plots
├── plot_zwt_hillslope_profile.py      # Water table vs elevation
├── plot_elevation_width_overlay.py    # Hillslope geometry
├── plot_col_areas.py                  # Column area bar charts
├── plot_pft_distribution.py           # PFT distribution
└── plot_vr_profile.py                 # Vertical profiles
```

---

## History Output

### Required History Streams

| Stream | Content | Use |
|--------|---------|-----|
| h0 | Gridcell averages | Standard CLM output |
| h1 | Column-level data | Hillslope column analysis |
| h2 | PFT-level data | Vegetation analysis |

### Key Variables for Hillslope Analysis

| Variable | Description |
|----------|-------------|
| ZWT | Water table depth |
| ZWT_PERCH | Perched water table depth |
| QOVER | Surface runoff |
| QDRAI | Subsurface drainage |
| QCHARGE | Aquifer recharge |
| QFLX_LAT_AQU | Lateral aquifer flux |
| TWS | Total water storage |
| TSOI | Soil temperature profile |
| H2OSOI | Soil moisture profile |

---

## Key References

- **Fan et al. 2019**: "Hillslope Hydrology in Global Change Research and Earth System Modeling" - Scientific foundation
- **Swenson & Lawrence 2025**: "Development of a Global Representative Hillslope Data Set" - Dataset methodology

Summaries in `hpg-esm-tools/docs/papers/`.

---

*See [History Output](../running-ctsm/history-output.md) for configuring output variables.*
