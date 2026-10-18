---
layout: home
author_profile: true
excerpt: "The EU-funded AI4SoilHealth (AI4SH) project (2022-2026) collected soil data from field sampling and multiple analytical instruments across Europe. This site documents how to create, seed, load data and model soil properties using a postgreSQL database and Machine Learning using the Xspatula framework."
---

# AI4SH in-situ data handling and modelling with Xspatula

The EU-funded [AI4SoilHealth (AI4SH)][ai4sh] project (2022-2026) collected and analysed soil data from field sampling across Europe. Different in-situ, home and laboratory tiered methods were applied for analysing the samples. To accommodate this data with [FAIR (Findability, Accessibility, Interoperability, and Reuse)][fair] principles the [Xspatula framework][xspatula] provides a comprehensive PostgreSQL database and a JSON-driven Python workflow for seeding a database, loading AI4SH data and then apply Machine Learning for modelling key soil properties from traditional laboratory analysis, spectra and various home and field based sensors. This site documents how to create, seed, load data and model soil properties using a postgreSQL database and Machine Learning using the Xspatula framework.

## Prerequisites

The first two stages, setting up a postgreSQl database and register processes in the database, are introduced in more detail in the general introduction to the [Xspatula framework][setup_core_db_docs]:

1. **Database setup** — schemas and tables created; see [Setup core db][setup_core_db]
2. **Process registration** — all framework processes registered; see [Setup processes][setup_core_processes]

These two stages are also covered, with focus on creating the AI4SH database and its core processes, in this site.

To get started you need to clone or download the AI4SH framework package from [GitHub][github]:

```bash
git clone https://github.com/xspatula/xspatula_ai4sh
```

## Outline of the AI4SH postgreSQL database

The AI4SH postgres database contains 10 schemas:

- **utility** — support tables for general information used across schemas (default framework schema)
- **community** — organisations and users; all users logging into the system must be registered here (default framework schema)
- **process** — all processes defined for the AI4SH database (default framework schema)
- **audit** — logs who changed what, when, in every audited table (default framework schema)
- **landscape_utility** — reference tables for landscape classification
- **landscape** — landscape observations
- **observation_utility** — catalogues and reference data required for FAIR-compliant soil observations (units, methods, instruments, software, eDNA primer pairs, lab protocols, method pipelines, etc.)
- **observation** — actual soil property data, organised through datasets, campaigns, samples and observations, including eDNA ASVs and abundances
- **organism_utility** — the taxonomic backbone: taxon tree, ranks, status, functions and taxonomy reference databases for environmental DNA (e.g. SILVA, UNITE)
- **organism** — reserved for organism-level observations (no tables loaded yet)

## Seeding the database

The AI4SH database is seeded in two stages:

1. **[Setup DB][setup_db]** — defines all schemas and tables using the Jupyter notebook `setup/setup_db.ipynb`
2. **[Setup processes][setup_process]** — registers all framework processes in the database using the notebook `setup/setup_processes.ipynb`

Both stages use the Xspatula JSON-driven workflow: a _scheme file_ points to a _job file_, which links to a _pilot file_ listing the individual _process files_ to execute. Alternatively, if you only have one _process_file_, you can point directly from the _job_file_ to this _process_file_ and skip a _pilot_file_. For a detailed explanation of this hierarchy, see the [Xspatula framework documentation][setup_core_db_docs_framework].

## Data loading overview

Data is loaded in a mandatory sequence — each stage depends on records from the previous one. All stages are loaded from a single notebook, `./ai4sh/import_data/insert_ai4sh_data.ipynb`, divided into numbered blocks and cells (see [Load all data][all_data]):

{% capture notice-1 %}
| Stage | Notebook block | Cells | Pages |
|---|---|---|---|
| [Utility data]({{ site.baseurl }}/utility/) | 1. Utilities | 1.1–1.4 | 8 |
| [Dataset metadata]({{ site.baseurl }}/dataset_meta/) | 2. Dataset metadata | 2.1 | 8 |
| [Sample data]({{ site.baseurl }}/sample/) | 3. Load AI4SH data | 3.1 | 5 |
| [Observation data]({{ site.baseurl }}/observation/) | 3.2 Laboratory and field observations | 3.2.1, 3.2.6–3.2.9 (3.2.2–3.2.5 pending) | 9 |
| [Spectral data]({{ site.baseurl }}/spectra/) | 3.3 AI4SH spectral data | 3.3.1–3.3.12 | 13 |
| [eDNA metabarcoding]({{ site.baseurl }}/edna/) | 3.4 AI4SH eDNA data | 3.4.1–3.4.6 | 7 |
{% endcapture %}

<div class="notice">{{ notice-1 | markdownify }}</div>

The notebook uses the single-step route throughout (spectra call `manage_*` directly). The 2-step translate-then-manage notebooks, mainly for updating existing records, are kept in `./ai4sh/import_data/more_notebooks/`.

When everything is loaded, [dump the database][ai4sh_dump] once. Restoring a dump takes minutes and replaces the setup and the import ([Dump and restore][dump_restore] on the core docs site).

## Including LUCAS data

To have LUCAS and AI4SH data side by side, load both projects into **the same database**:

1. Set up the database and register processes from this package (`xspatula_ai4sh`), as described under [Seeding the database](#seeding-the-database).
2. Clone the LUCAS project next to it:

   ```bash
   git clone https://github.com/xspatula/xspatula_lucas
   ```
3. In `xspatula_lucas/setup/zzz/`, replace `scheme_local_setup.json` with a copy of the  same file from the `xspatula_ai4sh` project; make sure that no process is set to overwrite or delete, and run the `setup_db` to create the LUCAS specific database entries.
4. Repeat the same procedure as in 3. for setup_processes to insert LUCAS specific processing.
5. Download the LUCAS data you want to use and run the Command Line Interface script (s) under `lucas/prepare_lucas_data`.
6. In `./lucas/scheme_lucas.json`, set `user_project` to a user already registered in the AI4SH database.
7. Load the utility, dataset metadata and LUCAS 2009/2015 campaigns following [xspatula_lucas_docs][xspatula_lucas_docs].

LUCAS and AI4SH records are then separated by their dataset (`lucas` vs `ai4sh`) and share all utility catalogues (units, indicators, methods, taxa), which is what makes cross-dataset modelling possible.

You can also do it the other way around, first setting up and loading LUCAS and then adding AI4SH to the LUCAS framework.

## Single-step vs 2-step

Every data-loading operation that starts from a spreadsheet can go through the database one of
two ways — a single-step `insert_tabular_data` route (reads Excel/CSV and inserts it
immediately, INSERT-only) or a 2-step translate-then-manage route (supports `UPDATE`
and lets you hand-inspect the generated JSON first):

```
Excel source data                          Excel source data
      ↓  [insert_tabular_data]                    ↓  [translate cell]
PostgreSQL database                        JSON process files
                                                  ↓  [manage cell]
                                            PostgreSQL database
```

See [Single-step vs dual-step][insert_vs_translate] for the full comparison and which one to
use when.

## Data access

The AI4SH data has no licence yet. It has not been reviewed and verified, and may at present only be shared within the [AI4SoilHealth][ai4sh] project partners. This covers the source files, a database dump holding the data, and any files extracted from it. See [AI4SH data: partners only][ai4sh_partners].

{% capture notice-3 %}
## Acknowledgments and Funding

This work was done as part of the AI4SoilHealth project, funded by the European Union's Horizon Europe Research and Innovation Programme under Grant Agreement No. 101086179.

_Funded by the European Union. The views and opinions expressed are those of the authors only and do not necessarily reflect those of the European Union or the European Research Executive Agency._
{% endcapture %}

<div class="notice--info">{{ notice-3 | markdownify }}</div>

[fair]: https://www.go-fair.org/fair-principles/
[ai4sh]: https://ai4soilhealth.eu
[ai4sh_dump]: {{ site.baseurl }}/all_data/#dump-once-restore-in-minutes
[ai4sh_partners]: {{ site.baseurl }}/all_data/#ai4sh-data-partners-only
[dump_restore]: https://xspatula.github.io/xspatula_core_docs/setup_db/dump_restore/
[setup_core_db_docs]: https://xspatula.github.io/xspatula_core_docs/
[setup_core_db_docs_framework]: https://xspatula.github.io/xspatula_core_docs/framework/
[setup_core_db]:https://xspatula.github.io/xspatula_core_docs/setup_db/
[setup_core_processes]:https://xspatula.github.io/xspatula_core_docs/setup_processes/
[setup_db]: {{ site.baseurl }}/setup_db/
[setup_process]: {{ site.baseurl }}/setup_process/
[utility]: {{ site.baseurl }}/utility/
[dataset_meta]: {{ site.baseurl }}/dataset_meta/
[sample]: {{ site.baseurl }}/sample/
[observation]: {{ site.baseurl }}/observation/
[spectra]: {{ site.baseurl }}/spectra/
[edna]: {{ site.baseurl }}/edna/
[all_data]: {{ site.baseurl }}/all_data/
[github]: https://github.com/xspatula/xspatula_ai4sh
[xspatula]: https://xspatula.github.io
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[xspatula_lucas_docs]: https://xspatula.github.io/xspatula_lucas_docs/
