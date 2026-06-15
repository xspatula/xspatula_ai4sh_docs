---
title: "Load All Data"
layout: single
sidebar:
  nav: "all_data"
  nav2: "loading_data"
excerpt: "Running all AI4SH data loading stages from a single Jupyter notebook: load_ai4sh_data.ipynb."
permalink: /all_data/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

`load_ai4sh_data.ipynb` consolidates all data loading stages into a single notebook. It runs the same cells as the five individual notebooks — utility, dataset metadata, sample, wetlab, and spectral — in the correct dependency order. Use it when loading a complete dataset from scratch in one session, or when you already understand the individual workflows and want a streamlined run.

## Notebook location

```
./ai4sh/import_data/load_ai4sh_data.ipynb
```

## Cell sequence

The notebook contains 64 active cells (plus LUCAS cells which are disabled with `%%script false`). The active loading sequence mirrors the individual notebooks:

| Stage | Cells | Notebook equivalent |
|---|---|---|
| Translate general utilities | 1 | [Load utility data] |
| Manage general utilities | 1 | [Load utility data] |
| Translate observation utilities | 1 | [Load utility data] |
| Manage observation utilities | 1 | [Load utility data] |
| Translate observation utilities with inheritance | 1 | [Load utility data] |
| Manage observation utilities with inheritance | 1 | [Load utility data] |
| Translate dataset | 1 | [Load dataset metadata] |
| Manage data source | 1 | [Load dataset metadata] |
| Manage persons | 1 | [Load dataset metadata] |
| Manage dataset | 1 | [Load dataset metadata] |
| Manage campaigns | 1 | [Load dataset metadata] |
| Manage sampling logs | 1 | [Load dataset metadata] |
| Translate geotag sample | 1 | [Load sample data] |
| Manage sample geolocation | 1 | [Load sample data] |
| Manage samples | 1 | [Load sample data] |
| Translate Agrolab observation log | 1 | [Load wetlab data] |
| Manage Agrolab observation log | 1 | [Load wetlab data] |
| Manage Agrolab observation | 1 | [Load wetlab data] |
| Manage FOSS DS2500 spectrometer | 1 | [Load spectral data] |
| Manage FOSS DS2500 observation log | 1 | [Load spectral data] |
| Manage FOSS DS2500 observation | 1 | [Load spectral data] |
| Manage Neospectra spectrometer | 1 | [Load spectral data] |
| Manage Neospectra observation log | 1 | [Load spectral data] |
| Manage Neospectra observation | 1 | [Load spectral data] |
| Manage FTIR spectrometer | 1 | [Load spectral data] |
| Manage FTIR observation log | 1 | [Load spectral data] |
| Manage FTIR observation | 1 | [Load spectral data] |
| Manage LIBS spectrometer | 1 | [Load spectral data] |
| Manage LIBS observation log | 1 | [Load spectral data] |
| Manage LIBS observation | 1 | [Load spectral data] |

## How to run

Each cell is preceded by `#%%script false --no-raise-error`, which disables execution by default. To run a cell, remove or comment out that line. This prevents accidental re-runs when iterating over partially loaded data.

Run the setup cells first (imports and scheme file), then enable and execute the loading cells in order. The sequence is mandatory — earlier stages must complete before later ones can reference their records.

## Scheme file

The notebook uses:

```python
scheme_file = '../scheme_ai4sh.json'
```

The scheme file must exist and point to a valid database connection before any loading cell can run. See the framework documentation for scheme file configuration.

## When to use this notebook vs. individual notebooks

| Situation | Recommendation |
|---|---|
| Loading a complete dataset from scratch | `load_ai4sh_data.ipynb` |
| Re-running one stage after a data correction | Individual notebook for that stage |
| Learning how each stage works | Individual notebooks with detailed docs |
| Production pipeline or automation | `load_ai4sh_data.ipynb` |

[Load utility data]: /utility/
[Load dataset metadata]: /dataset_meta/
[Load sample data]: /sample/
[Load wetlab data]: /wetlab/
[Load spectral data]: /spectra/
