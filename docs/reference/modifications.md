# Fork Modifications

This page documents modifications made in our CTSM fork and their rationale.

## Overview

Our forks include HiPerGator-specific changes and bug fixes for building tools:

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | Tool fixes, paths |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | Machine config |

## CTSM Modifications

### 1. CMakeLists.txt - STATIC vs SHARED

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
clmforcingindir = /blue/gerber/earth_models/inputdata
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
- Shared PIO path
- No `--exclusive` flag (HiPerGator uses shared nodes)
- Gerber group queues

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
