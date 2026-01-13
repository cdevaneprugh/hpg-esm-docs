# Glossary

Key terms for CTSM development on HiPerGator.

## Models & Components

**CTSM** - Community Terrestrial Systems Model. The land model, designed to run standalone or as part of CESM. [GitHub](https://github.com/ESCOMP/CTSM)

**CESM** - Community Earth System Model. Fully-coupled global climate model. CTSM is its land component.

**CLM** - Community Land Model. The land component within CTSM that models vegetation, hydrology, carbon cycling, etc.

**CIME** - Common Infrastructure for Modeling the Earth (pronounced "SEAM"). Provides the case control system for configuring, building, and running Earth system models.

**FATES** - Functionally Assembled Terrestrial Ecosystem Simulator. A vegetation demographic model that can be coupled to CLM for more detailed vegetation dynamics. [FATES GitHub](https://github.com/NGEET/fates)

## Tools & Utilities

**cprnc** - Tool for comparing NetCDF files. Used to verify model output. Each group builds their own (see [Prerequisites](installation/prerequisites.md)).

**git-fleximod** - CTSM's tool for managing submodules. Replaces the older `manage_externals` used by CESM.

**mksurfdata** - Tool for generating surface datasets (fsurdat files) for CTSM. Requires PIO library.

**PIO** (ParallelIO) - Parallel I/O library required for mksurfdata. Each group builds their own (see [Prerequisites](installation/prerequisites.md)).

## Configuration & Data

**CLM_USRDAT** - Resolution setting for user-provided datasets. Used with subset data for single-point runs.

**Compset** - Component set. A predefined combination of model components and configurations. Format: `YYYYA_B_C_D_E_F` where letters represent different components. Example: `I1850Clm60BgcCrop` = Pre-industrial (1850), CLM 6.0 with BGC and crops.

**Namelist** - Fortran configuration file format. Key files: `user_nl_clm`, `user_nl_datm_streams`.

**PFT** - Plant Functional Type. Categories of vegetation (e.g., broadleaf deciduous tree, C3 grass, corn) that share similar characteristics. CTSM uses PFTs to represent vegetation at each grid point.

**Surface data (fsurdat)** - NetCDF file containing land surface characteristics (soil, vegetation, topography).

## Output Files

**History files** - Model output files containing time-varying data (`.h0.nc`, `.h1.nc`, etc.). Configured via `hist_*` namelist variables. Use for analysis and plotting.

**Restart files** - Checkpoint files (`.r.nc`) that capture the complete model state. Used to continue simulations from a specific point. Required for branch runs and spinup continuation.

## Case Directories

**CASEROOT** - Case configuration directory. Contains XML settings, namelists, scripts.

**EXEROOT** - Build directory where compiled code lives.

**RUNDIR** - Runtime directory containing output, restarts, logs.

**DOUT_S_ROOT** - Archive directory for completed output.

## Run Types

**startup** - Fresh start with cold initialization.

**branch** - Exact continuation from a restart file. Preserves bit-for-bit answers.

**hybrid** - New start initialized from another case's restart file. Allows configuration changes.

## Spinup

**AD spinup** - Accelerated Decomposition spinup. First phase (~200 years) with accelerated soil carbon turnover.

**Post-AD spinup** - Second phase (several hundred years) with normal turnover rates to reach equilibrium.

## Infrastructure

**ccs_config** - Repository containing machine configurations for CIME. Our fork adds HiPerGator support.

**ESMF** - Earth System Modeling Framework. Provides coupling infrastructure.

**NUOPC** - National Unified Operational Prediction Capability. Layer built on ESMF for coupling standards.

**SCRIP** - Spherical Coordinate Remapping and Interpolation Package. Used for creating mapping files.

## HiPerGator Specific

**HPG** - HiPerGator. UF's supercomputer.

**lmod** - Module system for managing software environments.

**SLURM** - Job scheduler for submitting work to compute nodes.

**QOS** - Quality of Service. Defines resource limits for job queues.
