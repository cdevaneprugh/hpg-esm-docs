# CTSM Documentation

Due to conflicting info within the online documentation, shipped documentation, and CESM forum, I feel it necessary to create a document to untangle and clarify the available information.

## CLM Tools

From the [online documentation](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/using-clm-tools/what-are-the-clm-tools.html).

Located at `$CTSMROOT/tools`.

The list of generally important scripts and programs are as follows:

1. **DOESNT EXIST**`./mkmapgrids` to create SCRIP grid data files from old CLM format grid files that can then be used to create new CLM datasets (deprecated). 
2. There is also a NCL script (`./mkmapgrids/mkscripgrid.ncl`) to create SCRIP grid files for regular latitude/longitude grids.
3. **DOESNT EXIST**`./mkmapdata` to create SCRIP mapping data file from SCRIP grid files (uses ESMF).
4. `mksurfdata_esmf` to create surface datasets from grid datasets (clm4_0 and CTSM1 versions).
5. **DOESNT EXIST**`./mkprocdata_map` to interpolate output unstructured grids (such as the CAM HOMME dy-core “ne” grids like ne30np4) into a 2D regular lat/long grid format that can be plotted easily. Can be used by either clm4_0 or CTSM1.

## Creating Input for Surface Dataset Generation

### Generating SCRIP grid files

1. `mkmapdata.sh` needs SCRIP files which can be generated with `mkmapgrids`. **Neither script exists**

2. There is a NCL script (`$CTSMROOT/tools/mkmapgrids/mkscripgrid.ncl`) to create regular latitude longitude regional or single-point grids at the resolution the user desires.

SCRIP grid files for all the standard model resolutions and the raw surface datasets have already been done and the files are in the XML database. Hence, this step doesn’t need to be done – EXCEPT WHEN YOU ARE CREATING YOUR OWN GRIDS.

Use `mknoocnmap.pl` in `$CTSMROOT/tools/mkmapdata` to create a regular latitude/longitude single-point or regional grid. It will create both the SCRIP grid file you need (using `$CTSMROOT/tools/mkmapgrids/mkscripgrid.ncl`) AND an identity mapping file assuming there is NO ocean in your grid domain.

**`mkscripgrid.ncl` vs `mknoocnmap.pl`?** 

**What is an identity mapping file?**

### Creating mapping files for mksurfdata_esmf

`mkmapdata.sh` uses the above SCRIP grid input files to create SCRIP mapping data files with ESMF. **Script doesn't exist**

Theoretically it generates a list of maps from raw datasets that can be input to `mksurfdata_esmf`.

## Creating Surface Datasets

If you are just creating a replacement file (not sure why you would need to) just use the relevant tool. If you are creating a set of files for a new resolution, you must create a SCRIP grid file first to use as an input for the other tools.

<img src="https://escomp.github.io/ctsm-docs/versions/master/html/_images/mkmapdata_mksurfdata.jpeg" alt="img" style="zoom: 150%;" />

Basic idea:

1. Generate your SCRIP grid files using `mkscripgrid.ncl`.
2. Use `mkmapdata.sh` to create a list of SCRIP mapping files. **What to use instead?**
3. The mapping files will tell `mksurfdata_esmf` how to map between the grid and raw datasets.
4. The output of `mksurfdata_esmf` is a surface dataset used for running the model.

5. Enter the new datasets into the `build-namelist` XML database. This is optional, as you can enter things manually.

## Shipped Docs

From `$CTSMROOT/tools/README`

### Process sequence to create input datasets needed to run CTSM

1. Create SCRIP grid files (if needed)
   1. Use `mknoocnmap.pl` to create SCRIP grid files and a mapping file for single point or regional cases.
