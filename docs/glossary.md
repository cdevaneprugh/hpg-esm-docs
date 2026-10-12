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

**Compset** - Component set. A predefined combination of model components and configurations. Each compset has a short alias and a long name of the form `TIME_ATM_LND_ICE_OCN_ROF_GLC_WAV[_BGC]`, where each component may carry a `%` modifier. Example: the alias `I1850Clm60BgcCrop` has the long name `1850_DATM%GSWP3v1_CLM60%BGC-CROP_SICE_SOCN_MOSART_SGLC_SWAV` (pre-industrial 1850, data atmosphere, CLM 6.0 with BGC and crops).

**Namelist** - Fortran configuration file format. Key files: `user_nl_clm`, `user_nl_datm_streams`.

**PFT** - Plant Functional Type. Categories of vegetation (e.g., broadleaf deciduous tree, C3 grass, corn) that share similar characteristics. CTSM uses PFTs to represent vegetation at each grid point.

**Surface data (fsurdat)** - NetCDF file containing land surface characteristics (soil, vegetation, topography).

## Output Files

**History files** - Model output files containing time-varying data (`.clm2.h0a.*.nc`, `.clm2.h1a.*.nc`, etc.; since ctsm5.3.062 averaged and instantaneous fields go to separate `hXa` and `hXi` files). Configured via `hist_*` namelist variables. Use for analysis and plotting.

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

**AD spinup** - Accelerated Decomposition spinup. First phase (~200 years is the NCAR guideline; site runs may differ) with accelerated soil carbon turnover.

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

## Hillslope and OSBS Project Terms

Terms used in the [Swenson Implementation](swenson/index.md) and [Research](research/overview.md) pages.

**A_thresh** - Flow-accumulation threshold (in pixels) above which a cell is treated as stream. Set to `int(0.5 × Lc²)`; 63,362 pixels for OSBS.

**Chain index** - A hillslope column's 1-based position in the downhill chain (`column_index`). In the OSBS file, index 1 is the lake column (terminal, drains out of the hillslope), indices 2-25 are land Bins 1-24, and each column drains to the one below it.

**CDEPS** - Community Data models for Earth Prediction Systems. The NUOPC-based data components (including DATM) and their stream-reading code.

**DATM** - Data atmosphere component. Reads prescribed atmospheric forcing (e.g., CRUNCEP, NEON tower data) from stream files instead of running an atmosphere model.

**DTND** - Distance To Nearest Drainage. Straight-line horizontal distance from a pixel to the stream or open-water cell it drains to.

**FZ** - Flood zone. The 12 OSBS hillslope bins with raw HAND below 0 (below the level of the drainage cell); the other 12 are upland bins.

**HAND** - Height Above Nearest Drainage. Elevation of a pixel relative to the stream or open-water cell it drains to. Conditioned HAND is computed on the pit-filled, flat-resolved DEM and is ≥ 0. Raw HAND subtracts the depression-fill depth, so it is negative inside filled depressions.

**Lc** - Characteristic length scale of the terrain, from the peak of the Laplacian-of-elevation spectrum. 356 m (355.98 m) for the OSBS production domain.

**MERIT** - MERIT DEM, a ~90 m global elevation model. Swenson & Lawrence (2025) built the global hillslope dataset from it; this project reproduces that dataset to verify the pipeline.

**NWI** - National Wetlands Inventory (US Fish & Wildlife Service). Source of the OSBS open-water mask used for the lake column.

**O_SCALAR** - CTSM history field: the fraction by which soil decomposition is reduced due to anoxia (1 = no reduction). Only active when `use_lch4 = .true.`.

**OSBS** - Ordway-Swisher Biological Station, the NEON site in north-central Florida used as the test site.

**Phase (A-I)** - Work phases of the OSBS hillslope project, recorded in [`swenson/phases/`](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/phases) in hpg-esm-tools. Phase H Track A is the single-cell mesh workaround for CTSM Issue #1432 (see [Lateral Flow and Routing](swenson/lateral-flow-and-routing.md)).

**Q01 / Q99** - 1st and 99th percentiles of raw HAND over the domain (-6.35 m and +17.02 m in production). Pixels outside this range are discarded before binning.

**R4C5-R12C14** - The production domain: NEON 1 km DTM tile rows 4-12 and columns 5-14 (90 tiles, 9 × 10 km, no missing data).

**SourceMod** - A modified copy of a CTSM source file placed in a case's `SourceMods/src.clm/`, compiled in place of the original for that case only.

**TAI** - Terrestrial-aquatic interface. The shifting boundary between wet and dry ground in low-relief wetland landscapes; the project's central scientific target.
