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

2. **Validate methodology** -- Complete. Reproduced Swenson's published results using MERIT DEM. All 6 parameters >0.92 correlation, 5 above 0.98. See [MERIT Validation](merit-validation.md).

3. **Adapt for OSBS** -- Complete. Added UTM CRS support to pysheds fork (Phase A). Determined full 1m resolution is feasible and necessary (Phase B). Established Lc = 356 m via restricted-wavelength FFT (Phase C).

4. **Build pipeline** -- Complete. Rebuilt pipeline with hydrological DTND, Horn 1981 slope/aspect, 1m resolution. Extracted shared analysis module. Three audits completed: equation verification, full pipeline audit, line-by-line divergence audit (Phase D). See [Methodology](osbs-implementation.md).

5. **Refine parameter set** -- Complete. Locked the 24-bin TAI-focused HAND scheme (12 flood-zone + 12 upland, 0.25 m floor), single-aspect configuration, raw-HAND Q01/Q99 outlier discard, and dual water-mask strategy with NWI hole-fill (Phases E.5, E.6). See [HAND Binning and Lake Column](hand-binning-and-lake-column.md).

6. **Lake column construction** -- Complete. Added a submerged lake column at chain index 1 representing all NWI open water (≈ 10.68 km² aggregated); per-rep rescaling (`nhill_implicit ≈ 533`) calibrated lake `wtlunit` to 12.3 %. Production NetCDF released 2026-05-05 (Phase G Stage 1). See [HAND Binning and Lake Column](hand-binning-and-lake-column.md).

7. **Validate and deploy** -- In progress. A 600-yr accelerated AD spinup using the 2026-05-05 production hillslope file has completed (operative case `osbs.swenson.spinup`, 4-stream `h0/h1/h2/h3` configuration; `use_hillslope=.true.`, `use_hillslope_routing=.false.`). Analysis is in progress; findings will be documented in a subsequent pass. See [Lateral Flow and Routing](lateral-flow-and-routing.md) for the operative-case configuration.

8. **Stream-side coupling (routing-on)** -- Contingent. Phase H Track A (mesh-mode workaround for CTSM Issue #1432) is complete; Tracks B and C are on hold pending Phase F analysis. The original motivation — turn routing on to activate inter-column lateral flow — was retired when a 2026-05-19 source-code audit established that lateral flow already runs under `use_hillslope=.true.`. See [Lateral Flow and Routing](lateral-flow-and-routing.md).

## Current Status

The pipeline produces the OSBS production hillslope file `hillslopes_osbs_production_c260505.nc` — 25 columns (1 lake at chain index 1 + 24 land bins) on a single aspect, computed over the R4C5–R12C14 production domain (90 tiles, 9 × 10 km, 0 % nodata, 1 m resolution). The file is deployed in the operative case `osbs.swenson.spinup`; the 600-yr accelerated AD spinup has completed and analysis is in progress.

**Two items remain:**

- **Phase F analysis** -- writing up convergence, water-table differentiation across the column chain, lake-column dynamics, and any TAI signature. Findings will land in a separate documentation pass once the analysis pattern stabilizes.
- **Phase H Tracks B/C** -- whether to enable `use_hillslope_routing` to add the CTSM-internal stream-water ledger and stream-channel boundary condition. Decision depends on Phase F results.

---

## Development History

| Date | Phase | Key Decision |
|------|-------|--------------|
| 2025-12 | -- | Ported Swenson's pgrid.py to pysheds fork, created initial test suite |
| 2026-01 | -- | 9-stage MERIT validation achieved >0.95 on 5/6 parameters |
| 2026-02 | A | Added UTM CRS support to pysheds fork (82 tests, 0 failures) |
| 2026-02 | B | Full 1m resolution confirmed feasible (29 GB peak, 6 min for 90M pixels) |
| 2026-02 | C | Lc = 356 m established via restricted-wavelength FFT (20 m cutoff) |
| 2026-02 | -- | MERIT post-audit: area fraction improved 0.82 to 0.92 |
| 2026-03 | D | Pipeline rebuilt with all fixes; equation and divergence audits completed |
| 2026-04 | E.6 | NWI mask hole-fill (`binary_fill_holes`) closed ~400 K interior polygon gaps |
| 2026-04 | E.5 | Lake column placement at chain index 1; raw-HAND Q01/Q99 outlier discard locked (PI direction) |
| 2026-05 | E.5 | 24-bin TAI-focused HAND scheme locked; lake `hill_elev = −6.0 m`; dynamic `hill_distance` |
| 2026-05 | G | Production NetCDF `hillslopes_osbs_production_c260505.nc` released (25 columns, 1 aspect) |
| 2026-05 | H Track A | Mesh-mode workaround for CTSM Issue #1432 verified (paired 5-yr test/control smoke test) |
| 2026-05 | F | `osbs.swenson.spinup` 600-yr accelerated AD spinup completed (4-stream h0/h1/h2/h3 configuration) |
| 2026-05 | -- | Routing-gate source audit: inter-column lateral flow runs under `use_hillslope`, not `use_hillslope_routing`; Phase H Tracks B/C reframed as contingent |

Internal phase tracking files: `swenson/phases/A-pysheds-utm.md` through `H-lateral-flow.md`.

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
