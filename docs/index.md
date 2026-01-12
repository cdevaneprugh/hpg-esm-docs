# CTSM on HiPerGator

Documentation for running the Community Terrestrial Systems Model (CTSM) on HiPerGator at the University of Florida.

## What is CTSM?

CTSM is the land component of the Community Earth System Model (CESM). It simulates terrestrial ecosystem processes including:

- Vegetation dynamics and carbon cycling
- Hydrology and water balance
- Energy balance and heat transfer
- Biogeochemistry (carbon-nitrogen cycling)

CTSM can be run standalone (without ocean/atmosphere coupling) for land-focused research.

## Quick Start

If you're new to this documentation:

1. **[Onboarding](onboarding.md)** - HiPerGator basics and environment setup
2. **[Prerequisites](installation/prerequisites.md)** - Required modules and shared libraries
3. **[Fork Setup](installation/fork-setup.md)** - Clone our CTSM fork
4. **[Case Workflow](running-ctsm/case-workflow.md)** - Create and run your first case

## Our Setup

We maintain forks of CTSM with HiPerGator-specific modifications:

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | CTSM with tool fixes |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | HiPerGator machine config |

Local installation: `/blue/gerber/cdevaneprugh/ctsm5.3`

## Documentation Structure

| Section | Content |
|---------|---------|
| **Installation** | Prerequisites, fork setup, building tools |
| **Running CTSM** | Case workflow, single-point runs, output configuration |
| **Research** | Group-specific research context (placeholders) |
| **Reference** | Fork modifications, external resources |
| **Archive** | Historical CESM documentation |

## Key Resources

- [CTSM GitHub Wiki](https://github.com/ESCOMP/ctsm/wiki) - Official development docs
- [CTSM Technical Note](https://escomp.github.io/CTSM/tech_note/index.html) - Model science documentation
- [CTSM User's Guide](https://escomp.github.io/CTSM/users_guide/overview/introduction.html) - Setup and running
- [DiscussCESM Forums](https://bb.cgd.ucar.edu/cesm/) - Community support
