# Troubleshooting

Guide to diagnosing and fixing common CTSM failures on HiPerGator.

## First Steps

When a case fails:

1. **Check CaseStatus** - The `CaseStatus` file in your case directory logs all workflow steps with timestamps. Look for error messages and paths to log files.

2. **Follow the path** - CaseStatus points to the specific log containing the error. Read that file.

3. **Check SLURM output** - Look for `slurm-*.out` files in the case directory.

```bash
cd $CASES/my_case
cat CaseStatus | tail -20
```

## SLURM Failures

### NODE_FAIL

**Message:** `Job <job_id> failed, node failure`

**Cause:** Hardware/system failure on compute node.

**Solution:** Resubmit the job. If persistent, contact HiPerGator support.

### TIMEOUT

**Message:** `Job <job_id> cancelled at <timestamp> because it expired`

**Cause:** Job exceeded wall time limit.

**Solution:**
```bash
./xmlchange JOB_WALLCLOCK_TIME=08:00:00
./case.submit
```

Or reduce run length with `STOP_N`.

### OUT_OF_MEMORY

**Message:** `Job <job_id> killed due to out-of-memory`

**Cause:** Exceeded memory allocation.

**Solution:** Request more memory or reduce domain size.

### FAILED (Exit Code)

**Message:** `Job <job_id> failed with exit code <X>`

**Cause:** Model crash - segfault, missing file, numerical error.

**Solution:** Check the component log files (see below).

## Log File Analysis

### Finding Log Files

```bash
# Query run directory
RUNDIR=$(./xmlquery --value RUNDIR)

# List logs by modification time (most recent first)
ls -t $RUNDIR/*.log.*
```

### Key Log Files

| Log | Contains |
|-----|----------|
| `cesm.log.*` | Top-level model output |
| `lnd.log.*` | Land (CLM) component messages |
| `med.log.*` | Mediator (CMEPS) output, timing info |
| `drv.log.*` | Driver output, run summary |
| `atm.log.*` | Atmosphere (DATM) messages |

### Searching for Errors

```bash
# Search all logs for errors
cd $RUNDIR
grep -i "error\|abort\|fail" *.log.* | head -50

# Check end of land log
tail -100 lnd.log.*
```

## Did It Fail or Time Out?

Check if all log files were modified at the same time:

```bash
cd $RUNDIR
stat *.log.* | grep Modify
```

If times are within seconds of each other, it likely timed out (the job was killed while running). If times vary, the model crashed.

## Common Model Errors

### Missing Input Files

**Symptom:** Error about file not found in `lnd.log`

**Solution:** Check paths in namelists, verify input data exists:
```bash
ls $INPUT_DATA/path/to/file
```

### Namelist Errors

**Symptom:** Error parsing namelist in `lnd.log`

**Solution:** Check `user_nl_clm` for syntax errors. Run:
```bash
./preview_namelists
```

### Numerical Instability

**Symptom:** NaN or Inf values, model crashes mid-run

**Solution:**
- Check forcing data for extreme values
- Try reducing timestep
- Enable debug mode for more info

### Memory Issues

**Symptom:** Segfault, out-of-memory errors

**Solution:**
- Reduce domain size
- Increase memory allocation
- Check for array bounds issues (enable DEBUG=TRUE)

### Cycled DATM Forcing Crash (`dtlimit`)

**Symptom:** The model hard-crashes at a forcing/stream boundary — often the final timestep or a cycle wrap — with an error originating in `dshr_strdata_mod.F90:1050`.

**Cause:** When a DATM stream cycles (`taxmode = cycle`) and wraps past its last record, the jump in record-time spacing trips CDEPS's default `dtlimit = 1.5`. This is stock CDEPS behavior, not a data problem — it bites any finite forcing window that is cycled (e.g. the custom NEON OSBS forcing, 2017-02 → 2025-06).

**Solution:** Set `dtlimit = -1` on the affected stream(s) in `user_nl_datm_streams` — CDEPS's own escape hatch for streams that do not cycle on January boundaries. Namelist-only, no rebuild.

See [NEON Atmospheric Forcing](../swenson/neon-forcing.md) for the concrete OSBS case.

## Debugging Mode

### Runtime Diagnostics (No Rebuild)

```bash
./xmlchange INFO_DBUG=2
./case.submit
```

Adds diagnostic output from the driver and mediator (`drv.log`, `med.log`).

### Full Debug Mode (Requires Rebuild)

```bash
./xmlchange DEBUG=TRUE
./case.build --clean-all
./case.build
./case.submit
```

Enables bounds checking, floating-point exception trapping. **Runs much slower.**

## Build Failures

### Compilation Errors

Check the build log:
```bash
./xmlquery EXEROOT
cat $EXEROOT/lnd.bldlog.*
```

Common causes:
- Missing modules (run `module restore ctsm-modules`)
- Syntax errors in SourceMods

### Rebuild After Changes

If you modify source code or build settings:
```bash
./case.build --clean-all
./case.build
```

## Timing Analysis

Check simulation performance in `med.log`:

```bash
grep "Model Date" med.log.* | tail -20
```

Output shows time per model day:
```text
Model Date: 2018-01-03T00:00:00 wall clock = 2026-07-20T11:38:18 avg dt =     1.07 s/day, dt =     1.09 s/day, rate =   216.87 ypd
```

- `avg dt` = average wall-clock seconds per model day
- `dt` = seconds for this specific day
- `rate` = simulated years per wall-clock day (ypd)

The end of `drv.log` also reports overall throughput (`# simulated years / cmp-day`).

Large variations in `dt` suggest system performance issues.

## Restart File Issues

### Corrupted Restarts

If a job timed out while writing restarts, files may be corrupted.

**Solution:**
1. Check restart file sizes (should be consistent)
2. Remove corrupted files
3. Copy previous restart set
4. Continue from earlier point

```bash
# List restart files by size
ls -la $RUNDIR/*.r.*.nc

# Remove potentially corrupted latest set
rm $RUNDIR/*.r.0050-01-01*.nc

# Update rpointer files to point to earlier restart
```

## Getting Help

1. **Check official docs:** [CIME Troubleshooting](https://esmci.github.io/cime/versions/master/html/users_guide/troubleshooting.html)

2. **Search forums:** [DiscussCESM](https://bb.cgd.ucar.edu/cesm/)

3. **Include in bug reports:**
   - CaseStatus file
   - Relevant log excerpts
   - Machine and version info
   - Steps to reproduce
