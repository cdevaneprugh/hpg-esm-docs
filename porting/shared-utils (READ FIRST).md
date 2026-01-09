# Shared Utilities and Setup Guide

## 1. General Directory Structure

The **Gerber** group maintains a shared directory on HiPerGator for Earth system models at:

```
/blue/gerber/earth_models
```

Note that there are many ways your research group could decide to set this up, this is just what we have chosen to do.

This directory contains:

- **Earth model source code** (e.g., CESM, CTSM)
- **Input data** (boundary conditions, forcing datasets)
- **Shared utilities** (e.g., `cprnc`, `parallelio`, custom scripts)

```bash
earth_models/
├── cesm2.1.5			# cesm earth model root directory
├── config				# config files for each model as well as utility scripts
│   ├── cesm2.1
│   ├── ctsm5.3
│   └── hpg
├── ctsm5.3				# ctsm earth model root directory
├── docs				# documentation (you are here)
├── inputdata			# input data shared between models
├── README
├── scripts				# misc. scripts
└── shared				# shared libraries and utilities
    ├── cprnc
    └── parallelio
```

## 2. Required Modules and Environment Setup

### **Dedicated Module Environment**

For consistency, we define a **dedicated module collection** for Earth system modeling. This setup ensures the correct libraries are loaded every time.

There is a script in this repo located at `scripts/module.env.setup` that will set the environment up with the name "ctsm-modules".

The steps to do it manually are:
```bash
module purge  # Remove conflicting modules

# Load essential dependencies
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
module save MY_MODULE_COLLECTION

# Verify correct setup
module purge
module restore MY_MODULE_COLLECTION
module list
```

Using this saved environment minimizes issues when running scripts and compiling models. 

## 3. Configuring CIME on HiPerGator

CIME (Common Infrastructure for Modeling the Earth) manages the configuration, build, and execution of Earth models. There are three critical configuration files: one for compilers, one for the batch system, and one for the computing environment.

There are two ways to configure these files for CIME:

1. **Modify** the three files located in `$CIMEROOT/config/$model/machines/` by adding entries for HiPerGator.

2. Alternatively, create standalone personal config files in 

   ```
   ~/.cime/
   ```

   These append onto the default files when the model runs.

   ```bash
   ~/.cime/
   ├── config_batch.xml
   ├── config_compilers.xml
   └── config_machines.xml
   ```

If you plan on installing only one Earth model, option two is recommended. Basic HiPerGator configuration files can be found in the `config` directory of this repository. Due to subtle variations in config files between the Earth models and releases, these may require some modification. The Gerber group, uses multiple models, each using different CIME versions. Therefore, we have opted to fork the CTSM repository and incorporate configuration files there to simplify installation. Users only need to update relevant paths to match their research group and personal directories if they wish to run CTSM. It's possible we will fork CESM in the future as well.

### Example

Say you installed `cesm` in a directory named `my-cesm-install`. Here are the two possible methods of setting up the configuration files.

__Method 1: Appending Default files with HiPerGator Settings__

Using your text editor of choice, copy the contents of `config_machines.xml`, `config_batch.xml`, and `config_compilers.xml` (or `gnu_hipergator.cmake` depending on which version of cesm you are porting) and add them to the end of their matching files located in `my-cesm-install/cime/config/cesm/machines`. The `config_compilers.xml` file should generally be left alone. `config_batch.xml` will need to be edited to match your group's QoS limits. `config_machines.xml` will need to be edited to match your group's directory structure.

__Method 2: Add .cime to Your Home Directory__

```bash
# create your .cime directory
mkdir ~/.cime

# from the root of this directory copy the config files that correspond to the model you are porting
cp -r config/cesm ~/.cime
```

Once again, make sure to modify the configs to match your group's QoS and directory structure.

------

## 4. Installing Shared Libraries

### 4.1 For The Gerber Group

The utilities in this section have already been built. Sections 4.2 and 4.3 are meant to provide a record of how they were installed, as well as give guidance to other HiPerGator research groups.

### 4.2 Installing the `cprnc` Tool

`cprnc` compares **NetCDF** output files. While not required for running Earth models, it is recommended.

### **Installation Steps**

Ensure required modules are loaded:

```bash
module load gcc/14.2.0 openmpi/5.0.7 netcdf-c/4.9.3 netcdf-f/4.6.2
```

Clone and build:

```bash
git clone https://github.com/ESMCI/cprnc.git cprnc

cd cprnc

mkdir bld
cd bld
cmake ../
make
```

The executable will be located at:

```
cprnc/bld/cprnc
```

## 4.3 Building Parallel I/O (PIO)

If you plan on making your own surface datasets via the `mksurfdata` utility, this library is required. The location you install it in is up to you. We opted to [clone the repo](https://github.com/NCAR/ParallelIO) and build in  `earth_models/shared/parallelio` , as we have multiple Earth models running in parallel that may need this library. If you are only running one model, you could install it in the default directory at `$CTSMROOT/libraries/parallelio`.

There is also a script provided to do this located at `/scripts/build.pio`

```bash
# restore defalut modules
module restore MY_MODULE_COLLECTION

# cd to library location
cd $CTSMROOT/libraries/parallelio
mkdir bld

cmake \
  -DNetCDF_C_PATH=/apps/gcc/12.2.0/openmpi/5.0.7/netcdf-c/4.9.3 \
  -DNetCDF_Fortran_PATH=/apps/gcc/12.2.0/openmpi/5.0.7/netcdf-f/4.6.2 \
  -DWITH_PNETCDF=OFF \
  -DBUILD_SHARED_LIBS=ON \
  -DCMAKE_INSTALL_PREFIX=`pwd`/bld \
  -DCMAKE_INSTALL_RPATH=`pwd`/src/gptl \
  -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=TRUE \
  ../

make
make install
```

Verify successful installation:

```bash
ls -l bld/lib
```
