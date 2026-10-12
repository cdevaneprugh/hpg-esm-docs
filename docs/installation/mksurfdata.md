# Building mksurfdata

The `mksurfdata` tool generates surface datasets (fsurdat files) for CTSM. This page covers building the tool on HiPerGator.

## Prerequisites

Before building mksurfdata, ensure you have:

1. Module collection loaded: `module restore ctsm-modules`
2. PIO library built and path set (see [Prerequisites](prerequisites.md))
3. CTSM fork cloned (fixes are already applied)

## Building the Executable

```bash
# Load modules
module restore ctsm-modules

# Set PIO path (customize for your group)
export PIO="/blue/<group>/earth_models/shared/parallelio/bld"

# Navigate to mksurfdata directory
cd $CTSMROOT/tools/mksurfdata_esmf

# Build for HiPerGator
./gen_mksurfdata_build --machine hipergator
```

Expected output:
```
Successfully created mksurfdata_esmf executable for: hipergator_gnu for openmpi library
```

## Verify the Build

Check that PIO libraries are properly linked:

```bash
cd tool_bld
ldd mksurfdata | grep libpio
```

Expected output (paths will match your PIO location):
```
libpiof.so => /blue/<group>/earth_models/shared/parallelio/bld/lib/libpiof.so
libpioc.so => /blue/<group>/earth_models/shared/parallelio/bld/lib/libpioc.so
```

If you see "not found", the libraries weren't linked correctly.

## Troubleshooting

### Build Errors

If the build fails:

1. Delete the build directory: `rm -rf tool_bld`
2. Verify modules are loaded: `module list`
3. Verify PIO path: `ls $PIO/lib/libpiof.so`
4. Retry the build

### Common Issues

| Error | Cause | Solution |
|-------|-------|----------|
| "libpio not found" | PIO not in path | Set `export PIO=...` before building |
| Format string errors | Using unfixed upstream | Use our fork with fixes applied |
| CMake errors | Wrong module versions | Restore `ctsm-modules` collection |

## Fixes Applied in Our Fork

Upstream mksurfdata in `ctsm5.3.x` has bugs that prevent building on HiPerGator. Our fork includes these fixes. Fixes 1 and 2 were merged upstream and are included from `ctsm5.4.025` onward; fix 3 is fork-only.

### 1. CMakeLists.txt - STATIC vs SHARED

**Problem:** Declares libraries as STATIC but points to shared `.so` files.

**Fix:** Change `STATIC IMPORTED` to `SHARED IMPORTED`:

```cmake
# Before (buggy)
add_library(pioc STATIC IMPORTED)
add_library(piof STATIC IMPORTED)

# After (fixed)
add_library(pioc SHARED IMPORTED)
add_library(piof SHARED IMPORTED)
```

**File:** `tools/mksurfdata_esmf/src/CMakeLists.txt` lines 51-52 (upstream: [#3700](https://github.com/ESCOMP/CTSM/pull/3700), `ctsm5.4.021`)

### 2. mksurfdata.F90 - Format Specifiers

**Problem:** gfortran requires an explicit width in `I` edit descriptors. The `I` format without width is a legacy extension.

**Fix:** Use `I0` (minimal width) for the writes and an explicit width for the read:

```fortran
! Before (buggy)
write(ndiag,'(2(a,I))') ' npes = ', npes, ' grid size = ', grid_size
read(nfpio, '(i)', iostat=ier) pio_iotype

! After (fixed)
write(ndiag,'(2(a,I0))') ' npes = ', npes, ' grid size = ', grid_size
read(nfpio, '(I6)', iostat=ier) pio_iotype
```

**File:** `tools/mksurfdata_esmf/src/mksurfdata.F90` lines 271 and 328 (writes), 295 (read) (upstream: [#3701](https://github.com/ESCOMP/CTSM/pull/3701), `ctsm5.4.025`)

### 3. gen_mksurfdata_build - GCC Compatibility Flags

**Problem:** Since GCC 10, gfortran treats argument type mismatches and BOZ literals outside `DATA` statements as errors. CIME case builds get compatibility flags from the stock `gnu.cmake` macros, but mksurfdata's own CMake build does not.

**Fix:** Add compatibility flags, which downgrade those errors to warnings:

```bash
-DCMAKE_Fortran_FLAGS=" -fallow-argument-mismatch -fallow-invalid-boz -ffree-line-length-none"
```

**File:** `tools/mksurfdata_esmf/gen_mksurfdata_build` line 170

## Python Environment

Some mksurfdata workflows require the CTSM Python environment:

```bash
module load conda
cd $CTSMROOT
./py_env_create
conda activate ctsm_pylib
```

This creates a conda environment with CTSM's Python dependencies.

## Next Steps

After building mksurfdata:

- See [Single-Point Runs](../running-ctsm/single-point.md) for using subset data
- See the CTSM documentation for creating custom surface datasets
