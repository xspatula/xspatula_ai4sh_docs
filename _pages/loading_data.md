---
title: "Loading AI4SH Data"
layout: single
sidebar:
  nav: "loading_data"
excerpt: "Overview of the six data-loading stages for the AI4SoilHealth database. Data must be loaded in the order shown — each stage depends on records from the previous one."
permalink: /loading_data/
author_profile: false
date: 2026-06-15 08:00:00 +0200
last_modified_at: 2026-06-15 08:00:00 +0200
---

Loading AI4SH data into the database follows a strict dependency order. Each stage builds on records inserted by the previous one. The six stages below must be completed in sequence when populating the database from scratch.

All stages use the same [scheme file][scheme_file] to define the user and general settings:

`./ai4sh/scheme_ai4sh.json`

The scheme file object user_id must be a registered user in the postgreSQL database and the password set either explicitly (password: _"password"_) or in a .netrc file (user_netrc_id: _"user_netrc_id"_).

## Loading stages

### 1. [Utility data]

Utility data contains the controlled vocabularies and reference catalogues that all observation data depends on — territories, indicators, units, analysis methods, instruments, and provisions. **This stage must be completed first**, before any other data can be entered.

Notebook: `load_ai4sh_utility_data.ipynb`

### 2. [Dataset metadata]

Dataset metadata describes the provenance of observations: data sources, persons, datasets, campaigns, and sampling logs. Every sample must belong to a sampling log; every sampling log belongs to a campaign; and every campaign belongs to a dataset.

Notebook: `load_ai4sh_dataset_meta.ipynb`

### 3. [Sample data]

Samples are geolocated soil specimens with depth profiles. Each sample is linked to a sampling log registered in stage 2. Samples must exist before any observation can be inserted.

Notebook: `load_ai4sh_sample_data.ipynb`

### 4. [Wetlab data]

Wet laboratory data contains chemically determined soil properties (pH, organic carbon, nitrogen, texture, ions, and more) measured per sample. Each measurement is anchored to a sample through an observation log.

Notebook: `load_ai4sh_wetlab_data.ipynb`

### 5. [Spectral data]

Spectral data covers four instrument types: FOSS DS2500 (NIR), Neospectra (NIR), FTIR (mid-infrared), and LIBS. All arrays are stored in ascending wavelength order. Each instrument requires three steps: register the spectrometer, create observation logs, then insert spectral arrays.

Notebook: `load_ai4sh_spectral_data.ipynb`

### 6. [All data in one go]

A single notebook that runs all five stages above in the correct dependency order. Use it when loading a complete dataset from scratch in one session.

Notebook: `load_ai4sh_data.ipynb`

[scheme_file]: https://xspatula.github.io/setup_core_db_docs/framework/scheme_file/
[Utility data]: /utility/
[Dataset metadata]: /dataset_meta/
[Sample data]: /sample/
[Wetlab data]: /wetlab/
[Spectral data]: /spectra/
[All data in one go]: /all_data/
