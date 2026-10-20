# Fork Modifications

This page documents modifications made in our forks and their rationale.

## Overview

Our forks include HiPerGator-specific changes, bug fixes for building tools, and one research namelist parameter:

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | Tool fixes, paths, `spillheight` namelist parameter (retired) |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | Machine config |
| [cdevaneprugh/NCAR-NEON](https://github.com/cdevaneprugh/NCAR-NEON) | `uf-osbs` | NEON→DATM forcing generator (offline fork; AGPL-3.0) |

The first two are CTSM build/machine forks (detailed below). **NCAR-NEON** is a different kind of fork — the NEON atmospheric-forcing generator (`flow.api.clm.R`), modified to run fully offline on HiPerGator to *produce input data* rather than build the model. It is AGPL-3.0 (kept separate from the MIT `hpg-esm-tools` repository for that reason) and is documented in full, with the edit table, at [NEON Atmospheric Forcing](../swenson/neon-forcing.md).

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

**Upstream status:** Merged as [ESCOMP/CTSM #3700](https://github.com/ESCOMP/CTSM/pull/3700) (2026-02-18), first released in `ctsm5.4.021`.

### 2. mksurfdata.F90 - Format Specifiers

**File:** `tools/mksurfdata_esmf/src/mksurfdata.F90`

**Problem:** gfortran requires an explicit width in Fortran `I` edit descriptors. The `I` format without a width is a legacy extension:

```fortran
! Buggy - no width specified
write(ndiag,'(2(a,I))') ' npes = ', npes, ' grid size = ', grid_size
read(nfpio, '(i)', iostat=ier) pio_iotype
```

**Fix:** Use `I0` (minimal width) for the two writes and an explicit width for the read:

```fortran
write(ndiag,'(2(a,I0))') ' npes = ', npes, ' grid size = ', grid_size              ! line 271
read(nfpio, '(I6)', iostat=ier) pio_iotype                                        ! line 295
write (ndiag,'(a, I0, a, I0)') ' node_count = ', node_count, ' grid_size = ', grid_size  ! line 328
```

**Lines modified:** 271 and 328 (writes), 295 (read)

**Why it works at NCAR:** They likely use Intel compilers, which accept the width-less form.

**Upstream status:** Merged as [ESCOMP/CTSM #3701](https://github.com/ESCOMP/CTSM/pull/3701) (2026-03-12), first released in `ctsm5.4.025`. The upstream lines are identical to the fork's.

### 3. gen_mksurfdata_build - GCC Compatibility Flags

**File:** `tools/mksurfdata_esmf/gen_mksurfdata_build`

**Problem:** Since GCC 10, gfortran treats two legacy patterns as hard errors rather than warnings: mismatched argument types between calls to the same procedure (for example MPI calls through `mpif.h`), and BOZ literal constants used outside `DATA` statements. CIME case builds already receive the relevant flags from the stock `ccs_config/machines/cmake_macros/gnu.cmake` (for GCC 10 and later), but mksurfdata has its own CMake build that does not use those macros. HiPerGator builds with GCC 14.2.0.

**Fix:** Pass the compatibility flags to the mksurfdata CMake configure step (line 170):

```bash
-DCMAKE_Fortran_FLAGS=" -fallow-argument-mismatch -fallow-invalid-boz -ffree-line-length-none"
```

**Flags explained:**
- `-fallow-argument-mismatch`: Downgrade argument type mismatches from errors to warnings
- `-fallow-invalid-boz`: Downgrade invalid BOZ literal constants from errors to warnings
- `-ffree-line-length-none`: Remove the 132-column free-form line limit

**Upstream status:** Not upstreamed. These are workarounds, not fixes for the underlying code patterns.

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

CTSM 5.3+ uses CDEPS (Community Data models for Earth Prediction Systems) for atmospheric data and CMEPS (the NUOPC-based Community Mediator for Earth Prediction Systems) for coupling, replacing the MCT-based data models and CPL7 coupler. CDEPS requires **ESMF** (Earth System Modeling Framework).

On HiPerGator, ESMF is built with OpenMPI. When you try to build CTSM with `MPILIB=mpi-serial`:

```text
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

### 6. namelist_definition_ctsm.xml - spillheight (research parameter, retired)

**File:** `bld/namelist_files/namelist_definition_ctsm.xml`

**Change:** A new namelist entry `spillheight` (type `real`, group `hillslope_properties_inparm`, no default in the definition), added 2026-01-12 (commit `4a0319fee`):

```xml
<entry id="spillheight" type="real" category="physics"
        group="hillslope_properties_inparm">
Height threshold (m) for water to accumulate before draining to stream.
Subtracts from hillslope_elevation to create a bank effect.
</entry>
```

**Reason:** Stock CTSM has no such parameter. It exists only in the PI's `HillslopeHydrologyMod.F90` SourceMod, which reads it from the `hillslope_properties_inparm` group and subtracts it from every hillslope column's `hill_elev` at initialization (default 0.2 m in the SourceMod). The fork entry lets the value be set from `user_nl_clm` instead of by editing the SourceMod; without it, CTSM's namelist validation rejects the variable. It is only meaningful in a case that also carries that SourceMod.

**Status:** Retired. The offset was superseded by raw-HAND binning on 2026-04-30 and every production case sets `spillheight = 0.0`; the PI later removed the parameter from his SourceMods (2026-05-19). The entry is kept so that existing cases' `user_nl_clm` files remain valid. See [SPILLHEIGHT: the retired runtime offset](../swenson/hand-binning-and-lake-column.md#spillheight-the-retired-runtime-offset).

**Contribution candidate:** No - research-specific and retired.

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

```text
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
| CMakeLists STATIC/SHARED | Merged ([#3700](https://github.com/ESCOMP/CTSM/pull/3700), `ctsm5.4.021`) | Still needed on 5.3.x |
| Format specifiers | Merged ([#3701](https://github.com/ESCOMP/CTSM/pull/3701), `ctsm5.4.025`) | Still needed on 5.3.x |
| GCC compatibility flags | Not upstreamed | Local workaround |
| HiPerGator paths | Not applicable | Site-specific |
| `spillheight` namelist entry | Not applicable | Research-specific; retired, set to 0.0 in all production cases |
| ccs_config | Not applicable | Site-specific |

## Reference

- [GCC Format Descriptors](https://gcc.gnu.org/onlinedocs/gfortran/Default-widths-for-F_002fG_002fI-format-descriptors.html)
- [CESM Forum: PIO Errors](https://bb.cgd.ucar.edu/cesm/threads/the-error-about-pio.9238/)
