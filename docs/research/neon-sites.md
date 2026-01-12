# NEON Sites

*Documentation for NEON (National Ecological Observatory Network) site simulations.*

---

**Status:** Placeholder - content to be added.

## Overview

<!-- TODO: Document NEON site workflow -->
<!-- - What is NEON -->
<!-- - How we use NEON data -->
<!-- - Validation approach -->

## Sites of Interest

<!-- TODO: Document specific sites -->

| Site | Location | Ecosystem | Status |
|------|----------|-----------|--------|
| OSBS | Florida | Longleaf pine | Active |
| ... | ... | ... | ... |

## Data Extraction

<!-- TODO: Document NEON data workflow -->
<!-- - Where to get NEON data -->
<!-- - How to process for CTSM comparison -->
<!-- - Quality control -->

## Subset Data

Pre-generated subset data for NEON sites:

```
/blue/gerber/earth_models/shared.subset.data/
├── OSBS/
│   ├── surfdata_*.nc
│   ├── datmdata/
│   └── user_mods/
└── .../
```

## Configuration Files

Site-specific configurations in the CTSM fork:

```
$CTSMROOT/tools/site_and_regional/
├── osbs.cfg
└── ...
```

---

*See [Single-Point Runs](../running-ctsm/single-point.md) for creating site simulations.*
