# CESM Usage

1. [Basics](basic-usage-of-cesm)

   1.1 [Recommended Reading](recommended-reading)

   1.2 [File Structure](cesm-file-structure-on-hpg)

   1.3 [Creating, Building, and Running a Case](cesm_case)

   1.4 [Example](clm_example)

2. [Single Point Cases](pts_mode)

   2.1 [Best Practices](clm_best_practices)

   2.2 [Compset Testing](clm_compset_testing)

3. [Single Point With Spin Up](pts_slides)



## 1. Basics<a name="basic-usage-of-cesm"></a>

### 1.1 Recommended Reading<a name="recommended-reading"></a>

There are three pieces of documentation that I strongly suggest you read through to familiarize yourself with the process of creating cases on CESM.

1. The [quick start](https://escomp.github.io/CESM/versions/cesm2.1/html/quickstart.html) section of CESM's documentation.
2. The [Using the Case Control System](https://esmci.github.io/cime/versions/master/html/users_guide/index.html) section of the CIME documentation.
3. The [clm documentation](https://escomp.github.io/ctsm-docs/versions/release-clm5.0/html/users_guide/index.html), as the SWES department primarily uses `clm`, the land model component of CESM.

### 1.2 CESM File Structure on HPG<a name="cesm-file-structure-on-hpg"></a>

For the "gerber" group on HiPerGator, CESM has been installed in `/blue/gerber/earth_models/cesm2.1.5`. The scripts to create a new case or query information about case options are located in `/blue/gerber/earth_models/cesm2.1.5/cime/scripts`.

If you leave the `machine_config.xml` at its default settings, and create a "cases" directory at `/blue/GROUP/USER/cases`, the directory tree at `/blue/GROUP/USER` should look something like:

```bash
.
├── cases
│   └── EXAMPLE_CASE
└── earth_model_output
    ├── cesm_baselines
    ├── cime_output_root
    │   ├── archive
    │   └── EXAMPLE_CASE
    │       ├── bld
    │       └── run
    └── timings
```



### 1.3 Creating, Building, and Running a Case on HPG<a name="cesm_case"></a>

```bash
# cd to the cime scripts directory
cd /blue/gerber/earth_models/cime/scripts

# create your case, specifying the case location, compset, and resolution
./create_newcase --case /blue/GROUP/USER/cases/EXAMPLE_CASE --compset COMPSET --res RESOLUTION
```

CIME will output a bunch of text, then say whether the the case was created successfully, and where it was created.
If you are running a compset that is not scientifically validated, you will have to add the `--run-unsupported` option to your `create_newcase` command.

```bash
# go to the case directory
cd /blue/GROUP/USER/cases/case1
```

For the "Gerber" group, there are a few variables that we will most likely need to change. The first is the number of cores used by the model. Most compsets will request one (or several) nodes (128-512 cores) by default.
Our research group only has access to 20 cores on our default queue, so we need to make sure we're under the QOS limit. We can do this with the `xmlchange` script.

First check how many cores each component is requesting by running `./pelayout`, which will list out the individual components, along with what resources they are asking for.
The variable we want to pay atention to is `NTASKS`. This corresponds to how many cores will be requested when the case is submitted.

```bash
# change the number of cores to something more sensible
./xmlchange NTASKS=8
```

This will take us down to using only 8 cores when we run the case.

The downside of using fewer cores, is that we may have to run the case for longer on the compute node. We can extend the time requested by changeing the `JOB_WALLCLOCK_TIME` variable.

```bash
# check the current setting
./xmlquery JOB_WALLCLOCK_TIME

# change the variable if needed
./xmlchange JOB_WALLCLOCK_TIME=1:00:00
```

If you're unsure of the exact name of the varibale you want to check/change, you can list all the defined variables, then pipe to grep and search.
```bash
# running grep with the -i flag will ignore case distinction
./xmlquery --listall | grep -i SEARCH_TERM
```
Once you've set all of your variables, setup, build, and submit the case as usual.
```bash
./case.setup
./case.build
./case.submit
```

If you setup and build the case but realize you need to change some variables before submitting, it's a good idea to clean the case before rebuilding. We can do this with something like:

```bash
./xmlchange SOME_VARIABLE

./case.setup --clean
./case.build --clean

./case.setup
./case.build
./case.submit
```

### 1.4 Example Case<a name="clm_example"></a>
Here is an example of creating, building, and running a case with a compset typical of what we would use in the SWES department.
The long name for our compset is `1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV` and the resolution we will be using is `f19_g17`. 
While we can certainly use the long name for the compset, sometimes it's nicer to use the alias (assuming one is available). Here's a trick for finding your compset's alias.

```bash
# cd to the cesm, cime scripts
cd /blue/gerber/earth_models/cesm215/cime/scripts

# use the query config script, then pipe it to grep and search the compset's long name
./query_config --compsets all | grep 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV
```

Which should output the following.
```bash
I1850Clm50SpCru      : 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV
```

Now we can create the case.

```bash
# cd to the cesm, cime scripts
cd /blue/gerber/earth_models/cesm215/cime/scripts

# create the case using a sensible name, the compset alias, and desired resolution
./create_newcase --case /blue/GROUP/USER/cases/EXAMPLE_CASE --compset I1850Clm50SpCru --res f19_g17

# cd to the case directory
cd /blue/GROUP/USER/cases/EXAMPLE_CASE

# check the amount of cores being requested by default
./xmlquery NTASKS

# it's probably going to be higher than our QOS, so we need to change it along with extending the job time
./xmlchange NTASKS=8,JOB_WALLCLOCK_TIME=1:00:00

# setup the case
./case.setup

# check the submit script to make sure it's requesting the correct amount of resources
./preview_run

# if everything looks good, build the case (this can take a few minutes)
./case.build

# submit the case to the scheduler and run the experiment
./case.submit
```

Check your UF email for updates from the `SLURM` scheduler. A case can fail for many reasons, most of which should be pretty obvious.
If you accidentally requested more resources than your QOS allows, it will tell you in the email. If your case fails with an OOM (out of memory) error, try increasing the number of cores by changing the `NTASKS` variable.
You may want to switch to your burst QOS sometimes. You can set this manually by changing the `JOB_QUEUE` variable (using the `xmlchange` script) to the name of your burst QOS. On hipergator your burst queue is your group name with "-b" appended. So the burst queue for the "gerber" group is gerber-b.

## 2. Single Point Cases in CESM<a name="pts_mode"></a>

Following the instructions [here](https://escomp.github.io/ctsm-docs/versions/release-clm5.0/html/users_guide/running-single-points/running-pts_mode-configurations.html), we can run the clm model on a single grid cell by specifying a latitude and longitude. However, the instructions on the clm website seem to be a bit outdated. CIME no longer supports the `-pts_lat` or `-pts_lon`  arguments with the `create_newcase` script, also multi-character arguments should begin with `--` rather than `-`.  We can still run on a single point by creating a new case, then changing the appropriate variables before building the executable.

__A Note on DATM_MODE__ [source](https://www2.cgd.ucar.edu/events/2019/ctsm/files/practical4-wieder.pdf#page=16)

There are five modes used with CLM that specify the type of Meteorological data that’s used.
1. CLMGSWP3 (this is the preferred meteorological data to use w/ CLM5)
2. CLMCRUNCEP (Use global NCEP forcing at half-degree resolution from CRU goes from 1900-2010. GSWP3 similar time period and spatial resolution).
3. CLM_QIAN (Deprecated. Use NCEP forcing at T62 resolution corrected by Qian et. al. goes from 1948-2004).
4. CLM1PT (Use the local meteorology from your specific tower site).
5. CPLHIST (This name may have changed. Use atmospheric data from a previous CESM simulation).

```bash
# cd to cime/scripts 
cd /blue/gerber/earth_models/cime/scripts

# find the shortname for the compset
./query_config --compsets all | grep 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV
```

Which outputs the following to the terminal.

```
I1850Clm50SpCru      : 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV
```

To create our case we can do something like:

```bash
./create_newcase --case /blue/gerber/cdevaneprugh/cases/testPTS_OSBS --res f19_g17 --compset I1850Clm50SpCru

# cd to the case directory
cd /blue/gerber/cdevaneprugh/cases/testPTS_OSBS

# change variables to run on a single point
./xmlchange PTS_MODE=TRUE,PTS_LAT=29.7,PTS_LON=-82.0
./xmlchange CLM_FORCE_COLDSTART=on,RUN_TYPE=startup

# change variables to use a single core and adjust wall time
./xmlchange NTASKS=1
./xmlchange JOB_WALLCLOCK_TIME=1:00:00

# setup, build, and submit the case as usual
./case.setup
./case.build
./case.submit
```

This case will build and download input data, but fail during runtime with the following mpi error.  

```bash
MPI_ABORT was invoked on rank 0 in communicator MPI_COMM_WORLD with errorcode 2.

NOTE: invoking MPI_ABORT causes Open MPI to kill all MPI processes.
You may or may not see output from other processes, depending on exactly when Open MPI kills them.
```

Following advice on [my forum post](https://bb.cgd.ucar.edu/cesm/threads/issues-downloading-input-data-in-clm5-single-point-mode.9600/#post-55331), I tried a different compset which worked perfectly.

```bash
# create case
./create_newcase --case /blue/gerber/cdevaneprugh/cases/osbsPTSmod --compset 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_SROF_SGLC_SWAV --res f19_g17 --run-unsupported
cd /blue/gerber/cdevaneprugh/cases/osbsPTSmod

# change variables
./xmlchange NTASKS=1,JOB_WALLCLOCK_TIME=1:00:00
./xmlchange PTS_MODE=TRUE,PTS_LAT=29.7,PTS_LON=-82.0
./xmlchange CLM_FORCE_COLDSTART=on,RUN_TYPE=startup

# setup and build case like normal
./case.setup
./case.build

# check input data
./check_input_data

# submit case
./case.submit
```
The next section will go into where we went wrong, and some things we can do to have successful runs in the future.

### 2.1 CLM Best Practices and Where Our PTS Run Went Wrong<a name="clm_best_practices"></a>

The full comment from my [post](https://bb.cgd.ucar.edu/cesm/threads/issues-downloading-input-data-in-clm5-single-point-mode.9600/#post-55331) is:

> The error message in the cesm log is:
>
> MCT::m_SparseMatrixPlus:: FATAL--length of vector y different from row count of sMat.Length of y = 1 Number of rows in sMat = 13824
> 000.MCT(MPEU)::die.: from MCT::m_SparseMatrixPlus::initDistributed_()
>
> It seems like there must be mismatch between two of the datasets you are using. One seems to be single point (y=1) and the other is 1.9x2.5 (144x96=13824). I think it might be because the compset you are using has CISM and MOSART in it. The long name for that compset is:
>
> 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV
>
> Try specifying stubs for those, e.g.,
>
> 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_SROF_SGLC_SWAV

So the hypothesis is that any compset with MOSART or CISM will cause an issue in single point mode. Okay, so the easiest option is to just pick a compset we like, then specify stubs in the same way that was suggested in the post. The only problem is that doing this turns our compset into one that is not scientifically validated. This might not be an issue for our purposes. I also noticed an interesting thing in the [clm documentation's best practices section](https://escomp.github.io/ctsm-docs/versions/release-clm5.0/html/users_guide/overview/introduction.html#best-practices).

> CLM5.0 includes BOTH the old CLM4.0, CLM4.5 physics AND the new CLM5.0 physics and you can toggle between those three. The “standard” practice for CLM4.0 is to run with CN on, and with Qian atmospheric forcing. While the “standard” practice for CLM4.5 is to run with BGC on, and CRUNCEP atmospheric forcing. And finally the “standard” practice for CLM5.0 is to run with BGC and Prognostic Crop on, with the MOSART model for river routing, as well as the CISM ice sheet model, and using GSWP3 atmospheric forcing. “BGC” is the new CLM5.0 biogeochemistry and include CENTURY-like pools, vertical resolved carbon, as well as Nitrification and de-Nitrification

So if it is best practice to use MOSART as well as CISM for clm5.0, maybe we can just use the clm4.0 or 4.5 physics instead. For a test, I'm going to look at the initial compsets Stefan gave to me, then see if I can find the equivalent one that is running clm4.0 or 4.5 and test those.

Looking at the compsets, the one suggested in the forum post is effectively the clm4.0 version of what Stefan wanted, but with clm5.0 physics substituted (remember this doesn't follow best practices and is not scientifically validated). There are also plenty of scientifically valid compsets that don't use MOSART or CISM.

It would be interesting to see if I can narrow down which is messing us up. I'll find a compset with only CISM, see if that runs globally and in single point mode, then repeat with a compset containing MOSART.

With the abundance of compsets and resolutions, I'm going to need a better naming convention for my case directories. I think something like `compset.resolution.modifiers` would work. For example the name for our initial single point test (that failed) would be `I1850Clm50SpCru.f19_g17.PTS`.

### 2.2 Compset Testing<a name="clm_compset_testing"></a>

Anytime we get an error that is something like:

> MCT::m_SparseMatrixPlus:: FATAL--length of vector y different from row count of sMat.Length of y = 1 Number of rows in sMat = 55296

We can assume there is some issue with MOSART or CISM. The goal of running the following cases is to determine which is causing the error in single point mode.
I did this by choosing similar compsets that use physics from clm4.0 4.5 and 5.0 as well as running a global and PTS version of each compset.

__Compset Long Name	:	Compset Alias	:	Resolution Used__

**1850_DATM%CRUv7_CLM40%SP_SICE_SOCN_RTM_SGLC_SWAV	:	I1850Clm40SpCruGs	:	f19_g17**

1. Global: Success

2. PTS: Success


**1850_DATM%GSWP3v1_CLM45%CN_SICE_SOCN_RTM_CISM2%NOEVOLVE_SWAV	:	I1850Clm45Cn	:	f19_g17**

1. Global: Success

2. PTS: Fail 

   > MCT::m_SparseMatrixPlus:: FATAL--length of vector y different from row count of sMat.Length of y = 1 Number of rows in sMat = 13824
   >
   > 000.MCT(MPEU)::die.: from MCT::m_SparseMatrixPlus::initDistributed_()

**1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV	:	I1850Clm50SpCru	:	f19_g17**

1. Global: Success

2. PTS: Fail

   > MCT::m_SparseMatrixPlus:: FATAL--length of vector y different from row count of sMat.Length of y = 1 Number of rows in sMat = 13824
   >
   > 000.MCT(MPEU)::die.: from MCT::m_SparseMatrixPlus::initDistributed_()


**1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_SROF_SGLC_SWAV	:	NONE	:	f19_g17**

1. Global: Success
   
3. PTS: Success


**2000_DATM%GSWP3v1_CLM50%SP-VIC_SICE_SOCN_RTM_CISM2%NOEVOLVE_SWAV	:	I2000Clm50Vic	:	f09_g17**

1. Global:
   
3. PTS:


**HIST_DATM%GSWP3v1_CLM40%SP_SICE_SOCN_RTM_SGLC_SWAV	:	IHistClm40SpGswGs	:	f09_g17**

1. Global: Success
2. PTS: Success

**HIST_DATM%GSWP3v1_CLM45%SP_SICE_SOCN_RTM_SGLC_SWAV	:	IHistClm45SpGs	:	f09_g17**

1. Global: Fail (Out of Memory)

   > Primary job  terminated normally, but 1 process returned a non-zero exit code. Per user-direction, the job has been aborted.

2. PTS: Success

**HIST_DATM%QIA_CLM50%BGC_SICE_SOCN_MOSART_SGLC_SWAV	:	IHistClm50BgcQianGs	:	f09_g17**

1. Global: Fail (Out of Memory)
   
   > Primary job  terminated normally, but 1 process returned a non-zero exit code. Per user-direction, the job has been aborted.

3. PTS: Success

**HIST_DATM%GSWP3v1_CLM50%SP_SICE_SOCN_MOSART_CISM2%NOEVOLVE_SWAV	:	IHistClm50Sp	:	f09_g17**

1. Global: Fail (Out of Memory)

   > Primary job  terminated normally, but 1 process returned a non-zero exit code. Per user-direction, the job has been aborted.

2. PTS: Fail

   > MCT::m_SparseMatrixPlus:: FATAL--length of vector y different from row count of sMat.Length of y = 1 Number of rows in sMat = 55296
   > 
   > 000.MCT(MPEU)::die.: from MCT::m_SparseMatrixPlus::initDistributed_()

__Conclusion__
It looks like CISM is the issue. The Qian case (IHistClm50BgcQianGs) uses MOSART and the PTS mode case ran successfully.

## 3. Single Point With Spin Up ([Slides](https://www2.cgd.ucar.edu/events/2019/ctsm/files/practical4-wieder.pdf))<a name="pts_slides"></a>
I'll go through the exercises in order here, and note any issues I ran in to or modifications I had to make.

### Exercise 4a: Create a Global Case<a name='ex_4a"></a>
The compset I'll be using is `HIST_DATM%GSWP3v1_CLM50%SP_SICE_SOCN_MOSART_SGLC_SWAV` which is a modified version of the supported compset `IHistClm50Sp` that removes the CISM portion of the model.

```bash
# cd to cime scripts
cd /blue/gerber/earth_models/cesm215/cime/scripts

# create our case
./create_newcase --case /blue/gerber/cdevaneprugh/cases/IHistClm50Sp_001 --compset HIST_DATM%GSWP3v1_CLM50%SP_SICE_SOCN_MOSART_SGLC_SWAV --res f09_g17 --run-unsupported

# cd to case
cd /blue/gerber/cdevaneprugh/cases/IHistClm50Sp_001

./case.setup
./preview_namelists

# Look for the path to the surface data set and domain file
cat CaseDocs/lnd_in
```

### Exercise 4b: Generate Domain and Surface Data Sets<a name="ex_4b"></a>
It looks like there's a script called `singlept` that we need access to. I think this was provided during the workshop on the NCAR server being used. Additionally, there is a `ncar_pylib` I need access to, it's possible this is available to download with `conda`.
There was a list of many options that are changed in the `singlept` script but I can't find the variables they are setting when using `xmlquery` and looking in the `CaseDocs` directory.

### Exercise 4c: Create a New Case for the Single Point Run<a name="ex_4c"></a>

```bash
# cd to cime scripts
cd /blue/gerber/earth_models/cesm215/cime/scripts

# create a new compset with a stub ice and river model
./create_newcase --case /blue/gerber/cdevaneprugh/cases/ex_4c --compset 1850_DATM%CRUv7_CLM50%SP_SICE_SOCN_SROF_SGLC_SWAV --res f09_g17 --run-unsupported

# go to case directory
cd /blue/gerber/cdevaneprugh/cases/ex_4c
```

Following the guide there are many variables to change. Rather than list them all out, use the script `ready_clm_case` provided in this repository.
Note: You should not change the `MPILIB` variable to `mpi-serial`. On our system, things seem to break if you do this. I am still unsure why.
On [page 30](https://www2.cgd.ucar.edu/events/2019/ctsm/files/practical4-wieder.pdf#page=30) of the slides they need the new domain files that we would have made in exercise 4b. We'll have to skip this part for now.

It _looks_ like all the variables were changed successfully, and the model was built. Unfortunately without those scripts from the workshop, we're sort of stuck. 

### Exercise 4d: Single Point BGC_AD<a name="ex_4d"></a>
The xml variable changes are almost identical to exercise 4c, with one or two exceptions. Unfortunately, we run into the same problem here in that we need the scripts from exercise 4b.
