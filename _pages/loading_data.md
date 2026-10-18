---
title: "Loading AI4SH Data"
layout: single
sidebar:
  nav: "loading_data"
excerpt: "Overview of the seven data-loading stages for the AI4SoilHealth database. Data must be loaded in the order shown — each stage depends on records from the previous one."
permalink: /loading_data/
author_profile: false
date: 2026-06-15 08:00:00 +0200
last_modified_at: 2026-10-18 08:00:00 +0200
---

**Shortcut with a premade database:** if an AI4SoilHealth partner gives you a dump of a loaded AI4SH database, restore it instead of running the load data notebook (`insert_ai4sh_data.ipynb`). See [Dump once, restore in minutes][ai4sh_dump]. The AI4SH data may only be shared within the project partners.
{: .notice--info}

Loading AI4SH data into the database follows a strict dependency order. Each stage builds on records inserted by the previous one. The seven stages below must be completed in sequence when populating the database from scratch.

All stages use the same [scheme file][scheme_file] to define the user and general settings:

`./ai4sh/scheme_ai4sh.json`

The scheme file object user_id must be a registered user in the postgreSQL database and the password set either explicitly (password: _"password"_) or in a .netrc file (user_netrc_id: _"user_netrc_id"_).

## Single-step vs dual-step

All stages are loaded from one notebook, divided into numbered blocks and cells:

```
./ai4sh/import_data/insert_ai4sh_data.ipynb
```

The notebook uses the single-step route (reads a spreadsheet and inserts it in one step,
INSERT-only, safe to re-run). Spectral data is prepared for direct loading via JSON prepared process files. If you want to edit any data before inserting to the database, 2-step notebooks
(translate then manage — needed for `UPDATE`s or to hand-inspect the generated JSON first) are
kept in `./ai4sh/import_data/more_notebooks/`. See [Single-step vs dual-step][insert_vs_translate]
for the full comparison, and [Load all data][All data in one go] for the complete cell list.

## Loading stages

### 1. [Utility data]

Utility data contains the controlled vocabularies and reference catalogues that all observation data depends on — territories, indicators, units, analysis methods, instruments, provisions, and the eDNA method catalogues and pipelines (software, taxonomy references, primer pairs, lab protocols). **This stage must be completed first**, before any other data can be entered.

Notebook block: **1. Utilities**, cells 1.1–1.4

### 2. [Dataset metadata]

Dataset metadata describes the provenance of observations: data sources, persons, datasets, campaigns, and sampling logs. Every sample must belong to a sampling log; every sampling log belongs to a campaign; and every campaign belongs to a dataset.

Notebook block: **2. Dataset metadata**, cell 2.1

### 3. [Sample data]

Samples are geolocated soil specimens with depth profiles. Each sample is linked to a sampling log registered in stage 2. Samples must exist before any observation can be inserted.

Notebook block: **3. Load AI4SH data**, cell 3.1

### 4. [Observation data]

All observations except spectra and eDNA: wet laboratory soil properties (pH, organic carbon, nitrogen, texture, ions and more), enzymatic activity, field infiltration, microbial biomass and aggregate stability. Every source is loaded the same way — an observation log that links each sampling log to the provision, then one row of indicator values per sample.

Notebook block: **3.2**, cells 3.2.1 (Agrolab) and 3.2.6–3.2.9 (Digit Soil, BEST infiltration, Microbiometer, aggregate stability). Cells 3.2.2–3.2.5 (other wet laboratories) are pending.

### 5. [Spectral data]

Spectral data covers four instrument types: FOSS DS2500 (NIR), Neospectra (NIR), Bruker Invenio S FTIR (mid-infrared), and FOSS Micral (LIBS). All arrays are stored in ascending wavelength order. Each instrument requires three steps: register the spectrometer, create observation logs, then insert spectral arrays. There is no single-step route here — spectral data is manage-only, with no spreadsheet-translate step to collapse.

Notebook block: **3.3 AI4SH spectral data**, cells 3.3.1–3.3.12

### 6. [eDNA metabarcoding]

eDNA metabarcoding data describes the soil microbial community — prokaryotes (16S rRNA) and fungi (ITS) — per sample: 19 summary indicators (diversity and functional groups), the taxon tree, and the abundance of every amplicon sequence variant (ASV). The laboratory delivers one Excel workbook, which is converted into xspatula source files by a command line script before loading.

Order: run `edna_final_results_to_xspatula.py` → taxa → observation log and observations → ASVs.

Notebook block: **3.4 AI4SH eDNA data**, cells 3.4.1–3.4.6

### 7. [All data in one go]

The complete pipeline: every block and cell of `insert_ai4sh_data.ipynb` in dependency order, and how to switch off cells for the data you don't want to load.

Once everything is loaded, [dump the database][dump] once. A restore takes minutes instead of a full setup and import. The AI4SH data may only be shared within the AI4SoilHealth partners, see [AI4SH data: partners only][partners].

[scheme_file]: https://xspatula.github.io/xspatula_core_docs/framework/scheme_file/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[Utility data]: {{ site.baseurl }}/utility/
[Dataset metadata]: {{ site.baseurl }}/dataset_meta/
[Sample data]: {{ site.baseurl }}/sample/
[Observation data]: {{ site.baseurl }}/observation/
[Spectral data]: {{ site.baseurl }}/spectra/
[eDNA metabarcoding]: {{ site.baseurl }}/edna/
[All data in one go]: {{ site.baseurl }}/all_data/
[dump]: {{ site.baseurl }}/all_data/#dump-once-restore-in-minutes
[partners]: {{ site.baseurl }}/all_data/#ai4sh-data-partners-only
