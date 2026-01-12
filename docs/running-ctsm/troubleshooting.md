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
./xmlquery RUNDIR

# List logs by modification time (most recent first)
ls -t $RUNDIR/*.log.*
```

### Key Log Files

| Log | Contains |
|-----|----------|
| `cesm.log.*` | Top-level model output |
| `lnd.log.*` | Land (CLM) component messages |
| `cpl.log.*` | Coupler output, timing info |
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

## Debugging Mode

### Runtime Diagnostics (No Rebuild)

```bash
./xmlchange INFO_DBUG=2
./case.submit
```

Adds diagnostic output to `cpl.log`.

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

Check simulation performance in `cpl.log`:

```bash
grep tStamp cpl.log.* | tail -20
```

Output shows time per model day:
```
tStamp_write: model date = 10120 0 wall clock = 2024-01-15 09:10:46 avg dt = 58.58 dt = 58.18
```

- `avg dt` = average seconds per model day
- `dt` = seconds for this specific day

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
