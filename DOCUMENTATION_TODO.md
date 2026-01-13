# Documentation Todo List

Ongoing improvements to CTSM on HiPerGator documentation.

## Completed

- [x] Shift tone from "Gerber group docs" to "HiPerGator community resource"
- [x] Remove personal/group-specific paths, generalize with `<group>` placeholders
- [x] Reframe environment variables as recommendations
- [x] Add input data size warning
- [x] Create Quick Start page (clean installation guide)
- [x] Create CIME Configuration page (detailed config explanation)
- [x] Expand Fork Reference with detailed "why we fork" explanation
- [x] Add version notice for CTSM 5.3.085

## In Progress

- [ ] Review all pages for remaining Gerber-specific references
- [ ] Test all internal links work correctly

## Planned

### Content Expansion

- [ ] **Glossary**: Add terms (FATES, Compset, PFT, restart vs history, AD spinup)
- [ ] **Single-Point Runs**: Document multiple approaches
  - Direct NEON scripts
  - run_tower method
  - Manual subset workflow
- [ ] **Case Workflow**: Add branch case guide
- [ ] **History variables**: Comprehensive list as separate page

### Research Section

- [ ] **Hillslope Hydrology**: Flesh out placeholder with actual content
- [ ] **NEON Sites**: Flesh out placeholder
- [ ] **Gerber workflows**: Document group-specific single-point process (in Research section)

### Tools & Tips (Future Section)

- [ ] **Claude Code page** (tentative): How to use Claude for CTSM work
- [ ] **Analysis scripts**: Document hpg-esm-tools scripts
- [ ] **Recommended workflows**: Best practices accumulated over time

## Deferred

- [ ] Request HiPerGator-hosted shared inputdata (not documentation, but related)
- [ ] Add extensive comments to XML config files themselves
- [ ] Create "generic HiPerGator" config branch without group-specific QoS
- [ ] History variable comprehensive list (requires extracting from CLM source)

## Notes

- Core message: "Here's how to set this up yourself (and here's our fork if you want it)"
- Audience: Any HiPerGator user wanting to run CTSM, not just Gerber group
- Fork is a reference implementation, not the only way

---

*Last updated: 2025-01-13*
