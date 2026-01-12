# An Overview of Earth System Models<a name="esm_title"></a>

## Community Earth System Model (CESM)

**Created by**: The National Center for Atmospheric Research (NCAR) with contributions from the scientific community.

### History

- Developed from the **Community Climate System Model (CCSM)** in the 1990s.
- CESM v1.0 was released in 2010 as an improved coupled climate model.
- Current versions, including **CESM2.x**, support high-resolution simulations and enhanced climate interactions.

### Technical Overview

- CESM is a **fully coupled, global climate model** that simulates the interactions of the Earth’s atmosphere, land, ocean, and ice systems.
- Built on **Common Infrastructure for Modeling the Earth (CIME)**, providing tools for configuring, building, and executing model cases.
- Utilizes a **component-based architecture** with interchangeable modules for different Earth system components (atmosphere, land, ocean, etc.).
- Supports **various grid resolutions** and scientific use cases, such as paleoclimate studies, future climate projections, and single-point land simulations.

## Community Terrestrial Systems Model (CTSM)

**Created by**: NCAR, evolving from the **Community Land Model (CLM)**.

### History

- Initially, land modeling within CESM was handled by **CLM**, a component developed since the early 2000s.
- **CTSM** was introduced as an enhanced land model that can operate independently or within CESM.
- **CTSM5** is the latest version, widely used in land-atmosphere interactions and biogeochemical modeling.

### Technical Overview

- CTSM extends **CLM** and serves as an **alternative land model** within CESM or as a standalone model.
- Simulates **vegetation dynamics, carbon-nitrogen cycling, and land hydrology** processes.
- Employs **CIME infrastructure** for model configuration and case management.
- Can be used for **global, regional, or single-point simulations**, depending on research needs.
- Requires custom-built **surface datasets** and **mapping files** to translate model grids into usable land surface information.

## Energy Exascale Earth System Model (E3SM)

**Created by**: The U.S. Department of Energy (DOE).

### History

- Developed as a **fork of CESM v1** by DOE laboratories, including **LLNL, ANL, ORNL, and PNNL**.
- Launched as an independent model focused on **high-performance computing (HPC) and energy-related climate impacts**.
- Optimized for DOE’s **exascale computing resources**.

### Technical Overview

- Similar to CESM, E3SM is a **fully coupled Earth system model**, but optimized for **higher resolution and DOE-specific climate objectives**.
- Features **enhanced land-atmosphere coupling** and **regional refinement capabilities**.
- Utilizes **CIME infrastructure** but diverges in **model physics and component designs**.
- Supports targeted simulations for **climate change impacts on energy production, water availability, and extreme weather events**.

## How Coupled Climate and Earth Models Work

Coupled climate and Earth system models integrate multiple interacting components to simulate complex planetary systems over time. These models link the atmosphere, ocean, land, and ice to understand climate dynamics and make future predictions.

### Key Components

- **Atmosphere**: Simulates temperature, wind, humidity, and radiation through atmospheric circulation models.
- **Ocean**: Models currents, salinity, and temperature variations to study oceanic interactions with climate.
- **Land**: Includes vegetation, soil moisture, and carbon cycle processes to track terrestrial climate effects.
- **Ice**: Simulates ice sheets and sea ice to examine polar climate changes.

### Process Coupling

- Advanced numerical methods solve fluid dynamics and thermodynamic equations.
- Exchanges between different components occur through fluxes of heat, moisture, and momentum.
- High-performance computing (HPC) enables large-scale simulations with fine spatial and temporal resolutions.

### Applications

- Predicting future climate scenarios based on different greenhouse gas emission pathways.
- Studying extreme weather events, ocean circulation shifts, and ecosystem responses to climate change.
- Supporting decision-making in **climate policy, energy planning, and disaster management**.

For further details, refer to:

- [DOE Explanation of Earth System Models](https://www.energy.gov/science/doe-explainsearth-system-and-climate-models)
- [Nature Climate Modeling Overview](https://www.nature.com/scitable/knowledge/library/studying-and-projecting-climate-change-with-earth-103087065/)

## Shared Technical Aspects

- CIME (Common Infrastructure for Modeling the Earth)
  - A **framework** used by all three models for configuring, compiling, and running simulations.
  - Provides **machine-specific configurations**, job submission tools, and workflow management.
  - Used to adapt the models for **various HPC environments**, such as **HiPerGator**.
