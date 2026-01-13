# Fork Modifications

This page documents modifications made in our CTSM fork and their rationale.

## Overview

Our forks include HiPerGator-specific changes and bug fixes for building tools:

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | Tool fixes, paths |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | Machine config |

## CTSM Modifications

### 1. CMakeLists.txt - STATIC vs SHARED {#1-cmakeliststxt---static-vs-shared}

**File:** `tools/mksurfdata_esmf/src/CMakeLists.txt`

**Problem:** Upstream declares libraries as `STATIC IMPORTED` but points to shared `.so` files:

```cmake
# Buggy upstream code
add_library(pioc STATIC IMPORTED)
set_property(TARGET pioc PROPERTY IMPORTED_LOCATION $ENV{PIO}/lib/libpioc.so)
```

This is an internal inconsistency - you cannot declare `STATIC` and point to a shared library.

**Fix:** Change to `SHARED IMPORTED`:

```cmake
add_library(pioc SHARED IMPORTED)
add_library(piof SHARED IMPORTED)
```

**Why it works at NCAR:** Their systems likely have PIO as static libraries, or use different CMake handling.

**Contribution candidate:** Yes - genuine bug affecting anyone using shared PIO.

### 2. mksurfdata.F90 - Format Specifiers

**File:** `tools/mksurfdata_esmf/src/mksurfdata.F90`

**Problem:** GCC 10+ requires explicit width in Fortran format specifiers. The `I` format without width is a legacy extension:

```fortran
! Buggy - no width specified
write(ndiag,'(2(a,I))') ' npes = ', npes, ' grid size = ', grid_size
```

**Fix:** Add explicit widths:

```fortran
write(ndiag,'(2(a,I12))') ' npes = ', npes, ' grid size = ', grid_size
```

**Lines modified:** 271, 295, 328

**Why it works at NCAR:** They likely use Intel compilers or older GCC versions.

**Contribution candidate:** Yes - portability fix for modern GCC.

### 3. gen_mksurfdata_build - GCC 14 Flags

**File:** `tools/mksurfdata_esmf/gen_mksurfdata_build`

**Problem:** GCC 14 is stricter about legacy Fortran code patterns.

**Fix:** Add compatibility flags:

```bash
-DCMAKE_Fortran_FLAGS=" -fallow-argument-mismatch -fallow-invalid-boz -ffree-line-length-none"
```

**Flags explained:**
- `-fallow-argument-mismatch`: Allow type mismatches in procedure calls
- `-fallow-invalid-boz`: Allow invalid BOZ literal constants
- `-ffree-line-length-none`: No line length limit

**Contribution candidate:** Maybe - these are workarounds, not fixes for underlying issues.

### 4. single_point_case.py - MPILIB

**File:** `python/ctsm/site_and_regional/single_point_case.py`

**Change:** `MPILIB=mpi-serial` → `MPILIB=openmpi`

**Reason:** HiPerGator uses OpenMPI, not mpi-serial.

**Contribution candidate:** No - HiPerGator-specific.

### mpi-serial Incompatibility

!!! warning "Why We Can't Use mpi-serial"
    This section explains why single-point runs still require OpenMPI, even though CIME supports `mpi-serial` for truly serial execution.

**What is mpi-serial?**

`mpi-serial` is a stub MPI library that provides MPI function signatures without actual parallelization. It's designed for running MPI-dependent code on a single processor without a real MPI installation.

**Why we investigated it:**

- Single-point CTSM runs use only 1 MPI task
- Interactive execution without SLURM would be convenient for testing
- Simpler build environment (no MPI dependencies)

**Why it doesn't work with modern CTSM:**

CTSM 5.3+ uses CDEPS/CMEPS (Community Data Models for Earth Prediction Systems) for atmospheric data coupling, replacing the older data model. CDEPS requires **ESMF** (Earth System Modeling Framework).

On HiPerGator, ESMF is built with OpenMPI. When you try to build CTSM with `MPILIB=mpi-serial`:

```
# Build fails at link stage:
/apps/.../libesmf.a: undefined reference to symbol 'ompi_mpi_unsigned_short'
/apps/.../libmpi.so.40: error adding symbols: DSO missing from command line
```

The ESMF library contains calls to OpenMPI-specific symbols that mpi-serial doesn't provide.

**The workaround:**

Single-point runs work fine with OpenMPI using `NTASKS=1`. You still need to submit jobs via SLURM, but the job runs on a single core. This is how our fork is configured.

**Bottom line:** If you're coming from an older CTSM version or another machine where mpi-serial worked, be aware it's not an option with modern CTSM on HiPerGator.

### 5. default_data_*.cfg - Input Paths

**Files:** `tools/site_and_regional/default_data_2000.cfg`, `default_data_1850.cfg`

**Change:** NCAR paths → HiPerGator paths

**Example:**
```ini
# Before (NCAR)
[main]
clmforcingindir = /glade/campaign/...

# After (HiPerGator)
[main]
clmforcingindir = /blue/<group>/earth_models/inputdata
```

**Contribution candidate:** No - site-specific.

## ccs_config Modifications

### Machine Configuration

**Files in `machines/hipergator/`:**
- `config_machines.xml` - Machine definition
- `config_batch.xml` - SLURM settings
- `gnu_hipergator.cmake` - Compiler flags

**Key changes:**

- GCC 14.2.0, OpenMPI 5.0.7, ESMF 8.8.1
- Shared PIO library configuration (see below)
- No `--exclusive` flag (HiPerGator uses shared nodes)
- Group-specific QoS queues (customize for your group)

### Shared PIO Library Support

The fork configures CTSM to use a pre-built PIO library instead of rebuilding it for every case. This is controlled by environment variables in `config_machines.xml`:

```xml
<env name="PIO_VERSION_MAJOR">2</env>
<env name="PIO_TYPENAME_VALID_VALUES">netcdf</env>
<env name="LD_LIBRARY_PATH">/blue/<group>/earth_models/shared/parallelio/bld/lib:$ENV{LD_LIBRARY_PATH}</env>
```

**How CIME decides whether to use external PIO:**

```
if PIO_VERSION_MAJOR is set AND matches case PIO_VERSION:
    → use external PIO from PIO_LIBDIR
    → build log shows "Using installed PIO library"
else:
    → build PIO from source into EXEROOT/shared/pio2
    → adds several minutes to each case.build
```

**Variable purposes:**

| Variable | Purpose |
|----------|---------|
| `PIO_VERSION_MAJOR` | Triggers external PIO usage (value must match your PIO version) |
| `PIO_TYPENAME_VALID_VALUES` | Specifies I/O backends (our build supports `netcdf` only) |
| `LD_LIBRARY_PATH` | Required for runtime linking to shared `.so` files |

See [CIME Configuration](../installation/cime-config.md#pio-shared-library) for more details.

### Why Fork ccs_config?

The `~/.cime/` user override approach has CIME v3 bugs:
- Batch config loading inconsistent
- NODENAME_REGEX detection issues
- Each user must maintain configs

Forking gives a single source of truth.

## Upstream Contribution Status

| Modification | Status | Notes |
|--------------|--------|-------|
| CMakeLists STATIC/SHARED | Planned | Clear bug |
| Format specifiers | Planned | Portability fix |
| GCC 14 flags | Under review | Workarounds vs fixes |
| HiPerGator paths | Not applicable | Site-specific |
| ccs_config | Not applicable | Site-specific |

## Reference

- [GCC Format Descriptors](https://gcc.gnu.org/onlinedocs/gfortran/Default-widths-for-F_002fG_002fI-format-descriptors.html)
- [CESM Forum: PIO Errors](https://bb.cgd.ucar.edu/cesm/threads/the-error-about-pio.9238/)
