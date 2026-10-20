# CTSM on HiPerGator

Source for the documentation site at **https://cdevaneprugh.github.io/hpg-esm-docs/**.

The site covers two things:

- **Running CTSM on HiPerGator** (University of Florida): environment setup, installing CTSM 5.3.085 from our fork, machine configuration, case workflow, single-point runs, spinup, and troubleshooting.
- **OSBS representative hillslopes**: methods and results for building a CTSM hillslope dataset for the Ordway-Swisher Biological Station from 1 m NEON LIDAR, following Swenson & Lawrence (2025), plus custom NEON atmospheric forcing.

## Related repositories

| Repository | Purpose |
|------------|---------|
| [cdevaneprugh/CTSM](https://github.com/cdevaneprugh/CTSM) (`uf-ctsm5.3.085`) | CTSM fork with HiPerGator build fixes |
| [cdevaneprugh/ccs_config_cesm](https://github.com/cdevaneprugh/ccs_config_cesm) (`uf-hipergator`) | HiPerGator machine configuration |
| [cdevaneprugh/hpg-esm-tools](https://github.com/cdevaneprugh/hpg-esm-tools) | Analysis scripts and the OSBS hillslope pipeline |

## Building locally

The site uses [MkDocs](https://www.mkdocs.org/) with the Material theme.

```bash
pip install mkdocs-material   # or use an environment that already has it
mkdocs serve                  # preview at http://127.0.0.1:8000
mkdocs build --strict         # check for broken links and warnings
mkdocs gh-deploy              # publish to the gh-pages branch
```

Pages live in `docs/`; navigation is set in `mkdocs.yml`. `docs/archive/` holds older notes kept for history and is excluded from the build.

## License

Content is licensed under [CC BY 4.0](LICENSE).
