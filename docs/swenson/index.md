# Swenson Implementation Overview

*Custom hillslope parameters for OSBS derived from 1m NEON LIDAR data using the Swenson & Lawrence (2025) representative hillslope methodology.*

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

## Implementation Arc

1. **Port code** -- Complete. Forked pysheds, ported Swenson's `pgrid.py` with hillslope methods, fixed NumPy 2.0 compatibility, created test suite. See [pysheds Fork](pysheds-porting.md).

2. **Validate** -- Complete. Reproduced Swenson's published results using MERIT DEM. All 6 parameters >0.92 correlation, 5 above 0.98. See [MERIT Validation](merit-validation.md).

3. **Adapt for OSBS** -- Complete. Added UTM CRS support to pysheds fork (Phase A). Determined full 1m resolution is feasible and necessary (Phase B). Established Lc = 300m via restricted-wavelength FFT (Phase C).

4. **Build pipeline** -- Complete. Rebuilt pipeline with hydrological DTND, Horn 1981 slope/aspect, 1m resolution, and Lc = 356m. Extracted shared analysis module. Three audits completed: equation verification, full pipeline audit, line-by-line divergence audit (Phase D). See [Methodology](osbs-implementation.md).

5. **Complete parameters** -- Pending. Stream channel parameters, bedrock depth, and DEM conditioning approach require field data and PI decisions (Phase E).

6. **Validate and deploy** -- Pending. Compare custom hillslope file to baseline, run CTSM branch simulation against osbs2 (Phase F).

## Current Status

The pipeline produces scientifically defensible hillslope parameters for the OSBS production domain (R4C5-R12C14, 90 tiles, 90M pixels at 1m). The latest production run (2026-03-17) completed in 22.6 minutes with Lc = 356m and A_thresh = 63,362 pixels.

**Remaining work:**

- **Phase E (parameter completion):** Stream channel depth, width, and slope are interim power-law estimates. Bedrock depth is set to 0. The hillslope structure (4 aspects x 4 bins vs 1 aspect x 8 bins) is under consideration. These require PI decisions and potentially field data or empirical relationships.
- **Phase F (CTSM validation):** Run a short branch simulation from the osbs2 baseline (year 861) with the custom hillslope file and compare outputs.

---

## Development History

| Date | Phase | Key Decision |
|------|-------|--------------|
| 2025-12 | -- | Ported Swenson's pgrid.py to pysheds fork, created initial test suite |
| 2026-01 | -- | 9-stage MERIT validation achieved >0.95 on 5/6 parameters |
| 2026-02 | A | Added UTM CRS support to pysheds fork (82 tests, 0 failures) |
| 2026-02 | B | Full 1m resolution confirmed feasible (29 GB peak, 6 min for 90M pixels) |
| 2026-02 | C | Lc = 300m established via restricted-wavelength FFT |
| 2026-02 | -- | MERIT post-audit: area fraction improved 0.82 to 0.92 |
| 2026-03 | D | Pipeline rebuilt with all fixes; equation and divergence audits completed |

Internal phase tracking files: `swenson/phases/A-pysheds-utm.md` through `F-validate-deploy.md`.

---

## Tools and Repositories

| Resource | Location | Purpose |
|----------|----------|---------|
| pysheds fork | [cdevaneprugh/pysheds](https://github.com/cdevaneprugh/pysheds) (branch: `uf-development`) | Flow routing, HAND, DTND, hillslope classification |
| Swenson's codebase | [swensosc/Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) | Reference implementation |
| Processing scripts | [hpg-esm-tools/swenson/scripts/osbs/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/osbs) | OSBS pipeline |
| Validation scripts | [hpg-esm-tools/swenson/scripts/merit_validation/](https://github.com/cdevaneprugh/hpg-esm-tools/tree/main/swenson/scripts/merit_validation) | MERIT DEM regression test |

---

## Cross-References

- [Hillslope Hydrology](../research/hillslope.md) -- Theoretical background on CTSM hillslope mode, the six geomorphic parameters, and physical processes
- [Swenson & Lawrence 2025 summary](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/docs/papers/Swenson_2025_Hillslope_Dataset_Summary.md) -- Detailed paper summary with equations and methodology
- [Representative_Hillslopes](https://github.com/swensosc/Representative_Hillslopes) -- Swenson's original processing pipeline
