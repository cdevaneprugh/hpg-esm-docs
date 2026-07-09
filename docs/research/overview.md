# Research Overview

*This section documents research applications of CTSM on HiPerGator, focusing on wetlandscape hydrology and carbon dynamics.*

---

## DOE Project: Water and Carbon Dynamics of Coastal Plain Wetlandscapes

**Funding:** DOE-BER Terrestrial-Aquatic Interfaces program (2023-2026)

### The Problem

**Wetlandscapes** are low-relief mosaics of wetlands embedded in terrestrial uplands. The **Terrestrial-Aquatic Interface (TAI)** - the dynamic boundary between wet and dry areas - drives the bulk of carbon exchange, but current Earth System Models treat wetland extent as static.

Current ESM limitations (Fan et al. 2019):

- **1-D vertical hydrology** - no lateral flow between landscape positions
- **2-3m shallow soil** - misses deep moisture storage and Critical Zone processes
- **Free-draining** - instant drainage to rivers, no water table dynamics
- **No terrain structure** - flat slab with no ridges/valleys

### Research Goals

The grant's aspirational goals (full wetlandscape simulation) require first solving a fundamental problem: **individual wetlands aren't properly represented in ESMs**.

Current focus: Improve individual wetland representation in CTSM using hillslope hydrology to capture TAI dynamics.

### Technical Approach

Using CTSM's **hillslope hydrology** feature to represent within-gridcell topography:

- Multiple columns per gridcell at different elevations
- Lateral water flow between columns
- Different water table depths affect biogeochemistry
- Even subtle Florida topography (meters, not hundreds of meters) matters for TAI position

---

## Research Areas

### Hillslope Hydrology

CTSM's hillslope mode divides gridcells into multiple topographic positions with lateral water flow. This enables representation of:

- Ridge-to-valley water redistribution
- Aspect-dependent radiation (N, E, S, W facing slopes)
- Dynamic water table position
- Wetland-upland transitions

See [Hillslope Hydrology](hillslope.md) for technical details.

### NEON Site Simulations

NEON (National Ecological Observatory Network) provides high-quality observational data for model validation:

- Flux tower measurements (carbon, water, energy)
- Meteorological forcing data
- Soil and vegetation surveys
- High-resolution LIDAR topography

See [NEON Sites](neon-sites.md) for site-specific information.

---

## Key References

### Foundational Papers

| Paper | Topic | Relevance |
|-------|-------|-----------|
| Fan et al. 2019 | Hillslope Hydrology in ESMs | Scientific foundation for hillslope approach |
| Swenson & Lawrence 2025 | Global Hillslope Dataset | Methodology for creating hillslope parameters |

### Paper Summaries

Detailed summaries are available in `hpg-esm-tools/docs/papers/`:

- `DOE_Grant_Summary.md` - Project scope and actual focus
- `Fan_2019_Hillslope_Hydrology_ESM_Summary.md` - Scientific justification
- `Swenson_2025_Hillslope_Dataset_Summary.md` - Dataset methodology

---

## Test Site: OSBS

**Ordway-Swisher Biological Station** (North-central Florida)

| Attribute | Value |
|-----------|-------|
| Network | NEON/Ameriflux |
| Ecosystem | Sandhills with wetland depressions |
| Connectivity | Primarily groundwater, rarely surface connected |
| Topography | 1m resolution LIDAR available |
| Climate | R ~1450 mm/yr, PET ~1300 mm/yr |
| Wetland Coverage | ~19% |

OSBS represents a **low-relief wetlandscape** where lateral drainage improves soil aeration on slightly elevated mounds (position 3 in Fan 2019 Figure 6a).

---

## Current Status

**Phase:** Analysis and investigation. The 600-yr accelerated AD spinup with the custom OSBS hillslope file completed 2026-05-19. PI is investigating two open questions surfaced by the analysis: the absence of the anoxia-based TAI signal in decomposition output, and a bridge-zone water-table anomaly at flood-zone chain indices 3–6.

**Hillslope Data:** Custom NetCDF `hillslopes_osbs_production_c260505.nc` deployed 2026-05-05 (25 columns: 1 lake + 24 HAND bins on a single aspect). File is frozen during the PI investigation. Methodology on the [Swenson Implementation](../swenson/index.md) page.

**Reference Cases:** See [NEON Sites → Reference Cases](neon-sites.md#reference-cases).

---

*See [Hillslope Hydrology](hillslope.md) and [NEON Sites](neon-sites.md) for specific research areas.*
