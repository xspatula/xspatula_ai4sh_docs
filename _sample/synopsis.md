---
title: "Load Sample Data"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "Overview of sample data loading. Samples are geolocated profile observations linked to sampling logs. Two Excel files are translated and managed in three steps."
permalink: /sample/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

A sample in the AI4SH database is a physically collected soil specimen with a known geolocation and depth profile. Each sample is linked to a sampling log, which belongs to a campaign. Samples must exist in the database before any observation data (wetlab or spectral) can be entered.

Samples are loaded by the first cell of block **3. Load AI4SH data** in the single loading notebook (see [Load all data][all_data]):

```
./ai4sh/import_data/insert_ai4sh_data.ipynb   block 3, cell 3.1
```

The cell uses the single-step `insert_tabular_data` route, which is INSERT-only by construction
and therefore always safe to re-run. If you need to `UPDATE` an existing record or want to
hand-inspect the generated JSON before it's applied, use the 2-step notebook
`./ai4sh/import_data/more_notebooks/load_ai4sh_sample_data.ipynb` instead. See
[Insert sample data][insert_sample_data] for the single-step walkthrough and
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Two tables, two source files

Sample loading populates two database tables from two Excel files:

| Excel file | Database table | Description |
|---|---|---|
| `AI4SH_geolocation.xlsx` | `observation.geolocation` | GPS coordinates for each sampling point |
| `AI4SH_samples_geotag.xlsx` | `observation.geolocated_profile_sample` | Samples with depth profile linked to a geolocation and sampling log |

Source directory: `./ai4sh/import_data/sample/excel/`

Both files are translated in a single process file call, then managed separately.

## Sampling logs come from the dataset metadata

Every sample belongs to a sampling log, and in AI4SH the sampling logs are inserted with the
[dataset metadata][dataset_meta] (cell 2.1). Blocks 1 and 2 must therefore have run before cell 3.1.
The 2-step notebook `more_notebooks/load_ai4sh_sample_data.ipynb` starts with an extra cell that
re-runs `manage_sampling_log`; you can skip it if the sampling logs are already in place.

## Required loading sequence

**Single step** (default): **[Insert sample data]** — **cell 3.1 Load AI4SH geolocations**, one
combined process file, translate and insert together, geolocation then geolocated profile sample.

**2-step** (for `UPDATE`s or manual inspection), 3 cells in `more_notebooks/load_ai4sh_sample_data.ipynb`:

1. **[Translate geotag sample]** — translate both Excel files to JSON in one call
2. **[Manage sample geolocation]** — insert geolocation records (coordinates)
3. **[Manage samples]** — insert geolocated profile sample records linked to sampling logs and geolocations

[Insert sample data]: {{ site.baseurl }}/sample/insert_sample_data/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[Translate geotag sample]: {{ site.baseurl }}/sample/translate_geotag_sample/
[Manage sample geolocation]: {{ site.baseurl }}/sample/manage_sample_geolocation/
[Manage samples]: {{ site.baseurl }}/sample/manage_sample/
[dataset_meta]: {{ site.baseurl }}/dataset_meta/
[all_data]: {{ site.baseurl }}/all_data/
[insert_sample_data]: {{ site.baseurl }}/sample/insert_sample_data/
