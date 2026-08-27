# Lateral Flow and Routing

*CTSM hillslope mode is controlled by two related-but-distinct namelist toggles. This page documents what each one activates physically, based on a 2026-05-19 source-code audit of CTSM 5.3.085. The distinction is consequential: prior project framing conflated the two, and the operative `osbs.swenson.spinup` configuration is built on the corrected understanding.*

---

## Background

CTSM 5.3 exposes two namelist toggles for hillslope-mode physics:

```fortran
use_hillslope         = .true.    ! activates hillslope-mode column structure
use_hillslope_routing = .true.    ! activates stream-side machinery (optional)
```

A 2026-05-19 audit of CTSM 5.3.085 (`git rev dc3aa5ddc`) traced every source-code site gated by `use_hillslope_routing` and cross-checked the conclusions against empirical output from two cases: a 5-yr paired test/control smoke test (mesh-mode workaround validation, Phase H Track A) and the 600-yr accelerated AD spinup of the operative case. The audit established that **inter-column lateral subsurface flow runs under `use_hillslope = .true.` and does not require routing to be on**. Routing-on activates only the stream-side coupling at the chain terminus.

---

## What `use_hillslope = .true.` activates

### Inter-column lateral subsurface flow

`HydrologyDrainageMod.F90:127-160` dispatches the lateral-flow routines outside any routing gate:

```fortran
if (use_aquifer_layer()) then
   call Drainage(...)                          ! aquifer-layer mode (not used at OSBS)
else
   call PerchedLateralFlow(...)                ! ALWAYS, no routing gate
   call SubsurfaceLateralFlow(...)             ! ALWAYS, no routing gate
   if (use_hillslope_routing) then
      call HillslopeStreamOutflow(...)         ! routing-gated
      call HillslopeUpdateStreamWater(...)     ! routing-gated
   endif
endif
```

The OSBS case takes the `else` branch (`lower_boundary_condition=2` → `bc_zero_flux` → `use_aquifer_layer() = .false.`; see `bld/namelist_files/namelist_defaults_ctsm.xml:448` and `SoilWaterMovementMod.F90:229`). `PerchedLateralFlow` and `SubsurfaceLateralFlow` are therefore called every hydrology step regardless of routing.

### The Darcy gradient between adjacent columns

`SubsurfaceLateralFlow` (`SoilHydrologyMod.F90:2086+`) is the canonical inter-column water-table flow routine. Its main loop at `:2249-2402` processes every active hillslope column. Inside the loop, two paths:

**Internal columns** (those with a downhill neighbor, `col%cold(c) /= ispval`):

```fortran
! Darcy head gradient at :2261-2263
head_gradient = (col%hill_elev(c) - zwt(c)) &
              - (col%hill_elev(col%cold(c)) - zwt(col%cold(c)))
head_gradient = head_gradient / col%hill_distance(c)
```

The outflow volume at `:2358` is `qflx_latflow_out_vol = transmissivity × col%hill_width(c) × head_gradient`. The downhill column accumulates the inflow at `:2386-2388`. No routing gate anywhere in this path.

**Terminal column** (the chain bottom, `col%cold(c) == ispval`): the same Darcy machinery, but the "downhill neighbor" is the stream channel. Routing matters here for the stream-depth source — see the next section.

`PerchedLateralFlow` (`:1703+`) is structurally identical for the perched water table; the same column-to-column Darcy gradient runs at `:1817-1820`.

### Where the flow is applied to soil water

`SubsurfaceLateralFlow` at `:2433-2509`:

```fortran
qflx_net_latflow(c) = qflx_latflow_out(c) - qflx_latflow_in(c)
...
drainage(c) = qflx_net_latflow(c)                  ! for hillslope columns
...
drainage_tot = - drainage(c) * dtime
if (drainage_tot > 0.) then                         ! rising water table
   ! water added to h2osoi_liq, zwt rises
else                                                ! deepening water table
   ! water removed from h2osoi_liq, zwt falls
endif
```

A column that receives more lateral inflow than it sends out gets negative `drainage`, positive `drainage_tot`, and water added to `h2osoi_liq`. The transfer is real and applied to the soil-water state every step.

---

## What `use_hillslope_routing = .true.` additionally activates

Twelve source-code sites are gated by `use_hillslope_routing` in CTSM 5.3.085 (excluding declaration, broadcast, and log plumbing). Every one is on the stream-channel side of the column chain; none are inside the column-to-column flow path of `PerchedLateralFlow` or `SubsurfaceLateralFlow`.

| File:line | What it gates |
|---|---|
| `HillslopeHydrologyMod.F90:378-411` | Read `hillslope_stream_depth/width/slope` from the NetCDF |
| `HillslopeHydrologyMod.F90:475-507` | Compute `nhill_per_landunit`, `stream_channel_length`, `stream_channel_number` |
| `HillslopeHydrologyMod.F90:1078+` (`HillslopeUpdateStreamWater`, called only from `HydrologyDrainageMod.F90:150-158`) | Advance `stream_water_volume` |
| `HillslopeHydrologyMod.F90:977+` (`HillslopeStreamOutflow`, called only from `HydrologyDrainageMod.F90:151-153`) | Manning's-equation streamflow velocity |
| `SoilHydrologyMod.F90:1822-1829` | Perched-LF stream-depth source switch (internal ledger vs MOSART) |
| `SoilHydrologyMod.F90:2265-2272` | Subsurface-LF stream-depth source switch (internal ledger vs MOSART) |
| `SoilHydrologyMod.F90:2362-2367` | Losing-stream outflow cap |
| `WaterFluxType.F90:525-534` | Register `VOLUMETRIC_STREAMFLOW` history field |
| `WaterFluxType.F90:922-928` | Zero `volumetric_streamflow_lun` each step |
| `BalanceCheckMod.F90:274-280, 744-750` | Add `stream_water_volume` / `qflx_streamflow_grc` into gridcell water-balance ledger |
| `lnd2atmMod.F90:343+` | Sum streamflow over landunits for lnd→rof coupling |
| `lnd_import_export.F90:916-919` | Add stream component to subsurface-runoff field exported to coupler |

### The stream-depth source swap

The single physically-meaningful difference inside the lateral-flow code path is the stream-depth source at the terminal column. `SoilHydrologyMod.F90:2265-2272`:

```fortran
if (use_hillslope_routing) then
   stream_water_depth   = stream_water_volume(l) / &
                          lun%stream_channel_length(l) / &
                          lun%stream_channel_width(l)
   stream_channel_depth = lun%stream_channel_depth(l)
else
   stream_water_depth   = tdepth(g)               ! from MOSART via coupler
   stream_channel_depth = tdepth_bankfull(g)      ! from MOSART via coupler
endif
```

Routing-on uses an internal stream-water ledger updated each step by `HillslopeUpdateStreamWater`; routing-off uses MOSART's `tdepth_grc`, supplied via the coupler. The Darcy gradient at the terminal column is computed against some stream depth in both branches; only the source differs.

---

## Empirical confirmation

Two independent observations confirm inter-column flow is active under `use_hillslope_routing = .false.`:

1. **Routing.test vs routing.control 5-yr smoke test (2026-05-12).** QDRAI has negative values at hillslope columns in both the routing-on (`osbs.routing.test`) and routing-off (`osbs.routing.control`) cases. Min/max range is essentially identical: test = −4.32×10⁻⁵ / 1.47×10⁻⁵ mm/s; control = −4.15×10⁻⁵ / 1.43×10⁻⁵ mm/s. The lateral-flow signature is present whether routing is on or off; the values differ only at the ~10⁻⁶ scale (the boundary-condition delta).

2. **`osbs.swenson.spinup` h1a (600-yr AD, routing-off).** Column-level QRUNOFF min/max = −1.156×10⁻⁴ / 1.992×10⁻⁴ mm/s. ZWT spans 0.011 → 8.6 m across the column chain. If hillslope columns were hydrologically isolated 1-D soil columns receiving the same atmospheric forcing, there would be no negative column-level QRUNOFF and far less ZWT variation. Negative column QRUNOFF is the unambiguous fingerprint of inter-column lateral redistribution exceeding outflow at the receiving column.

---

## Implication for the OSBS configuration

The operative case `osbs.swenson.spinup` runs:

```fortran
use_hillslope         = .true.
use_hillslope_routing = .false.
```

with the 2026-05-05 production hillslope file (25 columns: 1 lake + 24 land bins on a single aspect). In this configuration, inter-column lateral subsurface flow is active between all 25 columns of the chain: Darcy gradients redistribute water from upland bins through the flood-zone bins to the lake column, and the terminal-column boundary condition at the lake is supplied by MOSART's `tdepth_grc` (which under the current DATM + MOSART setup defaults to 0, in effect treating the chain-bottom boundary as an empty stream channel).

A 600-yr accelerated AD spinup using this configuration completed 2026-05-14 and was analyzed 2026-05-19. Three verdicts: convergence PASS (`drift_50yr = 0.48 %`), TAI signal absent (`O_SCALAR ≈ 1.0`), lake column stable.

Phase H Tracks B and C would enable `use_hillslope_routing = .true.` and switch the terminal-column boundary to the internal `stream_water_volume` ledger. They were retired 2026-08-19 — the PI has the routing/drainage situation handled. The routing-on machinery is built and validated (Track A, below) but is not deployed.

---

## CTSM Issue #1432 and the mesh-mode workaround

!!! note "Status: built and validated, not deployed"
    The mesh-mode workaround described below is complete and verified but is **not active** in the operative `osbs.swenson.spinup` configuration. Issue #1432 only bites when `use_hillslope_routing = .true.`; the operative case runs with routing off, so `grc%area = spval` never reaches a live computation. Phase H Track A built and validated the workaround so it is ready if Tracks B/C later enable routing.

### The bug

In single-point NUOPC mode, `grc%area` is initialized to `spval` (~1×10³⁶) at `lnd_set_decomp_and_domain.F90:342`:

```fortran
ldomain%area(1) = spval
```

and propagated to `grc%area(g)` via `initGridCellsMod.F90:179`. The value is never repopulated. Under `use_hillslope_routing = .false.`, the routing gate at `HillslopeHydrologyMod.F90:475` protects every downstream use of `grc%area`, so the `spval` does not corrupt any computation.

Under routing-on, the same gate becomes a release valve. `HillslopeHydrologyMod.F90:486-487`:

```fortran
nhill_per_landunit(nh) = grc%area(g) * 1.e6_r8 * lun%wtgcell(l) &
     * pct_hillslope(l,nh) * 0.01 / hillslope_area(nh)
```

With `grc%area = spval`, `nhill_per_landunit ≈ 1×10³⁶` — physically nonsense, but no error is raised because CTSM has no defensive guards on `grc%area`. `stream_channel_length` inherits the `1e36` magnitude at `:501-502`, and from there every downstream stream-side computation produces garbage in silence.

This is canonical [CTSM Issue #1432](https://github.com/ESCOMP/CTSM/issues/1432) (open since 2021-07-20). The community has not encountered it in practice because Swenson's published hillslope users run gridded (the validated mode), and single-point users have run with routing off.

### The mesh-mode workaround

Replace the `PTS_LAT` / `PTS_LON` single-column shortcut with an explicit single-cell ESMF mesh (`LND_DOMAIN_MESH`). CTSM's mode-decision logic then routes through `lnd_set_decomp_and_domain_from_readmesh` instead of `lnd_set_decomp_and_domain_for_single_column`, and `ldomain%area` is populated from the mesh via `ESMF_FieldRegridGetArea()` — a real number, not `spval`.

Swenson recommends this workaround to single-point users. CTSM's shipped `tools/site_and_regional/mesh_maker.py` aborts on 1×1 input (`mesh_maker.py:214`), so Phase H Track A wrote [`make_osbs_scrip.py`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/scripts/osbs/make_osbs_scrip.py) (`swenson/scripts/osbs/`) to produce a single-cell SCRIP-format NetCDF; the prebuilt `ESMF_Scrip2Unstruct` binary then converts it to the ESMF mesh CTSM reads.

Phase H Track A (2026-05-11 to 2026-05-12) built the single-cell ESMF mesh for the production domain, the supporting input file conversions (surface dataset reformat, domain mesh), and a paired test/control case configuration. The 2026-05-12 smoke test confirmed `grc%area = 90.006 km²` (matching the production domain area, not `spval`) under mesh mode; gridcell aggregates are bit-identical between the mesh-mode test and the production-mode control. The workaround is OSBS-side; no upstream CTSM PR is planned.

---

## References

- Phase H source audit (this page's primary source): [`swenson/phases/H-lateral-flow.md`](https://github.com/cdevaneprugh/hpg-esm-tools/blob/main/swenson/phases/H-lateral-flow.md), Sections 7 and 8
- CTSM Issue #1432: [Area is not set for NUOPC single point cases](https://github.com/ESCOMP/CTSM/issues/1432) (open since 2021-07-20)
- Swenson, S. C., & Lawrence, D. M. (2025), [Development of a Global Representative Hillslope Data Set for Use in Earth System Models](https://doi.org/10.1029/2024MS004410), *Journal of Advances in Modeling Earth Systems*, 17 — hillslope-mode methodology (gridded-only validation; no single-point routing case)
- DiscussCESM thread, Johanna Teresa, Feb 2025: [Point-scale simulation with CTSM 5.3.11](https://bb.cgd.ucar.edu/cesm/threads/point-scale-simulation-with-ctsm5-3.11125/) — only public reference to Swenson's mesh-mode recommendation
- CTSM source: `git rev dc3aa5ddc` (audit target, CTSM 5.3.085)
