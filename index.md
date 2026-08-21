---
layout: home
author_profile: true
excerpt: "The EU-funded AI4SoilHealth (AI4SH) collected soil data from field sampling and multiple analytical instruments across Europe. This site documents how to create, seed, load data and model soil properties using a postgreSQL database and Machine Learning using the Xspatula framework."
---

# AI4SH in-situ data handling and modelling with Xspatula

The EU-funded AI4SoilHealth (AI4SH) project postgreSQL database stores soil data collected from field sampling across Europe, analysed by wet laboratory, spectroscopy, and a range of other laboratory, home and field based sensors. To accommodate this data with [FAIR (Findability, Accessibility, Interoperability, and Reuse)][fair] principles the [Xspatula framework][xspatula] provides a comprehensive PostgreSQL database and a JSON-driven Python workflow for seeding a database, loading AI4SH data and then apply Machine Learning for modelling key soil properties from spectra and with various home and field based sensors.

## Prerequisites

The first two stages, setting up a postgreSQl database and register processes in the database, are introduced in more detail in the general introduction to the [Xspatula framework][setup_core_db_docs]:

1. **Database setup** — schemas and tables created; see [Setup core db][setup_core_db]
2. **Process registration** — all framework processes registered; see [Setup processes][setup_core_processes]

These two stages are also covered, with focus on creating the AI4SH database and its core processes, in this site.

To get started you need to clone or download the AI4SH data loading package from [GitHub][github]:

```bash
git clone https://github.com/xspatula/load_ai4sh_db
```

## Outline of the AI4SH postgreSQL database

The AI4SH postgres database contains 9 schemas:

- **utility** — support tables for general information used across schemas (default framework schema)
- **community** — organisations and users; all users logging into the system must be registered here (default framework schema)
- **process** — all processes defined for the AI4SH database (default framework schema)
- **landscape_utility** — reference tables for landscape classification
- **landscape** — landscape observations
- **observation_utility** — catalogues and reference data required for FAIR-compliant soil observations (units, methods, instruments, taxa, eDNA metabarcoding methods, etc.)
- **observation** — actual soil property data, organised through datasets, campaigns, samples and observations, including eDNA results

## Seeding the database

The AI4SH database is seeded in two stages:

1. **[Setup DB][setup_db]** — defines all schemas and tables using the Jupyter notebook `setup/setup_db.ipynb`
2. **[Setup processes][setup_process]** — registers all framework processes in the database using the notebook `setup/setup_processes.ipynb`

Both stages use the Xspatula JSON-driven workflow: a _scheme file_ points to a _job file_, which links to a _pilot file_ listing the individual _process files_ to execute. Alternatively, if you only have one _process_file_, you can point directly from the _job_file_ to this _process_file_ and skip a _pilot_file_. For a detailed explanation of this hierarchy, see the [Xspatula framework documentation][setup_core_db_docs_framework].

## Data loading overview

Data is loaded in a mandatory sequence — each stage depends on records from the previous one:

| Stage | Notebook | Pages |
|---|---|---|
| [Utility data][utility] | `load_ai4sh_utility_data.ipynb` | 7 |
| [Dataset metadata][dataset_meta] | `load_ai4sh_dataset_meta.ipynb` | 7 |
| [Sample data][sample] | `load_ai4sh_sample_data.ipynb` | 4 |
| [Wetlab data][wetlab] | `load_ai4sh_wetlab_data.ipynb` | 4 |
| [Spectral data][spectra] | `load_ai4sh_spectral_data.ipynb` | 13 |

Alternatively, all stages can be run from a [single notebook][all_data] (`load_ai4sh_data.ipynb`).

## Two-step pattern

Every data loading operation follows the same translate-then-manage pattern:

```
Excel source data
      ↓  [translate cell]
JSON process files
      ↓  [manage cell]
PostgreSQL database
```

The translate step reads Excel files and writes JSON process files to disk. The manage step reads those JSON files and executes them against the database. The two steps are decoupled so you can inspect the JSON before committing to the database.

## Acknowledgments and Funding

This work was done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views and opinions expressed are those of the authors only and do not necessarily reflect those of the European Union or the European Research Executive Agency._

[fair]: https://www.go-fair.org/fair-principles/
[setup_core_db_docs]: https://xspatula.github.io/setup_core_db_docs/
[setup_core_db_docs_framework]: https://xspatula.github.io/setup_core_db_docs/framework/
[setup_core_db]:https://xspatula.github.io/setup_core_db_docs/setup_db/
[setup_core_processes]:https://xspatula.github.io/setup_core_db_docs/setup_processes/
[setup_db]: /setup_db/
[setup_process]: /setup_process/
[utility]: /utility/
[dataset_meta]: /dataset_meta/
[sample]: /sample/
[wetlab]: /wetlab/
[spectra]: /spectra/
[all_data]: /all_data/
[github]: https://github.com/xspatula/load_ai4sh_db
[xspatula]: https://xspatula.github.io
