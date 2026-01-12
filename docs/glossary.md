# Glossary

Key terms for CTSM development on HiPerGator.

## Models & Components

**CTSM** - Community Terrestrial Systems Model. The land model, designed to run standalone or as part of CESM. [GitHub](https://github.com/ESCOMP/CTSM)

**CESM** - Community Earth System Model. Fully-coupled global climate model. CTSM is its land component.

**CLM** - Community Land Model. The land component within CTSM that models vegetation, hydrology, carbon cycling, etc.

**CIME** - Common Infrastructure for Modeling the Earth (pronounced "SEAM"). Provides the case control system for configuring, building, and running Earth system models.

## Tools & Utilities

**cprnc** - Tool for comparing NetCDF files. Used to verify model output. Location: `/blue/gerber/earth_models/shared/cprnc/bld/cprnc`

**git-fleximod** - CTSM's tool for managing submodules. Replaces the older `manage_externals` used by CESM.

**mksurfdata** - Tool for generating surface datasets (fsurdat files) for CTSM. Requires PIO library.

**PIO** (ParallelIO) - Parallel I/O library required for mksurfdata. Our shared build: `/blue/gerber/earth_models/shared/parallelio/bld`

## Configuration & Data

**CLM_USRDAT** - Resolution setting for user-provided datasets. Used with subset data for single-point runs.

**Compset** - Component set defining which model components to use and their configurations (e.g., `I1850Clm60BgcCrop`).

**Namelist** - Fortran configuration file format. Key files: `user_nl_clm`, `user_nl_datm_streams`.

**Surface data (fsurdat)** - NetCDF file containing land surface characteristics (soil, vegetation, topography).

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
