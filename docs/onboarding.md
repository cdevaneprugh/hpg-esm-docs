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
| `/home/$USER/` | Config files, small files | 40GB quota, not backed up; daily and weekly snapshots in `~/.snapshot` |
| `/blue/<group>/$USER/` | Active projects, code, data | Group allocation, Lustre parallel filesystem, not backed up |
| `/orange/<group>/` | Archival storage | Slow - don't use for active work |

Replace `<group>` with your research group's allocation (e.g., `gerber`, `biology`, etc.).

## Module Environment

CTSM requires specific modules. We use a saved module collection:

```bash
# Load the CTSM module collection
module restore ctsm-modules

# Verify modules are loaded
module list
```

The collection includes: GCC 14.2.0, OpenMPI 5.0.7, NetCDF, HDF5, ESMF 8.8.1, CMake, Python 3.12

## Recommended Shortcuts

These environment variables can save typing. Add them to your `~/.bashrc` and customize for your setup:

```bash
# Your workspace (customize <group>)
export BLUE="/blue/<group>/$USER"
export CASES="$BLUE/cases"

# Your CTSM installation (after cloning)
export CTSMROOT="$BLUE/ctsm5.3"
export CIME_SCRIPTS="$CTSMROOT/cime/scripts"

# Your group's input data location
export INPUT_DATA="/blue/<group>/earth_models/inputdata"

# Shared single-point subset data (used on the Running CTSM pages)
export SUBSET_DATA="/blue/<group>/earth_models/shared.subset.data"

# Analysis scripts and the pysheds fork (used on the hillslope pages)
export TOOLS="$BLUE/hpg-esm-tools"
export PYSHEDS_FORK="$BLUE/pysheds_fork"
alias pysheds-env='export PYTHONPATH="$PYSHEDS_FORK:$PYTHONPATH"'
```

After editing, reload: `source ~/.bashrc`

!!! tip
    The exact paths depend on where you install CTSM and where your group stores input data. These are suggestions, not requirements.

## SLURM Basics

Submit jobs to compute nodes - never run heavy computation on login nodes.

```bash
# Check your running jobs
squeue -u $USER

# Check group jobs (replace <group>)
squeue -A <group>

# Cancel a job
scancel <job_id>

# View job details
scontrol show job <job_id>
```

### Group QOS Limits

Each group has its own resource limits. Check your group's QoS:

```bash
showQos <group>
```

This shows your group's CPU, memory, and GPU limits. CTSM cases will fail if they request more resources than your QoS allows.

## Useful Commands

```bash
# Show HiPerGator environment variables
env | grep HPC | sort

# View directory structure
tree | less

# Check disk usage (customize path)
du -sh /blue/<group>/$USER/
```

## Learning Resources

### HiPerGator Documentation
- [Getting Started](https://docs.rc.ufl.edu/quickstart/)
- [Training](https://docs.rc.ufl.edu/training/)
- [SLURM Scheduling](https://docs.rc.ufl.edu/scheduler/)
- [Module System](https://docs.rc.ufl.edu/software/modules_basics/)
- [Storage](https://docs.rc.ufl.edu/quickstart/practical_storage/)

### Linux Command Line
- [Linux Journey](https://linuxjourney.com/) - Comprehensive tutorials
- [UF Linux Training](https://github.com/UFResearchComputing/Linux_training) - HiPerGator-focused
- [The Art of Command Line](https://github.com/jlevy/the-art-of-command-line) - Reference guide

### Text Editors
The default editor is often `vim`. If you accidentally open it:

- Press `Esc`, then type `:q!` and press Enter to quit without saving
- Or press `Esc`, then `:wq` and Enter to save and quit

Alternatives: `nano` (simpler), `emacs`
