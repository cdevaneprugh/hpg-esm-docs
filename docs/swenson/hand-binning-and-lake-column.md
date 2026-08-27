# HAND Binning and Lake Column

*The OSBS production hillslope file departs from the Swenson default 4-aspect × 4-elevation-bin structure in two coupled ways: a 24-bin TAI-focused HAND scheme replaces the 4-bin equal-area partition, and a dedicated lake column at chain index 1 represents aggregated NWI open water. The two changes are documented together because the bin scheme and the lake column are co-designed — the lake anchors the bottom of the chain and the lowest land bin sits just above it.*

---

## Motivation

The Swenson global methodology bins HAND so each elevation bin holds approximately the same number of pixels per aspect, with a mandatory 2 m upper bound on the lowest bin. On flat terrain this produces near-zero lowest-bin mean HAND — earlier OSBS runs at 4 bins × 4 aspects gave Q25 = 0.00027 m for the lowest bin, because 25 % of the production-domain pixels have HAND below LIDAR noise. A bin whose mean elevation sits at the LIDAR noise floor cannot resolve the saturation gradient that defines the terrestrial-aquatic interface.

The 24-bin scheme addresses this by tilting bin resolution toward the TAI and lifting the bin-width floor to the LIDAR vertical noise budget.

![HAND map of the production domain (R4C5-R12C14, 9 × 10 km, 1 m). Dark purple is low HAND (near streams and flood-zone interiors); bright yellow is upland ridge; white pixels are NWI open water excluded from land bins. The TAI is the colour band where the saturation gradient is steepest — the 24-bin scheme concentrates bin resolution there.](images/production_hand_map.png)

---

## 24-bin TAI-focused scheme

The production scheme uses 24 bins total: 12 flood-zone bins (HAND < 0 m, set by the depression-filled reference DEM) plus 12 upland bins (HAND ≥ 0 m). The width allocation tilts toward the TAI core:

| Bin range | Width | Count |
|---|---|---|
| Deep tail sentinel | 2.35 m | 1 |
| Deeper flood zone | 1.0 m | 1 |
| Deep flood-zone TAI | 0.5 m | 2 |
| **TAI core (flood-zone + stream margin)** | **0.25 m** | **10** |
| Inner upland TAI | 0.5 m | 3 |
| Mid upland | 1.0 m | 4 |
| Mid-to-ridge | 2.0 m | 2 |
| Ridge sentinel | ~7 m | 1 |

The geometric width progression outward from the TAI core (0.25 m → 0.5 m → 1.0 m → 2.0 m → ~7 m) tracks the attenuation of TAI process gradients into the terrestrial realm: saturation, biogeochemistry, and vegetation transitions are sharpest near the wet/dry boundary and smooth into nominal upland tens of meters away.

### Why 0.25 m as the bin-width floor

NEON DP3.30024.001 (1 m raster LIDAR) has a stated vertical accuracy of 0.10 m. The empirical residual standard deviation at OSBS, measured against benchmark check points, is 0.058 m. Taking 2σ as the threshold below which two HAND values are not reliably distinguishable: 2σ ≈ 0.116 m, rounded to 0.25 m for headroom (accounts for systematic spatial trends in the residuals). Bins narrower than 0.25 m would partition pixels into adjacent classes by LIDAR noise rather than by terrain.

### Why we accept non-monotonic bin areas

Swenson's bin algorithm targets approximately equal per-bin areas; CTSM imposes no monotonicity requirement on `hill_area`. The 24-bin scheme allows bin areas to vary with terrain heterogeneity. The smoothness criterion used instead is **per-meter pixel density** (area ÷ bin width): the density curve rises smoothly to a peak and tapers, tracking the underlying HAND distribution. The production scheme's per-meter density rises from 0.62 km²/m at the deepest sentinel to a peak of 13.9 km²/m at bin 13 (just above stream level, where OSBS pixel density peaks) and tapers smoothly to 0.92 km²/m at the ridge sentinel. The density curve is monotonic toward the peak from each side — the physically meaningful invariant for gridcell aggregation.

---

## Outlier handling

Q01 (raw HAND) = −6.35 m and Q99 = +17.02 m define the binning envelope for the production domain. Pixels outside this envelope are **discarded**, not clipped into sentinel bins, for two reasons:

1. Both tails are real terrain. Outlier diagnostic on the production domain (2026-05-01) shows the sharpest density breaks at Q01 (33.5× density step) and Q99 (34.8×), with negligible singleton mass beyond. The tails are not isolated noise pixels; they are continuous topography that would inflate sentinel-bin weights if retained.
2. Clipping into sentinel bins would force the deepest pit and ridge extrema to share a bin with a much larger area of moderately deep / high terrain, distorting both the mean HAND and the gridcell-aggregated mass-and-energy balance for those bins.

The discard removes ~1.5 M pixels (~2 % of the production-domain land area).

Binning is performed on **raw HAND** (elevation minus pit-filled DEM, signed) rather than conditioned HAND (clipped to ≥ 0). Raw HAND preserves the depth of pixels in filled depressions, which is the physically relevant quantity for the flood-zone bins; conditioned HAND would force every below-stream pixel to HAND = 0, collapsing the FZ side of the chain.

---

## Water masking strategy

Two water masks are computed and used at different stages:

| Mask | Coverage | Used for |
|---|---|---|
| **Narrow stream mask** | Pixels above the accumulation threshold (`A_thresh = 0.5 × Lc² = 63,362` cells at 1 m) | Catchment delineation, flow-accumulation arithmetic, network density |
| **Wide water mask** | Stream mask ∪ NWI open-water polygons (Lacustrine + Palustrine Unconsolidated Bottom) | Exclusion from HAND binning and DTND statistics; aggregated into the lake column |

The wide mask ensures that lake-surface pixels do not contaminate land-bin HAND statistics: an NWI-identified lake has water-surface elevation that, in HAND terms, is whatever the depression-fill algorithm filled the lake to (usually a uniform fill level). Counting those pixels into land bins would skew the lowest land bins toward zero HAND, undoing the TAI signal the 24-bin scheme is designed to resolve.

### NWI mask hole-fill

Raw NWI polygons (downloaded from the NWI v3.2 OSBS subset) miss interior holes that the 1 m LIDAR confirms are contiguous open water: small islands inside larger features, sub-polygon gaps where the digitizer missed thin shore strips, and mis-aligned edges. A `scipy.ndimage.binary_fill_holes` pass over the rasterized NWI mask closes approximately 400,000 pixels of these holes. The hole-fill is applied before the mask is used downstream (wide-mask construction, lake-area aggregation, perimeter computation).

---

## Lake column construction

The OSBS production file places a dedicated lake column at chain index 1, shifting land columns to indices 2-25. The lake column represents the aggregate of all NWI open water in the production domain as a single submerged element.

| Field | Value | Rationale |
|---|---|---|
| `column_index` | 1 | Lake at chain bottom. Preserves the PI's SourceMod loop structure (terminal-column gate at `cold == ispval`) by keeping the lake first in the SurfaceWaterMod iteration. |
| `downhill_column_index` | -9999 | Terminal column (drains to stream channel under routing-on; bounded by `tdepth_grc` under routing-off). |
| `hillslope_index` | 1 | Same single aspect as the 24 land bins. |
| `hill_elev` | −6.0 m | Chain-bookkeeping value. Set 0.87 m below the deepest land bin's mean (−5.13 m, bin 1 of the production output) to keep the column chain monotonic in elevation. **Does not represent a physical lake bottom** — empirical lake bathymetry from NWI digitizing and Lee 2023 spill-stage data is far shallower (NWI mean ~−2.5 m, Lee/pipeline spill 2.6-3.3 m), but the deepest land bin's HAND (−5.13 m) is the binding constraint on monotonicity. PI direction (2026-05-04): tuning the lake elevation toward a physical lake depth is deferred to model-output review; the chain-integrity constraint comes first. |
| `hill_distance` | 0.5 × Bin 1 distance, computed dynamically per rep | The inter-column Darcy gradient computation in `SoilHydrologyMod.F90` divides by `col%hill_distance`. The denominator `d(Bin1) − d(Lake)` must stay positive. Bin 1's trap-fit DTND at OSBS is small (~3 m), so a static lake distance of ~5 m would invert the gradient sign for some reps. Setting the lake distance to half of Bin 1's distance per rep guarantees positivity by construction. |
| `hill_area` | Σ(water_mask × pixel_area) ≈ 11.08 km² (11,082,394 px; rescaled per rep to ≈ 0.021 km²) | Total NWI-masked water area in the production domain after hole-fill. The `binary_fill_holes` step adds ~400,000 interior pixels to the 10.68 km² pre-fill mask, giving 11.08 km². Production domain has 103 NWI water features. |
| `hill_width` | 0.5 × NWI total perimeter ≈ 48,270 m (rescaled per rep to ≈ 90 m) | PI direction (2026-04-25). Half the perimeter approximates the effective lateral exchange surface between the lake and its land-facing margin. Inert under the operative routing-off configuration. |
| `hill_slope` | 0 | Water surface is horizontal. |
| `hill_aspect` | 0 | Inconsequential for the flat lake column. |
| `hill_bedrock_depth` | 0 | Matches the hillslope convention; inert under CTSM's Uniform soil profile method. |

![Four hillslope parameters across the 25 columns of the production NetCDF: mean raw HAND (top-left), mean DTND (top-right), per-rep hillslope element area (bottom-left), and lower-edge width (bottom-right). The chain is ordered lake → deepest flood-zone bin (1) → ridge bin (24). Colour-coded: lake (light blue), flood zone with raw HAND < 0 (dark red), upland with raw HAND ≥ 0 (green). The HAND chain rises monotonically from −6.0 m at the lake to +12.5 m at the ridge bin. DTND grows smoothly from ~1 m at the lake to ~350 m at the ridge. Area is small in the flood zone (narrow bins resolve the TAI core) and largest in mid-upland (where bin widths broaden). Width is set by the trapezoidal plan-form fit and is largest near the stream-margin bins.](images/production_hillslope_params.png)

---

## Per-rep rescaling

The lake-column area (11.08 km²) is far larger than any single representative hillslope's footprint, because OSBS open water is the aggregate of 103 NWI features distributed across the 90 km² production domain. To fit the column inside a representative-hillslope shape, the pipeline computes:

- **`nhill_implicit`** — the implicit number of representative-hillslope copies that tile the gridcell. For the 2026-05-05 production: `nhill_implicit ≈ 533.7`.
- **Per-rep area** = total area / `nhill_implicit`. Lake column per rep: 11.08 km² / 533.7 ≈ 0.021 km² = 20,765 m².
- **Per-rep width** = `hill_width` / `nhill_implicit`. Lake per rep: 48,270 m / 533.7 ≈ 90.4 m. (`hill_width` = 48,270 m is half the total NWI perimeter of 96,540 m.)

Within the representative hillslope, the lake column ends up with `wtlunit ≈ 12.3 %`, calibrated to the NWI water footprint as a fraction of the production domain (~12 % open water). The land bins partition the remaining 87.7 % according to their NWI-masked, water-excluded pixel areas.

---

## SPILLHEIGHT and the SourceMod retirement

Before Phase E.5, the pipeline binned on **conditioned HAND** — the depression-filled DEM's elevation above stream. Conditioned HAND is zero or positive by construction: pixels inside filled depressions sit *at* the depression's fill level, not below it. Equal-area binning on this signal produced no flood-zone columns; the lowest land bin's mean HAND landed at LIDAR noise (Q25 = 0.00027 m on earlier 4-bin runs), as documented in the [Motivation](#motivation) section.

To represent the wet, low-lying terrain visible in the NEON LIDAR, the PI installed two SourceMods:

- **`HillslopeHydrologyMod.F90`** — modified `InitHillslope` to subtract a uniform `SPILLHEIGHT` constant (originally 0.2 m, hard-coded) from every column's `hill_elev` after reading the NetCDF. This effectively lowered the whole chain at runtime.
- **`SurfaceWaterMod.F90`** — added runoff suppression for any column satisfying `hill_elev + h2osfc < 0`. Columns lowered below the stream by the SPILLHEIGHT subtraction would pond instead of drain, producing a stratified submergence regime.

Combined, the two SourceMods turned the lowest land bins into "usually wet, sometimes dry" columns and the lake column (sitting at `hill_elev = −SPILLHEIGHT` in the input file) into the permanently submerged anchor of the chain.

### Why raw-HAND binning superseded this

Phase E.5 switched the pipeline's binning input from conditioned HAND to **raw HAND** (elevation above stream measured on the original DEM, signed). Raw HAND preserves the depression-fill depth as a negative HAND value, so flood-zone columns emerge from the data:

- The deepest land bin (chain index 2 in the production NetCDF) has a mean HAND of −5.13 m.
- Eleven additional flood-zone bins (chain indices 3 through 13) resolve the saturation gradient up to HAND = 0.
- The bin areas, per-meter density, and Darcy-relevant geometry all derive from the original LIDAR rather than from a runtime offset to the deposited input file.

The runtime elevation shift is no longer needed because the flood zone is in the file. The SourceMod's purpose — creating submerged columns to represent low-lying terrain — is solved at file-construction time rather than at runtime.

### PI decision (2026-04-30)

The SourceMod's runtime mechanism was retired. `SPILLHEIGHT = 0.0` set via namelist override (rather than editing the SourceMod constant) keeps the SourceMod files retained but inert:

```fortran
spillheight = 0.0
```

This lowers the maintenance burden across cases — no Fortran rewrite, no case-by-case SourceMod synchronization — while disabling the runtime elevation shift. The lake column's `hill_elev = −6.0 m` (chain-bookkeeping value, locked 2026-05-04; see Lake column construction above) replaces the earlier `−SPILLHEIGHT` framing as the chain anchor.

Both approaches produce a stratified submergence regime; the data-derived approach is more faithful to the underlying NEON LIDAR (basin depths read from the input file rather than imposed as a domain-wide uniform offset) and removes the need to keep the SourceMod's `SPILLHEIGHT` constant in sync with the pipeline.

---

## References

- Phase E.5 bin redesign and outlier strategy: [`swenson/phases/E.5-bin-redesign.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/phases/E.5-bin-redesign.md)
- Phase G lake column construction: [`swenson/phases/G-ctsm-lake-representation.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/phases/G-ctsm-lake-representation.md)
- Lake-column CTSM source audit (canonical parameter derivation): [`swenson/docs/lake-column-ctsm-audit.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/docs/lake-column-ctsm-audit.md)
- DOE BER (2017). [*Research Priorities to Incorporate Terrestrial-Aquatic Interfaces in Earth System Models*](https://science.osti.gov/-/media/ber/pdf/community-resources/Terrestrial-Aquatic_Interfaces_report.pdf), DOE/SC-0187 — TAI process-gradient framework
- Lee, E., Epstein, J. M., & Cohen, M. J. (2023). [Patterns of Wetland Hydrologic Connectivity Across Coastal-Plain Wetlandscapes](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2023WR034553). *Water Resources Research*, 59 — spill-stage data for chain-monotonicity sanity check
