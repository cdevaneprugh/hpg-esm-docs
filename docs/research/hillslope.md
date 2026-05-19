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

Hillslope mode partitions each gridcell into an arbitrary number of columns at different topographic positions. The cardinality is read at runtime from the hillslope input NetCDF (`nhillslope` × `nmaxhillcol`), so the same CTSM build supports varied column-structure decisions.

The Swenson global default uses 4 aspects × 4 elevation bins = 16 columns:

```
Gridcell (Swenson default)
├── North-facing hillslope (4 columns: Outlet → Lower → Upper → Ridge)
├── East-facing hillslope (4 columns)
├── South-facing hillslope (4 columns)
└── West-facing hillslope (4 columns)
```

The OSBS production file departs from this default, using 1 aspect × 24 elevation bins + 1 dedicated lake column = 25 columns. The aspect partition is collapsed because OSBS slopes are too shallow (median ~0.05 m/m, max ~0.06 m/m) for aspect-dependent insolation to carry a meaningful signal; the 24 elevation bins resolve the wet-to-dry gradient at the LIDAR vertical noise floor. See [HAND Binning and Lake Column](../swenson/hand-binning-and-lake-column.md) for the OSBS rationale.

Stream channel properties (depth, width, slope) are stored at the hillslope landunit level in either configuration, not as a separate column.

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

CTSM hillslope mode is controlled by two related-but-distinct toggles. A 2026-05-19 source-code audit of CTSM 5.3.085 established what each one activates; the [Lateral Flow and Routing](../swenson/lateral-flow-and-routing.md) page documents the file:line evidence.

### `use_hillslope`

```fortran
use_hillslope = .true.
```

Activates **inter-column lateral subsurface flow**. With this toggle on, `PerchedLateralFlow` and `SubsurfaceLateralFlow` (`SoilHydrologyMod.F90`) are dispatched unconditionally from `HydrologyDrainageMod.F90`; the Darcy gradient between adjacent columns is computed every hydrology step, and the net inflow/outflow is applied to each column's soil-water state. This is what makes a hillslope-mode case differ physically from a stack of independent 1-D soil columns.

### `use_hillslope_routing`

```fortran
use_hillslope_routing = .true.
```

Additionally activates the **stream-side machinery**: an internal `stream_water_volume` ledger, Manning's-equation streamflow at the chain bottom, a stream-channel boundary condition that supersedes MOSART's `tdepth_grc` at the terminal column, the `VOLUMETRIC_STREAMFLOW` history field, and the land-to-runoff streamflow export. Routing-on does **not** activate inter-column lateral flow — that already runs under `use_hillslope` alone.

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
