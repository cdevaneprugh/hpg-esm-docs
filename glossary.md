## Glossary <a name="glossary"></a>

__CESM__ - Community Earth System Model. CESM is a fully-coupled, community maintained, global climate model that provides state-of-the-art computer simulations of the Earth's past, present, and future climate states.

__CIME__ - Common Infrastructure for Modeling the Earth (pronounced “SEAM”) provides a Case Control System for configuring, compiling and executing Earth system models, data and stub model components, a driver and associated tools and libraries.

__CLM__ - [Community Land Model](https://www.cesm.ucar.edu/models/clm). The land component used in CESM. It has the capability to model specific processes such as vegetation composition, heat transfer in soil, carbon-nitrogen cycling, canopy hydrology, and many more.

__CLM_USRDAT_NAME__

* Provides a way to enter your own datasets into the namelist. The files you create must be named with specific naming conventions outlined in [Creating your own single-point dataset](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/running-single-points/running-single-point-configurations.html#creating-your-own-singlepoint-dataset).

__cprnc__

* Tool used to compare two NetCDF files.
* Appears to have been previously located at `$CIMEROOT/tools/cprnc`, now installed in `/blue/gerber/earth_models/cprnc`. 

__CTSM__ - [Community Terrestrial Systems Model](https://github.com/ESCOMP/CTSM). An extension of CLM that is meant to be run as an alternative to CESM.

__E3SM__ - Energy Exascale Earth System Model. E3SM is the Department of Energy's climate model. Forked from CESM v1 and developed independently by the DOE, they describe E3SM as "an ongoing, state-of-the-science Earth system modeling, simulation, and prediction project that optimizes the use of DOE laboratory resources to meet the science needs of the nation and the mission needs of DOE."

__HPG__ - HiPerGator. The supercomputer used at UF. I'll use HPG and HiPerGator interchangeably throughout the documentation.

__GGCMI__ [Global Gridded Crop Model Inter comparisons](https://agmip.org/aggrid-ggcmi/)

* Group of crop modelers who provide gridded crop and climate data.
* Relevant to the scripts in `$CTSMROOT/tools/crop_calendars`

__Namelist__ (in the context of `mksurfdata`)

* Looks like different variables/parameters for the model.

__nuopc__ [The National Unified Operational Prediction Capability](https://earthsystemmodeling.org)

*  A consortium of Navy, NOAA, and Air Force modelers and their research partners. It aims to advance the weather prediction modeling systems used by meteorologists, mission planners, and decision makers. 
*  NUOPC partners are working toward a standard way of building models in order to make it easier to collaboratively build modeling systems. To this end, they have developed the NUOPC Layer that defines conventions and a set of generic components for building coupled models using the Earth System Modeling Framework (ESMF).

__PTS_MODE__

* A deprecated way to run single point simulations (still works on cesm 2.1.5).
* Okay for quick and dirty runs.
* Lacks restart capability.

__SCRIP__ [Spherical Coordinate Remapping and Interpolation Package](https://github.com/SCRIP-Project/SCRIP)

* Computes addresses and weights for remapping and interpolating fields between grids in spherical coordinates.
* Looks like they are used to create the surface datasets.
