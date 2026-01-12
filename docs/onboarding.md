# Onboarding

This page provides essential HiPerGator information for running CTSM. For comprehensive Linux training, see the resources at the bottom.

## HiPerGator Basics

HiPerGator is UF's supercomputer - a cluster of connected nodes that share resources. You interact with it via the command line.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Login nodes** | Where you land when you SSH in. For light tasks only (editing, file management) |
| **Compute nodes** | Where jobs run. Access via SLURM scheduler |
| **SLURM** | Job scheduler - submits work to compute nodes |
| **lmod** | Module system for loading software (compilers, libraries) |

### Storage Locations

| Path | Purpose | Notes |
|------|---------|-------|
| `/home/$USER/` | Config files, small files | 40GB quota, backed up |
| `/blue/gerber/$USER/` | Active projects, code, data | Group allocation, fast |
| `/orange/gerber/` | Archival storage | Slow - don't use for active work |

## Module Environment

CTSM requires specific modules. We use a saved module collection:

```bash
# Load the CTSM module collection
module restore ctsm-modules

# Verify modules are loaded
module list
```

The collection includes: GCC 14.2.0, OpenMPI 5.0.7, NetCDF, HDF5, ESMF 8.8.1, CMake, Python 3.12

## Environment Variables

Add these to your `~/.bashrc`:

```bash
# Core paths
export BLUE="/blue/gerber/$USER"
export CASES="$BLUE/cases"
export ESM_OUTPUT="$BLUE/earth_model_output/cime_output_root"

# CTSM paths
export CTSMROOT="/blue/gerber/cdevaneprugh/ctsm5.3"
export CIME_SCRIPTS="$CTSMROOT/cime/scripts"

# Input data
export INPUT_DATA="/blue/gerber/earth_models/inputdata"
export SUBSET_DATA="/blue/gerber/earth_models/shared.subset.data"
```

After editing, reload: `source ~/.bashrc`

## SLURM Basics

Submit jobs to compute nodes - never run heavy computation on login nodes.

```bash
# Check your running jobs
squeue -u $USER

# Check group jobs
squeue -A gerber

# Cancel a job
scancel <job_id>

# View job details
scontrol show job <job_id>
```

### Group QOS Limits

| Resource | Limit |
|----------|-------|
| CPU cores | 10 |
| Memory | 80000M |

## Useful Commands

```bash
# Show HiPerGator environment variables
env | grep HPC | sort

# View directory structure
tree | less

# Check disk usage
du -sh /blue/gerber/$USER/
```

## Learning Resources

### HiPerGator Documentation
- [Getting Started](https://help.rc.ufl.edu/doc/Getting_Started)
- [Training Videos](https://help.rc.ufl.edu/doc/Training_Videos)
- [SLURM Scheduling](https://help.rc.ufl.edu/doc/HPG_Scheduling)
- [Module System](https://help.rc.ufl.edu/doc/Modules)

### Linux Command Line
- [Linux Journey](https://linuxjourney.com/) - Comprehensive tutorials
- [UF Linux Training](https://github.com/UFResearchComputing/Linux_training) - HiPerGator-focused
- [The Art of Command Line](https://github.com/jlevy/the-art-of-command-line) - Reference guide

### Text Editors
The default editor is often `vim`. If you accidentally open it:

- Press `Esc`, then type `:q!` and press Enter to quit without saving
- Or press `Esc`, then `:wq` and Enter to save and quit

Alternatives: `nano` (simpler), `emacs`
