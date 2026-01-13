# CIME Configuration

CTSM uses CIME (Common Infrastructure for Modeling the Earth) for building and running cases. This page explains the HiPerGator-specific configuration and what you may need to customize.

## How CIME Finds Machine Config

CIME looks for machine configurations in this order:

1. `~/.cime/` (user overrides)
2. `$CTSMROOT/ccs_config/machines/<machine>/` (machine-specific)
3. `$CTSMROOT/ccs_config/machines/` (defaults)

Our fork includes HiPerGator configs at:
```
ccs_config/machines/hipergator/
├── config_machines.xml    # Machine definition
├── config_batch.xml       # SLURM settings
└── gnu_hipergator.cmake   # Compiler flags
```

## Configuration Files

### config_machines.xml

The main machine definition. Key sections:

#### Basic System Info

```xml
<DESC>UF HiPerGator | hpg-default node | AMD ROME 128 cores per node | slurm</DESC>
<OS>LINUX</OS>
<COMPILERS>gnu</COMPILERS>
<MPILIBS>openmpi</MPILIBS>
```

#### File Paths

These define where CTSM reads/writes data:

```xml
<CIME_OUTPUT_ROOT>/blue/gerber/$ENV{USER}/earth_model_output/cime_output_root</CIME_OUTPUT_ROOT>
<DIN_LOC_ROOT>/blue/gerber/earth_models/inputdata</DIN_LOC_ROOT>
<DOUT_S_ROOT>$CIME_OUTPUT_ROOT/archive/$CASE</DOUT_S_ROOT>
```

| Variable | Purpose |
|----------|---------|
| `CIME_OUTPUT_ROOT` | Where case build/run directories are created |
| `DIN_LOC_ROOT` | Where CTSM looks for (and downloads) input data |
| `DOUT_S_ROOT` | Where output is archived after runs complete |

!!! warning "You Must Customize These"
    Change `/blue/gerber/` to your group's allocation path.

#### Node Configuration

```xml
<MAX_TASKS_PER_NODE>128</MAX_TASKS_PER_NODE>
<MAX_MPITASKS_PER_NODE>128</MAX_MPITASKS_PER_NODE>
```

HiPerGator's default nodes have 128 cores. This tells CIME how to distribute MPI tasks.

#### Module System

The config tells CIME which modules to load:

```xml
<modules>
  <command name="load">cmake/3.26.4</command>
  <command name="load">gcc/14.2.0</command>
  <command name="load">openmpi/5.0.7</command>
  <!-- ... more modules ... -->
</modules>
```

These must match your `ctsm-modules` collection.

#### Environment Variables

```xml
<environment_variables>
  <env name="NETCDF_PATH">$ENV{HPC_NETCDF_C_DIR}</env>
  <env name="PIO">/blue/gerber/earth_models/shared/parallelio/bld</env>
  <!-- ... more variables ... -->
</environment_variables>
```

!!! warning "PIO Path"
    The PIO paths point to the shared ParallelIO build. Update these to your group's build location.

#### PIO Shared Library {#pio-shared-library}

Our fork includes configuration to use a pre-built PIO library instead of rebuilding it for every case. This is controlled by three environment variables:

```xml
<environment_variables>
  <!-- Basic PIO paths -->
  <env name="PIO">/blue/<group>/earth_models/shared/parallelio/bld</env>
  <env name="PIO_LIBDIR">/blue/<group>/earth_models/shared/parallelio/bld/lib</env>
  <env name="PIO_INCDIR">/blue/<group>/earth_models/shared/parallelio/bld/include</env>

  <!-- These tell case.build to use external PIO -->
  <env name="PIO_VERSION_MAJOR">2</env>
  <env name="PIO_TYPENAME_VALID_VALUES">netcdf</env>
  <env name="LD_LIBRARY_PATH">/blue/<group>/earth_models/shared/parallelio/bld/lib:$ENV{LD_LIBRARY_PATH}</env>
</environment_variables>
```

| Variable | Purpose |
|----------|---------|
| `PIO_VERSION_MAJOR` | Tells CIME to use external PIO (must match your PIO version) |
| `PIO_TYPENAME_VALID_VALUES` | Specifies supported I/O backends (`netcdf` for our build) |
| `LD_LIBRARY_PATH` | Required for runtime dynamic linking to `.so` files |

**How it works:** During `case.build`, CIME checks if `PIO_VERSION_MAJOR` is set. If it matches the expected version, CIME uses your external PIO library instead of building one. You'll see "Using installed PIO library" in the build log.

!!! tip "Build Time Savings"
    Without these settings, every `case.build` compiles PIO from source, adding several minutes to each build. With external PIO, builds are noticeably faster.

### config_batch.xml

SLURM job submission settings:

```xml
<batch_system MACH="hipergator" type="slurm">
  <batch_submit>sbatch</batch_submit>
  <submit_args>
    <arg flag="--time" name="$JOB_WALLCLOCK_TIME"/>
    <arg flag="-q" name="$JOB_QUEUE"/>
    <arg flag="--mem" name="16GB"/>
  </submit_args>
  <directives>
    <directive>--partition=hpg-default</directive>
    <directive>--mail-type=NONE</directive>
  </directives>
  <queues>
    <queue jobmin="1" jobmax="20" default="true">gerber</queue>
    <queue jobmin="1" jobmax="128">gerber-b</queue>
  </queues>
</batch_system>
```

#### Key Settings

**No `--exclusive` flag**: Unlike many HPC centers, HiPerGator shares nodes by default. We don't include `--exclusive` because:
- Most CTSM runs use few cores
- Exclusive access wastes resources
- QoS limits often don't allow full-node jobs anyway

**Memory request**: Fixed at 16GB. Adjust if your runs need more.

**Queues section**:

!!! danger "You Must Change This"
    The queue names (`gerber`, `gerber-b`) are QoS names specific to the Gerber group. Replace these with your group's QoS names.

Check your group's QoS:
```bash
showQos -A <your_group>
```

The `jobmin` and `jobmax` values limit task counts for each queue.

### gnu_hipergator.cmake

Compiler flags and library linking:

```cmake
# NetCDF legacy macro support
string(APPEND CPPDEFS " -DNETCDF_ENABLE_LEGACY_MACROS")

# Link netcdf and lapack
string(APPEND SLIBS " ${SHELL_CMD_OUTPUT_BUILD_INTERNAL_IGNORE0}")
string(APPEND SLIBS " -L$(LAPACK_LIBDIR) -llapack -lblas")

# Lustre filesystem optimization
set(PIO_FILESYSTEM_HINTS "lustre")
```

You probably won't need to modify this file.

## Customizing for Your Group

### Required Changes

1. **File paths** in `config_machines.xml`:
   - `CIME_OUTPUT_ROOT`: Change `/blue/gerber/` to `/blue/<your_group>/`
   - `DIN_LOC_ROOT`: Point to your group's inputdata location
   - `PIO` paths: Point to your group's ParallelIO build

2. **Queue names** in `config_batch.xml`:
   - Replace `gerber` and `gerber-b` with your group's QoS names

### Optional Changes

- **Memory**: Increase `--mem` in `config_batch.xml` if needed
- **Email**: Change mail settings in `config_batch.xml`
- **Partition**: Change from `hpg-default` if using GPU nodes

## Making Changes

After modifying config files:

```bash
# If you already have cases built, they may need rebuilding
cd /path/to/your/case
./case.build --clean-all
./case.build
```

New cases will automatically use the updated configs.

## The ~/.cime Alternative

CIME supports user-level config overrides in `~/.cime/`. You could theoretically:

1. Clone upstream CTSM (no fork needed)
2. Put HiPerGator configs in `~/.cime/`

**Why we don't recommend this:**

- Batch config loading has bugs in CIME v3
- `NODENAME_REGEX` machine detection is unreliable
- Each user must maintain their own configs
- Harder to share and troubleshoot

The fork approach gives everyone consistent, working configs.

## Troubleshooting

**"Machine not found"**
: Check that `ccs_config/machines/hipergator/` exists and submodules are initialized

**Job submission fails**
: Verify queue names match your group's QoS

**Build can't find libraries**
: Check that module versions in config match your `ctsm-modules` collection

**Wrong output location**
: Check `CIME_OUTPUT_ROOT` path in `config_machines.xml`
