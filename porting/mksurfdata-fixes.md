# CTSM File Modifications

We need to modify several buggy files in CTSM in order to build `mksurfdata`. I have forked CTSM and fixed these bugs in the uf repo. If that is not available for some reason, here is how to fix the existing code. 

First go to the directory at `$CTSMROOT/tools/mksurfdata_esmf`.

## CMakeLists.txt

Then modify the `CMakeLists.txt` file at `CTSMROOT/tools/mksurfdata_esmf/src/CMakeLists.txt`.
`CMakeLists.txt` is a configuration file that CMake uses to define the build process, including source files, dependencies, build targets, and project settings.

At lines 51 and 52, change the word STATIC to SHARED so the lines read as follows.

```
add_library(pioc SHARED IMPORTED)
add_library(piof SHARED IMPORTED)
set_property(TARGET pioc PROPERTY IMPORTED_LOCATION $ENV{PIO}/lib/libpioc.so)
set_property(TARGET piof PROPERTY IMPORTED_LOCATION $ENV{PIO}/lib/libpiof.so)
```

If you notice in the bottom two lines, we are looking for the shared libraries `libpioc.so` and `libpiof.so`. This conflicted with the previous two lines looking for static libraries.

## mksurfdata.F90

Next we have to correct some Fortran syntax errors in `src/mksurfdata.F90`. If you try to compile you will be met with the following errors.

```
/ctsm5.3/tools/mksurfdata_esmf/src/mksurfdata.F90:271:24:

  271 |      write(ndiag,'(2(a,I))') ' npes = ', npes, ' grid size = ', grid_size
      |                        1
Error: Nonnegative width required in format string at (1)

/ctsm5.3/tools/mksurfdata_esmf/src/mksurfdata.F90:295:19:

  295 |      read(nfpio, '(i)', iostat=ier) pio_iotype
      |                   1
Error: Nonnegative width required in format string at (1)

/ctsm5.3/tools/mksurfdata_esmf/src/mksurfdata.F90:328:27:

  328 |         write (ndiag,'(a, I, a, I)') ' node_count = ', node_count, ' grid_size = ', grid_size
      |                           1
Error: Nonnegative width required in format string at (1)
```

To fix this we make the following changes at lines 271, 295, and 328 respectively.

```
271 |      write(ndiag,'(2(a,I10))') ' npes = ', npes, ' grid size = ', grid_size

295 |      read(nfpio, '(I5)', iostat=ier) pio_iotype

328 |         write (ndiag,'(a, I10, a, I10)') ' node_count = ', node_count, ' grid_size = ', grid_size
```

## Build Script

Finally, modify line 170 of the `gen_mksurfdata_build` script to the following.

```
CC=mpicc FC=mpif90 cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_Fortran_FLAGS=" -fallow-argument-mismatch -fallow-invalid-boz -ffree-line-length-none" $options $cwd/src
```
