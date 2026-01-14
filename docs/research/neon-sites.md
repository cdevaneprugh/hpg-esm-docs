# NEON Sites

*Documentation for NEON (National Ecological Observatory Network) site simulations.*

---

## Overview

NEON provides standardized, high-quality observational data across 81 field sites in the United States. For CTSM validation, NEON offers:

- **Flux tower data**: Carbon, water, and energy fluxes
- **Meteorological forcing**: Required for atmospheric data model (DATM)
- **Soil surveys**: Profiles, texture, organic carbon
- **Vegetation surveys**: Species composition, biomass
- **LIDAR topography**: 1m resolution DEMs for many sites

### CTSM Integration

CTSM includes built-in support for NEON sites:

- Pre-configured site definitions
- Automated data download scripts
- Standardized forcing data processing

See [Single-Point Runs](../running-ctsm/single-point.md) for creating site simulations.

---

## Primary Site: OSBS

**Ordway-Swisher Biological Station** is the primary test site for hillslope hydrology development.

### Site Characteristics

| Attribute | Value |
|-----------|-------|
| **NEON Site ID** | OSBS |
| **Location** | Putnam County, North-central Florida |
| **Coordinates** | 29.69°N, 82.00°W |
| **Network** | NEON Domain 3 (Southeast), Ameriflux |
| **Ecosystem** | Sandhills with wetland depressions |
| **Dominant Vegetation** | Longleaf pine, wiregrass, oak scrub |

### Hydrologic Setting

| Attribute | Value |
|-----------|-------|
| **Mean Precipitation** | ~1450 mm/yr |
| **PET** | ~1300 mm/yr |
| **Wetland Coverage** | ~19% |
| **Connectivity** | Primarily groundwater |
| **Surface Connection** | Rare |

### Why OSBS for Hillslope Research

OSBS represents a **low-relief wetlandscape** - the exact setting where hillslope hydrology matters most:

1. **Subtle topography**: Elevation differences of meters (not hundreds of meters) drive ecosystem patterns
2. **TAI dynamics**: Wetland-upland boundaries shift seasonally
3. **Groundwater-driven**: Lateral subsurface flow dominates over surface connectivity
4. **High-quality data**: Extensive NEON instrumentation and 1m LIDAR

From Fan et al. 2019:

> "In humid and low-relief regions where water is in excess, lateral drainage is also important... the slightly elevated hills can improve local drainage and alleviate waterlogging."

OSBS fits "Position 3" in Fan 2019 Figure 6a (ever-wet, low relief) where hillslope representation improves drainage representation on subtle mounds.

---

## Comparison Site: BEF

**Bradford Experimental Forest** is a comparison site with different connectivity characteristics.

| Attribute | OSBS | BEF |
|-----------|------|-----|
| **Ecosystem** | Sandhills | Pine flatwoods |
| **Wetland Type** | Isolated depressions | Cypress wetlands |
| **Connectivity** | Groundwater | Surface streams |
| **Surface Flow** | Rare | Flashy blackwater |
| **Wetland Coverage** | ~19% | ~25% |

BEF has higher surface connectivity and may be addressed in future work after OSBS hillslope representation is validated.

---

## Data Extraction

### NEON Data Portal

Primary data access: [NEON Data Portal](https://data.neonscience.org/)

Key data products for CTSM:

| Product | Code | Use |
|---------|------|-----|
| Eddy covariance | DP4.00200.001 | Flux validation |
| Meteorological | DP1.00002/3 | Forcing data |
| Soil profiles | DP1.10047.001 | Parameter validation |
| LIDAR DEM | DP3.30024.001 | Custom hillslope data |

### Built-in CTSM Scripts

```bash
# NEON-specific automation
$CTSMROOT/tools/site_and_regional/run_neon

# General tower site script
$CTSMROOT/tools/site_and_regional/run_tower
```

---

## Subset Data

Pre-generated subset data structure:

```
/blue/<group>/earth_models/shared.subset.data/
├── OSBS/
│   ├── surfdata_*.nc           # Surface dataset
│   ├── datmdata/               # Atmospheric forcing
│   │   └── atm_forcing.datm.CRUNCEP.*/
│   └── user_mods/              # Namelist modifications
│       ├── shell_commands
│       ├── user_nl_clm
│       └── user_nl_datm_streams
└── .../
```

### Current Data Strategy

- **Development**: Global data subset via `run_tower` script
- **Production goal**: Maximize use of NEON-provided local data

---

## Custom Hillslope Data for OSBS

The global Swenson hillslope dataset (~90m source resolution) is too coarse for OSBS. Custom data from 1m NEON LIDAR will provide:

- Actual wetland basin morphology
- Fine-scale drainage network identification
- TAI transition zone resolution

**Expected differences from global data:**

- Smaller characteristic length scale (Lc)
- Lower HAND values (meters, not tens of meters)
- Different aspect distribution (may not have 4 distinct hillslopes if flat)
- More accurate TAI representation

**Workflow:**

1. Download 1m LIDAR DEM from NEON (DP3.30024.001)
2. Apply Swenson 2025 methodology using pysheds
3. Generate custom hillslope surface data
4. Run comparison: global vs custom hillslope parameters

---

## Reference Cases

| Case | Purpose | Status |
|------|---------|--------|
| `osbs2.branch.spillheight/` | Spillheight mechanism testing | Development |
| `osbs2.branch.v2/` | Development branch | Active |
| `osbs2.branch.v3/` | Development branch | Active |

---

*See [Single-Point Runs](../running-ctsm/single-point.md) for creating site simulations.*
