# Documentation Todo List

Ongoing improvements to CTSM on HiPerGator documentation.

## Completed

**Framing and structural**
- [x] Shift tone from "Gerber group docs" to "HiPerGator community resource"
- [x] Remove personal/group-specific paths, generalize with `<group>` placeholders
- [x] Reframe environment variables as recommendations
- [x] Add input data size warning
- [x] Add version notice for CTSM 5.3.085
- [x] Review all pages for remaining Gerber-specific references
- [x] Test all internal links work correctly

**Installation & Running CTSM**
- [x] Quick Start page (clean installation guide)
- [x] CIME Configuration page (detailed config explanation)
- [x] Fork Reference expanded with detailed "why we fork" explanation
- [x] Case Workflow: branch case guide (run types section)

**Getting Started**
- [x] Glossary (FATES, Compset, PFT, restart vs history, AD spinup)

**Research section**
- [x] Research Overview (DOE project context, scientific goals)
- [x] Hillslope Hydrology (fleshed out from placeholder)
- [x] NEON Sites (fleshed out from placeholder)

**Swenson Implementation section**
- [x] Seven pages: index, pysheds-porting, merit-validation, osbs-implementation, hand-binning-and-lake-column, lateral-flow-and-routing, dataset-comparison
- [x] Production plots embedded in HAND binning + lake column page

**2026-06-27 audit + refresh**
- [x] Fix stale Research-section content (phase framing, hillslope-data status, reference-cases table)
- [x] Refresh case inventory on `research/neon-sites.md` (operative + secondary + reference cases)
- [x] Correct mesh-tooling reference on `swenson/lateral-flow-and-routing.md` (`mesh_maker.py` aborts on 1×1; `make_osbs_scrip.py` is the actual tool)
- [x] Phase F results summary in `swenson/index.md` Current Status
- [x] Development History table extended to 2026-06

## Planned

### Content expansion

- [ ] **Single-Point Runs**: multiple approaches (direct NEON scripts, `run_tower`, manual subset workflow). Per PI: `run_tower` pre-built forcing is insufficient; manual NCAR-NEON pipeline is the required path.
- [ ] **History variables**: comprehensive list as a separate page (requires extraction from CLM source)
- [ ] **Phase F results page** (`swenson/phase-f-results.md`) — pending PI verdict on TAI absence + bridge-zone anomaly. Embed TOTECOSYSC (convergence), h1a_ZWT (bridge-zone + lake), h1a_H2OSFC (lake dynamics).
- [ ] **Dataset comparison plots**: generate `production_elevation_width.png` and `production_col_areas.png` from the production NetCDF; replace TODO placeholders in `swenson/dataset-comparison.md`

### Repo hygiene

- [ ] **CLAUDE.md** at docs-repo root (Claude Code orientation for contributors)
- [ ] **README.md** at docs-repo root (GitHub landing page: site URL, build/serve commands, contributor pointers)

### Tools & Tips (future section)

- [ ] Analysis scripts inventory (hpg-esm-tools)
- [ ] Recommended workflows / best practices accumulated over time

## Deferred

- [ ] Request HiPerGator-hosted shared inputdata (not documentation, but related)
- [ ] Add extensive comments to XML config files themselves
- [ ] Create "generic HiPerGator" config branch without group-specific QoS

## Notes

- Core message: *"Here's how to set this up yourself (and here's our fork if you want it)."*
- Audience: Any HiPerGator user wanting to run CTSM, not just Gerber group.
- Installation and Running CTSM sections use `<group>` placeholders (external-audience convention). Research and Swenson Implementation sections use internal case-name labels (`osbs.swenson.spinup`, etc.) as descriptive identifiers for the project-team audience.
- Fork is a reference implementation, not the only way.

---

*Last updated: 2026-06-27*
