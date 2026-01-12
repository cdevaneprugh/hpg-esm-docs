# History Output

CTSM writes diagnostic output to history files (NetCDF format). This page covers configuring what variables are output, at what frequency, and how they're averaged.

## Configuration

History output is configured in `user_nl_clm`. Changes take effect on the next run (no rebuild required).

### Basic Example

```fortran
! Clear default output, add specific variables
hist_empty_htapes = .true.
hist_fincl1 = 'TSA', 'TSKIN', 'GPP', 'EFLX_LH_TOT', 'FSH'

! Daily output, one year per file
hist_nhtfrq = -24
hist_mfilt = 365
```

## Key Namelist Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `hist_empty_htapes` | Clear default output | `.false.` |
| `hist_fincl1` - `hist_fincl10` | Variables for h0-h9 streams | varies |
| `hist_nhtfrq` | Output frequency | `0` (monthly) |
| `hist_mfilt` | Samples per file | `1` |

## Output Frequency

The `hist_nhtfrq` variable controls output frequency:

| Value | Meaning | Example |
|-------|---------|---------|
| `0` | Monthly average | Default |
| `> 0` | Every N timesteps | `48` = every 48 timesteps |
| `< 0` | Every N hours | `-24` = daily, `-1` = hourly |

### Examples

```fortran
! Monthly averages (default)
hist_nhtfrq = 0

! Daily averages
hist_nhtfrq = -24

! 6-hourly output
hist_nhtfrq = -6

! Every timestep (30-min default)
hist_nhtfrq = 1
```

## Samples Per File

The `hist_mfilt` variable sets how many time samples go in each file:

```fortran
! One year of daily data per file
hist_nhtfrq = -24
hist_mfilt = 365

! One month of daily data per file
hist_nhtfrq = -24
hist_mfilt = 31

! 12 months per file (monthly data)
hist_nhtfrq = 0
hist_mfilt = 12
```

## Averaging Flags

By default, variables are averaged over the output period. Append a flag to change this:

| Flag | Meaning | Example |
|------|---------|---------|
| `A` | Average (default) | `'GPP'` or `'GPP:A'` |
| `I` | Instantaneous | `'TSA:I'` |
| `M` | Minimum | `'TSA:M'` |
| `X` | Maximum | `'TSA:X'` |
| `SUM` | Sum | `'RAIN:SUM'` |

```fortran
! Daily max and min temperature
hist_fincl1 = 'TSA:X', 'TSA:M', 'GPP'
```

## Multiple History Streams

You can configure up to 10 independent history streams (h0-h9):

```fortran
! h0: Monthly averages of key variables
hist_fincl1 = 'GPP', 'NPP', 'NEE', 'TLAI'
hist_nhtfrq(1) = 0
hist_mfilt(1) = 12

! h1: Daily water balance
hist_fincl2 = 'QSOIL', 'QVEGE', 'QVEGT', 'RAIN', 'SNOW'
hist_nhtfrq(2) = -24
hist_mfilt(2) = 365

! h2: Hourly temperature for heat waves
hist_fincl3 = 'TSA', 'TSKIN'
hist_nhtfrq(3) = -1
hist_mfilt(3) = 24
```

## Common Variables

### Energy Balance
| Variable | Description | Units |
|----------|-------------|-------|
| `FSH` | Sensible heat flux | W/m² |
| `EFLX_LH_TOT` | Latent heat flux | W/m² |
| `FSA` | Absorbed solar radiation | W/m² |
| `FIRA` | Net infrared radiation | W/m² |

### Carbon Fluxes
| Variable | Description | Units |
|----------|-------------|-------|
| `GPP` | Gross primary production | gC/m²/s |
| `NPP` | Net primary production | gC/m²/s |
| `NEE` | Net ecosystem exchange | gC/m²/s |
| `ER` | Ecosystem respiration | gC/m²/s |

### Hydrology
| Variable | Description | Units |
|----------|-------------|-------|
| `QRUNOFF` | Total runoff | mm/s |
| `QSOIL` | Ground evaporation | mm/s |
| `QVEGE` | Canopy evaporation | mm/s |
| `QVEGT` | Canopy transpiration | mm/s |
| `TWS` | Total water storage | mm |
| `ZWT` | Water table depth | m |

### Temperature
| Variable | Description | Units |
|----------|-------------|-------|
| `TSA` | 2m air temperature | K |
| `TSKIN` | Surface temperature | K |
| `TSOI` | Soil temperature | K |
| `TV` | Vegetation temperature | K |

### Vegetation
| Variable | Description | Units |
|----------|-------------|-------|
| `TLAI` | Total LAI | m²/m² |
| `TOTVEGC` | Total vegetation carbon | gC/m² |
| `TOTSOMC` | Total soil organic carbon | gC/m² |

## Finding Available Variables

List all available history variables:

```bash
# Search the history field definitions
grep -r "hist_" $CTSMROOT/bld/namelist_files/
```

Or check the CTSM documentation: [History Fields](https://escomp.github.io/CTSM/tech_note/History_Fields/index.html)

## Output File Naming

History files follow this naming convention:

```
<case_name>.clm2.h0.YYYY-MM.nc    # Monthly h0 stream
<case_name>.clm2.h1.YYYY-MM-DD.nc # Daily h1 stream
```

## Tips

- **Start minimal:** Begin with `hist_empty_htapes = .true.` and add only what you need
- **Check file sizes:** High-frequency output can generate large files
- **Use multiple streams:** Separate high-frequency and low-frequency output
- **Document your choices:** Note what variables you're using and why
