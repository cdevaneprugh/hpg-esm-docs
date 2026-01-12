# Porting CESM

## Introduction<a name="introduction"></a>

CESM has [two primary releases](https://www.cesm.ucar.edu/models), the current development release (v2.2.2 at the time of this writing), and the production release (v2.1.5). We will be using the production release. I am following the [CESM documentation](https://escomp.github.io/CESM/versions/cesm2.1/html/index.html), as well as the [CIME porting documentation](https://esmci.github.io/cime/versions/master/html/users_guide/porting-cime.html) while adding the steps needed to get this working on HiPerGator.

## Downloading the Code<a name="downloading-the-code"></a>

While the repository is not that large, it should be put on the `/blue` drive for fast access and not in your `home` directory. It is completely up to the user and group how the file structure is organized. We decided to have a shared directory called `/earth_models` that contains the source code for the different models, as well as the input data directory (which can get extremely large). We have planned to move the input data directory to `orange` at some point in the future. Remember to make sure that all users in the group have read/write privileges to this directory.

Follow directions in the [CESM documentation](https://escomp.github.io/CESM/versions/cesm2.1/html/downloading_cesm.html) to clone the repository and checkout the external components.

```bash
# cd into the directory you want cesm installed
cd /blue/$GROUP/earth_models

# clone the cesm repository
git clone -b release-cesm2.1.5 https://github.com/ESCOMP/CESM.git cesm2.1.5

# cd into your cesm directory
cd cesm2.1.5

# download the external components
./manage_externals/checkout_externals

# check that the components were installed correctly
./manage_externals/checkout_externals -S
```

The last command should output something like:

```bash
Processing externals description file : Externals.cfg
Processing externals description file : Externals_CLM.cfg
Processing externals description file : Externals_POP.cfg
Processing externals description file : Externals_CISM.cfg
Checking status of externals: clm, fates, ptclm, mosart, ww3, cime, cice, pop, cvmix, marbl, cism, source_cism, rtm, cam,
    ./cime
    ./components/cam
    ./components/cice
    ./components/cism
    ./components/cism/source_cism
    ./components/clm
    ./components/clm/src/fates
    ./components/clm/tools/PTCLM
    ./components/mosart
    ./components/pop
    ./components/pop/externals/CVMix
    ./components/pop/externals/MARBL
    ./components/rtm
    ./components/ww3
```

If there were issues with any of the components, you would see an error:

```bash
e-  ./components/clm
```

By default, CESM downloads the most recent version of the CIME code, which unfortunately causes some bugs. We can fix that by checking out an older, more stable version.

```bash
cd cime
git pull origin maint-5.6
git checkout maint-5.6
```

If you run `./checkout_externals -S` after changing the CIME branch, it may show an error that CIME is not using the correct version. You can ignore this.

### Regression Testing<a name="regression-testing"></a>

Once the config files are setup, we need to run regression tests to ensure things are working correctly. The regression tests script will test various parameters of the Earth model in isolation, then send a dozen or two small cases to the scheduler to be run. This script is not resource intensive and can be run from a login node.

You'll need python to run the script. 

```bash
# load python
module load python/3.12

# go to the location of the regression tests
cd /blue/GROUP/earth_models/CESM/cime/scripts/tests
```

You can run the script with several options. Use the `--help` tag when running the script for the full list of them. For example, if you want to run the script and test the intel compilers, using a specific output directory, you could do something like:

```
./scripts_regression_tests.py --compiler gcc --test-root PATH/TO/TEST/OUTPUT
```

### Ensemble Consistency Testing<a name="ensemble-consistency-testing"></a>

Follow the guide [here](https://www.cesm.ucar.edu/models/cesm2/python-tools) in order to complete these tests for scientific validation. You'll just need to change the case output location, machine, and compiler name to reflect your setup. I should note that to run the script to create these tests we have to load an older version of python.

```bash
module load python-core/2.7.14
```

