# Hillslope Hydrology

*Documentation for hillslope hydrology research and CTSM modifications.*

---

**Status:** Placeholder - content to be added.

## Overview

<!-- TODO: Document hillslope hydrology research -->
<!-- - What is hillslope mode in CTSM -->
<!-- - Research questions being addressed -->
<!-- - Key parameters (spillheight, etc.) -->

## Hillslope Structure

<!-- TODO: Document column structure -->
<!-- - 4 hillslopes per gridcell (N, E, S, W aspects) -->
<!-- - 4 elevation positions per hillslope (Outlet, Lower, Upper, Ridge) -->
<!-- - 16 hillslope columns + 1 stream column -->

## Key Parameters

<!-- TODO: Document namelist parameters -->
<!-- - spillheight -->
<!-- - Other hillslope-specific settings -->

## Analysis Scripts

Analysis scripts are available in [hpg-esm-tools](https://github.com/cdevaneprugh/hpg-esm-tools):

```
scripts/hillslope.analysis/
├── bin_temporal.sh           # Temporal binning
├── plot_timeseries_*.py      # Time series plots
├── plot_zwt_hillslope_profile.py  # Water table profiles
├── plot_elevation_width_overlay.py # Hillslope geometry
└── plot_col_areas.py         # Column areas
```

## History Output

Relevant variables for hillslope analysis:

| Variable | Description |
|----------|-------------|
| `ZWT` | Water table depth |
| `QRUNOFF` | Total runoff |
| `QOVER` | Surface runoff |
| `QDRAI` | Subsurface drainage |

---

*See [History Output](../running-ctsm/history-output.md) for configuring output.*
