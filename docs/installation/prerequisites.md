# Prerequisites

Required modules, libraries, and paths for running CTSM on HiPerGator.

## Directory Structure

Each research group should set up a directory structure for CTSM resources. Here's a recommended layout:

```
/blue/<group>/earth_models/
├── ctsm5.3/              # CTSM installation (your fork or clone)
├── inputdata/            # Global input data (downloaded from NCAR)
├── shared/
│   ├── cprnc/            # NetCDF comparison tool
│   └── parallelio/       # PIO library for mksurfdata
└── shared.subset.data/   # Subset data for single-point runs
```

!!! warning "Input Data Storage"
    CTSM downloads input data on-demand from NCAR servers. This can grow to **multiple terabytes** over time. Plan your storage allocation accordingly, and coordinate with your group to avoid duplicate downloads.

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

## Why Modules (Not Conda)

If you're familiar with conda, you might wonder if you can manage CTSM dependencies that way instead of using lmod. Here's why we use system modules for CTSM builds.

### HPC Build Dependencies Need System Tools

| Aspect | Conda | lmod (System Modules) |
|--------|-------|----------------------|
| MPI/SLURM integration | Problematic (PMIx often missing) | Native support, cluster-tuned |
| Compiler optimization | Generic builds | Tuned for node hardware |
| Library compatibility | May conflict with system libs | Configured by HPC admins |
| ESMF availability | Exists but may not match MPI | Built for cluster's MPI |
| Best for | Python packages, dev tools | Compiled HPC dependencies |

### ESMF Requires Real MPI

Modern CTSM (5.3+) uses CDEPS/CMEPS for atmospheric data coupling. This component **requires ESMF** (Earth System Modeling Framework), which in turn requires a real MPI implementation.

On HiPerGator, ESMF is built with OpenMPI and configured for SLURM. Conda's MPI packages often lack:

- **PMIx support** - Required for SLURM job launch
- **Cluster fabric optimization** - InfiniBand tuning, NUMA awareness
- **Consistent ABI** - May conflict with system libraries

**Result:** Even single-point runs that use only 1 MPI task still require OpenMPI because of ESMF.

!!! note "Why not mpi-serial?"
    CIME supports `mpi-serial` for truly serial execution, but it doesn't work with modern CTSM. The ESMF library links against OpenMPI symbols that mpi-serial can't provide. See [Fork Modifications](../reference/modifications.md#mpi-serial-incompatibility) for details.

### When Conda Is Appropriate

Conda works well for:

- **Python analysis packages** - xarray, matplotlib, pandas for post-processing
- **Development tools** - Editors, linters, formatters
- **CTSM Python tools** - The `ctsm_pylib` environment for subset_data and other scripts

We use a hybrid approach:

- **lmod modules** for compiled dependencies (compilers, MPI, NetCDF, ESMF)
- **conda environments** for Python-only tools and analysis

This gives you the reliability of system-optimized builds with the flexibility of conda for scripting.

## Shared Libraries

These libraries need to be built once per group and can be shared among group members.

### cprnc (NetCDF Comparison Tool)

Used to compare NetCDF output files. Useful for validating model output.

### ParallelIO (PIO)

PIO (Parallel I/O) is a library that provides a high-level interface to NetCDF and other I/O formats. CTSM uses PIO for:

- **mksurfdata_esmf** - The tool that generates surface datasets
- **CTSM runtime** - Reading input data and writing history output

!!! info "Why Build PIO Separately?"
    CTSM can build PIO automatically during `case.build`, but this rebuilds it for every new case. Building PIO once as a shared library saves significant time. With proper configuration (see [CIME Configuration](cime-config.md#pio-shared-library)), CTSM uses your pre-built PIO instead of rebuilding.

!!! warning "Shared Libraries Required"
    PIO must be built with `BUILD_SHARED_LIBS=ON` for mksurfdata to work correctly. The upstream CMakeLists.txt has a bug that declares libraries as STATIC but expects shared `.so` files. See [Fork Modifications](../reference/modifications.md#1-cmakeliststxt---static-vs-shared) for details.

Set the `PIO` environment variable to point to your build:

```bash
export PIO="/blue/<group>/earth_models/shared/parallelio/bld"
```

## Building Shared Libraries

Build these once for your group. Each group member can then use the same installation.

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

**Key CMake flags:**

| Flag | Purpose |
|------|---------|
| `BUILD_SHARED_LIBS=ON` | **Required** - Creates `.so` files instead of `.a` |
| `WITH_PNETCDF=OFF` | Skip parallel-netcdf (not needed for CTSM) |
| `CMAKE_INSTALL_RPATH` | Ensures runtime library paths are embedded |

After building, verify you have the shared libraries:

```bash
ls bld/lib/*.so
# Should show: libgptl.so libpioc.so libpiof.so
```

## Environment Variables

See [Onboarding](../onboarding.md#recommended-shortcuts) for recommended environment variables. Key variables for prerequisites:

```bash
# Shared libraries (customize path)
export PIO="/blue/<group>/earth_models/shared/parallelio/bld"
```

## Verification

After building the shared libraries, verify your setup:

```bash
# Check modules
module restore ctsm-modules
module list

# Check PIO was built correctly
ls $PIO/lib/libpiof.so
```
