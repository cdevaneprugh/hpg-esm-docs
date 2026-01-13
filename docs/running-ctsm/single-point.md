# Single-Point Runs

Single-point (site-level) simulations allow you to run CTSM for a specific location rather than a global grid. This is useful for:

- Validating against tower observations (NEON, FLUXNET)
- Testing model changes quickly
- Site-specific research

## Subset Data Workflow

CTSM provides the `subset_data` tool to extract site-level data from global datasets.

### Overview

```
subset_data → creates site data → create_newcase with CLM_USRDAT
```

The subset data creates:
- Surface data for the site
- Atmospheric forcing data (DATM)
- User mods for case configuration

### Creating Subset Data

```bash
cd $CTSMROOT/tools/site_and_regional

# Activate CTSM Python environment
module load conda
conda activate ctsm_pylib

# Run subset_data for a site
./subset_data point \
    --lat 29.6893 \
    --lon -82.0096 \
    --site OSBS \
    --create-surface \
    --create-datm \
    --datm-syr 2000 \
    --datm-eyr 2020 \
    --outdir $SUBSET_DATA/OSBS
```

**Key arguments:**

| Argument | Description |
|----------|-------------|
| `--lat`, `--lon` | Site coordinates |
| `--site` | Site name (used in filenames) |
| `--create-surface` | Generate surface data |
| `--create-datm` | Generate atmospheric forcing |
| `--datm-syr`, `--datm-eyr` | Forcing data year range |
| `--outdir` | Output directory |

### Subset Data Output

The tool creates a directory structure:

```
$SUBSET_DATA/OSBS/
├── surfdata_*.nc           # Surface dataset
├── datmdata/               # Atmospheric forcing
│   └── atm_forcing.*.nc
└── user_mods/
    ├── shell_commands      # XML changes
    ├── user_nl_clm         # CLM namelist additions
    └── user_nl_datm_streams # DATM configuration
```

### Creating a Case with Subset Data

```bash
cd $CIME_SCRIPTS

./create_newcase \
    --case $CASES/OSBS.I1850Clm60BgcCrop \
    --compset I1850Clm60BgcCrop \
    --res CLM_USRDAT \
    --machine hipergator \
    --run-unsupported \
    --user-mods-dirs $SUBSET_DATA/OSBS/user_mods
```

**Key difference:** Use `--res CLM_USRDAT` and `--user-mods-dirs` to point to your subset data.

### Configure and Run

```bash
cd $CASES/OSBS.I1850Clm60BgcCrop

# Verify settings (user_mods should have applied changes)
./xmlquery CLM_USRDAT_NAME DATM_MODE

# Setup and build
./case.setup
./case.build

# Modify run length as needed
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=5
./xmlchange JOB_WALLCLOCK_TIME=02:00:00

# Submit
./case.submit
```

## Available Forcing Data

Our fork is configured for CRUNCEP forcing:

| Dataset | Years | Description |
|---------|-------|-------------|
| CRUNCEPv7 | 1901-2016 | 6-hourly reanalysis |

The `default_data_*.cfg` files in the fork specify HiPerGator paths.

## NEON Sites

For NEON tower sites, CTSM provides pre-configured options:

```bash
# List available NEON sites
./subset_data point --help
```

Common sites:

| Site | Location | Ecosystem |
|------|----------|-----------|
| OSBS | Florida | Longleaf pine |
| HARV | Massachusetts | Mixed deciduous |
| BART | New Hampshire | Northern hardwood |

## Shared Subset Data

Your group may maintain pre-generated subset data for common sites. Check with your group before generating new ones - subset data can be shared to avoid duplicate downloads.

Typical location: `/blue/<group>/earth_models/shared.subset.data/`

## Troubleshooting

### subset_data Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "No data found" | Coordinates outside data range | Verify lat/lon |
| Import errors | Missing Python packages | Activate `ctsm_pylib` |
| Path errors | Wrong input data location | Check `$INPUT_DATA` |

### Case Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "CLM_USRDAT_NAME not set" | user_mods not applied | Check `--user-mods-dirs` path |
| Missing surface data | File not found | Verify `$SUBSET_DATA` path |

## Tips

- **Start small:** Run a short test (1 year) before long simulations
- **Check user_mods:** After `case.setup`, verify the settings were applied
- **Keep subset data:** It's faster to reuse than regenerate
- **Coordinate precision:** Use enough decimal places for your site

## PTS_MODE (Deprecated)

Older documentation may reference `PTS_MODE` for single-point runs. This is deprecated in CTSM and has limitations (no restart capability). Use the subset data workflow instead.
