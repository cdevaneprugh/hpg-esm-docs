# Fork Reference

This page explains why and how we forked CTSM for HiPerGator. Use this as a reference for understanding our modifications or for creating your own fork.

## Why We Fork

CTSM isn't plug-and-play on HiPerGator. Several issues require source-level modifications.

### 1. Machine Configuration (ccs_config)

CTSM uses CIME (Common Infrastructure for Modeling the Earth) for builds and job submission. CIME needs machine-specific configuration files that define:

- Compiler paths and flags
- MPI library settings
- SLURM batch directives
- Input data locations

HiPerGator isn't in the upstream machine list, so we need custom configs.

**The complication:** Machine configs live in a separate repository (`ccs_config_cesm`) that CTSM pulls in as a git submodule. To add HiPerGator support, we had to:

1. Fork `ccs_config_cesm`
2. Add HiPerGator configs to our fork
3. Fork CTSM to point its `.gitmodules` at our ccs_config fork

This submodule-of-a-submodule situation makes the fork necessary.

### 2. User Config (~/.cime) Doesn't Work Reliably

CIME v3 theoretically supports user config overrides in `~/.cime/`. We tried this approach and encountered bugs:

- Batch configuration loading was inconsistent
- `NODENAME_REGEX` machine detection failed intermittently
- Each user had to maintain their own config files

Forking ccs_config gives us a single source of truth that works reliably for everyone.

### 3. Build Tool Bugs

Several CTSM build tools have issues with newer GCC versions:

| Issue | Symptom | Fix |
|-------|---------|-----|
| PIO linking | mksurfdata fails to link | Change `STATIC` to `SHARED` in CMakeLists.txt |
| Format specifiers | GCC 10+ compilation errors | Add explicit width to Fortran format specs (`I` → `I12`) |
| GCC 14 flags | Compilation warnings as errors | Add `-fallow-argument-mismatch` flag |

These are bugs that should be fixed upstream, but until they are, our fork includes the fixes.

### 4. HiPerGator-Specific Paths

Default paths in some tools point to NCAR systems. Our fork updates paths to work with HiPerGator's storage layout.

## Repository Structure

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | CTSM with tool fixes |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | Machine configuration |

The ccs_config fork is included as a submodule of the CTSM fork.

!!! note "Version Notice"
    These forks are based on CTSM 5.3.085. Other versions may have different issues or may have fixed some of these bugs upstream.

## Modifications Detail

### CTSM Modifications

| File | Change | Reason |
|------|--------|--------|
| `tools/mksurfdata_esmf/src/CMakeLists.txt` | `STATIC` → `SHARED` | Fix for shared PIO libraries |
| `tools/mksurfdata_esmf/src/mksurfdata.F90` | Format specifiers `I` → `I12` | GCC 10+ requires explicit widths |
| `tools/mksurfdata_esmf/gen_mksurfdata_build` | Added GCC 14 flags | Compatibility workarounds |
| `python/ctsm/site_and_regional/single_point_case.py` | `MPILIB=openmpi` | HiPerGator uses OpenMPI |
| `tools/site_and_regional/default_data_*.cfg` | Input paths | HiPerGator data locations |

### ccs_config Modifications

The `machines/hipergator/` directory contains:

- **config_machines.xml** - Full machine definition (compilers, paths, MPI settings)
- **config_batch.xml** - SLURM batch settings (no `--exclusive` flag for shared nodes)
- **gnu_hipergator.cmake** - Compiler flags and library paths

See [CIME Configuration](cime-config.md) for detailed explanation of these files.

## Using This Fork

### Option 1: Clone Directly

The simplest approach - clone our fork and use it as-is. See [Quick Start](quickstart.md) for installation steps.

You'll need to customize the QoS settings in the config files for your group (see [CIME Configuration](cime-config.md)).

### Option 2: Fork from Us

If you need additional modifications:

1. Fork `cdevaneprugh/CTSM` on GitHub
2. Clone your fork
3. Make your modifications
4. Update the ccs_config submodule if needed

This gives you a customizable base with our fixes already applied.

### Option 3: Start from Upstream

If you need a different CTSM version or want full control:

1. Fork `ESCOMP/CTSM` on GitHub
2. Fork `ESMCI/ccs_config_cesm`
3. Apply the modifications listed above
4. Update `.gitmodules` to point to your ccs_config fork

This is more work but gives you a clean slate.

## Updating the Fork

When upstream releases a new CTSM version:

```bash
# Add upstream remote (one time)
git remote add upstream https://github.com/ESCOMP/CTSM.git

# Fetch new tags
git fetch upstream --tags

# Create new branch from new tag
git checkout -b uf-ctsm5.3.090 ctsm5.3.090

# Cherry-pick modifications from old branch
git cherry-pick <commit-hash>...

# Update submodules
./bin/git-fleximod update

# Test and push
git push -u origin uf-ctsm5.3.090
```

!!! tip "Check if fixes are still needed"
    Before cherry-picking, check if upstream has fixed the issues. You may not need all the modifications for newer versions.
