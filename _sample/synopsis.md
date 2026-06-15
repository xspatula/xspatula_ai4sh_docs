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
last_modified_at: 2026-06-10 08:00:00 +0200
---

A sample in the AI4SH database is a physically collected soil specimen with a known geolocation and depth profile. Each sample is linked to a sampling log, which belongs to a campaign. Samples must exist in the database before any observation data (wetlab or spectral) can be entered.

All sample loading is driven by:

```
./ai4sh/import_data/load_ai4sh_sample_data.ipynb
```

## Two tables, two source files

Sample loading populates two database tables from two Excel files:

| Excel file | Database table | Description |
|---|---|---|
| `AI4SH_geolocation.xlsx` | `observation.geolocation` | GPS coordinates for each sampling point |
| `AI4SH_samples_geotag.xlsx` | `observation.geolocated_profile_sample` | Samples with depth profile linked to a geolocation and sampling log |

Source directory: `./ai4sh/import_data/sample/excel/`

Both files are translated in a single process file call, then managed separately.

## Preliminary cell: manage sampling log

The notebook begins with a cell that re-runs `manage_sampling_log` from the dataset metadata workflow. This is a convenience step to ensure sampling logs are up to date before samples are added. If sampling logs are already current from the [dataset metadata workflow][dataset_meta], this cell can be skipped.

## Required loading sequence

1. **[Translate geotag sample]** — translate both Excel files to JSON in one call
2. **[Manage sample geolocation]** — insert geolocation records (coordinates)
3. **[Manage samples]** — insert geolocated profile sample records linked to sampling logs and geolocations

[Translate geotag sample]: /sample/translate_geotag_sample/
[Manage sample geolocation]: /sample/manage_sample_geolocation/
[Manage samples]: /sample/manage_sample/
[dataset_meta]: /dataset_meta/
