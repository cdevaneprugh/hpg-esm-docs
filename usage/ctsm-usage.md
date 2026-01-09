## Basic Usage of CTSM

See the section in `CESM`. The scripts for general use are located in `$CTSMROOT/cime/scripts`.

# Documentation Issues

After reading the documentation and the following two comments from CESM forum admins, it looks like __everything__ in the documentation's single point case creation is either outdated or broken.

Admin Comments:
1. "Support for PTS_MODE in cime was dropped at some point so you are likely using a version of the model in which support was dropped."
   * It is still fine to use on our `CESM` install.

2. "If you want to subset existing global datasets to regional or single point please see the README in `tools/site_and_regional`. If you want to create your own regional datasets from scratch please see the `README` in `tools/mksurfdata_esmf`."

# Single Point Subset
module load conda

conda activate ctsm_pylib

cd $CTSM/tools/site_and_regional

./subset_data point --lat $my_lat --lon $my_lon --site $my_site_name --create-surface --create-datm \
--datm-syr $my_start_year --datm-eyr $my_end_year --create-user-mods --outdir $my_output_dir

$my_lat : latitude of point, should be between -90 and 90 degrees
$my_lon : longitude of point, should be between 0 and 360 or -180 and 180 degrees
$my_site_name : name of site, used for file naming
$my_start_year : start year for DATM data to subset, default between 1901 and 2014
$my_end_year : end year for DATM data to subset, default between 1901 and 2014
$my_output_dir : output directory to place the subset data and user_mods directory

```
# cd to subset data tool and run script
cd $CTSM/tools/site_and_regional

./subset_data point --lat 42.53562 --lon 287.82438 --site demo_pt --create-surface --create-datm --datm-syr 1901 --datm-eyr 1902 --create-user-mods --outdir /blue/gerber/cdevaneprugh/my_subset_data/demo_pt --overwrite

# cd to cime scripts and create new case from dataset
cd $CTSMROOT/cime/scripts

./create_newcase --case $CASES/singlept.demo --res CLM_USRDAT --compset I1850Clm50Bgc --run-unsupported --user-mods-dirs /blue/gerber/cdevaneprugh/my_subset_data/demo_pt/user_mods/

# cd into case directory, change variables, and run case
cd /blue/gerber/cdevaneprugh/cases/singlept.demo

./case.setup

./xmlchange MPILIB=openmpi
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=1
./xmlchange RUN_STARTDATE='1901-01-01'
./xmlchange DATM_YR_ALIGN=1901
./xmlchange DATM_YR_START=1901
./xmlchange DATM_YR_END=1902

./case.build
./case.submit
```
