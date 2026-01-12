# Porting CTSM<a name="ctsm_port"></a>

Porting `CTSM` is largely the same process as `CESM`. You need to clone the repository, and download the external components. There are just a couple small differences in how this is achieved due to a different version of `CIME` being used. Additionally, we have a forked the `CTSM` repository, so the configuration files and necessary code fixes have already been handled.

## Downloading the Code

```bash
# go the install directory. The Gerber group uses earth_models
cd /blue/$GROUP/earth_models

# clone the forked repo and cd into it
git clone https://github.com/cdevaneprugh/CTSM.git ctsm
cd ctsm

# checkout the desired release. for us it is the uf-ctsm branch
git checkout uf-ctsm5.3
```

Similar to `CESM` we need to download the externals. Instead of `checkout_externals` we use a script called `git-fleximod`.

```bash
# in your ctsm root directory
./bin/git-fleximod update
```

The configuration files are already provided in the forked repository. If you checked out a different branch, you will need to add configuration files as explained in `shared-utils.md`.

## Regression Testing

If you'd like, you can run regression tests again. The `scripts_regression_tests.py` script is located in `$CTSMROOT/cime/CIME/tests`. The process is identical to as it was in `CESM`. I found these tests to randomly fail more often than the `CESM` ones. I'm not sure why this is, possibly because `CTSM` does not include all the components found in other Earth models.

## Building `mksurfdata`

The `mksurfdata` executable is used to generate fsurdat files for `CTSM`. If you want to use your own surface data, this tool must be built. It requires four libraries be properly configured before building. The first three (`MPI` `NetCDF` and `ESMF`) have already been configured on HiPerGator. The last library needed is `parallelio` (aka `PIO`). Instructions for building `parallelio` are found in `shared-utils.md`.

To build the executable:

```bash
# load the necessary modules
module restore esm_gcc_env

# navigate to the ctsm tools
cd $CTSMROOT/tools/mksurfdata_esmf

# run the script to build the executable
./gen_mksurfdata_build --machine hipergator
```
This should give the following output.

```
Successfully created mksurfdata_esmf executable for: hipergator_gnu for openmpi library
```

It is a good idea to check that the `parallelio` libraries were properly linked.
```bash
# go to the build directory
cd tool_bld

# examine linked libraries in the executable
ldd mksurfdata | grep libpio
```

You should see something like
```
libpiof.so => /parallelio/bld/lib/libpiof.so (0x00001500753d4000)
libpioc.so => /parallelio/bld/lib/libpioc.so (0x0000150075182000)
```

If it says "not found" then the libraries were not linked correctly and you need to retry building the executable.


__Note:__ If there is a build error and you need to rerun the script, be sure to delete the `tool_build` directory before attempting to rebuild.
## Setup ctsm_pylib

There is a `conda` environment that needs to be setup in order to use `mksurfdata`.

If you've never used `conda` environments, I recommend reading the [documentation on HiPerGator's website](https://help.rc.ufl.edu/doc/Conda). This has info on configuration, and building and managing environments. Similar to the `.bashrc` file there is a `.condarc` file located in your `home` directory. I would follow the HiPerGator recommendation to add the following lines to `.condarc` and change the environment and package install path to somewhere on your `/blue` storage to avoid crowding your home directory.

``` bash
envs_dirs:
- /blue/group/user/conda/envs
pkgs_dirs:
- /blue/group/user/conda/pkgs
```

Once you have `conda` set up to your liking, follow directions in the [CTSM tools](https://github.com/cdevaneprugh/CTSM/tree/master/tools/mksurfdata_esmf#running-for-a-single-submission) directory to install the `conda` environment.

```bash
# Assuming pwd is the tools/mksurfdata_esmf directory
 module load conda
 cd ../..  # or ../../../.. for a CESM checkout)
 ./py_env_create    # Assuming at the top level of the CTSM/CESM checkout
 conda activate ctsm_pylib
```

__NOTE: WE'RE STILL WORKING ON HOW TO USE THESE TOOLS AS WELL AS RUN SINGLE POINT CASES WITH THIS CTSM VERSION__
