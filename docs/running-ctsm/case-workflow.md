# Case Workflow

CTSM uses a case-based workflow where each simulation is configured, built, and run from a dedicated case directory.

## Overview

```
create_newcase → case.setup → case.build → case.submit
```

Each step creates or modifies files in the case directory.

## Case Lifecycle

### 1. Create a New Case

```bash
cd $CIME_SCRIPTS

./create_newcase \
    --case $CASES/my_case_name \
    --compset I1850Clm60BgcCrop \
    --res f09_g17 \
    --machine hipergator \
    --run-unsupported
```

**Key arguments:**

| Argument | Description |
|----------|-------------|
| `--case` | Path to the case directory |
| `--compset` | Component set (model configuration) |
| `--res` | Resolution (grid) |
| `--machine` | Target machine |
| `--run-unsupported` | Required for non-NCAR machines |

### 2. Configure the Case

```bash
cd $CASES/my_case_name

# Modify XML settings
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=5
./xmlchange JOB_WALLCLOCK_TIME=04:00:00

# Query current values
./xmlquery STOP_OPTION STOP_N
```

**Common XML variables:**

| Variable | Description | Example |
|----------|-------------|---------|
| `STOP_OPTION` | Time units | `nyears`, `nmonths`, `ndays` |
| `STOP_N` | Number of time units | `5` |
| `JOB_WALLCLOCK_TIME` | SLURM time limit | `04:00:00` |
| `NTASKS` | Number of MPI tasks | `32` |

### 3. Setup the Case

```bash
./case.setup
```

This creates:
- Namelist files (`user_nl_clm`, etc.)
- Build scripts
- Run scripts

### 4. Modify Namelists

Edit `user_nl_clm` to customize model behavior:

```fortran
! Example: Configure history output
hist_empty_htapes = .true.
hist_fincl1 = 'TSA', 'GPP', 'EFLX_LH_TOT'
hist_nhtfrq = -24
hist_mfilt = 365
```

### 5. Build the Case

```bash
./case.build
```

This compiles the model. Takes 10-30 minutes depending on configuration.

**Rebuilding:** If you change Fortran source code or XML build variables:
```bash
./case.build --clean-all
./case.build
```

### 6. Submit the Case

```bash
./case.submit
```

This submits the job to SLURM. Monitor with:
```bash
squeue -u $USER
```

## Directory Structure

After setup, each case has these key directories:

| Directory | Variable | Purpose |
|-----------|----------|---------|
| Case root | `CASEROOT` | Configuration, scripts, namelists |
| Build | `EXEROOT` | Compiled executables |
| Run | `RUNDIR` | Output, restarts, logs |
| Archive | `DOUT_S_ROOT` | Archived output |

Query paths:
```bash
./xmlquery EXEROOT RUNDIR DOUT_S_ROOT
```

## Common Compsets

| Compset | Description |
|---------|-------------|
| `I1850Clm60BgcCrop` | Pre-industrial BGC with crops |
| `I2000Clm60BgcCrop` | Present-day BGC with crops |
| `I1850Clm60Sp` | Pre-industrial satellite phenology |
| `IHistClm60BgcCrop` | Historical transient run |

List available compsets:
```bash
./query_config --compsets clm
```

## Common Resolutions

| Resolution | Description |
|------------|-------------|
| `f09_g17` | ~1° atmosphere, ~1° ocean |
| `f19_g17` | ~2° atmosphere, ~1° ocean |
| `CLM_USRDAT` | User-provided data (single-point) |

## Checking Case Status

The `CaseStatus` file logs all workflow steps:

```bash
cat CaseStatus
```

Look for timestamps and success/failure messages. When a case fails, CaseStatus points to the relevant log file.

## Run Types

CTSM supports three run types:

| Type | Use Case | Configuration Changes? | Bit-for-bit? |
|------|----------|----------------------|--------------|
| **startup** | Fresh simulation | N/A | N/A |
| **branch** | Exact continuation | No | Yes |
| **hybrid** | New start from restart | Yes | No |

### Continuing a Run (CONTINUE_RUN)

To continue a run from where it stopped:

```bash
./xmlchange CONTINUE_RUN=TRUE
./case.submit
```

This is the simplest continuation - the run picks up exactly where it left off.

### Branch Runs

Branch runs continue from another case's restart, maintaining bit-for-bit reproducibility:

```bash
cd $CIME_SCRIPTS

# Create new case
./create_newcase \
    --case $CASES/my_branch_case \
    --compset I1850Clm60BgcCrop \
    --res f09_g17 \
    --machine hipergator \
    --run-unsupported

cd $CASES/my_branch_case

# Configure as branch run
./xmlchange RUN_TYPE=branch
./xmlchange RUN_REFCASE=source_case_name
./xmlchange RUN_REFDATE=0005-01-01
./xmlchange RUN_REFDIR=/path/to/source/case/run

# Copy restart files (if not in RUN_REFDIR)
./xmlchange GET_REFCASE=FALSE

./case.setup
./case.build
./case.submit
```

!!! note "Branch Run Restrictions"
    Branch runs must use the **same** compset and resolution as the source case. You cannot change model configuration in a branch run.

### Hybrid Runs

Hybrid runs initialize from restart files but allow configuration changes:

```bash
./xmlchange RUN_TYPE=hybrid
./xmlchange RUN_REFCASE=source_case_name
./xmlchange RUN_REFDATE=0005-01-01
./xmlchange GET_REFCASE=FALSE
```

Use hybrid runs when you need to:
- Change compset or resolution
- Modify namelist settings that affect initialization
- Start a new experiment from a spun-up state

## Tips

- **Naming convention:** Include compset and date in case names (e.g., `I1850Clm60BgcCrop.f09_g17.240115`)
- **Check before building:** Use `./preview_namelists` to verify configuration
- **Don't edit generated files:** Modify `user_nl_*` files, not the generated `*_in` files
- **Keep case directories:** They're small and useful for reproducing runs
