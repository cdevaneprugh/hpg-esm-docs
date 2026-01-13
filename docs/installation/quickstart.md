# Quick Start

Get CTSM running on HiPerGator. This guide covers the minimum steps to clone, build, and run your first case.

## Before You Begin

You'll need:

- HiPerGator account with group allocation on `/blue/`
- Basic familiarity with the Linux command line
- ~50GB free space for CTSM installation (input data is separate)

New to HiPerGator? See [Onboarding](../onboarding.md) first.

## Step 1: Set Up Modules

CTSM requires specific compiler and library versions. Create a module collection:

```bash
module purge

# Load required modules
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

# Save the collection for future sessions
module save ctsm-modules
```

From now on, just run `module restore ctsm-modules` to load all modules.

## Step 2: Clone CTSM

We maintain a fork with HiPerGator-specific fixes. Clone it:

```bash
cd /blue/<group>/$USER

# Clone from our fork
git clone https://github.com/cdevaneprugh/CTSM.git ctsm5.3
cd ctsm5.3

# Checkout the HiPerGator branch
git checkout uf-ctsm5.3.085
```

Replace `<group>` with your research group's allocation name.

## Step 3: Initialize Submodules

CTSM uses `git-fleximod` to manage submodules (including the machine configuration):

```bash
./bin/git-fleximod update
```

This downloads all required components. It may take a few minutes.

## Step 4: Verify Installation

Check that everything is in place:

```bash
# Check submodule status
./bin/git-fleximod status

# Verify HiPerGator config exists
ls ccs_config/machines/hipergator/
```

You should see: `config_machines.xml`, `config_batch.xml`, `gnu_hipergator.cmake`

## Step 5: Customize for Your Group

The HiPerGator config includes QoS limits that need to match your group's allocation. Check your group's limits:

```bash
showQos -A <group>
```

Then edit the config file to match (see [CIME Configuration](cime-config.md) for details):

```bash
vim ccs_config/machines/hipergator/config_machines.xml
```

Look for the `MAX_TASKS_PER_NODE` and `MAX_MPITASKS_PER_NODE` settings.

## Step 6: Create a Test Case

Test your installation with a simple case:

```bash
cd cime/scripts

./create_newcase \
    --case /blue/<group>/$USER/cases/test_case \
    --compset I2000Clm60Sp \
    --res f45_f45_mg37 \
    --machine hipergator \
    --run-unsupported
```

## Step 7: Build and Run

```bash
cd /blue/<group>/$USER/cases/test_case

# Set up the case
./case.setup

# Build (this takes ~20 minutes the first time)
./case.build

# Submit to the queue
./case.submit
```

## Step 8: Check Results

Monitor your job:

```bash
squeue -u $USER
```

Check for output:

```bash
# Find your run directory
./xmlquery RUNDIR

# List output files
ls $(./xmlquery --value RUNDIR)/*.nc
```

## What's Next?

- **[Case Workflow](../running-ctsm/case-workflow.md)** - Detailed case configuration
- **[Single-Point Runs](../running-ctsm/single-point.md)** - Site-specific simulations
- **[Prerequisites](prerequisites.md)** - Build additional tools (mksurfdata, cprnc)
- **[CIME Configuration](cime-config.md)** - Understand and customize machine config
- **[Fork Reference](fork-setup.md)** - Why we forked and what's modified

## Troubleshooting

**Build fails immediately**
: Check that modules are loaded: `module list`

**Job stays pending**
: Check QoS limits match your config: `showQos -A <group>`

**Missing machine config**
: Verify submodules initialized: `./bin/git-fleximod status`

See [Troubleshooting](../running-ctsm/troubleshooting.md) for more help.
