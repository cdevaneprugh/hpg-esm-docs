# Prerequisites

Required modules, libraries, and paths for running CTSM on HiPerGator.

## Shared Directory Structure

The Gerber group maintains shared resources at:

```
/blue/gerber/earth_models/
├── ctsm5.3/              # CTSM installation (fork)
├── inputdata/            # Global input data
├── shared/
│   ├── cprnc/            # NetCDF comparison tool
│   └── parallelio/       # PIO library for mksurfdata
└── shared.subset.data/   # Subset data for single-point runs
```

## Module Environment

CTSM requires specific compiler and library versions. We maintain a module collection:

### Setting Up the Module Collection

```bash
module purge

# Load required modules
module load subversion/1.9.7
module load perl/5.24.1
module load cmake/3.26.4
module load python/3.12
module load gcc/14.2.0
module load lapack/3.11.0
module load openmpi/5.0.7
module load netcdf-c/4.9.3 netcdf-f/4.6.2
module load hdf5
module load esmf/8.8.1

# Save the collection
module save ctsm-modules
```

### Using the Collection

```bash
# Restore modules (do this at the start of each session)
module restore ctsm-modules

# Verify
module list
```

## Shared Libraries

### cprnc (NetCDF Comparison Tool)

Used to compare NetCDF output files. Already built at:

```
/blue/gerber/earth_models/shared/cprnc/bld/cprnc
```

### ParallelIO (PIO)

Required for building mksurfdata. Our shared build:

```
/blue/gerber/earth_models/shared/parallelio/bld
```

Set the environment variable before building mksurfdata:

```bash
export PIO="/blue/gerber/earth_models/shared/parallelio/bld"
```

## Building Shared Libraries (Reference)

These libraries are already built for the Gerber group. This section documents how they were installed.

### Building cprnc

```bash
module restore ctsm-modules

git clone https://github.com/ESMCI/cprnc.git
cd cprnc
mkdir bld && cd bld
cmake ../
make
```

### Building PIO

```bash
module restore ctsm-modules

git clone https://github.com/NCAR/ParallelIO.git parallelio
cd parallelio
mkdir bld

cmake \
  -DNetCDF_C_PATH=/apps/gcc/14.2.0/openmpi/5.0.7/netcdf-c/4.9.3 \
  -DNetCDF_Fortran_PATH=/apps/gcc/14.2.0/openmpi/5.0.7/netcdf-f/4.6.2 \
  -DWITH_PNETCDF=OFF \
  -DBUILD_SHARED_LIBS=ON \
  -DCMAKE_INSTALL_PREFIX=`pwd`/bld \
  -DCMAKE_INSTALL_RPATH=`pwd`/src/gptl \
  -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=TRUE \
  ../

make
make install
```

## Environment Variables

Add to your `~/.bashrc`:

```bash
# User directories
export BLUE="/blue/gerber/$USER"
export CASES="$BLUE/cases"
export ESM_OUTPUT="$BLUE/earth_model_output/cime_output_root"

# CTSM installation
export CTSMROOT="/blue/gerber/cdevaneprugh/ctsm5.3"
export CIME_SCRIPTS="$CTSMROOT/cime/scripts"

# Shared data
export INPUT_DATA="/blue/gerber/earth_models/inputdata"
export SUBSET_DATA="/blue/gerber/earth_models/shared.subset.data"

# Shared libraries
export PIO="/blue/gerber/earth_models/shared/parallelio/bld"
```

Reload after editing: `source ~/.bashrc`

## Verification

Verify your setup:

```bash
# Check modules
module restore ctsm-modules
module list

# Check paths exist
ls $CTSMROOT
ls $INPUT_DATA
ls $PIO/lib/libpiof.so

# Check cprnc
/blue/gerber/earth_models/shared/cprnc/bld/cprnc --help
```
