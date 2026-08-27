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

OSBS production cases currently use globally-subset **CRUNCEPv7** atmospheric forcing and surface data extracted via CTSM's `subset_data` script (in `tools/site_and_regional/`). A **site-native NEON forcing dataset** has since been produced (Phase I): a fork of the NCAR-NEON pipeline, run fully offline on HiPerGator, generates the OSBS tower record as 101 monthly DATM files spanning 2017-02 → 2025-06. It is engineering-complete, validated against the pre-built NCAR-NEON v4 product, and available for adoption in production. See [NEON Atmospheric Forcing](../swenson/neon-forcing.md) for the full methodology and the `dtlimit = -1` requirement for cycled runs.

An earlier report that the pre-built `run_tower` NEON forcing was "insufficient" was traced to a namelist year-cap, not a data limit — the pre-built product reaches 2024-12. `run_tower` remains useful for basic NEON site tests, but for OSBS production forcing it is superseded by the offline pipeline above.

---

## Custom Hillslope Data for OSBS

The global Swenson hillslope dataset (~90 m source resolution) is too coarse for OSBS. Custom hillslope parameters were generated from 1 m NEON LIDAR (DP3.30024.001) using the Swenson & Lawrence (2025) methodology, resolving:

- Actual wetland basin morphology
- Fine-scale drainage network identification
- TAI transition zone resolution

The finished parameters bear out the contrasts anticipated against the global data: a much smaller characteristic length scale (**Lc = 356 m**), low HAND values (meters, not tens of meters), and a single-aspect structure rather than four distinct hillslopes. The production file `hillslopes_osbs_production_c260505.nc` (25 columns: 1 lake + 24 land bins) is deployed in the operative spinup case.

This work is documented in full in the **Swenson Implementation** section — see [Overview](../swenson/index.md), [Methodology](../swenson/osbs-implementation.md), and [HAND Binning and Lake Column](../swenson/hand-binning-and-lake-column.md).

---

## Reference Cases

| Case | Role | Configuration |
|------|------|---------------|
| `osbs.swenson.spinup` | **Operative** — 600-yr accelerated AD spinup | `use_hillslope=.true.`, `use_hillslope_routing=.false.`; 2026-05-05 production hillslope file; 4-stream h0/h1/h2/h3 |
| `osbs.swenson.post-ad` | **Secondary** — 200-yr post-AD continuation | Completed 2026-05-21; idle pending PI investigation of Phase F open questions |
| `osbs2` | Reference — 860+ yr spinup with the global Swenson hillslope file | Source of restart data for early branch-case tests |

The earlier `osbs2.branch.spillheight`, `.v2`, and `.v3` cases exercised the runtime `SPILLHEIGHT` SourceMod retired 2026-04-30 (superseded by raw-HAND binning at file-construction time) and are no longer active.

---

*See [Single-Point Runs](../running-ctsm/single-point.md) for creating site simulations.*
