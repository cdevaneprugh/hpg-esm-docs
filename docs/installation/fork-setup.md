# Fork Setup

We maintain forks of CTSM with HiPerGator-specific modifications. This page explains the fork strategy and how to use it.

## Why We Fork

CTSM requires modifications to work on HiPerGator:

1. **Machine configuration** - HiPerGator isn't a supported machine, so we need custom configs
2. **Build tool fixes** - Some tools have bugs that affect GCC/shared library builds
3. **Path customization** - Default paths point to NCAR systems

Rather than manually applying patches, we maintain forks that track these changes in git.

## Repository Structure

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | CTSM with tool fixes |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | Machine configuration |

The ccs_config fork is included as a submodule of the CTSM fork.

## Using the Fork

### Clone the Repository

```bash
cd /blue/gerber/$USER

# Clone from our fork
git clone https://github.com/cdevaneprugh/CTSM.git ctsm5.3
cd ctsm5.3

# Checkout the HiPerGator branch
git checkout uf-ctsm5.3.085
```

### Initialize Submodules

CTSM uses `git-fleximod` (not the older `manage_externals`):

```bash
./bin/git-fleximod update
```

This pulls in all required submodules including our ccs_config fork with HiPerGator configs.

### Verify Setup

```bash
# Check submodule status
./bin/git-fleximod status

# Verify machine config exists
ls ccs_config/machines/hipergator/
```

You should see: `config_machines.xml`, `config_batch.xml`, `gnu_hipergator.cmake`

## Modifications in the Fork

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

## Why Not ~/.cime?

CIME v3 supports user config overrides in `~/.cime/`, but this approach has bugs:

- Batch config loading is inconsistent
- NODENAME_REGEX detection issues
- Each user must maintain their own config

Forking ccs_config gives us a single source of truth that works reliably.

## Updating the Fork

When upstream releases a new version:

1. Fetch upstream changes
2. Create a new branch from the upstream tag
3. Cherry-pick or rebase our modifications
4. Test with a simple case
5. Push to the fork

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

## Shared Installation

For the Gerber group, a shared installation exists at:

```
/blue/gerber/cdevaneprugh/ctsm5.3
```

You can use this directly without cloning your own copy. Just ensure you have the environment variables set (see [Prerequisites](prerequisites.md)).

## Creating Your Own Fork

If you need to make additional modifications:

1. Fork `cdevaneprugh/CTSM` on GitHub
2. Clone your fork
3. Create a feature branch
4. Make modifications
5. Push to your fork

For modifications that benefit everyone, consider submitting them back to the group fork.
