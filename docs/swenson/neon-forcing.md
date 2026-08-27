# NEON Atmospheric Forcing

*Site-native NEON flux-tower meteorology for OSBS, generated offline on HiPerGator to replace CRUNCEPv7 in the CTSM atmospheric forcing stream.*

---

## Motivation

The operative OSBS case (`osbs.swenson.spinup`) is driven by **CRUNCEPv7** — a 0.5° global reanalysis interpolated to the OSBS point. It does not reflect site meteorology and discards the on-site NEON flux-tower record, including the tower's *measured* incident longwave (which CRUNCEP supplies by parameterization). Because the terrestrial-aquatic-interface hydrology this project targets is sensitive to the local water and energy balance, site-native forcing is a meaningful input-quality upgrade — independent of any hillslope-parameter change.

Two NEON-derived forcing products exist for OSBS:

| Product | Coverage | Provenance |
|---|---|---|
| **Pre-built NCAR-NEON v4** | 2018-01 → 2024-12 (84 files) | Generated on NEON's cloud by the standard NCAR-NEON pipeline; adopted here as the **reference** the custom set is validated against |
| **Custom full-record set** | 2017-02 → 2025-06 (101 files) | Generated **offline on HiPerGator** by a fork of the NCAR-NEON pipeline (this page); a strict **superset of v4** |

!!! note "Adoption status"
    The custom forcing is **engineering-complete and validated**, and is *offered* for the production respin. That adoption is a PI-gated decision; until it is made, the operative case remains CRUNCEP-driven. This page documents how the forcing was produced, not a change to the running configuration.

---

## Source data

Forcing is built from the OSBS NEON tower (D03) instrument products, **RELEASE-2026** (the released cut ends 2025-06). CTSM's DATM requires seven meteorological drivers; the NCAR-NEON pipeline draws each from a specific NEON data product, plus two radiation products used only as gap-fill/partitioning inputs:

| DATM variable | NEON product | OSBS start |
|---|---|---|
| Air temperature (TBOT) | DP1.00003.001 — Triple Aspirated Air Temperature (tower top) | 2014-08 |
| Precipitation (PRECTmms) | DP1.00044.001 — Weighing Gauge (primary) | 2016-09 |
| Precipitation, secondary | DP1.00045.001 — Tipping Bucket | 2016-08 |
| Shortwave + longwave (FSDS, FLDS) | DP1.00023.001 — Net Radiometer | 2014-08 |
| Pressure (PSRF) | DP1.00004.001 — Barometric Pressure | 2014-08 |
| Wind (WIND) | DP1.00001.001 — 2D Wind Speed & Direction | 2014-08 |
| Relative humidity (RH) | DP1.00098.001 — Relative Humidity | 2015-06 |
| *(gap-fill inputs)* | DP1.00024.001 (PAR), DP1.00014.001 (direct/diffuse SW) | — |

Carbon dioxide is **not** a forcing-file variable — CTSM supplies CO₂ through a separate mechanism (`CCSM_CO2_PPMV` or a `co2tseries` stream). NEON's bundled eddy-covariance product (DP4.00200.001, from 2017-02) is a validation target, not an input; its role here is discussed under [the CO₂-partition edit](#the-offline-conversion-edits) and [Validation](#validation).

The underlying pipeline is NCAR-NEON (Wieder et al., 2023, *Geosci. Model Dev.* 16, 5979–6000; DOI [10.5194/gmd-16-5979-2023](https://doi.org/10.5194/gmd-16-5979-2023)), which pulls raw tower data through the NEON API, gap-fills by marginal distribution sampling (Reichstein et al., 2005) as implemented in **REddyProc** (Wutzler et al., 2018, *Biogeosciences*), and writes CTSM-DATM NetCDF.

---

## Forcing generator: a fork of NCAR-NEON

The forcing is produced by NCAR-NEON's `flow.api.clm.R`, **modified to run fully offline on HiPerGator against a pre-staged raw archive**. The modifications live in a fork rather than a vendored copy for a licensing reason: NCAR-NEON (and its `NEON.gf` gap-filling package) are licensed **AGPL-3.0**, while `hpg-esm-tools` is MIT. Copyleft is one-directional, so the modified generator is kept in a separate AGPL fork and referenced, not copied into the MIT repository.

- **Fork:** [`cdevaneprugh/NCAR-NEON`](https://github.com/cdevaneprugh/NCAR-NEON), branch `uf-osbs`
- **Base:** two commits ahead of upstream `NEONScience/NCAR-NEON` at merge-base `43a0cf4` (`2d30ebb` mechanical offline plumbing; `07786dd` offline data path, tested)
- **Change footprint:** `TowerTools_ForcingData/flow.api.clm.R`, +77 / −31 lines

Upstream, the script assumes an interactive, live-network run on the author's workstation that uploads its output to a Google Cloud Storage bucket. Four properties had to change for an unattended HiPerGator run: it had to (a) read from a **read-only shared archive** of pre-downloaded products, (b) write **locally**, (c) generate **sub-year windows** so a multi-year record could be built in chunks, and (d) survive OSBS's **missing eddy-covariance CO₂ flux**.

### The offline-conversion edits

Every modification, as a reproducibility record:

| Edit | Change | Rationale |
|---|---|---|
| Fail-fast deps | `packReq` auto-install loop → `requireNamespace()`-or-`stop()` | The `neon-forcing` environment is complete; never trigger a live-CRAN install mid-run |
| Local output (**M**) | `MethOut` default `[2]` (GCS) → `[1]` (local) | Upstream uploads to a Google Cloud Storage bucket needing a credential we lack; write to disk instead |
| Offline switch | added `doDnld <- FALSE` | Master gate — every network fetch and cleanup below is guarded by this flag |
| Archive path (**D**) | `DirDnld` `/home/ddurden/…` → `/blue/gerber/earth_models/neon/raw` | Repoint from the author's home directory to the shared OSBS raw archive |
| Working-copy isolation (**C**) | copy staged zips to a per-session `tempdir()`, repoint `DirDnld` there | `stackByTable`/`stackEddy` mutate — and delete — their input zips; the read-only shared archive must never be stacked in place |
| Guarded EC download (**G**) | wrap the EC-bundle `zipsByProduct` in `if(doDnld)` | Bundle is pre-staged; skip the network fetch offline |
| Window clip (**T**) | clip the regularized flux frame to `DATEBGN`/`DATEEND` (REddyProc period-end convention) | Lets a sub-year window run against a full archive → chunked (annual) generation. REddyProc requires ≥ 90 days of flux, so windows are annual, not monthly |
| Guarded wipes (**W**) | wrap both end-of-run `file.remove` calls in `if(doDnld)` | Offline, `DirDnld` *is* the shared archive — an unguarded wipe would delete it |
| Secondary-precip product | `PRECTmms_MDS`: DP1.00006.001 → DP1.00045.001 | The stock secondary-precip product has zero months at OSBS; OSBS's secondary gauge is the tipping bucket DP1.00045.001 |
| Offline met stacking (**S**) | met `loadByProduct` (online) → `stackByTable` of pre-staged zips, fresh `tempdir()` per call | The core online→offline swap; per-call isolation because `FLDS_MDS` and `Rg` both map to DP1.00023.001 and stacking deletes its inputs |
| Drop live metadata call | `getProductInfo("DP1.00044.001")$siteCodes` → `sitePrecip <- "OSBS"` + offline primary-precip stack | Removes the last live-network call; OSBS is verified to carry primary precip |
| CO₂-partition resilience (**P2**) | `sMRFluxPartition`/`sGLFluxPartition` → non-fatal `try()`; NaN-fill missing CO₂-derived evaluation columns | OSBS eddy-covariance CO₂ flux (NEE/FC) is **absent at the NEON source** (raw flux all-NA, its own QC flag all-bad), so flux partitioning cannot run. The atmospheric forcing uses no CO₂ variable; this only lets the co-written evaluation file finalize |

!!! warning "The wipe guard (edit W) is load-bearing"
    Upstream, `DirDnld` is a scratch download directory that the script deletes on exit. Offline, `DirDnld` is repointed at the shared raw archive (edit D). Without edit W, a normal run would erase `/blue/gerber/earth_models/neon/raw`. Edit C (per-session working copies) is the primary safeguard; edit W is the backstop.

The **absent OSBS CO₂ flux** (edit P2) is a genuine NEON data gap, not an artifact of the offline conversion — the energy fluxes are intact, and the pre-built v4 product, which exists for OSBS, can only have been generated the same way. The consequence is that **no CO₂-flux validation is possible for OSBS**; it does not affect the meteorological forcing.

---

## Computational environment

NCAR-NEON is interpreted R with a large, version-sensitive dependency tree (originally R 4.0.5, a 195-package `renv.lock` including Bioconductor `rhdf5` and GitHub-only `eddy4R`). Rather than restore that lockfile, the stack was **reconstructed as a conda environment** (`neon-forcing`) with a conda base and a thin source-built layer for the packages on no conda channel:

- **R 4.2** (pinned, not 4.4): the archived `ffbase` — a hard `eddy4R.base` dependency — will not build against modern R, and conda ships no `r44` `ffbase`. conda *does* provide a precompiled `r42` `ff`/`ffbase` binary pair, so pinning R 4.2 sidesteps the compile entirely.
- **conda-forge toolchain** (`gcc`/`gxx`/`gfortran`), *not* the cluster's lmod compilers, ABI-matched to conda's R; strict channel priority so bioconda never shadows a conda-forge build.
- **Source layer** (`remotes`, `upgrade="never"` so conda binaries are never rebuilt): `neonUtilities` (~2.4.x, matching the `getProductInfo`/`loadByProduct` API the script uses), `REddyProc` 1.3.2 (with `solartime`, `bigleaf`), `eddy4R.base` and `eddy4R.qaqc` at `NEONScience/eddy4R` commit `898a72d` (the v4 `renv.lock` pin), and `NEON.gf` (installed from the fork).
- **Pinned package sources:** the source layer resolves against four dated **Posit CRAN snapshots** (2022-02-28 … 2023-10-22) that match the v4 era and predate CRAN's archiving of several `eddy4R` dependencies (e.g. `DataCombine`). A project `Makevars` (`-std=gnu17`) lets those older sources compile under conda's modern gcc.

The whole environment builds from one script ([`build_env.sh`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/scripts/neon_forcing/setup/build_env.sh): solve → source-install → smoke-test).

!!! note "Team reference — HiPerGator"
    Build on a node with outbound internet and CPU/time headroom (an interactive `hpg-dev` session), not a login node — long compiles trip login-node process limits. `build_env.sh` runs `module purge` first so no lmod compiler leaks into the solve. Specification files: [`scripts/neon_forcing/setup/`](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/neon_forcing/setup) (`environment.yml`, `install_source_pkgs.R`, `Makevars.conda`, `build_env.sh`).

---

## Forcing-generation procedure

Generation is a two-step pipeline: an authenticated raw download, then a fully offline conversion.

**Step 1 — raw download (authenticated).** NEON's `/data/` endpoint returns 403 to anonymous requests from HPC address ranges; a free NEON API token (scope `rate:public`) lifts that block, so the raw pull runs on a compute node. `download_raw.R` (via `run_download.sh`) fetches the ten source products for the requested span and stages them, integrity-checked, into a shared archive.

**Step 2 — offline conversion (no network).** The forked `flow.api.clm.R` (`doDnld = FALSE`, via `run_forcing.sh`) copies the staged zips into a per-session working directory, stacks and gap-fills them with REddyProc, and writes one combined CTSM-DATM NetCDF per month. The full record was produced in **annual chunks** under a 64 GB allocation (each chunk covers one year; REddyProc's ≥ 90-day flux minimum precludes monthly chunks).

!!! note "Team reference — HiPerGator"
    Raw archive: `/blue/gerber/earth_models/neon/raw/OSBS` — the full 2016-08 → 2025-06 span, ten products, ~11 GB (1063 zips). The token is read from `~/.neon_token` (mode 600, never committed). Generation scope is set by `NEON_START`/`NEON_END` in the wrapper.

### The 2017 precipitation remediation

NEON's **primary weighing gauge (DP1.00044.001) was physically offline at OSBS from July through December 2017** (raw values all-NA, final QC flag set), so the generator wrote gap-fill for those six months. The record was repaired from the **secondary tipping bucket (DP1.00045.001)** as a post-processing step (`splice_2017_precip.py`) — **no source edit**: the substituted precipitation is written back with its quality flag set to `PRECTmms_fqc = 5`, and a backup is written first.

The substitution was validated three ways:

- The two gauges agree to **r = 0.96 / ~2 % on monthly totals** across their 35 overlapping months.
- The spliced values **bit-match** the raw tipping-bucket record at each UTC timestamp.
- The heaviest September-2017 rainfall lands on **2017-09-10**, coincident with **Hurricane Irma**'s passage over north-central Florida — an independent temporal-alignment check.

The likely reason the pre-built v4 set begins in 2018 is this same 2017 gauge outage; recovering it is one of the ways the custom record extends v4.

---

## Output dataset

**101 monthly NetCDF files, `OSBS_atm_2017-02.nc` … `OSBS_atm_2025-06.nc`** — generated on HiPerGator; the `*.nc` are gitignored and not distributed in the repository (regenerate from the fork and environment specification below).

| Property | Value |
|---|---|
| Cadence | one combined file per calendar month |
| Grid | single point — `lat = 1`, `lon = 1` (with `LATIXY`/`LONGXY`) |
| Time axis | half-hourly, `gregorian` calendar (48 records/day; e.g. 1344 for February) |
| Fill value | `-9999` |
| Meteorology | `FLDS`, `FSDS`, `PRECTmms`, `RH`, `PSRF`, `TBOT`, `WIND`, and reference height `ZBOT` |
| Quality flags | a `<var>_fqc` gap-fill flag on each of the seven measured variables (not on `ZBOT`) |

Humidity is carried as **relative humidity (`RH`, %)**; CDEPS converts RH → specific humidity internally, so no `QBOT` is stored. The record is a **strict superset of pre-built v4** (2018-01 → 2024-12): it adds the 11 months from 2017-02 (the eddy-covariance bundle's start) and the 6 months through 2025-06 (the RELEASE-2026 cut), RELEASE-2026 throughout.

---

## Validation

The custom dataset was validated mechanically and scientifically before hand-off.

**Reproduce-v4 (fidelity).** Run over 2018 and compared to pre-built v4 timestep-by-timestep: at every step where both records carry a real measurement (`fqc = 0`), the two are **bit-identical for all seven measured variables**. All divergence is confined to gap-filled timesteps (differing RELEASE and gap-fill-window). Precipitation annual total differs by +2.9 %. Where the pipeline carries a real measurement, it reproduces v4 to machine precision. *(`neon_v4_regression.py` — the forcing analog of the MERIT regression test.)*

**Whole-record QC.** Structural checks (record count = days × 48, monotonic time, quality flags present), gap-fill fractions, and physical-range/climatology sniff tests across all 101 files — **0 NaN**. *(`neon_forcing_qc.py`.)*

**CTSM ingestion.** The full 101-file stream drove a cold-start single-point + hillslope case end-to-end; forcing-driven fields matched the v4-driven run to **≤ 0.03 %** over the overlap. This surfaced one production requirement — see the callout below.

**AD spinup.** A cold-start accelerated-decomposition spinup on the custom forcing **converged**: ~195 simulated years, 20-year block-mean drift **< 1 %** on all carbon pools.

!!! warning "Production requirement: `dtlimit = -1` on both NEON streams"
    A cycled forcing stream (`taxmode = cycle`) that wraps past its last record trips CDEPS's default `dtlimit = 1.5` and hard-crashes the model (`dshr_strdata_mod.F90:1050`). This is a stock CDEPS behavior, not a data problem. Setting **`dtlimit = -1`** on both NEON streams in `user_nl_datm_streams` (CDEPS's own escape hatch for streams not cycling on January boundaries) resolves it — namelist-only, no rebuild. It is required for any cycled spinup on this forcing.

---

## Reproducibility and availability

| Component | Location (GitHub) |
|---|---|
| Forcing generator (fork) | [`cdevaneprugh/NCAR-NEON`](https://github.com/cdevaneprugh/NCAR-NEON) `uf-osbs` @ `07786dd` — fork of [`NEONScience/NCAR-NEON`](https://github.com/NEONScience/NCAR-NEON) (AGPL-3.0) |
| Environment build | [`scripts/neon_forcing/setup/`](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/neon_forcing/setup) — conda reconstruction of upstream [`renv.lock`](https://github.com/NEONScience/NCAR-NEON/blob/main/renv.lock) |
| Driver + QC scripts | [`scripts/neon_forcing/`](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/neon_forcing) — `download_raw.R`, `run_download.sh`, `run_forcing.sh`, `splice_2017_precip.py`, `neon_v4_regression.py`, `neon_forcing_qc.py` |
| Phase record | [`phases/I-neon-forcing.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/phases/I-neon-forcing.md) |

!!! note "Team reference — HiPerGator paths"
    Raw archive `/blue/gerber/earth_models/neon/raw/OSBS`; custom output `swenson/data/datm/neon_OSBS/custom/OSBS/atm/` (the `*.nc` are gitignored; provenance in the directory README). External readers can regenerate from the fork + environment specification above against their own NEON download.

To drive a CTSM case with this forcing (compset, coordinates, stream settings, and the `dtlimit = -1` requirement), see the drop-in case recipe: [`docs/neon-forcing-case-recipe.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/docs/neon-forcing-case-recipe.md) in the hpg-esm-tools repo.

---

## Cross-References

- [NEON Sites](../research/neon-sites.md) — site background and the OSBS reference cases
- [Swenson Implementation Overview](index.md) — the hillslope-parameter track this forcing work runs alongside
- NCAR-NEON pipeline — Wieder et al. (2023), *Geosci. Model Dev.* 16, 5979–6000, DOI [10.5194/gmd-16-5979-2023](https://doi.org/10.5194/gmd-16-5979-2023)
