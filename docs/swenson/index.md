# Swenson Implementation Overview

*Custom hillslope parameters for OSBS derived from 1m NEON LIDAR data using the Swenson & Lawrence (2025) representative hillslope methodology.*

---

## Goal

Swenson & Lawrence (2025) published a global hillslope dataset derived from MERIT DEM at ~90m resolution. This dataset is available independently (not bundled with CTSM) and can be used with CTSM's hillslope hydrology mode. However, Swenson & Lawrence explicitly noted that this resolution "may not be fine enough to capture topographic variations in areas of very low topographic relief, such as wetlands." OSBS is exactly this case — a low-relief wetlandscape where the Terrestrial-Aquatic Interface depends on meter-scale elevation differences.

The goal of this work was to implement Swenson's methodology using 1m NEON LIDAR data to produce custom hillslope parameters that resolve the fine-scale topography controlling wetland-upland transitions at OSBS.

---

## Key Differences from Swenson

| Aspect | Swenson (Global) | Our Implementation |
|--------|-------------------|--------------------|
| DEM source | MERIT (~90m) | NEON LIDAR (1m) |
| Coordinate system | Geographic (lat/lon) | UTM Zone 17N (meters) |
| pysheds version | Swenson's fork | Our fork (`uf-development`) with UTM CRS support |
| Processing scope | Global tiles | Single site (OSBS, 90 km^2) |
| Resolution | Direct processing at ~90m | Full 1m resolution (no subsampling) |
| Edge handling | Not needed (continuous global data) | Connected-component extraction + edge trimming |
| DTND calculation | Hydrological, haversine distance | Hydrological, Euclidean distance (UTM-aware pysheds) |
| Lc determination | FFT natural peak | Restricted-wavelength FFT (min 20m cutoff) |

---

## Guiding Principle

**Do not reinvent the wheel.** Swenson's paper and codebase served as the template throughout. The implementation followed Swenson's algorithms as closely as possible, deviating only where the change in resolution or coordinate system required adaptation.

---

## Phases

The pipeline was built up incrementally. Early work (December 2025 to January 2026) focused on porting Swenson's `pgrid.py` to the pysheds fork and reproducing his published MERIT results as a methodology validation, before the phase scheme was introduced. As the OSBS implementation revealed problems that couldn't be solved in a single pass, remaining work was split into discrete phases (A–H) — each addressing a specific blocker. The phases cluster into five groups matching the roadmap in [`swenson/STATUS.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/STATUS.md).

**Pre-phase (2025-12 to 2026-01) — Methodology validation.** Ported Swenson's `pgrid.py` to the pysheds fork; nine-stage MERIT regression achieved >0.95 correlation on five of six parameters (area fraction later improved from 0.82 to 0.92 in a 2026-02 post-audit).

**Phases A–D (2026-02 to 2026-03) — Pipeline foundations.** A added UTM CRS support to the pysheds fork so it could process OSBS NEON tiles (EPSG:32617) in projected coordinates; 82 unit tests, 0 failures. B confirmed the full 1 m LIDAR could be processed at 64 GB without subsampling (29 GB peak for 90 M pixels). C established **Lc = 356 m** via a restricted-wavelength FFT (20 m cutoff), circumventing the k² Laplacian artifact that surfaces at high-resolution DEMs. D rebuilt the pipeline with the A/B/C fixes; three independent audits (equation verification, full pipeline audit, line-by-line divergence audit against Swenson's original code) all cleared.

**Phases E.5 + E.6 (2026-04 to 2026-05) — Parameter set.** E.5 replaced Swenson's 4-bin equal-area scheme with the 24-bin TAI-focused scheme (12 flood-zone + 12 upland, 0.25 m floor) and locked raw-HAND binning with Q01/Q99 outlier discard; lake column placement at chain index 1 with `hill_elev = −6.0 m`. E.6 closed ~400 K interior polygon gaps in the rasterized NWI water mask via `scipy.ndimage.binary_fill_holes`.

**Phases F + G (2026-05, complete) — Validate and deploy.** G Stage 1 completed the submerged lake column and released the production NetCDF `hillslopes_osbs_production_c260505.nc` on 2026-05-05. Phase F consumed that file: a 600-yr accelerated AD spinup with the operative case `osbs.swenson.spinup` completed 2026-05-14 and was analyzed 2026-05-19. Three verdicts: convergence PASS (`drift_50yr = 0.48 %`), TAI signal absent (`O_SCALAR ≈ 1.0`), lake column stable. The bridge-zone water-table anomaly at chain indices 3–6 was resolved by the PI (2026-08-19); the PI continues to investigate the O_SCALAR anoxia absence (see Current Status below).

**Phase H (2026-05) — Stream-side coupling.** Track A completed the mesh-mode workaround for CTSM Issue #1432 (see [Lateral Flow and Routing](lateral-flow-and-routing.md)) and remains complete and available. Tracks B and C — enabling `use_hillslope_routing = .true.` — were retired 2026-08-19 (the PI has the routing/drainage situation handled); the 2026-05-19 routing-gate source audit established that inter-column lateral flow already runs under `use_hillslope = .true.` regardless.

Detailed phase records: [`swenson/phases/`](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/phases) in the hpg-esm-tools repo.

---

## Current Status

The pipeline produces the OSBS production hillslope file `hillslopes_osbs_production_c260505.nc` — 25 columns (1 lake at chain index 1 + 24 land bins) on a single aspect, computed over the R4C5–R12C14 production domain (90 tiles, 9 × 10 km, 0 % nodata, 1 m resolution). Deployed in the operative case `osbs.swenson.spinup`.

**Phase F results (2026-05-19 closeout):**

- **Convergence PASS** — `drift_50yr = 0.48 %`, well under the 3 % threshold.
- **TAI signal absent** — `O_SCALAR ≈ 1.0` across the full 25-column × 600-yr array; the expected anoxia depression in decomposition output does not emerge.
- **Lake column stable** — max 5.78 m at year 107, drained to 2.5 m by year 600. No runaway accumulation; the Darcy-drain mitigation contemplated in Phase H is not required.

**Production hillslope file was un-frozen 2026-07-15**; the PI is proceeding via soil-value adjustments. Of the two scientific questions Phase F surfaced:

1. **O_SCALAR anoxia absence** — the TAI carbon-side signature the column structure was designed to resolve is not visible in the current output. Headline issue for the project's central scientific question; still under PI investigation.
2. **Bridge-zone anomaly** at chain indices 3–6 (HAND −3 to −1.5 m) — the deepest water tables of any lower-hillslope column. **Resolved by the PI (2026-08-19)**; the hillslope drains properly now (the specific fix is not recorded on our side).

Post-AD continuation `osbs.swenson.post-ad` (200 yr, completed 2026-05-21) is idle pending the PI investigation. Phase H Tracks B/C (enable `use_hillslope_routing` for the CTSM-internal stream-water ledger) were retired 2026-08-19.

---

## Tools and Repositories

The local paths below reflect our team's HiPerGator setup, where `$BLUE = /blue/gerber/cdevaneprugh/`. They are provided for internal reference — external readers should clone the repositories at right into whatever workspace they prefer.

| Resource | Team local path | Repository | Purpose |
|----------|-----------------|------------|---------|
| pysheds fork | `$BLUE/pysheds_fork/` | [cdevaneprugh/pysheds](https://github.com/cdevaneprugh/pysheds) (branch: `uf-development`) | Flow routing, HAND, DTND, hillslope classification |
| Swenson's codebase | `$BLUE/Representative_Hillslopes/` | [swensosc/Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) | Reference implementation |
| Processing scripts | `$BLUE/hpg-esm-tools/swenson/scripts/osbs/` | [hpg-esm-tools/swenson/scripts/osbs/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/osbs) | OSBS pipeline |
| Validation scripts | `$BLUE/hpg-esm-tools/swenson/scripts/merit_validation/` | [hpg-esm-tools/swenson/scripts/merit_validation/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/merit_validation) | MERIT DEM regression test |

---

## Cross-References

- [Hillslope Hydrology](../research/hillslope.md) — Theoretical background on CTSM hillslope mode, the six geomorphic parameters, and physical processes
- [Swenson & Lawrence 2025 summary](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/docs/papers/Swenson_2025_Hillslope_Dataset_Summary.md) — Detailed paper summary with equations and methodology
- [Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) — Swenson's original processing pipeline
