# Swenson Implementation Overview

*Implementing the Swenson & Lawrence (2025) representative hillslope methodology for OSBS using 1m NEON LIDAR data.*

---

## Goal

Swenson & Lawrence (2025) published a global hillslope dataset derived from MERIT DEM at ~90m resolution. This dataset is available independently (not bundled with CTSM) and can be used with CTSM's hillslope hydrology mode. However, Swenson & Lawrence explicitly noted that this resolution "may not be fine enough to capture topographic variations in areas of very low topographic relief, such as wetlands." OSBS is exactly this case -- a low-relief wetlandscape where the Terrestrial-Aquatic Interface depends on meter-scale elevation differences.

The goal of this work was to implement Swenson's methodology using 1m NEON LIDAR data to produce custom hillslope parameters that resolve the fine-scale topography controlling wetland-upland transitions at OSBS.

---

## Key Differences from Swenson

| Aspect | Swenson (Global) | Our Implementation |
|--------|-------------------|--------------------|
| DEM source | MERIT (~90m) | NEON LIDAR (1m) |
| Coordinate system | Geographic (lat/lon) | UTM Zone 17N (meters) |
| pysheds version | Swenson's fork | Our own fork (`uf-development` branch) |
| Processing scope | Global tiles | Single site (OSBS) |
| Resolution handling | Direct processing | 4x subsampling for flow routing |
| Edge handling | Not needed (continuous global data) | Required (irregular NEON tile coverage) |
| DTND calculation | Haversine (geographic) | Euclidean (UTM meters) |

---

## Guiding Principle

**Do not reinvent the wheel.** Swenson's paper and codebase served as the template throughout. The implementation followed Swenson's algorithms as closely as possible, deviating only where the change in resolution or coordinate system required adaptation.

---

## Implementation Arc

The work has proceeded through six phases:

1. **Port code** -- Fork pysheds, copy Swenson's `pgrid.py` with hillslope methods, fix NumPy 2.0 compatibility, create test suite. See [pysheds Porting](pysheds-porting.md).

2. **Validate** -- Reproduce Swenson's published results using MERIT DEM through a 9-stage validation pipeline. Achieved >0.95 correlation on 5 of 6 geomorphic parameters. See [MERIT Validation](merit-validation.md).

3. **Plan OSBS** -- Document resolution-sensitive issues, identify adaptations needed for 1m data, design processing pipeline.

4. **Execute** -- Process 233 NEON LIDAR tiles through the pipeline, debug pysheds scaling issues, produce CTSM-compatible NetCDF output. **In progress — blocked by known issues.** See [OSBS Implementation](osbs-implementation.md).

5. **Fix pipeline** -- Resolve the four issues identified by the February 2026 audit: fix pysheds for UTM coordinates, determine processing resolution, establish trustworthy Lc, validate slope/aspect. *Pending.*

6. **Validate and deploy** -- Produce corrected hillslope parameters, verify physical plausibility, run CTSM branch simulations against the osbs2 baseline. *Pending.*

## Current Status

The methodology is validated and the pipeline runs, but the OSBS output has four known issues that affect the scientific validity of the hillslope parameters:

1. **DTND uses the wrong algorithm** — Euclidean distance to the nearest stream pixel instead of hydrological distance to the drainage-linked stream pixel. Corrupts distance and width parameters.
2. **4x subsampling discards 93.75% of data** — adopted after a single OOM failure at 64GB; full-resolution and 2x subsampling were never tested.
3. **Lc computed at 4m, not 1m** — the characteristic length scale has never been determined at full resolution. NumPy handles this easily.
4. **Slope/aspect not validated for UTM** — a known sign-convention bug was fixed in the MERIT validation but the fix was not carried to the OSBS pipeline.

The current NetCDF output should not be used for CTSM simulations until these are resolved. See [OSBS Implementation — Known Limitations](osbs-implementation.md#known-limitations) for technical details.

The fix work is organized into six phases (A through F), with Phases A-C (pysheds UTM fix, resolution testing, Lc determination) proceeding in parallel, followed by a pipeline rebuild (D), parameter completion (E), and validation (F).

---

## Tools and Repositories

| Resource | Location | Purpose |
|----------|----------|---------|
| pysheds fork | [cdevaneprugh/pysheds](https://github.com/cdevaneprugh/pysheds) (branch: `uf-development`) | Flow routing, HAND, DTND, hillslope classification |
| Swenson's codebase | [swensosc/Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) | Reference implementation |
| Processing scripts | [hpg-esm-tools/swenson/scripts/osbs/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/osbs) | OSBS pipeline |
| Validation scripts | [hpg-esm-tools/swenson/scripts/merit_validation/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/merit_validation) | MERIT DEM validation stages 1-9 |

---

## Cross-References

- [Hillslope Hydrology](../research/hillslope.md) -- Theoretical background on CTSM hillslope mode, the six geomorphic parameters, and physical processes
- [Swenson & Lawrence 2025 summary](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/docs/papers/Swenson_2025_Hillslope_Dataset_Summary.md) -- Detailed paper summary with equations and methodology
- [Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) -- Swenson's original processing pipeline
