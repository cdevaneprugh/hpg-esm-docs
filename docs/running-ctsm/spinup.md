# Spinup

BGC (biogeochemistry) simulations require spinup to bring soil carbon pools to equilibrium. This process can take hundreds to thousands of simulated years.

## Why Spinup?

Carbon pools in soil take centuries to equilibrate. Starting from arbitrary initial conditions would produce unrealistic results. Spinup runs the model repeatedly until carbon pools stabilize.

## Two-Phase Spinup

CTSM uses a two-phase approach:

### Phase 1: Accelerated Decomposition (AD) Spinup

Artificially accelerates soil carbon turnover to reach approximate equilibrium faster.

```bash
./xmlchange CLM_ACCELERATED_SPINUP=on
```

**Duration:** ~200 years (can vary by site)

### Phase 2: Post-AD Spinup

Returns to normal decomposition rates and runs to final equilibrium.

```bash
./xmlchange CLM_ACCELERATED_SPINUP=off
```

**Duration:** Several hundred years

## Spinup Workflow

### 1. Create AD Spinup Case

```bash
cd $CIME_SCRIPTS

./create_newcase \
    --case $CASES/my_site.AD_spinup \
    --compset I1850Clm60BgcCrop \
    --res CLM_USRDAT \
    --machine hipergator \
    --run-unsupported \
    --user-mods-dirs $SUBSET_DATA/my_site/user_mods

cd $CASES/my_site.AD_spinup

# Enable accelerated spinup
./xmlchange CLM_ACCELERATED_SPINUP=on

# Run for 200 years
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=50
./xmlchange RESUBMIT=3  # 4 submissions × 50 years = 200 years

./case.setup
./case.build
./case.submit
```

### 2. Monitor AD Spinup

Check that carbon pools are trending toward equilibrium:

```bash
# Look at TOTSOMC (soil organic carbon) over time
ncdump -v TOTSOMC $RUNDIR/*.clm2.h0.*.nc | tail
```

### 3. Create Post-AD Case

```bash
cd $CIME_SCRIPTS

./create_newcase \
    --case $CASES/my_site.postAD_spinup \
    --compset I1850Clm60BgcCrop \
    --res CLM_USRDAT \
    --machine hipergator \
    --run-unsupported \
    --user-mods-dirs $SUBSET_DATA/my_site/user_mods

cd $CASES/my_site.postAD_spinup

# Hybrid start from AD spinup
./xmlchange RUN_TYPE=hybrid
./xmlchange RUN_REFCASE=my_site.AD_spinup
./xmlchange RUN_REFDATE=0201-01-01  # End of AD spinup
./xmlchange GET_REFCASE=FALSE

# Copy restart files
cp $AD_RUNDIR/*.rpointer* $RUNDIR/
cp $AD_RUNDIR/*.r.*.nc $RUNDIR/

# Normal decomposition
./xmlchange CLM_ACCELERATED_SPINUP=off

# Run several hundred years
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=100
./xmlchange RESUBMIT=4  # 500 years total

./case.setup
./case.build
./case.submit
```

### 4. Check Equilibrium

Spinup is complete when:
- Less than ~3% of land surface shows carbon disequilibrium
- TOTSOMC, TOTVEGC, TOTECOSYSC are stable year-to-year

## Variables to Monitor

| Variable | Description |
|----------|-------------|
| `TOTECOSYSC` | Total ecosystem carbon |
| `TOTSOMC` | Soil organic matter carbon |
| `TOTVEGC` | Vegetation carbon |
| `GPP` | Gross primary production |
| `TWS` | Total water storage |

Include these in your history output:

```fortran
hist_fincl1 = 'TOTECOSYSC', 'TOTSOMC', 'TOTVEGC', 'GPP', 'TWS'
hist_nhtfrq = 0  ! Monthly
hist_mfilt = 12  ! 12 months per file
```

## Equilibrium Criteria

The model reaches equilibrium when carbon fluxes balance:

- NEE ≈ 0 (net ecosystem exchange)
- Year-to-year changes in carbon pools < 0.1%

Arctic and boreal regions take longest (~1000 years).

## Tips

- **Start with AD:** Never skip AD spinup for BGC runs
- **Check frequently:** Don't run hundreds of years without checking progress
- **Save restarts:** Keep restart files at key points
- **Site-specific duration:** Some sites equilibrate faster than others
- **SP mode doesn't need spinup:** Satellite phenology runs don't have prognostic carbon

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Carbon still drifting | Not enough spinup time | Continue running |
| Oscillating values | Numerical issues | Check for extreme climate |
| Very slow equilibration | Cold/wet sites | Normal for arctic/boreal |

## Satellite Phenology (SP) Mode

If you don't need prognostic carbon pools, SP mode skips the carbon cycle entirely:

```bash
--compset I1850Clm60Sp  # No BGC, no spinup needed
```

SP runs use prescribed LAI from satellite observations and don't require spinup.
