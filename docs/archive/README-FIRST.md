# Earth Model Documentation

## Overview
The primary goal of this documentation is to serve as a guide for porting the [CESM](https://www.cesm.ucar.edu/), [CTSM](https://github.com/ESCOMP/CTSM), and [E3SM](https://e3sm.org/) Earth models to HiPerGator. I follow the official documentation for each Earth model, and add steps specific to HiPerGator as needed. Additionally, I'm including a section that can serve as an introduction to using Linux, which many students have limited exposure to. Ultimately, the goal at UF is to have several Earth models installed in some shared directory on HiPerGator. Until then this document can serve as a guide to do an install for your research group.

## Important Notes for Porting Earth Models
- **Start with `shared-utils.md`**: This file is essential for understanding directory structures and module dependencies before beginning the porting process.
- **Ensure the correct environment is loaded**: Use `module restore esm_gnu_env` before working with models to avoid dependency issues.
- **Follow the porting documentation step by step**: Missing configurations can cause failures, so adhere closely to the provided instructions.
- __Group specific setup:__ This documentation details how these Earth models were ported specifically for the Gerber research group. Your needs may be different.

## Directory Tree 

```
docs/
├── glossary.md                 # Glossary of terms related to Earth system models and HiPerGator
├── intro-to-earth-models.md    # Overview of Earth system models and their porting process
├── linux-overview.md           # Introduction to Linux and HiPerGator usage
├── porting/                    
│   ├── cesm-porting.md         # Steps to port CESM to HiPerGator
│   ├── ctsm-porting.md         # Steps to port CTSM to HiPerGator
│   ├── forking-ctsm.md	        # Steps to fork CTSM and associated submodules (not required)
│   ├── mksurfdata-fixes.md     # Steps to fix bugs in the CTSM/tools directory (not required)
│   └── shared-utils.md         # *READ FIRST* - Shared directory structure and module setup
└── usage/                      
    ├── cesm-usage.md           # Instructions for running CESM
    ├── ctsm-doc-comparison.md  # Clarifications on CTSM documentation inconsistencies
    └── ctsm-usage.md           # Instructions for running CTSM
```

## File Descriptions

### **General Documentation**
- **`glossary.md`**: Contains definitions of common terms used throughout the documentation.
- **`intro-to-earth-models.md`**: Provides background information on Earth system models, their structure, and functionality.
- **`linux-overview.md`**: Introduces Linux basics, command-line usage, and HiPerGator-specific commands.

### **Porting Guides (`porting/`)**
- **`shared-utils.md`** (*Read this first*)
  - Describes the shared directory structure for Earth models.
  - Explains required modules and module collections for proper setup.
  - Provides an overview of CIME configuration files.
- **`cesm-porting.md`**
  - Covers the steps to install and configure CESM on HiPerGator.
  - Includes downloading, setting up external components, and running regression tests.
- **`ctsm-porting.md`**
  - Details the process for setting up CTSM on HiPerGator.
  - Includes configuring the `mksurfdata` tool and setting up required dependencies.
- __`forking-ctsm.md`__
  - The process for forking CTSM. 
  - You can duplicate these steps if you wish to fork CTSM or our UF repo for your own modifications.
  - Not required reading if you are only interested in porting.
- __`mksurfdata-fixes.md`__
  - A record of all the bug fixes and modifications in `$CTSM/tools`.

### **Usage Guides (`usage/`)**
- **`cesm-usage.md`**
  - Instructions for creating, building, and running CESM cases.
  - Details about adjusting job resources, troubleshooting, and running single-point cases.
- **`ctsm-usage.md`**
  - Steps to configure and run CTSM simulations.
  - Details on setting up single-point or regional grid cases.
- **`ctsm-doc-comparison.md`**
  - Clarifies inconsistencies in CTSM documentation.
  - Summarizes missing tools and alternative approaches for dataset creation.

## Additional Resources
For further support, refer to the official documentation:
- **CESM:** [CESM Documentation](https://escomp.github.io/CESM/versions/cesm2.1/html/index.html)
- **CTSM:** [CTSM Documentation](https://escomp.github.io/ctsm-docs/versions/master/html/users_guide/index.html)
