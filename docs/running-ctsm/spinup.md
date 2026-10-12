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

**Duration:** ~200 years is the NCAR guideline (can vary by site). Our OSBS single-point AD spinups ran 600 years (CRUNCEP forcing) and ~195 years (NEON forcing, converged by ~year 180).

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
# (run from the case directory; history files are archived to DOUT_S_ROOT/lnd/hist)
HIST=$(./xmlquery --value DOUT_S_ROOT)/lnd/hist
ncdump -v TOTSOMC $(ls $HIST/*.clm2.h0a.*.nc | tail -1) | tail
```

Since ctsm5.3.062, history files are split into averaged (`h0a`, `h1a`, ...) and instantaneous (`h0i`, ...) streams, so look for `*.clm2.h0a.*.nc` rather than `*.clm2.h0.*.nc`.

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

# Startup run initialized from the final AD restart
./xmlchange RUN_TYPE=startup
./xmlchange CLM_ACCELERATED_SPINUP=off

# Point finidat at the AD case's last land restart
AD_REST=$(cd $CASES/my_site.AD_spinup && ./xmlquery --value DOUT_S_ROOT)/rest/0201-01-01-00000
echo "finidat = '$AD_REST/my_site.AD_spinup.clm2.r.0201-01-01-00000.nc'" >> user_nl_clm

# Run several hundred years
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=100
./xmlchange RESUBMIT=4  # 500 years total

./case.setup
./case.build
./case.submit
```

!!! note "Why startup + finidat, not branch"
    The AD to post-AD transition rescales the soil carbon and nitrogen pools, and that code path only runs when `nstep < 2` (`SoilBiogeochemCarbonStateType.F90`, `SoilBiogeochemNitrogenStateType.F90`). A branch run keeps the prior case's `nstep` and fails with "Error in entering/exiting spinup - should occur only when nstep = 1". A startup run with `finidat` set starts at `nstep` 1. Our `osbs.swenson.post-ad` case uses this setup.

### 4. Check Equilibrium

Spinup is complete when:

- TOTSOMC, TOTVEGC, TOTECOSYSC are stable year-to-year
- For global grids, NCAR's criterion is that less than ~3% of the land surface shows carbon disequilibrium. This does not apply to a single-point run.
- For our single-point site runs we use a block-mean drift test: the change between consecutive 20-year means of TOTECOSYSC, TOTSOMC, and TOTVEGC is below 1%

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
- Carbon pool drift below the thresholds in [Check Equilibrium](#4-check-equilibrium)

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
