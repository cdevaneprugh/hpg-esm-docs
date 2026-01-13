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
2. **[Quick Start](installation/quickstart.md)** - Get CTSM running step by step
3. **[Case Workflow](running-ctsm/case-workflow.md)** - Detailed case configuration

For deeper understanding, see [Prerequisites](installation/prerequisites.md) and [Fork Reference](installation/fork-setup.md).

## Reference Fork

We maintain forks of CTSM with HiPerGator-specific modifications. You can use these as a starting point for your own installation, or fork them to make additional changes.

| Repository | Branch | Purpose |
|------------|--------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) | `uf-ctsm5.3.085` | CTSM with tool fixes |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) | `uf-hipergator` | HiPerGator machine config |

!!! note "Version Notice"
    These forks are based on CTSM 5.3.085. Other versions may require different fixes.

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
- [CIME Documentation](https://esmci.github.io/cime/versions/master/html/index.html) - Build and case management
- [DiscussCESM Forums](https://bb.cgd.ucar.edu/cesm/) - Community support
