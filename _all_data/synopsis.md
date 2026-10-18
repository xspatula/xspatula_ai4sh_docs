---
title: "Load All Data"
layout: single
sidebar:
  nav: "all_data"
  nav2: "loading_data"
excerpt: "The complete AI4SH loading pipeline in a single Jupyter notebook, insert_ai4sh_data.ipynb, divided into numbered blocks and cells that the rest of this site points to."
permalink: /all_data/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-18 08:00:00 +0200
---

All AI4SH data is loaded from one notebook:

```
./ai4sh/import_data/insert_ai4sh_data.ipynb
```

The notebook is divided into three numbered blocks, with one numbered cell per loading step. Every page on this site that describes a loading step names the block and cell number (e.g. **cell 3.3.2**) so you can find it directly in the notebook. The notebook runs top to bottom in dependency order, and a full run loads the complete database.

## Before the blocks

Two setup cells must run first, every session:

1. **Imports** — adds the project root to `sys.path` and imports `Initiate_process` and the custom `Run_process` from the `ai4sh` package.
2. **Scheme file** — `scheme_file = '../scheme_ai4sh.json'`; see [scheme file][scheme_file].

## Pipeline

{% capture notice-1 %}
| Cell | Name | `job_file` / `process_file` | Documentation |
|---|---|---|---|
| **1** | **Utilities** | | [Load utility data]({{ site.baseurl }}/utility/) |
| 1.1 | Load general utilities | `utility/job_insert_general_utility.json` | [Insert utility data]({{ site.baseurl }}/utility/insert_utility_data/) |
| 1.2 | Load observation utilities | `utility/job_insert_observation_utility.json` | [Insert utility data]({{ site.baseurl }}/utility/insert_utility_data/) |
| 1.3 | Load observation utilities with inheritance | `utility/job_insert_observation_utility_inherit.json` | [Insert utility data]({{ site.baseurl }}/utility/insert_utility_data/) |
| 1.4 | Load landscape utility | `utility/job_insert_landscape_utility.json` | [Insert utility data]({{ site.baseurl }}/utility/insert_utility_data/) |
| **2** | **Dataset metadata** | | [Load dataset metadata]({{ site.baseurl }}/dataset_meta/) |
| 2.1 | Load dataset metadata | `dataset_metadata/job_insert_dataset_metadata.json` | [Insert dataset metadata]({{ site.baseurl }}/dataset_meta/insert_dataset_meta/) |
| **3** | **AI4SH data** | | |
| 3.1 | Load AI4SH geolocations | `sample/insert_process/insert_geotag_sample.json` | [Insert sample data]({{ site.baseurl }}/sample/insert_sample_data/) |
| **3.2** | **Laboratory, enzymatic, infiltration, microbiometer and aggregate stability** | | [Load observation data]({{ site.baseurl }}/observation/) |
| 3.2.1 | Load Agrolab (wetlab) observation data | `wetlab/agrolab/insert_process/insert_agrolab_observation_log+observation.json` | [Insert Agrolab data]({{ site.baseurl }}/observation/insert_agrolab/) |
| 3.2.2 | Load Veltia (wetlab) observation data | not yet completed | — |
| 3.2.3 | Load iBEC (wetlab) observation data | not yet completed | — |
| 3.2.4 | Load SLU (wetlab) observation data | not yet completed | — |
| 3.2.5 | Load Neiker (wetlab) observation data | not yet completed | — |
| 3.2.6 | Load Digit Soil enzymatic observation data | `digit_soil/insert_process/insert_digitsoil_observation_log+observation.json` | [Insert Digit Soil data]({{ site.baseurl }}/observation/insert_digitsoil/) |
| 3.2.7 | Load BEST infiltration observation data | `infiltration/insert_process/insert_infiltration_observation_log+observation.json` | [Insert BEST infiltration data]({{ site.baseurl }}/observation/insert_best_infiltration/) |
| 3.2.8 | Load Microbiometer observation data | `microbiometer/insert_process/insert_microbiometer_observation_log+observation.json` | [Insert Microbiometer data]({{ site.baseurl }}/observation/insert_microbiometer/) |
| 3.2.9 | Load aggregate stability data | `aggstab/insert_process/insert_aggstab_observation_log+observation.json` | [Insert aggregate stability data]({{ site.baseurl }}/observation/insert_aggregate_stability/) |
| **3.3** | **Spectral data** | | [Load spectral data]({{ site.baseurl }}/spectra/) |
| 3.3.1 | Load FOSS DS2500 spectrometer | `DS2500_spectra/job_DS2500_spectrometer.json` | [FOSS DS2500 spectrometer]({{ site.baseurl }}/spectra/manage_foss_ds2500_spectrometer/) |
| 3.3.2 | Load FOSS DS2500 observation log | `DS2500_spectra/job_DS2500_observation_log.json` | [FOSS DS2500 observation log]({{ site.baseurl }}/spectra/manage_foss_ds2500_obs_log/) |
| 3.3.3 | Load FOSS DS2500 spectra observations | `DS2500_spectra/job_DS2500_observation.json` | [FOSS DS2500 observation]({{ site.baseurl }}/spectra/manage_foss_ds2500_obs/) |
| 3.3.4 | Load Neospectra spectrometer | `neospectra/job_neospectra_spectrometer.json` | [Neospectra spectrometer]({{ site.baseurl }}/spectra/manage_neospectra_spectrometer/) |
| 3.3.5 | Load Neospectra observation log | `neospectra/job_neospectra_observation_log.json` | [Neospectra observation log]({{ site.baseurl }}/spectra/manage_neospectra_obs_log/) |
| 3.3.6 | Load Neospectra observations | `neospectra/job_neospectra_observation.json` | [Neospectra observation]({{ site.baseurl }}/spectra/manage_neospectra_obs/) |
| 3.3.7 | Load Bruker Invenio S FTIR spectrometer | `FTIR_spectra/job_ftir_spectrometer.json` | [FTIR spectrometer]({{ site.baseurl }}/spectra/manage_ftir_spectrometer/) |
| 3.3.8 | Load Bruker Invenio S FTIR observation log | `FTIR_spectra/job_ftir_observation_log.json` | [FTIR observation log]({{ site.baseurl }}/spectra/manage_ftir_obs_log/) |
| 3.3.9 | Load Bruker Invenio S FTIR observations | `FTIR_spectra/job_ftir_observation.json` | [FTIR observation]({{ site.baseurl }}/spectra/manage_ftir_obs/) |
| 3.3.10 | Load FOSS Micral (LIBS) spectrometer | `LIBS_spectra/job_libs_spectrometer.json` | [LIBS spectrometer]({{ site.baseurl }}/spectra/manage_libs_spectrometer/) |
| 3.3.11 | Load FOSS Micral observation log | `LIBS_spectra/job_libs_observation_log.json` | [LIBS observation log]({{ site.baseurl }}/spectra/manage_libs_obs_log/) |
| 3.3.12 | Load FOSS Micral observations | `LIBS_spectra/job_libs_observation.json` | [LIBS observation]({{ site.baseurl }}/spectra/manage_libs_obs/) |
| **3.4** | **eDNA data** | | [Load eDNA data]({{ site.baseurl }}/edna/) |
| 3.4.1 | Load prokaryote taxa (SILVA) | `eDNA/insert_process/insert_taxon_prokaryotes.json` | [Insert taxa]({{ site.baseurl }}/edna/insert_taxon/) |
| 3.4.2 | Load fungal taxa (UNITE) | `eDNA/insert_process/insert_taxon_fungi.json` | [Insert taxa]({{ site.baseurl }}/edna/insert_taxon/) |
| 3.4.3 | Load eDNA observation log | `eDNA/insert_process/insert_eDNA_observation_log.json` | [Insert eDNA observations]({{ site.baseurl }}/edna/insert_edna_observation/) |
| 3.4.4 | Load eDNA summary observation | `eDNA/insert_process/insert_eDNA_observation.json` | [Insert eDNA observations]({{ site.baseurl }}/edna/insert_edna_observation/) |
| 3.4.5 | Load eDNA ASV prokaryote abundances | `eDNA/insert_process/insert_asv_prokaryotes.json` | [Insert ASVs]({{ site.baseurl }}/edna/insert_edna_asv/) |
| 3.4.6 | Load eDNA ASV fungal abundances | `eDNA/insert_process/insert_asv_fungi.json` | [Insert ASVs]({{ site.baseurl }}/edna/insert_edna_asv/) |
{% endcapture %}

<div class="notice">{{ notice-1 | markdownify }}</div>

All paths are relative to `./ai4sh/import_data/`. Cells 1.x and 2.1 call a `job_file` (a pilot file lists several process files); cells 3.1, 3.2.x and 3.4.x call a single `process_file`; the spectral cells 3.3.x call a `job_file` again.

Blocks 1 to 3.2 and block 3.4 use the single-step `insert_tabular_data` route (see [Single-step vs dual-step][insert_vs_translate]). The spectral cells (3.3.x) call `manage_*` processes directly — the spectra are converted to process files by a separate script, so there is no spreadsheet to insert from.

The AI4SH observation data (blocks 3.x) is not included in the `xspatula_ai4sh` repo; you have to request access to it. The utility data and dataset metadata (blocks 1 and 2) are shipped with the repo.

## Dependency order

- Block 1 must run before anything else: every later record references utility catalogues.
- Block 2 must run before block 3: every sample belongs to a sampling log.
- Cell 3.1 must run before any other cell in block 3: observations are only accepted for samples registered with a geolocation.
- Within each instrument in 3.3: spectrometer → observation log → observations.
- Within 3.4: taxa (3.4.1–3.4.2) must be loaded before the ASVs (3.4.5–3.4.6), and the observations (3.4.3–3.4.4) before the ASVs.

## Running only part of the notebook

Every loading cell starts with the line

```python
#%%script false --no-raise-error
```

As shipped the line is commented out (`#`), so the cell runs. To switch a cell **off**, remove the `#`: the cell magic then skips the cell without raising an error. Some cells (e.g. the spectral cells and 3.2.7) are shipped switched off.

Use this to:

- skip block 1 and 2 when the catalogues and metadata are already in the database and you only add new observations;
- load a single data type, e.g. only the eDNA cells 3.4.1–3.4.6;
- skip cells for data you don't have access to.

All cells are INSERT-only, so re-running a cell that has already run inserts nothing and changes nothing.

## The 2-step notebooks

The earlier per-stage notebooks, including the 2-step translate-then-manage notebooks, have been moved to:

```
./ai4sh/import_data/more_notebooks/
```

They are kept for the cases where you need to `UPDATE` existing records or inspect the generated JSON before it is applied. The pages titled _Translate …_ and _Manage …_ under each loading stage describe those notebooks. For a normal load, use `insert_ai4sh_data.ipynb`.

## Dump once, restore in minutes

Setting up the database and running the full notebook is slow. Once the database is complete, write it to a single dump file with `xspatula_ai4sh/setup/dump_restore_db.ipynb`. Restoring the dump later, on the same machine or another one, takes minutes and replaces re-running [Setup DB][setup_db], [Setup processes][setup_process] and the whole import.

Set `dump_path` under `postgresdb` in `setup/zzz/scheme_local_setup.json`:

```json
"dump_path": "~/xspatula/db_dump/xspatula_ai4sh.dump"
```

Without `dump_path` the dump is written to `~/xspatula/db_dump/ai4sh.dump`. Keep the file outside the project directory; it is large and holds the user records.

- ⚠️ A restore drops an existing `ai4sh` database (after confirmation) and replaces it with the dump.
- Dump again after every change to the import. A restore is only as current as its dump.
- The dump restores the whole database only. AI4SH data cannot be restored on top of another project's database (e.g. LUCAS): the `SERIAL` ids collide. To have both, load them into the same database as described on the [home page][home_lucas], then dump that database.

The full instructions, including command line alternatives, are on the core docs site: [Dump and restore][dump_restore].

## AI4SH data: partners only

The AI4SH data has no licence yet. It has not been reviewed and verified, and until it is, it may only be shared within the [AI4SoilHealth][ai4sh] project partners. This applies to the source files and to anything that contains the data:

- **Do not share an AI4SH dump** outside the AI4SoilHealth partners. The core docs describe [sharing a premade database][dump_restore_share]; for AI4SH that option is limited to the partners.
- The same goes for the Excel source files, the generated JSON process files and any files extracted from the database for machine learning.
- A dump also holds the application users in `community.user`, with e-mail addresses and password hashes. Remove or replace personal users before you give a dump to another partner.

[setup_db]: {{ site.baseurl }}/setup_db/
[setup_process]: {{ site.baseurl }}/setup_process/
[home_lucas]: {{ site.baseurl }}/#including-lucas-data
[dump_restore]: https://xspatula.github.io/xspatula_core_docs/setup_db/dump_restore/
[dump_restore_share]: https://xspatula.github.io/xspatula_core_docs/setup_db/dump_restore/#sharing-a-premade-database
[ai4sh]: https://ai4soilhealth.eu
[scheme_file]: https://xspatula.github.io/xspatula_core_docs/framework/scheme_file/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[Load utility data]: {{ site.baseurl }}/utility/
[Insert utility data]: {{ site.baseurl }}/utility/insert_utility_data/
[Load dataset metadata]: {{ site.baseurl }}/dataset_meta/
[Insert dataset metadata]: {{ site.baseurl }}/dataset_meta/insert_dataset_meta/
[Insert sample data]: {{ site.baseurl }}/sample/insert_sample_data/
[Insert Agrolab data]: {{ site.baseurl }}/observation/insert_agrolab/
[Load spectral data]: {{ site.baseurl }}/spectra/
[FOSS DS2500 spectrometer]: {{ site.baseurl }}/spectra/manage_foss_ds2500_spectrometer/
[FOSS DS2500 observation log]: {{ site.baseurl }}/spectra/manage_foss_ds2500_obs_log/
[FOSS DS2500 observation]: {{ site.baseurl }}/spectra/manage_foss_ds2500_obs/
[Neospectra spectrometer]: {{ site.baseurl }}/spectra/manage_neospectra_spectrometer/
[Neospectra observation log]: {{ site.baseurl }}/spectra/manage_neospectra_obs_log/
[Neospectra observation]: {{ site.baseurl }}/spectra/manage_neospectra_obs/
[FTIR spectrometer]: {{ site.baseurl }}/spectra/manage_ftir_spectrometer/
[FTIR observation log]: {{ site.baseurl }}/spectra/manage_ftir_obs_log/
[FTIR observation]: {{ site.baseurl }}/spectra/manage_ftir_obs/
[LIBS spectrometer]: {{ site.baseurl }}/spectra/manage_libs_spectrometer/
[LIBS observation log]: {{ site.baseurl }}/spectra/manage_libs_obs_log/
[LIBS observation]: {{ site.baseurl }}/spectra/manage_libs_obs/
[Load eDNA data]: {{ site.baseurl }}/edna/
[Insert taxa]: {{ site.baseurl }}/edna/insert_taxon/
[Insert eDNA observations]: {{ site.baseurl }}/edna/insert_edna_observation/
[Insert ASVs]: {{ site.baseurl }}/edna/insert_edna_asv/
